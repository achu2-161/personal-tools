# personal-tools

The playbook for Adrian's personal web apps: how they're built, deployed, tested and maintained.
This repo holds **docs only**. Each app's code lives in its own repo.

It's written for two readers:
- **Claude**, picking up work in the "Tools" Claude Project. Read this before starting or changing any app.
- **Adrian**, as a reference for how the pipeline works and why.

Teaching tools (Canvas modules, calculators) are deliberately **out of scope**. They'll get their own project.

## What's here

| File | Read it when… |
|---|---|
| [PORTFOLIO.md](PORTFOLIO.md) | You need to know what exists, where it lives and its current state |
| [PIPELINE.md](PIPELINE.md) | Starting a new app, or a large change. The end-to-end lifecycle |
| [ARCHITECTURE.md](ARCHITECTURE.md) | Deciding how an app is structured: files, storage, identity, offline |
| [FIREBASE.md](FIREBASE.md) | An app needs sync between devices or people |
| [DEPLOY.md](DEPLOY.md) | Putting an app on GitHub Pages or making it installable |
| [TESTING.md](TESTING.md) | Before handing over any build |
| [WORKFLOW.md](WORKFLOW.md) | Before proposing or delivering anything. How Adrian likes to work |
| [LESSONS.md](LESSONS.md) | Debugging, or touching storage, import, deploy or install logic |
| [apps/](apps/) | Working on a specific app. One file per app |

## The house stack in one paragraph

Plain HTML/CSS/JS with no framework and no build step, hosted on GitHub Pages under `achu2-161`, installable as a PWA on Android and iOS. Data lives in one of three tiers: on the device only, Firebase Realtime Database, or Firestore. All data access goes through a small store interface so the backend can be swapped. There are no real logins; people are identified by a link. Every app is built and tested in Claude's sandbox first (Playwright at phone size), then deployed and tested live on Adrian's phone.

**Visual style is per app.** Each app gets its own look. These docs only cover the pipeline.

## Keeping this repo current

At the end of any session that changes an app:
1. Update that app's file in `apps/` (state, files, decisions, open questions).
2. Add any new general lesson to `LESSONS.md`, and any new pattern to `ARCHITECTURE.md`.
3. Update the status line in `PORTFOLIO.md`.

When a new app starts, copy `apps/_template.md`.

If a doc and the code disagree, **the code wins**. Fix the doc.
