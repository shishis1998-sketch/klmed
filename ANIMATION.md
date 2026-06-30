# ANIMATION.md — 动效系统规范 (Motion Specification)

> 来源:`sop-index.html`。下列每条 keyframes、transition、easing、duration 均从该文件真实
> 实现中提取。新动效必须复用这里已有的时长与缓动,不要发明新曲线。

---

## 1. Motion Principles 运动原则

1. **细腻、物理、不抢戏。** 动效服务于"理解状态变化",绝不为装饰而动。位移/缩放幅度都很小
   (`translateY(-3px)`、`scale(.94)`、`scale(1.01)`)。
2. **弹性而非机械。** 折叠、展开、详情打开统一用 expo-out 类弹性缓动 `cubic-bezier(.22,1,.36,1)`,
   避免线性收放。
3. **短时长为主。** 微交互 `.12–.15s`,状态切换 `.2–.42s`,无超过 ~0.42s 的常规动效(错误抖动
   总长 280ms 是特例)。
4. **方向有意义。** 卡片悬停上浮(`-3px`)表示"可点";阶段展开从顶部 `transform-origin:top`
   向下舒展;详情从被点卡片处放大铺开(`transform-origin` 由 JS 设)。
5. **聚焦即退焦。** 当前阶段高亮的同时,其它阶段轻微 `scale(.97)+blur(1px)+opacity` 退焦,把
   视觉焦点集中过来。
6. **永远尊重 `prefers-reduced-motion`。** 见 §13。

---

## 2. Duration 时长清单

| 时长 | 用途 |
|------|------|
| `.12s` | 按钮按下回弹(`transform`)、图标按钮/搜索结果项底色、各种微交互 |
| `.15s` | hover 颜色/边框/阴影、卡片 hover、控制按钮显隐 |
| `.2s` | 详情遮罩 visibility/opacity、侧栏抽屉滑动、rail-dot 底色 |
| `.24s` | 折叠箭头旋转 |
| `.26s` | 详情面板 opacity |
| `.3s` | 阶段 opacity/transform/filter 退焦、阶段圆点状态、toast |
| `.32s` | 详情面板 transform(放大展开) |
| `.35s` | rail-node 圆点状态 |
| `.38s` | 节点弹入 `sop-node-in` |
| `.4s` | rail-seg 连接段底色 |
| `.42s` | **阶段折叠/展开高度过渡** + 阶段展开弹性 `sop-stage-pop`(主力时长);内联子步骤逐个淡入 `ni-item-in` |
| `.5s` | 内联子步骤容器 `max-height` 展开(`cubic-bezier(.22,1,.36,1)`) |
| `.16s` | 节点弹出 `sop-node-out`、**内联子步骤收起**(返回比展开快) |
| `150ms` / `280ms` | 搜索输入边框 `ease-out` / 错误态红框回退 |
| `~280ms` | 搜索错误抖动总时长(80+60+80+60) |
| `~320ms` | 吸顶锚定 `anchorHead` 的 RAF 跟随窗口 |

---

## 3. Easing 缓动曲线

| 曲线 | 角色 | 用在哪 |
|------|------|--------|
| `cubic-bezier(.22,1,.36,1)` | **主力 expo-out 弹性**(快起慢收) | 阶段展开 pop、`steps-wrap` 高度、详情面板放大、圆点缩放、rail 圆点、错误抖动各段 |
| `cubic-bezier(.34,1.56,.64,1)` | **回弹/过冲 back-out**(终点轻微越界) | 节点弹入 `sop-node-in`(弹性出现) |
| `cubic-bezier(.4,0,.2,1)` | **标准 material ease** | 折叠箭头旋转 |
| `ease` | 通用 | 按钮 `transform .12s ease`、详情遮罩、多数 transition |
| `ease-in` | 退出 | 节点弹出 `sop-node-out`(加速消失) |
| `ease-out` | 输入边框 | 搜索 `t-input` 边框过渡 |
| `linear` | 多段关键帧驱动 | 抖动动画外层(分段内各自带 `.22,1,.36,1`) |

**规则:** 新增"出现/展开/移动"用 `cubic-bezier(.22,1,.36,1)`;要弹性"蹦一下"用
`cubic-bezier(.34,1.56,.64,1)`;退出用 `ease-in` 并比进入更短。

---

## 4. Keyframes 关键帧动画(全部 5 个)

