# URSULA

A self-contained browser-based reading tracker. No server, no installation, no database required — everything runs in a single HTML file using localStorage for persistence.

---

## Quick Start

1. Download `reading-dashboard.html`
2. Open it in your browser (double-click, or drag into an open browser window)
3. That's it — you're running the app

No build step, no dependencies, no internet connection required.

---

## Requirements

**Browser:** Chrome or Firefox recommended. Safari works but has known quirks with localStorage in private/incognito mode.

**Local vs. server:** The file is designed to be opened directly from your filesystem (`file://` protocol). It also works if served from a local or remote web server, but note that **data is tied to the exact file path and origin** — moving the file or changing how you open it will make previously saved data inaccessible (the data still exists in localStorage, it's just keyed to the old path).

---

## Data Persistence

All data is stored in your browser's localStorage under these keys:

| Key | Contents |
|-----|----------|
| `rl2_entries` | All reading entries |
| `rl2_settings` | User preferences |
| `rl2_archive` | Archived yearly data |
| `rl2_last_export` | Timestamp of last backup export |
| `rl2_backup_dismissed` | Backup reminder state |
| `rl2_first_open` | First-open timestamp |

**This means:**
- Clearing browser data/cache will delete your reading log
- Data does not sync across devices or browsers
- Opening the file in a different browser = empty slate (different localStorage)
- Moving or renaming the file will make previously saved data inaccessible (the data still exists in localStorage under the old path, but the new path won't find it)

**Updating the file:** When a new version of the file is saved to the **same folder with the same filename** and opened in the same browser, all existing data carries over automatically. No re-import needed. This is the intended update workflow — keep the filename (`reading-dashboard.html`) and folder consistent across all versions.

**Backing up:** Use the Export function inside the app to download a JSON file. Keep this somewhere safe (an external drive, cloud storage, etc.). You can restore from it using the Import function. Export before every update as a safety net.

---

## Making Changes

The entire app — HTML, CSS, and JavaScript — lives in a single file (`reading-dashboard.html`). Open it in any text editor to make changes.

**Recommended workflow for development:**

1. Make a versioned copy before editing (e.g. `reading-dashboard-v23.html`) so you have a rollback point
2. Edit the copy in your text editor
3. Open the edited copy in your browser to test
4. When satisfied, replace your working copy

**Do not test with your real data file.** Open a separate copy of the file for development, or use a different browser profile so localStorage data stays separate.

### Build Safety Checks (apply before shipping any version)

From the v23 handoff, these checks must pass before finalising a new version:

1. **ID cross-reference** — every `getElementById('x')` in JavaScript has a matching `id='x'` in HTML
2. **Listener audit** — after removing any HTML element, confirm no orphaned event listeners remain
3. **CSS size check** — the CSS block should be within 10% of the previous version's size unless the change is intentional (a drop from ~28KB to ~9KB was the root cause of the v15 regression)

---

## Importing Data

Two import formats are supported, both accessible from the Import button at the bottom of the page.

**StoryGraph (TSV):** Export your library from StoryGraph, then import the `.tsv` file. The importer handles date normalisation, half-star ratings, HTML stripping from reviews, and excludes did-not-finish entries. A confirmation screen shows before any data is committed.

**Custom CSV:** Expects headers on row 1 or 2, supports tab or comma delimiters, and handles date formats including `August 1, 2025`. A confirmation screen shows before commit.

**JSON backup restore:** Restores a previously exported JSON backup. This replaces all current data.

---

## File Size Reference

As of v22:

| Component | Size |
|-----------|------|
| CSS | ~28,300 characters |
| JavaScript | ~2,280 lines |
| Total file | ~135KB |

A significant unexplained drop in CSS size is a warning sign of a regression (see v15 in the changelog).

---

## Current Version

**v22** — See `reading-dashboard-changelog.docx` for full version history, and `reading-dashboard-handoff.docx` for architecture details, data model, and roadmap.
