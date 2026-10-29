# Karimov & Co.

**A tongue-in-cheek book "acquisition" service: Ella and Nina (and Fiona) request Kindle books from Adrian's fictional Uzbek contact Ulug'bek Karimov, who asks clarifying questions, delivers, and invoices them in panna cotta.**

| | |
|---|---|
| Site | `https://achu2-161.github.io/karimov-co/` (client) · `…/master.html` (back office) |
| Repo | `achu2-161/karimov-co` |
| Data tier | Firestore, or local test mode when `firebase-config.js` is `null` |
| Firebase project | to be created (suggested `karimov-co`, `australia-southeast2`) |
| Users and devices | Clients on their own devices via one shared link; Adrian on the back office |
| Status | Built and sandbox-tested (Oct 2026). Firebase setup and Pages not yet switched on |

## Files

| File | Purpose |
|---|---|
| `index.html`, `client.js` | Client page: place requests, answer clarifications, add notes, see invoices |
| `master.html`, `master.js` | Back office: requests (status, clarifications, edit, delete), invoices, payments, backup/restore |
| `shared.js` | `K.Store` (dual-mode), helpers, invoice card and thread rendering |
| `app.css` | Shared styles |
| `firebase-config.js` | `window.FIREBASE_CONFIG`; `null` = local test mode |
| `firestore.rules` | Open on `requests` and `invoices`, closed elsewhere |
| `karimov-backup-starting.json` | The two invoices carried over from the earlier Claude-artifact ledger |

## Data model

- `requests/{id}`: `title`, `author`, `notes`, `requestedBy`, `status` (`new` · `query` · `sourcing` · `delivered` · `declined`), `createdAt`, `updatedAt`, `invoiceId`, `messages[]` (`{id, from: 'client'|'karimov', name, kind: 'query'|'note', text, at}`).
- `invoices/{id}`: `number`, `date`, `billTo`, `billNote`, `item`, `qty`, `total`, `rate`, `unit`, `unitPlural`, `notes`, `requestIds[]`, `deductions[]` (`{id, amount, date, method, note}`), `createdAt`.
- Derived, not stored: amount paid and remaining, totals owed per currency, "needs attention" (status `new`, or client wrote last on an open request).

## How it works

- Messages append with `FieldValue.arrayUnion` together with any status change, in one write. Asking for clarification sets `query`; a client's answer sets it back to `new`.
- Saving an invoice links and unlinks its requests' `invoiceId` in one batch.
- Renders are guarded by a data signature, and unsent reply text is kept across live updates.
- No native dialogs: two-tap confirm for deletes.

## Look

Aged-paper sheet with a double rule border, Adrian's own brand assets (armillary logo, red dotted divider, red seal, "No payment required" stamp for paid-off invoices). Fonts: Noto Serif (brand), Courier Prime (body), Special Elite (headings and figures). Statuses are small rubber-stamp labels. The paper texture is a generated seamless tile at 2× for sharpness.

## Decisions

- **Moved out of Claude artifacts** so clients don't need claude.ai accounts.
- **One shared client link** (Adrian's choice): every client sees every request and invoice. Names are typed per request and remembered per device.
- **Trust-based**: `master.html` has no password; rules are open on the two collections.

## Open questions

- Firebase project and Pages not set up yet.
- Existing invoice 7762's notes mention "Adrian Chu", which breaks the fiction on the client page.
