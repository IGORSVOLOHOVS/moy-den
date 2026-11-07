# moy-den

"Moy den" is Russian for "my day". This is a personal daily diary: one static
page that shows a questionnaire and saves the answers to its owner's Google
Drive. The page itself is in Russian. It is one person's own tool and is not
offered to anyone else.

## What is here

- `index.html` is the whole page, with its style and script inline. The only
  thing it loads from elsewhere is Google's sign-in library.
- `questions.enc` is the questionnaire, encrypted with AES-256-GCM. The list is
  deliberately not here in the clear, because the repository is public: without
  the key the file is noise. The key arrives once in a short link,
  `.../moy-den/#k=<key>`, and stays in the browser's localStorage. The page
  reads the file again at every opening, so a new version of the questionnaire
  arrives by itself. The older route, the whole list in the link's fragment as
  `#q=...`, still works.
- `app-config.json` holds the OAuth web client ID, which is not a secret: it is
  visible in the address bar at sign-in.

Answers go to Google Drive under the `drive.file` scope, so the page sees only
the files it created itself: one `YYYY-MM-DD.json` a day, in a folder the page
makes. Sending the same day again overwrites that day's file.

`questions.enc` is written by `tools/publish_diary_questions.py --push` of the
`personal-agent` project. There is no build step and no dependency. GitHub Pages
serves the page from the GitHub copy of this repository at
<https://igorsvolohovs.github.io/moy-den/>. Nothing pushes from this Gitea copy
to GitHub, so a change made here does not reach that address by itself.

## Working on it

This repository ships no program. `.\recipe.ps1` is still its whole interface:
`.\recipe.ps1 test` runs pytest and ruff over `.claude`, and `.\recipe.ps1 info`
prints the metadata. The other verbs print `nothing to <verb>`. `CLAUDE.md`
holds the rules.

## License

MIT. See [LICENSE](LICENSE).

## Lab requirements

This repository complies with [revision 4.0.0](http://127.0.0.1:3000/igors-lab/software-requirements/src/tag/v4.0.0/BEST_REQUIREMENTS.md) of the lab requirements, except R-02 (#7), R-09 (#8), R-37 (#9), R-41 (#10), R-42 (#11).
