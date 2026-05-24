# Reading Log Dashboard — Data Dictionary

*Current version: v22 | Last updated: May 23, 2026*

---

## Table of Contents

1. [Storage Structure](#1-storage-structure)
2. [Entry Fields](#2-entry-fields)
   - 2.1 [Core Identity Fields](#21-core-identity-fields)
   - 2.2 [Bibliographic Fields](#22-bibliographic-fields)
   - 2.3 [Classification Fields](#23-classification-fields)
   - 2.4 [Reading Status Fields](#24-reading-status-fields)
   - 2.5 [Rating Fields](#25-rating-fields)
   - 2.6 [Re-read Fields](#26-re-read-fields)
   - 2.7 [Notes & Review Fields](#27-notes--review-fields)
   - 2.8 [Journal Fields](#28-journal-fields)
3. [The Collection Linking Mechanism](#3-the-collection-linking-mechanism)
4. [Content Types Reference](#4-content-types-reference)
5. [Status Values Reference](#5-status-values-reference)
6. [Settings Object](#6-settings-object)

---

## 1. Storage Structure

All data lives in the browser's localStorage under six keys. The `rl2_` prefix namespaces the app to avoid conflicts with other tools stored in the same browser.

| localStorage Key | Type | Contents |
|-----------------|------|----------|
| `rl2_entries` | JSON array | All reading entries — completed, currently reading, and TBR |
| `rl2_settings` | JSON object | User preferences and annual goals |
| `rl2_archive` | JSON array | Past-year entries moved out of the main view via year archiving |
| `rl2_last_export` | string | ISO timestamp of the last JSON backup export |
| `rl2_backup_dismissed` | boolean | Whether the backup reminder banner has been dismissed |
| `rl2_first_open` | string | ISO timestamp of first launch, used to schedule backup reminders |

**Important:** Every entry — novels, poems, essays, collections — is stored as a flat JSON object in the same `rl2_entries` array. There are no separate tables or sub-arrays by type. Relationships between entries (e.g. an essay belonging to a collection) are resolved at read time through field-value matching, not through stored references.

---

## 2. Entry Fields

Every entry shares the same field schema. Fields that don't apply to a given type are left empty or null — they are never omitted from the object entirely.

---

### 2.1 Core Identity Fields

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Unique identifier for the entry. Generated on creation. Never changes. Used as the stable reference for re-read linking. |
| `status` | enum | The entry's current reading status. See [Status Values Reference](#5-status-values-reference). |
| `type` | enum | The content type. See [Content Types Reference](#4-content-types-reference). |

---

### 2.2 Bibliographic Fields

| Field | Type | Description |
|-------|------|-------------|
| `title` | string | The title of the work. For individual works belonging to a collection, this is the title of the individual work (e.g. the essay title), not the collection. |
| `author` | string | The author's name as entered. For collections, this is the primary author. For anthologies (not yet built), this will be the editor's name. |
| `translator` | string | Translator name, if applicable. Extracted from the Contributors column during StoryGraph import. |
| `language` | string | The language the work was read in. Selected from a dropdown that includes Danish and Icelandic among others. |
| `seriesName` | string | Name of the series this work belongs to, if any. |
| `seriesNum` | string or number | Position within the series. |

---

### 2.3 Classification Fields

| Field | Type | Description |
|-------|------|-------------|
| `genre` | string | Displayed as "Category" in the UI. Applies to all book types including non-fiction. "Classic" is a valid genre option (the era/classic system was removed in v11/v14 — Classic is now treated as a genre rather than a structural distinction). |
| `lgbtqAuthor` | boolean | Flags the author as LGBTQ+. Used for demographic tracking and filtering. |
| `lgbtqContent` | boolean | Flags the work as containing LGBTQ+ content or themes. |
| `format` | enum | `Text` or `Audio`. Tracks whether the work was read or listened to. |

---

### 2.4 Reading Status Fields

| Field | Type | Description |
|-------|------|-------------|
| `dateFinished` | string | ISO date string (`YYYY-MM-DD`). Populated when an entry is marked complete. Used for annual goal counting and year filtering. |
| `startedDate` | string | ISO date string (`YYYY-MM-DD`). The date reading began. |
| `progress` | number | Integer 0–100 representing percentage complete. Only meaningful for `currentlyReading` entries. The spinner arrows are hidden from the input field in the UI. |
| `journal` | array | An array of journal entry objects associated with a `currentlyReading` entry. Each object contains the entry text and a timestamp. Carried forward if the entry is later completed. |

---

### 2.5 Rating Fields

| Field | Type | Description |
|-------|------|-------------|
| `rating` | number | The current rating. Supports half-increment values (1, 1.5, 2, 2.5 … 5). Displayed as a dropdown in the UI (`—`, `1`, `1.5` … `5`). A value of `0` or empty means unrated. |
| `ratingHistory` | array | An array of previous ratings, one entry per re-read. Each element stores the rating given at the time. The current `rating` field always reflects the most recent rating; `ratingHistory` holds the historical record. |

---

### 2.6 Re-read Fields

| Field | Type | Description |
|-------|------|-------------|
| `isReread` | boolean | `true` if this entry represents a re-read of another entry. Re-read entries are hidden from search results and do not count toward annual goals. |
| `rereadSourceId` | string | The `id` of the original entry this is a re-read of. Used to link re-read entries back to the source for display in the Edit Entry modal and Collection Detail Panel. Only populated when `isReread` is `true`. |
| `wantReread` | boolean | A flag the user can set to mark an entry as something they want to re-read in future. Used for filtering. |

**Re-read behaviour by type:**
- Short-form types (Poem, Essay, Short Story): re-reads log immediately without going through Currently Reading
- Long-form types (Novel, Drama, etc.): re-reads go to Currently Reading first, then complete normally

---

### 2.7 Notes & Review Fields

| Field | Type | Description |
|-------|------|-------------|
| `notes` | string | **This field is the collection linking key.** See [Section 3](#3-the-collection-linking-mechanism) for full explanation. For standalone entries, this can hold free-form reading notes. For individual works (Essay, Poem, Short Story) that belong to a collection, this field must exactly match the parent collection's `title`. |
| `review` | string | A longer-form review field. Stored separately from `notes`. Imported from the StoryGraph `Review` column (with HTML stripped). Not shown in the log table view. |

---

### 2.8 Journal Fields

| Field | Type | Description |
|-------|------|-------------|
| `journal` | array | Array of timestamped journal entries written while a book is in Currently Reading. Each element is an object with `{ text: string, date: string }`. Persists after the entry is moved to completed. |

---

## 3. The Collection Linking Mechanism

Because localStorage has no relational capabilities, the app simulates parent-child relationships between entries using field-value matching at read time.

**The rule:** An individual work (Essay, Poem, or Short Story) belongs to a parent collection when:
1. Its `notes` field equals the collection's `title`, **AND**
2. Its `author` field equals the collection's `author`

```
Individual work:  { type: "Essay", author: "Joan Didion", notes: "We Tell Ourselves Stories in Order to Live" }
Collection:       { type: "Essay Collection", author: "Joan Didion", title: "We Tell Ourselves Stories in Order to Live" }
```

Both conditions must match. This means the `notes` field on individual works is not a free-text field — it is a surrogate foreign key. Writing anything other than the exact collection title into `notes` on an essay will break the link.

**Consequences of this design:**

- The `notes` field cannot be used for actual notes on any individual work that belongs to a collection
- Renaming a collection title requires manually updating the `notes` field on every child work
- Multi-author collections (anthologies) cannot be linked this way, because the author fields differ — this is a documented limitation addressed by the upcoming anthology system (see known limitations)

**Same-name special case:** If an individual work has the same title as its parent collection (e.g. an essay titled the same as the collection), it appears in search results both as a standalone result *and* nested under the collection in the collection detail panel.

**Duplicate detection:** When a new entry is added with the same title and author as an existing entry of a different type, the system flags it as a potential duplicate and auto-populates the `notes` field with the matching collection's title, creating the link automatically.

---

## 4. Content Types Reference

| Type | Category | Counts toward Books goal | Can belong to a collection |
|------|----------|--------------------------|---------------------------|
| Novel | Fiction | Yes | No |
| Drama | Fiction | Yes | No |
| Short Story Collection | Fiction | Yes | No (is a collection) |
| Memoir | Non-fiction | Yes | No |
| Non-fiction | Non-fiction | Yes | No |
| Essay Collection | Non-fiction | Yes | No (is a collection) |
| Poetry Collection | Non-fiction | Yes | No (is a collection) |
| Short Story | Individual work | No | Yes → Short Story Collection |
| Essay | Individual work | No | Yes → Essay Collection |
| Poem | Individual work | No | Yes → Poetry Collection |

The `BOOK_TYPES` array in the JavaScript (`['Novel', 'Short Story Collection', 'Essay Collection', 'Poetry Collection', 'Drama', 'Memoir', 'Non-fiction']`) controls what counts toward the annual Books goal. Re-reads never count toward goals regardless of type.

---

## 5. Status Values Reference

| Value | Display name | Description |
|-------|-------------|-------------|
| `completed` | Completed | Entry appears in the completed log. Must have a `dateFinished` to count toward annual goals. |
| `currentlyReading` | Currently Reading | Entry appears as a card in the Currently Reading section. Has access to progress tracking and journal. |
| `tbr` | TBR | To Be Read. Appears in the TBR list. No dates required. |

---

## 6. Settings Object

Stored under `rl2_settings`. Contains user preferences and annual reading goals. The exact shape may evolve — this reflects the state as of v22.

| Key | Type | Description |
|-----|------|-------------|
| `goalBooks` | number | Annual target for books read (types in BOOK_TYPES) |
| `goalPoems` | number | Annual target for poems read |
| `goalEssays` | number | Annual target for essays read |
| `goalShortStories` | number | Annual target for short stories read |
| `currentYear` | number | The year currently being tracked in the main view |
