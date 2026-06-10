# Database Best Practices

## General Principles

- **Migrations are code**: all schema changes go through versioned migration files, committed to
  version control and applied through CI/CD. Never modify production schemas manually.
- **Connection pooling**: always use a connection pool. Size it based on workload, not guesswork.
  For serverless environments, use an external pooler (PgBouncer, RDS Proxy).
- **Indexes are intentional**: create indexes based on actual query patterns, not guesses. Every
  index has a maintenance cost — justify each one. Composite indexes should match query predicate
  order.
- **Parameterized queries**: never interpolate user input into queries. Use parameterized statements
  or ORM bindings to prevent SQL injection.

## Online Schema Migrations (Expand / Contract)

Production tables often can't tolerate a `LOCK TABLE` for the duration of a
migration. The **expand/contract** pattern (a.k.a. parallel change) ships
schema changes safely without downtime, in three deploys:

1. **Expand**: add the new shape *additively*. Create new column / table /
   index `IF NOT EXISTS`. New code writes to *both* old and new (or just new
   if you're replacing). Old code keeps reading old. Backfill in batches
   (`UPDATE ... WHERE id BETWEEN ? AND ?`) — never one giant `UPDATE`.
2. **Migrate**: switch reads to the new shape once backfill is verified.
   Both paths still functional.
3. **Contract**: drop the old column / table / index. Remove the dual-write
   code.

Specific Postgres patterns:
- **Add NOT NULL column**: add nullable + default + backfill, then
  `ALTER COLUMN ... SET NOT NULL`. Postgres ≥ 12 makes the validation a
  table scan but doesn't rewrite the heap if you set a constant default.
- **Add index**: `CREATE INDEX CONCURRENTLY` (no `ACCESS EXCLUSIVE` lock).
  Watch for `INVALID` index state if it fails — drop and retry.
- **Drop column**: `ALTER TABLE ... DROP COLUMN` is fast (metadata only) but
  app code must already not reference it. Drop *after* the deploy that stops
  using it.
- **Rename**: don't. Add new, dual-write, switch reads, drop old. A direct
  rename forces a coupled deploy of code + schema; expand/contract decouples.

Tools that automate this: `pgroll`, `pt-online-schema-change` (MySQL),
`gh-ost` (MySQL).

## Connection Pool Sizing

A pool that's too big kills the database (each connection costs RAM + a backend
process). Too small starves your app.

