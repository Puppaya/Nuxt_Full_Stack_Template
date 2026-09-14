---
name: coding-standards
description: Project coding standards for Nuxt full-stack development. Use when writing or reviewing API routes, services, repositories, Vue pages, composables, forms, auth, validation, or unit tests in this repository.
---

# Coding Standards

## Stack

Nuxt 4 · Vue 3 · Nuxt UI · Zod · Vitest · nuxt-auth-utils

## Architecture

```
Client (app/)
  └─ pages / components / composables
       └─ $fetch / useApi / useApiAction
            └─ server/api/          ← HTTP + auth + validation
                 └─ server/services/ ← business logic
                      └─ repositories + BaseRepository
                           └─ Database
```

Do not mandate a specific ORM or database — follow whatever the project already uses. All data access stays in the repository layer.

Reference files:
- API: `server/api/users/index.post.ts`, `server/api/users/[id].patch.ts`
- Service: `server/services/user.service.ts`
- Repository: `server/utils/base-repository.ts`, `server/utils/repositories.ts`
- Validation: `server/utils/validation.ts`
- Response: `server/utils/api-response.ts`
- RBAC: `server/utils/rbac.ts`
- Frontend: `app/pages/users.vue`, `app/components/users/UserFormModal.vue`
- Tests: `tests/unit/services/user.service.spec.ts`

---

## Adding a new module (checklist)

### 1. Validation (`server/utils/validation.ts`)

```typescript
export const FooSchema = z.object({
  name: z.string().min(2),
  companyId: z.coerce.number().min(1)  // coerce for string-encoded numbers
})
export const FooUpdateSchema = FooSchema.partial()
```

### 2. Repository

Extend `BaseRepository` in `server/utils/repositories.ts` — wire up the project's existing data access:

```typescript
class FooRepository extends BaseRepository<Foo> {
  constructor() { super(/* project's model/table accessor */) }
  // add findByXxx only if needed
}
export const fooRepository = new FooRepository()
```

### 3. Service (`server/services/foo.service.ts`)

```typescript
export class FooService {
  async getFoos(params: { page: number; pageSize: number; search?: string }) { /* ... */ }

  async createFoo(data: CreateFooInput) {
    return fooRepository.create(data)
  }

  async updateFoo(id: number, data: UpdateFooInput) {
    const existing = await fooRepository.findById(id)
    if (!existing || existing.deletedAt) {
      throw createError({ statusCode: 404, statusMessage: 'Not found' })
    }
    return fooRepository.update(id, data)
  }

  async deleteFoo(id: number, ctx?: AuditContext) {
    const existing = await fooRepository.findById(id)
    if (!existing || existing.deletedAt) {
      throw createError({ statusCode: 404, statusMessage: 'Not found' })
    }
    return fooRepository.update(id, {
      deletedAt: new Date(),
      updatedBy: resolveAuditActor(null, ctx)
    })
  }
}
export const fooService = new FooService()
```

### 4. API routes (`server/api/foos/`)

| File | Method | Auth |
|------|--------|------|
| `index.get.ts` | GET list | per requirement |
| `index.post.ts` | POST create | `requireAdmin` or `checkRole` |
| `[id].patch.ts` | PATCH update | same |
| `[id].delete.ts` | DELETE | same |

Handler template:

```typescript
export default defineEventHandler(async (event) => {
  await requireAdmin(event)

  const id = Number(getRouterParam(event, 'id'))
  if (!id || isNaN(id)) return sendApiError('Invalid ID', 400)

  const data = await validateRequest(event, FooUpdateSchema)
  const result = await fooService.updateFoo(id, data)
  return sendSuccess(result, 'Updated successfully')
})
```

### 5. Frontend

- List page: copy structure from `app/pages/users.vue`
- Form modal: copy from `app/components/users/UserFormModal.vue`
- Use `useApiAction` for create/update/delete with Lao toast messages
- Destructive actions: `ConfirmModal`

### 6. Tests (`tests/unit/services/foo.service.spec.ts`)

Required setup for 404 tests:

```typescript
import { createError } from 'h3'

beforeAll(() => {
  vi.stubGlobal('createError', createError)
})
```

Required cases for `update`/`delete`:

```typescript
it('should throw 404 when record does not exist', async () => {
  vi.mocked(fooRepository.findById).mockResolvedValue(null)
  await expect(fooService.updateFoo(999, data)).rejects.toMatchObject({ statusCode: 404 })
  expect(fooRepository.update).not.toHaveBeenCalled()
})

it('should throw 404 when record is already soft-deleted', async () => {
  vi.mocked(fooRepository.findById).mockResolvedValue({ id: 1, deletedAt: new Date() } as any)
  await expect(fooService.updateFoo(1, data)).rejects.toMatchObject({ statusCode: 404 })
  expect(fooRepository.update).not.toHaveBeenCalled()
})
```

Delete must assert audit field:

```typescript
expect(fooRepository.update).toHaveBeenCalledWith(
  id,
  expect.objectContaining({ deletedAt: expect.any(Date), updatedBy: expect.any(String) })
)
```

Auth test for soft-deleted users:

```typescript
it('should return null if user is soft-deleted', async () => {
  vi.mocked(userRepository.findByUsername).mockResolvedValue({ ...user, deletedAt: new Date() } as any)
  await expect(userService.authenticate('u', 'p')).resolves.toBeNull()
})
```

---

## API response format

```typescript
// Success
{ success: true, message?: string, data?: T, meta?: { total, page, pageSize } }

// Error (via sendApiError)
{ success: false, message: string, errors?: fieldErrors }
```

Never throw raw errors to client. Use `sendApiError(message, statusCode, errors?)`.

---

## Auth & RBAC

**Server:** `requireAdmin(event)` or `checkRole(event, ['ADMIN', 'EDITOR'])`

**Client:** `definePageMeta({ roles: ['ADMIN'] })` — enforced in `app/middleware/auth.global.ts`

**Login:** `authenticate()` must return `null` if user has `deletedAt`.

---

## Frontend patterns

### List + CRUD page

```typescript
const page = ref(1)
const pageSize = ref(10)
const search = ref('')

const { data, pending, refresh } = useApi(() =>
  `/api/foos?page=${page.value}&pageSize=${pageSize.value}&search=${search.value}`
)

const { execute, loading } = useApiAction()
```

### Form submit

```typescript
const body = { ...state }
if (props.record && !body.password) delete (body as any).password
else if (body.password) body.password = encryptPassword(body.password)

await execute(() => $fetch(url, { method, body }), { successMessage: '...' })
```

### Common mistakes

| Wrong | Correct |
|-------|---------|
| `body: state` | `body: { ...state }` |
| `companyId: props.id \|\| undefined` | `companyId: props.id ?? undefined` |
| `UFormField name="role"` but schema has `status` | names must match |
| `items: [{ label: 'All', value: '' }]` | use `placeholder` on USelect |
| Business logic in `.vue` | move to service or composable |

---

## Security

- Passwords: AES encrypt client → decrypt API → bcrypt hash service
- Never log or commit `SECRET_KEY`, `NUXT_SESSION_PASSWORD`, DB credentials
- Validate all inputs with Zod before service layer
- Check role on both API and page level

---

## Code review checklist

- [ ] Follows API → Service → Repository layers
- [ ] Zod validation on all write endpoints
- [ ] Route ID validated for NaN
- [ ] `sendSuccess` / `sendApiError` used consistently
- [ ] Frontend uses `useApi` / `useApiAction`
- [ ] Forms match Zod schema field names
- [ ] Tests added/updated for changed business logic
- [ ] No secrets in diff
