# Brownie Points

**A shared, live-synced points tracker between Adrian and Yixiao. Either can log points with a reason and spend credits; changes appear on both phones within a second.**

| | |
|---|---|
| Site | `https://achu2-161.github.io/brownie-points/` |
| Repo | `achu2-161/brownie-points` (public) |
| Data tier | Firebase Realtime Database |
| Firebase project | `brownie-points-tracker`, `asia-southeast1` |
| Users and devices | Adrian (Android), Yixiao (iPhone) |
| Status | Deployed. Core mechanic under reconsideration |

## Files

| File | Purpose |
|---|---|
| `index.html` | Everything, including the Firebase config |
| `manifest.json` | Android install |
| `sw.js` | Caches the app shell; never caches Firebase or font requests |
| `icon-192.png`, `icon-512.png`, `icon-512-maskable.png` | Android icons |
| `apple-touch-icon.png` | iOS icon (plus iOS meta tags in `index.html`) |

The icon is a chocolate square with a bite out, drawn with Pillow.

## Data model

Single node `brownieTracker`:
- `plus`, `minus`: cumulative points earned and lost. Only ever increase. ("Lost", not "Spent", which is reserved for credits.)
- `creditsSpent`: cumulative credits redeemed
- `log`: list via `push()`, each `{ type, amount, description, who, ts }`
- `lastExport`: `{ content, ts }`, the latest `.txt` export, so either phone can re-download it

**Derived:** net = `plus − minus`; credits earned = `floor(max(net, 0) / 50)`; credits available = earned − `creditsSpent`.

Counters use `transaction()`. Reset clears everything except `lastExport`.

## How it works

- `dataLayer` interface: `init`, `applyAction`, `reset`, `setExport`. Three build variants used it (chat preview, localStorage phone-test, Firebase deployed).
- Identity: `?u=1` (Adrian) / `?u=2` (Yixiao), remembered in localStorage, "Who's this?" fallback, "Not you?" link. Names live in the `identities` object near the top of the script.
- Rules open on `brownieTracker` only, closed everywhere else.
- Live/Offline pill from `.info/connected`.

## Features

Net score; progress tray towards the next credit; quick +1/−1 (logged and attributed); custom amounts with a required reason; credit spending with balance check; history behind the hamburger menu (totals, log, export `.txt`, reset with confirmation and automatic export first).

## Look

Dark chocolate card (`#2B1810`) on warm cream (`#F6EDE1`), caramel and gold accents, burnt orange for lost points. Fraunces, Manrope, IBM Plex Mono. Fits one phone screen without scrolling.

## Decisions

- **No real login.** Firebase Auth (email, Google, magic link) and a client-side password gate were discussed. Not built: attribution was the need, not security.
- **Don't port the retry/debounce scaffolding** from the chat-preview build into the Firebase version. Firebase doesn't need it.

## Open questions

The earn/spend loop became ambiguous once both people could earn and spend. Options on the table, none chosen:
1. **Keep the current system.**
2. **Two independent personal ledgers**: earning credits your own bank (for something nice you did for your partner); spending debits your own bank. Proposed as keeping the vocabulary clear. A two-bank prototype (`banks: { "1": {...}, "2": {...} }`) was built and scrapped before deployment.
3. **Zero-sum pool**: 1000 points split 500/500, transfers only. Concerns raised: changed incentives, no endgame, "spend" vs "give" confusion, no redemption, concurrent-write races.
