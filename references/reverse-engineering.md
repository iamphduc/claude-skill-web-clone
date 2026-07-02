# Reverse-Engineering WebGL/Canvas-heavy Frontend Sites

**English** · [中文](reverse-engineering.zh-CN.md)

When recon determines that a site is a WebGL/Canvas/Three.js-heavy frontend (`window.THREE` exists, or there are multiple `<canvas>` elements, or there is a `<script type="x-shader/*">`), follow this playbook.

> This piece covers **how to read and understand the rendering architecture**; the **evidence discipline** during reverse-engineering (SOURCE/PARTIAL/GUESS grading, no-compensation, baseline-first gating) and the **runtime-capture fallback for when you can't find the source** → `effect-extraction.md`. Use the two together.

## General Teardown Steps

1. **Get the single-file source first.** Many demo sites are fully self-contained in one HTML file (GitHub raw / browser view-source); don't rush to scrape the site with Playwright.
2. **Count canvases, read SVG defs, grep shaders**: How many `<canvas>` elements? Is there a `<filter>`? Where are the shader scripts? — these three determine the rendering architecture.
3. **Determine "what the WebGL is computing"** (crucial; it decides whether to trust second-hand analyses):
   - grep `texture2D` / `sampler2D` → sampling a texture / framebuffer
   - grep `+= dS` / `map(` / `MAX_STEPS` inside a `for` loop → ray-marching (step-based)
   - grep `b*b` / discriminant / `sqrt(` + quadratic equation → **analytic intersection** (closed-form solution, common for spheres/planes)
   - ⚠️ Don't intuitively assume ray-marching — refractive-glass demos are often analytic sphere intersection.
4. **Find the GPU↔DOM bridge**: if the WebGL canvas isn't displayed directly but is `toDataURL`'d / fed to `<feImage>` / `feDisplacementMap`, it means it's generating a **data image** (displacement/normal/depth) for another layer to use. This is a hallmark technique of advanced frontends, and also the layer most easily gotten backwards by second-hand analyses.
5. **Read physics/audio separately**: usually these are pure JS modules decoupled from rendering, and can be verified independently.

## Transferable Advanced Paradigms Worth Collecting

- **Displacement-map refraction of the DOM**: offscreen WebGL computes a PNG with RG = displacement, B = auxiliary quantity, and SVG `<feDisplacementMap scale=N>` uses it to distort real HTML. The `scale` on the GPU side and the SVG side must be aligned. It can produce liquid glass, magnifiers, water ripples, and any "lens laid over the webpage" effect, and **what gets refracted is the live, interactive DOM** — something Three.js `MeshPhysicalMaterial(transmission)` cannot do (it can only produce a "glass-ball appearance," not "refract the entire webpage").
- **One shader + a mode uniform for multiple uses**: refraction/reflection/shadow/foreground share the same fragment shader, branching on `u_mode`. Saves code and compilation.
- **Downscale auxiliary data + skip frames when at rest**: displacement/reflection/shadow images are visually insensitive to resolution → drop to 1/2 or 1/4. When objects are at rest → stop rendering entirely (settleFrames). Standard performance tricks for heavy frontend sites.
- **A full-screen big triangle instead of a quad**: 3 vertices `[-1,-1, 3,-1, -1,3]` cover the full screen, saving one diagonal.

## Cloning-path Decisions

- **Can achieve a 1:1 reproduction + license permits** → directly take the real source and change the copy/colors/parameters (a single-file native site preserved byte-for-byte = the most faithful).
- **Want an approximate effect, don't care about consistency** → find a similar open-source template and swap in the content (e.g. awesome-threejs). But note: **switching the implementation route often loses the original site's soul mechanism** (like this case's "refracting the real DOM"); first confirm whether that mechanism is the selling point the user wants.
- **This site's specific math (intersection formulas / magic numbers) must be rewritten when transferred**: swapping the analytic "sphere" intersection for another shape requires changing the formula (or you may genuinely then need SDF/ray-marching); hand-tuned magic numbers like refractive index, Fresnel coefficients, and absorption must be retuned when changing materials.

## Verification (Hard Requirement)
Start a local server → open in the browser → capture the console (there must be no JS/WebGL compilation errors) → take screenshots to compare against the original site.
Honestly record the parts you can't verify: for example, Playwright's synthesized PointerEvent has `isTrusted=false` and can't trigger drag hit-testing logic — **write this truthfully into NOTES; don't fabricate a "drag succeeded."** For physics-type sites, you can use "the initial state differs across two loads" as indirect proof that the engine is running.
