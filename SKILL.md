---
name: web-clone
description: >
  网站复刻 / 克隆方法论。USE WHEN 用户说 复刻网站、克隆网站、clone website、抄个站、仿站、
  照着这个站做一个、reproduce site、还原某个网页效果、把这个站搬下来改成我的、
  复刻某个交互/WebGL/Canvas/Three.js 效果。提供「先拿真源码 → 判路径 → 逆向拆解 →
  搭工程 → 替换内容」的可移植决策树，覆盖静态站 / React-Vue-Next 内容站 /
  WebGL-Canvas 重前端站三大分支，并强制核对任何 AI 二手分析里的可执行代码。
metadata:
  author: jane (xiaoer)
  version: "1.0.0"
  use_case: 个人本地复刻/学习网站，沉淀自 website-clones 克隆中枢
---

# Web Clone · 网站复刻方法论

把"复刻一个网站"做成可重复的流程。配套工程区：`~/projects/website-clones/`（每个复刻是一个子目录）。

## 头号铁律：真源码至上，绝不信 AI 推测的代码

> 任何 AI 生成的"复刻分析/施工图"，**正文的概念骨架可以参考，但里面的可执行代码块默认全是臆造的**，必须逐行用真源码核对，否则照抄必崩。

**实证（marbles 案例）**：一份 AI 分析文档把原站"解析法求光线-球体交点 + 把光学结果编码成位移图、交给 SVG `feDisplacementMap` 去扭曲真实 DOM"的真架构，臆造成了"ray-marching + SDF + 把 DOM 当纹理采样"——两套完全不同的实现，照抄做不出原效果且慢 N 倍。详见 `references/marbles-case.md`。

所以第一动作永远是：**拿到真源码**。

## 决策树（按顺序走，不许跳）

### Step 1 · 先去 GitHub 搜源码，别急着抓站

```bash
unset SSL_CERT_FILE   # macOS 怪癖，bash 前先解
# 按站名/产品名搜
SSL_CERT_FILE=/etc/ssl/cert.pem gh api "search/repositories?q=<关键词>" \
  | jq -r '.items[] | "\(.full_name) ⭐\(.stargazers_count) \(.description)"' | head -10
# vercel.app/netlify.app/github.io 的 URL slug 常常就是 仓库名 / 部署者用户名
```
- 单文件站（github.io / 纯 HTML）→ 直接抓 raw：`curl -sL https://raw.githubusercontent.com/<user>/<repo>/main/index.html`
- **找到源码且许可允许 → 跳 Step 4 直接 clone。** 教训：先搜 GitHub 能省 30 分钟弯路。

### Step 2 · 没找到源码 → 浏览器侦察（探针）

加载 `Browser` skill 或 playwright MCP，跑探针抽信号（框架 / `window.THREE` / canvas 数 / 平滑滚动库 / 字体 / scrollHeight）。截图 1440/768/390 三档 + 侦察 JSON 存 `<站>/RECON/`。

> 登录态私域站才用 `bb-browser`；localhost / 无登录公开站用 `Browser` skill 或 playwright（别动 bb-browser，会抢 Brave）。

### Step 3 · 按侦察结果选路径

| 侦察结果 | 走哪条路 |
|---|---|
| 静态 HTML/CSS，无框架 | `wget --mirror` 抓镜像 → 删追踪脚本 → 改文案 |
| React / Vue / Next（内容为主） | 重建模板（如 `ai-website-cloner-template`，Node 24+），灌内容 |
| **WebGL / Canvas / Three.js 重前端** | **深度逆向真源码（见下）→ 忠实复刻 或 找同类开源 3D 模板换内容**。单文件原生站常常逐字节保留=最忠实复刻 |
| 大型开源生态（Astro/Hugo 主题） | 去对应主题市场找源主题 |

### Step 4 · 在克隆中枢里搭工程

```bash
mkdir ~/projects/website-clones/<站名>-clone && cd $_
# git 源码：clone 进来；单文件：放进来。原始源码留一份只读基准 index-original.html
# 检查 Node 版本（package.json engines），nvm use 对应版本，钉 .nvmrc
```

### Step 5 · 删追踪 + 写元信息 + 验证

- **删追踪**：Google Analytics（`gtag` / `googletagmanager`）、像素、热图——逐行精确切除（GA 块常在 `<head>` 顶部）。
- **写 NOTES.md**（必须）+ 复杂站写 TEARDOWN.md（技术拆解）。模板见 `references/deliverables.md`。
- **浏览器真验证**（硬要求，不许只看代码就说"应该能跑"）：起本地服务器 → 浏览器打开 → 抓 console（不能有 JS/WebGL 编译错误）→ 截图对照原站。诚实记录验证不了的部分（如合成 PointerEvent `isTrusted=false` 触发不了拖拽，要如实写，别伪造"拖动成功"）。

### Step 6 · 替换成 Jane 自己的东西

目标永远是"做她自己的站"，不是搬一个一模一样的。替换三件套：文字（`index.html`/`data/*.json`/`content/*.md`）、媒体（`public`/`assets`）、品牌色（CSS 变量 / Tailwind theme）。结构非平凡就写 REPLACE_GUIDE.md。

## 逆向拆解 WebGL/Canvas 重前端（核心手艺）

把交互站拆成**技术支柱**，逐柱定位真实实现 + 标行号：渲染（WebGL/着色器算法）、合成（SVG filter / 多 canvas / 后期）、物理、交互、音频。然后才去核对任何二手分析。

**可迁移的高级范式**（值得攒着）：
- **位移图折射 DOM**：离屏 WebGL 算出 RG=位移/B=菲涅尔的"位移图"，再用 SVG `<filter><feDisplacementMap scale=N>` 拿它去扭曲真实的、活的、可交互的 HTML——折射的是真 DOM，WebGL 全程不碰 DOM 像素。这是 marbles 的灵魂，也是 Three.js `MeshPhysicalMaterial(transmission)` 做不到的事（它只能做"玻璃球外观"，做不出"折射整个网页"）。

详细方法 + marbles 真架构逐项拆解 → `references/reverse-engineering.md`、`references/marbles-case.md`。

## 许可与署名（clone 前必查）

```bash
SSL_CERT_FILE=/etc/ssl/cert.pem gh api repos/<u>/<r> | jq '.license'  # + 找 LICENSE 文件 + 看 README
```

| 许可 | 能做什么 |
|---|---|
| MIT / Apache / BSD / Unlicense | 可改、可上线，保留致谢即可 |
| **NONE（无 LICENSE 文件 / 未声明）** | **默认保留所有权利**。仅本地学习/复刻，须署名原作者，**未经许可不得公开重新部署**。别因为代码公开就当成可自由用 |
| 专有 / 明确禁止 | 只读学习，不复制不部署 |

⚠️ 别把"GitHub 上是公开的"或"gh api 一时查不到"等同于 MIT——核实到底。

## 产物规范
- 每个子项目根目录：`NOTES.md`（源信息/技术栈/license/替换地图/跑起来命令）
- 复杂交互站追加：`TEARDOWN.md`（技术拆解，标行号）
- `RECON/screenshots/`：原站 vs 克隆对照图
- 复刻完更新中枢 `README.md` 索引行

## 旗舰案例
`~/projects/website-clones/marbles-clone/` — 原生 WebGL + SVG Filter + 自研物理的玻璃弹珠站，逐字节忠实复刻 + 完整 TEARDOWN，是"WebGL 重前端分支"的范例。
