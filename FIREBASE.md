# Firebase

Used when people or devices need to share live data. Two products are in use. Pick using [ARCHITECTURE.md](ARCHITECTURE.md#choosing-a-data-tier).

| | Realtime Database | Firestore |
|---|---|---|
| Apps | Brownie Points, RSA Quiz | Adripedia |
| Shape | One JSON tree | Collections of documents |
| Live updates | `ref.on('value')` | `onSnapshot` |
| Atomic counters | `transaction()` | `FieldValue.increment()` |
| Lists | `push()`, which gives unique keys | one document per record |
| Connection status | `.info/connected` | no direct equivalent |
| Size limits | generous for small apps | **1 MiB per document** |

Both run on the free Spark plan. Neither needs Cloud Functions for what these apps do.

## Plans, limits and scaling up

Checked on Firebase's pricing page, 1 Nov 2026. Re-check before relying on figures.

| | Spark (free, no card) | Blaze (pay as you go, card on file) |
|---|---|---|
| Firestore | 1 GiB stored, 10 GiB egress/month, 50K reads/day, 20K writes/day | Same free allowance, then billed |
| Realtime Database | Free tier | Same free tier, then billed |
| **Cloud Storage** (files, photos) | **Not available** since 3 Feb 2026 | 5 GB stored, 100 GB downloads/month, 5K uploads and 50K downloads/month free (current `*.firebasestorage.app` buckets) |
| Authentication | Free to 50K monthly users; Google sign-in free | Same; SMS sign-in billed per message |

- **Blaze is per project.** One app can move to Blaze while the others stay on Spark. Keep apps with open rules (Brownie Points, RSA Quiz) on their own Spark projects.
- **Blaze has no hard spending cap.** Budget alerts only send email. Lock rules down before switching.
- Beyond the free allowance, rough rates are about US$0.02/GB/month stored and US$0.12/GB downloaded (from a third-party 2026 guide, US regions).
- Not verified: whether the free Storage allowance applies to Australian bucket regions. At these apps' sizes it costs a fraction of a cent either way.
- Google services, Firebase included, are generally blocked in mainland China without a VPN.

**Where the apps sit (1 Nov 2026):** nowhere near any limit. The largest data set is the Field Trip Tracker's ~65 MB backup (~49 MB of actual photos), which is still on the phone, not in Firebase. Under US$10/month on Blaze would cover tens of thousands of photos and a few dozen viewers.

**What would push costs up:** video, many viewers of photo-heavy galleries, listeners that re-read whole collections, open write rules on Blaze, SMS sign-in.

**Photo options, by plan:**
- **Spark:** compressed base64 inside Firestore documents (see [Writing safely](#writing-safely)). Free, but costs photo quality.
- **Blaze:** Cloud Storage, files named by their SHA-256 hash so re-uploads can't create duplicates, with the metadata in Firestore.

Worked example: the Field Trip Tracker sync discussion, in the "Personal Tools" project note `claude/handover-firebase-storage-blaze.md`.

## Loading the SDK

Use the **`-compat`** builds via plain `<script>` tags:

```html
<script src="https://www.gstatic.com/firebasejs/<version>/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/<version>/firebase-database-compat.js"></script>
<!-- or firebase-firestore-compat.js -->
```

The non-compat files are ES modules. Loaded with plain `<script src>` they fail with `Unexpected token 'export'`, followed by `firebase is not defined`.

## Config

Keep the config in its own file, `firebase-config.js`:

```js
window.FIREBASE_CONFIG = null;   // null = local test mode
```

- The app runs in local mode until the config is pasted in. See the dual-mode store in [ARCHITECTURE.md](ARCHITECTURE.md#the-store-interface).
- Realtime Database configs **must include `databaseURL`**.
- Ask Adrian to **copy-paste** the config from the console. Transcribing from a screenshot introduced spaces into Adripedia's API key, which went unnoticed because nothing used it yet.
- The web API key is **meant to be public**. GitHub's secret scanning will flag it. That's a false positive and nothing needs rotating. The rules are the real access boundary.

## Console setup (the console has changed since older tutorials)

1. **Add project**. Analytics can be off.
2. **Create the database separately.** A registered web app is not enough. Look under **Build** or **Databases and storage** depending on the console version.
   - Realtime Database: start in **locked mode**, then paste rules.
   - Firestore: choose the edition and location first. Test mode is offered at the *Configure* step.
   - **Location is permanent.** Used so far: `asia-southeast1` (Singapore) and `australia-southeast2` (Melbourne).
3. **Rules** tab: paste and publish. The version history sidebar is how to confirm they took.
4. **Project settings → Your apps → Web (`</>`)**: register an app (no hosting) and copy the config.

## Rules templates

### Realtime Database: open one path, close everything else

```json
{
  "rules": {
    ".read": false,
    ".write": false,
    "quiz": { ".read": true, ".write": true }
  }
}
```

Replace `quiz` with the app's root node (Brownie Points uses `brownieTracker`).

### Firestore: open one collection

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /entries/{id} {
      allow read, write: if true;
    }
  }
}
```

Both are **identification, not security**. They're fine for trusted users and non-sensitive data.

### Time-gated reads: keeping something secret until a date

Worked out for Anila's Doors (later dropped when the brief changed). It lets a public page read a field only after a server-checked time, without Cloud Functions:

```
match /milestones/{id} {
  allow read: if request.time >= resource.data.unlockAt;
  // the client may flip one field only
  allow update: if request.time >= resource.data.unlockAt
                && request.resource.data.diff(resource.data).affectedKeys().hasOnly(['opened']);
}
```

Put the secret in a document of its own (or a subcollection) gated like this. Fields readable from the same document before unlock are not secret.

## Writing safely

- **Counters**: `transaction()` (Realtime Database) or `FieldValue.increment()` (Firestore). Never read, add and write back.
- **Lists**: `push()`, which can't overwrite when two devices write at once.
- **One action = one write.** A counter bump and its log entry go together.
- **Document IDs from content** (`term + '_' + owner`) make renames tricky. Capture the old ID before mutating, write the new document, and delete the old one only after the write succeeds.
- **Images in Firestore**: base64 in the document works and needs no Storage bucket (so it stays on Spark), but compress first (1000 px longest edge, JPEG 0.72 → typically 60–150 KB). Base64 adds ~33%, so anything over ~780 KB raw fails.
- **Catch blocks must tell the user.** A write that fails while the UI looks saved is silent data loss.

## Verifying

A `console.log('Saved')` proves nothing. Confirm persistence by refreshing, opening on a second device, or looking at the data in the Firebase console.

## Alternatives considered

- **Supabase**: used briefly for Anila's Doors. Fine, but Adrian already knows Firebase, so prefer Firebase unless there's a reason.
- **SharePoint / Microsoft 365 (Graph API)**: discussed for work contexts. It has no real-time listener equivalent, needs Azure AD sign-in with IT involvement, and Power Apps is the officially supported route.
