# SOP流程 — 设计系统文档 (Design System Documentation)

> 本仓库的设计系统文档**仅针对 `sop-index.html`**。
> 修改任何代码前,请依次阅读:`README.md` → `DESIGN.md` → `COMPONENTS.md` → `ANIMATION.md`。

---

## 1. Project Purpose

`sop-index.html` 是一个**自包含的单文件交互式 SOP(标准作业流程)编辑器**。它是一个独立的
HTML 页面(内联 CSS + 内联 JavaScript,**零构建步骤、零外部依赖**),用于记录销售/物流
业务流程,并允许非技术操作员**直接阅读、编辑并重新保存**该流程——无需任何服务器或工具链。

所记录的流程属于一个跑在西班牙语系统(`VENTAS`、FX 收单软件、IPAD 服务器)上、由中文团队
操作的销售业务,因此界面语言为**简体中文**,同时原样保留西班牙语业务术语。

页面以 **模块(Module) → 阶段(Stage) → 步骤卡片(Node/Step) → 子步骤(Substep)**
的四级结构组织流程内容。

> 仓库内另有 `索赔退款流程.html` 与 `索赔退款流程_扁平风.html` 两份文档,**不在本设计系统
> 文档范围内**。本套文档(README / DESIGN / COMPONENTS / ANIMATION)描述的全部规则均
> 来自 `sop-index.html` 的真实实现。

### How persistence works (重要心智模型)

**没有后端,没有数据库。** 页面在编辑*它自己*:

1. 文本通过 `contenteditable` 就地编辑。
2. **保存** (`saveFile()`) 把整个实时 DOM 序列化成 HTML 字符串,触发浏览器下载同名文件
   `sop-index.html`。
3. 重新打开下载的文件即可还原一切——因为保存前会把运行时状态写回 `data-*` 属性
   (每张卡片的 `data-detail`、激活模块 `data-active-module` 等)。
4. **导出 PDF** 即 `window.print()`,配合专门的 `@media print` 样式。

任何对结构或样式的改动都必须能挺过这条
**编辑 → 序列化 → 重新打开 → 重建** 的往返链路。

---

## 2. Folder Structure

```
klmed/
├── README.md                  ← 项目概览 + AI 工作约定(本文件)
├── DESIGN.md                  ← 视觉设计规范(颜色 / 字体 / 间距 / 圆角 / 阴影 / 布局 / 图标)
├── COMPONENTS.md              ← 全部可复用组件目录
├── ANIMATION.md               ← 动效系统规范
├── sop-index.html               ← 本设计系统的唯一来源(application shell)
├── 索赔退款流程.html           ← (不在本文档范围内)
├── 索赔退款流程_扁平风.html     ← (不在本文档范围内)
└── assets/
    └── contabilizacion-stock.mp4
```

`sop-index.html` 内部结构(单文件,全内联):

```
<head>
  <style> … </style>           ← 全部 CSS(约 1100 行):两套 :root 令牌 + 旧外壳 + APP 外壳覆盖层
<body>
  <svg defs>                   ← 线性图标 sprite(#i-search / #i-edit / …)
  #appTopbar                   ← 固定顶栏(品牌 + 面包屑 + 搜索 + 编辑/保存/导出)
  #appSidebar                  ← 固定左侧栏(模块 → 阶段 两级导航)
  .module-tabs / .module-panel ← 旧模块切换(被 APP 外壳隐藏,但仍是数据载体)
    .flow > .stage > .steps > .node
      └ .node-inline-substeps  ← 点卡片后原位内联展开的子步骤区(JS 动态插入)
  #detail / #miniModal / #imgview / #fmtbar / .saved-toast  ← 各类浮层
  <script> … </script>         ← 全部 JS(约 1700 行):折叠 / 拖拽 / 搜索 / 内联展开 / 保存 / 侧栏
```

没有 CSS/JS 子目录——**整页完全内联**,这是刻意约束(见 Development Philosophy)。

---

## 3. Main Technologies

