# Frontend & Node.js Best Practices

## TypeScript: Always

All JavaScript/TypeScript projects use **strict TypeScript**. No `any` unless truly unavoidable (and
if so, comment why). Enable `strict: true` in `tsconfig.json`.

- Prefer `interface` for object shapes, `type` for unions and utility types.
- Use discriminated unions over optional fields where the shape varies by case.
- Zod for runtime validation at boundaries (API responses, form inputs, env vars).
- Barrel exports (`index.ts`) for public API of modules; avoid deep import paths.

## Next.js (React)

### Architecture

Use the **App Router** (Next.js 13+). Server Components by default; Client Components only where
interactivity is needed (`"use client"` directive).

```
src/
├── app/
│   ├── layout.tsx            # root layout
│   ├── page.tsx              # home page
│   ├── (auth)/               # route group
│   │   ├── login/page.tsx
│   │   └── register/page.tsx
│   └── api/                  # route handlers
│       └── users/route.ts
├── components/
│   ├── ui/                   # generic reusable components
│   └── features/             # feature-specific components
├── lib/
│   ├── actions/              # server actions
│   ├── hooks/                # custom React hooks
│   ├── utils/                # pure utility functions
│   └── types/                # shared TypeScript types
├── services/                 # API client / data fetching layer
└── styles/
```

### Key Patterns

- **Server Actions** for mutations. Define in `lib/actions/` with `"use server"`.
- **Data fetching**: fetch in Server Components directly, or use React Query / SWR for client-side.
  Keep fetching logic in `services/` — components don't call APIs directly.
- **Error boundaries**: use `error.tsx` files in route segments. Global error boundary at app root.
- **Loading states**: use `loading.tsx` or Suspense boundaries.
- **Middleware**: `middleware.ts` at project root for auth checks, redirects, headers.
- Styling: Tailwind CSS preferred. CSS Modules acceptable. No styled-components with Server Components.

## Nuxt.js (Vue)

### Architecture

Use Nuxt 3 with the Composition API and `<script setup>` syntax exclusively.

```
├── app.vue
├── pages/
│   ├── index.vue
│   └── users/
│       ├── index.vue
│       └── [id].vue
├── components/
│   ├── ui/
│   └── features/
├── composables/              # auto-imported composable functions
│   ├── useAuth.ts
│   └── useApi.ts
├── server/
│   ├── api/                  # Nitro API routes
│   │   └── users/
│   │       ├── index.get.ts
│   │       └── [id].get.ts
│   ├── middleware/
│   └── utils/
├── stores/                   # Pinia stores
├── types/
└── utils/
```

### Key Patterns

- **Composables** for reusable logic (`composables/` directory, auto-imported by Nuxt).
- **Pinia** for state management. Define stores in `stores/` with the setup store syntax.
- **Server routes**: use Nitro's file-based routing in `server/api/`. Validate with Zod.
- **`useFetch` / `useAsyncData`** for data fetching — these are SSR-aware.
- **Auto-imports**: Nuxt auto-imports Vue APIs, composables, and utilities. Don't manually import
  `ref`, `computed`, `watch`, etc.
- Styling: Tailwind CSS or UnoCSS preferred.

## React (Standalone / Vite)

When React is used outside Next.js (e.g., with Vite, CRA, or as a library):

- Functional components only. No class components.
- Custom hooks extract all non-trivial logic from components.
- State management: Zustand for global state (lightweight), React Query/TanStack Query for server
  state. Redux only if the project already uses it.
- Component composition over prop drilling. Use React Context sparingly — mainly for theme, auth,
  and locale.
- Memoization (`useMemo`, `useCallback`, `React.memo`) only when profiling shows a need. Don't
  prematurely optimize.

## Vue (Standalone)

When Vue 3 is used outside Nuxt:

