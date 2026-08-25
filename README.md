# README to Landing Page

[![CI](https://img.shields.io/github/actions/workflow/status/bmoler68/README-to-landing-page/ci.yml?branch=main&label=CI)](https://github.com/bmoler68/README-to-landing-page/actions/workflows/ci.yml)
[![Release](https://img.shields.io/github/v/release/bmoler68/README-to-landing-page)](https://github.com/bmoler68/README-to-landing-page/releases/latest)
[![License](https://img.shields.io/github/license/bmoler68/README-to-landing-page)](https://github.com/bmoler68/README-to-landing-page/blob/main/LICENSE)
[![Last commit](https://img.shields.io/github/last-commit/bmoler68/README-to-landing-page)](https://github.com/bmoler68/README-to-landing-page/commits/main)
[![Issues](https://img.shields.io/github/issues/bmoler68/README-to-landing-page)](https://github.com/bmoler68/README-to-landing-page/issues)
![Language](https://img.shields.io/github/languages/top/bmoler68/README-to-landing-page)

**README to Landing Page** is a GitHub Action that converts a repository `README.md` into a static landing page and publishes it to GitHub Pages. This file is a **conversion fixture**: it describes the real project at [bmoler68/README-to-landing-page](https://github.com/bmoler68/README-to-landing-page.git) while exercising headings, tables, code, lists, and sanitizer edge cases.

> Add a workflow in **your** repository. No local CLI or extra tooling is required.

The badges above are live [Shields.io](https://img.shields.io) queries against that repo (CI on `ci.yml`, latest GitHub release, license, last commit, open issues, top language). They are not hardcoded “passing” stickers.

![Generated landing page (fixture URL)](https://cdn.readme-to-landing-page.example/images/hero.png "Invented URL so the converter still sees an image without a real photo host")

The image URL is **invented**. Only badge images load from a real host.

## Table of contents

- [What it does](#what-it-does)
- [At a glance](#at-a-glance)
- [How it works](#how-it-works)
  - [Parse](#parse)
  - [Generate](#generate)
  - [Publish](#publish)
- [Add the workflow](#add-the-workflow)
- [Configure with `with:`](#configure-with-with)
- [Enable GitHub Pages](#enable-github-pages)
- [What gets published](#what-gets-published)
- [Runtime and tests](#runtime-and-tests)
- [Markdown fixture gallery](#markdown-fixture-gallery)
- [Sanitizer fixtures](#sanitizer-fixtures)
- [This test repository](#this-test-repository)
- [License](#license)

---

## What it does

Maintainers already write a README. The action turns that markdown into `index.html` + `styles.css`, then pushes the site to a **`gh-pages`** branch so GitHub Pages can serve it.

Use it when you want a public landing page that **stays in sync** with the README. Skip it if you already maintain a custom Pages site by hand.

The action is a **Docker** container (`runs.using: docker` in `action.yml`). CI on the source repo installs Node **20**, runs `npm test`, and verifies `docker build`.

## At a glance

| Piece | What it is | Source of truth |
| :--- | :--- | :---: |
| Action ref | `bmoler68/readme-to-landing-page@v1` | [Releases](https://github.com/bmoler68/README-to-landing-page/releases) |
| Package | `readme-to-landing-page` `1.0.0` | `package.json` |
| Runtime | Docker image from repo `Dockerfile` | `action.yml` |
| Markdown | [marked](https://www.npmjs.com/package/marked) `^15` | `package.json` |
| HTML filter | [sanitize-html](https://www.npmjs.com/package/sanitize-html) `^2.14` | `package.json` |
| Tests | `node --test test/*.test.js` | `package.json` scripts |
| License | MIT | repo `LICENSE` |

Inline emphasis check: **bold**, *italic*, ***bold italic***, ~~strikethrough~~, `inline code`, a [named link](https://github.com/bmoler68/README-to-landing-page), an [anchor](#add-the-workflow), and `code **inside** ticks`.

HTML extras the sanitizer allows: H<sub>2</sub>O in copy, 10<sup>3</sup> files, <ins>added TOC</ins>, <del>hand-copied HTML</del>.

## How it works

```
README.md  →  parse (marked lexer + GFM)
           →  generate (template + TOC + badges)
           →  publish (git push to gh-pages)
           →  GitHub Pages serves / (root)
```

### Parse

`src/parser.js` lexes GFM with `breaks: true`. It:

- Takes the first **H1** as the page title
- Treats the following badge paragraph (`![...](...)` / linked badges) as the hero badge row
- Walks headings, paragraphs, lists, tables, code, blockquotes, HTML, and `hr`

Heading ids are slugified (punctuation stripped, spaces to `-`).

### Generate

`src/generator.js` fills `templates/default.html`. Optional TOC is built from **H2 and H3 only**. Content is sanitized before write.

### Publish

`src/publisher.js` clones or creates `gh-pages`, copies the generated site (not the whole source repo), and pushes with `GITHUB_TOKEN`.

![Pipeline diagram (fixture URL)](https://cdn.readme-to-landing-page.example/images/pipeline.png "Fictional diagram asset")

[![Linked thumbnail fixture](https://cdn.readme-to-landing-page.example/images/thumb.png)](https://github.com/bmoler68/README-to-landing-page)

HTML `<img>` with width/height (allowed tags; fictional `src`):

<img src="https://cdn.readme-to-landing-page.example/images/toc.png" alt="Fictional TOC screenshot" width="480" height="240" title="HTML img tag">

## Add the workflow

In the **target** repository (the project whose README should become a landing page), create `.github/workflows/landing-page.yml`:

```yaml
name: Build Landing Page

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: write

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: bmoler68/readme-to-landing-page@v1
```

Do **not** create `gh-pages` yourself, and do **not** point Pages at `main`. The action publishes a complete static site to the **root** of `gh-pages`. Using `main` (`/` or `/docs`) would serve the source tree or get overwritten.

## Configure with `with:`

Omit `with:` to use defaults: root `README.md`, publish to `gh-pages`, include TOC.

```yaml
      - uses: bmoler68/readme-to-landing-page@v1
        with:
          readme-path: README.md
          output-dir: site
          branch: gh-pages
          include-toc: 'true'
          github-token: ${{ secrets.GITHUB_TOKEN }}
```

| Input | Description | Default |
| --- | --- | --- |
| `readme-path` | README path relative to repo root | `README.md` |
| `output-dir` | Working directory before push | `site` |
| `branch` | Branch that receives the site | `gh-pages` |
| `include-toc` | Include generated TOC (`true` / `false`) | `true` |
| `github-token` | Token used to push | `${{ github.token }}` |

YAML treats `true`/`false` as booleans. Quote them (`'true'`, `'false'`) so Actions receives the string the action expects.

You only need `github-token` if you are not using the default `GITHUB_TOKEN`. The workflow still needs `permissions: contents: write`.

Example: README in a subdirectory, no TOC:

```yaml
      - uses: bmoler68/readme-to-landing-page@v1
        with:
          readme-path: docs/README.md
          include-toc: 'false'
```

Nested list of what the workflow must have:

- **Checkout** the target repo
- **Run** `bmoler68/readme-to-landing-page@v1`
  - Docker pulls/builds from the action’s `Dockerfile`
  - Parser + generator write `site/`
  - Publisher pushes `gh-pages`
- **Pages** later serves that branch

Recovery if publish fails:

1. Confirm `permissions: contents: write`.
2. Confirm the token can push (`github-token` / `GITHUB_TOKEN`).
3. Re-run the workflow (`workflow_dispatch` or a push to `main`).
4. Then, in this order:
   1. Wait until `gh-pages` exists
   2. Refresh **Settings → Pages**
   3. Select **`gh-pages` / `(root)`**

## Enable GitHub Pages

Until `gh-pages` exists, **Settings → Pages** only lists branches that already exist (usually `None` and `main`). That is expected.

1. Leave Pages set to **None** (or skip Pages until after the first successful run).
2. Add the workflow and run it once (push to `main`, or **Actions → Run workflow**).
3. The action creates `gh-pages` if it is missing and pushes `index.html`, `styles.css`, and `assets/`. `main` is not rewritten.
4. Return to **Settings → Pages**, choose **Deploy from a branch**, then **`gh-pages`** and **`/ (root)`**. Refresh if `gh-pages` is not in the list yet.

After that, each push to `main` updates `gh-pages`.

## What gets published

```
README.md → landing page → gh-pages branch → GitHub Pages
```

```
gh-pages branch:
├── index.html
├── styles.css
└── assets/
```

Task list for first-time setup:

- [x] Action published as `v1` on GitHub
- [x] Consumer workflow in this test repo
- [ ] Pages pointed at `gh-pages` (do this after the first green run)
- [ ] Visual QA of tables, TOC, and sanitizer stripping

Commands operators actually use are GitHub UI + `git push`, not a CLI named `loom`.

> The generated TOC is **not** a backup of this markdown list.  
> It is rebuilt from H2/H3 at generate time.
>
> > If `include-toc` is `'false'`, the sidebar TOC is omitted.

## Runtime and tests

```bash
# in bmoler68/README-to-landing-page
npm install
npm test
docker build -t readme-to-landing-page .
```

Python is **not** a runtime. TypeScript snippet below is a fixture for fenced `ts`:

```ts
export const action = `bmoler68/readme-to-landing-page@v1`;

export function ref(tag: string): string {
  return tag ? `bmoler68/readme-to-landing-page@${tag}` : action;
}
```

`package.json` fragment (real dependency names):

```json
{
  "name": "readme-to-landing-page",
  "version": "1.0.0",
  "license": "MIT",
  "dependencies": {
    "@actions/core": "^1.11.1",
    "@actions/exec": "^1.1.1",
    "marked": "^15.0.6",
    "sanitize-html": "^2.14.0"
  }
}
```

Plain fenced block:

```
PARSE   ok   README.md
GENERATE ok  site/index.html  site/styles.css
PUBLISH  ok  gh-pages
```

CI workflow excerpt (`ci.yml`):

```yaml
name: CI
on:
  push:
    branches: [main, master]
  pull_request:
  workflow_dispatch:
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm install
      - run: npm test
```

Mailto fixture (sanitizer `mailto:` allow-list): [does not exist as a project inbox](mailto:none@example.invalid).

External link with title: [Project homepage](https://www.brianmoler.com "brianmoler.com").

## Markdown fixture gallery

### Headings deeper than H3

Generated TOC indexes **H2 and H3**. These check that H4–H6 still appear in the body.

#### H4 — Badge extraction

##### H5 — Linked vs bare badges

###### H6 — `img.shields.io` vs static SVGs

### Mixed paragraph fixtures

A paragraph with a hard line break (two spaces at the end of the next line)  
should become a `<br>` when `breaks` is enabled.

Autolink: https://github.com/bmoler68/README-to-landing-page.git

Escaped punctuation: \*not italic\*, \`not code\`, \[not a link\].

Unicode: “smart quotes”, an em-dash — and a ⚙️ for the Docker runtime.

### Another image after prose

![Stylesheet fixture](https://cdn.readme-to-landing-page.example/images/styles.png "Fictional CSS screenshot")

| Align left | Center | Right |
| :--- | :---: | ---: |
| `readme-path` | `true` | `@v1` |
| `output-dir` | `false` | `gh-pages` |

## Sanitizer fixtures

<details>
<summary>details/summary is not in the allow list and should flatten or vanish.</summary>

Not a real secret: `github-token` is the workflow token, not a password in this file.
</details>

Keyboard tags are typically stripped: <kbd>Ctrl</kbd>+<kbd>K</kbd>.

Script must never render: <script>alert('xss-fixture')</script>

javascript: links must never survive: [bad](javascript:alert(1))

Horizontal rule follows.

---

Log-shaped inline code: `2026-08-24T21:53:11Z INFO published branch=gh-pages`

```js
const action = "bmoler68/readme-to-landing-page@v1";
export const jobs = ["test", "docker-build"];
```

## This test repository

[README-2-landing-page-test](https://github.com/bmoler68/README-2-landing-page-test) consumes the action. After a successful run:

1. Wait for the **Build Landing Page** workflow.
2. Point GitHub Pages at **`gh-pages` / `(root)`**, not `main`.
3. Confirm live badges match [Actions](https://github.com/bmoler68/README-to-landing-page/actions) and [Releases](https://github.com/bmoler68/README-to-landing-page/releases/latest) on the source repo.

Visual QA:

- [ ] Title is **README to Landing Page**
- [ ] Badge images load from Shields and match CI / release / license
- [ ] Auto TOC lists H2/H3 and skips most H4+
- [ ] Tables, task lists, and fenced code survive
- [ ] Non-badge images stay as broken/fictional HTTPS URLs
- [ ] `javascript:` link is gone

## License

MIT, matching [bmoler68/README-to-landing-page](https://github.com/bmoler68/README-to-landing-page/blob/main/LICENSE). Fixture-only image URLs and sanitizer probes in this file are not part of the action’s published API.
