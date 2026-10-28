# Architecture

The structural patterns shared across the apps. Visual design is per app and not covered here.

## Principles

1. **Static files only.** No framework, no npm build, no server code. Anything GitHub Pages can serve as-is.
2. **Every network dependency is a deliberate choice.** Google Fonts and the Firebase SDK are the accepted ones. Small libraries get bundled locally instead of loaded from a CDN (the quiz copies `qrcode.js` into the repo). Flag any new dependency to Adrian before adding it.
3. **Data access goes through one interface.** UI code never talks to storage directly.
4. **Derive, don't accumulate.** Scores, totals and balances are computed from stored records, not kept as running counters that can drift. The exception is a server-side counter updated atomically (Brownie Points' `transaction()`).
5. **Nothing fails silently.** Any error from a write has to reach the user, not just the console.

## File layout

Three layouts are in use. Pick by size and number of screens.

| Layout | Used by | When |
|---|---|---|
| **Single file**: everything in `index.html` | Field Trip, Brownie Points, Adripedia | One screen, moderate size. Simplest to hand over and edit. Field Trip reached ~300 KB this way, which is about the limit of comfortable. |
| **Multi-file**: `index.html` + `app.js` + `app.css` | Anila's Doors | Larger code, or when a preview copy will be generated. Keep this as the canonical version and generate a single-file copy when needed. |
| **Multi-page + shared script**: several `.html` pages plus `shared.js` | RSA Quiz | Several roles or screens that share logic (host, buzzer, MC). |

Separate a file out when someone other than Claude will edit it, such as `questions.js` (quiz content) and `firebase-config.js` (pasted config).

Standard supporting files: `manifest.json` (or `manifest.webmanifest`), `sw.js`, icons, `README.md`. Add `.nojekyll` if any file or folder name starts with an underscore.

## The store interface

Every app with data keeps it behind a small object, so the backend can be swapped without touching UI code. Brownie Points' version:

```js
const dataLayer = {
  init(onChange),          // load, then call onChange(state) on every update
  applyAction(action),     // one atomic write: counter bump + log entry together
  reset(),
  setExport(content),
};
```

The quiz's `Store` goes further: it chooses its backend at start-up.

```js
const useFirebase = !!window.FIREBASE_CONFIG;
// Firebase Realtime Database if configured,
// otherwise localStorage + the `storage` event to sync tabs on one computer.
```

This **dual-mode store** is the pattern to default to for any synced app. The app works end to end in local mode before Firebase exists, and rehearsals don't touch real data.

Rule of thumb: one user action = one write. Combine the counter bump and the log entry into a single call to avoid races and half-applied actions.

## Choosing a data tier

| Need | Tier | Notes |
|---|---|---|
| One person, one device, private data | **Device only**: `localStorage` for small text, IndexedDB for blobs like photos | Must have JSON backup/restore. Data is lost if site data is cleared. |
| Two or more people or devices, live sync, simple shape (counters, a log, a game state) | **Firebase Realtime Database** | One JSON tree. `transaction()` for counters, `push()` for lists. `.info/connected` for a Live/Offline pill. |
| Collection of records that get added, edited, sorted and queried | **Firestore** | One document per record. `onSnapshot` for live updates. 1 MiB per document. |
| One user must not see something yet | **Server-enforced rules** (Firestore rules with `request.time`) | Anything sent to the browser can be found. A client-side lock only deters. |

Claude's own artifact storage (`window.storage`) is **not** a backend. It fails intermittently under rapid writes. Use it only for design-review previews.

## Identity without logins

The apps use **identification, not authentication**:
- A link parameter says who you are (`?u=1`, `?team=3`).
- It's remembered on the device in `localStorage`.
- A "Who's this?" picker is the fallback, and a "Not you?" link clears it.
- Names live in one config object (`identities = { 1: 'Adrian', 2: 'Yixiao' }`), not scattered through the code.

Database rules are open on the app's own path and closed everywhere else. This is fine when two trusted people share non-sensitive data. When real secrecy is needed, see the time-gated rules in [FIREBASE.md](FIREBASE.md#time-gated-reads-keeping-something-secret-until-a-date).

A passcode checked in the browser is **not** security, only a deterrent. Say so when offering it.

## Backup and restore

Every device-only app needs it, and synced apps benefit from it.
- Export as a downloaded JSON file. Embedding photos as base64 is fine in a file, even though it isn't in `localStorage`.
- Every record needs a **stable ID** so a re-import overwrites instead of duplicating. Use a content hash (SHA-256) for blobs like photos.
- Run duplicate clean-up on every import, as a safety net for old backups.
- On iOS, use the share sheet for export.
- Device-only apps should call `navigator.storage.persist()` so the browser is less likely to evict data, and can nudge the user when the last backup is old.
- Warn when a restore will write to shared data. For example, restoring Adripedia pushes every entry to Firestore and onto Yixiao's phone.

## Offline and service workers

Three strategies are in use, and the choice should be deliberate:

| Strategy | Used by | Use when |
|---|---|---|
| Pass-through (registers, caches nothing) | Field Trip | Only to satisfy install criteria |
| App shell cached, data requests always live | Brownie Points | Synced apps. Never cache Firebase or font requests. |
| Network-first, cache fallback | Adripedia | Content changes often and stale pages would confuse |

Default for new apps: **cache the app shell, never cache data requests**, and bump a cache version string on each deploy.

## UI mechanics that recur

These are behaviours rather than styling:
- **No native `confirm()`, `alert()` or `prompt()`.** They're blocked silently in some embedded contexts. Use an in-app modal, or a two-tap confirm ("Tap again to delete").
- **Modals as bottom sheets** on phones, with a tall variant for long content.
- **View/edit toggle** for content cards. Show content in view mode by default, and use a pencil icon to edit. Permanently visible forms look unfinished.
- **Secondary features behind a hamburger menu** so the main screen fits without scrolling.
- **Fill-the-space layouts** use container queries (`container-type: size` + `cqw`/`cqh`) with one custom property driving every dimension. See the quiz buzzer.
- Use `100dvh` rather than `100vh` for full-height phone layouts.
