# Anila's Doors

**A PhD milestone advent calendar for Anila. She adds her milestones and fills a surprise jar; each new milestone secretly draws a surprise, revealed when its door opens. Daily goals work the same way on a smaller scale, with their own jar of small treats.**

| | |
|---|---|
| Site | `https://achu2-161.github.io/anila-doors/` |
| Repo | `achu2-161/anila-doors` (**private**; Pages works through GitHub Pro) |
| Data tier | Device only: localStorage |
| Users and devices | Anila, on her own phone (Android or iPhone) |
| Status | Deployed. Built 28 Oct 2026; check-in step added 30 Oct; daily goals added 31 Oct (`sw.js` cache `doors-v3`) |

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
{ app: "anila-doors", version: 2, milestones: [], jar: [], treats: [], goals: [], settings, lastBackup }
```

- `jar` holds milestone treats (big), `treats` holds daily treats (small). `goals` items: `{ id, title, date: "YYYY-MM-DD", done, doneAt, secret }`.
- Version 1 data (no `treats` or `goals`) loads fine: missing fields default to empty. Restore accepts both v1 and v2 backups.
- A milestone's surprise (and a goal's treat) is stored **sealed**: base64, reversed. It isn't encryption, just an envelope so a glance at storage or a backup file doesn't spoil anything.
- Creating a milestone draws a random jar item and removes it from the jar. Deleting an unopened milestone returns its surprise to the jar. Goals do the same with the `treats` jar.
- If a jar is empty at creation time, the draw happens at reveal time instead, with a fallback text if it's still empty.
- A milestone moved to a later date (via the check-in or editing) keeps its sealed surprise; `postponed` counts how many times it was moved.
- Door state is derived from the unlock time and an `opened` flag.
- The app calls `navigator.storage.persist()` to ask the browser not to evict its data.

## Features

Live countdown to the next milestone; numbered doors that flip open with confetti once their date passes; monthly calendar for adding milestones; long-press to edit; "PhD road" progress bar between start and submission; surprise jar; JSON backup and restore (share sheet on iOS) with a backup reminder.

**Check-in before a door opens (30 Oct).** Tapping a ready door asks "Is this one finished?".
- *Yes* asks again with one of eight randomly picked "are you sure?" questions, each with its own pair of answers. Confirming reveals the surprise.
- *Not yet* (or backing out of the second question) asks for a new date and time, defaulting to a week out. The door re-seals with the same surprise.

**Daily goals (31 Oct).** A "Today's goals" panel on the home screen, under the countdown. Kept off the tab bar to avoid a fifth tab.
- Adding a goal secretly draws a small treat from the Daily treats jar.
- Ticking needs two taps: the first arms it ("Really done? Tap the box again", disarms after 5 s), the second reveals the treat with a smaller confetti burst.
- Unfinished goals from earlier days show under "Still open from earlier" with a "Move to today" button. They keep their treat.
- The Jar tab has a toggle between Milestone treats and Daily treats.

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
- Animated buttons (the wiggling goal confirm) never count as "stable" for Playwright. Use `click(force=True)` in tests; real taps are fine.

## Open questions

- Daily goals don't appear on the calendar, and there's no streak or history view. Not asked for yet.
- Goals are always added for today. There's no way to plan a goal for a later day.
