# Karimov & Co.

**A tongue-in-cheek book "acquisition" service: Ella and Nina (and Fiona) request Kindle books from Adrian's fictional Uzbek contact Ulug'bek Karimov, who asks clarifying questions, delivers, and invoices them in panna cotta.**

| | |
|---|---|
| Site | Client: `https://achu2-161.github.io/karimov-co/` · Back office: `https://karimov-co.netlify.app/office/` (Adrian's install) — both hosts serve the same repo; `master.html` forwards to `office/` |
| Repo | `achu2-161/karimov-co` (private) |
| Hosting | GitHub Pages and Netlify (project `karimov-co`, auto-deploys `main`, publish dir `.`, visibility set to Public) |
| Data tier | Firestore, or local test mode when `firebase-config.js` is `null` |
| Firebase project | `karimov-co`, Firestore in `australia-southeast2` (Spark plan) |
| Users and devices | Clients on their own devices via one shared link; Adrian on the back office |
| Status | Live on GitHub Pages with Firestore (30 Oct 2026). Beta testing |

## Files

| File | Purpose |
|---|---|
| `index.html`, `client.js` | Client page: place requests, answer clarifications, add notes, see invoices |
| `office/index.html`, `office/manifest.webmanifest`, `master.js` | Back office (own folder, scope and red-seal icon so it installs separately): requests (status, clarifications, edit, delete), invoices, payments, backup/restore |
| `shared.js` | `K.Store` (dual-mode), helpers, invoice card and thread rendering |
| `app.css` | Shared styles |
| `firebase-config.js` | `window.FIREBASE_CONFIG`; `null` = local test mode |
| `firestore.rules` | Open on `requests` and `invoices`, closed elsewhere |
| `karimov-backup-starting.json` | The two invoices carried over from the earlier Claude-artifact ledger |

## Data model

- `requests/{id}`: `title` (book or collection), `author`, `notes`, `requestedBy`, `books` (1 or more for a series), `deliveredBooks`, `status` (`new` · `query` · `sourcing` · `waiting` · `delivered` · `declined`), `createdAt`, `updatedAt`, `messages[]` (`{id, from: 'client'|'karimov', name, kind: 'query'|'note', text, at}`).
- `invoices/{id}`: `number` (random `NNNN/UZB/MM-YY`), `date`, `billTo`, `billNote`, `item`, `qty`, `price` (per book), `total`, `unit` + `unitPlural`, `rate` (display text), `notes`, `lines[]` (`{requestId, books}`), `deductions[]` (`{id, amount, date, method, note}`), `createdAt`. Older invoices may have `requestIds[]` instead of `lines`; `linesOf()` reads both.
- Derived, not stored: books billed per request (sum of invoice lines), books still billable (delivered − billed), amount paid and remaining, totals owed per currency, "needs attention".

## How it works

- Messages append with `FieldValue.arrayUnion` together with any status change, in one write. Asking for clarification sets `query`; a client's answer sets it back to `new`.
- Invoices own the link to requests through `lines`; requests are never written when invoicing, so a series can be billed across several invoices.
- Renders are guarded by a data signature, and unsent reply text is kept across live updates.
- No native dialogs: two-tap confirm for deletes.

## Look

Aged-paper sheet with a double rule border, Adrian's own brand assets (armillary logo, red dotted divider, red seal, "No payment required" stamp for paid-off invoices). Fonts: Noto Serif (brand), Courier Prime (body), Special Elite (headings and figures). Statuses are small rubber-stamp labels. The paper texture is a generated seamless tile at 2× for sharpness.

## Decisions

- **Lore**: Adrian Chu is Ulug'bek Karimov's agent, so "payable to Adrian Chu" on invoices is in character.
- **Moved out of Claude artifacts** so clients don't need claude.ai accounts.
- **One shared client link** (Adrian's choice): every client sees every request and invoice. Names are typed per request and remembered per device.
- **Back office in `office/`**: with both pages at the root sharing scope `./`, Chrome said "already installed" on `master.html` and offered only a shortcut.
- **Trust-based**: the back office has no password; rules are open on the two collections.

## App-specific lessons

- Adrian's S22 said "already installed" / "Could not open app" for every Karimov page on `achu2-161.github.io`. Cause: a Chrome 156 bug that refuses a second app on an address that already has one (see [LESSONS.md](../LESSONS.md#deploy-and-install)), not a broken record. Serving the repo from `karimov-co.netlify.app` worked for the back office only because it was the first app there; the client page on Netlify then failed the same way (1 Nov). Other phones were never affected.
- Netlify sign-up with GitHub failed ("Email address is invalid") while GitHub's primary email was a Gmail "+" address; switching the primary email to the unimelb address worked.
- New Netlify projects from a private repo defaulted to **Private** visibility (team login required). Set Project configuration → Visitor access → Customize → Public.
- The free-plan "Powered by Netlify" badge couldn't be switched off in settings. It's an injected inline script, so `_headers` sets `Content-Security-Policy: script-src 'self' https://www.gstatic.com`. Keep the pages free of inline `<script>` blocks (the service worker is registered from `shared.js`; `master.html` uses `redirect.js`). Add any new script host to that header.

## Open questions

- Once regular Chrome on Adrian's S22 updates past 156, retry installing the client page.
