# Field Trip Tracker

**A phone-installed app tracking Yixiao's 2026 China fieldwork: Guanba → Xinyi → Fushou → Laohegou, 31 Jul–24 Sep 2026.**

| | |
|---|---|
| Site | `https://achu2-161.github.io/fieldwork-tracking/` |
| Repo | `achu2-161/fieldwork-tracking` |
| Data tier | Device only: localStorage + IndexedDB |
| Users and devices | Adrian, on his Samsung Galaxy S22 Ultra |
| Status | Deployed and installed as a real WebAPK. The trip has finished; the summary screen should now show |

## Files

| File | Purpose |
|---|---|
| `index.html` | Everything: markup, CSS, JS (~300 KB) |
| `manifest.json` | References real icon files, never data URIs |
| `sw.js` | Pass-through service worker, only for install criteria |
| `icon-192.png`, `icon-512.png` | Android icons |

No iOS assets. It only ever needed to install on Android.

## Data model

**localStorage** (keys still say "holiday" on purpose, see Decisions):
- `holidayTrackerData`: title, departure and return dates, `stages[]` of `{ city, start, end, image, description }`
- `holidayTrackerJournal`: object keyed by `YYYY-MM-DD`, each `{ happened, places, people, food }`
- `holidayTrackerInfo`: `[{ id, type, title, line1, line2 }]`, type is `contact | transport | accommodation | other`

**IndexedDB** `holidayTrackerPhotoDB`, store `journalPhotos`: `{ id, date, blob, addedAt, hash }`. The SHA-256 `hash` drives duplicate detection on import. Don't remove it.

## Features

Donut progress ring; dual clock (Melbourne and China, DST-aware via `Intl.DateTimeFormat`); current-stage hero card with next-stage countdown; drawer with Itinerary, Journal (calendar with text and photo indicators, search) and Info (typed cards, filter, drag to reorder); four-field journal entries with multiple photos and a swipeable lightbox; JSON backup/restore including photos; automatic duplicate clean-up on import; PDF export via a hidden print view and `window.print()`; edge-swipe gestures; trip summary screen.

## Look

Dark jade (`#0E3E34` cards on `#05201B`) with a muted gold accent (named `--orange` in code for historical reasons) and light tan used sparingly. Space Grotesk, DM Sans, DM Mono. Boarding-pass motif on a textured paper background baked in as base64.

## Decisions

- **Storage keys keep the old "holiday" names.** The visible name changed to "Field Trip Tracker"; renaming the keys would orphan saved data without a migration.
- **Photos in IndexedDB as Blobs**, not base64 in localStorage, which doesn't scale.
- **The map was built with Leaflet and then removed** at Adrian's request. If it comes up again, check it's wanted, and consider a self-contained illustrated map with no CDN.
- **The "clean up duplicates" button** went through add, remove, re-add. It's now a single button. Check before changing it.

## App-specific lessons

- The install saga: see [DEPLOY.md](../DEPLOY.md#checking-an-install-really-worked-android). `chrome://webapks` was what found the base64-icon root cause.
- The data-loss incident: see [LESSONS.md](../LESSONS.md#data-loss-and-shared-data).
- Edge-swipe gestures were never tested on the device and may clash with Android's back gesture.

## Discussed, not built

- **Firebase sync** (Firestore + Storage + Google sign-in), discussed again 1 Nov 2026 after the data loss. Recommendation: Blaze plan, Google sign-in restricted to Adrian, Firestore for text, Cloud Storage for photos named by SHA-256, IndexedDB kept as an offline cache, export/import kept as a safety net, built in Claude Code. Alternatives: Spark with compressed photos in Firestore, a backup-only copy, or saving exports to Drive by hand. Current data (~65 MB backup) is far below every free allowance; see [FIREBASE.md](../FIREBASE.md#plans-limits-and-scaling-up). Open questions: card on file for Blaze, offline behaviour, bucket region, mainland China access. Full notes: project note `claude/handover-firebase-storage-blaze.md`.
- **Multi-trip support**, browsing several trips at once. Every storage key would need a trip-ID scope. Adrian wants the full version eventually.