Rule of thumb for Postgres: **`pool_size = max(2, num_app_replicas × 1.1)`** of
the database's `max_connections`, and **never more than `(num_cpu_cores × 2) +
effective_spindle_count`** per database (the classic `connections = cores × 2 +
io_devices` heuristic from PgBouncer docs).

Concretely:
- Small services (<10 RPS): 5–10 connections per replica is plenty.
- API services with mixed workload: start at 20 per replica, profile under
  load, raise only if you see `pg_stat_activity` rows in `idle in transaction`
  *and* app-side wait times.
- **Use PgBouncer in transaction-pooling mode** in front of Postgres for
  high-replica-count apps (serverless, Kubernetes deploys with HPA).
  Transaction pooling lets you set tiny app-side pools (2–5 per replica) and
  multiplex through the pooler.

For MySQL: similar logic, but `max_connections` defaults higher. Still cap
per-replica pool size — the database server's CPU is the bottleneck, not the
connection count.

## PostgreSQL

### Query Optimization

- Use `EXPLAIN ANALYZE` to understand query plans. Look for sequential scans on large tables,
  nested loops with high row estimates, and sort operations without indexes.
- Prefer `EXISTS` over `IN` for subqueries. Prefer `JOIN` over correlated subqueries when possible.
- Use partial indexes for queries that consistently filter on a condition:
  `CREATE INDEX idx_active_users ON users (email) WHERE active = true;`
- Use `JSONB` for semi-structured data, with GIN indexes for containment queries. But if you're
  querying specific JSON fields frequently, consider normalizing into columns.
- CTEs are optimization barriers in older Postgres versions. In Postgres 12+, the planner can
  inline CTEs, but `MATERIALIZED` / `NOT MATERIALIZED` hints are available for control.
- Batch inserts: use `INSERT INTO ... VALUES (...), (...), (...)` or `COPY` for bulk loading.
  With Bun ORM (Go), use `db.NewInsert().Model(&items).Exec(ctx)` for bulk inserts.

### Schema Design

- **UUIDv7 for all primary keys** (non-negotiable). UUIDv7 gives you time-sortability for efficient
  B-tree indexing plus global uniqueness. Generate application-side using the stack's UUIDv7 library
  (Go: `google/uuid` v1.6+, Python: `uuid7`, Node: `uuidv7`). Store as `UUID` type in Postgres
  (native 16-byte storage, not text). Migration example:
  ```sql
  CREATE TABLE users (
      id UUID PRIMARY KEY,  -- generated app-side as UUIDv7
      email TEXT NOT NULL UNIQUE,
      created_at TIMESTAMPTZ NOT NULL DEFAULT now()
  );
  ```
  Do not use `gen_random_uuid()` as the generation strategy — it produces UUIDv4 (random, poor
  index locality). Use it only as a column default safety net if desired, but always generate
  UUIDv7 in application code before insert.
- Timestamps: always `timestamptz` (timestamp with time zone), never `timestamp`. Store in UTC.
- Soft deletes: add `deleted_at timestamptz` and filter in queries. Use partial indexes to exclude
  soft-deleted rows from common queries. Soft-delete + UNIQUE constraints
  break naively (you can't re-create a "deleted" email): use a **partial
  unique index** instead of a column-level UNIQUE:
  ```sql
  CREATE UNIQUE INDEX users_email_active_uniq
      ON users (email) WHERE deleted_at IS NULL;
  ```
  This enforces uniqueness only across non-deleted rows.
- Enums: prefer `text` columns with CHECK constraints over Postgres ENUM types (easier to evolve).

### Go Integration (Bun)

Bun handles Postgres naturally. Use `pgdriver` as the driver:
```go
dsn := "postgres://user:pass@localhost:5432/dbname?sslmode=disable"
sqldb := sql.OpenDB(pgdriver.NewConnector(pgdriver.WithDSN(dsn)))
db := bun.NewDB(sqldb, pgdialect.New())
```

### Python Integration

SQLAlchemy 2.0 with `asyncpg` driver for async, `psycopg` for sync.

### Read Replicas

When read load exceeds a single primary's capacity, route reads to a replica:

- **Application-side routing**: maintain two pools (`primary`, `replica`).
  Repository methods accept a `read_consistency` flag; default to `replica`
  for reads, `primary` for writes and read-after-write within a transaction.
- **Replication lag**: monitor `pg_replication_slots.flush_lag` /
  `pg_stat_replication.replay_lag`. Define a tolerance (e.g., 100ms p99) and
  route reads back to the primary on breach.
- **Read-your-writes**: after a write, the same user's next read should hit
  the primary or wait for `pg_wait_for_replay_lsn` on the replica. Don't try
  to be clever with timestamps — use the LSN.
- **PgBouncer + multiple databases**: separate `db_primary` and `db_replica`
  in `pgbouncer.ini`. The app picks via DSN.

## MySQL

### Query Optimization

- InnoDB is the only acceptable storage engine. Verify table engine before optimizing.
- Use covering indexes where possible — include all columns referenced in SELECT, WHERE, and ORDER BY.
- Avoid `SELECT *`. Specify columns to benefit from covering indexes and reduce I/O.
- For pagination, use keyset pagination (`WHERE id > ? ORDER BY id LIMIT ?`) instead of `OFFSET`
  on large tables. OFFSET scans and discards rows.
- Use `FORCE INDEX` hints sparingly and only when the optimizer consistently makes wrong choices.
  Document why.
- JSON support: MySQL 8.0+ has `JSON` type with functional indexes. Less mature than Postgres
  JSONB but usable for semi-structured data.

### Schema Design

- `utf8mb4` charset and `utf8mb4_unicode_ci` collation for all text columns. Never `utf8` (it's
  3-byte, not real UTF-8).
- **UUIDv7 for all primary keys**. Store as `BINARY(16)` for efficient indexing and compact storage.
  Use `UUID_TO_BIN(?, 1)` for inserts and `BIN_TO_UUID(id, 1)` for reads (the `1` flag swaps the
  time-high and time-low bytes for optimal B-tree ordering with time-based UUIDs). Generate UUIDv7
  application-side; MySQL's `UUID()` function generates v1, not v7. Migration example:
  ```sql
  CREATE TABLE users (
      id BINARY(16) PRIMARY KEY,
      email VARCHAR(255) NOT NULL UNIQUE,
      created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
  ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
  ```
  In application code, always generate UUIDv7 and convert to binary before insert.
- Timestamps: `DATETIME` with explicit UTC convention, or `TIMESTAMP` (auto-converts to UTC).

### Go Integration (Bun)

```go
dsn := "user:pass@tcp(localhost:3306)/dbname"
sqldb, _ := sql.Open("mysql", dsn)
db := bun.NewDB(sqldb, mysqldialect.New())
```

## DynamoDB

### Data Modeling

- **Single-table design**: prefer one table with carefully designed partition key (PK) and sort key
  (SK) patterns. Use GSIs for alternate access patterns. Multiple tables are acceptable for truly
  independent domains.
- Design access patterns first, then model the table. DynamoDB is not relational — you can't
  decide the schema and add queries later.
- PK/SK naming: use generic names (`PK`, `SK`) with prefixed values (`USER#<uuidv7>`,
  `ORDER#2024-01-01`). Generate UUIDv7 application-side for all entity identifiers. The time-sorted
  property of UUIDv7 makes it especially useful as sort key components for chronological queries
  (e.g., `SK: "ORDER#<uuidv7>"` naturally sorts by creation time).
