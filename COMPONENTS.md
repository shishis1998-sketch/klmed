# COMPONENTS.md — 组件目录 (Component Catalog)

> 来源:`SOP流程.html`。每个组件均为该文件中真实存在的可复用单元。任务清单中列出但本项目
> **不存在**的组件,在文末"Not present"小节如实标注,避免凭空发明。
>
> 每个组件按统一结构记录:**Purpose / Visual / States / Variants / Spacing / Interaction /
> Accessibility**。通用规则:圆角 `var(--r)=10px`(旧组件残留 4px)、主色 `--accent #0070c2`、
> 过渡 `.12s–.15s`(动效细节见 `ANIMATION.md`)。

---

## 1. Buttons 按钮

**Purpose** 触发操作。分主操作、次操作、图标按钮、虚线"添加"四类。

**Visual**
- **次按钮(默认)** `.toolbar button` / `.tb-actions button`:白底、`1px var(--line)` 边、
  `--ink` 文字、`10px` 圆角(顶栏)、`13px/600`。
- **主按钮 primary**:`--accent` 蓝底白字,hover 加深到 `#003f7a`。
- **图标圆按钮 `.icon-btn`**:`26×26`、正圆、半透明墨底 `rgba(53,71,84,.07)`,hover 翻蓝底白字;
  危险变体 `.del` hover `--rose`,复制 `.copy` hover `--gold`。
- **虚线添加 `.add-step` / `.side-add` / `.add-stage`**:透明底 + `1.5px dashed var(--line)` +
  蓝字 `+`,hover `--accent-soft` 底、蓝边。
- **返回按钮 `.back-btn`**:白底灰字 + 图标,hover `--paper`。

**States** default / hover(变蓝或加深)/ active(`transform:scale(.94)` 回弹)/ `.on`
(开关态,蓝底白字,如编辑按钮)/ disabled(无专用样式,靠隐藏)。

**Variants** primary(蓝)| ghost/default(白)| icon(正圆)| icon.del / icon.copy(语义色)|
dashed-add(虚线)。

**Spacing** 顶栏按钮 `height:34px; padding:0 13px; gap:6px`(图标+文字);工具栏 `7px 14px`;
按钮组 `gap:8px`。

**Interaction** hover 颜色过渡 `.15s`;按下 `scale(.94)` 120ms 弹回(`transform .12s ease`)。

**Accessibility** 图标按钮带 `title`;`cursor:pointer`;reduced-motion 下取消按下缩放。

---

## 2. Cards 卡片 / Process Nodes 流程节点 / Workflow Cards

> 项目核心。一张 `.node` 同时是"卡片""流程节点""工作流卡片"。

**Purpose** 表示流程中的一个**步骤**;点击在**原位内联展开**该步骤的元信息与子步骤(不再跳转下钻页)。

**Visual** 灰底 `--card-bg #f3f4f6` + `1px var(--card-bd)` 边 + `10px` 圆角(APP 外壳标准);
旧外壳下为白底 + `--shadow` + **彩色顶边 `border-top:4px`** 标识类型。内含:顶部操作条
`.node-bar`(拖拽手柄 + 删除)、类型标签 `.tag`、标题 `.node-title`(15px/700)、描述
`.node-desc`(13.5px,`#5a5e6e`)、"详情 →" 链接 `.node-open`。

**States** default / **hover**(旧:`translateY(-3px)` 上浮 + 蓝调阴影 + 蓝边;APP 外壳:不上浮,
仅 `border-color:#c3d6ec` + 底色 `#eef2f7`)/ `.editing`(`0 0 0 2px var(--accent)` 蓝环)/
`.dragging`(`opacity:.4`)/ `.drag-over`(`0 0 0 3px var(--gold)` 橙环)/ `.search-dim`
(`opacity:.28`,搜索未命中)/ `.anim-in`/`.anim-out`(增删弹入弹出)。

**Variants(类型,彩色顶边 + 标签配色)**

| 变体 | 顶边色 | 标签底/字 | 语义 |
|------|--------|-----------|------|
| `.node.system` | `--blue #0070c2` | `#e8f4ff` / `--blue` | 系统操作 |
| `.node.decision` | `--rose #354754` | `#dfe4e8` / `--rose` | 决策判断 |
| `.node.check` | `--green #003f7a` | `#e8f4ff` / `#003f7a` | 核对/检查 |
| (默认) | `--gold #fe9a00` | `--lilac` / `--gold` | 操作步骤 |

