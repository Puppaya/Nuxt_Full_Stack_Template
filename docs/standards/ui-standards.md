# UI/UX Standards

> Single source of truth for UI work in `app/**/*.{vue,css}` — for every developer and every AI tool.

## Before coding UI

Answer first, then build:

| Question | Example |
|----------|---------|
| Who is the user? | Admin, Editor |
| What is the goal? | Approve customer records |
| Primary action? | Create / Save / Approve |
| Key information? | Name, status, date |
| What can go wrong? | Empty list, API error, unauthorized |

If unclear, ask one concise question — do not guess with a decorative layout.

Then read the closest existing pattern and copy its structure:

1. `app/pages/users.vue`, `app/pages/customers.vue` (CRUD lists)
2. `app/components/users/UserFormModal.vue`, `app/components/customers/CustomerFormModal.vue`
3. `app/components/ConfirmModal.vue`
4. `app/layouts/default.vue`
5. `app/app.config.ts` (theme)

Do not invent a new layout system.

## Reuse (required)

- Nuxt UI: `UButton`, `UTable`, `UModal`, `UForm`, `UFormField`, `UInput`, `USelect`, `UBadge`, `UPagination`
- Shared components: `ConfirmModal`, `AppLoading`, existing `*FormModal.vue`
- Theme tokens from `app/app.config.ts` — primary: indigo, neutral: slate; no new palette
- Icons: lucide via `i-lucide-*`
- Fonts: Outfit + Public Sans; Noto Sans Lao for Lao text
- Do not add UI libraries or one-off styled components without team approval

## Forbidden

- Decorative gradients, glassmorphism, heavy shadows, hero sections in admin pages
- KPI/chart cards without a clear business decision
- Multiple competing primary buttons on one screen
- Card nesting without information purpose
- Vague copy: "Welcome", "Manage efficiently"; buttons labeled "Submit" / "Action" / "Click here"
- Icon-only critical actions without an accessible label
- Arbitrary spacing — use the 4/8/12/16/24/32 scale
- Custom CSS that duplicates Nuxt UI

## Required states

Every screen handles all four:

| State | Implementation |
|-------|----------------|
| Loading | `AppLoading` or table skeleton |
| Empty | Message + next action (e.g. "Create first customer") |
| Error | User-friendly message + retry if applicable |
| Success | Toast notification |

Destructive actions use `ConfirmModal` with clear consequence text.

## Language

UI supports Lao. Follow the existing copy style in `ConfirmModal.vue` and existing toast messages.

## CRUD list page structure

```
Page header (title = module name + primary action)
Toolbar (search, filters)
Table (priority columns only)
Pagination
Modal/Drawer for create-edit
```

## Forms & data

See [coding-standards.md](./coding-standards.md#frontend) — `UFormField name` matches the Zod key, `$fetch` body is `{ ...state }`, `??` for numeric IDs, `USelect` uses `placeholder`.

## Pre-merge checklist

- [ ] User knows what to do without explanation
- [ ] One clear primary action
- [ ] Consistent with existing pages
- [ ] Loading / empty / error / success handled
- [ ] Mobile/tablet usable
- [ ] No decorative elements that do not support the workflow
- [ ] Slop pass done: unused cards, extra primary buttons, marketing headings, unneeded fields removed
