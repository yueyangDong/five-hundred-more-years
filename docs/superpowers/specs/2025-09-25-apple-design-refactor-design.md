# 还想再活五百年 · Apple 风格界面改造设计

- 日期：2025-09-25
- 设计依据：`emilkowalski/skills` 仓库的 `apple-design` skill（本项目已安装到 `.reasonix/skills/apple-design/SKILL.md`），
  内容源自 Apple WWDC `Designing Fluid Interfaces` 等设计演讲的 Web 化翻译。
- 作用范围：仅视觉与交互表现层（`index.html` 的 1 个 `<style>` 块 + HTML 骨架 + 少量 JS 动效胶水）。
  **不触碰**任何事件池、数值平衡、命理算法、存档结构。

## 1. 目标与非目标

目标：把已有一层“v3 Apple 式界面”（`index.html` 636–813 行）升级为完整的 Apple 设计体系——
语义色 + 双主题、材质层级、spring 动效、三条无障碍降级。

非目标：不改玩法、不改文案口径、不引入任何第三方依赖（项目卖点是单文件零依赖，spring 引擎手写）。

## 2. 设计 Token

| 类别 | 取值 |
| --- | --- |
| 字体 | UI/正文 `system-ui, -apple-system, "PingFang SC", "Microsoft YaHei"`；品牌/干支/碑文/印章保留衬线（东方符号） |
| 字号与排版 | 大标题 `letter-spacing: -0.02em` + `line-height 1.05`；正文 `letter-spacing: 0` + `1.5`；间距用 `rem` 以随系统字号缩放 |
| 中性色 | `label / secondaryLabel / tertiaryLabel` = `#000` 87% / 60% / 30%（深色反相） |
| 系统色 | blue `#007AFF`、red `#FF3B30`、green `#34C759`、orange `#FF9500`、yellow `#FFCC00`、purple `#AF52DE`、gray `#8E8E93` |
| 背景 | `systemBackground` / `secondarySystemGroupedBackground`（卡片实色，不做半透明） |
| 东方色 | `--accent`（朱红）仅用于：印章、天干、命魂、命簿符号；`--gold` 仅用于：稀有事件、功德、天赋 |
| 栅格 | 8pt：4 / 8 / 12 / 16 / 24 / 32 |
| 圆角 | 同心：外层 20 → 内嵌 14 → 内部 10（内 = 外 − 间距） |

主按钮、链接、选中态改用 `systemBlue`（原来使朱红）；朱红退为符号点缀色——这是“Apple 为主、东方符号保留朱金”的直接后果。

## 3. 材质与层级

- 浮动 chrome（侧边栏、顶栏、底部合规条、弹层 scrim）使用材质：`backdrop-filter: blur(20px) saturate(180%)`，内容从其下方滚过。
- 内容卡片改为实色（`secondarySystemGroupedBackground` + 1px hairline），**避免浅色半透明叠浅色半透明**导致可读性坍塌。
- 大面板（弹窗/侧边栏）模糊与阴影强于小 chip；层级由材质重量表达。
- 滚动边缘用渐隐遮罩替代 1px 分割线（仅当浮动 chrome 与内容重叠处）。
- 弹层入场同时动 blur 半径与 scale（materialize），不是单纯 opacity 淡入。

## 4. 动效规范（spring）

手写 spring 引擎（≈50 行，rAF，半隐式欧拉 + 固定子步）。参数采用 Apple 的 damping / response 口径：

| 场景 | damping | response |
| --- | --- | --- |
| 屏幕转场、卡片错峰入场、事件卡入场 | 1.0 | 0.35 |
| 弹层 / sheet（materialize） | 0.8 | 0.3 |
| 选项选中 snap（点击带动量） | 0.85 | 0.28 |
| 按钮位移类反馈 | 1.0 | 0.25 |

规则（SKILL 原文口径）：

- 反馈发生在 **pointer-down**，不是 release；视觉即时（100ms ease-out）。
- 转场**从当前值起播**（读实时 transform/opacity），任何时刻可被打断并从当前位置续接——不用固定时长 `@keyframes` 承载手势相关的动效。
- 进出同一路径；弹层 `transform-origin` 锚定触发元素。
- 回弹（damping < 1）只给**带动量**的交互（点击选中、flick），纯淡入不弹。
- 只动 `transform` / `opacity`，必要时 `will-change`。

## 5. 无障碍

三条独立媒体查询：

- `prefers-reduced-motion: reduce` → 位移/spring 全部降级为 200ms opacity 交叉淡入，去掉 overshoot。
- `prefers-reduced-transparency: reduce` → 材质转实底、去 blur。
- `prefers-contrast: more` → 近实底背景 + 明确描边。

## 6. 主题

`html[data-theme]` 三态：`auto`（默认，跟随 `prefers-color-scheme`）/ `light` / `dark`，选择写入 `localStorage`。
切换入口放顶栏（与现有“极速模式”按钮同排）。

## 7. 实现落点

1. `</style>` 前追加 v4 层（覆盖 v3，不就地重写 43KB 旧样式，保持可回滚）。
2. 顶栏加 `#btnTheme`。
3. JS 追加：spring 引擎、主题控制、`show()` 转场改写、弹层 materialize、全局 pointerdown 反馈。

## 8. 验证

用仓库现有 Playwright 链路 `_shots.js` 截 10 张图（桌面 5 屏 + 移动 3 屏）对照检查：浅色/深色各一轮、窄屏底部标签栏不重叠、无页面报错。
