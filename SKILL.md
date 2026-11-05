---
name: pata
description: Apply Francis's PATA framework (Parameters → Assets → Transactions → Analytics) to app development. Use when starting a new app or project, designing a database or schema, adding a table, field, dropdown, status, category or new feature, building an admin panel, reviewing code for hard-coded values, or whenever the user mentions PATA. The core rule is that nothing configurable is hard-coded.
---

# PATA Framework

PATA is the user's standard way of structuring every app. Every table, field and piece of business logic belongs to exactly one of four layers:

| Layer | What lives here | Examples |
|---|---|---|
| **P — Parameters** | Choices and configuration the business controls without a deploy | categories, statuses, roles, currencies, formats, country/city codes |
| **A — Assets** | The core "things" the system manages | users, products, listings, rooms, profiles |
| **T — Transactions** | Records of something happening between users and Assets | bookings, orders, payments, messages, reviews |
| **A — Analytics** | Insights derived from the other three | revenue by category, booking trends, dashboards |

**The golden rule: nothing configurable is hard-coded.** No enums, no `switch` on magic strings, no hard-coded dropdown arrays, no free-text fields pretending to be categories. If it shapes choices, filtering, categorisation or display, it is a Parameter.

The full framework (with the reasoning behind it) is in `references/pata-framework.md`. Read it the first time this skill runs in a session.

## The user

The user is a business analyst and professional no-code (Bubble.io) developer, not a programmer. Explain decisions in plain business language and keep jargon out of summaries. Code and file contents can be technical; the explanation to the user should not be.

## Classifying anything new

Before writing any schema or UI, ask in order and stop at the first "yes":

1. Is this a choice the business wants to control without a deploy? → **Parameter**
2. Is this a "thing" the system manages? → **Asset**
3. Is this a record of something happening? → **Transaction**
4. Is this a derived insight from the above? → **Analytics**

Red flag: if a field "feels like" free text or a fixed list of options, it is almost always a Parameter in disguise.

## Mode 1 — New project kickoff

When the user starts a new app or asks to set up a project with PATA:

1. **Understand the app.** Read any brief, PRD or existing code. If the purpose is unclear, ask a few short questions about what the app does and who uses it.
2. **Draft the PATA Map.** List every proposed table under P / A / T / A, with one line on what each holds and which Parameters each Asset or Transaction references. Show this to the user and get a yes before building — it is the blueprint.
3. **Create `docs/PATA.md`** in the project: the PATA Map, kept up to date as the single source of truth. Every later change updates it.
4. **Build Parameters first**, then Assets, then Transactions. Analytics last.
5. **Seed every Parameter table** with sensible defaults so the app works out of the box.
6. **Build the admin Parameters screen(s)** so an admin can add, rename, reorder and deactivate options without touching code.
7. **Add a project `CLAUDE.md` line** (create the file if missing): `This project follows the PATA framework. See docs/PATA.md. Never hard-code configurable values.`

## Mode 2 — Adding a feature or field

1. Classify every new table and field with the four questions above.
2. For any new Parameter: create the table with the standard pattern, seed it, add admin CRUD, and reference it by foreign key.
3. Update `docs/PATA.md`.
4. Tell the user in one or two plain sentences which layer each new piece went into and why.

## Mode 3 — Audit an existing project

When asked to check, review or "PATA-fy" an existing codebase:

1. Scan for violations:
   - database enums or check constraints listing allowed values
   - `switch` / `if` chains comparing against string literals like `"yoga"`, `"pending"`, `"admin"`
   - hard-coded arrays feeding dropdowns, filters, tabs or badges
   - text columns holding category-like values (look for inconsistent casing or spacing in data)
   - status, type or role values duplicated across frontend and backend
2. Report findings as a plain-language table: *what's hard-coded → where → which Parameter table it should become → risk level*.
3. Propose a fix order (highest business impact first). Do not refactor until the user approves, because migrations touch live data.
4. When migrating a text column to a Parameter: create the table, seed it from the distinct cleaned-up existing values, add the FK column, backfill, then remove the old column only after the user confirms.

## Standard Parameter table pattern

Every Parameter table follows the same shape (adapt syntax to the project's database/ORM — Postgres, Supabase, Drizzle, Prisma, etc.):

```sql
CREATE TABLE "modalities" (
  "id"          uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  "slug"        varchar(80) NOT NULL UNIQUE,
  "label"       text NOT NULL,
  "description" text,
  "sort_order"  integer NOT NULL DEFAULT 0,
  "active"      boolean NOT NULL DEFAULT true,
  "created_at"  timestamp DEFAULT now() NOT NULL,
  "updated_at"  timestamp DEFAULT now() NOT NULL
);
```

Rules:
- Admin CRUD routes: `GET/POST /api/admin/parameters/<name>`, `PATCH/DELETE /api/admin/parameters/<name>/:id`, protected so only admins can use them.
- Assets and Transactions reference Parameters by FK (`uuid` or `slug`) — never copy the raw label string.
- UI reads options from the Parameter table, ordered by `sort_order`, showing only `active = true`.
- Prefer **deactivate** over delete, so old records that point to an option don't break.
- If the app is multi-tenant, add a tenant/organisation column so each tenant can have its own options.

## When code must know a specific value

Sometimes logic genuinely depends on one option (e.g. "when a booking becomes `confirmed`, send the receipt"). In that case:
- Reference the **slug**, never the label (labels are for admins to rename freely).
- Keep all such slugs in **one** constants file per project, each with a comment saying why code depends on it.
- Mark those rows as protected in the admin screen (e.g. an `is_system` boolean) so an admin can rename the label but cannot delete or change the slug.

Everything not on that short list stays fully data-driven.

## Bubble.io projects

If the user is working in Bubble rather than code: Option Sets are edited in the editor and need a deploy, so treat them as hard-coded. Configurable choices that admins should control belong in a **Data Type** with the same fields (slug, label, sort order, active), plus an admin page to manage them. Option Sets are fine only for values that truly never change without a developer.

## Before finishing any task

Check:
- [ ] No new enums, magic strings or hard-coded option lists
- [ ] Every new table classified and recorded in `docs/PATA.md`
- [ ] New Parameters are seeded and manageable from the admin screen
- [ ] Assets/Transactions reference Parameters by FK, not raw text
- [ ] Summary to the user is in plain language: what went into which layer and why
