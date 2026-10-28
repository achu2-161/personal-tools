# Pipeline

How an app goes from idea to installed and maintained. This is the sequence every app so far has followed, written down so the next one can follow it on purpose.

```
1 Brief → 2 Discuss → 3 Pick data tier → 4 Build in sandbox → 5 Test in sandbox
   → 6 Package → 7 Deploy → 8 Live test on phone → 9 Iterate → 10 Document
```

## 1. Brief

Adrian gives a short brief: purpose, who uses it, maybe a palette or a feeling. "Whatever other feature seems interesting" is common. That's an invitation to propose features, but discuss them before building.

Capture early:
- **Who uses it, on which devices.** One person on one phone? Two people syncing? A room of phones?
- **Does anything need to stay secret** from one of the users? This is the question that changes the architecture most (see Anila's Doors).
- **Does it need to live outside Claude?** So far the answer has always been yes. Assume a standalone GitHub Pages app unless told otherwise.

## 2. Discuss before building

Adrian wants design and architecture talked through before code, especially for mechanics and anything structural. Lay out options with tradeoffs and let him choose. Small, clear asks can go straight to building.

## 3. Pick the data tier

Use the decision guide in [ARCHITECTURE.md](ARCHITECTURE.md#choosing-a-data-tier). In short:
- One person, one device → **device only**, with JSON backup/restore.
- Several people or devices that must see the same data live → **Firebase**.
- Something must be hidden from a user → it needs **server-enforced rules**, or an honest conversation about accepting trust instead.

Requirements can change mid-build. Anila's Doors went Supabase → Firestore with time-gated rules → passcode → device only, each time because the brief changed. Keeping the data layer behind an interface is what made each switch cheap.

## 4. Build in the sandbox

- Plain HTML/CSS/JS, no build step. See [ARCHITECTURE.md](ARCHITECTURE.md) for file layout.
- If Firebase is involved, put the config in its own file (`firebase-config.js`). Leaving it `null` runs a **local mode**, so everything can be built and tested before Adrian creates a Firebase project.
- Generate icons programmatically (Pillow) to match the app's palette. Make all of them up front: 192, 512, maskable 512, and 180 apple-touch.
- Use a task list for anything with several stages.

## 5. Test in the sandbox

Before handing anything over, follow [TESTING.md](TESTING.md):
- Serve over `python3 -m http.server`, not `file://`.
- Run Playwright at phone size (390×844), plus laptop and landscape when layout matters.
- Look at the screenshots. Don't just check that the script passed.
- For multi-device apps, open several tabs and test the sync.

## 6. Package

- **First delivery** of a multi-file app: a zip of the whole folder, with a README covering setup and deploy steps.
- **Later rounds**: only the changed files. A zip only when many files changed.
- If Adrian wants to try it inside Claude first: an inlined single-file preview copy, with the service worker made a no-op. See [TESTING.md](TESTING.md#preview-copies-inside-claude).

## 7. Deploy

See [DEPLOY.md](DEPLOY.md). Two routes:
- **Claude pushes**: Adrian creates the empty repo and gives Claude's GitHub app access to it. Claude pushes. Adrian switches on Pages (Claude can't).
- **Manual upload**: Adrian drags the unzipped files into the repo on github.com.

Firebase setup is walked through step by step, with screenshots back and forth. Ask Adrian to **copy-paste** the config block rather than reading it from a screenshot.

## 8. Live test on the phone

Adrian installs it on his Samsung Galaxy S22 Ultra (and an iPhone, for shared apps), uses it, and reports back with screenshots or short notes. "yeah na" means something visibly wrong. Diagnose from the screenshot.

For desktop bugs, ask for a **DevTools console screenshot** early. A red error usually ends several turns of guessing.

## 9. Iterate

- Small, explicit asks. Do exactly what's asked; don't bundle in extra changes.
- Deliver the changed file only. When Adrian is editing his own copy, give exact snippets with line references instead.
- Warn **before** anything destructive or anything that touches shared data (clearing storage, imports that write to Firebase, resets).
- Verify the actual state before saying something works.

## 10. Document

At the end of a session, update this repo: the app's file in `apps/`, plus `LESSONS.md` and `ARCHITECTURE.md` if something general was learned. See the [README](README.md#keeping-this-repo-current).
