# AGENTS.md — Project Standards (all AI tools & developers)

This repository is a team template. **Every AI assistant (Antigravity, Claude Code, Cursor, Copilot, Codex, Windsurf, etc.) and every developer must follow the same standards.**

## Read before writing any code

1. [docs/standards/coding-standards.md](docs/standards/coding-standards.md) — always
2. [docs/standards/ui-standards.md](docs/standards/ui-standards.md) — when touching `app/**/*.{vue,css}`

These files are the single source of truth. Do not duplicate or override them in tool-specific config; edit them instead.

## Non-negotiables (summary)

- Layered flow only: **API → Service → Repository → Database**; no DB calls outside repositories
- API: validate with Zod (`validateRequest`), authorize with `requireAdmin`/`checkRole`, respond with `sendSuccess`/`sendApiError`
- Services: class + singleton; `update`/`delete` check existence and soft-delete → 404
- Frontend: `useApi`/`useApiAction`, Nuxt UI components only, `ConfirmModal` for destructive actions, loading/empty/error/success states
- No decorative UI (gradients, glassmorphism, hero sections, vague KPI cards) in admin pages
- Tests in `tests/unit/` mirroring source; update when business logic changes
- Indent: 4 spaces in `server/**`, 2 spaces elsewhere (see `.editorconfig`)
- Never commit secrets; reuse existing utils; avoid `any`

## Before finishing a task

```bash
bun run lint && bun run typecheck && bun run test
```

If a request conflicts with these standards, say so and ask rather than silently deviating.