```css
/* 1. 阶段展开:轻微放大再回弹,营造弹性展开 */
@keyframes sop-stage-pop{ 0%{scale 1} 45%{scale 1.01} 100%{scale 1} }
  /* .42s cubic-bezier(.22,1,.36,1); transform-origin:top center */

/* 2. 步骤卡片弹入(新增/展开时) */
@keyframes sop-node-in{ from{opacity:0; translateY(18px) scale(.93)} to{opacity:1; translateY(0) scale(1)} }
  /* .38s cubic-bezier(.34,1.56,.64,1) both */

/* 3. 步骤卡片弹出(删除/收起时,更快) */
@keyframes sop-node-out{ from{opacity:1; scale(1)} to{opacity:0; translateY(10px) scale(.96)} }
  /* .16s ease-in both */

/* 4. 内联子步骤逐个淡入(卡片原位展开时) */
@keyframes ni-item-in{ to{opacity:1; translateY(0)} }
  /* 起始态由 .node-inline-substeps>* 给(opacity:0; translateY(14px));
     .42s cubic-bezier(.34,1.4,.64,1) forwards,按 --i 递增 .09s 延迟逐个出现 */

/* 5. 搜索无结果:左右抖动错误态 */
@keyframes t-input-shake{ 0→28.57%→57.14%→78.57%→100% 在 ±6px/±4px 间衰减 }
  /* 总时长 calc(80ms*2 + 60ms*2)=280ms linear,各段 timing-function: cubic-bezier(.22,1,.36,1) */
```

---

## 5. Hover Effects 悬停

| 元素 | 效果 |
|------|------|
| **步骤卡片(旧)** | `translateY(-3px)` 上浮 + 蓝调阴影 `0 12px 22px -12px rgba(0,112,194,.13)` + 边框变蓝,`.15s` |
| **步骤卡片(APP 外壳)** | 不上浮,仅 `border-color:#c3d6ec` + 底色 `#eef2f7` |
| **阶段卡片** | `border-color:var(--accent)` + 轻阴影 |
| **退焦阶段被 hover** | 临时恢复清晰(`opacity:.9; scale(1); blur 0`) |
| **次按钮 / 顶栏按钮** | 底色变 `--paper` 或翻蓝 |
| **图标圆按钮** | 翻蓝底白字(`.del`→红,`.copy`→橙) |
| **虚线添加** | `--accent-soft` 底 + 蓝边 |
| **搜索结果项 / 侧栏项** | `--accent-soft` / `--paper` 底,`.12s` |
| **插入条 `.arrow:hover`** | 浮出 `.insert-here` 按钮(`opacity 0→1`) |

---

## 6. Focus Effects 焦点

- **可编辑文本聚焦:** `outline:2px dashed var(--accent); outline-offset:3px; border-radius:4px`。
- **输入框聚焦:** `border-color:var(--accent)`;顶栏搜索外层 `:focus-within` 变白底蓝边。
- **当前阶段圆点:** `0 0 0 3px var(--accent-soft)` 蓝色光环 + `scale(1.35)`。
- **拖拽目标:** `0 0 0 3px var(--gold)`(节点)/ `0 0 0 3px var(--accent)`(阶段)彩色环。
- **搜索结果项 focus:** `--accent-soft` 底,`outline:none`(用底色替代轮廓)。

> 焦点/当前/命中态偏好用 **`0 0 0 3px <soft>` 光环**表达,而非粗描边。

---

## 7. Card Animations 卡片动效

- 新增/展开:`.node.anim-in` 跑 `sop-node-in`(弹性上移淡入),动画结束后 JS 移除类。
- 删除/收起:`.node.anim-out` 跑 `sop-node-out`(下移淡出,`.16s`)后移除节点。
- 拖拽:`.dragging{opacity:.4}`,目标 `.drag-over` 彩色环。
- 搜索退焦:`.search-dim{opacity:.28}`。

---

## 8. Button Animations 按钮动效

- **按下回弹:** 主要按钮/图标按钮/卡片操作按钮统一
  `transition:transform .12s ease, …`,`:active{ transform:scale(.94) }` —— 按下缩到 .94 再
  平滑弹回,约 120ms,手感"实在"。
- hover 为颜色/边框/阴影过渡 `.15s`。

---

## 9. Modal Transitions 弹窗过渡

- **详情弹窗 `#detail`:** 遮罩用 `visibility+opacity` `.2s ease` 淡入淡出;面板 `.detail-panel`
  从 `scale(.92) translateY(14px); opacity:0` → `scale(1) translateY(0); opacity:1`,
  `transform .32s cubic-bezier(.22,1,.36,1), opacity .26s ease`,`transform-origin` 由 JS 按
  被点卡片位置设定 —— **从卡片处放大铺开到全屏**。
- **迷你弹窗 `#miniModal`:** `display:flex` 切换显示(无独立入场动画)。
- **图片大图 `#imgview`:** `display:grid` 切换。

---

## 10. Sidebar Transitions 侧栏过渡

- 桌面:侧栏固定常驻,无入场动画;阶段项激活态为底色/左条颜色过渡。
- 移动端(`≤760px`):抽屉 `transform:translateX(-100%)` ↔ `.open translateX(0)`,
  `transition:transform .2s`,叠加 `0 0 0 100vmax rgba(0,0,0,.3)` 全屏遮罩。