- **HTML5** — 语义化,单文件承载整套流程数据。
- **Vanilla CSS** — 单个 `<style>` 块。设计令牌为 `:root` 上的 CSS 自定义属性。无预处理器、
  无工具类框架。注意:文件内有**两套 `:root`**——早期"旧外壳"令牌(`--accent`、`--radius:4px`)
  与后加的"APP 外壳"令牌(`--r:10px`、`--side-w`…),后者通过覆盖层接管真实外观(见 DESIGN.md)。
- **Vanilla JavaScript (ES6+)** — 单个 `<script>` 块。无框架、无打包器、无 npm。仅用 DOM
  原生能力:`contenteditable`、`requestAnimationFrame`、`Blob` 下载、原生拖拽、滚动驱动的
  "scroll spy"(滚动监听点亮当前阶段)。
- **内联 SVG 图标 sprite** — 隐藏的 `<svg><defs>` + `<symbol>`,通过 `<use href="#i-…">`
  引用(线性、24×24、描边风格)。
- 运行时**无任何外部网络请求**,字体使用系统字体。

---

## 4. Development Philosophy

1. **单文件、零依赖。** 双击 `.html` 即可离线运行,无需安装、无需服务器。永远不要引入构建
   步骤、包管理器或外部 CDN/字体/脚本。
2. **文件即数据库。** 编辑与保存通过在浏览器内重新序列化 DOM 完成。把运行时生成的 DOM
   (如 `.flow-rail` 路线导航、scroll-spy 类名)**排除在保存输出之外**,改为在加载时重建。
3. **默认只读,按需编辑。** 页面以干净的只读模式打开。编辑用的删除按钮、拖拽手柄、"加卡片"
   占位等,在进入编辑模式(`body.edit-all`)或聚焦单张卡片前一律隐藏。
4. **渐进增强与优雅打印。** 页面提供完整的 `@media print` 路径,导出 PDF 时去除外壳、铺平布局。
5. **可访问性是内建而非补丁。** 尊重 `prefers-reduced-motion`;保留焦点轮廓;移动端保持足够
   大的点按目标。
6. **令牌优先于魔法数字。** 颜色、圆角、间距、阴影、控件高度、吸顶偏移都是 CSS 变量。复用它们,
   不要硬编码新的 hex 值或一次性像素圆角。

---

## 5. Design Philosophy

`sop-index.html` 的视觉身份是**干净、专业、面向仪表盘的企业级软件**——克制的配色、充足的留白、
柔和的层次,以商务蓝为主、单一活力橙为点缀。

- **蓝色承载品牌**(`--accent #0070c2`);**橙色是唯一的强调色**(`--gold #fe9a00`),只
  保留给需要"被注意"的地方(eyebrow 小标、"+ 新建"、搜索命中高亮)。
- **浅灰纸面背景上托白色/浅灰卡片**,配 1px 细边框与低扩散柔和阴影,而非厚重投影。
- **运动细腻而有物理感**——短时长、弹性缓动、轻微的缩放/位移反馈,绝不为装饰而动。

完整规范(精确 hex、两套令牌的关系、字阶、间距、圆角、阴影层级)见 `DESIGN.md`。

---

## 6. Naming Conventions

- **CSS 类名:** 小写、连字符分隔、语义化 —— `stage-head`、`node-desc`、`step-count`、
  `flow-rail`、`mod-stages`、`ss-title`。复合组件用短前缀:`node-`(步骤卡片)、`stage-`
  (阶段)、`mod-`(侧栏模块)、`ss-`(substep 子步骤)、`tb-`(顶栏)、`li-`(链接项)、
  `sr-`(搜索结果)。
- **状态类**是由 JS 切换的形容词:`.open`、`.active`、`.detail-active`、`.editing`、
  `.dragging`、`.drag-over`、`.is-current`、`.is-passed`、`.is-upcoming`、`.head-stuck`、
  `.search-dim`、`.hit`、`.show`、`.on`、`.anim-in`、`.anim-out`。页面级模式挂在
  `<body>`(`body.edit-all`)。
- **ID** 仅留给单例:`#appTopbar`、`#appSidebar`、`#detail`、`#miniModal`、
  `#imgview`、`#searchbox`、`#fmtbar`、`.saved-toast(#toast)`。
