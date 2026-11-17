# Spec

A personal daily diary ("moy den" is Russian for "my day"): one static `index.html` page, served by GitHub Pages, that shows an encrypted questionnaire and saves each day's answers as a JSON file in the owner's Google Drive.

Every `##` section below is one task. `Status` is one of To Do, In Progress, In Review, Done. The whole file is one ticket in Jira project KAN (the `Jira` line below), where every task is a checkbox: tick it there and the sync marks the task Done here; add one there and it is appended here.

Jira: KAN-61

## Load the encrypted questionnaire
Status: Done

On every opening the page fetches `questions.enc` (`{v, iv, ct}`, AES-GCM) with `cache: "no-store"` and decrypts it with WebCrypto, using a key that arrives once as `#k=<key>` and is kept in localStorage under `moyden.key`; a key the file rejects is not stored. When the fetch or the decryption fails, the page keeps the last saved list and says why in the status bar, and the older `#q=<base64>` route that carries the whole list in the fragment still works. The fragment is cleared from the address once `q` or `k` has been read. This lives in `fetchQuestions` and `init` in `index.html`.

## Fill in the day's questionnaire
Status: Done

`render`, `onToggle` and `paint` in `index.html` draw each section: rows with a done and an enjoyed toggle (marking enjoyed also marks done, unmarking done clears enjoyed), red done toggles in `negative` sections, a row of chips for `kind: "choice"`, and a slider for `kind: "hours"` (0 to 16 hours unless the section sets `min`, `max` or `step`) whose value stays unset until it is touched and can be cleared again. A free-text diary field follows the sections, and the header shows the day, the questionnaire version and how many items are marked out of the total. Without a questionnaire the page shows what the key link looks like instead.

## Pick the day and keep per-day drafts
Status: Done

The day rolls over at 05:00, not at midnight (`dayKey`), so an entry made at 00:30 belongs to the previous day. The date picker in `setupDayPicker` allows earlier days, clamps a future date to today, shows a past-day notice and reloads the state for the chosen day. Every change is saved as a draft in localStorage under `moyden.draft.<YYYY-MM-DD>`, and ticks passed in an `#auto=` fragment are applied only to items the draft has not touched yet.

## Save the day to Google Drive
Status: Done

The send button gets a token through Google Identity Services with only the `drive.file` scope (`getToken`), finds or creates the diary folder and caches its id under `moyden.folderId` (`ensureFolder`), and writes one `<YYYY-MM-DD>.json` per day, overwriting that day's file with a PATCH when it already exists (`upload`). The file holds `date`, `submitted_at` (UTC), `submitted_local`, `tz_offset_min`, `questions_version`, the `done`, `enjoyed` and `asked` item ids, `diary` and the slider `values`. The status bar reports saved or overwritten with the file name, or the error text as it came back. The OAuth client ID is read from `app-config.json`.

## Show how-to-enjoy protocols
Status: Done

Items with a protocol carry a `?` mark, and tapping the label opens a sheet with the protocol's group, title, minutes and steps (`openGuide`). An item without a protocol of its own falls back to the `base` protocol, or to `sad_refusal` in a negative section (`protocolsFor`). A separate button lists every protocol grouped by `group`, so protocols tied to no item can still be opened (`openAllGuides`). Protocols come from the `protocols` array of the loaded questionnaire; without it the labels are simply not clickable.

## Repository tooling and lab gates
Status: Done

`recipe.ps1` is the repository's one interface: `test` points `core.hooksPath` at `.claude/hooks/git` and runs pytest (accepting exit 5, no tests collected), `ruff check` and `ruff format` over `.claude`; `info` prints the metadata, `clean` removes tool caches, and the other verbs print `nothing to <verb>`. The git hooks check the commit subject and the Signed-off-by trailer (`.claude/hooks/git/commit-msg.ps1`) and the branch name and pushes to `release` (`.claude/hooks/git/pre-push.ps1`), and `.claude/hooks/gate.ps1`, wired into `.claude/settings.json` for Claude Code's Bash and PowerShell calls, refuses recursive deletes, force-pushes, history rewrites on `test` or `release` and tool calls it cannot read. The Gitea workflows in `.gitea/workflows/` run `recipe.ps1 test` on proposals into `test` and on `test` itself, gate the way into `release`, and sweep merged work branches.

## Automated tests for the page logic
Status: To Do

All the behaviour lives in `index.html`, but nothing tests it: `.claude/pytest.ini` says the repository carries no tests yet, `recipe.ps1 test` passes on pytest's exit 5, and the commit history records only manual runs in Chrome. Done when `recipe.ps1 test` runs tests that check the 05:00 rollover in `dayKey`, the future-date clamp in `setupDayPicker`, the enjoyed-implies-done rule in `onToggle`, and the `{v, iv, ct}` shape of `questions.enc` with a 12-byte `iv`, and fails when any of them breaks.

## Recover from a stale cached Drive folder
Status: To Do

`ensureFolder` in `index.html` returns the folder id cached under `moyden.folderId` without checking it. If that folder is deleted, every send fails with a Drive error until the browser storage is cleared by hand, and if it is only trashed, new day files are written into the trashed folder, out of reach of the Drive for Desktop import described in the header comment of `index.html`. Done when a cached id that Drive reports as missing or trashed is dropped, the folder is found or created again, and the same send saves the day file into a live folder.

## Clear the auto-ticks fragment from the address
Status: To Do

`init` in `index.html` clears the fragment only when it carries `q` or `k`, so a link with only `#auto=` stays in the address bar and in browser history. `init` reads only `done` from that payload and applies it to the current day, so reloading that page or reopening it from history on a later day ticks the same items on that day's draft, which is plausibly wrong data rather than a visible error. Done when, after a link carrying only `#auto=` is opened, `location.hash` is empty and a reload applies no ticks.

## English comments and notes (R-37)
Status: To Do

`README.md` lists R-37 (#9), the lab rule that everything in the repository is written in English, as an exception, and the code is part of the reason: the header comment and the code comments in `index.html` and the `_` note in `app-config.json` are in Russian. Done when those comments and the note are in English, the text shown on the page stays Russian as the README describes, and the R-37 exception in `README.md` says what still remains.
