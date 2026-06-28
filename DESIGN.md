# DESIGN.md — 视觉设计规范 (Visual Design Specification)

> 来源:`SOP流程.html`。以下每条规则均从该文件的真实实现中提取,未发明任何不存在的样式。

`SOP流程.html` 内有**两套 `:root` 令牌**:

- **旧外壳令牌**(`--ink`/`--accent`/`--radius:4px`…)— 早期版本遗留,大量旧组件仍引用其
  *颜色*。
- **APP 外壳令牌**(`--r:10px`/`--side-w`/`--card-bg`…)— 后加的"APP 外壳改版覆盖层",通过
  大量 `!important` 覆盖,**接管页面的最终外观**(圆角、卡片底色、布局尺寸)。

**判定标准:** 颜色以旧令牌为权威(两套颜色一致或互补);**圆角、卡片底色、布局尺寸以 APP 外壳
令牌为权威**(它是覆盖层,是用户实际看到的)。文中对每个有冲突的属性都标注了哪个是 **Dominant
(标准)**。

---

## 1. Brand Personality

| 关键词 | 体现 |
|--------|------|
| **Professional / Enterprise** | 商务蓝主色、克制配色、面包屑 + 侧栏的标准后台框架 |
| **Dashboard-oriented** | 固定顶栏 + 固定左侧栏 + 可滚动内容区的应用外壳 |
| **Clean / Minimal** | 白色卡片、1px 细边、大量留白、单一强调色 |
| **Soft** | 低扩散柔和阴影、10px 圆角、弹性缓动的细腻动效 |
| **Modern** | 线性 SVG 图标、scroll-spy、下钻详情页、reduced-motion 适配 |

一句话:**干净、柔和的企业级仪表盘软件**,蓝为骨、橙为点。

---

## 2. Colors

### 2.1 核心调色板(旧外壳 `:root`,颜色权威来源)

| 角色 | 令牌 | Hex | 用途 |
|------|------|-----|------|
| **Primary 主色(商务蓝)** | `--accent` / `--blue` | `#0070c2` | 主操作、激活态、链接、边框高亮、阶段圆点、面包屑强调 |
| **Primary soft 主色浅底** | `--accent-soft` / `--green-soft` | `#e8f4ff` | 激活项背景、hover 浅底、徽章底、命中行底 |
| **Primary deep 主色加深** | (字面量) | `#003f7a` | primary 按钮 hover、`--green` 同值,深海蓝 |
| **Secondary / Accent(活力橙)** | `--gold` | `#fe9a00` | eyebrow 小标、"+ 新建模块"、搜索命中高亮 `mark`、复制按钮 hover |
| **Decision 深蓝灰** | `--rose` / `--ink` | `#354754` | 正文主色;同时作"决策"类型色与删除危险态(hover) |
| **Check 深海蓝** | `--green` | `#003f7a` | "核对/检查"类型色 |
| **Danger / Error 错误红** | `--t-error` | `#e5484d` | 搜索无结果输入框红框、错误抖动态 |

> 说明:本项目把"语义色"复用为"类型色"。蓝/橙/深蓝灰三色同时担当流程节点的三种类型
> (系统 system = 蓝、决策 decision = 深蓝灰、核对 check = 深海蓝),橙色用于"操作步骤"强调。
> **没有独立的绿色 success;`--green` 实为深海蓝。** 唯一的真正"危险/警示"色是 `#e5484d`(错误)
> 与 hover 时的 `--rose`。

### 2.2 中性色 / 表面 / 文字