- **SVG 图标 symbol** 前缀 `i-`:`#i-search`、`#i-edit`、`#i-save`、`#i-export`、
  `#i-trash`、`#i-plus`、`#i-back`、`#i-link`、`#i-image`、`#i-copy`、`#i-x` …
- **CSS 自定义属性:** 语义角色名(`--accent`、`--ink`、`--line`、`--muted`、`--card-bg`)、
  布局度量(`--topbar-h`、`--side-w`、`--gutter`、`--nav-item-h`、`--top-offset`)、
  比例尺(`--r` 圆角、`--sp` 间距)。
- **JS:** `camelCase`,以动作命名(`toggleStage`、`openDetail`、`runSearch`、
  `switchModule`、`buildSidebar`、`expandNodeInline`、`saveFile`)。模块/阶段/卡片 ID 是短随机串
  (如 `mmqqfumch0`)。
- **注释一律用中文**,解释*为什么*(意图、边界情况),与现有密集注释保持一致。延续这种风格。

---

## 7. How to Extend the Project

**动手前:** 先读 `DESIGN.md`、`COMPONENTS.md`、`ANIMATION.md`。

在 `sop-index.html` 内扩展时:

- **复用 `:root` 令牌块**,不要重新定义颜色/圆角/间距。新值一律走变量。
- **复用 APP 外壳**(`#appTopbar`、`#appSidebar`、`.module-panel.active` 内容区)。
- **复用现有组件**:卡片 = `.node`、标签 = `.tag`、徽章 = `.step-count`、弹窗 =
  `#detail`/`#miniModal`、提示 = `.saved-toast`、搜索 = `#searchbox`。仅当确无现成组件
  可用时才新增,并同步登记到 `COMPONENTS.md`。
- **新增图标**:在 `<svg><defs>` sprite 里加一个新的 `i-…` symbol,用同样的 24×24 描边风格
  绘制(`fill:none; stroke:currentColor; stroke-width:2; round caps/joins`)。
- **保持单文件、零依赖。**
- **运行时生成的 DOM 不写入保存输出**,改为加载时重建(参考 `saveFile()` 与 `buildRail()`)。
- **保留 `@media print` 路径** 与 **`@media (prefers-reduced-motion: reduce)` 路径**。
- 完成前必须验证 **编辑 → 保存 → 重新打开** 往返无损。

确需引入新令牌/值/组件/动效时,**在同一次提交里更新对应的 `.md` 文件**,让文档始终是唯一来源。

---

## 8. Instructions for AI

本仓库主要由 AI 编码助手维护。**每个任务都遵循**以下工作约定。

### 必读顺序(每次动手前都要做)

修改**任何**代码前,依次阅读:

1. **`README.md`**(本文件)—— 目的、约束、约定。
2. **`DESIGN.md`** —— 视觉契约(颜色、字体、间距、圆角、阴影、布局、图标)。
3. **`COMPONENTS.md`** —— 组件目录及各组件状态/变体。
4. **`ANIMATION.md`** —— 动效系统。

在吃透这四份文档之前不要开始编辑。它们是**唯一真实来源**;代码必须服从文档,而非相反。

### Rules of engagement

- **推导,绝不臆造。** 只使用 `DESIGN.md` / `ANIMATION.md` 中已定义的颜色、圆角、间距、阴影
  与运动值。不要为了解决一次性问题就引入新 hex、新圆角或新缓动曲线。
- **贴合周边代码。** 沿用现有命名、中文注释的风格与密度、缩进与惯用法。
- **尊重约束:** 单文件、零依赖、无构建步骤、无外部网络请求、默认只读、reduced-motion 支持、
  可用的打印路径,以及干净的 编辑→保存→重新打开 往返。
- **保持文档同步。** 若改动确实新增或修改了某个令牌、组件或动效,在同一次改动里更新对应 `.md`,
  让后续任务继承新的真实状态。
- **声明完成前先验证。** 打开受影响页面,确认渲染正确,并确认保存/重开仍正常。如实汇报你测了什么。
