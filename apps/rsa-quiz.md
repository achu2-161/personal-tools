# RSA Quiz

**A Prime Time-style quiz for an MSD Research Students Association bonding event: a projected board, a phone buzzer per team, and an MC answer screen.**

| | |
|---|---|
| Site | _not recorded_ |
| Repo | _not recorded_ |
| Data tier | Firebase Realtime Database, or local test mode when unconfigured |
| Firebase project | to be created (suggested `rsa-quiz`, `asia-southeast1`) |
| Users and devices | Host laptop on a projector, up to 5 team phones, MC phone |
| Status | Built and refined (Oct 2026). Firebase setup and deployment not confirmed |

## Files

| File | Purpose |
|---|---|
| `index.html` | Host board, run on the projector laptop |
| `buzzer.html` | Team buzzer, opened via `?team=1`…`?team=5` QR codes |
| `mc.html` | MC screen: answer, alternatives and a fun fact as soon as a tile opens; Correct/Wrong/Open buzzers buttons |
| `bank.html` | Printable list of all questions; played ones greyed out |
| `questions.js` | Question bank (`window.CATEGORIES`). Edit to change questions |
| `firebase-config.js` | `window.FIREBASE_CONFIG`; `null` = local test mode |
| `shared.js` | Store and game logic used by all screens |
| `qrcode.js` | `qrcode-generator`, bundled locally (no CDN), SVG output |
| `logo-mark.png`, `logo-card.png` | RSA logo, background removed |

## How it works

- `Store` in `shared.js` picks Firebase if `FIREBASE_CONFIG` is set, otherwise `localStorage` with `storage`-event sync across tabs on one computer. This is the model for the dual-mode store in [ARCHITECTURE.md](../ARCHITECTURE.md#the-store-interface).
- Question IDs are a hash of the question text, so editing a question changes its ID.
- Each new game draws unplayed questions first. 91 questions (3 per tile across six categories) give about three games before repeats. "Forget which questions were used" resets the cycle.
- The board re-renders only when its data signature changes, not on every score update. Re-rendering on every update was eating clicks.
- Rules: open on `quiz`, closed everywhere else.

## Features

Six categories × five levels (100–500; "Reviewer 2" at 200–1,000); a hidden Daily Double with a wager; up to five teams with rename from the phone; buzzers open automatically when a tile is revealed (toggle for manual Space); buzzer test mode with arrival order and timing gaps; keyboard controls (Space, C, X, Esc, F, 1–5 to award a buzz by hand if a phone fails); printable QR cards; optional penalty for wrong answers; credit line "Created by Adrian Chu for RSA 2026".

## Look

Black and white from the RSA logo, with a team colour flooding the screen on a buzz. Reviewer 2 is red, the Daily Double gold. Buzzer sized with container query units so it fills any window.

## Decisions

- **Built in chat, not Claude Code.** The project was small enough.
- **Trust-based.** Answers in `questions.js` are readable by anyone who looks, and `mc.html` has no password. Don't put the repo link on the slides.
- **Close buzzes are decided by arrival order at Firebase**, so a slow phone can lose by a fraction of a second.

## Open questions

- Repo name and whether it's deployed.
- Whether to add a tiebreaker round.