- Composition API with `<script setup>` exclusively. No Options API.
- Pinia for state management.
- Vue Router for routing, with typed route params.
- VueUse for common composables (don't reinvent what VueUse provides).

## Cross-Cutting Frontend Concerns

These apply to every Next.js / Nuxt / React / Vue project, not just Node backends.

### Accessibility (a11y)

- **Baseline**: WCAG 2.2 AA. Every interactive element is keyboard-reachable
  with a visible focus state. Every image has `alt` (decorative → `alt=""`).
  Every form field has an associated `<label>`.
- **Use semantic HTML**: `<button>` not `<div onClick>`, `<nav>`, `<main>`,
  `<header>`, headings in order (no skipping `<h2>` to `<h4>`).
- **Lint**: `eslint-plugin-jsx-a11y` (React) or `eslint-plugin-vue` with
  `vue/no-v-html` and a11y rules. CI fails on a11y errors, not just warnings.
- **Test**: `axe-core` via `@axe-core/playwright` in E2E or
  `vitest-axe` in component tests. Run on at least one snapshot per route.

### Performance Budgets / Core Web Vitals

- **Targets** (mobile, p75): LCP ≤ 2.5s, INP ≤ 200ms, CLS ≤ 0.1.
- **Bundle budget**: per-route JS ≤ 200KB gzipped for the critical path.
  Use `@next/bundle-analyzer` or `nuxt-vite-bundle-analyzer` in CI.
- **Telemetry**: ship Web Vitals to a real-user monitoring (RUM) endpoint
  via `web-vitals` package. Don't trust lab tests alone.
- **Images**: `next/image` (Next), `<NuxtImg>` (Nuxt). Always specify width
  and height to prevent CLS. AVIF/WebP via the framework's built-in
  optimizer.

### Environment Variable Validation

Never read `process.env.X` raw — typos and missing values become silent
production bugs. Validate at startup with a typed schema:

```typescript
// src/lib/env.ts (Next/Node)
import { z } from "zod";

const envSchema = z.object({
  NODE_ENV: z.enum(["development", "test", "production"]),
  DATABASE_URL: z.string().url(),
  NEXT_PUBLIC_API_URL: z.string().url(),
});

export const env = envSchema.parse(process.env);
```

Or use `@t3-oss/env-nextjs` for the split between server-only and `NEXT_PUBLIC_*`
variables enforced at the type level.

### Error Tracking

- **Sentry** (preferred) or **Datadog RUM** wired up in `instrumentation.ts`
  (Next.js) or a Nuxt plugin. Capture `request_id` from the response header
  and tag every error with it for cross-stack correlation
  (see `references/observability.md`).
- **Error boundaries**: route-level (`error.tsx` in Next, `<NuxtErrorBoundary>`
  in Nuxt) plus a global one. Don't render the raw error message — show a
  user-friendly state and log the detail to Sentry.

### Internationalization

- **Next**: `next-intl` (preferred for App Router) or `next-i18next`.
- **Nuxt**: `@nuxtjs/i18n`.
- Locale-detect via `Accept-Language`, persist explicit user choice in a cookie.
- Never concatenate translated strings (`t("you_have") + " " + n + " " + t("items")`)
  — use ICU `MessageFormat` placeholders.

## Express.js

Express is for backend Node.js services — not frontend:

```
src/
├── routes/
│   ├── index.ts              # route registration
│   └── users.route.ts
├── controllers/              # request handling, validation, response
├── services/                 # business logic
├── repositories/             # data access
├── middleware/
│   ├── auth.ts
│   ├── errorHandler.ts
│   └── validation.ts
├── types/
├── utils/
└── app.ts                    # Express app setup (not server listen)
```

### Key Patterns

- Layered architecture: routes → controllers → services → repositories. Each layer has a single
  responsibility.
- **UUIDv7 for all entity IDs**: use the `uuidv7` npm package. Generate IDs in the service or
  repository layer before insert — never rely on database auto-increment:
  ```typescript
  import { uuidv7 } from "uuidv7";
  const user = { id: uuidv7(), email, ...data };
  ```
- Error handling middleware as the last middleware. Use a custom `AppError` class with status codes.
- Validation middleware using Zod: validate `req.body`, `req.params`, `req.query` before the
  controller runs.
- Never `app.listen()` in the app module — export the app and start it in `server.ts` or `index.ts`
  (makes testing with supertest clean).
- **Express 5** (current default since Oct 2024) handles `Promise`-rejecting
  async handlers natively — `express-async-errors` is no longer needed. On
  Express 4, install `express-async-errors` or wrap handlers manually.
- Helmet for security headers, cors for CORS, **pino** for logging
  (see `references/observability.md` — pino is the JSON logger of choice).
- Observability: see `references/observability.md` for request-ID middleware,
  `/healthz`/`/readyz`, OpenTelemetry auto-instrumentation
  (`@opentelemetry/auto-instrumentations-node`), and SIGTERM handling.

## Testing (All Frontend/Node)

- **Framework**: Vitest (preferred for Vite-based projects, also works with Next.js) or Jest.
- **Component testing**: React Testing Library (React/Next) or Vue Testing Library (Vue/Nuxt).
  Test behavior, not implementation. Query by role/text, not by class/id.
- **API route testing**: Supertest for Express. For Next.js route handlers, use the built-in
  test utilities or HTTP-level tests.
- **E2E**: Playwright (preferred) or Cypress for critical user flows.
- **Mocking**: `vi.mock()` (Vitest) or `jest.mock()` for modules. MSW (Mock Service Worker) for
  API mocking in tests and development.
- Test file convention: `*.test.ts` or `*.spec.ts` colocated with source files.

## Docker (Node.js apps)

Three stages: install dev deps for build, install prod-only deps separately,
run as non-root.

```dockerfile
# syntax=docker/dockerfile:1.7
# Stage 1 — install all deps and build
FROM node:22-alpine@sha256:<digest> AS builder
WORKDIR /app
COPY package.json package-lock.json ./
RUN --mount=type=cache,target=/root/.npm \
    npm ci --include=dev
COPY . .
RUN npm run build

# Stage 2 — install prod-only deps in a clean layer
FROM node:22-alpine@sha256:<digest> AS prod-deps
WORKDIR /app
COPY package.json package-lock.json ./
RUN --mount=type=cache,target=/root/.npm \
    npm ci --omit=dev

# Stage 3 — runtime, non-root, distroless-ish
FROM node:22-alpine@sha256:<digest>
RUN addgroup -S app && adduser -S app -G app
WORKDIR /app
COPY --from=prod-deps --chown=app:app /app/node_modules ./node_modules
COPY --from=builder  --chown=app:app /app/.next ./.next         # Nuxt: /app/.output
COPY --from=builder  --chown=app:app /app/public ./public
COPY --from=builder  --chown=app:app /app/package.json ./
USER app
EXPOSE 3000
ENV NODE_ENV=production
CMD ["npm", "start"]
```

Adjust `COPY` paths for the framework's build output (Nuxt → `.output/`,
Vite → `dist/`, plain Node → `dist/`). Resolve `<digest>` via
`docker pull node:22-alpine && docker images --digests`.

`.dockerignore`:

```
.git
.github
node_modules
.next
.nuxt
.output
dist
build
coverage
.env
.env.*
!.env.example
**/*.test.ts
**/*.spec.ts
*.md
```

## Makefile

```makefile
.PHONY: install dev build test lint docker-build docker-up seed clean

install:
	npm ci

dev:
	npm run dev

build:
	npm run build

test:
	npm run test

lint:
	npm run lint

docker-build:
	docker build -t $(APP_NAME) .

docker-up:
	docker compose up -d

seed:
	npm run seed

clean:
	rm -rf node_modules .next .nuxt .output dist build coverage
```
