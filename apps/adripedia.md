# Adripedia

**A shared word and expression repository for Adrian and Yixiao: two encyclopedias (Adripedia, Yixiaopedia) plus an All view, with favourites, sorting, study mode and screenshots.**

| | |
|---|---|
| Site | `https://achu2-161.github.io/adripedia/` |
| Repo | `achu2-161/adripedia` |
| Data tier | Firestore |
| Firebase project | `adripedia-app`, `australia-southeast2` (Melbourne) |
| Users and devices | Adrian and Yixiao; desktop and phones |
| Status | Deployed, stable |

## Files

| File | Purpose |
|---|---|
| `index.html` | Everything, including Firebase wiring |
| `adripedia-importer.html` | Standalone CSV importer: validates, previews, checks clashes, writes to Firestore |
| `manifest.json` | Android install |
| `sw.js` | Network-first, cache fallback |
| `icon-192.png`, `icon-512.png` | Android icons |
| `apple-touch-icon.png` | iOS icon |

## Data model

Collection `entries`, document ID `term + '_' + owner`.

Fields: `term`, `def`, `owner` (`adrian` | `yixiao`), `date` (`YYYY-MM-DD`, displayed), `createdAt` (ISO, sort key, never displayed), `link`, `image` (compressed base64), `favorite`, `updatedAt`.

- `createdAt` exists because day-precision dates made same-day entries tie. Older entries fall back to `updatedAt`, then `date`.
- Images are base64 in the document: no Firebase Storage. `storageBucket` in the config is unused (and possibly wrong).
- Rules: `allow read, write: if true` on `entries`. `owner` is a label, not access control.

## How it works

- **The DOM is the source of truth.** Cards live in `#cardsList`; search, study mode and backup read from `.card` elements. This is the first thing to change if entries ever need querying without rendering.
- `filterCards()` hides with `display: none` and reorders with `appendChild`. **Never `innerHTML = ''`** on the list.
- `onSnapshot` drives `updateLocalCard()` and `removeLocalCard()`, matched on term + owner.
- `setFavorite(card, value)` is the **only** path for favourite changes.
- Renames: capture old term and owner, save the new document, delete the old only on success.
- Owner picker appears in the form only in the All view, defaulting to Adripedia.
- Study mode reads the DOM at start. Flashcards use the whole deck; the quiz takes 10 at random (`QUIZ_LENGTH`) and needs at least 4 entries (`MIN_QUIZ_ENTRIES`). Score is derived from stored answers.

## Features

Two encyclopedias plus All; add, edit, delete; favourites and a favourites filter; sort popover (name or date, independent direction); search; flashcards and quiz; backup and restore JSON; compressed screenshots; CSV importer handling Australian dates, Windows-1252 encoding and non-breaking spaces; PWA on Android and iOS.

## Look

Wikipedia-like: Georgia serif body, sans-serif UI chrome, blue `#2563eb`, white cards on grey `#e8eaed`.

## App-specific lessons

Most of Adripedia's bugs were general and are in [LESSONS.md](../LESSONS.md#firebase). Specific to this app:
- Restoring a backup writes every entry to Firestore and Yixiao's phone. Use the stubbed test build for test data ([TESTING.md](../TESTING.md#testing-a-firebase-app-without-touching-real-data)).
- The encyclopedia names are hardcoded in about six places (menu, `switchEncyclopedia`, logo letter, title, owner badge, form picker).

## Discussed, not built

- **Tags or categories**, mainly so quiz distractors come from the same group.
- Usage examples, etymology, related entries, CSV/PDF export.
- **Spaced repetition**: review progress is per person so can't sit on the shared document; `term_owner` IDs would orphan history on rename; DOM-based study can't answer "what's due today".
- **A config object for the encyclopedia names**, like Brownie Points' `identities`. This would make the app reusable for other subjects.
- Persisting quiz stats (kept in-session on purpose).
- SharePoint / Microsoft 365 as a backend for work apps (see [FIREBASE.md](../FIREBASE.md#alternatives-considered)).
