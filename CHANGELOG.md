# Reading Log Dashboard — Changelog

*All versions — May 2026*

---

## Project Milestone — GitHub Repository Established

*May 24, 2026*

URSULA is now version controlled and publicly hosted on GitHub at
github.com/nicholastremblay94/URSULA. This marks the formal start
of the commit-based workflow. All future builds will follow the
Build → Test → Confirm → Commit → Push cycle.

Pre-v23 history is documented in this changelog. From v23 onwards,
each commit will correspond to a confirmed stable build.

**Repository structure established:**
- `prototype/` — current single-file HTML dashboard
- `docs/` — handoff, data dictionary, roadmap, migration scoping,
  known bugs & limitations
- `CHANGELOG.md` and `README.md` at root level
- `.gitignore` configured to exclude Mac system files

## v22 — Re-reads, Search Clear

Added re-reads section to Edit Entry modal and Collection detail panel. Search bar now clears and dropdown closes when clicking any result.

## v21 — Re-reads Hidden from Search

Re-reads no longer appear in search results. Only visible in completed log.

## v20 — Auto-expand Search, Back to Search, Quick Re-read Fix

All author groups in search now auto-expand. Added Back to search button in collection detail panel top left. Fixed confirmQuickReread() failing silently due to undefined qrRating variable. Confirmed single bottom action bar.

## v19 — Collection Panel CSS, Back Buttons, History Toggle, Spinner Removal

Added all missing collection panel CSS (cp-stat, cp-child-row, cp-child-title). Back to collection button working from child works and Edit collection. Back to search button in Edit Entry modal for search-originated edits. History pill now toggles back to main view on second click. Removed spinner arrows from progress % input. Re-read button only shows for completed entries in collection panel. Duplicate detection reverted to flag cross-type same-name matches with auto-link on add.

## v18 — Collection Detail Panel, Same-name Nesting, Bottom Bar, Duplicate Detection

Same-name individual works (e.g. essay named same as collection) now visible and nested under parent collection in search. Import/Add buttons added to bottom of page. Duplicate detection changed to flag cross-type matches. When flagged duplicate added, auto-links to parent collection. Stats show singular/plural correctly. Child work rows larger with more padding.

## v17 — Search Overhaul (Clean Rebuild)

Complete search rebuild. Results grouped by author. Priority: Currently Reading first, Completed, TBR. Collections show with child count. Individual works belonging to collection hidden from results. Clicking collection opens Collection Detail Panel. Clicking standalone work opens Edit Entry modal. Fixed missing collectionOverlay HTML element (was causing data to disappear on refresh by preventing load() from running).

## v16 — Stable Base After v15 Regression

Rebuilt from v14 stable base. Carried over confirmed working fixes: import confirmation, topbar layout, Add entry rename, All types dropdown removal, rating dropdown, LGBTQ+ label, Danish/Icelandic, action bar position, Tabler icons, SVG search icon.

## v15 — Search Overhaul Attempt 1 (Regressed)

First search overhaul attempt. Introduced CSS regression (stylesheet reduced from 27KB to 9KB) and duplicate const declarations breaking data persistence. Reverted. Root causes identified and fixed in v17.

## v14 — StoryGraph Fixes, New Types, Era Removal, Category Rename

StoryGraph import: fixed undefined dates, preserved .5 ratings, stripped HTML from reviews, stored reviews in separate field, excluded did-not-finish entries. Memoir and Non-fiction added as book types, both count toward Books goal. Genre renamed to Category, applies to all book types. Classic/Contemporary Era system fully removed. Rating display changed to dropdown (—, 1, 1.5...5). Tabler icon font loaded for trashcan icons. SVG search icon replaces emoji.

## v13 — Import Confirmation, LGBTQ+ Options, Search Redesign Attempt

Import confirmation screen before committing CSV and StoryGraph imports. LGBTQ+ filter options: Author, Content, Author & Content. No date filter in year dropdown. Type pills moved above completed log. History pill stays at top. All types dropdown removed from filter bar. Danish and Icelandic added to language dropdown. Memoir and Non-fiction added to type list.

## v12 — Type Pills, Scroll, Search Grouping

Type pills moved to above completed log. History pill stays top right. Type pills now only filter completed log and TBR — not Goals, Currently Reading, or Demographics. Completed log and TBR fixed height with scrollbar. Search redesigned with author grouping, status sections, and detail panel.

## v11 — Re-reads, Rating History, Progress Live Update, Era Removal

Re-read flow: completed entries get re-read button. Short-form types (Poem, Essay, Short Story) log immediately. Long-form go to Currently Reading. Re-reads don't count toward goals. Rating history stored per entry. Want to re-read flag. Edit pencil on Currently Reading cards. Final review field in mark complete modal. Rating display as '4 ★' format. Era/Classic/Contemporary fully removed. Classic added as genre option.

## v10 — Re-read System Foundation

Initial re-read architecture: re-read button on completed log rows, currently reading linked entries, quick-log modal for short-form types, want-to-reread toggle, re-read filter in log.

## v9 — StoryGraph Improvements, No Date Filter

Smart translator extraction from Contributors column. StoryGraph format mapping (audio detection). No date filter option in year dropdown. Undated entries sort to bottom in normal view.

## Earlier Versions (v1–v8)

Initial builds established: core data model and localStorage persistence, completed log with filtering, currently reading cards with journal, TBR list, goals tracker, StoryGraph CSV import, custom CSV import, duplicate detection, backup/restore, AI TBR recommendations, settings panel, year archiving/history, demographics section, LGBTQ+ tagging, series tracking, format (Text/Audio) tracking, language dropdown, backup reminder system.
