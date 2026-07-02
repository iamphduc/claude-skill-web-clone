# Evidence Discipline + Baseline Gate for Effect Extraction (WebGL/Canvas reverse-engineering branch reinforcement)

**English** · [中文](effect-extraction.zh-CN.md)

`reverse-engineering.md` covers "how to read a rendering architecture"; this piece covers **how not to fool yourself when reverse-engineering effects**,
plus the fallback route when the real source can't be found. The three-piece set: **evidence grading → no-compensation → baseline-first gate**.

> The discipline paradigm is inspired by [lixiaolin94/skills · web-shader-extractor](https://github.com/lixiaolin94/skills) (that repo has no LICENSE,
> so all rights reserved by default—here we **only borrow the methodological concepts, rewrite everything in this skill's own words, and copy none of its code or original text**).
> It shares the same spirit as web-clone's number-one iron law "real source above all", just upgrading "cite line numbers" into a systematic evidence regime.

## 1. Evidence grading (tag every conclusion)

When writing the TEARDOWN, tag every key fact in the render pipeline with a level, and **default to the lowest level**:

| Tag | Meaning | Examples |
|---|---|---|
| `SOURCE` | Direct, hard evidence bound to the target | Public real-source lines, modules restored from source-map, runtime object dumps, captured shader/WGSL text, frame captures, network response bodies with hashes |
| `PARTIAL` | A handle for the next probe, not yet conclusive | Class/function/field names, minified bundle slices, framework objects, got the shader but missing uniform/pass/input state |
| `GUESS` | Reconstructed value with no direct evidence | Visual fitting, name-based inference, applied defaults, hand-tuned magic numbers, any "looks right" behavior reconstruction |

- **Untagged = treat as GUESS.** Don't let unevidenced things slip into "known".
- This institutionalizes the marbles lesson ("the AI fabricated analytic intersection into ray-marching"): **any GUESS-level implementation must be upgraded to SOURCE before it's copied verbatim.**

## 2. no-compensation (don't mask lack of understanding by tuning parameters)

> **Strictly forbidden**: changing brightness / speed / position / noise values just to make the image "look right", masking real errors in timing, color, FBO, resources, coordinate system, or state model.

- A fitted constant made the output more similar → **it's still a GUESS**, and write down "what evidence is needed to upgrade it".
- Wiring-type facts (pass order, coordinate transforms, time units, input coupling) **do not become correct just because the image looks right**; they must be independently traced down to evidence.
- Corresponds to the old web-clone rule: write down truthfully what you can't verify, **don't fake a "drag succeeded"**.

## 3. baseline-first gate (reproduce first, then refactor)

The **most common mistake** when reverse-engineering effects is extracting, rewriting, and beautifying all at once, ending up neither resembling the original site nor able to say which step went wrong. Instead, split into gates:

```
Locate the render surface → capture the minimal truth → RAW REPLAY (minimal as-is reproduction) → ✅BASELINE frame-by-frame comparison passes
                                                                                                    ↓ only allowed after passing
                                                                                              PROJECTIZE (refactor into an editable project) → PACKAGE
```

- **RAW REPLAY**: use the captured real draw calls / shaders / uniforms / vertex data to build an **as-small, as-as-is as possible** runnable reproduction—no optimizing, no swapping frameworks, no changing parameters.
- **BASELINE gate**: the RAW REPLAY must be frame-by-frame (or multi-frame sampled) visually identical to the original site to count as passing. **Without passing this gate, refactoring is not permitted.**
- Only after passing the gate, PROJECTIZE: switch to a maintainable form (raw WebGL / Three.js TSL / Babylon etc.), still tagging each spot's evidence level.
- Three closing states, tag them truthfully: `DONE_BASELINE_VERIFIED` (reproduced and verified) / `DONE_PROJECTIZED` (projectized) / `DONE_BASELINE_WITH_GAPS` (reproduced but with documented gaps).

## 4. When the real source can't be found—runtime capture fallback

web-clone's first move is always "go to GitHub / source-map for the real source". But **effect sites are often sourceless, minified to the bone**.
At that point don't fall back to "write it to look similar" (that's GUESS); instead **grab the runtime truth at the render boundary**:

- Intercept on the WebGL/WebGPU context: the actual draw calls, bound programs, compiled shader source, uniform values, FBO/texture sizes, blend/depth state.
- Tooling direction: spector.js-style frame capture, patching the `WebGLRenderingContext` prototype to log calls, `getShaderSource` to get the compiled shader, a preload script to inject hooks before the page scripts run.
- What you capture counts as `SOURCE` level—it is the new "real source", fed into the baseline-first flow.

## 5. When to delegate to web-shader-extractor

If you've already installed `web-shader-extractor` (the skill from `npx skills add lixiaolin94/skills`),
**in the following situations delegate the effect portion straight to it, with web-clone acting only as the overall entry point**:

- The site is WebGL/WebGPU/heavy-Canvas effects, and neither GitHub nor source-map yields the real source;
- You need runtime frame capture, frame-by-frame comparison, shader/uniform-level extraction;
- You want the full gated flow of "baseline reproduce first, then projectize independently".

Once the delegated output (a minimal reproducible baseline + evidence pack) comes back, **merge it into `~/projects/website-clones/<site-name>-clone/`**,
fill in NOTES/TEARDOWN per web-clone's output conventions, and continue with Step 5 verification + Step 6 replacement.

Not having it installed doesn't matter either: the discipline of the four sections above can be followed as-is; web-shader-extractor just packages the capture machinery of section 4 into a ready-made tool.

## Integration with existing outputs

- Add an evidence tag (`SOURCE`/`PARTIAL`/`GUESS`) to each item in TEARDOWN.md's "A. Real technical teardown".
- "B. Second-hand analysis check table" is already about catching GUESS masquerading as SOURCE—same framing.
- Keep the baseline reproduction under `<site-name>-clone/RECON/baseline/`, alongside the original screenshots as ironclad proof of "verified".
