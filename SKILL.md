---
name: web-clone
description: >
  Website cloning / reproduction methodology. USE WHEN the user says clone this
  website, reproduce this site, copy this site, make one like this site, rebuild
  a webpage's effect, mirror this site and make it mine, reverse-engineer some
  interactive / WebGL / Canvas / Three.js effect — including the Chinese phrasings
  复刻网站、克隆网站、抄个站、仿站、照着这个站做一个、还原某个网页效果、
  把这个站搬下来改成我的、复刻某个交互/WebGL/Canvas/Three.js 效果. Provides a
  portable decision tree — get the real source first → pick a path → reverse-engineer
  → scaffold the project → swap in your content — covering the three branches of
  static sites / React-Vue-Next content sites / WebGL-Canvas heavy frontends, and
  enforces line-by-line verification of any executable code in a second-hand AI analysis.
metadata:
  author: jane (xiaoer)
  version: "1.6.0"
  use_case: Personal local cloning / study of websites, distilled from the website-clones clone hub
---

# Web Clone

**English** · [中文](SKILL.zh-CN.md)

Turn "cloning a website" into a repeatable process. Companion project area: `~/projects/website-clones/` (each clone is a subdirectory).

## Iron rule: real source first, never trust AI-guessed code

> For any AI-generated "clone analysis / blueprint", **you may reference the conceptual skeleton of the prose, but assume every executable code block in it is fabricated** until verified line by line against the real source — otherwise copying it will break.

**Case in point (the marbles case)**: an AI analysis document turned the site's real architecture — "analytic ray-sphere intersection + encode the optical result into a displacement map + hand it to an SVG `feDisplacementMap` to distort the real DOM" — into "ray-marching + SDF + sampling the DOM as a texture". Two entirely different implementations; copying it produces nothing like the original and runs many times slower. See `references/marbles-case.md`.

So the first action is always: **get the real source**.

## The decision tree (follow it in order, no skipping)

### Step 0 · Build the standard project skeleton first

```bash
node /Users/jane/.shared-skills/web-clone/scripts/init-clone.mjs <site> --url <source-url>
```

This script creates `~/projects/website-clones/<site>-clone/`, `NOTES.md`, and `RECON/screenshots/`, so you don't hand-miss deliverables each time.

### Step 1 · Search GitHub for the source first, don't rush to scrape

```bash
unset SSL_CERT_FILE   # macOS quirk, clear it before bash
# search by site name / product name
SSL_CERT_FILE=/etc/ssl/cert.pem gh api "search/repositories?q=<keyword>" \
  | jq -r '.items[] | "\(.full_name) ⭐\(.stargazers_count) \(.description)"' | head -10
# a vercel.app/netlify.app/github.io URL slug is often the repo name / deployer's username
```
- Single-file site (github.io / plain HTML) → grab raw directly: `curl -sL https://raw.githubusercontent.com/<user>/<repo>/main/index.html`
- **Source found and the license allows it → skip to Step 4 and clone directly.** Lesson: searching GitHub first can save 30 minutes of detours.

### Step 2 · No source found → browser recon (probes)

Load the `Browser` skill or the playwright MCP, run probes to extract signals (framework / `window.THREE` / canvas count / smooth-scroll library / fonts / scrollHeight). Save 1440/768/390 screenshots at three widths + recon JSON to `<site>/RECON/`.

Prefer the built-in scripts to run the standard recon:

```bash
node /Users/jane/.shared-skills/web-clone/scripts/recon-site.mjs \
  --url <source-url> \
  --out ~/projects/website-clones/<site>-clone/RECON \
  --label original

node /Users/jane/.shared-skills/web-clone/scripts/asset-harvest.mjs \
  --recon ~/projects/website-clones/<site>-clone/RECON/original-recon.json \
  --out ~/projects/website-clones/<site>-clone/assets/original \
  --manifest ~/projects/website-clones/<site>-clone/RECON/asset-manifest.json

node /Users/jane/.shared-skills/web-clone/scripts/network-capture.mjs \
  --url <source-url> \
  --out ~/projects/website-clones/<site>-clone/RECON/network \
  --label original

node /Users/jane/.shared-skills/web-clone/scripts/route-crawl.mjs \
  --url <source-url> \
  --out ~/projects/website-clones/<site>-clone/RECON/routes \
  --label original \
  --max-pages 25 \
  --max-depth 2

node /Users/jane/.shared-skills/web-clone/scripts/interaction-probe.mjs \
  --url <source-url> \
  --out ~/projects/website-clones/<site>-clone/RECON/interactions \
  --label original

node /Users/jane/.shared-skills/web-clone/scripts/sourcemap-hunt.mjs \
  --recon ~/projects/website-clones/<site>-clone/RECON/original-recon.json \
  --out ~/projects/website-clones/<site>-clone/RECON/sourcemaps
```

