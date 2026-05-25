# URSULA — Known Bugs & Limitations

*Current version: v22 | Last updated: May 23, 2026*

---

## Table of Contents

1. [How to Use This Document](#1-how-to-use-this-document)
2. [Pending Fixes (Built, Needs Testing)](#2-pending-fixes-built-needs-testing)
3. [Known Limitations (By Design)](#3-known-limitations-by-design)
4. [Historical Bugs (Resolved)](#4-historical-bugs-resolved)
5. [Regression Risk Areas](#5-regression-risk-areas)

---

## 1. How to Use This Document

**Pending Fixes** are features that have been built but not yet verified in testing. They may work, may be partially broken, or may have introduced regressions — they are not considered stable.

**Known Limitations** are structural constraints of the current architecture. They are not bugs — they are expected behaviours given deliberate design decisions. Most will be resolved in the platform migration to Node.js + Supabase.

**Historical Bugs** are resolved issues kept here for context, particularly where the fix introduced a significant architectural rule or safety check.

**Regression Risk Areas** are parts of the codebase that have caused problems before and warrant extra care when editing nearby code.

---

## 2. Pending Fixes (Built, Needs Testing)

These three features were built in v22. They should be the first things tested before beginning any new v23 work.

---

### PF-01 — Re-reads Section in Edit Entry Modal

**Status:** Built in v22. Untested.

**What it should do:** When opening the Edit Entry modal for a completed entry that has been re-read, a Re-reads section should appear at the bottom showing each re-read as a clickable row (date, rating).

**How to test:**
1. Find an entry in the completed log that has been re-read at least once
2. Click its edit (pencil) button
3. Confirm a Re-reads section appears with the correct re-read date and rating
4. Click a re-read row and confirm it opens that re-read's own Edit Entry modal

**Risk:** If the re-read source linking (`rereadSourceId`) is not correctly populated in the data, rows may appear empty or the section may not render at all.

---

### PF-02 — Re-reads Section in Collection Detail Panel

**Status:** Built in v22. Untested.

**What it should do:** When viewing a collection in the Collection Detail Panel that has been re-read, a Re-reads section should appear showing re-read history.

**How to test:**
1. Find a collection (Essay Collection, Poetry Collection, or Short Story Collection) that has been re-read
2. Click it in search to open the Collection Detail Panel
3. Confirm the Re-reads section appears with correct data

**Risk:** Collections re-read may have different `rereadSourceId` structures than individual works. If the panel reads these incorrectly, the section may not render.

---

### PF-03 — Search Clears on Result Click

**Status:** Built in v22. Untested.

**What it should do:** Clicking any search result (either opening an Edit Entry modal or a Collection Detail Panel) should clear the search input and close the dropdown.

**How to test:**
1. Type a search query
2. Click a result
3. Confirm the search bar is empty and the dropdown is gone after the modal/panel opens
4. Close the modal/panel and confirm the search remains cleared

**Risk:** Low. This is a UI-only change with no data implications.

---

## 3. Known Limitations (By Design)

These are not bugs. They are documented constraints of the current architecture that are either acceptable for now or scheduled to be resolved.

---

### KL-01 — Anthologies Cannot Be Linked (Multi-Author Collections)

**Severity:** High — actively prevents correct data display for multi-author works

**Description:** The collection linking mechanism requires both `notes === collection.title` AND `author === collection.author`. For anthologies edited by one person but containing works by many different authors, the author fields will never match, so individual works cannot be linked to the collection.

**Current behaviour:** Only works by the editor/primary author will link to the collection. All other works appear as unlinked standalone entries.

**Real-world example:** *Elbows Up! — Canadian Voices of Resilience and Resistance* (ed. Elamin Abdelmahmoud). The collection is logged under Elamin Abdelmahmoud, but 25 of 26 essays are by different authors. Only the one essay by Abdelmahmoud links correctly.

**Workaround:** None available currently. Do not add individual works from anthologies until the anthology feature is built — they will not link and will create orphan entries.

**Resolution:** The anthology system (next major feature). Will add an `isAnthology` flag and change the linking logic to notes-only matching (no author match required) when the flag is set.

---

### KL-02 — `notes` Field Cannot Hold Free Text for Collected Works

**Severity:** Medium — data entry confusion risk

**Description:** For any Essay, Poem, or Short Story that belongs to a collection, the `notes` field is fully occupied by the collection link key (the parent collection's title). It cannot simultaneously hold reading notes.

**Workaround:** Use the `review` field for notes on individual works. It is a separate field that displays in the Edit Entry modal but is not used for linking.

**Resolution:** Deferred. A future state design proposes semi-colon-separated values in `notes` to support multi-anthology membership, but this has not been designed in detail.

---

### KL-03 — Renaming a Collection Breaks All Child Links

**Severity:** High — silent data corruption risk

**Description:** Because child works link to a collection via `notes === collection.title`, renaming the collection title does not cascade to child works. All previously linked works will silently become unlinked — they will stop appearing in the Collection Detail Panel and will reappear as standalone search results.

**Workaround:** If a collection must be renamed, manually update the `notes` field on every child work to match the new title. There is no bulk edit feature, so this must be done one entry at a time via the Edit Entry modal.

**Resolution:** Will be resolved when moving to a relational database (platform migration). With a real foreign key, the collection `id` rather than `title` will be the link — renames will be safe.

---

### KL-04 — Data Is Tied to Browser and File Path

**Severity:** High — data loss risk if not understood

**Description:** localStorage data is scoped to the exact file path and browser. Opening `reading-dashboard.html` in Chrome and Firefox gives two completely separate data stores. Moving the file to a different folder, renaming it, or opening it via a web server instead of directly from disk will all result in an apparently empty app (the old data still exists in localStorage under the old path's key, but the new path won't find it).

**The update workflow (important):** When a new version of the file is saved to the **same folder with the same filename** and opened in the same browser, all existing data carries over automatically. No re-import is needed. This is the intended way to receive updates — same filename, same folder, same browser.

**Workaround:** Decide on a permanent home and filename for the file before adding real data, and never change either. Always open it from the same browser. Export a JSON backup regularly — the app has a backup reminder system for this purpose.

**Resolution:** Platform migration to Node.js + Supabase will store data server-side, eliminating this constraint entirely.

---

### KL-05 — Re-reads Do Not Count Toward Goals

**Severity:** Low — this is intentional, but worth documenting

**Description:** Re-read entries (`isReread: true`) are explicitly excluded from all annual goal counts. This is a deliberate design decision. A re-read of a novel does not increment the Books goal.

**Resolution:** None planned. This is the intended behaviour.

---

### KL-06 — No Individual Work Ratings in Collection Panel

**Severity:** Low

**Description:** The Collection Detail Panel shows aggregate stats (number of works, completion count, average rating of the collection itself) but individual child works cannot be rated separately. A poem or essay does not have its own rating field — only the parent collection is rated.

**Resolution:** Deferred to individual type pages (post-platform migration).

---

### KL-07 — StoryGraph Import Excludes Did-Not-Finish Entries

**Severity:** Low — informational

**Description:** The StoryGraph importer explicitly filters out entries marked as did-not-finish. These entries are silently dropped during import with no notification.

**Workaround:** If tracking abandoned books is important, they must be added manually after import.

**Resolution:** None planned for now.

---

## 4. Historical Bugs (Resolved)

These are resolved issues kept on record because their fixes established important architectural rules.

---

### HB-01 — The v15 CSS Regression

**Versions affected:** v15 only (reverted)

**What happened:** The first search overhaul attempt accidentally truncated the CSS block from ~27KB to ~9KB and introduced duplicate `const` declarations in the JavaScript. The duplicate declarations caused `load()` to fail silently on page refresh, which made all data appear to disappear. The data was not actually lost — it was in localStorage — but the app could not read it.

**How it was fixed:** v15 was fully reverted. v16 rebuilt from the v14 stable base. v17 re-implemented the search overhaul as a clean rebuild.

**Rule introduced:** CSS size check — the CSS block must stay within 10% of the previous version's size unless the change is intentional. This is now a required build safety check before shipping any version.

---

### HB-02 — The v17 collectionOverlay Missing Element Bug

**Versions affected:** v17 (fixed in v17)

**What happened:** The `collectionOverlay` HTML element was missing from the document. The `load()` function attempted to reference it on startup. When the element didn't exist, `load()` threw an error and stopped executing, which caused data to appear to disappear on every page refresh.

**How it was fixed:** The missing HTML element was added back.

**Rule introduced:** ID cross-reference check — every `getElementById('x')` in JavaScript must have a matching `id='x'` in HTML. This is now a required build safety check before shipping any version.

---

### HB-03 — confirmQuickReread() Failing Silently

**Versions affected:** v19 (fixed in v20)

**What happened:** The `confirmQuickReread()` function referenced a variable `qrRating` that was undefined in scope. The function failed silently — no error shown to the user, no re-read logged.

**How it was fixed:** The undefined variable was corrected in v20.

**Rule introduced:** Reinforced the importance of testing re-read flows specifically, as they involve multiple interacting functions that can fail silently.

---

### HB-04 — StoryGraph Import Mangling Author Field

**Versions affected:** Pre-v14

**What happened:** The StoryGraph Contributors column contains entries like "Translated by Margaret Jull Costa" or "Foreword by Patti Smith." These were being imported directly into the `author` field, resulting in author names like "Joan Didion, Introduction by Mary Gordon."

**How it was fixed:** v9 added smart translator extraction. v14 extended this to strip all role-labelled contributors (Foreword by, Afterword by, Introduction by, Preface by, Illustrator) from the author field entirely.

---

## 5. Regression Risk Areas

These are parts of the codebase that have caused problems before or are structurally fragile. Treat any changes in these areas with extra caution and run the full build safety checklist afterward.

| Area | Risk | Notes |
|------|------|-------|
| CSS block | Accidental truncation | The v15 regression was caused here. Always check size after edits. |
| `load()` function | Silent failure on missing elements | Any new HTML element referenced in `load()` must exist in the DOM. |
| Search result click handlers | Silent orphaned listeners | The v15 regression also broke click handlers. After any search UI change, verify edit modal and delete buttons still work in the completed log. |
| `const` / `let` declarations | Duplicate declaration errors | Duplicate `const` declarations in the same scope will cause silent failures in `load()`. Scan for duplicates after any large JS addition. |
| Collection linking logic | Author-match sensitivity | The `notes === title AND author === author` matching is exact string comparison. Whitespace, capitalisation, or punctuation differences will silently break links. |
| Re-read flows | Multi-function chain failures | Re-reads involve several interacting functions. A failure in any one of them (as in HB-03) will silently drop the re-read with no user feedback. |
