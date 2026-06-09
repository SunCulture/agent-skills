# Queue & Messaging Best Practices

## General Principles

Regardless of the broker, these patterns apply:

- **Idempotent consumers**: design every consumer to handle duplicate messages safely. Use
  idempotency keys (UUIDv7-based), deduplication tables, or conditional writes.
- **Message IDs**: use UUIDv7 for all message/correlation/idempotency IDs. The time-sorted property
  makes debugging and log correlation straightforward — IDs naturally sort by creation time.
- **Dead letter handling**: always configure a dead letter queue/topic. Messages that fail after
  N retries go to DLQ for inspection, not into the void.
- **Schema evolution**: use versioned message schemas. Include a `version` field or use a schema
  registry. Consumers must handle unknown fields gracefully.
- **Observability**: instrument producers and consumers with metrics (messages produced/consumed,
  lag, error rate) and distributed tracing (propagate trace context in message headers).
- **Graceful shutdown**: consumers must finish processing the current message before shutting down.
  Handle SIGTERM properly.
- **Connection resilience**: implement automatic reconnection with exponential backoff for all
  broker connections.
- **Trace context propagation**: every outbound message carries W3C trace
  context (`traceparent`, `tracestate`) in headers. Consumers extract it
  to continue the trace. See `references/observability.md` for the
  service-side instrumentation; OTel's messaging propagators handle the
  serialization for AMQP, Kafka, and NATS.

## The Transactional Outbox Pattern

Whenever you must atomically (a) commit a database change AND (b) publish a
message, **do not** publish from inside the transaction or right after commit.
Both fail-modes lose data: pre-commit publish leaks a message that the DB
later rolls back; post-commit publish drops the message if the process
crashes between the two.

The pattern:

1. In the same transaction as the business write, insert a row into an
   `outbox` table:
   ```sql
   CREATE TABLE outbox (
       id            UUID PRIMARY KEY,        -- UUIDv7
       aggregate_type TEXT NOT NULL,
       aggregate_id  UUID NOT NULL,
       event_type    TEXT NOT NULL,
       payload       JSONB NOT NULL,
       headers       JSONB NOT NULL DEFAULT '{}'::jsonb,
       created_at    TIMESTAMPTZ NOT NULL DEFAULT now(),
       published_at  TIMESTAMPTZ
   );
   CREATE INDEX outbox_unpublished
       ON outbox (created_at) WHERE published_at IS NULL;
   ```
2. A separate **relay** process polls `outbox WHERE published_at IS NULL` (or
   uses Postgres logical decoding via Debezium for change-data-capture — same
   pattern, different mechanism), publishes to the broker, then sets
   `published_at = now()`. Use `FOR UPDATE SKIP LOCKED` for safe parallel
   relays.
3. Consumers must be **idempotent** (next section) — the relay can publish
   the same message twice during crash recovery.

This is at-least-once delivery with no message loss, and no XA / two-phase
commit ceremony. It's the right answer 95% of the time you think you need
"transactional messaging."

## Idempotency in Practice

Idempotent consumer = same message delivered twice produces the same final
state. Two patterns:

1. **Idempotency key + dedup table**: each message has a UUIDv7 idempotency
   key. Consumer inserts into a `processed_messages(message_id PRIMARY KEY,
   processed_at)` table inside the same transaction as the business write.
   On duplicate-key error, skip. Trim old entries on a schedule.
2. **Conditional writes**: design the business operation so the second
   application is a no-op (`UPDATE ... WHERE state = 'pending'`,
   `INSERT ... ON CONFLICT DO NOTHING`). Avoids the dedup table entirely
   when feasible.

Pick (1) for arbitrary side effects, (2) for state transitions you control.

## RabbitMQ

### Patterns

- Use **topic exchanges** for flexible routing. Direct exchanges for simple point-to-point.
  Fanout for broadcast. Avoid the default exchange in production.
- Declare exchanges and queues with `durable: true`. Messages should be `persistent` (delivery
  mode 2) for anything that matters.
- Use **publisher confirms** (not transactions) for reliable publishing.
- Prefetch count: set `prefetch` (QoS) to match your consumer's throughput. Start with 10-20 and
  tune. Never leave it unlimited.
- **Acknowledge after processing**, not before. Use manual ack mode. Nack with requeue for transient
  failures; nack without requeue (or reject) to send to DLQ.

### Go Integration

