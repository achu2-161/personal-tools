# Workflow

How Adrian and Claude work together on these apps.

## Before building

- **Discuss first** when it's a new app, a change to a mechanic, or anything architectural. Give options with tradeoffs and let Adrian decide.
- **Say which tool suits the job.** Chat with the sandbox is right for apps this size. Claude Code is worth it when a project needs npm or CLI tooling, or a git-backed test loop (for example, adding Firebase sync to the Field Trip Tracker). Explain why.
- **Flag new network dependencies** (CDNs, APIs, backends) as a tradeoff. Don't add them by default.
- **Check before rebuilding something that was removed.** The Field Trip map was built and then removed; ask whether it's wanted before bringing it back.

## While building

- Use a task list for multi-stage work.
- Test in the sandbox before handing over ([TESTING.md](TESTING.md)).
- Keep secrets and shared-data effects visible: say plainly what's public, what syncs to the other person, and what a passcode does and doesn't protect.

## Delivering

- **First delivery**: a zip with a README for setup and deploy.
- **After that**: only the changed file. A zip only if many files changed.
- **When Adrian is editing his own copy**: exact snippets with line references, not a rebuilt file.
- Say what changed in a line or two, and what wasn't tested.

## Iterating

- Adrian sends short lists of changes, often numbered. Do exactly those. Don't bundle in unrequested changes.
- Feedback can be terse ("yeah na"). Diagnose from the screenshot rather than asking him to elaborate.
- He wants to know **why** something broke, not only the fix.
- He leans towards less: removing clutter, merging duplicate controls, simpler interactions.
- Don't re-litigate settled choices (for example the single "clean up duplicates" button) without checking what he wants now.

## Safety

- **Warn before anything destructive** and say what to back up first. "Clear site data" once wiped real trip data because the warning came too late.
- **Warn before anything that touches shared data**: imports, resets, migrations.
- **Don't rename storage keys** without a migration that reads the old keys. Field Trip still uses `holidayTracker*` keys for this reason.
- Never assert something works on partial evidence.

## Communication

- Technical explanations are welcome. Keep them plain.
- Setup tasks (Firebase console, GitHub settings) go step by step, with screenshots back and forth, rather than one long list up front.
- Credit lines on public apps use Adrian's full name, e.g. "Created by Adrian Chu for RSA 2026".

## Where things live

- Code: one repo per app under `achu2-161`.
- Playbook: this repo, `personal-tools`.
- Conversation: the "Tools" Claude Project, which should have this repo as context.