| 角色 | 令牌 | Hex | 用途 |
|------|------|-----|------|
| **Background 页面纸面** | `--paper` / `--lilac` | `#f0f0f0` | `<body>` 背景、弹窗面板底、固定顶栏底 |
| **Surface 内容区底(白)** | `--content-bg` | `#ffffff` | 右侧内容区、顶栏、侧栏、详情面板 |
| **Card 卡片底(灰)** | `--card-bg` | `#f3f4f6` | APP 外壳下的步骤卡片 `.node` 底色(**Dominant**) |
| **Card border 卡片描边** | `--card-bd` | `#e4e6ea` | APP 外壳下卡片边框 |
| **Border 通用描边线** | `--line` | `#d5d9dc` | 分割线、输入框/按钮边框、虚线占位 |
| **Text 主文字(墨色)** | `--ink` | `#354754` | 标题、正文主色 |
| **Text muted 次要文字** | `--muted` | `#7a8898` | 元信息、面包屑、占位、未激活侧栏项 |
| **Text desc 描述灰** | (字面量) | `#5a5e6e` | 卡片描述 `.node-desc`、子步骤描述 |
| **Text faint 更浅** | (字面量) | `#aab2bd` / `#aab2bd` | 侧栏分组标题、计数、脚注 |
| **Hint 提示棕灰** | (字面量) | `#7a7363` | `.hint` 使用提示语 |

### 2.3 颜色使用规则

- 主操作、激活、当前态一律用 **`--accent #0070c2`**;其浅底 **`#e8f4ff`** 配套。
- "被强调/可新增/被命中"用 **橙 `--gold #fe9a00`**,且**只在这些语义**上用,避免泛滥。
- primary 按钮 hover 加深到 **`#003f7a`**。
- 卡片永远是 **白底或 `#f3f4f6` 灰底 + `--line`/`--card-bd` 细边**,不要用强投影替代边框。
- 错误唯一色 **`#e5484d`**,仅用于搜索无结果反馈。

---

## 3. Typography

### 3.1 字体族(Font families)

```css
font-family: "Segoe UI","PingFang SC","Microsoft YaHei", sans-serif;   /* 全局 */
/* 等宽(仅个别代码/格式串场景沿用): "Consolas","Courier New", monospace */
```

系统字体优先,跨平台覆盖中英文。无 Web Font。

### 3.2 字号 / 字重 / 层级

| 角色 | font-size | font-weight | 备注 |
|------|-----------|-------------|------|
| **H1 文档标题** | `30px`(移动端 `21px`) | `800` | `letter-spacing:-.01em`,左侧 5px 蓝色竖条 |
| **顶栏品牌 tb-brand** | `19px` | `800` | |
| **详情弹窗 H2** | `22px` | `800` | 蓝底白字 |
| **下钻页步骤标题 step-title** | `22px` | `800` | |
| **阶段标题 stitle(展开)** | `20px` | `800` | APP 外壳下放大;收起时 `14px`/`700` |
| **子步骤标题 ss-title** | `16px` | `700` | |
| **卡片标题 node-title** | `15px` | `700` | |
| **模块 tab / 顶栏控件** | `15px`/`13px` | `700`/`600` | |
| **Body 正文 / 输入框** | `14px` | `400–500` | 基准字号 |
| **卡片描述 node-desc** | `13.5px` | `400` | 颜色 `#5a5e6e`,`line-height:1.55` |
| **Label 字段标签 attr label** | `11px` | `600` | 颜色 `#8a8270` |
| **侧栏分组标题 side-label** | `11px` | `700` | `letter-spacing:.14em`,大写,`#aab2bd` |
| **Eyebrow 小标** | `12px` | `700` | `letter-spacing:.32em`,大写,橙色 |
| **Tag 标签** | `11px` | `700` | `letter-spacing:.04em` |
| **Badge 步骤数 step-count** | `11px` | `700` | 胶囊徽章 |
| **Small 计数/脚注** | `11–13px` | `400–600` | `--muted` 或更浅 |

**层级原则:** 标题 `800` 粗、正文 `400–500`、强调/标签/徽章 `600–700`。字号梯度约为
`30 / 22 / 20 / 16 / 15 / 14 / 13.5 / 12 / 11`。正文行高 `1.5–1.7`。

---

## 4. Spacing

间距基于约 **4px 的节奏**,常见取值:

```
2  4  6  8  10  12  14  16  18  20  22  24  26  32  34  44
```

关键令牌与惯例:

