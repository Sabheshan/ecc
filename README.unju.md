# unju-ai/ecc — Unju Fork of Everything Claude Code

This is the unju-ai curated fork of [affaan-m/everything-claude-code](https://github.com/affaan-m/everything-claude-code).

## What's different from upstream

- **Vetted hooks only** — the hooks.json is stripped to a minimal, audited set. No inline Node.js one-liners we haven't reviewed.
- **Unju-specific skills added** — `cloudflare-workers`, `unju-stack`
- **Unju-specific agents added** — `unju-api-specialist`
- **Unju conventions rule** — bun-first, wrangler.toml correctness, OpenAPI-first

## Pulling upstream changes

```sh
git remote add upstream https://github.com/affaan-m/everything-claude-code.git
git fetch upstream
git diff HEAD upstream/main -- hooks/hooks.json  # review hooks before merging
git merge upstream/main
# Review all changes before pushing, especially hooks/
```

**Always review hooks/** before merging from upstream. Hooks execute code.

## Pinned upstream commit

Base: `b3d3eac532dfb5bcce8689c8300a6e57763d1a43` (v1.7.0)