```go
// In adapters/queue/rabbitmq.go — implements a port interface
type RabbitMQPublisher struct {
    channel *amqp.Channel
    exchange string
}

func (p *RabbitMQPublisher) Publish(ctx context.Context, routingKey string, msg []byte) error {
    return p.channel.PublishWithContext(ctx,
        p.exchange, routingKey, false, false,
        amqp.Publishing{
            ContentType:  "application/json",
            Body:         msg,
            DeliveryMode: amqp.Persistent,
            MessageId:    uuid.Must(uuid.NewV7()).String(),
            Timestamp:    time.Now(),
        },
    )
}
```

### Python Integration

Use `aio-pika` for async or `pika` for sync. Wrap in an adapter class that implements the outbound
port interface.

## Kafka

### Patterns

- **Topics**: name as `<domain>.<entity>.<event>` (e.g., `payments.order.completed`).
- **Partitioning**: choose partition keys that distribute load evenly while preserving ordering
  where needed (e.g., user ID for user events ensures per-user ordering).
- **Consumer groups**: one group per logical consumer. Set `enable.auto.commit = false` and commit
  offsets after successful processing.
- **Exactly-once semantics**: use idempotent producers (`enable.idempotence = true`) and transactional
  producing where cross-topic atomicity is needed.
- **Retention**: set retention based on use case. Event sourcing: indefinite or very long. Standard
  messaging: 7 days default.
- **Schema Registry**: use Confluent Schema Registry with Avro or Protobuf schemas. JSON Schema
  as a lighter alternative.

### Go Integration

Use `confluent-kafka-go` (librdkafka-based, production-grade) or `segmentio/kafka-go` (pure Go,
simpler API).

### Python Integration

Use `confluent-kafka` (production) or `aiokafka` (async).

## Redis (Pub/Sub & Streams)

### Pub/Sub

- Use for ephemeral, fire-and-forget messaging where message loss is acceptable (e.g., cache
  invalidation, real-time notifications).
- Not suitable for reliable messaging — if the subscriber is down, messages are lost.
- Keep channel names namespaced: `app:domain:event`.

### Redis Streams (Preferred for Reliable Messaging)

- Use Streams with consumer groups for reliable, persistent messaging via Redis.
- `XADD` to produce, `XREADGROUP` to consume, `XACK` to acknowledge.
- Set `MAXLEN` or `MINID` to cap stream size and prevent unbounded growth.
- Handle pending entries (`XPENDING` + `XCLAIM`) for messages that weren't acknowledged — this is
  your retry mechanism.

### Go Integration

Use `redis/go-redis` v9+. Wrap stream operations in adapter methods.

### Python Integration

Use `redis-py` with async support (`redis.asyncio`).

## NATS

### Patterns

- **Core NATS**: for lightweight pub/sub where at-most-once delivery is fine.
- **JetStream**: for persistent messaging with at-least-once or exactly-once delivery. Always
  prefer JetStream for production workloads that need reliability.
- **Subjects**: dot-separated hierarchy (`orders.created`, `orders.*.shipped`). Use wildcards
  (`*` single token, `>` multi-token) for flexible subscriptions.
- **Queue groups**: for load-balanced consumption across multiple instances of the same service.
  Automatic — just use the same queue group name.

### JetStream Configuration

- Define streams explicitly with retention policy (limits, interest, or work-queue).
- Use durable consumers with explicit ack for reliable processing.
- Configure `MaxDeliver` for retry limits and a dead letter subject for failed messages.

### Go Integration

Use `nats-go` (official client). JetStream API is built into the same library.

### Python Integration

Use `nats-py` (official async client).

## Replay and Catch-Up Consumers

When you need a new consumer to process historical messages — for example,
adding a new projection or recovering from a bad deploy that consumed but
mis-processed events:

- **Kafka**: `--from-beginning` or seek to a specific offset / timestamp via
  `kafka-consumer-groups.sh --reset-offsets`. Always do this with the consumer
  group **stopped**.
- **Redis Streams**: read from `0` (start) instead of `>` (new only). Use a
  separate consumer group for catch-up so live consumers aren't disturbed.
- **NATS JetStream**: create a new durable consumer with
  `DeliverPolicy=DeliverAll` or `OptStartTime` set to the desired point.
- **RabbitMQ**: classic queues delete messages on ack — no replay. For replay,
  use **RabbitMQ Streams** (RMQ 3.9+) or persist events to the outbox
  table and replay from there.
- **Outbox-based event log**: replay from the `outbox` table itself by
  resetting `published_at`. The relay re-publishes to a dedicated replay
  topic that the new consumer subscribes to.

Always isolate replay traffic on a separate topic / queue / consumer group
so live consumers aren't impacted.
