# SprocketKit

[![Build](https://img.shields.io/badge/build-passing-brightgreen.svg)](https://github.com/bmoler68/README-to-landing-page)
[![Release](https://img.shields.io/badge/release-v2.4.1-blue.svg)](https://github.com/bmoler68/README-to-landing-page)
[![License](https://img.shields.io/badge/license-MIT-yellow.svg)](LICENSE)
[![Coverage](https://img.shields.io/badge/coverage-94%25-success.svg)](https://github.com/bmoler68/README-to-landing-page)
![Node](https://img.shields.io/badge/node-%3E%3D20-339933.svg)
![Status](https://img.shields.io/badge/status-experimental-orange.svg)

**SprocketKit** is a fictitious widget toolkit for assembling dashboard gadgets: counters, knobs, spark-strips, and “honest clocks” that refuse to show a timezone they cannot prove. It is not a real product — this README is a *stress test* for [README to Landing Page](https://github.com/bmoler68/README-to-landing-page.git).

> *“A widget should click, glow, and confess when it is lying.”*  
> — **June Pell**, fictional lead designer, Workshop B

![Hero — kit of dashboard widgets](https://cdn.sprocketkit.example/images/hero-workbench.png "Fictional SprocketKit workbench")

The image URL above is **invented**. It exists so the converter still sees markdown images without fetching a real photo host.

## Table of contents

- [Why widgets](#why-widgets)
- [At a glance](#at-a-glance)
- [Architecture](#architecture)
  - [Host shell](#host-shell)
  - [Widget runtime](#widget-runtime)
  - [Latch bus](#latch-bus)
- [Quick start](#quick-start)
- [Configuration](#configuration)
- [CLI cookbook](#cli-cookbook)
- [Widget recipes](#widget-recipes)
- [Observability](#observability)
- [API surface](#api-surface)
- [Shop-floor playbooks](#shop-floor-playbooks)
- [Markdown fixture gallery](#markdown-fixture-gallery)
- [Roadmap](#roadmap)
- [Contributing](#contributing)
- [License](#license)

---

## Why widgets

Product teams keep reinventing the same **little machines**: a status pebble, a range knob, a tally that animates when a ticket closes. Copy-paste works until two knobs disagree about units and a third one draws in `canvas` for no reason.

SprocketKit treats every gadget as a **sprocket**. Sprockets snap onto a **latch** (a typed event bus). Latches compose into **panels**. Panels can be paused, themed, or hot-swapped without remounting the host app.

You should reach for SprocketKit when:

1. You need **dozens of small UI machines**, not one giant chart.
2. Design wants **click, glow, and disabled** states that match across surfaces.
3. Engineers want a **single registry** without giving up local overrides.

You should *not* reach for SprocketKit to build a word processor.

## At a glance

| Capability | What it means in practice | Maturity |
| :--- | :--- | :---: |
| Sprocket registry | Discover widgets by id and slot | **GA** |
| Latch bus | Typed events between sprockets | **GA** |
| Theme lathe | Tokens for color, click, and glow | **beta** |
| Ghost preview | Replay last 72 interactions in the lab | **beta** |
| Quill captions | Attach designer notes to any sprocket | **preview** |
| Knob physics | Inertia and snap points | **preview** |

Inline emphasis check: this sentence uses **bold**, *italic*, ***bold italic***, ~~strikethrough~~, `inline code`, a [named link](https://github.com/bmoler68/README-to-landing-page), an [anchor link](#cli-cookbook), and mixed `code **inside** ticks` that should stay literal.

Subscript / superscript via HTML: torque is logged at 10<sup>−3</sup> N·m. Ins/del: <ins>added glow token</ins> after <del>legacy hex-only fills</del>.

## Architecture

```
                    ┌──────────────────────────┐
   host app   ──►   │       Host shell         │
   theme file ──►   │  registry · identity     │──► panel UI
   flags      ──►   │  policy · slots          │
                    └────────────┬─────────────┘
                                 │
                    ┌────────────▼─────────────┐
                    │     Widget runtime       │
                    │  sprockets · previews    │
                    └────────────┬─────────────┘
                                 │
                    ┌────────────▼─────────────┐
                    │        Latch bus         │
                    │  click · glow · tick     │
                    └──────────────────────────┘
```

### Host shell

The host is **boring on purpose**. It owns the panel DOM, loads the registry, and never paints a knob itself.

- Identity: OIDC, plus a fictional **bench token** for kiosks with no browser profile.
- Policy: allow lists per sprocket (`render`, `annotate`, `rewind`).
- Catalog: content-addressed widget bundles; names are just aliases.

### Widget runtime

Sprockets pull props, not the other way around. A widget that throws in `render` should not take the panel with it.

1. **Mount** into a named slot.
2. **Latch** onto bus topics the panel declared.
3. **Paint** using theme tokens only.
4. **Ack** so Ghost Preview can reconstruct the same click later.

### Latch bus

The bus is a tiny, ordered queue. Topics look like `click.knob.primary` and `glow.strip.ok`.

![Panel of mixed widgets](https://cdn.sprocketkit.example/images/panel-mixed.png "Fictional mixed widget panel")

[![Linked thumbnail of a knob close-up](https://cdn.sprocketkit.example/images/knob-thumb.png)](https://cdn.sprocketkit.example/images/knob-full.png)

HTML `<img>` with width/height (allowed by the landing-page sanitizer; URL is still fictional):

<img src="https://cdn.sprocketkit.example/images/spark-strip.png" alt="Fictional spark-strip widget" width="480" height="240" title="HTML img tag">

## Quick start

Requirements: Node **20+**, a bundler, and a sense of humor.

```bash
# fictional installer — do not run in a real storefront
curl -fsSL https://sprocketkit.example/install.sh | sh
skit init --panel workshop-b --theme brass
skit sprocket add counter tally-a
skit preview --bind 127.0.0.1:8787
```

Python client (fictional):

```python
from sprocketkit import Client, Slot

client = Client(panel="workshop-b", token="bench-demo")
slot = Slot.parse("hero/left")

for sprocket in client.sprockets(prefix="knob."):
    state = client.pull(sprocket, slot)
    print(sprocket.name, state.clicks, "clicks")
```

TypeScript snippet with nested backticks in a string:

```ts
export const banner = `skit status --json`;

export function latch(ids: string[]): string {
  return ids.join("~>") || "empty-panel";
}
```

JSON catalog fragment:

```json
{
  "panel": "workshop-b",
  "version": 12,
  "sprockets": ["counter.tally", "knob.primary", "clock.honest"],
  "theme": "brass"
}
```

Plain fenced block with no language tag:

```
MOUNT  ok   knob.primary     slot=hero/left
LATCH  ok   click.knob.primary
ACK    ok   runtime-17
```

## Configuration

SprocketKit reads `skit.toml` first, then environment variables, then flags. Later sources win.

```toml
[panel]
name = "workshop-b"
theme = "brass"

[bus]
url = "wss://bus.sprocketkit.example:8788"
topic = "workshop-b/+/click"

[preview]
retention = "72h"
ghost_workers = 4
```

| Flag | Env | Default | Notes |
| --- | --- | --- | --- |
| `--panel` | `SKIT_PANEL` | *(required)* | Short name, lowercase |
| `--theme` | `SKIT_THEME` | `steel` | `steel`, `brass`, or `night` |
| `--ui.port` | `SKIT_UI_PORT` | `8787` | Bind locally in demos |
| `--preview.retention` | `SKIT_PREVIEW_RETENTION` | `72h` | Go-duration syntax |

Nested list for builders who skim:

- **Sprockets**
  - Counter (`tally`, `delta`, `cap`)
  - Knob (snap points, inertia)
  - Honest clock (refuses unknown zones)
- **Latch**
  - Click, glow, tick
  - User WASM (preview)
- **Emit**
  - Analytics envelope
  - Webhook (signed)

Ordered recovery steps:

1. Confirm the host shell (`skit host ping`).
2. Drain the latch spool (`skit bus status`).
3. If spool > 80%, **pause** non-critical sprockets.
4. Resume in this order:
   1. Registry / theme
   2. Counters and knobs
   3. Spark-strips
   4. Quill captions

## CLI cookbook

Task list (GFM checkboxes):

- [x] `skit init` scaffolds a panel
- [x] `skit sprocket add` registers a gadget
- [ ] `skit theme pack` is still preview
- [ ] `skit quill export --figma` is a joke ticket

Inline commands worth memorizing: `skit status`, `skit panel diff`, and `skit preview --dry-run`.

Blockquote with nested quote:

> Ghost Preview is not a screenshot.  
> It is a **click machine with a fuse**.
>
> > If you rewind past retention, SprocketKit will refuse rather than invent clicks.

## Widget recipes

### Workshop B panel

Mount a tally, a brass knob, and an honest clock onto one latch, then emit a 5-second rollup of clicks.

```yaml
name: workshop-b
theme: brass
nodes:
  - id: tally
    type: sprocket.counter
    slot: hero/left
  - id: knob
    type: sprocket.knob
    slot: hero/center
  - id: clock
    type: sprocket.clock
    honest: true
  - id: rollup
    type: reduce.sum
    input: latch.click
    step: 5s
```

### Wide table of catalog SKUs

| SKU | Kind | Latency | Notes |
| ---: | --- | --- | --- |
| `CTR-01` | counter | 16 ms | Caps at 999 |
| `KNB-BR` | knob | 8 ms | Brass theme only |
| `CLK-H` | clock | 1 s | Refuses `Etc/Unknown` |
| `SPK-7` | spark-strip | 32 ms | Drops frames on kiosk GPUs |

Alignment in tables is a common converter footgun: left, center, and right columns should all survive.

## Observability

Metrics are Prometheus-shaped but named like hardware.

| Metric | Type | Meaning |
| --- | --- | --- |
| `skit_sprocket_lag_ms` | gauge | How far a widget is behind the latch |
| `skit_panel_commits_total` | counter | Successful panel paints |
| `skit_ghost_preview_seconds` | histogram | Cost of a rewind |

Log line examples (inline + pre):

`2026-08-24T21:53:11Z WARN runtime-17 spool=81% sprocket=knob.primary`

Horizontal rule below this paragraph tests `<hr>` rendering.

---

## API surface

Base URL (fictional): `https://api.sprocketkit.example/v1`

### `GET /sprockets`

Returns widget descriptors. Query params: `prefix`, `limit`.

### `POST /panels/{name}:apply`

Compiles and hot-swaps a panel. Returns `202` with a lease id.

Mailto fixture for sanitizer scheme allow-lists: [ops@sprocketkit.example](mailto:ops@sprocketkit.example).

External link with title attribute: [Workshop B handbook](https://docs.sprocketkit.example/workshop-b "Fictional handbook").

## Shop-floor playbooks

<details>
<summary>This HTML details/summary block should be stripped or flattened by sanitizer (not in the allow list). If you still see a disclosure widget, sanitizer config changed.</summary>

Secret-looking but fake: `bench-token-demo-not-real`
</details>

Keyboard tags are also typically stripped: press <kbd>Ctrl</kbd>+<kbd>K</kbd> in the fictional UI.

Script tags must never render: <script>alert('xss-fixture')</script>

javascript: links must never survive: [bad](javascript:alert(1))

## Markdown fixture gallery

### Headings deeper than H3

The generated landing-page TOC usually indexes **H2 and H3 only**. These next headings check whether H4–H6 still appear in the body.

#### H4 — Quill caption schema

##### H5 — Attachment types

###### H6 — `image/png` vs `text/plain`

### Mixed paragraph fixtures

A paragraph with a hard line break (two spaces at the end of the next line)  
should become a `<br>` when `breaks` is enabled.

Autolink: https://github.com/bmoler68/README-to-landing-page.git

Escaped punctuation: \*not italic\*, \`not code\`, \[not a link\].

Unicode and emoji: Workshop B keeps a ⚙️ on the bench. Torque is +1.2 N·m. Café notes use “smart quotes” and an em-dash — like this.

### Another image, after lots of prose

![Honest clock widget](https://cdn.sprocketkit.example/images/honest-clock.png "Fictional honest clock")

## Roadmap

| Quarter (fictional) | Theme | Status |
| --- | --- | --- |
| Q1 2026 | Latch bus GA | Done |
| Q2 2026 | WASM sprockets | In progress |
| Q3 2026 | Multi-panel kits | Planned |
| Q4 2026 | Knob physics pack | Planned |

```js
// Highlight-class fixture: fenced js
const sprockets = ["tally", "knob", "clock"];
export const roster = sprockets.map((s) => s.toUpperCase());
```

## Contributing

This repository is a **conversion fixture**, not a real community project. If you are testing the GitHub Action:

1. Push this README to `main` (or run the workflow manually).
2. Wait for [bmoler68/readme-to-landing-page](https://github.com/bmoler68/README-to-landing-page) to publish `gh-pages`.
3. Point GitHub Pages at **`gh-pages` / `(root)`**, not `main`.

Checklist for visual QA on the generated page:

- [ ] Title is **SprocketKit**, not a fallback
- [ ] Badge row appears under the hero
- [ ] Auto TOC lists H2/H3 and skips most H4+
- [ ] Tables keep headers and body cells
- [ ] Fenced code keeps indentation and language
- [ ] Task list checkboxes render
- [ ] Markdown images are present; only **badge** images load from a real host
- [ ] `mailto:` link works; `javascript:` link is gone
- [ ] Footer mentions MIT from `LICENSE`

## License

MIT. 

The SprocketKit product, Workshop B, and designer quotes are **fiction** created to exercise markdown conversion.
