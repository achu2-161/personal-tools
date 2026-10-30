# Portfolio

Last reviewed: 2026-10-29. All sites are under `https://achu2-161.github.io/`.

| App | What it does | Repo | Data tier | Status |
|---|---|---|---|---|
| [Field Trip Tracker](apps/field-trip-tracker.md) | Itinerary, journal and info cards for Yixiao's 2026 China fieldwork | `fieldwork-tracking` | Device only (localStorage + IndexedDB) | Deployed, installed on Adrian's S22. Trip finished 24 Sep 2026 |
| [Brownie Points](apps/brownie-points.md) | Shared points and credits between Adrian and Yixiao | `brownie-points` | Firebase Realtime Database | Deployed. Core mechanic under reconsideration |
| [Adripedia](apps/adripedia.md) | Two shared word encyclopedias, study and quiz mode, CSV importer | `adripedia` | Firestore | Deployed, stable |
| [RSA Quiz](apps/rsa-quiz.md) | Game-show quiz: projected board, phone buzzers, MC screen | _not recorded_ | Firebase Realtime Database (or local test mode) | Built and refined. Deployment not confirmed |
| [Anila's Doors](apps/anila-doors.md) | PhD milestone advent calendar with a secret surprise jar | `anila-doors` (private) | Device only (localStorage) | Deployed from a private repo |
| [Karimov & Co.](apps/karimov-co.md) | Book requests from friends' kids to a fictional Uzbek "contact", clarifications, and panna cotta invoices | `karimov-co` | Firestore | Live (Pages + Firestore), beta testing |

## Patterns at a glance

| | Field Trip | Brownie | Adripedia | Quiz | Anila |
|---|---|---|---|---|---|
| File layout | single file | single file | single file + importer page | multi-page + shared JS | multi-file (`app.js`, `app.css`) |
| Users | 1 | 2, attributed | 2, attributed | many devices, 1 game | 1 |
| Identity | none | `?u=1/2` link | owner label | `?team=N` link | none |
| Backup | JSON export/import (incl. photos) | `.txt` export, stored centrally | JSON backup/restore | n/a | JSON export/import |
| Service worker | pass-through | app shell cached, data live | network-first | not recorded | offline cache |
| Installable | Android | Android + iOS | Android + iOS | n/a (browser pages) | Android + iOS |

## Open threads

- **Brownie Points**: choose between the current system, two independent ledgers, or a zero-sum pool. See its app file.
- **Field Trip Tracker**: Firebase sync and multi-trip support were discussed, not built.
- **Adripedia**: tags, spaced repetition and a config object for encyclopedia names were discussed, not built.
- **RSA Quiz**: confirm the repo name and whether it has been deployed.
