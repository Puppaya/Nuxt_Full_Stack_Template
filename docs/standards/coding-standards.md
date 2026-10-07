# Coding Standards

> Single source of truth for **every developer and every AI tool**.
> Tool-specific files (`CLAUDE.md`, `GEMINI.md`, `.cursor/`, `.github/copilot-instructions.md`) only point here. Edit this file, not the pointers.

## Stack

Nuxt 4 · Vue 3 · Nuxt UI (Tailwind v4) · Prisma + SQL Server · Zod · Vitest · nuxt-auth-utils · Bun

## Architecture

Layered flow only: **API → Service → Repository → Database**

| Layer | Location | Responsibility |
|-------|----------|----------------|
| API | `server/api/` | Auth, validation, HTTP, response format |
| Service | `server/services/` | Business logic, hashing, rules |
| Repository | `server/utils/repositories.ts` + `BaseRepository` | Data access |
| Frontend | `app/pages/`, `app/components/` | UI, forms, API calls |

- No business logic in API handlers beyond orchestration
- No direct DB/ORM calls outside repositories
- Export service as class + singleton: `export const userService = new UserService()`

Reference files:
- API: `server/api/users/index.post.ts`, `server/api/users/[id].patch.ts`
- Service: `server/services/user.service.ts`
- Repository: `server/utils/base-repository.ts`, `server/utils/repositories.ts`
- Validation: `server/utils/validation.ts`
- Response: `server/utils/api-response.ts`
- RBAC: `server/utils/rbac.ts`
- Frontend: `app/pages/users.vue`, `app/components/users/UserFormModal.vue`
- Tests: `tests/unit/services/user.service.spec.ts`

## Formatting

| Area | Indent |
|------|--------|
| `server/**` | 4 spaces |
| everything else (`app/`, `tests/`, config, JSON, md) | 2 spaces |

LF line endings, UTF-8, final newline. Enforced by `.editorconfig`.

## Adding a new module (checklist)

1. **Validation** — add `FooSchema` / `FooUpdateSchema` to `server/utils/validation.ts` (`z.coerce.number().min(1)` for numeric IDs/inputs)
2. **Repository** — extend `BaseRepository` in `server/utils/repositories.ts`; export `fooRepository`
3. **Service** — `server/services/foo.service.ts`, class + singleton
4. **API routes** — `server/api/foos/` (`index.get.ts`, `index.post.ts`, `[id].patch.ts`, `[id].delete.ts`)
5. **Frontend** — copy `users.vue` + `UserFormModal.vue`; `useApi` / `useApiAction`; `ConfirmModal` for delete
6. **Tests** — `tests/unit/services/foo.service.spec.ts`

## API

```typescript
export default defineEventHandler(async (event) => {
    await requireAdmin(event)                       // or checkRole(event, ['ADMIN', 'EDITOR'])

    const id = Number(getRouterParam(event, 'id'))
    if (!id || isNaN(id)) return sendApiError('Invalid ID', 400)

    const data = await validateRequest(event, FooUpdateSchema)
    const result = await fooService.updateFoo(id, data)
    return sendSuccess(result, 'Updated successfully')
})
```

- Always respond with `sendSuccess` / `sendApiError`; never throw raw errors to the client
- Password from client: `decryptPassword()` in API before calling service
- Response shape:
  - Success: `{ success: true, message?, data?, meta?: { total, page, pageSize } }`
  - Error: `{ success: false, message, errors? }`

## Service

- One service class per domain (`UserService`, `CustomerService`)
- `update` / `delete`: `findById` first → throw 404 if missing or soft-deleted
- Soft-delete: include `updatedBy: resolveAuditActor(null, ctx)` in the update payload
- `authenticate()`: check `deletedAt` before password; return `null` for soft-deleted users

## Frontend

- Data fetching: `useApi` (lists) or `useApiAction` (mutations)
- Forms: `UForm` + Zod; `UFormField name` must match schema key
- `$fetch` body: `{ ...state }` — never pass reactive proxy directly
- Numeric IDs: use `??` not `||`
- `USelect`: use `placeholder`, never `value: ''` in items
- Passwords: `encryptPassword()` before send; omit empty password on edit
- Page RBAC: `definePageMeta({ roles: [...] })` (enforced in `app/middleware/auth.global.ts`)
- No business logic in `.vue` files — move to service or composable

| Wrong | Correct |
|-------|---------|
| `body: state` | `body: { ...state }` |
| `companyId: props.id \|\| undefined` | `companyId: props.id ?? undefined` |
| `UFormField name="role"` but schema has `status` | names must match |
| `items: [{ label: 'All', value: '' }]` | use `placeholder` on `USelect` |

## Security

- Passwords: AES encrypt on client → decrypt in API → bcrypt hash in service
- Never log or commit `SECRET_KEY`, `NUXT_SESSION_PASSWORD`, DB credentials, or `.env`
- Validate all input with Zod before the service layer
- Check role on both API and page level

## Testing

- Location: `tests/unit/` mirroring source structure
- Mock repositories/crypto — no real DB or HTTP
- Naming: `should [behavior] when [condition]`
- Service specs with 404 paths: stub global `createError` from `h3` in `beforeAll`:

```typescript
import { createError } from 'h3'
beforeAll(() => { vi.stubGlobal('createError', createError) })
```

- Every `update()` / `delete()`: test 404 for missing **and** soft-deleted records, and assert `repository.update` was not called
- Every soft-delete `delete()`: assert `updatedBy` in the update call:

```typescript
expect(fooRepository.update).toHaveBeenCalledWith(
    id,
    expect.objectContaining({ deletedAt: expect.any(Date), updatedBy: expect.any(String) })
)
```

## General

- TypeScript strict; avoid `any` unless matching an existing pattern
- Reuse existing utils before creating new ones
- Add/update tests when business logic changes
- Do not leave scratch files (e.g. `tmp_*.ts`) in the repo

## Definition of done

Run before finishing any task:

```bash
bun run lint
bun run typecheck
bun run test
```

- [ ] Follows API → Service → Repository layers
- [ ] Zod validation on all write endpoints
- [ ] Route ID validated for NaN
- [ ] `sendSuccess` / `sendApiError` used consistently
- [ ] Frontend uses `useApi` / `useApiAction`
- [ ] Form field names match Zod schema keys
- [ ] Tests added/updated for changed business logic
- [ ] No secrets in the diff