**Spacing** padding `15px 16px`(APP 外壳)/ `14px 16px`(旧);网格 `gap:var(--gutter)`;
`node-bar` 下边距 6px。

**Interaction** 整卡可点 → `expandNodeInline(node)` **原位展开**:被点卡片置顶并锁定原始格宽
(`--card-w`)、其余同阶段卡片 `display:none` 隐藏,卡片下方滑出「返回 + 负责人/预计/频率 +
子步骤卡片」,子步骤逐个淡入(详见 ANIMATION.md);点「← 返回」/ ESC 收起。编辑模式
(`body.edit-all`)下展开视图额外提供「＋ 添加子步骤」与逐张改字/删除。hover 反馈;编辑模式
下显示右上角编辑/删除(`.node-bar` `display:flex`);可拖拽排序(grip 触发 `draggable`)。

> 注:旧的全屏下钻页 `#stepView` / `openStep` / `renderStepView` 已移除,改为上述内联展开。

**Accessibility** 可编辑字段用 `contenteditable`,聚焦显示 `outline:2px dashed var(--accent)`;
拖拽手柄/删除带 `title`;reduced-motion 关闭 hover 上浮与弹入弹出。

---

## 3. Stage 阶段卡片(可折叠分组)

**Purpose** 把若干步骤卡片归为一个"阶段";思维导图式可折叠。

**Visual** `1.5px var(--line)` 边 + `10px` 圆角白底;标题栏 `.stage-head`(中性小圆点
`.stage-num` + 标题 `.stitle` + 负责人 `.meta` + 步骤数徽章 `.step-count` + 控制按钮组)。
APP 外壳下阶段去边框、标题放大为 20px/800 并吸顶。

**States** default / `.open`(展开,蓝边 + 聚焦阴影 + `sop-stage-pop` 弹性放大动画)/
`.detail-active`(APP 外壳中"当前显示的阶段")/ `.head-stuck`(标题吸顶后加分层投影)/
scroll-spy:`.is-current`(圆点蓝色放大 + 光晕)、`.is-passed`(灰)、`.is-upcoming`(浅灰)/
`.editing` / `.dragging` / `.drag-over` / `.search-dim`。未展开且同列有其他展开阶段时
`opacity:.55; scale(.97); blur(1px)` 退焦。

**Variants** 收起(四角圆,标题即全部)| 展开 | APP 外壳激活态(吸顶标题栏)。

**Spacing** 标题 `min-height:46px (--nav-item-h)`,padding `10px 12px`;`.steps` 内边距 `14px`。

**Interaction** 点标题 `toggleStage()` 折叠/展开(`steps-wrap` 高度 `.42s` 弹性过渡);
`.stage-toggle` 箭头旋转 90°;可拖拽排序;hover 显示控制按钮。

**Accessibility** 标题可编辑(聚焦取消两行截断);圆点为装饰(不暗示先后顺序);reduced-motion
关闭弹性动画与圆点缩放。

---

## 4. Tags 标签

**Purpose** 标注步骤类型/属性的小色块。

**Visual** `inline-block`、`11px/700`、`letter-spacing:.04em`、`padding:2px 9px`、`3px` 圆角。
默认 `--lilac` 底 / `--gold` 字。

**Variants** `.t-system`(`#e8f4ff`/`--blue`)、`.t-check`(`#e8f4ff`/`#003f7a`)、
`.t-decision`(`#dfe4e8`/`--rose`)。详情区另有 `.tag-preview` 预览态。

**States** default;可编辑(`contenteditable`);APP 外壳读模式下卡片内 `.tag` 隐藏。

**Spacing** `padding:2px 9px`;`align-self:flex-start`(只占文字宽)。

**Interaction / Accessibility** 编辑模式下可改文案与类型(配 select);空标签保留最小点击区。

---

## 5. Badge 徽章(步骤数 / 计数)

**Purpose** 显示阶段下步骤数量或模块计数。

**Visual** `.step-count`:`11px/700`、蓝字 `--accent`、白底 + `1px var(--accent)` 边、
`20px`(旧)/ `999px`(APP 外壳)全圆胶囊、`padding:2px 9px`。侧栏 `.badge`:蓝字 +
`--accent-soft` 底 + `999px`。

**States / Variants** 边框胶囊(步骤数)| 实底胶囊(侧栏模块计数)。编辑模式下侧栏 badge 隐藏让位
给编辑按钮。

