# Skill: Cloudflare Workers

## When to Use

When working with Cloudflare Workers, Wrangler, Hono, Durable Objects, D1, KV, R2, Service Bindings, or Queues. Use this skill before writing any Worker code to avoid the most common pitfalls.

## Wrangler.toml

**Routes MUST be an array:**
```toml
# ✓ CORRECT
[[routes]]
pattern = "api.unju.ai/*"
zone_name = "unju.ai"

# ✗ WRONG (breaks every deploy)
[routes]
pattern = "api.unju.ai/*"
```

**D1 database binding:**
```toml
[[d1_databases]]
binding = "UNJU_DB"
database_name = "unju"
database_id = "your-actual-d1-id"
```

**Service bindings (for inter-worker calls):**
```toml
[[services]]
binding = "WALLET"
service = "unju-wallet"
```

**Verify config before pushing:**
```sh
bun run build    # runs: wrangler deploy --dry-run --outdir dist
```

## Hono patterns

```typescript
import { Hono } from 'hono'
import type { Env } from '../index'

// Always type the bindings
export const myRouter = new Hono<{ Bindings: Env }>()

myRouter.get('/path', async (c) => {
  const db = c.env.UNJU_DB  // D1 binding
  return c.json({ ok: true })
})
```

## D1 with Kysely

```typescript
import { Kysely } from 'kysely'
import { D1Dialect } from 'kysely-d1'
import type { DB } from './generated-types'

export function getDb(env: { UNJU_DB: D1Database }): Kysely<DB> {
  return new Kysely<DB>({ dialect: new D1Dialect({ database: env.UNJU_DB }) })
}

// Column names come from generated-types.ts — never guess
// If you get a type error on a column, check generated-types.ts first
```

## Node.js compatibility

Some packages need `nodejs_compat` flag:
```toml
compatibility_flags = ["nodejs_compat"]
```
If you see "package X wasn't found on the file system but is built into node" — add this flag.

## Durable Objects

```typescript
export class MyDO implements DurableObject {
  state: DurableObjectState
  constructor(state: DurableObjectState, env: Env) {
    this.state = state
  }
  async fetch(request: Request): Promise<Response> { ... }
}

// Register in wrangler.toml:
// [[durable_objects.bindings]]
// name = "MY_DO"
// class_name = "MyDO"
```

## Secrets

```sh
# Set in production (never commit)
wrangler secret put SECRET_NAME
# In code: c.env.SECRET_NAME
```

## Size limits

- Workers: 10MB (after gzip ≈ 1MB limit for free tier)
- D1: 2GB max, 10MB per row
- KV: 25MB max value
- If bundle too large: split into multiple workers with Service Bindings

## Testing locally

```sh
wrangler dev           # local dev with live reload
wrangler dev --remote  # dev against remote D1/KV bindings
bun run build          # dry-run deploy check (no actual deploy)
```

## Common mistakes

1. `[routes]` instead of `[[routes]]` — breaks every deploy
2. Missing `nodejs_compat` flag for Node builtins
3. Using `process.env` instead of `c.env` in Workers
4. Blocking I/O — Workers are async-only, use `await`
5. Importing Node.js modules that aren't CF-compatible
6. Hand-editing `openapi.json` — always regenerate from the script
