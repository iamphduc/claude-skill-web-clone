# 1:1 Faithful Clone of Static-Built Sites: Full Asset Mirror

**English** · [中文](static-mirror.zh-CN.md)

> Applies to: **Astro / Vite SSG / Hugo / Eleventy / any site whose client runtime is emitted as downloadable static assets**—even a WebGL/Canvas/Gaussian-splat front-end-heavy one.
> Does not apply to: true server-side rendering / data-driven SPAs (business data behind an API) → use `network-capture.mjs` to stand in for the API, not this route.

## Core insight (why this achieves 1:1)
For these sites the "**real source is not on GitHub**", but **the deployed static assets are the truth**: HTML + bundled JS + CSS + binaries the runtime fetches (`.sog`/`.buf`/`.wasm`/`.riv`/fonts/images/video). Mirror these **as-is** and serve them from the web root, and you're running **real code + real assets**, not a rebuild—so it's byte-for-byte 1:1, reproducing even the original site's bugs/quirks.

This is the "real source above all" iron law extended to static sites: **for a static site, "get the real source" = "mirror the entire deployed asset set"**.

> ⚠️ Decision-tree trap: don't see `astro:true` and immediately go "hunt the source theme on the theme marketplace"—that only holds for sites **using an off-the-shelf open-source theme**. **A custom Astro site (e.g. Lusion's oryzo.ai) has no theme you can buy**; the right answer is the full mirror described here.

## Why "real browser, full scroll capture" is mandatory—grep / wget won't do
- Binaries like `.buf`/`.sog`/`.riv` are **fetched by the JS runtime according to scroll progress**, and their URLs are often **assembled dynamically** in code → grepping the bundle won't catch them all, and `wget --mirror` can't discover them either (it only follows static links in the HTML).
- The only reliable way: **real browser load + scroll from top to bottom**, record every **network request that actually happens**, then mirror according to this "actual request list".

## One-shot script
```bash
node /Users/jane/.shared-skills/web-clone/scripts/mirror-site.mjs \
  --url https://<site>/ \
  --out ~/projects/website-clones/<site-name>-clone
```
Outputs:
- `<out>/site/…`: mirrored **same-origin** assets (paths preserved; directory URLs stored as `index.html`)
- `<out>/own-asset-urls.txt`: same-origin asset list
- `<out>/third-party.json`: third-party hosts + hints for **webfont CSS that needs self-hosting** (Typekit/Google)
- `<out>/mirror-manifest.json`: all requests + statuses

The script uses a real browser for full-scroll capture + downloads via the browser network stack (cookies/TUN/proxy identical to the page).

## Manual finishing after mirroring (to make it run offline 1:1)
The script only moves same-origin assets and doesn't rewrite automatically—**handle third parties manually per `third-party.json`**:

1. **Self-host domain-locked fonts (most common = Adobe Typekit)**
   A Typekit kit locks to authorized domains, so a remote `@import` may not render after the domain changes → self-host:
   ```bash
   # ① Download the kit CSS (direct connection; typekit is often blocked by proxies → do NOT go through a proxy)
   curl -sL -A "Mozilla/5.0 …Chrome…" -e "https://<site>/" "https://use.typekit.net/<kit>.css" -o site/typekit/kit.css
   # ② Pull the use.typekit.net/af/... font URLs out of kit.css's @font-face src, download each into site/typekit/fonts/
   #    Same font, 3 suffixes: /l=woff2  /d=woff  /a=otf (go by file magic wOF2/wOFF/0x00010000, don't trust the filename)
   # ③ Write local @font-face (relative url + keep the format hint), see below
   ```
   Local `@font-face`:
   ```css
   @font-face{ font-family:"<same-name>"; src:url("./fonts/x.woff2") format("woff2"),
     url("./fonts/x.woff") format("woff"), url("./fonts/x.otf") format("opentype");
     font-display:swap; font-weight:<original-range>; }
   ```
   Then change the references to local—**note that Typekit is often the first line of the main CSS `@import"https://use.typekit.net/<kit>.css"`, not an HTML `<link>`**:
   ```bash
   perl -0pi -e 's{\@import"https://use\.typekit\.net/<kit>\.css"}{\@import"/typekit/kit-local.css"}g' site/_astro/<main>.css
   ```

2. **Remove tracking**: cloudflare beacon / GA / pixels—cut the `<script>` precisely.

3. **Public CDNs (Rive wasm@unpkg / third-party player)**: public CDNs load fine cross-origin online, and work when serving locally + connected → you can leave them online (they break offline; note it in NOTES). To go fully offline, mirror them too and rewrite the injection point.

4. **Vimeo/YouTube embeds**: the iframe plays online, unavailable offline → usually not core above-the-fold; just note it in NOTES.

## Serve + verify
```bash
cd ~/projects/website-clones/<site-name>-clone/site
python3 -m http.server 8124      # must serve site/ as the web root so root-relative paths (/_astro /models …) resolve
```
Then follow SKILL.md Step 5: 0 console errors in the browser + `visual-diff.mjs` pixel comparison against the original site. For WebGL-heavy sites, remember to **screenshot at each section as you scroll** for comparison (a static full-page screenshot won't capture scroll-triggered GL frames).

## worked example: oryzo.ai (Lusion, L6)
- 135 same-origin assets (HTML+bundle+CSS + 25×`.buf` geometry/camera animation + 2×`.sog` Gaussian splats + sorting wasm + `.riv` + MSDF + fonts + 80+ images)
- Only rewrites: Typekit `@import`→local self-hosted halyard + removed cloudflare beacon
- Result: `scrollHeight` exactly matches, **0 console errors**, hero pixel diff **36/1.3M (5/5)**
- Left online: Vimeo gallery video + unpkg Rive wasm (non-core)
- Full record: `~/projects/website-clones/oryzo-clone/` (NOTES.md + TEARDOWN.md)