**Accessibility** 纯展示,无交互。

---

## 6. Navigation — Module Tabs 模块标签页

**Purpose** 切换业务模块(销售/财务/VENTAS…)。注:APP 外壳上线后顶部 tab 被隐藏,导航主入口
迁移到侧栏,但 tab 仍是数据/激活态载体。

**Visual** `.module-tab`:`15px/700`、未激活 `--muted` 字、hover 蓝字 + `--accent-soft` 底、
`.active` 蓝底白字;底部 `2px var(--accent)` 整条下边线。`.add-module` 橙字。

**States** default / hover / `.active` / `.add-module`(新建)。

**Interaction** 点击 `switchModule(mod)`;删除按钮 `.module-del`(hover `--rose`)。

---

## 7. Sidebar 左侧栏(两级导航)

**Purpose** 应用主导航:模块(一级)→ 阶段(二级)折叠树 + 新建入口 + 脚注。

**Visual** 固定 `256px` 白底栏。分组标题 `side-label`(11px/700 大写 `#aab2bd`);模块头
`.mod-head`(图标 `--accent` + 名称 + 计数 badge);阶段项 `.stage-item`(缩进 34px,`--muted`,
激活时 `--accent-soft` 底 + 蓝字 + 左侧 3px 蓝条 `::before`);"+ 新建模块" 虚线按钮;脚注
`side-foot`(负责人/更新时间)。

**States** 模块 `.open`(展开二级)、阶段 `.active`(当前)/ `.hit`(搜索命中,左侧 3px 橙条
`inset 3px 0 0 var(--gold)`);`body.edit-all` 下显示模块重命名/删除、新建阶段。

**Spacing** 项内边距 `8–10px`;阶段缩进 `padding-left:34px`;`--r` 圆角。

**Interaction** 点模块头折叠;点阶段项切换 `.detail-active` 阶段;hover 高亮。

**Accessibility** 移动端转抽屉(`translateX(-100%)` ↔ `.open`),带全屏遮罩。

---

## 8. Flow Rail 阶段路线导航 / Timeline / Progress

**Purpose** 旧版左侧竖栏 + scroll-spy,作为流程**时间线/进度指示**(随阅读点亮当前阶段)。
运行时由 JS `buildRail()` 生成,**不写入保存输出**。

**Visual** `232px` 白底栏,逐项 `.rail-dot`(圆点 `.rail-node` + 标签 `.rail-label`);吸顶
`top:calc(--tabs-h + 12px)`。

**States** `.current`(蓝底浅蓝、圆点放大 + 光晕)/ `.passed` / `.upcoming` / `.hit`(命中,
左侧橙标记)。圆点为纯色,不带序号(顺序不固定)。

**Interaction** 点项跳转并展开对应阶段;滚动驱动 `updateScrollSpy()` 自动点亮当前。

**Accessibility** reduced-motion 下关闭圆点缩放。

> 这是项目中最接近 **Timeline / Progress** 的组件;没有独立的进度条或时间轴组件。

---

## 9. Search 全局搜索 + 结果下拉 + 空态 + 错误态

**Purpose** 跨所有阶段搜索步骤,列出命中并跳转高亮。

**Visual** `#searchbox` 输入框(顶栏内,左置搜索图标,`34px` 高,`--r` 圆角,聚焦变蓝底白);
清除按钮;结果下拉 `.search-results`(白底浮层,`max-height:60vh`,阴影 `0 14px 34px -10px`)。
结果项 `.sr-item`(阶段名 `.sr-stage` + 标题 `.sr-title` + 摘要 `.sr-snip`),命中词 `mark`/
`mark.hit` 橙底高亮。

**States**
- **Empty 空态** `.sr-empty` / `.sr-empty`:"无结果"文案。
- **Error 错误态**:无结果时输入框 `.is-error` 红框 `--t-error` + `.is-shaking` 左右抖动
  (`t-input-shake`,见 ANIMATION.md)+ 右侧计数染红。
- 结果项 hover/focus:`--accent-soft` 底。
- 卡片层:命中卡正常,未命中 `.search-dim`(`opacity:.28`)。

**Interaction** `oninput=runSearch()`;点结果项跳转到该阶段并高亮;`clearSearch()` 复位。

**Accessibility** 结果项可键盘聚焦(`.sr-item:focus`);reduced-motion 关闭抖动。

---

## 10. Dropdown 下拉浮层

