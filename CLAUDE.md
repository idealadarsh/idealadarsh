# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

The GitHub *profile* README for `idealadarsh`. The repo name matches the username, so `README.md` is what renders at github.com/idealadarsh. There is no application code, no package manager, no build, no tests, and no linter — the entire tracked content is `README.md` and `.github/workflows/waka-readme-stats.yml`. Every task here is a content edit to one of those two files.

## The generated block in README.md — do not hand-edit

`.github/workflows/waka-readme-stats.yml` runs `anmol098/waka-readme-stats@master` on `workflow_dispatch` and on a `cron: '0 0 * * *'` schedule. It rewrites everything between `<!--START_SECTION:waka-->` and `<!--END_SECTION:waka-->` in `README.md` and commits directly to `master` as `readme-bot` with the message `Updated with Dev Metrics` — which is effectively the whole commit history.

- Anything you write inside those markers is overwritten on the next run. To change what appears there, change the action's `with:` inputs instead (currently `SHOW_PROJECTS: False`, `SHOW_LINES_OF_CODE: True`, `SHOW_LOC_CHART: False`, `LOCALE: en`).
- The action needs the repo secrets `WAKATIME_API_KEY` and `GH_TOKEN`.
- Since the bot pushes to `master` daily, pull before editing `README.md`; a local checkout goes stale within a day.

## Editing README.md

It is raw HTML inside markdown (`<div align="center">`, `<h2 align="center">`, `<table>`).
Match that style; headings inside a centered `<div>` must be HTML tags, since markdown
syntax does not render there.

**Icon sources are deliberately consolidated.** Every logo comes from devicon via jsDelivr,
with a few white variants from `cdn.simpleicons.org`. The page went from 16 incidental
image hosts to 6, because half the original icons had rotted. Do not reintroduce one-off
CDNs (wikimedia thumbnails, pngegg, ibb.co, blogspot…) — see
`docs/superpowers/specs/2026-08-11-readme-redesign-design.md` for the full audit.

- Variant names are not uniform. Check `devicon.json` rather than guessing: GraphQL is
  `graphql-plain` (no `original`), Firebase is `firebase-plain`, and AWS exists only as a
  wordmark — use `amazonwebservices-plain-wordmark`, which is orange and so legible on both
  themes.
- Wikimedia thumbnail URLs (`.../thumb/…/1200px-Foo.svg.png`) now return
  `400 Use thumbnail sizes listed on https://w.wiki/GHai`. That one policy change broke 15
  icons at once. Don't use them.

The stack is three tables — Languages & Frameworks (6 per row), Backend & Data (5), Cloud &
DevOps (5) — with 29 cells of this shape:

```html
<td align="center" width="88">
  <img src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/typescript/typescript-original.svg"
       width="48" height="48" alt="TypeScript" />
  <br /><sub><b>TypeScript</b></sub>
</td>
```

**Dark-theme rule:** a logo that is black or near-black disappears on GitHub's dark
background (`#0d1117`). Four are wrapped in `<picture>` for this reason — Next.js, Vercel,
Flask, Bash — plus the activity graph, which ships light and dark colour variants whose
params must stay in sync:

```html
<picture>
  <source media="(prefers-color-scheme: dark)" srcset="https://cdn.simpleicons.org/nextdotjs/white" />
  <img src="…/nextjs/nextjs-original.svg" width="48" height="48" alt="Next.js" />
</picture>
```

Before adding an icon, check its fill: if the SVG has no `fill` attribute it renders black
through an `<img>` tag, so it needs the `<picture>` treatment.

## Verifying a change

There is nothing to build or test, but images rot silently — so sweep them. Every URL must
return `200` with an `image/*` content type. A `200` alone is not enough: dead widget hosts
return `200 text/plain` or an SVG whose body reads `Something went wrong!`.

```bash
{ grep -oE 'src="[^"]+"' README.md    | sed 's/src="//;s/"$//'
  grep -oE 'srcset="[^"]+"' README.md | sed 's/srcset="//;s/"$//'; } | sort -u |
while IFS= read -r u; do
  printf '%s %s\n' "$(curl -s -o /dev/null -L --max-time 20 -w '%{http_code} %{content_type}' "$u")" "$u"
done
```

Note for zsh: do not read a URL into a variable named `path` in a loop — `path` is bound to
`$PATH` and assigning it destroys the shell's command lookup.

To check rendering, use GitHub's own pipeline rather than a local markdown preview, which
won't match GitHub's HTML sanitizer. Confirm the `<picture>`/`<source>` elements survive:

```bash
gh api --method POST /markdown -f mode=gfm -f text="$(cat README.md)" | grep -c '<picture>'   # expect 5
```

To eyeball the design, screenshot the rendered HTML in both themes. Note that headless
Chrome inherits the OS appearance, so on a dark-mode machine it reports
`prefers-color-scheme: dark` for *both* renders; strip the `<source>` tags to force a
truthful light-mode preview.

To exercise a workflow change:

```bash
gh workflow run "Waka Readme" && gh run watch
```

## Git

`origin` is `git@personal:idealadarsh/idealadarsh.git` — `personal` is an SSH host alias from the user's `~/.ssh/config`, not a typo for `git@github.com`. Leave it as-is.