| 令牌 / 场景 | 值 | 用途 |
|-------------|-----|------|
| `--sp` 基础间距单位 | `16px` | 子步骤间距、详情区块间距 |
| `--gutter` 内容区边距 | `24px` | 内容区左右内边距 **=** 卡片网格列间距(三处留白相等) |
| 卡片内边距 `.node` | `14–16px`(APP 外壳 `15px 16px`) | |
| 阶段标题内边距 | `10px 12px` | |
| 卡片网格 gap | `var(--gutter) 24px` | 两栏步骤卡片间距 |
| 区块外边距 | `header 26px`、`.flow 34px`、`legend 34px` | 大区块之间 |
| 详情弹窗内边距 | `detail-top 20px 26px` / `detail-body 26px` | |

**规则:** 同类元素之间用同一档间距;内容区左右留白、卡片列间距、卡片到边的留白保持相等
(均为 `--gutter`),这是该设计"对齐感"的核心。

---

## 5. Border Radius

> **冲突属性。** 旧令牌 `--radius:4px`,APP 外壳令牌 `--r:10px`。覆盖层用 `--r` 接管了卡片、
> 输入框、按钮、侧栏项、图片等的最终圆角,因此 **10px 是当前标准(Dominant)**;4px 仍残留在
> 少数未被覆盖的旧组件(标签、箭头插入条等)上。

| 元素 | 当前圆角(Dominant) | 来源 |
|------|---------------------|------|
| **卡片 Cards `.node`** | `10px` | `--r`(覆盖 `!important`) |
| **输入框 Inputs**(顶栏搜索) | `10px` | `--r` |
| **按钮 Buttons**(顶栏 tb-actions) | `10px` | `--r` |
| **侧栏项 / 模块头** | `10px` | `--r` |
| **子步骤卡片 / 信息卡** | `10px` | `--r` |
| **图片 / 缩略图** | `10px` / 旧 `4px` | `--r` |
| **Modal 弹窗面板** `.detail-panel` | `4px` | 旧令牌(未被覆盖) |
| **Mini 弹窗** `.mini-box` | `6px` | 字面量 |
| **Badge 步骤数 / 徽章** | `20px` / `999px`(全圆胶囊) | 字面量 |
| **Tag 标签 `.tag`** | `3px` | 字面量 |
| **小圆按钮 icon-btn** | `50%`(正圆) | |
| **阶段圆点 stage-num** | `50%`(正圆) | |

**规则:** 新增 application 组件的圆角统一用 `var(--r)`(10px)。胶囊/徽章用 `999px`,纯圆点
用 `50%`。除非维护旧组件,否则不要再引入 4px。

---

## 6. Shadows

| 层级 | 值 | 用途 |
|------|-----|------|
| **令牌 `--shadow`(卡片基础)** | `0 2px 8px rgba(53,71,84,.10)` | 卡片默认、toast |
| **侧栏面板** | `0 2px 8px rgba(53,71,84,.06)` | flow-rail / 顶栏更淡 |
| **顶栏 appTopbar** | `0 1px 3px rgba(53,71,84,.06)` | 极淡底边投影 |
| **卡片 hover 上浮** | `0 12px 22px -12px rgba(0,112,194,.13)` | 负扩散、向下收的蓝调阴影 |
| **阶段展开 open** | `0 12px 22px -12px rgba(0,112,194,.13)` | 同上,聚焦当前阶段 |
| **吸顶标题 head-stuck** | `0 8px 16px -10px rgba(53,71,84,.45)` / 内容区 `0 6px 12px -10px` | 标题钉住后与滚动内容分层 |
| **搜索结果下拉** | `0 14px 34px -10px rgba(53,71,84,.28)` | 浮层 |
| **详情弹窗面板** | `0 20px 60px rgba(0,0,0,.3)` | 最高层模态 |
| **Mini 弹窗** | `0 20px 60px rgba(0,0,0,.3)` | 模态 |
| **格式工具条 fmtbar** | `0 8px 24px rgba(0,0,0,.3)` | 深色浮动条 |
| **焦点光晕 / 当前阶段圆点** | `0 0 0 3px var(--accent-soft)` / `0 0 0 3px var(--gold)` | 环形高亮(非投影) |

