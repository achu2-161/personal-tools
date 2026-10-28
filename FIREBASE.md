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
- **Images in Firestore**: base64 in the document works and needs no Storage bucket, but compress first (1000 px longest edge, JPEG 0.72 → typically 60–150 KB). Base64 adds ~33%, so anything over ~780 KB raw fails.
- **Catch blocks must tell the user.** A write that fails while the UI looks saved is silent data loss.

## Verifying

A `console.log('Saved')` proves nothing. Confirm persistence by refreshing, opening on a second device, or looking at the data in the Firebase console.

## Alternatives considered

- **Supabase**: used briefly for Anila's Doors. Fine, but Adrian already knows Firebase, so prefer Firebase unless there's a reason.
- **SharePoint / Microsoft 365 (Graph API)**: discussed for work contexts. It has no real-time listener equivalent, needs Azure AD sign-in with IT involvement, and Power Apps is the officially supported route.
