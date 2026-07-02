# Deliverables Spec & Templates

**English** · [中文](deliverables.zh-CN.md)

Standard deliverables for each clone subproject (`~/projects/website-clones/<site>-clone/`).

## NOTES.md (required)

```markdown
# <site> · Clone Notes

## Source Info
- Original URL:
- Source repo: (if any)
- Original author:
- License: MIT / Apache / NONE / proprietary  ← must verify, see the license table in SKILL.md
- Attribution requirement:

## Tech Stack
- Framework / key libraries / Node version:

## Pre-clone Assessment
- Complexity level: L1 / L2 / L3 / L4 / L5 / L6 (see the web-clone skill's references/assessment.md)
- Recommended mode: faithful clone / visual clone / content overhaul / technical teardown
- Parts that can be high-fidelity:
- Parts needing approximation or substitution:
- Parts not to clone:
- Main risks:

## Getting It Running
\`\`\`bash
cd ~/projects/website-clones/<site>-clone
# single-file static site: python3 -m http.server 8123
# framework site: nvm use <ver> && npm install && npm run dev
\`\`\`

## What Changed (vs. original)
- Removed tracking scripts: ... (GA/gtag line numbers)
- ...

## Original vs. Clone
| Module | Original behavior | Clone implementation | Difference / trade-off | Evidence |
|---|---|---|---|---|
| Hero |  |  |  | screenshot / file:line |
| Navigation |  |  |  |  |
| Core animation |  |  |  |  |
| Content blocks |  |  |  |  |
| Mobile |  |  |  |  |

## Clone Score
- Source evidence: /5
- Structural fidelity: /5
- Visual fidelity: /5
- Animation/interaction: /5
- Responsiveness: /5
- Functional completeness: /5
- Content replacement: /5
- Legal/deployment risk: /5
- Overall:

## Replacement Map (what to swap, where to edit)
- Text → file, line
- Images/media → directory
- Color scheme → CSS variables / theme
- 3D models / fonts → ...

## Verification
- [ ] Runs locally, 0 console errors
- [ ] Screenshots compared against original (RECON/screenshots/)
- [ ] `route-crawl.mjs` outputs original/clone route maps (required for multi-page sites)
- [ ] `interaction-probe.mjs` outputs hover/click/scroll/canvas state evidence (required for interactive sites)
- [ ] `visual-diff.mjs` outputs a screenshot diff report (when feasible)
- [ ] `audit-clone.mjs` outputs a residue audit report
- Points that can't be verified (record honestly, don't fake): ...
```

## TEARDOWN.md (added for complex interactive sites)

Technical teardown doc, **every conclusion cites a real source line number**. Structure:
- **0. One-sentence essence**
- **A. Real technical teardown** (by pillar: rendering/compositing/physics/interaction/audio, each cites a line number **+ evidence level `SOURCE`/`PARTIAL`/`GUESS`**, see `effect-extraction.md`)
- **B. Secondhand-analysis check table** (if there's an AI analysis doc: `claim | real source | accuracy ✅/⚠️/❌ | notes`, focus on catching substantive errors)
- **C. Transferable methodology** (general patterns / site-specific / clone path)

Example: `~/projects/website-clones/marbles-clone/TEARDOWN.md`.

## RECON/ (reconnaissance deliverables)
- `screenshots/original-{1440,768,390}.png` original at three breakpoints
- `screenshots/clone-1440.png` clone shot (for comparison)
- `screenshots/visual-diff-1440.png` screenshot diff image (when feasible)
- `global-recon.json` recon probe output (framework/canvas/scroll library/fonts, etc.)
- `routes/original-route-map.json` / `.md` original's internal route map and per-route screenshots
- `routes-clone/clone-route-map.json` / `.md` clone's internal route map and per-route screenshots
- `interactions/original-interactions.json` / `.md` original's interaction states, network, console, screenshot evidence
- `interactions-clone/clone-interactions.json` / `.md` clone's interaction states, network, console, screenshot evidence
- `asset-manifest.json` original's asset manifest and download status (when feasible)
- `network/original-network.json` API/XHR capture manifest (required for SPA/SaaS)
- `network/fixtures/` saved JSON/text responses
- `sourcemaps/sourcemap-manifest.json` source map search results (do first for complex frontends)
- `visual-diff-1440.json` pixel diff metrics
- `design-dna.json` structured design identity (produced in visual clone / content overhaul modes, kicked off by `dna-scaffold.mjs`, completed manually; schema see `design-dna.md`)
- `baseline/` "minimal as-is reproducible" deliverable + evidence pack for WebGL/effect reverse-engineering (see the baseline-first gate in `effect-extraction.md`)

## CLONE_REPORT.md (added when reporting externally or evaluating skill effectiveness)

```markdown
# <site> · Original vs. Clone Evaluation Report

## Conclusion
- Complexity level:
- Clone mode:
- Overall fidelity:
- Best used for: local learning / further overhaul / deployable showcase / technical teardown only

## Comparison
| Dimension | Original | Clone | Conclusion |
|---|---|---|---|
| Information architecture |  |  |  |
| Visual language |  |  |  |
| Animation/interaction |  |  |  |
| Responsiveness |  |  |  |
| Content replacement |  |  |  |
| Functional boundary |  |  |  |

## Scoring
Score across the 8 dimensions in the web-clone skill's `references/assessment.md`.

## Known Gaps
- 

## Next-step Upgrade Suggestions
- 
```

## CLONE_AUDIT.md (added before going live)

Generated by `scripts/audit-clone.mjs`. Focus on:
- Whether tracking scripts / analytics pixels have been removed
- Whether the original's brand name, Japanese text, or TODOs remain
- Whether external URLs still point to the original site or uncontrollable third-party resources
- Whether asset and license risks have been recorded

## Wrap-up
- Update the hub `~/projects/website-clones/README.md` index row (status emoji: 🟡recon / 🟢running / 🔵remodeling / ✅live / 🔴stuck / 🗂️archived)
- Keep the original source as a read-only baseline `index-original.html`, don't edit on top of it
- Shut down the local server process when done
