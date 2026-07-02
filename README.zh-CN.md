# 🪞 Web Clone · 网站复刻方法论

> 把任何一个网站(从单文件静态页到 WebGL 重前端 demo)忠实复刻出来 —— 不靠 AI 臆测代码硬抄。

[English](README.md) · **中文**

---

## 这个 skill 是干什么的

你看到一个喜欢的网站,想复刻 —— 学它怎么做、改成你自己的版本、或者离线跑起来。AI 工具最爱产出**看起来煞有其事**的"复刻分析文档",里面的代码块几乎全是臆造的,照抄必崩。

这个 skill 把"真源码至上"做成可重复的流程,**6 步决策树**覆盖三大分支:

- 静态 HTML/CSS 站 → `wget --mirror`
- React / Vue / Next 内容站 → 重建模板灌内容
- **WebGL / Canvas / Three.js 重前端** → 逆向真源码,逐行核对;找不到源码时用运行时帧捕获 + **baseline-first 复现闸门** + `SOURCE/PARTIAL/GUESS` 证据分级
- **静态构建站(Astro/Vite SSG/Hugo),哪怕重 WebGL** → `mirror-site.mjs` 全程滚动捕获 + 镜像每一个部署资产(含运行时 fetch 的 `.sog/.buf/.wasm`),做真 1:1 忠实复刻——对静态站,"拿到真源码"="镜像部署资产整套"(范例:Lusion oryzo.ai,高斯泼溅,hero 像素 diff 5/5)
- **视觉复刻 / 内容爆改模式** → 把"那个站的感觉"蒸馏成可版本化的 `design-dna.json`(设计 token + 风格 + 特效),然后"DNA 留着、内容换掉"
- 浏览器真验证,如实记录,最后替换成你自己的内容

## 头号铁律:真源码至上,绝不抄 AI 臆造代码

> 任何 AI 写的"复刻分析",**概念骨架可以参考,但里面的可执行代码块默认全是臆造的**,必须用真源码核对,否则照抄必崩。

**实证案例**:一份 AI 分析文档把原站"解析法求光线-球体交点 + 把光学结果编码成位移图 + 交给 SVG `feDisplacementMap` 扭曲真实 DOM"的真架构,臆造成了"ray-marching + SDF + 把 DOM 当纹理采样"—— 两套完全不同的实现,照抄做不出原效果且慢 N 倍。详见 `references/marbles-case.md`。

## 安装

这是一个 [Claude Code skill](https://docs.claude.com/en/docs/agents-and-tools/skills)。clone 到你的 skills 目录:

```bash
git clone https://github.com/Jane-xiaoer/claude-skill-web-clone.git ~/.claude/skills/web-clone
```

然后跟 Claude / Codex / Cursor 说:
- 「复刻这个网站: https://...」
- 「把这个站抄下来改成我的」
- 「逆向拆解这个 WebGL demo」

AI 会自动加载 `SKILL.md` 走决策树。

## 旗舰范例

`references/marbles-case.md` 拆解了 [chiuhans111/marbles](https://github.com/chiuhans111/marbles) —— 1067 行单文件 WebGL + SVG filter 玻璃弹珠站,真架构对比 AI 分析文档的错误,是"为什么必须真源码核对"的标准教材。

---

## 致谢

- **方法论沉淀来源**:Jane 的克隆中枢 `~/projects/website-clones/` 工作流(2026-05-27 marbles 案例跑通后抽出 skill)
- **旗舰范例原作**:[chiuhans111/marbles](https://github.com/chiuhans111/marbles) by Hans Chiu —— 没有这份单文件 1067 行的玻璃弹珠源码就没有这个 skill 的"真源码至上"铁律
- **设计身份层 schema**:`references/design-dna.md` 的三维 DNA JSON 改编自 [zanwei/design-dna](https://github.com/zanwei/design-dna)(MIT),用于"视觉复刻 / 内容爆改"模式
- **特效提取纪律**:`references/effect-extraction.md` 的证据分级 + baseline-first 闸门受 [lixiaolin94/skills · web-shader-extractor](https://github.com/lixiaolin94/skills) 启发(该仓库无 LICENSE,本 skill 只借方法概念、用自己的话重写,未复制其代码或原文)
- **迭代打磨**:Jane(`@xiaoerzhan`) + Claude Code(多轮 reverse-engineering)

---

## License

MIT. 自由使用、二次创作、发布皆可。欢迎署名,但非必须。

如果这个 skill 让你免于在 AI 臆造代码上浪费一个下午,欢迎 ⭐ 这个仓库。

---

## 📱 关注作者

如果这个仓库对你有帮助,欢迎关注我。后面我会持续更新更多 AI Skill、设计方法、网站美学和创意工作流。

- X (Twitter): [@xiaoerzhan](https://x.com/xiaoerzhan)
- 微信公众号:扫码关注

<p align="center">
  <img src="./follow-wechat-qrcode.jpg" alt="Jane WeChat Official Account QR code" width="300" />
</p>

<p align="center">欢迎关注我的公众号,一起研究 AI Skill、设计原则、网站表达和创意工作流。</p>
