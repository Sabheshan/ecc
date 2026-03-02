# Skill: Unju Stack

## When to Use

When working on any unju-ai repository. This is the authoritative reference for unju's architectural decisions, conventions, and patterns that differ from generic Node.js/TypeScript advice.

## Repository Map

| Repo | Language | Role |
|------|----------|------|
| `unju-api` | TypeScript / Hono | Main API, Cloudflare Workers |
| `unju-realtime` | TypeScript / CF Workers | Gemini Realtime WS proxy |
| `unju-wallet` | TypeScript / CF Workers | On-chain (EVM) operations |
| `unju-ionic` | TypeScript / Next.js + React | Frontend (OpenNext → CF) |
| `unju-swift` | Swift | macOS/iOS app |
| `unju-vm` | Dockerfile / Python | Agent workspace container |
| `unju-a2a` | Solidity | On-chain agent identity contracts |
| `nanoclaw` | TypeScript | Swarm runtime (private fork) |

## Architecture Decisions (ADRs)

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Chain | HyperEVM (not SKALE) | Production EVM, EIP-7212 P-256 precompile |
| Auth | SIWE + Better Auth | EVM-native identity + email/OAuth fallback |
| DB | Cloudflare D1 (SQLite) | Edge-native, near zero cold start |
| ORM | Kysely (not Drizzle/Prisma) | Type-safe SQL, CF-compatible |
| Runtime | Bun (not npm/node) | Faster installs, built-in bundler |
| API compat | OpenAI-compatible endpoints | Drop-in for existing tooling |
| Agent identity | AgentNFT (ERC-721) + ERC-6551 TBA | On-chain identity + token account |
| Realtime | LiveKit | Self-hosted, not vendor-locked |
| Frontend | OpenNext (Cloudflare) | Next.js → CF Workers (same infra) |

## Bun-first

```sh
bun install           # not npm install
bun run <script>      # not npm run
bun add <pkg>         # not npm install <pkg>
bun add -d <pkg>      # not npm install --save-dev
bun build --compile   # for CLI binaries (bun runtime embedded)
```

## Agent Swarm Identity

Each swarm agent has:
- **Token ID** (numeric) — unique identity in the swarm
- **AgentNFT** — ERC-721 on HyperEVM
- **ERC-6551 TBA** — smart wallet derived from AgentNFT
- **SSH key fingerprint** — stored as AgentNFT metadata attribute (not signer)
- **secp256k1 key** — the actual on-chain signer (via viem)

Known agents:
- Token 0: Sera (Engineer)
- Token 1: Yafu (Researcher)
- Token 2: Kimiko (Companion)
- Token 3: Esper (Orchestrator)
- Token 4: Bhaiṣajyaguru (Reviewer — `sera-plz` on GitHub)
- Token 5: Green Tara (White Hat — `hiddendragonXVII`)
- Token 6: Lakshmi (Revenue)
- Token 7: Saraswati (Knowledge)

## ACP (Agent Communication Protocol)

Internal swarm messaging:
```typescript
// D1-backed inbox, dual-delivered via LiveKit
// Daemon: listens on LiveKit room `agent:{agentId}`
// REST: POST /v1/acp/messages

// CLI: unju acp send --to agent-id --message "..."
```

## On-chain (unju-a2a)

```typescript
// AgentNFT: ERC-721 on HyperEVM
// ERC-6551 TBA: smart wallet per agent
// TaskEscrow: payment in UnjuCredits (USDC-backed)
// Never use SKALE — chain is HyperEVM
```

## Error patterns

Always return typed error responses:
```typescript
// ✓ Always
return c.json({ error: 'User not found' }, 404)

// ✗ Never
return c.json({ message: 'User not found' }, 404)
// ✗ Never
return c.text('error', 500)
```

## Environment variables

Access via `c.env` in Hono (not `process.env`):
```typescript
const apiKey = c.env.OPENAI_API_KEY   // ✓
const apiKey = process.env.OPENAI_API_KEY  // ✗ doesn't work in Workers
```

## Migration pattern

```sql
-- File: migrations/YYYYMMDD_description.sql
-- Never modify existing migrations
-- Always additive (no DROP TABLE in production migration)
ALTER TABLE User ADD COLUMN newField TEXT;
```

## OpenAPI-first development

1. Design the endpoint in `scripts/generate-openapi.ts` (Zod schema)
2. Implement the route handler
3. Run `bun run scripts/generate-openapi.ts`
4. Run `bun run build` to verify

Never hand-edit `openapi.json` — it's generated.

## Testing

```sh
# Unit tests
bun test                    # run all tests
bun test tests/unit/foo.test.ts  # run specific

# Type check
bun run tsc --noEmit

# Build check (wrangler dry-run)
bun run build
```