- Use sparse GSIs: only items with the GSI attributes appear in the index, making them efficient
  for filtered queries.

### Query Patterns

- **Never scan in production**. Always use `Query` or `GetItem`. If you need a scan, you probably
  have a modeling problem.
- Use `begins_with()` on sort keys for hierarchical queries.
- Batch operations: `BatchGetItem` (max 100 items) and `BatchWriteItem` (max 25 items). Handle
  unprocessed items in retry logic.
- Transactions: `TransactWriteItems` for atomic multi-item writes. Max 100 items per transaction.

### Go Integration

Use AWS SDK v2 (`aws-sdk-go-v2/service/dynamodb`). Wrap in a repository adapter.

### Python Integration

`boto3` (sync) or `aiobotocore` (async). Use `boto3.resource('dynamodb')` for higher-level API.

## MongoDB

### Data Modeling

- **Embed by default**: if data is queried together, embed it. Normalize (reference) only when
  documents would exceed 16MB, when the embedded data changes independently at high frequency,
  or when many-to-many relationships exist.
- Design schemas around query patterns. Denormalization is expected and encouraged.
- **Use UUIDv7 for `_id`** instead of the default ObjectId. ObjectId is time-sortable but only
  12 bytes and MongoDB-specific. UUIDv7 gives you globally unique, time-sortable identifiers that
  are portable across databases. Set `_id` explicitly on insert:
  ```javascript
  { _id: uuidv7(), email: "user@example.com", ... }
  ```
  In Go, use `google/uuid` to generate and store as BSON Binary subtype 4. In Python, use `uuid7`
  and pass as a standard `uuid.UUID` — pymongo/motor handle the BSON conversion.

### Query Optimization

- Create indexes for every query pattern. Use `explain()` to verify index usage.
- Compound indexes: field order matters. Equality → sort → range (ESR rule).
- Use projections to return only needed fields. Reduces network transfer and memory.
- Aggregation pipeline: prefer `$match` and `$project` early in the pipeline to reduce the
  working set. `$lookup` is expensive — consider embedding if you're joining frequently.

### Go Integration

Use the official `mongo-go-driver`. Wrap in repository adapters.

### Python Integration

`pymongo` (sync) or `motor` (async).

## Supabase

Supabase is Postgres under the hood. All PostgreSQL best practices above apply.

### Additional Supabase Patterns

- **UUIDv7 for all primary keys** — same as Postgres above. When using Supabase client libraries,
  generate UUIDv7 client-side (JS: `uuidv7` package, Python: `uuid7` package) and pass it as the
  `id` field. Do not rely on `gen_random_uuid()` default — it produces UUIDv4.
- Use Row Level Security (RLS) policies for multi-tenant or user-scoped data. Enable RLS on
  every table and define policies explicitly.
- Use Supabase client libraries (`@supabase/supabase-js` for JS/TS, `supabase-py` for Python)
  for auth-aware queries from the client.
- Edge Functions (Deno) for serverless logic. Keep them thin — business logic belongs in your
  backend service, not in edge functions.
- Realtime subscriptions for live data. Use channels and filters to scope subscriptions narrowly.
- Migrations: use `supabase db diff` and `supabase migration new` for local-first migration
  workflows.

## Firebase (Firestore & RTDB)

### Firestore

- **Document-collection model**: keep documents small and queries simple. Denormalize aggressively.
- **UUIDv7 for document IDs**: instead of Firestore's auto-generated IDs, set document IDs to
  UUIDv7 values. This gives you globally unique, time-sortable IDs that are consistent with your
  other databases. Generate client-side or server-side before writing:
  ```javascript
  // JS/TS
  import { uuidv7 } from "uuidv7";
  await db.collection("users").doc(uuidv7()).set({ ... });
  ```
- **Composite indexes**: Firestore requires composite indexes for compound queries. Plan indexes
  as part of data modeling — they're not afterthoughts.
- Security Rules: define granular rules. Never ship with open rules (`allow read, write: if true`).
  Test rules with the Firebase Emulator.
- **Subcollections** for hierarchical data that needs independent querying (e.g., `/users/{uid}/orders`).
- Batch writes for atomicity (max 500 operations). Use transactions for read-then-write patterns.

### Go Integration

Use `cloud.google.com/go/firestore`. Wrap in repository adapters.

### Python Integration

`firebase-admin` SDK.

### Firebase RTDB (Realtime Database)

- Use only when Firestore is overkill (e.g., simple presence, typing indicators).
- Keep data flat — deep nesting kills query performance.
- Security rules are JSON-based. Test with the emulator.