> Only use `bb-browser` for logged-in / private sites; for localhost / public sites with no login use the `Browser` skill or playwright (don't touch bb-browser, it will hijack Brave).

### Step 2.5 · Rate the complexity first, don't promise blindly

Based on the recon, write a first-pass "pre-clone assessment": complexity L1-L6, recommended mode (faithful clone / visual clone / content overhaul), the expected reproducible scope, and an explicit list of what won't be cloned. Grading and scoring rules are in `references/assessment.md`.

**If the mode is "visual clone" or "content overhaul"** → produce the structured design identity `design-dna.json` along the way, turning "the feel of that site" into a versionable, comparable token spec, so that when Step 6 does the swap you have a basis for "keep the DNA, swap the content":

```bash
node /Users/jane/.shared-skills/web-clone/scripts/dna-scaffold.mjs \
  --recon ~/projects/website-clones/<site>-clone/RECON/original-recon.json \
  --out   ~/projects/website-clones/<site>-clone/RECON/design-dna.json \
  --name  "<site>"
```

The script best-effort pre-fills the reconned fonts / color candidates / framework-effect signals, leaving the rest to be completed manually during Analyze. The three-axis structure (design_system / design_style / visual_effects), full schema, and applicability boundaries are in `references/design-dna.md`.
> ⚠️ **Don't use DNA on the "faithful clone" branch**: the real source is the truth; don't let an "approximate style" DNA dilute the byte-for-byte iron rule.

### Step 3 · Pick a path based on the recon

| Recon result | Which path |
|---|---|
| Static HTML/CSS, no framework | `wget --mirror` to grab a mirror → delete tracking scripts → edit copy |
| React / Vue / Next (content-driven) | Rebuild on a template (e.g. `ai-website-cloner-template`, Node 24+), pour in content |
| SPA / SaaS / data-driven pages | Run `network-capture.mjs` first to save API fixtures → stand in with a local JSON/mock server |
| Multi-page marketing / product site | Run `route-crawl.mjs` first to build a route map → extract a template per page type → swap content uniformly |
| Complex interactive site | Run `interaction-probe.mjs` first to record hover/click/scroll/canvas-drag states → build interactions per state, don't just screenshot the first screen |
| **WebGL / Canvas / Three.js heavy frontend** | **Deep-reverse-engineer the real source (see below) → faithful clone, or find a comparable open-source 3D template and swap content**. Single-file native sites are often kept byte-for-byte = the most faithful clone. **When no real source is findable, go runtime frame-capture + baseline gate**; discipline in `references/effect-extraction.md` (can be delegated to web-shader-extractor) |
| **Static-built site (Astro/Vite SSG/Hugo), incl. heavy WebGL** | **`mirror-site.mjs` mirrors the full deployed assets → self-host fonts + strip tracking → serve from a local web root = a true 1:1 faithful clone of the real source**. For a static site, "get the real source" = "mirror the whole deployed bundle". Recipe in `references/static-mirror.md`. Example: oryzo.ai (Lusion, L6, Gaussian splatting, hero pixel diff 5/5) |
| Site using an off-the-shelf open-source theme (Astro/Hugo theme) | Go to the corresponding theme market and find the source theme (**only for sites that apply a ready-made theme**; custom sites take the full-mirror row above, not this one) |

For L4-L6 complex sites, follow `references/complex-playbooks.md` — don't use the ordinary marketing-site flow.

### Step 4 · Scaffold the project inside the clone hub

```bash
mkdir ~/projects/website-clones/<site>-clone && cd $_
# git source: clone it in; single-file: drop it in. Keep a read-only baseline index-original.html of the original source
# check the Node version (package.json engines), nvm use the matching version, pin .nvmrc
```

### Step 5 · Strip tracking + write metadata + verify

- **Strip tracking**: Google Analytics (`gtag` / `googletagmanager`), pixels, heatmaps — excise line by line precisely (the GA block is often at the top of `<head>`).
- **Write NOTES.md** (mandatory): include complexity, clone mode, original-vs-clone comparison, fidelity scores, known gaps. Template in `references/deliverables.md`.
- **Write TEARDOWN.md for complex sites** (technical teardown). Every conclusion tagged with a real source line number.
- **Post-clone scoring**: score structure / visual / interaction / responsive / content-swap / functional-completeness per `references/assessment.md`. Scores must be supportable by screenshots, source, and run results.
- **Real browser verification** (a hard requirement — you may not just read the code and say "it should run"): start a local server → open a browser → capture the console (no JS/WebGL compile errors) → screenshot against the original. Honestly record what you can't verify (e.g. a synthetic PointerEvent with `isTrusted=false` can't trigger the drag — write that down truthfully, don't fake a "drag succeeded").

After the clone is done, run the clone-site recon once more and generate an automatic comparison report:

```bash
node /Users/jane/.shared-skills/web-clone/scripts/recon-site.mjs \
  --url http://127.0.0.1:<port>/ \
  --out ~/projects/website-clones/<site>-clone/RECON \
  --label clone

node /Users/jane/.shared-skills/web-clone/scripts/route-crawl.mjs \
  --url http://127.0.0.1:<port>/ \
  --out ~/projects/website-clones/<site>-clone/RECON/routes-clone \
  --label clone \
  --max-pages 25 \
  --max-depth 2

node /Users/jane/.shared-skills/web-clone/scripts/interaction-probe.mjs \
  --url http://127.0.0.1:<port>/ \
  --out ~/projects/website-clones/<site>-clone/RECON/interactions-clone \
  --label clone

node /Users/jane/.shared-skills/web-clone/scripts/compare-recon.mjs \
  --original ~/projects/website-clones/<site>-clone/RECON/original-recon.json \
  --clone ~/projects/website-clones/<site>-clone/RECON/clone-recon.json \
  --visual-diff ~/projects/website-clones/<site>-clone/RECON/visual-diff-1440.json \
  --original-routes ~/projects/website-clones/<site>-clone/RECON/routes/original-route-map.json \
  --clone-routes ~/projects/website-clones/<site>-clone/RECON/routes-clone/clone-route-map.json \
  --original-interactions ~/projects/website-clones/<site>-clone/RECON/interactions/original-interactions.json \
  --clone-interactions ~/projects/website-clones/<site>-clone/RECON/interactions-clone/clone-interactions.json \
  --out ~/projects/website-clones/<site>-clone/CLONE_REPORT.md

node /Users/jane/.shared-skills/web-clone/scripts/visual-diff.mjs \
  --original ~/projects/website-clones/<site>-clone/RECON/screenshots/original-1440.png \
  --clone ~/projects/website-clones/<site>-clone/RECON/screenshots/clone-1440.png \
  --out ~/projects/website-clones/<site>-clone/RECON/visual-diff-1440.json \
  --diff ~/projects/website-clones/<site>-clone/RECON/screenshots/visual-diff-1440.png

node /Users/jane/.shared-skills/web-clone/scripts/audit-clone.mjs \
  --project ~/projects/website-clones/<site>-clone \
  --brand "<brand>" \
  --out ~/projects/website-clones/<site>-clone/CLONE_AUDIT.md
```

### Step 6 · Replace with Jane's own material

The goal is always "make her own site", not to carry over an identical copy. The three swap targets: text (`index.html`/`data/*.json`/`content/*.md`), media (`public`/`assets`), and brand colors (CSS variables / Tailwind theme). If the structure is non-trivial, write a REPLACE_GUIDE.md.

**If you produced a `design-dna.json` (visual/overhaul mode)**: this step is where it pays off — **keep the DNA, swap the content**. Realize `design_system` as CSS custom properties, make subjective choices per `design_style`, pick an implementation tier per `visual_effects.effect_intensity` (lightweight CSS / medium Canvas+GSAP / heavy Three.js); for assets, prefer pulling the original's real images via `asset-harvest.mjs` rather than having AI re-draw approximations. Generation flow in `references/design-dna.md`.

## Reverse-engineering WebGL/Canvas heavy frontends (the core craft)

Break an interactive site into **technical pillars**, and for each pillar locate the real implementation + tag line numbers: rendering (WebGL/shader algorithms), compositing (SVG filter / multi-canvas / post-processing), physics, interaction, audio. Only then verify against any second-hand analysis.

**The three disciplines when reverse-engineering effects** (they cure "polishing while you extract, ending up neither faithful nor explainable"):
1. **Evidence grading**: tag each conclusion `SOURCE` (real source / source-map / runtime dump / frame capture), `PARTIAL` (a name/slice, to be proven), or `GUESS` (visual fit / magic number). **Untagged = GUESS; it must be upgraded to SOURCE before copying.**
2. **no-compensation**: it is forbidden to tweak brightness/speed/position/noise to mask timing/coordinate/state errors; a fitted value is still tagged GUESS, with a note on what evidence is needed to upgrade it.
3. **baseline-first gate**: first do a "minimal, as-is reproducible RAW REPLAY" using real draw calls/shaders/uniforms → pass frame-by-frame comparison → **only then** are you allowed to refactor into an editable project.
See `references/effect-extraction.md` (includes the runtime-capture fallback + when to delegate to web-shader-extractor).

**Transferable advanced patterns** (worth saving up):
- **Displacement-map refraction of the DOM**: offscreen WebGL computes a "displacement map" with RG=displacement / B=Fresnel, then an SVG `<filter><feDisplacementMap scale=N>` uses it to distort the real, live, interactive HTML — what's refracted is the real DOM, and WebGL never touches DOM pixels. This is the soul of marbles, and something Three.js `MeshPhysicalMaterial(transmission)` cannot do (it can only produce a "glass-ball look", not "refract the whole webpage").

Detailed method + item-by-item teardown of the marbles real architecture → `references/reverse-engineering.md`, `references/marbles-case.md`.

## License & attribution (check before you clone)

```bash
SSL_CERT_FILE=/etc/ssl/cert.pem gh api repos/<u>/<r> | jq '.license'  # + find the LICENSE file + read the README
```

| License | What you can do |
|---|---|
| MIT / Apache / BSD / Unlicense | Modify, redeploy, just keep the credits |
| **NONE (no LICENSE file / unstated)** | **All rights reserved by default**. Local study/cloning only, must credit the original author, **do not redeploy publicly without permission**. Don't treat "the code is public" as free to use |
| Proprietary / explicitly forbidden | Read-only study, no copying, no deployment |

⚠️ Don't equate "it's public on GitHub" or "gh api couldn't find it right now" with MIT — verify it for real.

## Deliverable spec
- Each subproject root: `NOTES.md` (source info / tech stack / license / replace map / run commands)
- Complex interactive sites add: `TEARDOWN.md` (technical teardown, line numbers tagged)
- When you need to report out or assess skill effectiveness, add: `CLONE_REPORT.md` (full original-vs-clone comparison)
- Before going live, add: `CLONE_AUDIT.md` (tracking scripts, original brand/language residue, TODOs, external-link risks)
- `RECON/screenshots/`: original-vs-clone comparison images
- After cloning, update the hub `README.md` index line

## Built-in scripts
- `scripts/init-clone.mjs`: initialize the clone project skeleton and `NOTES.md`.
- `scripts/recon-site.mjs`: open the page with Playwright, collect framework/resource/DOM-structure/console-error signals, and save three screenshot widths.
- `scripts/asset-harvest.mjs`: download the original's images, scripts, styles from the recon JSON and generate an asset manifest.
- `scripts/network-capture.mjs`: capture XHR/fetch requests and save JSON/text responses as local fixtures for SPA/SaaS.
- `scripts/mirror-site.mjs`: real-browser full-scroll capture of every real request → mirror same-origin assets by path (incl. JS-runtime-fetched `.sog/.buf/.wasm/.riv`/fonts), for a 1:1 faithful clone of static-built sites (Astro/Vite SSG/Hugo). See `references/static-mirror.md`.
- `scripts/route-crawl.mjs`: crawl same-site internal links, save screenshot/title/H1/structure signals per route, solving the "multi-page site only cloned the homepage" problem.
- `scripts/interaction-probe.mjs`: automatically perform scroll, hover, safe click, canvas drag, and save before/after states, screenshots, network and console evidence.
- `scripts/sourcemap-hunt.mjs`: find source maps in the JS chunks and save the source mapping when available.
- `scripts/compare-recon.mjs`: read the original and clone recon JSON, route maps, and interaction evidence, and generate `CLONE_REPORT.md`.
- `scripts/visual-diff.mjs`: do screenshot pixel diffing via a browser canvas, output a visual score and a diff image.
- `scripts/audit-clone.mjs`: scan for tracking scripts, original brand residue, Japanese residue, TODOs, and external-URL risks.
- `scripts/dna-scaffold.mjs`: generate the `design-dna.json` design-identity skeleton from the recon JSON (best-effort pre-fill of fonts / color candidates / framework-effect signals), for "visual clone / content overhaul" mode. See `references/design-dna.md`.

## Capability boundaries (default stance)
- **Can do at high fidelity**: static marketing pages, corporate sites, content-type React/Vue/Next frontends, animation sites whose source is directly obtainable.
- **Can visually reproduce but will simplify**: CMS backend data, complex scroll storytelling, multi-device breakpoints, WebGL/Canvas effects, third-party embeds.
- **Won't promise a full clone by default**: login, payment, checkout, search/recommendation, permission systems, server-side business logic, proprietary APIs, copyright-restricted assets. When needed, only build a demonstrable frontend stand-in.
- **In content-overhaul mode**: prefer preserving the original's information architecture, rhythm, motion, and visual grammar, while swapping the copy, images, brand colors, and value proposition for Jane's own content.

## Flagship case
`~/projects/website-clones/marbles-clone/` — a native WebGL + SVG Filter + hand-rolled physics glass-marbles site, cloned byte-for-byte + a full TEARDOWN, the exemplar of the "WebGL heavy-frontend branch".