- `flow-rail` 圆点/连接段:`background .35–.4s`、圆点 `transform .35s cubic-bezier(.22,1,.36,1)`。

---

## 11. Loading / Charts 加载与图表

- **无加载动画、无骨架屏、无图表动效。** 数据为本地即时渲染,不存在异步等待态。
- 若未来引入异步加载,建议沿用 `--accent` + `.22,1,.36,1`,做轻量淡入而非旋转 spinner(见 §14)。

---

## 12. Page Transitions / Scrolling / Micro-interactions

**Page / view 切换**
- 模块切换 `switchModule()`、阶段切换 `.detail-active` 为即时显示(无整页转场),靠卡片弹入与
  阶段 pop 提供"变化感"。

**卡片内联展开子步骤(`expandNodeInline` / `.node-inline-substeps`)**
- 点卡片 → `.steps` 网格临时切单列 flow(`has-expanded`),被点卡片置顶并锁定 `--card-w`、
  统一靠左,其余卡片 `display:none`。
- **展开:** 子步骤容器 `max-height:0→4000px` + `opacity` 过渡
  `.5s cubic-bezier(.22,1,.36,1)` / `.4s ease`(下一帧加 `.show` 触发);内部每个元素
  `ni-item-in`(`.42s cubic-bezier(.34,1.4,.64,1)`,上移淡入),按 `--i` 递增 `.09s` 延迟
  **逐个出现**。
- **滚动补偿:** 布局切换前后量被点卡片视口 `top`,差值用 `window.scrollBy` 补回,卡片"留在
  原地"、不整页跳动。
- **收起(← 返回 / ESC):** 用更短的 `max-height .16s ease` 快速收拢(约 76ms),收完再恢复
  其余卡片;比展开明显更快,返回干脆。
- reduced-motion:过渡/逐个淡入全部关闭,直接即时显示。

**Scrolling 滚动行为**
- **scroll-spy:** `window scroll`(passive)→ `requestAnimationFrame(updateScrollSpy)`,按阅读
  位置给阶段加 `.is-current/.is-passed/.is-upcoming` 并点亮 rail 圆点。
- **吸顶:** 阶段标题 `position:sticky; top:var(--topbar-h)`;吸顶后 `.head-stuck` 加分层投影。
- **锚定跟随 `anchorHead`:** 展开阶段时用 RAF 在 ~320ms 内把标题平滑锚定到吸顶位置,避免抖动。
- `scrollStageIntoAnchor(stage, smooth)` 平滑滚动定位。

**Micro-interactions 微交互**
- 折叠箭头 `.stage-toggle` 展开时 `rotate(90deg)`,`.24s cubic-bezier(.4,0,.2,1)`。
- 阶段圆点 scroll-spy 放大 + 光环。
- 控制按钮显隐:`opacity .15s, max-width .15s`(收起时连占位宽一起收掉,防误触)。
- toast `opacity .3s` 淡入,1600ms 后淡出。
- 搜索错误抖动 + 红框 + 计数染红(280ms)。

---

## 13. Reduced Motion 减弱动态(必须保留)

`@media (prefers-reduced-motion: reduce)` 块统一关闭/降级:

- `steps-wrap` 高度过渡 → `none`;阶段 pop、节点弹入弹出 → `animation:none`。
- 卡片 hover 上浮 → `transform:none`;搜索抖动 → `none !important`。
- 详情面板 → 仅 `opacity .2s`,不缩放位移。
- 阶段圆点/退焦/rail 圆点缩放 → `transform:none; filter:none`。
- 按钮按下缩放 → `transform:none`。

**新增任何动效都必须在此块提供静态/降级回退。**

---

## 14. Recommended Future Animation Rules 未来动效建议

1. **缓动统一三选一:** 出现/移动 → `cubic-bezier(.22,1,.36,1)`;弹性强调 →
   `cubic-bezier(.34,1.56,.64,1)`;退出 → `ease-in`(且更短)。不要引入新曲线。
2. **时长落在既有梯度:** 微交互 `.12–.15s`、状态切换 `.2–.42s`,退出≈进入的一半。
3. **幅度克制:** 位移 ≤ ~18px、缩放 0.92–1.02 区间,符合现有"细腻物理感"。
4. **高亮用光环不用粗描边:** 沿用 `0 0 0 3px <soft-color>`。
5. **任何异步态用淡入,不用旋转 spinner**,与"克制、企业级"气质一致。
6. **必配 reduced-motion 回退**,并保证动画不破坏 编辑→保存→重开 往返(运行时类名不写入保存)。
7. **方向承载语义:** 展开从来源点/顶部生长,退出向下淡出,保持空间一致性。
