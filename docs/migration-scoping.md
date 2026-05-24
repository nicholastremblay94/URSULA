# URSULA — Migration Scoping Notes

*Living document — updated as decisions are made | Started May 2026*

---

## Table of Contents

1. [Project Overview & Migration Goals](#1-project-overview--migration-goals)
2. [Stack Decisions](#2-stack-decisions)
3. [Data Migration](#3-data-migration)
4. [User & Access Model](#4-user--access-model)
5. [Feature Parity Checklist](#5-feature-parity-checklist)
6. [Open Questions](#6-open-questions)
7. [Decision Log](#7-decision-log)

---

## 1. Project Overview & Migration Goals

URSULA is the named future state of the Reading Log Dashboard — a self-contained browser tool currently running as a single HTML file with localStorage persistence. The migration moves it to a proper web application stack. URSULA is named after Ursula K. Le Guin.

### Why Migrate

- localStorage is fragile — tied to a specific file path and browser; data can be lost by moving the file
- Single-file architecture makes the codebase hard to maintain as features grow
- Future roadmap items (individual type pages, writing goals, public reading list) require a real backend
- A proper stack makes the PM portfolio story more compelling

### Migration Trigger

Migration is not imminent. The homepage prototype continues to be built until the anthology system is stable and several further updates are complete. This document accumulates scoping decisions in the meantime.

---

## 2. Stack Decisions

| Topic | Status | Notes |
|-------|--------|-------|
| Backend runtime | Node.js | Confirmed |
| Database | Supabase (PostgreSQL) | Confirmed |
| Hosting | Vercel | Confirmed |
| Frontend framework | TBD | Next.js likely — pairs naturally with Vercel; to confirm |
| Auth | TBD — probably Supabase Auth | Single user only; see Section 4 |
| Public reading list | Squarespace domain | Confirmed destination; integration TBD |
| PWA install support | Confirmed May 2026 | Late-stage addition post-migration. Web manifest + service worker gives dock icon and native-feeling window without changing the stack. |

**Notes:**

↳ Next.js + Vercel has near-zero setup for CI/CD — each GitHub commit auto-deploys. Good fit for a solo developer workflow.

↳ Supabase Auth is straightforward for single-user: one account, no multi-tenancy design needed.

↳ PWA vs. full web app is a false choice. URSULA will be both. The Vercel-hosted app is the product; PWA support is what makes it feel native. Sequencing: build core app → confirm feature parity → add manifest + service worker (estimated one afternoon).

---

## 3. Data Migration

The homepage prototype stores all data as JSON (via localStorage), with a JSON export/import feature already built. This is a natural bridge to the migration.

### Current Storage Keys

| Key | Contents |
|-----|----------|
| `rl2_entries` | All reading entries |
| `rl2_settings` | User preferences |
| `rl2_archive` | Year-archived entries |
| `rl2_last_export` | Timestamp of last backup export |
| `rl2_backup_dismissed` | Backup reminder state |
| `rl2_first_open` | First-open timestamp |

### Migration Approach (Preliminary)

- Export full JSON from browser before migration
- Write a one-time import script to seed Supabase tables from the JSON export
- Map `rl2_entries` → entries table; `rl2_archive` → archived_entries or a flag on the same table
- `rl2_settings` → user_settings table (single row)

### Schema Considerations

↳ The `notes` field currently doubles as a collection-linking foreign key (child works store the collection title in notes). In a real database this should become an explicit `parent_id` or `collection_id` column.

↳ `ratingHistory` array on each entry is currently stored as a JSON array. Could stay as JSONB in Postgres or be normalized into a separate ratings table.

↳ `journal` array — same question: JSONB or separate table.

↳ The anthology system (not yet built) will introduce `isAnthology` flag and editor field — these should be designed for the database schema, not retrofitted.

---

## 4. User & Access Model

URSULA is a single-user application — no multi-tenancy required. If others want to use URSULA, they can find it on GitHub and deploy their own copy.

### Primary User

- One account, one dataset, no user isolation needed in the database schema
- Auth is primarily to protect the app from public access, not to separate user data

### Testing Access — Open Question

| Option | Status | Notes |
|--------|--------|-------|
| Shared login | Open | Simplest — share credentials with tester. No isolation; tester can see/modify real data. |
| Staging environment | Preferred direction | Separate Supabase instance with seeded data. Cleaner; more setup. Decision deferred. |
| Read-only public view | On roadmap | Public reading list on Squarespace. Already planned; no app access needed. |
| Second user account | Not planned | Requires multi-user schema design — not in scope. |

---

## 5. Feature Parity Checklist

Features that must exist at migration parity before the homepage prototype is retired:

| Feature | Status | Notes |
|---------|--------|-------|
| Completed log with filtering | Prototype complete | Carry over |
| Currently Reading cards + journal | Prototype complete | Carry over |
| TBR list | Prototype complete | Carry over |
| Goals tracker | Prototype complete | Carry over |
| Demographics section | Prototype complete | Carry over |
| Search system (v17+) | Prototype complete | Carry over |
| Collection detail panel | Prototype complete | Carry over |
| Re-read system | Prototype complete (v22, testing) | Carry over once confirmed |
| StoryGraph import | Prototype complete | Rewrite for server-side |
| CSV import | Prototype complete | Rewrite for server-side |
| JSON backup / restore | Prototype complete | Superseded by Supabase |
| Anthology system | NOT YET BUILT | Must be built + stable before migration |
| Individual type pages | On roadmap | New — build post-migration |
| Writing goals page | On roadmap | New — build post-migration |
| Public reading list | On roadmap | New — Squarespace integration |
| Year archiving | Prototype complete | Carry over |
| Settings panel | Prototype complete | Carry over |
| PWA install support | Post-migration addition | Web manifest + service worker; add after core feature parity confirmed |

---

## 6. Open Questions

| Topic | Status | Notes |
|-------|--------|-------|
| Frontend framework | Open | Next.js assumed but not confirmed |
| Auth provider | Open | Supabase Auth likely; confirm when stack is finalised |
| Schema: notes-as-collection-link | Open | Should become explicit `parent_id` in DB |
| Schema: ratingHistory | Open | JSONB column vs separate table |
| Schema: journal array | Open | JSONB column vs separate table |
| Testing access model | Open | Deferred; staging environment preferred direction |
| Squarespace integration method | Open | Read-only Supabase query? API? TBD |
| Domain / URL for URSULA | Open | Not yet decided |
| Anthology schema | Open | Should be designed before migration |
| CI/CD pipeline | Open | Vercel + GitHub assumed; to confirm |

---

## 7. Decision Log

| Topic | Decision | Notes |
|-------|----------|-------|
| App name | URSULA | Named after Ursula K. Le Guin |
| Scope: single user only | Confirmed May 2026 | No multi-tenancy; one account, one dataset. Others who want to use it can deploy their own copy from GitHub. |
| Migration timing | Post-anthology + further updates | Prototype continues until stable and feature-ready |
| Public list destination | Squarespace domain | Confirmed on roadmap |
| Data bridge strategy | JSON export → seed script | rl2 JSON export format is a natural migration bridge |
| PWA install support | Confirmed May 2026 | Added as late-stage feature on top of Vercel deployment. Web manifest + service worker. Gives dock icon and native-feeling window without changing the stack. |