**Purpose** 浮层列表容器。两处:搜索结果 `.search-results`、格式工具条调色板 `#fmtbar .pop`。

**Visual** 白底 + `1px var(--line)` + `--r` 圆角 + 浮层阴影;`position:absolute`,`[hidden]` 隐藏。

**Interaction** 点外部关闭;颜色格 `grid` 排布。

---

## 11. Modal 弹窗(三种)

**Purpose** 模态层:步骤详情、迷你确认/输入、图片大图。

**Visual & Variants**
- **详情弹窗 `#detail`**:半透明遮罩 `rgba(28,37,38,.45)` + `backdrop-filter:blur(2px)`;
  面板 `.detail-panel`(`max-width:760px`,纸面底,`4px` 圆角,`0 20px 60px` 阴影);顶部
  `.detail-top` 蓝底白字(返回按钮 + 类型小标 + H2);主体 `.detail-body`(分区标题 `h3` 橙色大写
  带下划线 + 描述/属性网格/清单/链接/图片)。
- **迷你弹窗 `#miniModal`**:居中 `.mini-box`(`max-width:380px`,`6px` 圆角),标题 + 列表选择
  `.mini-list` / 输入 `.mini-input` + 操作区(取消 / primary 确认)。用于"新建模块""复制到哪个模块"。
- **图片大图 `#imgview`**:`rgba(0,0,0,.8)` 全屏,`place-items:center`,`cursor:zoom-out`。

**States** `.show`(可见);详情用 `visibility+opacity` 过渡;面板从点击卡片处缩放展开
(`scale(.92)→1`,`transform-origin` 由 JS 设)。

**Interaction** 打开 `openDetail()`;点遮罩或返回关闭;reduced-motion 简化为纯淡入。

**Accessibility** 遮罩 `pointer-events` 随状态切换;焦点可达;返回按钮显著。

---

## 12. Toast 轻提示

**Purpose** 保存/操作成功的短反馈。

**Visual** `.saved-toast`(`#toast`):底部居中、`--accent` 蓝底白字、`10px 22px`、`4px` 圆角、
`--shadow`、`opacity` 过渡。

**States** 默认隐藏(`opacity:0; pointer-events:none`)→ `.show` 淡入,JS 1600ms 后移除。

**Accessibility** 非阻塞;短暂可见。

---

## 13. Forms / Inputs 输入框

**Purpose** 文本输入与就地编辑。

**Visual**
- **搜索输入 `.t-input`**:透明底(背景在外层 wrap),`34px` 高,左 32px 留图标,`--r` 圆角,
  聚焦外层 `:focus-within` 变蓝边白底。
- **标签编辑 `.tag-edit input`** / **迷你输入 `.mini-input`**:白底 + `1px var(--line)` +
  `4px` 圆角 + `9–10px 12px`,聚焦 `border-color:var(--accent)`。
- **就地编辑 `[contenteditable="true"]`**:聚焦 `outline:2px dashed var(--accent); offset:3px`;
  只读态 `contenteditable="false"` 取消编辑光标;空字段保留最小点击区。

**States** default / focus(蓝边或蓝虚线)/ error(搜索 `.is-error` 红框)/ readonly。

**Accessibility** `placeholder` 提示;焦点态清晰;hover 不抢焦点。

---

## 14. Select 下拉选择

**Purpose** 选择标签类型 / 格式选项。

**Visual** `.tag-edit select`:白底 + `1px var(--line)` + `4px` 圆角 + `9px 12px`。`#fmtbar select`:
深色 `#333` 底白字(深色工具条内)。

**States / Interaction** 原生 `<select>`;改变即更新标签预览/格式。

---

## 15. Floating Format Toolbar 浮动格式工具条

**Purpose** 选中可编辑文本时浮出的富文本工具(加粗/颜色/等)。

**Visual** `#fmtbar`:深色 `#222` 底、`4px` 圆角、`0 8px 24px rgba(0,0,0,.3)` 阴影、白色按钮
(`30×30`,hover `rgba(255,255,255,.2)`)、分隔条、`input[type=color]`、深色 `select`。
`position:fixed`,跟随选区定位。

**States** 默认 `display:none`;有选区时显示。

**Interaction** 选中文本触发定位;点按钮执行 `execCommand` 类格式化。

---

## 16. Detail Sections — Stat/KPI Cards、Checklist、Link Item、Image Grid

详情弹窗 `.detail-body` 与卡片内联展开区 `.node-inline-substeps` 内的可复用块:

