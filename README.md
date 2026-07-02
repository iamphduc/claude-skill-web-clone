# 🪞 Web Clone

> Reproduce any website — from a single-file static page to a WebGL-heavy interactive demo — without faking it from AI hallucinations.

**English** · [中文](README.zh-CN.md)

---

## What this skill does

You see a website you love. You want to clone it — to learn how it works, to remix it into your own thing, or to run it offline. AI tools love to produce *plausible-looking* "clone analysis" documents full of code blocks that are entirely fabricated and break the moment you run them.

This skill is a **methodology plus executable probes that put real source first**, with a 6-step decision tree covering:

- Static HTML/CSS sites → `wget --mirror`
- React / Vue / Next content sites → rebuild on a template
- Multi-page sites → crawl internal routes before designing templates
- Interactive sites → record hover / click / scroll / canvas-drag states before cloning
- **WebGL / Canvas / Three.js heavy frontends** → reverse-engineer real source, line by line; when there's no findable source, runtime frame-capture + a **baseline-first replay gate** with `SOURCE / PARTIAL / GUESS` evidence grading
- **Static-built sites (Astro / Vite SSG / Hugo), even WebGL-heavy** → `mirror-site.mjs` full-scroll-captures and mirrors every deployed asset (incl. runtime-fetched `.sog/.buf/.wasm`) for a true 1:1 clone — for a static site, "get the real source" = "mirror the whole deployed bundle"
- Verify in a real browser, document the truth, replace content with yours
- **Visual / rebrand modes** → distill a versionable `design-dna.json` (design tokens + style + effects), then "keep the DNA, swap the content"

## Iron rule: real source first, never copy AI-guessed code

> Any AI-written "clone analysis" — treat the conceptual skeleton as a hint, but **assume every executable code block is hallucinated** until you've verified it against real source code line by line.

**Case in point** (see `references/marbles-case.md`): A real WebGL marbles demo uses **analytic ray-sphere intersection** + an SVG `feDisplacementMap` to refract the live DOM. An AI analysis confidently described it as **ray-marching + SDF + sampling the DOM as a texture** — two entirely different implementations. Copying the AI version produces nothing like the original and runs many times slower.

So **step 1 is always: get the real source**.

## What's in here

```
claude-skill-web-clone/
├── SKILL.md                       ← The methodology (loaded by Claude Code / Codex / Cursor)
├── README.md
├── LICENSE                         ← MIT
├── follow-wechat-qrcode.jpg
└── references/
    ├── reverse-engineering.md     ← How to dissect a WebGL/Canvas frontend, line by line
    ├── effect-extraction.md       ← Evidence grading + baseline-first gate for extracting effects
    ├── static-mirror.md           ← 1:1 clone of static-built sites (Astro/Vite SSG/Hugo) via full asset mirror
    ├── design-dna.md              ← Structured design-identity layer (visual / rebrand modes)
    ├── marbles-case.md            ← Flagship case: real architecture vs AI hallucination
    └── deliverables.md            ← NOTES.md / TEARDOWN.md / RECON/ templates
└── scripts/
    ├── recon-site.mjs             ← Browser screenshots + DOM/framework/resource signals
    ├── mirror-site.mjs            ← Full-scroll capture + mirror every same-origin asset (static-site 1:1 clone)
    ├── route-crawl.mjs            ← Same-site route map + screenshot per route
    ├── interaction-probe.mjs      ← Scroll/hover/click/canvas-drag state evidence
    ├── network-capture.mjs        ← XHR/fetch capture for SPA fixtures
    ├── asset-harvest.mjs          ← Download discovered source assets
    ├── dna-scaffold.mjs           ← Build a design-dna.json skeleton, prefilled from recon
    └── visual-diff.mjs            ← Pixel comparison for original vs clone screenshots
```

## Install

This is a [Claude Code skill](https://docs.claude.com/en/docs/agents-and-tools/skills). Drop it into your skills directory:

```bash
git clone https://github.com/Jane-xiaoer/claude-skill-web-clone.git ~/.claude/skills/web-clone
```

Then use it by saying things like:
- "复刻这个网站: https://..."
- "Clone this site for me"
- "Reverse-engineer this WebGL demo"
- "把这个站抄下来改成我的"

Claude / Codex will load `SKILL.md` and walk the decision tree.

## The decision tree (one-screen summary)

| Step | What to do |
|---|---|
| **1** | `gh api search/repositories?q=<name>` — find the real source on GitHub FIRST |
| **2** | If no source: browser-probe the site, crawl routes, capture network, and probe interactions |
| **3** | Pick the path: `wget` mirror / template rebuild / **WebGL reverse-engineering** / theme market |
| **4** | Set up `~/projects/website-clones/<name>-clone/` (or your equivalent), keep `index-original.html` read-only |
| **5** | Strip tracking, write `NOTES.md` + `TEARDOWN.md`, verify in a real browser with screenshots |
| **6** | Replace text / media / brand colors with yours |

## License & attribution discipline

Before deploying anything publicly, **check the source repo's license**:

| License | What you can do |
|---|---|
| MIT / Apache / BSD / Unlicense | Modify, redeploy, just keep credits |
| **NONE / unstated** | Default = **All Rights Reserved**. Local learning only, must credit, **do not redeploy publicly without permission** |
| Proprietary / explicitly forbidden | Read-only learning, no copying, no deployment |

⚠️ "Code is on GitHub" ≠ "code is MIT". Many viral demos have no LICENSE file and default to All Rights Reserved.

## Flagship case

`marbles-case.md` documents the full reverse-engineering of [chiuhans111/marbles](https://github.com/chiuhans111/marbles) — a 1067-line single-file WebGL + SVG-filter glass marbles interactive demo — and contrasts the real architecture with an AI-generated clone analysis that got the core mechanism completely backwards. This is the canonical example of why you must verify against real source.

---

## Credits

- **Origin**: Jane's clone hub `~/projects/website-clones/` workflow (skill extracted after the marbles case landed on 2026-05-27)
- **Flagship case author**: [chiuhans111/marbles](https://github.com/chiuhans111/marbles) by Hans Chiu — without this single-file, 1067-line glass-marbles source there'd be no "real source first" iron rule
- **Design-DNA schema**: the three-axis DNA JSON in `references/design-dna.md` is adapted from [zanwei/design-dna](https://github.com/zanwei/design-dna) (MIT), used for the visual / rebrand modes
- **Effect-extraction discipline**: the evidence grading + baseline-first gate in `references/effect-extraction.md` is inspired by [lixiaolin94/skills · web-shader-extractor](https://github.com/lixiaolin94/skills) (that repo has no LICENSE, so this skill borrows only the method concept, rewritten in its own words — no code or text copied)
- **Iteration**: Jane (`@xiaoerzhan`) + Claude Code (multiple rounds of reverse-engineering)

---

## License

MIT. Use it freely, remix it, ship it. Attribution welcome but not required.

If this skill saves you a frustrating afternoon of debugging fake AI code, ⭐ the repo.

---

## 📱 Follow Me

If this repo helped you, follow me for more AI skills, design systems, web aesthetics, and creative workflows.

- X (Twitter): [@xiaoerzhan](https://x.com/xiaoerzhan)
- WeChat Official Account: scan the code below to follow

<p align="center">
  <img src="./follow-wechat-qrcode.jpg" alt="Jane WeChat Official Account QR code" width="300" />
</p>

<p align="center">Follow my WeChat Official Account for more AI skills, design principles, web aesthetics, and creative workflows.</p>
