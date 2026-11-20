# Spec

A personal daily diary ("moy den" is Russian for "my day"): one static `index.html` page, served by GitHub Pages, that shows an encrypted questionnaire and saves each day's answers as a JSON file in the owner's Google Drive.

The task list of this file mirrors Jira ticket KAN-61. Each `### 3.N` subsection
of section 3 other than 3.7 is a task that is done: the behaviour it describes
is in the code.
Each row of the table in 3.7, "Declared but not implemented", is a task that is
not done yet. The first version of this file (commit `9c94cf0`, 2026-11-17)
held the same ten tasks as `##` sections with a `Status` line. Its six Done
tasks are subsections 3.1 to 3.6, under the same titles. Its four To Do tasks
are the first four rows of the table in 3.7.

Written 2026-11-18 from commit `952f0cf` (branch `release`) and checked again
on 2026-11-20 against branch `md-61/docs/spec-md` at `9c94cf0`. That commit
adds only `SPEC.md` to `38165e7`, the tip of `test`, and every other file in it
is identical to `952f0cf` (`git diff --stat 952f0cf 9c94cf0` lists only
`SPEC.md`), so the line references below hold on both branches. FR-42 was
added on 2026-11-20 from the first version's protocol task; it sits in 3.4 so
that FR-1 to FR-41 keep their numbers. Each requirement names the place in the
code it was read from. "no test" means no
automated test exists; the repository has none
(`python -m pytest --collect-only -q -p no:cacheprovider .claude`: "no tests
collected", exit 5, run 2026-11-20).

<!-- jira-sync:start -->
Jira: KAN-61

## Tasks

This list is one ticket in Jira project KAN, [KAN-61](https://igorsvolohovs.atlassian.net/browse/KAN-61). Each line is a checkbox there: tick it in Jira and the sync ticks it here; add a checkbox there and it is appended here. Done items are the features described in section 3; open items are what the project declares but the code does not do yet.

- [x] Load the encrypted questionnaire (section 3.1) <!-- spec-load-the-encrypted-questionnaire -->
- [x] Pick the day and keep per-day drafts (section 3.2) <!-- spec-pick-the-day-and-keep-per-day-drafts -->
- [x] Fill in the day's questionnaire (section 3.3) <!-- spec-fill-in-the-day-s-questionnaire -->
- [x] Show how-to-enjoy protocols (section 3.4) <!-- spec-show-how-to-enjoy-protocols -->
- [x] Save the day to Google Drive (section 3.5) <!-- spec-save-the-day-to-google-drive -->
- [x] Repository tooling and lab gates (section 3.6) <!-- spec-repository-tooling-and-lab-gates -->
- [ ] Automated tests for the page logic (declared, not implemented) <!-- spec-automated-tests-for-the-page-logic -->
- [ ] Recover from a stale cached Drive folder (declared, not implemented) <!-- spec-recover-from-a-stale-cached-drive-folder -->
- [ ] Clear the auto-ticks fragment from the address (declared, not implemented) <!-- spec-clear-the-auto-ticks-fragment-from-the-address -->
- [ ] English comments and notes (R-37) (declared, not implemented) <!-- spec-english-comments-and-notes-r-37 -->
- [ ] Lab-requirement exceptions R-02, R-09, R-37, R-41, R-42 (declared, not implemented) <!-- spec-lab-requirement-exceptions-r-02-r-09-r-37-r-41-r-4 -->
- [ ] Tests for the scripts under `.claude` (declared, not implemented) <!-- spec-tests-for-the-scripts-under-claude -->
- [ ] Manual test list `tests/manual/tests.json` (declared, not implemented) <!-- spec-manual-test-list-tests-manual-tests-json -->
- [ ] Wiki generator `tools/wiki.py` (declared, not implemented) <!-- spec-wiki-generator-tools-wiki-py -->
- [ ] Installer steps for an `$App` (declared, not implemented) <!-- spec-installer-steps-for-an-app -->
- [ ] Section field `key` in the questionnaire (declared, not implemented) <!-- spec-section-field-key-in-the-questionnaire -->

<!-- jira-sync:end -->

## 1. Purpose and scope

The page lets its owner fill in a daily questionnaire on a phone, keeps a draft of each day's answers in the browser, and saves the day as one JSON file in the owner's Google Drive (`README.md`, `index.html:16-17`, `:24-32`, `:500-510`).

In scope:

- the page `index.html` with its inputs `questions.enc` and `app-config.json`:
  loading the questionnaire, filling in one day, per-day drafts, protocols,
  saving the day to Google Drive;
- the repository tooling that ships with it: `recipe.ps1`, the git hooks and
  Claude Code hooks under `.claude/`, and the workflows under `.gitea/workflows/`.

Out of scope (named in this repository, implemented elsewhere):

- writing `questions.enc` and making the key link:
  `tools/publish_diary_questions.py --push` of the `personal-agent` project
  (`README.md`);
- sending the key link ("её присылает бот", "the bot sends it",
  `index.html:361`) and producing `#auto=` links ("по данным часов", "from the
  watch data", `index.html:687`);
- moving the day files to a computer (Drive for Desktop) and importing them
  (`index.html:31-32`);
- the GitHub Pages hosting configuration (`README.md`).

## 2. Context

Actors:

| Actor | Role | Source |
|---|---|---|
| Owner | the only user: fills in the page, publishes the questionnaire, the only name in `$Reviewers` | `README.md`, `recipe.ps1:22`, `CLAUDE.md` |
| Claude Code sessions | work in the repository under the rules in `CLAUDE.md` and the hooks in `.claude/settings.json` | `CLAUDE.md`, `.claude/settings.json` |
| `lab-bot` | the identity the release workflow commits and tags as | `.gitea/workflows/release.yaml:38-43` |

External systems:

| System | Use | Source |
|---|---|---|
| GitHub Pages | serves `index.html`, `questions.enc`, `app-config.json` | `README.md`, `index.html:9` |
| Google Identity Services (`accounts.google.com/gsi/client`) | OAuth token client | `index.html:176`, `:527-534` |
| Google Drive API v3 (`www.googleapis.com`) | folder lookup and creation, day-file upload | `index.html:542-620` |
| Gitea with Actions, runner label `windows` | pull requests, workflows, releases, wiki | `.gitea/workflows/*.yaml` |

Runtime environment:

- The page: a browser with `fetch`, `crypto.subtle` (AES-GCM), `localStorage`,
  `TextDecoder`, `history.replaceState`, `<input type="date">` and
  `<input type="range">`; the script uses `??` (`index.html:381`). The
  comments name a phone as the device (`index.html:16-17`, `:20`), and so does
  the setup text (`index.html:363`); `index.html:74-75` says it is filled in
  with a thumb.
  The viewport and safe-area insets are set for mobile screens
  (`index.html:5`, `:55-56`, `:103`).
- The tooling: Windows; Windows PowerShell 5.1 (`.gitea/pull_request_template.md`;
  `shell: powershell` in every workflow), Python with pytest, ruff `==0.16.1`
  (`.claude/ruff.toml`), git, optional Git LFS (`.claude/hooks/git/pre-push`).

## 3. Functional requirements

### 3.1 Load the encrypted questionnaire

Code: `readFragment` (`index.html:245-254`), `fetchQuestions`
(`index.html:268-285`) and `init` (`index.html:642-697`).

- **FR-1.** The system shall read the address fragment as `&`-separated
  `key=value` pairs and ignore pairs with an empty key or value.
  Source: `index.html:245-254`. Verification: no test.
- **FR-2.** When the fragment carries `q`, the system shall decode it as
  base64url UTF-8 JSON, use it as the questionnaire and store it in
  localStorage under `moyden.questions`; if decoding fails it shall show
  "не разобрал список вопросов из ссылки" ("could not parse the question list
  from the link"). Source: `index.html:256-263`, `:649-654`.
  Verification: no test.
- **FR-3.** When the fragment carries `q` or `k`, the system shall replace the
  address with the path alone, so neither stays in browser history.
  Source: `index.html:655-656`. Verification: no test.
- **FR-4.** Without `q`, the system shall start from the questionnaire saved
  under `moyden.questions`. Source: `index.html:657-659`.
  Verification: no test.
- **FR-5.** When a key is known (from `#k=`, else from `moyden.key`) and the
  fragment has no `q`, the system shall fetch `questions.enc` with
  `cache: "no-store"`, and decrypt `ct` with AES-GCM using `iv` and the key.
  It shall then use the result as the questionnaire and store it under
  `moyden.questions`.
  Source: `index.html:268-285`, `:668-673`. Verification: no test.
- **FR-6.** The system shall store a key from `#k=` under `moyden.key` when
  decryption succeeds, or when the failure is not a key rejection. It shall
  not store a key that the file rejects.
  Source: `index.html:674`, `:676`. Verification: no test.
- **FR-7.** The system shall name the failure of FR-5 in the status line.
  The reasons are "нет сети" (no network), "файла вопросов нет на сайте
  (HTTP <code>)" (no question file on the site) and "ключ из ссылки не
  подходит к файлу вопросов" (the key from the link does not fit the question
  file). Any other error, such as a key the browser cannot import
  (`index.html:277`, outside the `try`), shows its own text and is not a key
  rejection for FR-6. With a saved list the line reads "список не обновил:
  <reason> — показываю сохранённый v<version>" (list not updated, showing the
  saved version); without one, "не загрузил список вопросов: <reason>" (did
  not load the question list).
  Source: `index.html:270-283`, `:677-679`. Verification: no test.
- **FR-8.** Without any questionnaire, the system shall show the expected link
  form `<origin><path>#k=…` and leave the send button disabled.
  Source: `index.html:172`, `:359-366`. Verification: no test.

### 3.2 Pick the day and keep per-day drafts

Code: `dayKey` (`index.html:201-206`), `targetDay` (`index.html:199`),
`setupDayPicker` (`index.html:211-234`), `saveDraft` and `loadDraft`
(`index.html:500-510`).

- **FR-9.** The system shall date the current day by local time, counting
  00:00-04:59 as the previous day. Source: `index.html:201-206`.
  Verification: no test.
- **FR-10.** The system shall offer a date picker limited to today (by FR-9),
  reset a later date to today, provide a "сегодня" (today) button, and show
  "Заполняешь прошедший день" (you are filling in a past day) when the chosen
  day is not today. Source: `index.html:161-166`, `:211-234`.
  Verification: no test (manual check described in commit `a1a0597`).
- **FR-11.** On a change of day, the system shall clear the in-memory answers,
  load that day's draft and render again. Source: `index.html:222-226`.
  Verification: no test.
- **FR-12.** The system shall save the answers (`done`, `joy`, `diary`,
  `values`) to localStorage under `moyden.draft.<YYYY-MM-DD>` after every
  toggle, slider move, slider reset and diary keystroke.
  Source: `index.html:399`, `:405`, `:459`, `:483`, `:500-502`.
  Verification: no test.
- **FR-13.** When the fragment carries `auto`, the system shall decode it as
  base64url JSON and set `done` for each id in its `done` array whose value
  in the draft is still undefined. Source: `index.html:685-693`.
  Verification: no test.

### 3.3 Fill in the day's questionnaire

Code: `render` (`index.html:355-466`), `onToggle` (`index.html:472-484`) and
`paint` (`index.html:486-497`).

- **FR-14.** For a section without `kind`, the system shall render one row per
  item with a ✓ (done) and a ♥ (enjoyed) button, headed by the section's
  `done` and `enjoyed` labels. Source: `index.html:423-440`.
  Verification: no test.
- **FR-15.** The system shall set done when enjoyed is set, and clear enjoyed
  when done is cleared. Source: `index.html:476-481`. Verification: no test.
- **FR-16.** The system shall colour the done button red in sections marked
  `negative`. Source: `index.html:85`, `:370`. Verification: no test.
- **FR-17.** For `kind: "choice"`, the system shall render the items as a row
  of buttons whose state is kept in `done`. Source: `index.html:410-421`.
  Verification: no test.
- **FR-18.** For `kind: "hours"`, the system shall render a slider for the
  section's first item with `min`, `max`, `step` defaulting to 0, 16 and 1.
  It shall show "—" until the slider is moved, record the value only after a
  move, and remove it on "не отвечать" (don't answer).
  Source: `index.html:378-408`. Verification: no test (manual check described
  in commit `a84bfb3`).
- **FR-19.** The system shall provide a free-text field "Дневник дня" (diary of
  the day) after the sections. Source: `index.html:453-461`.
  Verification: no test.
- **FR-20.** The system shall show in the header the target day, `опросник
  v<version>` (questionnaire version) and `отмечено N из M` (N of M marked).
  N is the number of ids set in `done` and M the number of items in all
  sections. Source: `index.html:486-497`. Verification: no test.

### 3.4 Show how-to-enjoy protocols

Code: `allProtocols` (`index.html:292`), `protocolsFor` (`index.html:299-305`),
`openGuide` (`index.html:307-319`), `closeGuide` (`index.html:321`) and
`openAllGuides` (`index.html:327-347`).

- **FR-21.** The system shall select an item's protocols as those whose
  `item` equals the item id. If there are none, it shall select those with
  slug `sad_refusal` in a `negative` section and `base` elsewhere.
  Source: `index.html:299-305`. Verification: no test (manual check described
  in commit `68b8e65`).
- **FR-22.** For a row with at least one protocol, the system shall show a `?`
  mark and, on a tap on the label, open a sheet with each protocol's group,
  title, minutes and steps. Source: `index.html:307-319`, `:349-351`,
  `:431-434`. Verification: no test.
- **FR-23.** The system shall offer a "Все протоколы — список" (all protocols,
  as a list) button that lists every protocol grouped by `group`.
  Source: `index.html:327-347`, `:443-451`. Verification: no test.
- **FR-24.** The system shall close the sheet on "Закрыть" (close) or on a tap
  on the backdrop. Source: `index.html:321`, `:352`. Verification: no test.
- **FR-42.** The system shall take the protocols only from the `protocols`
  array of the loaded questionnaire and treat a missing array as empty. Then
  no row gets the `?` mark or opens a sheet, and the "Все протоколы — список"
  button is still shown but opens nothing.
  Source: `index.html:292`, `:309`, `:329`, `:432-434`, `:443-451`.
  Verification: no test.

### 3.5 Save the day to Google Drive

Code: `getToken` (`index.html:516-540`), `drive` (`index.html:542-548`),
`ensureFolder` (`index.html:550-567`), `upload` (`index.html:569-621`) and the
send handler (`index.html:623-639`).

- **FR-25.** On "Отправить" (send), the system shall request an access token
  from Google Identity Services. The request uses the `clientId` from
  `app-config.json` and scope `https://www.googleapis.com/auth/drive.file`,
  with `prompt: "consent"` while the page holds no token and `prompt: ""`
  after that. Source: `index.html:186`, `:516-540`, `:643-646`.
  Verification: no test.
- **FR-26.** The system shall stop with "не задан clientId в app-config.json"
  (no clientId in app-config.json) when the ID is missing, "библиотека Google
  не загрузилась" (the Google library did not load) when the library is
  absent, and "доступ не выдан" (access not granted) when no token comes back.
  Source: `index.html:518-532`. Verification: no test.
- **FR-27.** The system shall use the Drive folder id cached under
  `moyden.folderId`. Without one, it shall search for a non-trashed folder
  named `Мой день — записи`, create it if none is found, and cache the id.
  Source: `index.html:185`, `:550-567`. Verification: no test.
- **FR-28.** The system shall look for a non-trashed `<day>.json` in that
  folder. It shall update that file with a multipart PATCH if found, and
  otherwise create it with a multipart POST.
  Source: `index.html:602-620`. Verification: no test.
- **FR-29.** The system shall write the fields `date`, `submitted_at`,
  `submitted_local`, `tz_offset_min`, `questions_version`, `done`, `enjoyed`,
  `asked`, `diary`, `values` as JSON indented by two spaces (see section 5).
  Source: `index.html:579-600`, `:612`. Verification: no test (manual check
  described in commit `6668b58`).
- **FR-30.** The system shall report "сохранено: <file>" (saved) or
  "перезаписано: <file>" (overwritten) on success. On failure it shall show
  the error text, for Drive as `Drive <status>: <first 200 characters of the
  body>`. Source: `index.html:546`, `:631-635`. Verification: no test.
- **FR-31.** The system shall disable the send button while a send runs and
  enable it afterwards, whatever the outcome. Source: `index.html:624-638`.
  Verification: no test.

### 3.6 Repository tooling and lab gates

Code: `recipe.ps1`, the hooks under `.claude/hooks/`, `.claude/settings.json`
and the workflows under `.gitea/workflows/`.

- **FR-32.** `recipe.ps1 test` shall set `core.hooksPath` to
  `.claude/hooks/git` and run `python -m pytest` over `.claude`, accepting
  exit 0 or 5. It shall then run `python -m ruff check` and
  `python -m ruff format` (in place) over the same folder, exiting 1 at the
  first failure. Source: `recipe.ps1:41-55`. Verification: no test (CI runs it).
- **FR-33.** `recipe.ps1 info` shall print `Name`, `Code`, `Version`,
  `Summary`. Source: `recipe.ps1:56-62`. Verification: no test; run on
  2026-11-20 on this branch, exit 0.
- **FR-34.** `recipe.ps1 clean` shall remove every `__pycache__`,
  `.pytest_cache` and `.ruff_cache` folder under the root.
  Source: `recipe.ps1:63-68`. Verification: no test.
- **FR-35.** `recipe.ps1 build|install|uninstall|run|publish` shall print
  `nothing to <verb>` and exit 0. Source: `recipe.ps1:69-73`.
  Verification: no test; `run` executed on 2026-11-20 on this branch.
- **FR-36.** `recipe.ps1` shall load `NAME=value` lines from `.claude/.env`
  into the environment, skipping names that are already set.
  Source: `recipe.ps1:28-38`. Verification: no test.
- **FR-37.** The `commit-msg` hook shall refuse a message that starts with a
  UTF-8 BOM, a subject outside `<type>(<scope>): <text>` (types and scopes
  read from `recipe.ps1`, case-sensitive), or a message without a
  `Signed-off-by: Name <address>` trailer. It shall pass a `Merge ...` subject
  while a merge is in progress. Source: `.claude/hooks/git/commit-msg.ps1`.
  Verification: no test.
- **FR-38.** The `pre-push` hook shall refuse a branch outside
  `md-<issue>/<type>/<slug>` (`test` and `from-*` are exempt) and any deletion
  of `release`. It shall refuse a push to `release` unless a pushed commit
  carries the override phrase from `recipe.ps1` alone on a line and a
  `Why: ` line. It shall pass the first push that creates `release`.
  Source: `.claude/hooks/git/pre-push.ps1`. Verification: no test.
- **FR-39.** The Claude Code gate, wired as the PreToolUse hook for
  `Bash|PowerShell` in `.claude/settings.json:10-12`, shall deny, for Bash and
  PowerShell calls: a recursive delete whose paths are not all under
  `$env:TEMP`; `git push` with `--force`, `-f`, `--mirror` or a `+ref`
  (allowing `--force-with-lease`); and, on `test` or `release`, `reset --hard`
  (except to `origin/<branch>` or `@{u}`), `clean -f`, `commit --amend`,
  `rebase` and `filter-branch`. It shall also deny a PreToolUse payload it
  cannot read. Source: `.claude/hooks/gate.ps1`. Verification: no test.
- **FR-40.** The hooks shall append one JSON line per event to
  `.claude/logs/ai.jsonl` (UTF-8 without BOM, `detail` cut to 200
  characters). At SessionStart they shall move the file aside when it exceeds
  10 MB, set `core.hooksPath`, and report the branch and issue number to the
  session. Source: `.claude/hooks/log.ps1`,
  `.claude/hooks/session-start-stop.ps1`. Verification: no test.
- **FR-41.** The workflows shall do the following:
  - `propose`, on pull requests into `test`: judge the branch name and the
    commit subjects and sign-offs, run `recipe.ps1 test` and
    `git diff --exit-code`, and request review and comment on open and reopen.
  - `release-gate`, on pull requests into `release`: accept only `test` as the
    source, run the tests, and hold a manual round.
  - `release`, on push to `release`: when tag `v<Version>` does not exist
    yet, write the release notes, create release `v<Version>` with an SBOM
    and close the shipped issues. On every run, record overrides and bring
    `test` level with `release`. Write the changelog only for a project that
    names an `$App` or `$Provides`, which this one does not.
  - `test`, on push to `test`, on cron `0 3 * * *` and on demand: check runner
    health (free space on `C:`, and `python`, `ruff`, `pytest` and `mypy`
    answering `--version`, `test.yaml:32-45`) and run the tests.
  - `sweep`, on cron `0 4 * * 1` (Mondays): delete work branches merged into `test`,
    nudge pull requests idle for 30 days and close those idle for 60.

  Source: `.gitea/workflows/propose.yaml`, `release-gate.yaml`,
  `release.yaml`, `test.yaml`, `sweep.yaml`. Verification: no test.

### 3.7 Declared but not implemented

The first four rows are the To Do tasks of the first version of this file. The
other rows are what the repository names or calls but does not have.

| Item | What is declared and what is missing | Source |
|---|---|---|
| Automated tests for the page logic | Nothing tests the behaviour in `index.html`, since `recipe.ps1 test` accepts pytest's exit 5 (`recipe.ps1:47-48`); done when `recipe.ps1 test` runs tests of the 05:00 rollover in `dayKey`, the future-date clamp in `setupDayPicker`, the enjoyed-implies-done rule in `onToggle` and the `{v, iv, ct}` shape of `questions.enc` with a 12-byte `iv`, and fails when any of them breaks. | Jira KAN-61 (task proposed 2026-11-17) |
| Recover from a stale cached Drive folder | `ensureFolder` returns the folder id cached under `moyden.folderId` without asking Drive about it (`index.html:551-552`), so a deleted or trashed folder is never replaced; done when a cached id that Drive reports as missing or trashed is dropped, the folder is found or created again, and the same send saves the day file into a live folder. | Jira KAN-61 (task proposed 2026-11-17) |
| Clear the auto-ticks fragment from the address | `init` clears the fragment only when it carries `q` or `k` (`index.html:656`), so a link with only `#auto=` stays in the address and history and, reloaded or reopened on a later day, ticks the same items in that day's draft (`index.html:685-693`); done when `location.hash` is empty after such a link is opened and a reload applies no ticks. | Jira KAN-61 (task proposed 2026-11-17) |
| English comments and notes (R-37) | The header comment and the code comments of `index.html` and the `_` note of `app-config.json` are in Russian (`index.html:8-33`, `app-config.json:2`); done when they are in English, the text shown on the page stays Russian, and the R-37 exception in `README.md` says what still remains. | Jira KAN-61 (task proposed 2026-11-17) |
| Lab-requirement exceptions R-02, R-09, R-37, R-41, R-42 | `README.md` names five exceptions to revision 4.0.0 of the lab requirements: R-02 (#7), R-09 (#8), R-37 (#9), R-41 (#10), R-42 (#11). The text of R-37, R-41 and R-42 is not in this repository; `CLAUDE.md` describes only R-02 and R-09. | `README.md:45`, `CLAUDE.md` |
| Tests for the scripts under `.claude` | `.claude/pytest.ini` holds a pytest section for the scripts the repository carries and says "None yet". | `.claude/pytest.ini:2` |
| Manual test list `tests/manual/tests.json` | `release-gate` reads this file; it does not exist, so the manual round passes as "no manual tests defined". | `.gitea/workflows/release-gate.yaml:111`, `:139-150` |
| Wiki generator `tools/wiki.py` | `test.yaml` calls it to regenerate the wiki, and the file does not exist. The step runs only when the recipe names an `$App` or `$Provides`, which it does not. | `.gitea/workflows/test.yaml:73-85`, `recipe.ps1:16-17` |
| Installer steps for an `$App` | The installer steps in `release-gate`, `release` and `test` serve an `$App`, which this recipe does not name. The build step runs regardless and `recipe.ps1 build` prints `nothing to build`; the install and uninstall steps of `release-gate` and `test` run only for an `$App`. | `.gitea/workflows/release-gate.yaml:55-74`, `release.yaml:72-74`, `test.yaml:59-72`, `recipe.ps1:16`, `:69-73` |
| Section field `key` in the questionnaire | The comment on the questionnaire shape names a section field `key`; no code reads it. | `index.html:189` |

## 4. Interfaces

### 4.1 Address fragment

| Parameter | Value | Effect | Source |
|---|---|---|---|
| `k` | base64url raw AES key | decrypts `questions.enc`; stored as `moyden.key` (FR-6) | `index.html:668-676` |
| `q` | base64url UTF-8 JSON questionnaire | used and stored as `moyden.questions`; disables the fetch of `questions.enc` for that opening | `index.html:649-654`, `:670` |
| `auto` | base64url UTF-8 JSON `{"done":[id,...]}` | ticks untouched items of the current day | `index.html:685-693` |

### 4.2 Files served with the page

| File | Format | Source |
|---|---|---|
| `questions.enc` | JSON `{"v":1,"iv":"<base64url, 12 bytes in the current file>","ct":"<base64url ciphertext with GCM tag>"}` | `index.html:265-285`; file measured 2026-11-20 on this branch: `v` 1, 12-byte `iv`, 12638-byte `ct` |
| `app-config.json` | JSON `{"_": "<note>", "clientId": "<OAuth web client ID>"}` | `app-config.json`, `index.html:188` |

### 4.3 Google APIs

| Call | Method and path | Source |
|---|---|---|
| token | `google.accounts.oauth2.initTokenClient({client_id, scope, callback})`, `requestAccessToken({prompt})` | `index.html:527-538` |
| find folder | `GET drive/v3/files?q=name='Мой день — записи' and mimeType='application/vnd.google-apps.folder' and trashed=false&fields=files(id)` | `index.html:553-555` |
| create folder | `POST drive/v3/files?fields=id`, JSON `{name, mimeType}` | `index.html:558-562` |
| find day file | `GET drive/v3/files?q=name='<day>.json' and '<folderId>' in parents and trashed=false&fields=files(id)` | `index.html:602-603` |
| write day file | `POST upload/drive/v3/files?uploadType=multipart&fields=id` or `PATCH upload/drive/v3/files/<id>?uploadType=multipart&fields=id`, `multipart/related` with metadata and JSON body | `index.html:606-619` |

All Drive calls send `Authorization: Bearer <token>` (`index.html:543-545`).

### 4.4 `recipe.ps1`

| Verb | Effect | Exit | Source |
|---|---|---|---|
| `test` | FR-32 | 0 or 1 | `recipe.ps1:41-55` |
| `info` | FR-33 | 0 | `recipe.ps1:56-62` |
| `clean` | FR-34 | 0 | `recipe.ps1:63-68` |
| `build`, `install`, `uninstall`, `run`, `publish` | print `nothing to <verb>` | 0 | `recipe.ps1:69-73` |

Any other verb is refused by `ValidateSet` (`recipe.ps1:3`); checked on
2026-11-20 with the verb `bogus`: parameter validation error, exit 1.

### 4.5 CI triggers

| Workflow | Trigger | Job timeout | Source |
|---|---|---|---|
| `propose` | pull request opened, reopened, synchronized into `test` | 60 min | `.gitea/workflows/propose.yaml:2-15` |
| `release-gate` | pull request opened, reopened, synchronized into `release` | 60 min (`checks`), 170 min (`manual-gate`) | `.gitea/workflows/release-gate.yaml:2-19`, `:87-91` |
| `release` | push to `release` | 60 min | `.gitea/workflows/release.yaml:2-14` |
| `test` | push to `test`, cron `0 3 * * *`, `workflow_dispatch` | 60 min | `.gitea/workflows/test.yaml:2-17` |
| `sweep` | cron `0 4 * * 1` | 15 min | `.gitea/workflows/sweep.yaml:2-14` |

## 5. Data

Questionnaire (decrypted `questions.enc` or `#q=`), as the code reads it:

| Field | Type | Use | Source |
|---|---|---|---|
| `version` | any | shown in the header, written as `questions_version` | `index.html:496`, `:591` |
| `sections[]` | array | rendered in order | `index.html:368` |
| `sections[].title`, `.done`, `.enjoyed` | string | heading and column labels | `index.html:423-425` |
| `sections[].negative` | boolean | red done buttons, `sad_refusal` fallback | `index.html:370`, `:431` |
| `sections[].kind` | `"hours"`, `"choice"` or absent | rendering mode | `index.html:378`, `:410` |
| `sections[].min`, `.max`, `.step` | number | slider range, defaults 0, 16, 1 | `index.html:381` |
| `sections[].items[]` | `{id, text}` | `id` is numeric: the code converts it with `Number()` | `index.html:351`, `:592-593` |
| `protocols[]` | `{item, slug, group, title, minutes, steps[]}` | protocol sheets; absent means no protocols (FR-42) | `index.html:292-347` |

Browser storage (localStorage):

| Key | Content | Source |
|---|---|---|
| `moyden.questions` | the last questionnaire, JSON | `index.html:180`, `:652`, `:673` |
| `moyden.key` | the last accepted key, base64url | `index.html:183`, `:674`, `:676` |
| `moyden.folderId` | Drive folder id | `index.html:182`, `:551`, `:565` |
| `moyden.draft.<YYYY-MM-DD>` | `{"done":{id:bool},"joy":{id:bool},"diary":"...","values":{id:number}}` | `index.html:181`, `:190`, `:500-510` |

No code removes any of these keys.

Day file `<YYYY-MM-DD>.json` in the Drive folder `Мой день — записи`, one per day,
overwritten on a repeat send:

| Field | Type | Meaning | Source |
|---|---|---|---|
| `date` | `YYYY-MM-DD` | the day described (FR-9, FR-10) | `index.html:585` |
| `submitted_at` | ISO 8601 UTC (`toISOString`) | moment of sending | `index.html:586` |
| `submitted_local` | `YYYY-MM-DDTHH:MM:SS`, no offset | local wall-clock time of sending | `index.html:581-582`, `:587` |
| `tz_offset_min` | integer | minutes east of UTC (`-getTimezoneOffset()`) | `index.html:590` |
| `questions_version` | as in the questionnaire, or `null` | questionnaire version | `index.html:591` |
| `done`, `enjoyed` | array of numbers | ids marked | `index.html:592-593` |
| `asked` | array of ids | every item id of every section | `index.html:594` |
| `diary` | string | free text | `index.html:595` |
| `values` | object id → number | slider values that were set | `index.html:599` |

Commit `a1a0597` states that `submitted_at` and `submitted_local` stay the
real moment of sending when a past day is chosen. The code matches: both come
from `new Date()` at send time (`index.html:579-587`).

Repository data: `.claude/logs/ai.jsonl` (FR-40), gitignored; `.claude/.env`,
gitignored (`.gitignore`).

## 6. Non-functional requirements

- **NFR-1.** No build step and no runtime dependency besides Google's sign-in
  script; style and script are inline. Source: `README.md`, `index.html:176`,
  `recipe.ps1:16-17`.
- **NFR-2.** The questionnaire is kept in the repository only encrypted with
  AES-GCM. The key is never in the repository. Source: `README.md`,
  `index.html:11-17`, `CLAUDE.md` rule 4. `README.md` says AES-256-GCM. The
  key length is set by the key in the link, so 256 bits is not checkable
  from the repository.
- **NFR-3.** The questionnaire, key and auto-ticks travel in the address
  fragment, not the query, so they are not sent to the server.
  Source: `index.html:21-22`, `:242-244`.
- **NFR-4.** `q` and `k` are removed from the address and history after reading
  (FR-3). Source: `index.html:655-656`.
- **NFR-5.** Drive access is limited to the `drive.file` scope.
  Source: `index.html:186`, `README.md`.
- **NFR-6.** `questions.enc` and `app-config.json` are fetched with
  `cache: "no-store"`. Source: `index.html:271`, `:644`.
- **NFR-7.** Search engines are asked not to index or follow:
  `<meta name="robots" content="noindex, nofollow">`. Source: `index.html:6`.
- **NFR-8.** Questionnaire text, titles, groups and steps are HTML-escaped
  before insertion. Source: `index.html:468-470`. See section 9 for the fields
  that are not.
- **NFR-9.** Toggle buttons are 3.1rem × 2.6rem for thumb use.
  Source: `index.html:74-81`.
- **NFR-10.** Drive error bodies are cut to 200 characters in the status line.
  Source: `index.html:546`.
- **NFR-11.** Keys are never committed: `.claude/.env` and `.claude/logs/` are
  gitignored. Source: `.gitignore`, `CLAUDE.md` rules 4 and 5.
- **NFR-12.** Line endings: LF by default, CRLF and UTF-8 working-tree encoding
  for `*.ps1`, Git LFS for `*.pdf`, `*.png`, `*.jpg`, `*.zip`, `*.7z`.
  Source: `.gitattributes`.
- **NFR-13.** Hook log writes retry up to 20 times with 10·n ms waits; the log
  rotates above 10 MB. Source: `.claude/hooks/log.ps1`,
  `.claude/hooks/session-start-stop.ps1`.
- **NFR-14.** CI: job timeouts as in 4.5. The manual gate waits 150 minutes
  (`release-gate.yaml:101`). The `test` job fails below 5 GB free on `C:` and
  warns below 20 GB (`test.yaml:32-36`).
- **NFR-15.** ruff is pinned to `==0.16.1`; another version refuses to run.
  Source: `.claude/ruff.toml`.

## 7. Constraints and dependencies

The repository has no lock file, `package.json`, `pyproject.toml` or
requirements file (`git ls-files`).

| Dependency | Version | Source |
|---|---|---|
| Google Identity Services client | unversioned URL `https://accounts.google.com/gsi/client` | `index.html:176` |
| Google Drive API | v3 | `index.html:555`, `:614` |
| Browser APIs | `fetch`, `crypto.subtle`, `localStorage`, `TextDecoder`, `history.replaceState`; ES2020 `??` | `index.html` |
| Windows PowerShell | 5.1 | `.gitea/pull_request_template.md` |
| Python, pytest | not pinned | `recipe.ps1:46`, `.claude/pytest.ini` |
| ruff | `==0.16.1` | `.claude/ruff.toml` |
| mypy | not pinned; asked for only by the runner health check of `test` | `.gitea/workflows/test.yaml:38` |
| Git LFS | optional, used by `pre-push` when present | `.claude/hooks/git/pre-push` |
| `actions/checkout` | `v4` | `.gitea/workflows/*.yaml` |
| `actions/upload-artifact` | `v3` | `.gitea/workflows/release-gate.yaml:442` |
| Gitea secret `LAB_BOT_TOKEN` | - | `.gitea/workflows/*.yaml` |

Licence: MIT (`LICENSE`).

## 8. Acceptance

Automated:

- `.\recipe.ps1 test` exits 0 and leaves `git diff --exit-code` clean
  (`CLAUDE.md` rule 3; `propose.yaml:144-152`). Today this checks only the
  tooling under `.claude` (no tests, ruff check and format). It checks nothing
  in `index.html`.
- `propose` and `release-gate` are green on the pull request (`CLAUDE.md`).

Manual, in a browser on the published page (none of these is automated):

1. Open the plain address with no stored key and no stored list: the setup
   text with the `#k=…` form is shown and "Отправить" is disabled (FR-8).
2. Open `<address>#k=<key>`: the list appears, the address loses its fragment,
   and the header shows `опросник v<version>` (FR-3, FR-5, FR-20).
3. Open with a wrong key of a valid AES length: the status names the key,
   and the previously stored key still works on the next plain opening
   (FR-6, FR-7).
4. Tap ♥ on an unmarked row: ✓ is set as well. Clear ✓: ♥ is cleared too
   (FR-15).
5. Choose yesterday in "Сутки": the past-day notice shows and the marks are
   that day's. A future date is reset to today (FR-10, FR-11).
6. Move the slider, then "не отвечать": the value shows "—" again (FR-18).
7. Tap a label with `?`: the sheet opens. "Все протоколы — список" shows all
   groups (FR-22, FR-23).
8. "Отправить" twice for one day: first "сохранено: <day>.json", then
   "перезаписано: <day>.json". The Drive file has the fields of section 5
   (FR-28 to FR-30).

## 9. Open questions

- Which branch GitHub Pages serves is not recorded in the repository.
  `README.md` says nothing pushes from the Gitea copy to GitHub, but the GitHub
  copy carries `main`, `release`, `test` and `md-61/docs/spec-md`, the last
  three with the lab history, and `origin/HEAD` in a clone of it points at
  `release` (`git branch -a`). How these reach GitHub is not described.
- The tool that keeps the task list of this file and Jira KAN-61 in step is not
  in this repository (`git ls-files`).
- The key length (README: AES-256) and the questionnaire schema are fixed by
  `tools/publish_diary_questions.py` in `personal-agent`, which is not here.
  The types of `version` and of item ids beyond "numeric" are not fixed by
  this code.
- Which program builds `#auto=` links, from which data, and how often is not
  in this repository (`index.html:687`).
- Commit `135233c` describes the OAuth client as shared with another site.
  `app-config.json` has not changed since that commit
  (`git log -- app-config.json`), so the page still uses that client; whether
  the other site still does cannot be checked here. Under `drive.file` two
  sites on one client see each other's files.
- Supported browsers and devices are not stated beyond the comments naming a
  phone.
- Retention or backup of the day files in Drive is not addressed.
- Whether a Gitea runner serves this repository, and whether branch protection
  has reached stage 2, is not recorded (`CLAUDE.md`).
- Behaviours that are present in the code but were not examined in a browser,
  and may be unintended:
  - With no date picked, the day is recomputed at each draft save and send
    (`index.html:198-199`). A page left open across 05:00 then files the
    answers under the new day.
  - Item `id`, protocol `minutes` and a section's `min`/`max`/`step` reach
    `innerHTML` unescaped (`index.html:314`, `:341`, `:389`, `:415`, `:433`,
    `:435`). Any `#q=` link supplies the questionnaire (`index.html:649-652`).
  - `#q=` and `#k=` in one link: the key is neither checked nor stored, since
    the fetch is skipped when `q` is present (`index.html:670`).
  - A `#k=` key the browser cannot import (wrong length) fails before
    decryption, is not flagged as rejected, and is stored over the previous
    key (`index.html:277`, `:676`).
- The text of R-37 is not in this repository. The task "English comments and
  notes (R-37)" takes it to be the rule that everything is written in
  English; `CLAUDE.md` rule 6 states that rule under §5.
