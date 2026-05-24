# Reading Log Dashboard

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

**Backing up:** Use the Export function inside the app to download a JSON file. Keep this somewhere safe. You can restore from it using the Import function.

---

## Making Changes

The entire app — HTML, CSS, and JavaScript — lives in a single file (`reading-dashboard.html`). Open it in any text editor to make changes.

### The Build → Test → Confirm → Commit Cycle

From v23 onwards, versions follow a formalised release cycle:

1. **Build** — a new version is produced (in Claude) as a release candidate
2. **Test** — the specific features that changed are verified in the browser
3. **Confirm** — once testing passes, the version is considered stable
4. **Commit** — the confirmed stable file is committed to GitHub with a descriptive message

A version is not "shipped" until it has passed testing. This is why v22 has three features listed as pending testing — they were built but not yet confirmed before this cycle was formalised at v23.

**Note:** Pre-v23 builds used a more informal process (build → rename → maybe test later). The changelog reflects this — earlier entries describe what was built; from v23 onwards, entries describe what was confirmed stable.

### Recommended File Setup

Keep two copies open during development:

- `prototype/reading-dashboard.html` — your live, real-data file. Never edit this directly.
- `prototype/reading-dashboard-dev.html` — your working copy for testing. Use a separate browser profile so localStorage stays isolated from your real data.

When a build is confirmed stable, replace the live file with the dev copy and commit.

### Build Safety Checks (required before any commit)

These checks must pass before finalising a new version:

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

## Repository Structure

```
ursula/
├── README.md
├── CHANGELOG.md
├── prototype/
│   └── reading-dashboard.html
└── docs/
    ├── handoff.md
    ├── data-dictionary.md
    ├── known-bugs-and-limitations.md
    └── ROADMAP.md
```

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

## Version History

Formal versioned releases begin at **v23**. Earlier versions (v1–v22) were built iteratively in a single-file prototype — see `CHANGELOG.md` for the full history.

The project will eventually migrate to a proper stack (Node.js + Supabase + Vercel) under the name **URSULA** (named after Ursula K. Le Guin). See `docs/ROADMAP.md` for migration plans.

---

## Current Version

**v22** — See `CHANGELOG.md` for full version history and `docs/handoff.md` for architecture details and roadmap.
