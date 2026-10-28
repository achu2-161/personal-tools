# Anila's Doors

**A PhD milestone advent calendar for Anila. She adds her milestones and fills a surprise jar; each new milestone secretly draws a surprise, revealed when its door opens.**

| | |
|---|---|
| Site | `https://achu2-161.github.io/anila-doors/` |
| Repo | `achu2-161/anila-doors` (**private**; Pages works through GitHub Pro) |
| Data tier | Device only: localStorage |
| Users and devices | Anila, on her own phone (Android or iPhone) |
| Status | Deployed. Built 28 Oct 2026 |

## Files

| File | Purpose |
|---|---|
| `index.html` | Markup |
| `app.js` | All logic |
| `app.css` | All styles |
| `sw.js` | Offline service worker |
| `manifest.webmanifest` | Android install |
| `icons/` | Custom door icon set |
| `README.md` | Deploy and install steps |
| `.nojekyll` | Tells Pages to serve files as-is |

This is the first app with JS and CSS split out. The split is canonical; a single-file preview copy is generated from it when needed (see [TESTING.md](../TESTING.md#preview-copies-inside-claude)).

## Data model

One localStorage key, `anila-doors`:

```js
{ app: "anila-doors", version: 1, milestones: [], jar: [], settings, lastBackup }
```

- A milestone's surprise is stored **sealed**: base64, reversed. It isn't encryption, just an envelope so a glance at storage or a backup file doesn't spoil anything.
- Creating a milestone draws a random jar item and removes it from the jar. Deleting a milestone returns its surprise to the jar.
- Door state is derived from the unlock time and an `opened` flag.
- The app calls `navigator.storage.persist()` to ask the browser not to evict its data.

## Features

Live countdown to the next milestone; numbered doors that flip open with confetti once their date passes; monthly calendar for adding milestones; long-press to edit; "PhD road" progress bar between start and submission; surprise jar; JSON backup and restore (share sheet on iOS) with a backup reminder.

## Look

Yellow and purple (theme colour `#2A1250`), a playful tone with encouraging and teasing messages ("No peeking!").

## Decisions

The app went through five versions as the brief changed. Worth keeping as a worked example of choosing a data tier:
1. **Claude artifact** with artifact storage: first prototype.
2. **Supabase**, client page plus a master page with login, so a partner could set prizes Anila couldn't see. Prizes released by the server only after unlock time.
3. **Firestore** instead, since Adrian already uses Firebase. Time-gated rules (see [FIREBASE.md](../FIREBASE.md#time-gated-reads-keeping-something-secret-until-a-date)).
4. **Passcode** in place of logins, accepting weaker security for a low-stakes app; master and client pages merged.
5. **Device only** (final): Anila manages everything herself, and secrecy comes from the random draw rather than from a server.

## App-specific lessons

- Native `confirm()` blocked delete in the preview. Replaced with a two-tap confirm.
- A decorative sun covered the seconds digit on phones.
- Claude can push to the repo but couldn't create it or switch on Pages.
