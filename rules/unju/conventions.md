# Unju Coding Conventions

These rules apply to all unju-ai repositories. They override ECC defaults where they conflict.

## Package manager: Bun

ALWAYS use bun, never npm or yarn in new commands:
- `bun install` (not npm install)
- `bun run <script>` (not npm run)
- `bun add <pkg>` (not npm install)
- `bun add -d <pkg>` (not npm install --save-dev)

Exception: legacy scripts that still reference npm are acceptable until updated.

## Error responses

API errors MUST use the `{ error: string }` shape. Never use `{ message: ... }`.
Status codes must be correct (400 for validation, 401 for unauth, 403 for forbidden, 404 for not found, 500 for server error).

## Cloudflare Workers specifics

- `wrangler.toml` routes must use `[[routes]]` (TOML array), not `[routes]` (TOML table)
- Always run `bun run build` (wrangler dry-run) before committing Dockerfile or wrangler changes
- Access bindings via `c.env.BINDING_NAME` — never `process.env`
- If you see "Expected routes to be an array" — the wrangler.toml is wrong

## OpenAPI spec

- Never hand-edit `openapi.json`
- Every public endpoint must be added to `scripts/generate-openapi.ts`
- Regenerate after any route change: `bun run scripts/generate-openapi.ts`

## Database

- Column names come from `src/lib/kysely/generated-types.ts` — the authoritative source
- Never write queries against column names you haven't verified in `generated-types.ts`
- Migrations are append-only — never modify existing migration files

## Security (extra vigilant for this stack)

- Wallet/on-chain code: always reviewed by security-reviewer agent before commit
- SIWE verification: always verify the full message matches the expected format
- Never log private keys, session tokens, or SIWE signatures
- Service tokens (HMAC-SHA256) have 60s TTL — always check `ts` claim validity

## Commit style

```
type(scope): short description

Body optional. Types: feat, fix, chore, docs, refactor, test, perf
Scope: the domain/repo area (e.g. wallet, auth, openapi, cli)
```

Never commit:
- `console.log` or `console.debug` statements
- Hardcoded API keys, secrets, or wallet private keys
- `TODO` comments without a corresponding GitHub issue number