| 子组件 | Purpose | Visual |
|--------|---------|--------|
| **属性卡 `.attr`(Stat/KPI 卡)** | 详情弹窗显示负责人/耗时/频率等键值 | 白底 + `1px var(--line)` + `4px`;`label`(11px `#8a8270`)+ `val`(14px/700)。三栏 `attr-grid` |
| **描述框 `.d-desc`** | 详情弹窗长描述正文 | 白底框,`line-height:1.7`,`min-height:80px` |
| **清单/添加 `.add-link` / `.add-img` / `.ni-addsub`** | 虚线"添加项"按钮 | `1.5px dashed` + 蓝字,hover `--accent-soft`;`.ni-addsub`=内联展开里的「＋ 添加子步骤」,与子步骤卡片同宽 |
| **链接项 `.link-item`** | 外链/附件条目 | 白底 + `1px` 边 + `4px`;图标(网址蓝/附件橙)+ 文字 + 删除;文件变体显示大小 |
| **图片缩略 `.img-thumb`** | 详情弹窗截图附件 | `120×90` 覆盖裁切缩略,角标删除按钮;点开 `#imgview` 大图 |
| **内联子步骤卡 `.ni-card`** | 卡片内联展开里的子步骤小卡 | **淡蓝底 `#eef5fc` + `1.5px #d3e3f4` 边** + `--r`;序号 `.ni-num`(24×24 蓝字方块,底 `#d8e7f7`)+ 标题 `.ni-name` 与描述 `.ni-desc` **同 14px**(仅粗细/颜色区分);无连接箭头,纯堆叠 |

**States** 内联子步骤卡 `.ni-card`:读态只读;编辑态(`.ni-card.editing`,随 `body.edit-all`)
标题/描述变白底输入框、右上角浮出删除按钮 `.ni-del`。详情弹窗内的块同样含读/编辑两态。

---

## 17. Breadcrumb 面包屑

**Purpose** 顶栏显示当前模块/阶段路径。

**Visual** `.tb-crumb`(`14px --muted`,单行省略),当前段 `b` 用 `--accent` 加粗。

**Interaction** 随导航更新(JS)。

---

## 18. Drag Handles & Insert Affordances 拖拽手柄 / 插入条

**Purpose** 编辑模式下重排与插入。

**Visual** 手柄 `.grip`/`.node-grip`(`⠿`,`opacity:.35–.45`,`cursor:grab`);阶段间插入条
`.arrow`(竖线 + hover 浮出 `.insert-here` "+ 在此插入阶段")。

**States** `cursor:grab` → `:active grabbing`;拖拽中卡片 `.dragging`,目标 `.drag-over`(彩色环)。

---

## 19. Legend / Hint(辅助说明,近似 Alert)

**Purpose** 图例与使用提示。本项目**没有标准 Alert 组件**,以下为最接近的说明类元素。

**Visual** `.legend`:白底 + `1px var(--line)` + `4px`,内含色点 `.dot`(`12×12`,`3px`)+ 文字
说明各类型色。`.hint`:`13px #7a7363` 提示文案。

---

## Not present(任务清单列出但本项目不存在,勿凭空发明)

| 组件 | 状态 / 最接近的替代 |
|------|---------------------|
| **Tables 数据表格** | 无 `<table>` 数据表;键值信息用 `.attr-grid`(属性卡网格)表达 |
| **Alert** | 无专用 alert;用 `.legend` / `.hint` / 搜索错误红框表达提示 |
| **Tabs** | 仅有 `.module-tabs` 模块切换(APP 外壳下已隐藏);无内容区 tab 组件 |
| **Progress** | 无进度条;`.flow-rail` scroll-spy 充当流程进度指示 |
| **Charts** | 无图表 |
| **Loading / Spinner** | 无加载动画;数据为本地即时渲染,无异步 loading |
| **Filter** | 无独立筛选器;只有全局搜索 |
| **Tooltip** | 无自定义 tooltip;统一使用原生 `title` 属性 |
| **Pagination** | 无分页(单页全量渲染) |
| **Map Markers** | 无地图/标记 |
| **Empty State** | 仅搜索空态 `.sr-empty`,无通用空状态插画组件 |

> 若未来确需新增以上组件,请遵循 `DESIGN.md` 的令牌(`--r`、`--accent`、`--line`、`--shadow`、
> `--gutter`)与本目录的命名/状态/交互范式,并把新组件登记回本文件。