**层级原则(由低到高):** 顶栏/侧栏(≈0.06)< 卡片基础(`--shadow`)< hover/展开(蓝调负扩散)
< 浮层下拉(0.28)< 模态弹窗(0.3)。阴影**普遍用负扩散值收窄横向渗出**,色调偏蓝
(`rgba(0,112,194,…)`)或墨色(`rgba(53,71,84,…)`),避免纯黑厚重投影。环形高亮用
`0 0 0 3px <soft>` 实现,不算阴影层级。

---

## 7. Layout

应用外壳为经典的**固定顶栏 + 固定左侧栏 + 可滚动内容区**三段式。

### 7.1 关键尺寸令牌

| 令牌 | 值 | 含义 |
|------|-----|------|
| `--topbar-h` | `56px` | 顶栏高度(固定贴上缘,横贯整宽) |
| `--side-w` | `256px` | 左侧栏宽度(固定贴左缘,占满高度;移动端 `0`) |
| `--gutter` | `24px` | 内容区左右内边距 = 卡片列间距 |
| `--ctl-h` | `34px` | 顶栏控件(搜索框/按钮)统一高度 |
| `--nav-item-h` | `46px` | 侧栏项 / 阶段标题统一最小高度 |
| 内容主区最大宽度 | `1200px` | `header`/`.flow`/`.toolbar` 等旧区块的 `max-width`;APP 外壳内容区铺满去除上限 |
| 详情/下钻最大宽度 | `760px` / `1240px` | 详情弹窗 760;下钻页布局 1240 |

### 7.2 Grid 网格

- **步骤卡片网格:** `display:grid; grid-template-columns:repeat(2,minmax(0,1fr)); gap:var(--gutter)`
  —— 两栏铺满内容区;`≤1000px` 退化为单栏。用 `minmax(0,1fr)` 防止内容把列撑出横向溢出。
- **属性网格 attr-grid:** 三栏 `repeat(3,1fr)`;`≤640px` 两栏;`≤380px` 单栏。
- **下钻页 step-layout:** `grid-template-columns:7fr 3fr`(左子步骤宽、右元信息窄),中间竖分割线;
  `≤760px` 单栏。

### 7.3 Header / Sidebar / Content

- **Header(顶栏 `#appTopbar`):** `position:fixed; top/left/right:0; height:56px`,白底 + 底边
  `--line` + 极淡投影。内含:品牌(800/19px)、面包屑(`--muted`,蓝色强调)、搜索框(弹性
  `0 1 320px`)、右侧操作组(编辑/保存 primary/导出)。
- **Sidebar(左侧栏 `#appSidebar`):** `position:fixed; top:56px; left:0; bottom:0; width:256px`,
  白底 + 右边 `--line`。结构:分组标题 `流程模块` → 可滚动区(模块头 `mod-head` → 阶段项
  `stage-item`,两级折叠)→ "+ 新建模块" 虚线按钮 → 脚注(负责人/更新时间)。移动端
  (`≤760px`)转抽屉:`translateX(-100%)`,`.open` 滑入并加 `0 0 0 100vmax rgba(0,0,0,.3)` 遮罩。
- **Content(内容区 `.module-panel.active`):** `margin-left:var(--side-w)`,顶部留
  `--topbar-h + 20px`,左右 `--gutter`,底部 64px,`background:var(--content-bg)`,`min-height:100vh`。
  阶段标题在内容区内 `position:sticky; top:var(--topbar-h)` 吸顶。
- **Step drill-down(`#stepView`):** `position:fixed; top:56px; left:256px; right/bottom:0`,覆盖
  内容区,可滚动;`.show` 显示。返回按钮 + 标题 + 7:3 两栏布局。

### 7.4 Content spacing / Card spacing

- 内容区到边、卡片列间距、卡片到内容边的留白**三者相等**,均为 `--gutter (24px)`,这是排版的
  对齐基准。
- 卡片之间纵横间距统一 `--gutter`;子步骤之间 `--sp (16px)`;详情区块之间 `≈24px`。

