# Deploy

Every app is hosted on GitHub Pages at `https://achu2-161.github.io/<repo>/`.

## Public or private repo

Adrian's account has **GitHub Pro through the education pack**, so Pages works from a **private** repo too.
- Private hides the **code** only. The **site is still public** to anyone with the link.
- On a free account, Pages from a private repo fails **silently**: no "your site is live" banner, and the URL 404s. This is what happened during Brownie Points' setup.
- A private repo doesn't protect answers or secrets in files the site serves (the quiz's `questions.js`, for example). Anything the browser downloads is readable.

## Route A: Claude pushes (preferred)

Claude can push to existing repos but can't create repos or switch on Pages.

1. **Adrian** creates an empty repo at <https://github.com/new>: no README, no .gitignore, no licence.
2. **Adrian** gives Claude's GitHub app access to it. New repos aren't added automatically:
   GitHub → profile picture → **Settings** → **Applications** → **Installed GitHub Apps** → **Claude** → **Configure** → *Select repositories* → add it → **Save**.
3. **Claude** clones, adds the files (plus `.nojekyll` if needed), commits on `main` and pushes.
4. **Adrian** switches on Pages: **Settings** → **Pages** → *Deploy from a branch* → `main`, `/ (root)` → **Save**.
5. Wait a minute or two, then check the live link. Claude's sandbox may not be able to reach `github.io`, in which case Adrian checks.

## Route B: manual upload

1. Create the repo as above.
2. Click **uploading an existing file** and drag in all the unzipped files, including any folders like `icons/`.
3. Switch on Pages as in step 4 above.

## Second host: Netlify

Used for Karimov & Co. as a **workaround** when Adrian's phone wouldn't install a new app from `github.io`. The real cause was a Chrome 156 bug (see [LESSONS.md](LESSONS.md#deploy-and-install)), and a second app on the same Netlify address then hit the same bug. Each Netlify project gets its own address (`<name>.netlify.app`), so one project per app would work around it; a second app on an existing project's address won't. Chrome Beta is the simpler workaround. A Netlify project can serve the same repo at a new origin (`<name>.netlify.app`) and auto-deploys every push to `main`. Settings: no build command, publish directory `.`. Then set **Project configuration → Visitor access → Public** (new projects from private repos can default to private). Sign up with Google or a plain email if GitHub sign-up fails on a "+" address. Free-plan projects created since Aug 2026 get a "Powered by Netlify" badge; if the settings toggle isn't offered, a `_headers` file with a `script-src` CSP that omits `'unsafe-inline'` keeps it off (no inline scripts in the app, list every script host).

## Pages quirks

- The first deployment sometimes doesn't trigger. Set Source to **None**, save, then back to **Deploy from a branch**.
- `/index.html` can 404 for a while when the bare folder URL already works. Share the bare URL.
- Changes can take a minute or two to appear, and a service worker may serve the old version until the app is reopened. Bump the cache version on every deploy.

## Making it installable (PWA)

Android and iOS need **different assets**. Include both from the start.

| File | For | Notes |
|---|---|---|
| `manifest.json` / `manifest.webmanifest` | Android | `name`, `short_name`, `start_url`, `display: standalone`, colours, icons |
| `icon-192.png`, `icon-512.png` | Android | These exact sizes. **Real files**, never base64 data URIs in the manifest |
| `icon-512-maskable.png` | Android | Full-bleed background, art inside the central ~80% |
| `apple-touch-icon.png` (180×180) | iOS | iOS ignores the manifest for install |
| `<meta>` tags | iOS | `apple-mobile-web-app-capable`, `apple-mobile-web-app-title`, status-bar style |
| `sw.js` | Android | Registered service worker. See [ARCHITECTURE.md](ARCHITECTURE.md#offline-and-service-workers) |

### Checking an install really worked (Android)

- `chrome://webapks` on the phone is the only reliable check. If the app isn't listed, it's a shortcut, not an install, however convincing the "Installing…" notification looked.
- Launchers like Nova may call a real install a "shortcut". Android's **Settings → Apps** list is the ground truth.
- The base64-icons-in-manifest mistake passed Chrome's local install check but broke Google's server-side WebAPK step.

### Installing on iPhone

Safari → Share → **Add to Home Screen**. No manifest involved.

## After deploying

- Update the status in [PORTFOLIO.md](PORTFOLIO.md) and the app's file.
- If Firebase is involved, check the Live pill (or equivalent) on two devices.
