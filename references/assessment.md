# Complexity Grading and Reproduction Scoring

**English** · [中文](assessment.zh-CN.md)

Used for pre-reproduction prediction, post-reproduction acceptance, and explaining to Jane "how faithfully this site can be cloned."

## Reproduction modes

| Mode | Goal | Applicable scenario |
|---|---|---|
| Faithful reproduction | Keep the original source / original layout / original interactions as much as possible | Found legal source, single-file site, learning complex front-end techniques |
| Visual reproduction | Looks close, internal implementation is replaceable | No source, commercial official sites, content sites, componentized rebuild |
| Content overhaul | Keep the original site's rhythm and visual grammar, swap in Jane's business content | Turn a reference site into your own site, brand page, product intro |
| Technical breakdown | Don't rush to rebuild; first confirm the real implementation | WebGL/Canvas/complex interactions, contradictory AI analyses |

## Complexity L1-L6

| Level | Type | Typical signals | Usual reproducibility | Default boundary |
|---|---|---|---|---|
| L1 | Static HTML/CSS | Little JS, no framework, few pages | 90-98% | Can approach pixel level; asset copyright is separate |
| L2 | CMS / enterprise content site | Multiple pages, CMS-generated, forms/news/regional sites | 70-90% | Front end can be reproduced; CMS back office not cloned |
| L3 | React/Vue/Next content front end | hydration, chunks, routing, content fetched via APIs | 65-90% | Data/API can use local JSON stand-ins |
| L4 | Animation-heavy brand site | GSAP, Lenis, complex scroll, video masks | 50-80% | Key visuals can be reproduced; microinteractions often approximated |
| L5 | WebGL/Canvas/Three.js | shaders, physics, post-composition, GPU resources | 30-95% | High with source; without source, break down first then decide |
| L6 | SaaS / e-commerce / login business systems | Accounts, payment, orders, permissions, search & recommendation | Presentation layer doable | Server-side business logic not cloned by default |

## Pre-reproduction prediction template

```markdown
## 复刻前预判
- 复杂度等级:
- 推荐模式: 忠实复刻 / 视觉复刻 / 内容爆改 / 技术拆解
- 可高保真的部分:
- 需要近似或替代的部分:
- 不克隆的部分:
- 主要风险: 许可 / 素材 / 登录态 / API / 性能 / WebGL / 响应式
```

## Post-reproduction scoring

0-5 points per item. Only give scores that can be backed by source code, screenshots, or browser run results.

| Dimension | 5 points | 3 points | 1 point |
|---|---|---|---|
| Source evidence | Found source or complete static assets, key conclusions have file/line numbers | Has runtime recon and asset harvesting | Mostly relies on visual estimation and inference |
| Structural fidelity | Information architecture, block order, breakpoints all consistent | Main blocks consistent, details merged/trimmed | Only the general style is preserved |
| Visual fidelity | Fonts, spacing, colors, image ratios highly close | Key visuals close, some ratios differ | Clearly looks like a different design |
| Motion/interaction | Scroll, hover, video, Canvas/WebGL behavior close | Only core interactions kept | Basically static |
| Responsive | Desktop/tablet/mobile all verified with no misalignment | 1-2 widths verified | Obvious layout breakage on mobile |
| Functional completeness | Navigation, forms, media, external links, local run all work | Main browsing paths work | Many dead links or errors |
| Content replacement | Already changed to Jane's content, replacement map is clear | Partially reworked, still has original-site remnants | Lots of original-site copy/brand remnants |
| Legal/deployment risk | Licensing clear, tracking removed, asset boundaries defined | Risks recorded but not fully resolved | Risks unclear |

Recommended output:

```markdown
## 复刻评分
- 源证据: /5
- 结构保真: /5
- 视觉保真: /5
- 动效/交互: /5
- 响应式: /5
- 功能完整: /5
- 内容替换: /5
- 法务/部署风险: /5
- 总评:
```

## Original site vs clone site comparison table

```markdown
| 模块 | 原站表现 | 克隆实现 | 差异 / 取舍 | 证据 |
|---|---|---|---|---|
| 首屏 |  |  |  | screenshot / file:line |
| 导航 |  |  |  |  |
| 核心动效 |  |  |  |  |
| 内容区块 |  |  |  |  |
| 移动端 |  |  |  |  |
```
