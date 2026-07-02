# Flagship Case · Glass Marbles (Real Architecture vs AI Fabrication)

**English** · [中文](marbles-case.zh-CN.md)

Source site https://chiuhans111.github.io/marbles/ · author Hans Chiu · single file, 1067 lines · license **NONE**.
For the full line-by-line teardown see `~/projects/website-clones/marbles-clone/TEARDOWN.md`; this is the condensed version for the skill.

## The Real Architecture in One Sentence

A full-screen WebGL fragment shader uses the **analytic method (quadratic equation `b*b-c`)** to find the intersection of rays with the sphere, performs up to 4 refraction/reflection iterations, and encodes the optical result into a **displacement-map PNG** (RG = pixel displacement, B = Fresnel value); then an SVG `<filter>`'s `feDisplacementMap(scale=200)` uses this image to distort the **real, live, interactive web DOM** (background color blocks + heading text).

> The glass marble you drag is essentially a lens, and behind the lens is this HTML page. WebGL never touches DOM pixels at all — the refraction is done by the SVG filter. Physics and audio are both pure hand-written, zero libraries.

## The Three Pillars (Real Implementation Points)

1. **WebGL optics**: analytic sphere intersection (not ray-marching); refractive index N=1.3, 4 iterations; Fresnel `0.05+0.95*pow(1-cosθ,2.0)` (exponent 2, not Schlick's 5); 2 internal bubbles + a parabolic colored core + Beer-Lambert volumetric absorption; **one shader reused via `u_mode` (0 refraction / 1 reflection / 2 foreground highlight / 3 shadow)**; the displacement encoding `DISPLACEMENT_SCALE=200` is strictly aligned with the SVG side.
2. **SVG filter compositing**: 4 canvases (1 main + 3 offscreen), the offscreen images fed each frame via `toDataURL` into `<feImage>`. The real chain: shadow `feGaussianBlur(8)` → `feBlend multiply` onto the DOM → refraction `feDisplacementMap` → reflection `feDisplacementMap` → `feGaussianBlur(2)` → the reflection image's B channel (Fresnel) turned into an alpha mask via `feColorMatrix` → `feComposite` in two steps. The refracted `SourceGraphic` is the `#container` that has `filter:url(#marble-filter)` applied.
3. **Physics**: pure hand-written, `mass=r³`, gravity 0.8, 3D elastic collisions (restitution 0.8, solved only when nearby), ground restitution 0.55 + tiny-bounce zeroing, quaternion rolling, drag-lift targetZ=200; `settleFrames` completely stops rendering when at rest.
4. **Audio**: Web Audio procedural synthesis (zero files), base frequency `800+(60-r)*20` + 5 overtones, volume scaling with collision speed.

## ⚠️ Where the AI Analysis Document Went Wrong (a cautionary tale; remember these pitfall types)

That `弹珠网站复刻分析.md` document's main text has a **basically correct conceptual skeleton** (the 8 steps and three-pillar direction are all right), but the **accompanying "cloning code blocks" are almost entirely fabricated**:

| Fabrication | The Truth | Pitfall Type (general warning) |
|---|---|---|
| ray-marching + SDF + `MAX_STEPS=100` + 6-tap finite-difference normals | analytic intersection, normal `normalize(rp-center)` | **Don't intuitively assume a refraction demo uses ray-marching** — spheres have a closed-form solution, and many demos use the analytic method, which is both fast and accurate |
| `sampler2D uBackground` sampling the DOM as a texture | the shader doesn't read the background; refraction is handed off to SVG via the displacement map | **Got the GPU↔DOM hierarchy backwards** — the most core architectural idea was dropped |
| `feBlend screen` + a single displacement + `feComposite over` to composite the shadow | double displacement + Fresnel mask + multiply shadow | **A second-hand analysis's filter chain is untrustworthy; verify it node by node against the real source** |
| `MARBLE_COUNT=5`, arrays sized 10 | hard-coded 2 | even the constants were guessed |
| screen-center NDC coordinates | top-left pixel + Y flip | the coordinate-system conventions were entirely conjectured |

**Lesson**: an AI-written "cloning blueprint" = refer to its skeleton of ideas, but **don't directly copy a single line of its code blocks** — you must verify against the real source. This is the origin of this skill's number-one iron rule.
