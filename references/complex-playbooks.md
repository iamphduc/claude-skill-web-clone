# Complex Site Cloning Playbooks

**English** · [中文](complex-playbooks.zh-CN.md)

For L4-L6 sites; the goal is to break down "looks complex" into an executable path.

## L4 Animated Brand Sites

1. `recon-site.mjs` captures three-tier screenshots, sections, and video/canvas counts.
2. Record scroll-library signals: GSAP / Lenis / ScrollTrigger / Locomotive.
3. Split the page into five categories: hero, scroll narrative, video masks, hover, and transitions.
4. Prioritize reproducing the rhythm and visual grammar; micro-interactions can be approximated, but differences must be flagged in `CLONE_REPORT.md`.
5. Use `visual-diff.mjs` to score screenshots of the hero and key scroll positions.

## L5 WebGL / Canvas / Three.js

1. First look for the real source code, source maps, and public repos; only reverse-engineer the bundle if none can be found.
2. `recon-site.mjs` records canvas dimensions, counts, and framework signals.
3. `sourcemap-hunt.mjs` attempts to download source maps.
4. Break down the technical pillars: rendering, shaders, post-processing, physics, interaction, audio, and asset loading.
5. Don't blindly write complex shaders; first build a minimal runnable scene, then add materials, lighting, and post-processing one at a time.
6. Interaction verification must use actual browser operations or screenshot/video evidence — you can't rely on reading the code alone.

## L6 SaaS / E-commerce / Login-based Business Systems

By default, only clone the presentation layer and demonstrable flows; do not promise real accounts, payments, orders, or permissions.

1. For pages behind login, first confirm authorization and privacy boundaries.
2. `network-capture.mjs` saves XHR/fetch responses as fixtures.
3. Split the APIs into: content APIs, search/filter, user state, and transaction/write.
4. Content APIs can use local JSON stand-ins; transaction/write APIs only mock success/failure states.
5. Preserve empty, loading, error, and insufficient-permission states — don't just do the happy path.
6. `audit-clone.mjs` scans for external links, original-brand residue, and tracking scripts.

## Multi-page / CMS / Enterprise Sites

1. First capture the sitemap, navigation, footer, and main templates.
2. Only clone representative templates: home, list, detail, search/filter, and contact pages.
3. Generate repeated content from data files rather than hand-writing each page.
4. Don't clone the CMS backend; if editing capability is needed, substitute local JSON/Markdown.

## Success Criteria

- Original-site evidence is complete: screenshots, recon JSON, network manifest, and asset manifest.
- The clone runs locally, with console/page errors cleared to zero or clearly explained.
- `CLONE_REPORT.md` has scores for structure, visuals, interaction, responsiveness, and functional boundaries.
- `CLONE_AUDIT.md` has no tracking scripts, original-brand residue, or unexplained Japanese-text/external-link risks.
- Clearly document boundaries for what can't be done: real backends, proprietary APIs, and copyrighted assets.
