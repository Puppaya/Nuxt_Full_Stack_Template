---
name: anti-ai-slop-ui
description: Create or review UI/UX without AI slop. Use when building pages, components, forms, tables, dashboards, modals, or when user mentions AI slop, generic UI, or UX review in this Nuxt UI project.
---

# Anti AI Slop UI/UX

## When to use

- Creating or modifying pages/components in `app/`
- Reviewing UI for generic AI patterns
- User asks to fix slop, improve UX, or match existing design

## Workflow

### 1. Understand the task (required)

Answer before writing code:

| Question | Example |
|----------|---------|
| Who is the user? | Admin, Editor |
| What is the goal? | Approve customer records |
| Primary action? | Create / Save / Approve |
| Key information? | Name, status, date |
| What can go wrong? | Empty list, API error, unauthorized |

If unclear, ask one concise question — do not guess with a decorative layout.

### 2. Find existing patterns

Inspect in order:

1. Closest page: `app/pages/users.vue` or `app/pages/customers.vue` for CRUD lists
2. Form modals: `app/components/users/UserFormModal.vue`, `app/components/customers/CustomerFormModal.vue`
3. Confirmations: `app/components/ConfirmModal.vue`
4. Layout shell: `app/layouts/default.vue`
5. Theme: `app/app.config.ts`

**Rule:** Copy structure and component choices from these files. Do not invent a new layout system.

### 3. Implement

#### CRUD list page pattern

```text
definePageMeta + layout
computed columns for UTable
search / filter / pagination refs
$fetch or useApi with loading + error handling
primary action button in header toolbar
modal for create/edit, ConfirmModal for delete
```

#### Form pattern

```text
UForm + Zod schema
UFormField name matches schema key exactly
UInput / USelect / UTextarea from Nuxt UI
submit → toast on success, inline errors on validation fail
body: { ...state } for $fetch
```

#### States (all required)

| State | Implementation |
|-------|----------------|
| Loading | `AppLoading` or table skeleton |
| Empty | Message + suggested next action (e.g. "Create first customer") |
| Error | User-friendly message + retry if applicable |
| Success | Toast notification |

### 4. Slop removal pass

After implementation, remove:

- Unused cards, charts, or KPI blocks
- Extra primary buttons
- Marketing-style headings
- Custom CSS that duplicates Nuxt UI
- Fields not required for the task

### 5. Final review

Run checklist from `.cursor/rules/anti-ai-slop-ui.mdc`.

## Project conventions

- **Stack:** Nuxt 4 + Nuxt UI + Tailwind v4
- **Colors:** indigo (primary), slate (neutral) — do not introduce new palette
- **Icons:** lucide via `i-lucide-*`
- **Language:** UI supports Lao — use existing copy style in `ConfirmModal.vue`
- **Enterprise focus:** density, scannability, fast repetitive tasks — not consumer marketing UI

## Anti-patterns vs fixes

| Slop | Fix |
|------|-----|
| Dashboard with 6 KPI cards | Show only metrics that drive a decision |
| "Welcome to your dashboard" | Page title = module name ("Customers") |
| Full-width form with 12 fields | Group related fields; hide advanced fields |
| Custom modal markup | Reuse `UModal` or `ConfirmModal` |
| Chart for 3 values | Use table or inline stat |
| Delete without confirmation | `ConfirmModal` with consequence text |

## Output

When reviewing UI, report:

1. Slop items found (if any)
2. What was removed or changed
3. Which existing file was used as reference
