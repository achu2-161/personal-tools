# Lessons

Bugs and mistakes that actually happened, grouped by topic. Each one says what went wrong and the rule that came out of it. The app it happened in is in brackets.

## Data loss and shared data

- **"Clear site data" wipes localStorage and IndexedDB, with no undo.** It was suggested as a debugging step without a clear warning, and real trip data was lost. Rule: back up first, and say so before suggesting it. *(Field Trip)*
- **A caught-and-logged write error is silent data loss.** The card looked saved; nothing reached the cloud. Rule: every catch around a persistence call must reach the user. *(Adripedia)*
- **Restoring a backup writes to the shared database** and syncs to the other person's device. Rule: don't use the real import for test data; build a stubbed test copy. *(Adripedia)*
- **Renaming storage keys orphans saved data.** Rule: only with a migration that reads old and new keys. *(Field Trip)*

## Import and duplicates

- **Records without stable IDs duplicate on re-import.** Rule: every record gets an ID; import overwrites by ID. *(Field Trip info cards)*
- **Photos duplicate the same way.** Rule: SHA-256 of the blob as a content hash, matched per date. *(Field Trip)*
- **Old backups can already contain duplicates.** Rule: run clean-up on every import as a safety net. *(Field Trip)*

## Firebase

- **Modular SDK via `<script src>` fails** with `Unexpected token 'export'`. Rule: use `-compat` builds. *(Adripedia)*
- **The database must be created separately** from the Firebase project. *(Adripedia)*
- **Wrapping a function (`window.f = …`) to add saving silently did nothing.** Rule: edit the function itself. *(Adripedia)*
- **A log line said "Saved" and "Deleted" while nothing was written.** Rule: verify by refreshing or in the console, never by a log. *(Adripedia)*
- **Hardcoded demo entries reappeared on refresh**, looking exactly like broken deletes. Rule: remove seed data once a backend exists. *(Adripedia)*
- **Base64 images over ~780 KB exceed Firestore's 1 MiB document limit.** Rule: compress on upload. *(Adripedia)*
- **Renaming mutated the ID before saving**, orphaning the old document. Rule: capture the old ID, write new, delete old on success. *(Adripedia)*
- **Config transcribed from a screenshot had errors** that stayed hidden until a future feature needed them. Rule: copy-paste the config. *(Adripedia)*
- **View state leaked into data**: entries made from the "All" view were saved with `owner: "all"` and vanished from both lists. Rule: watch for UI state values written into records. *(Adripedia)*

## Concurrency and counting

- **Read-then-write counters lose taps** when two devices write at once. Rule: `transaction()` or `increment()`. *(Brownie Points)*
- **Two writes per action caused overlap errors.** Rule: one action = one combined write. *(Brownie Points)*
- **Incrementing a score double-counted** when going back and re-answering. Rule: store answers, derive the score. *(Adripedia quiz)*
- **Claude's artifact storage failed under rapid writes.** Rule: not a backend, even for testing. Don't port its retry scaffolding to Firebase. *(Brownie Points)*

## Deploy and install

- **Base64 icons in the manifest passed the local install check but broke WebAPK minting.** Rule: real hosted icon files; check `chrome://webapks`. *(Field Trip)*
- **Pages on a private repo fails silently on a free account.** Now on Pro, so private works, but the site stays public. *(Brownie Points, Anila's Doors)*
- **First Pages deploy didn't trigger.** Rule: toggle Source to None and back. *(Brownie Points)*
- **Two installable apps on one site need separate folders.** A second page sharing the first app's scope was treated as "already installed" and could only be added as a shortcut. Rule: give each installable role its own folder and manifest scope (e.g. `office/`). *(Karimov & Co.)*
- **iOS ignores the manifest.** Rule: ship `apple-touch-icon.png` and the iOS meta tags from the start. *(Brownie Points)*
- **Claude's GitHub app can't see newly created repos** until they're added in its settings, and it can't create repos or switch on Pages. *(Anila's Doors)*

## JavaScript

- **`classList.toggle(name, undefined)` toggles instead of removing.** Rule: always pass a real boolean. *(Adripedia)*
- **`sort()` then `reverse()` flipped tied items on every render.** Rule: put direction inside the comparator and add a tiebreaker. *(Adripedia)*
- **`const` inside `try` isn't visible in `catch`**, so the error handler itself threw. Rule: declare with `let` before the `try`. *(Adripedia)*
- **`list.innerHTML = ''` destroyed hidden cards.** Rule: hide with `display: none` and reorder with `appendChild`. *(Adripedia)*
- **Re-rendering on every update replaced DOM nodes mid-click.** Rule: re-render only when the relevant data changes. *(RSA Quiz)*
- **Native `confirm()` was silently blocked**, so delete and reset did nothing. Rule: in-app modals or two-tap confirm. *(Brownie Points, Anila's Doors)*

## CSS and layout

- **Percentage padding resolves against the container's width**, which squeezed buzzer text to one letter per line in wide windows. Rule: size padding from the same custom property as the element. *(RSA Quiz)*
- **A sticky header in a padded scroll container leaves a gap.** Rule: zero the container's top padding; let the header pad itself. *(Field Trip)*
- **`100vh` plus body padding overflowed**, adding a stray scrollbar. Rule: `100dvh`. *(Adripedia)*
- **Borders and shadows on same-coloured backgrounds read as ghost outlines.** Rule: remove decoration that frames nothing. *(Adripedia)*
- **A decorative graphic covered the seconds digit on mobile.** Rule: check screenshots at phone size. *(Anila's Doors)*

## Process

- **The sandbox copy diverged from what was delivered** (a reset; a half-finished edit left orphaned CSS). Rule: diff against the last delivered version when in doubt. *(Field Trip)*
- **"It's working" was claimed on partial evidence**, and a screenshot was misread. Rule: check actual state; believe the user over the logs. *(Adripedia)*
- **A console screenshot solved in one step what guessing hadn't in several.** Rule: ask for it early. *(Adripedia)*
