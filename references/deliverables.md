# 产物规范与模板

每个复刻子项目（`~/projects/website-clones/<站名>-clone/`）的标准产物。

## NOTES.md（必须）

```markdown
# <站名> · 克隆笔记

## 源信息
- 原站 URL:
- 源码仓库:（如有）
- 原作者:
- 许可证: MIT / Apache / NONE / 专有  ← 必须核实，见 SKILL.md 许可表
- 致谢要求:

## 技术栈
- 框架 / 关键库 / Node 版本:

## 跑起来
\`\`\`bash
cd ~/projects/website-clones/<站名>-clone
# 单文件静态站: python3 -m http.server 8123
# 框架站: nvm use <ver> && npm install && npm run dev
\`\`\`

## 改了什么（对照原版）
- 删了追踪脚本: ...（GA/gtag 行号）
- ...

## 替换地图（要换什么改哪）
- 文字 → 文件 行
- 图片/媒体 → 目录
- 配色 → CSS 变量 / theme
- 3D 模型 / 字体 → ...

## 验证
- [ ] 本地跑通、console 0 error
- [ ] 截图对照原站（RECON/screenshots/）
- 验证不了的点（如实记，别伪造）: ...
```

## TEARDOWN.md（复杂交互站追加）

技术拆解文档，**所有结论标真源码行号**。结构：
- **0. 一句话本质**
- **A. 真实技术拆解**（按支柱：渲染/合成/物理/交互/音频，每条标行号）
- **B. 二手分析校验表**（若有 AI 分析文档：`声称 | 真源码 | 准确度✅/⚠️/❌ | 说明`，重点揪实质性错误）
- **C. 可迁移方法论**（通用套路 / 本站特有 / 复刻路径）

范例：`~/projects/website-clones/marbles-clone/TEARDOWN.md`。

## RECON/（侦察产物）
- `screenshots/original-{1440,768,390}.png` 原站三档
- `screenshots/clone-1440.png` 克隆图（对照）
- `global-recon.json` 侦察探针输出（框架/canvas/滚动库/字体等）

## 收尾
- 更新中枢 `~/projects/website-clones/README.md` 索引行（状态 emoji：🟡侦察/🟢跑通/🔵改造/✅上线/🔴卡住/🗂️归档）
- 原始源码留只读基准 `index-original.html`，不要在它上面改
- 跑完关掉本地服务器进程
