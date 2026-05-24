# Reading Log Dashboard — Handoff

*Current version: v22 | Last updated: May 24, 2026*

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Data Storage](#2-data-storage)
3. [Content Types](#3-content-types)
4. [Data Model — Entry Fields](#4-data-model--entry-fields)
5. [Collection System](#5-collection-system)
6. [Key Architecture Rules](#6-key-architecture-rules)
7. [Import System](#7-import-system)
8. [Search System](#8-search-system)
9. [Collection Detail Panel](#9-collection-detail-panel)
10. [Author Biographical Lookup (Wikipedia)](#10-author-biographical-lookup-wikipedia)
11. [Build Safety Checks](#11-build-safety-checks)
12. [Pending Fixes (as of v22)](#12-pending-fixes-as-of-v22)

---

## 1. Project Overview

A self-contained HTML file reading tracker that runs entirely in the browser using localStorage for data persistence. No server, no installation, no database required. Designed to consolidate reading data previously managed across spreadsheets.

**Current file:** `reading-dashboard.html` — single file, all CSS/JS/HTML inline. CSS ~28,300 chars. ~2,280 lines of JavaScript. Total size ~135KB.

---

## 2. Data Storage

localStorage keys: `rl2_entries`, `rl2_settings`, `rl2_archive`, `rl2_last_export`, `rl2_backup_dismissed`, `rl2_first_open`. JSON export/import for backup and restore. Data is tied to the exact file path and filename in the browser.

---

## 3. Content Types

**Fiction**
- Novel
- Drama
- Short Story Collection

**Non-fiction**
- Memoir
- Non-fiction
- Essay Collection
- Poetry Collection

**Individual works**
- Short Story
- Essay
- Poem

---

## 4. Data Model — Entry Fields

| Field | Notes |
|-------|-------|
| `id` | Unique identifier. Never changes. Used as stable reference for re-read linking. |
| `status` | `completed` / `currentlyReading` / `tbr` |
| `type` | See Content Types above |
| `title` | Work title. For individual works in a collection, this is the individual title. |
| `author` | Author name as entered. Multiple authors comma-separated. |
| `translator` | Extracted from StoryGraph Contributors column on import. |
| `language` | Selected from dropdown (includes Danish, Icelandic). |
| `genre` | Displayed as "Category" in UI. Applies to all types. Classic is a valid genre. |
| `lgbtqAuthor` | boolean — flags author as LGBTQ+ |
| `lgbtqContent` | boolean — flags work as containing LGBTQ+ content |
| `seriesName` | Series name if applicable |
| `seriesNum` | Position within series |
| `format` | `Text` or `Audio` |
| `dateFinished` | ISO date string `YYYY-MM-DD`. Required for goal counting. |
| `startedDate` | ISO date string `YYYY-MM-DD` |
| `progress` | Integer 0–100. Only meaningful for `currentlyReading` entries. |
| `rating` | Supports half-increments (1, 1.5 … 5). 0 or empty = unrated. |
| `ratingHistory` | Array of previous ratings, one per re-read. |
| `notes` | **Collection linking key** — see Section 5. Not free-text for collected works. |
| `review` | Longer-form review. Separate from notes. Imported from StoryGraph Review column. |
| `journal` | Array of `{ text, date }` objects written during Currently Reading. |
| `wantReread` | boolean — marks entry for future re-read |
| `isReread` | boolean — true if this entry is a re-read of another entry |
| `rereadSourceId` | `id` of the original entry this is a re-read of |

---

## 5. Collection System

Collections (Essay Collection, Poetry Collection, Short Story Collection) are parent entries. Individual works link to their parent via the `notes` field — the notes value must exactly match the collection's `title`, and the `author` fields must also match. This is the core grouping mechanism used in search, collection detail panels, and goal counting.

**The linking rule:**
```
Individual work:  { type: "Essay", author: "Joan Didion", notes: "We Tell Ourselves Stories in Order to Live" }
Collection:       { type: "Essay Collection", author: "Joan Didion", title: "We Tell Ourselves Stories in Order to Live" }
```

Both `notes === title` AND `author === author` must match. Exact string comparison — whitespace, capitalisation, or punctuation differences will silently break links.

**Anthology system: NOT YET BUILT.** Next major feature (v23). Will add an `isAnthology` flag enabling notes-only matching (no author match required) for multi-author collections. Test case: *Elbows Up! — Canadian Voices of Resilience and Resistance*.

---

## 6. Key Architecture Rules

- `BOOK_TYPES` array = `['Novel','Short Story Collection','Essay Collection','Poetry Collection','Drama','Memoir','Non-fiction']` — controls Books goal counting
- Re-reads do not count toward annual goals
- Re-reads are hidden from search results, visible only in completed log
- Individual works belonging to a collection are hidden from search, surface via collection detail panel
- Same-name individual work (e.g. essay titled same as its collection) shows in search AND nested under its collection
- Duplicate detection flags same title + same author (any type)
- When a flagged duplicate is added, auto-links to parent collection via notes field

---

## 7. Import System

### StoryGraph Import

TSV format. Handles: date normalisation (`YYYY/MM/DD` → `YYYY-MM-DD`), rating preservation (.5 values), HTML stripping from reviews, reviews stored in separate `review` field (not `notes`), did-not-finish exclusion, translator extraction from Contributors column, format mapping. Confirmation screen before commit.

### CSV Import

Handles blank first rows, headers on row 2+, blank first column, date formats including `August 1, 2025`. Auto-detects tab vs comma delimiter. Confirmation screen before commit.

---

## 8. Search System

Groups results by author. Priority order: Currently Reading first (orange dot), Completed, TBR. All author groups auto-expand. Collections show with child count (e.g. `20 essays`). Individual works belonging to a collection are hidden. Clicking a collection opens Collection Detail Panel. Clicking a standalone work opens Edit Entry modal with Back to search button. Search clears on result click.

---

## 9. Collection Detail Panel

Shows: title, author, type/language/category/status tags, stats (works inside, completed, rating), scrollable child list (each clickable → Edit Entry modal with Back to collection button), Edit collection + Re-read buttons. Re-read only shows for completed collections. Back to search button in top left. Re-reads section if collection has been re-read.

---

## 10. Author Biographical Lookup (Wikipedia)

A two-phase approach to author biographical data.

### Phase 1 — Wikipedia lookup (v24 candidate)

A lightweight, stateless author info card surfaced inside the Edit Entry modal and Collection Detail Panel. No new data model or localStorage keys. The author name already in the entry is used to query the Wikipedia REST API on demand. Nothing is written to storage.

**What the card shows:** biographical extract (3–4 sentences), author photo or initials avatar fallback, link to the full Wikipedia article, graceful "No Wikipedia article found" state.

**Display states by author count:**

| Author count | Display |
|---|---|
| 1 name | Full card: photo/initials, bio extract, Wikipedia link |
| 2 names | Compact dual card: side-by-side, initials, one-line description, link. Bio extract omitted. |
| 3+ names | No fetch: stacked initials avatars only. Covers anthology imports with multiple contributors. |

**Implementation notes:**
- Endpoint: `https://en.wikipedia.org/api/rest_v1/page/summary/{Author_Name}`
- Returns `extract`, `thumbnail.source`, `content_urls.desktop.page`
- No proxy or auth needed — Wikipedia allows browser-side cross-origin requests
- Split author field on comma to detect count before any fetch
- Always implement initials fallback — Wikimedia photo hotlinking can be inconsistent
- A `?` bubble next to the Author field shows on hover: *"If multiple authors, please separate with a comma"* — queued for v23

### Phase 2 — Full author database (post-migration)

A dedicated authors table in Supabase with full author pages, demographic tracking, and click-through navigation from any author name. Post-migration feature — requires relational data not suited to localStorage.

---

## 11. Build Safety Checks

Applied from v23 onwards. Must pass before finalising any version:

1. **ID cross-reference** — every `getElementById('x')` in JS has a matching `id='x'` in HTML
2. **Listener audit** — after any HTML element removal, confirm no orphaned event listeners remain
3. **CSS size check** — CSS block within 10% of previous version size unless intentional

---

## 12. Pending Fixes (as of v22)

These were built in v22 and must be tested before v23 work begins:

- **PF-01** — Re-reads section in Edit Entry modal (built, untested)
- **PF-02** — Re-reads section in Collection Detail Panel (built, untested)
- **PF-03** — Search clears on result click (built, untested)