### 7.5 Responsive behavior(断点)

| 断点 | 行为 |
|------|------|
| `≤1000px` | 步骤卡片网格 → 单栏 |
| `≤760px` | 侧栏 → 抽屉(`--side-w:0`);下钻页 `left:0`;`#stepView` 7:3 → 单栏;`.flow` 单栏 |
| `≤680px` | `.node` 最大宽度放开 |
| `≤640px` | 整体收紧:body 内边距 12px、模块栏对齐、左右导航改横排滚动、字号下调、attr-grid 两栏 |
| `≤380px` | attr-grid 单栏 |

---

## 8. Icons

| 维度 | 规范 |
|------|------|
| **风格 Style** | **线性 / outline**(描边),Lucide/Feather 风格,**无填充图标** |
| **画布 / viewBox** | `24 × 24` |
| **描边 Stroke** | `stroke:currentColor; stroke-width:2; stroke-linecap:round; stroke-linejoin:round; fill:none` |
| **着色** | `currentColor` —— 跟随文字色(顶栏 `--muted`、模块头 `--accent`、危险态 `--rose`) |
| **尺寸 Sizes** | 通过容器 `font-size` 控制 `.ic{ font-size:… } .ic svg{ width:1em;height:1em }`:顶栏操作 `15px`、搜索 `16px`、模块头 `17px`、加载/小按钮 `14px` |
| **实现** | 隐藏 `<svg><defs>` sprite + `<symbol id="i-…">`,通过 `<use href="#i-…"/>` 复用 |
| **现有图标集** | `i-search, i-edit, i-save, i-export, i-bag, i-wallet, i-folder, i-gear, i-plus, i-back, i-link, i-image, i-trash, i-x, i-copy` |

> 旧 UI 中仍有少量 **emoji 图标**(📦💰📁💾🖨⠿✎✕＋▼)残留于模块 tab、拖拽手柄、旧工具栏。
> **标准是线性 SVG sprite**;新增图标请加 `i-…` symbol,不要新增 emoji 图标。

**Usage:** 图标恒与文字配对(按钮 = `<span class="ic"><svg><use/></svg></span> + 文字`),
`gap:6px`;纯图标按钮(icon-btn)为正圆,`26px`,hover 翻蓝底白字。

---

## 9. Visual Language(recurring patterns 总览)

1. **蓝为骨、橙为点、灰为底。** 商务蓝主导一切交互态;橙色只在"被强调/可新增/被命中"出现;
   浅灰纸面托白/灰卡片。
2. **白/灰卡片 + 1px 细边 + 柔和负扩散阴影**,而非厚重投影;卡片有彩色顶边区分类型(系统蓝 /
   决策深蓝灰 / 核对深海蓝 / 默认橙)。
3. **10px 统一圆角(`--r`)**;胶囊徽章 `999px`;圆点 `50%`;标签 `3px`。
4. **三段式应用外壳:** 固定顶栏(56px)+ 固定侧栏(256px)+ 可滚动内容区,辅以"模块→阶段→
   步骤→子步骤"四级信息架构与下钻详情页。
5. **吸顶 + scroll-spy:** 阶段标题吸顶在顶栏下方;滚动时点亮"当前阶段"圆点(蓝色放大 + 浅蓝光晕)。
6. **默认只读、按需编辑:** 编辑控件(删除/拖拽/加卡)在 `body.edit-all` 或聚焦卡片前隐藏;
   `contenteditable` 聚焦时显示蓝色虚线轮廓 `outline:2px dashed var(--accent)`。
7. **环形高亮替代描边变色:** 焦点/当前/命中用 `0 0 0 3px <soft>` 光环表达。
8. **充足且等距的留白:** 内容边距 = 列间距 = 卡片边距 = `--gutter`。
9. **细腻物理动效:** 短时长 + 弹性缓动 + 轻微缩放/位移(详见 `ANIMATION.md`)。
10. **完整的边界适配:** `@media print` 去外壳铺平、`prefers-reduced-motion` 关动效、多断点响应式。
