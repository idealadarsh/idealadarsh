# README redesign — design

Date: 2026-08-11
Repo: `idealadarsh/idealadarsh` (GitHub profile README)

## Problem

The profile README renders with most of its imagery broken. A full audit of all 49 image
URLs found:

**All four dynamic cards are dead**, and cannot be fixed by re-pointing:

| Widget | Host | Status |
| --- | --- | --- |
| Main stats card | `github-readme-stats.vercel.app` | `503 DEPLOYMENT_PAUSED` |
| Top languages card | `github-readme-stats.vercel.app` | `503 DEPLOYMENT_PAUSED` |
| GitHub trophies | `github-profile-trophy.vercel.app` | `402 Payment required / DEPLOYMENT_DISABLED` |
| Twitter card | `github-readme-twitter.gazf.vercel.app` | `400 Bad Request` |

The community mirror `github-readme-stats-sigma-five.vercel.app` returns HTTP 200 but the
body is an error SVG (`Something went wrong! … Maximum retries exceeded`), so it is not a
usable substitute.

**20 of 40 tech-stack icons are broken.** 15 share one root cause: Wikimedia now rejects
arbitrary thumbnail widths, returning `400 Use thumbnail sizes listed on
https://w.wiki/GHai`, which kills every `1200px-…` / `1024px-…` / `2367px-…` URL. The
files still exist; only the sizes are refused. The remaining 5: MySQL (`worldvectorlogo`
404), Material UI (`media.zeemly.com` DNS failure), Flask (`iconape.com` 403), Linux (a
hardcoded `camo.githubusercontent.com` URL, 403 outside its original context), Heroku
(`pbs.twimg.com` 404).

The 20 icons that still work are spread across pngegg, blogspot, ibb.co, ggpht,
brandlogos and freebiesupply — the same class of incidental host, equally likely to rot.

Two content defects: the headline still reads "Cloud Architect role at Inspektlabs" while
the body says Basis Worldwide, and five link-reference definitions are unused.

## Goals

Every image on the page resolves; icon sources consolidate onto durable CDNs; the page
reads as deliberately designed; the daily `waka-readme-stats` automation keeps working.

## Decisions

Confirmed with the repo owner:

1. **Keep the icon-table format** (48px logos with text labels), repointed at reliable
   sources — not replaced with a `skillicons.dev` grid or shields badges.
2. **Replace dead cards with widgets verified to work today**, rather than dropping cards
   or self-hosting `github-readme-stats`.
3. **Curate the stack** from 40 to 29 entries.
4. **Group the table into three labelled sections** instead of one undifferentiated block.

## Icon source

All 29 icons come from devicon via jsDelivr:

```
https://cdn.jsdelivr.net/gh/devicons/devicon/icons/<name>/<name>-<variant>.svg
```

Every one of the 29 URLs was verified to return `200 image/svg+xml`. Variant names are not
uniform and were taken from `devicon.json`, not guessed:

- `graphql` has no `original` — use `graphql-plain`.
- `firebase` — use `firebase-plain`.
- `amazonwebservices` ships **wordmark variants only**. `original-wordmark` is mostly
  `#252f3e` navy, which is nearly invisible on GitHub's dark background, so use
  `amazonwebservices-plain-wordmark` instead: it is pure AWS orange (`#f90`) and therefore
  legible on both themes with no `<picture>` needed. It letterboxes inside a 48×48 box;
  acceptable.

### Dark-theme handling

Four icons are black or near-black and disappear against GitHub's dark background
(`#0d1117`): `nextjs-original` (`<circle>` with no `fill`), `vercel-original` (bare
`<path>`), `flask-original` (`fill="#010101"`), and `bash-original` (a `#293138` block
that reads as an empty square). For these four, a `<picture>` swaps in a white icon on
dark themes — GitHub honours `prefers-color-scheme` in `<picture>`:

