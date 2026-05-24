# URSULA — Roadmap

*Last updated: May 24, 2026*

---

## Table of Contents

1. [Current Phase — Prototype Stabilisation](#1-current-phase--prototype-stabilisation)
2. [Near-term Builds](#2-near-term-builds)
3. [Migration to URSULA](#3-migration-to-ursula)
4. [Post-Migration Features](#4-post-migration-features)
5. [Future State (Deferred)](#5-future-state-deferred)

---

## 1. Current Phase — Prototype Stabilisation

The homepage prototype (single HTML file, localStorage) continues to be built until the anthology system is stable and the feature set is ready for migration.

**Immediate next step:** Test the three v22 pending fixes before any new builds proceed.

- PF-01 — Re-reads section in Edit Entry modal
- PF-02 — Re-reads section in Collection Detail Panel
- PF-03 — Search clears on result click

---

## 2. Near-term Builds

Items queued for the prototype phase, roughly in priority order:

| Feature | Version | Notes |
|---------|---------|-------|
| Anthology / edited collection system | v23 | Next major build. Adds `isAnthology` flag, notes-only linking for multi-author collections. Test case: *Elbows Up!* |
| `?` bubble on Author field | v23 | Hover text: "If multiple authors, please separate with a comma" |
| Wikipedia biographical lookup | v24 candidate | Lightweight author info card in Edit Entry modal and Collection Detail Panel. Stateless — no new localStorage keys. |

---

## 3. Migration to URSULA

**Migration is gated on:** anthology system built and stable, plus further updates as needed. Not imminent.

**Planned stack:**

| Layer | Decision |
|-------|----------|
| Backend | Node.js |
| Database | Supabase (PostgreSQL) |
| Hosting | Vercel |
| Frontend | Next.js (likely) |
| Auth | Supabase Auth (single user) |
| PWA | Web manifest + service worker, added post-migration as late-stage feature |

**Data bridge:** JSON export from localStorage → one-time seed script into Supabase.

**Schema notes to resolve before migration:**
- `notes` field used as surrogate collection foreign key → should become explicit `parent_id` / `collection_id`
- `ratingHistory` array → JSONB or separate ratings table
- `journal` array → JSONB or separate table
- Anthology schema should be designed for the database, not retrofitted

See `docs/migration-scoping.md` for full scoping notes.

---

## 4. Post-Migration Features

Features that require the real stack and are not planned for the prototype:

- **Individual type pages** — dedicated views for Poetry, Essays, Short Stories, Novels, Drama
- **Full author database** — author pages with demographics, click-through from any author name, reading history per author
- **Writing goals page** — tracking writing output alongside reading
- **Public reading list** — read-only view hosted on Squarespace domain
- **Demographics filtering** — full demographic tracking across reading history (nationality, language, gender)

---

## 5. Future State (Deferred)

Items noted but not yet designed or scheduled:

- Semi-colon separated `notes` for multi-anthology membership
- Non-fiction subtypes (Biography, History, Criticism)
- Individual work ratings in collection panel
- Author-level goals (e.g. read 10 women writers this year)
- Classic cutoff system (removed in v11/v14 — Classic is now a genre option)
