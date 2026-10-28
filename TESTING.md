# Testing

Two loops: Claude tests in the sandbox before handing anything over, then Adrian tests live on his phone. The first loop exists so the second one finds fewer problems.

## Build variants

Know which one you're editing.

| Variant | Storage | Purpose |
|---|---|---|
| **Chat preview** | Claude artifact storage, or none | Design review inside Claude. Never deployed |
| **Local / phone-test** | `localStorage` (or a stubbed Firebase) | Runs anywhere with no backend. Rehearsals, phone tests before Firebase exists |
| **Deployed** | Real backend | Production |

With a dual-mode store ([ARCHITECTURE.md](ARCHITECTURE.md#the-store-interface)), the local and deployed variants are the **same files**: only `firebase-config.js` differs. That's the setup to aim for. Older apps (Brownie Points) kept separate files per variant.

## Sandbox testing

### Serve over HTTP

```bash
python3 -m http.server 8765
```

Don't open files via `file://`. Some browsers block `localStorage` and the `storage` event between tabs, and service workers won't register.

### Playwright at phone size

Use headless Chromium at **390×844**. Add a laptop size and a landscape phone when layout matters, and a long name or long text to stress wrapping.

Test the real flows, not just page loads: add, edit, delete, import a backup, open a door, buzz. Then **look at the screenshots**. Several layout bugs (squeezed buzzer text, a decoration covering a digit) passed functional checks and were only visible in a picture.

### Multi-device apps

Open several tabs in one browser context: host, two or three buzzers, MC. With local mode they sync through the `storage` event. Check ordering (who buzzed first) and that every screen updates.

If a click is lost in tests, check whether a re-render is replacing the DOM node between click and handler. Guard re-renders with a signature of the data that actually matters.

### Testing a Firebase app without touching real data

When an app has no local mode, build a test copy that stubs Firebase. Define the stub **before** the app's own Firebase code, so no app code changes:

```js
const _stubDoc        = { set: async () => {}, delete: async () => {} };
const _stubCollection = { doc: () => _stubDoc, get: async () => ({ forEach: () => {} }), onSnapshot: () => {} };
const _stubFirestore  = { collection: () => _stubCollection, enablePersistence: () => Promise.resolve() };
const firebase = { initializeApp: () => {}, firestore: () => _stubFirestore };
```

Then strip the Firebase CDN tags and the service worker, bake in demo data, change the `<title>` and add a visible "TEST" badge. Generate the test copy from a script so it can be regenerated after each change.

Restoring a backup in the real app writes to Firebase and syncs to the other person. **Never use the real app's import to load test data.**

## Preview copies inside Claude

When Adrian wants to try an app in the chat:
- Inline the CSS and JS into one HTML file.
- Make the service worker registration a no-op.
- Add the `downloads` capability so backup/export still works.
- Tell him the preview's data is separate from the installed app's.

Native dialogs (`confirm`, `alert`, `prompt`) are blocked in the preview. The apps don't use them anyway. See [ARCHITECTURE.md](ARCHITECTURE.md#ui-mechanics-that-recur).

## Live testing (Adrian)

- Android: Samsung Galaxy S22 Ultra. iOS: an iPhone, for shared apps.
- Desktop: Chrome with DevTools open. **Ask for a console screenshot early** when something breaks.
- Install checks: see [DEPLOY.md](DEPLOY.md#checking-an-install-really-worked-android).

## Before saying it works

- Check the actual state: refresh, a second device, the Firebase console. A log line saying "saved" isn't evidence.
- Diff the working copy against the last delivered version if there's any doubt about which is current. The sandbox can reset, and a half-finished edit once left CSS with no matching JS.
- When Adrian says something still doesn't work, believe him over the logs.
- Say what wasn't tested ("phones haven't been tested, that needs your Firebase project").