```html
<picture>
  <source media="(prefers-color-scheme: dark)" srcset="https://cdn.simpleicons.org/nextdotjs/white">
  <img src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/nextjs/nextjs-original.svg"
       width="48" height="48" alt="Next.js" />
</picture>
```

`cdn.simpleicons.org/<slug>/white` was verified to return SVG with an explicit
`fill="#ffffff"`. Slugs: `nextdotjs`, `vercel`, `flask`, `gnubash`. Simple Icons has **no**
AWS icon under any slug (`aws`, `amazonaws`, `amazon`, `amazonwebservices` all 404), which
is why AWS is solved with the orange devicon variant instead.

`linux-plain` (2.8 KB) also has no fill and would vanish on dark, so `linux-original`
(193 KB) is kept — Tux's white and yellow areas read on both themes. It is the single
heaviest asset on the page; a deliberate trade of bytes for visibility.

## Stack curation

**Dropped (11):** Bootstrap, Material UI, Bulma, Redux, Gatsby, Electron, WordPress,
TensorFlow, Apache, Heroku, DigitalOcean.

**Kept (29), in three tables:**

| Section | Row width | Entries |
| --- | --- | --- |
| Languages & Frameworks | 6 | TypeScript, JavaScript, Python, Kotlin, Bash, React, Next.js, Vue.js, Svelte, Tailwind, Sass |
| Backend & Data | 5 | Node.js, Flask, GraphQL, PostgreSQL, MySQL, MongoDB, Redis, Firebase |
| Cloud & DevOps | 5 | AWS, GCP, Azure, Docker, Kubernetes, Nginx, Linux, Git, Vercel, Netlify |

Row widths differ per section (6/5/5) so no section ends in an orphaned single cell.

The `<a href="#idealadarsh">` wrapper around each icon is removed. It was malformed
(`id="#idealadarsh"` and `href="##idealadarsh"`) and only ever self-linked.

## Page structure

1. **Header**, centred — typing SVG on the current `readme-typing-svg.demolab.com` domain
   (the existing `herokuapp.com` URL still responds but that platform is retired), name,
   corrected role line, and LinkedIn / X / profile-view badges. All badges use white
   `logoColor` so they stay legible whatever the pill contrast.
2. **About** — the three existing bullets, with the stale Inspektlabs headline removed. No
   invented biography.
3. **Tech Stack** — the three grouped tables above.
4. **GitHub Activity** — `github-readme-activity-graph.vercel.app`, served as a
   `<picture>` with two colour variants so the card matches the reader's theme. An HTML
   comment above it records that this is a free third-party service, that the two URLs'
   colour params must stay in sync, and how to diagnose the next failure — since it is the
   same class of host that just broke.

   `ghchart.rshah.org` was verified working and was in the approved design, but was dropped
   during implementation after screenshot review: its empty cells are hardcoded `#eeeeee`,
   which renders as a glaring white grid on dark themes, and it has no theme parameter to
   fix that. It also duplicates the contribution calendar GitHub already renders natively
   directly above the README on the profile page.
5. **Dev Metrics** — the `waka-readme-stats` block, untouched.

The five unused link-reference definitions are deleted.

## Constraint: do not disturb the Waka block

`<!--START_SECTION:waka-->` and `<!--END_SECTION:waka-->` and everything between them must
stay byte-identical, or the daily action breaks. The new file is therefore assembled by
writing the new content above the marker and concatenating the existing block extracted
verbatim with `sed`, never by retyping it.

## Verification

1. Extract every `src=` and markdown image URL from the rebuilt README and assert each
   returns `200` with an `image/*` content type — zero failures required.
2. Assert both markers are still present exactly once and the block content is unchanged
   against `git show HEAD:README.md`.
3. Render through GitHub's own pipeline (`gh api --method POST /markdown -f mode=gfm`) and
   confirm the HTML sanitizer preserves the `<picture>`/`<source>` elements and tables.
