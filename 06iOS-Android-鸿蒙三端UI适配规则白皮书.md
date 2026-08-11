# iOS / Android / 鸿蒙 三端 UI 适配规则白皮书

面向：iOS SwiftUI / UIKit、Android Jetpack Compose / View、鸿蒙 ArkTS / ArkUI  
生成口径：基于 `鸥我白盒8.2-改ui-开发需求备注版-GitHub最终交付版.html` 最终交付结果  
版本日期：2026-08-02  
单位说明：iOS 使用 pt，Android 使用 dp/sp，鸿蒙使用 vp/fp；Web 原型中的 `px` 仅作为逻辑像素近似值，原生实现时必须换成平台逻辑单位。

## 0. 最终修改状态

本白皮书基于最终原型状态重新生成，不沿用旧审查中“未改前”的结论。外观设置只考虑三种模式：跟随系统、浅色、深色；不再考虑暖色/冷色。

| 审查项 | 最终原型状态 | 已落地内容 | 剩余风险 | 开发落地要求 |
|---|---|---|---|---|
| 2. 暗黑模式 | 部分完成 | 已接入 `data-owo-theme="system/light/dark"`、`color-scheme: light dark`、暗色 token、`prefers-color-scheme` 回退；HTML 中大量白底已改为运行时 token | 仍存在少量玻璃态 `rgba(255,255,255,...)`、图片未提供 dark 版本；品牌紫 `#9F97EB` 在部分浅底上对比度偏弱 | 原生端必须使用系统语义色 / Material colorScheme / ArkUI 资源色，不依赖系统强制反色 |
| 3. 点击区域 | 部分完成 | 已定义 `--owo-touch-target: 48px`；按钮、Tab、导航项、小按钮通过命中扩展层补齐 | 个别 `div onClick`、自绘 checkbox 仍可能只有 20×20 视觉与命中范围 | 所有可点元素 iOS ≥44×44pt，Android/鸿蒙 ≥48×48dp/vp |
| 4. 安全区 | 部分完成 | 已增加 `--owo-safe-*`、`--owo-safe-status-top`、`--owo-nav-safe-bottom`；状态栏、Home Indicator、底部导航已使用安全区 token | 首页搜索头部仍存在固定顶部 padding；横屏和 Dynamic Island 场景仍需真机确认 | 原生端全部读取系统 inset，不写死状态栏、刘海、手势条高度 |
| 5. 字体/平台栈 | 部分完成 | 已加入 SF Pro、Roboto/Noto、HarmonyOS Sans、MiSans、OPPO Sans、vivo Sans 等平台字体栈 | 原型仍有 10px、10.5px、11px 等小号文本；未完整实现 Dynamic Type / fontScale | 原生端所有正文与控件文本使用动态字体体系；最小可读字号 ≥11 |
| 6. 平台组件反馈 | 部分完成 | 已增加 Android/Harmony 平台 toast/snackbar 位置差异、按钮 state layer、底部弹窗平台样式能力 | 仍是 Web 高保真统一原型，不是完整原生组件分支 | iOS 用原生导航/表单/弹窗；Android 用 Material 3；鸿蒙用 ArkUI/Harmony 组件 |
| 7. 折叠屏/大屏 | 已补能力层 | 已增加 `data-owo-layout="compact/medium/expanded"`、`data-owo-foldable`、`.owo-adaptive-screen`、`.owo-adaptive-list-detail`、`.owo-navigation-rail-slot`、`.owo-foldable-safe`、`.owo-foldable-hinge-mask` | 当前手机预览不会自动变双栏；具体页面需显式接入这些 class | ≥600dp/vp 使用 List+Detail；≥840dp/vp 增加 Navigation Rail；折痕区域禁止放主 CTA |

结论：最终修改已经覆盖用户指定的 2、3、4、5、6、7，但其中 2-6 属于“低侵入能力修复 + 关键风险下降”，不是全量原生级重构。若按上架级规范验收，仍需要针对颜色对比、动态字体、全部点击语义、图片暗色版本和真机安全区做二次收敛。

## 1. 全局适配规则总表

| 规范项 | iOS 标准 | 原生 Android 标准 | 华为鸿蒙 | 小米 HyperOS | OPPO ColorOS | vivo OriginOS | 荣耀 MagicOS | 统一处理建议 |
|---|---:|---:|---:|---:|---:|---:|---:|---|
| 系统字体 | SF Pro / San Francisco | Roboto / Noto Sans | HarmonyOS Sans | MiSans | OPPO Sans | vivo Sans | Honor Sans / 系统默认 | 按平台分支设置字体栈；中文 fallback 加 PingFang SC / Noto Sans SC |
| 字号单位 | pt | sp | fp | sp | sp | sp | sp | 禁止正文固定 px；跟随系统字体缩放 |
| 页面大标题 | 34pt Bold | 57sp Display Large | 32fp Bold 工程基线 | 36sp Bold 工程基线 | 34sp Bold 工程基线 | 34sp Bold 工程基线 | 34sp Bold 工程基线 | 营销/首页可用大标题，业务页谨慎使用 |
| 正文 | 17pt Regular | 16sp Body Large | 16fp | 16sp | 16sp | 16sp | 16sp | 正文统一 16-17 逻辑单位 |
| 最小可读字号 | 11pt | 11sp | 11fp | 11sp | 11sp | 11sp | 11sp | 低于 11 只允许装饰性数字，不承载关键信息 |
| 最小点击区域 | 44×44pt | 48×48dp | 48×48vp | 48×48dp | 48×48dp | 48×48dp | 48×48dp | 视觉可小，命中框必须达标 |
| 页面左右边距 | 16pt | 16dp | 24vp 工程基线 | 16dp | 16dp | 16dp | 16dp | 手机 16；鸿蒙/平板/折叠展开 24-32 |
| 状态栏高度 | SE 20pt；刘海 44pt；Dynamic Island 59pt 设计基线 | 默认约 24dp，必须动态读取 | 24-32vp 工程基线 | 24-28dp 工程基线 | 24-26dp 工程基线 | 24-26dp 工程基线 | 24-28dp 工程基线 | 不写死，全部读取系统 inset |
| 底部安全区 | 全面屏 34pt；SE 0pt | 0-24dp 工程基线 | 16-24vp 工程基线 | 16-20dp 工程基线 | 12-16dp 工程基线 | 12-16dp 工程基线 | 16-20dp 工程基线 | 底部操作区 padding = bottom inset + 16 |
| 顶部导航栏高 | 标准 44pt；Large Title 96pt | Top App Bar 64dp | 56vp 工程基线 | 56dp 工程基线 | 56dp 工程基线 | 56dp 工程基线 | 56dp 工程基线 | iOS 首页/列表可 Large Title；Android/鸿蒙默认 56-64 |
| 底部导航栏高 | Tab Bar 49pt，不含 Home Indicator | Navigation Bar 80dp | 56vp 工程基线 | 56dp 工程基线 | 56dp 工程基线 | 56dp 工程基线 | 56dp 工程基线 | iOS 49+safe；Android 80+inset；厂商 56-80+inset |
| 图标尺寸 | Tab 25×25pt；工具栏约 22pt | 24×24dp | 24×24vp | 24×24dp | 24×24dp | 24×24dp | 24×24dp | 源文件用矢量；按平台换 SF Symbols / Material Symbols 风格 |
| 网格基准 | 无强制，常用 4/8pt | 8dp | 4vp | 8dp | 8dp | 8dp | 8dp | 组件内部 4，布局间距 8 |
| 圆角基准 | 无强制，常用 8/12/16pt | Shape scale；按钮 8-20dp，卡片 12dp，Dialog 28dp | 4vp 基线 | 4dp 基线 | 4dp 基线 | 4dp 基线 | 4dp 基线 | token 化：按钮 8-14，卡片 12-16，弹窗 24-28 |
| 动效时长 | 250-400ms | 150-300ms | 200-300ms 工程基线 | 200-300ms 工程基线 | 200-300ms 工程基线 | 200-300ms 工程基线 | 200-300ms 工程基线 | iOS 用 spring；Android/鸿蒙用标准 easing |
| 暗黑模式 | 系统级语义色 | Material colorScheme | ArkUI 资源色 | Android dark resources + 厂商测试 | Android dark resources + 厂商测试 | Android dark resources + 厂商测试 | Android dark resources + 厂商测试 | 主动适配，禁止只依赖系统强制反色 |
| 大屏断点 | iPad ≥768pt 进入 regular 思路 | Compact <600；Medium 600-839；Expanded ≥840 | 同 Android，按 vp | 同 Android | 同 Android | 同 Android | 同 Android | ≥600 双栏，≥840 导航 rail + List/Detail |

## 2. 组件差异化规则

### 2.1 顶部导航栏

| 平台 | 高度 | 标题 | 左侧操作 | 右侧操作 | 背景/材质 | 阴影/分割线 | 工程实现 |
|---|---:|---|---|---|---|---|---|
| iOS | 44pt 标准；96pt Large Title；另加 top safe area | 标准 17pt Semibold；Large Title 34pt Bold | 返回箭头约 22pt + 可选文字，命中 44×44pt | 文字按钮 17pt 或图标 22pt | `systemBackground` / `ultraThinMaterial` | 0.5pt separator，通常无重阴影 | SwiftUI `NavigationStack`；UIKit `UINavigationController` |
| 原生 Android | 64dp | Title Large 22sp 或 Title Medium 16sp | 返回/菜单图标 24dp，命中 48×48dp | 图标 24dp，命中 48×48dp | `Surface` / `SurfaceContainer` | 0-3dp elevation 或 1dp outline | Compose `TopAppBar`；View `MaterialToolbar` |
| 华为鸿蒙 | 56vp + top inset | 20fp Semibold 工程基线 | 24vp，命中 48×48vp | 24vp 图标或文字 | ArkUI Navigation / 沉浸式材质按系统能力 | 1vp divider 或无 | ArkUI `Navigation` / `NavDestination` |
| 小米 HyperOS | 56dp + inset | 20sp Semibold 工程基线 | 24dp，命中 48×48dp | 24dp | 可适配标题栏模糊/沉浸 | 1dp divider | Android TopAppBar + HyperOS inset |
| OPPO ColorOS | 56dp + inset | 20sp Semibold 工程基线 | 24dp，命中 48×48dp | 24dp | 可适配动态取色 | 1dp divider | Android TopAppBar + ColorOS 主题测试 |
| vivo OriginOS | 56dp + inset | 20sp Semibold 工程基线 | 24dp，命中 48×48dp | 24dp | Surface | 1dp divider | Android TopAppBar |
| 荣耀 MagicOS | 56dp + inset | 20sp Semibold 工程基线 | 24dp，命中 48×48dp | 24dp | 避让灵动胶囊/挖孔 | 1dp divider | Android TopAppBar + display cutout |

原型落地提示：当前 Web 原型已有安全区 token，但首页搜索头仍有固定顶部 padding。原生实现时顶部导航必须全部挂系统安全区，不要复刻固定值。

### 2.2 底部导航

| 平台 | 高度 | 项数 | 图标 | 文本 | 选中态 | 安全区 | 工程实现 |
|---|---:|---:|---:|---|---|---|---|
| iOS | 49pt，不含 34pt Home Indicator | 3-5 | 25×25pt | 10-12pt | tint color + filled symbol | 总高 = 49pt + safeArea.bottom | SwiftUI `TabView`；UIKit `UITabBarController` |
| 原生 Android | 80dp | 3-5 | 24×24dp | Label 12sp | Primary + active indicator | 80dp + navigationBars inset | Compose `NavigationBar`；View `NavigationBarView` |
| 华为鸿蒙 | 56vp 工程基线 | 3-5 | 24×24vp | 12fp | 品牌色，可模糊背景 | 56vp + bottom inset | ArkUI `Tabs` |
| 小米 HyperOS | 56-80dp 工程基线 | 3-5 | 24×24dp | 12sp | 主色，支持动态模糊测试 | 56-80dp + gesture inset | Android NavigationBar |
| OPPO ColorOS | 56-80dp 工程基线 | 3-5 | 24×24dp | 12sp | 主色或动态取色 | 56-80dp + gesture inset | Android NavigationBar |
| vivo OriginOS | 56-80dp 工程基线 | 3-5 | 24×24dp | 12sp | 主色 | 56-80dp + gesture inset | Android NavigationBar |
| 荣耀 MagicOS | 56-80dp 工程基线 | 3-5 | 24×24dp | 12sp | 主色 | 56-80dp + gesture inset | Android NavigationBar |

统一规则：底部导航不能被系统手势条遮挡。当前原型已将底部导航接入 `--owo-nav-safe-bottom`，原生端需等价接入 `safeAreaInset` / `WindowInsets.navigationBars` / ArkUI 安全区。

### 2.3 按钮

| 平台 | 主按钮高度 | 命中区 | 圆角 | 字体 | 阴影 | 点击态 | 禁用态 |
|---|---:|---:|---:|---|---|---|---|
| iOS | 44-50pt | ≥44×44pt | 8-14pt；胶囊 999pt | 17pt Semibold | 通常无阴影 | opacity 0.65 或 scale 0.98；可 Haptic | 背景 `#E5E5EA`，文字 `#8E8E93` |
| Android | 40dp 视觉高；48dp 命中 | ≥48×48dp | 8-20dp，按 Material shape | Label Large 14sp Medium | Filled 无阴影；Elevated 1-3dp | Ripple / StateLayer 8-12% | onSurface 38% |
| 华为鸿蒙 | 40-48vp | ≥48×48vp | 8-16vp | 16fp Medium | 轻阴影或无 | 透明度/缩放/触感 | 低对比灰态 |
| 小米 HyperOS | 44-48dp | ≥48×48dp | 12dp 工程基线 | 15-16sp Semibold | 轻阴影 | Ripple/按压态 | 灰态 |
| OPPO ColorOS | 44-48dp | ≥48×48dp | 12dp 工程基线 | 15-16sp | 轻阴影 | Ripple/按压态 | 灰态 |
| vivo OriginOS | 44-48dp | ≥48×48dp | 12dp 工程基线 | 15-16sp | 轻阴影 | Ripple/按压态 | 灰态 |
| 荣耀 MagicOS | 44-48dp | ≥48×48dp | 12dp 工程基线 | 15-16sp | 轻阴影 | Ripple/按压态 | 灰态 |

原型落地提示：最终原型没有强行把所有小图标视觉尺寸改大，而是用 `.owo-hit-area` / 伪元素扩展命中区。原生端也应采用“视觉不变、命中区扩大”的方式。

### 2.4 输入框

| 平台 | 高度 | 圆角 | 字体 | 占位符 | 焦点态 | 错误态 |
|---|---:|---:|---|---|---|---|
| iOS | 44pt | 10-14pt | Body 17pt | `placeholderText`，≥11pt | 光标/边框品牌色 1pt | `#FF3B30`，提示 13pt |
| Android | 56dp | Filled/Outlined 4dp；品牌圆角可 12dp | Body Large 16sp | `onSurfaceVariant` | Label 浮动，主色 outline 1-2dp | Error `#B3261E`，supporting text 12sp |
| 华为鸿蒙 | 48-56vp | 8-12vp | 16fp | `text_tertiary` | 品牌色边框/光标 | error 12fp |
| 小米/OPPO/vivo/荣耀 | 48-56dp | 8-12dp | 16sp | 次要文字色 | 品牌色边框/光标 | error 12sp |

统一规则：清除按钮视觉 20-24，命中 44/48；错误提示离输入框 4-8；输入框不得低于 44/48 命中高度。

### 2.5 列表

| 平台 | 行高 | 左右内边距 | 分割线 | 操作 | 选中态 | 空态 |
|---|---:|---:|---|---|---|---|
| iOS | 最小 44pt；常规 56-72pt | 16pt | 0.5pt，通常左缩进 16pt | 左滑删除/更多 | 灰色 highlight | 标题 17pt，说明 15pt |
| Android | 48-72dp | 16dp | 1dp，全宽或内容缩进 | 长按菜单/多选 | StateLayer 8% | 图标 48dp + 14sp 说明 |
| 华为鸿蒙 | 48-72vp | 16-24vp | 1vp | 长按/悬停态 | hover/focus state | ArkUI Empty |
| 小米/OPPO/vivo/荣耀 | 48-72dp | 16dp | 1dp | 长按/多选 | Ripple/按压态 | 图标 + 文案 |

统一规则：头像 40-48；主标题 16-17；副标题 13-14；整行命中 Android/鸿蒙 不低于 48。

### 2.6 弹窗 / 对话框

| 平台 | 宽度 | 圆角 | 按钮排列 | 蒙层 | 关闭 | 动画 |
|---|---:|---:|---|---|---|---|
| iOS Alert | 270pt 基准，最大屏宽 - 32pt | 13-16pt | 1-2 个水平；3+ 垂直 | 黑色 20-40% | 通常不可点蒙层关闭 | 250-350ms spring/fade |
| Android Dialog | 280-560dp | 28dp | 文字按钮右对齐 | Scrim 32% | 可按业务点蒙层关闭 | 150-300ms fade/scale |
| 华为鸿蒙 | 280-560vp | 24-28vp | 右侧或底部 | Scrim 30-40% | 业务决定 | 200-300ms |
| 小米/OPPO/vivo/荣耀 | 280-560dp | 24-28dp | Android 规则 | Scrim 32% | 业务决定 | 200-300ms |

危险操作：iOS 用红色文字；Android/鸿蒙用 Error 色。确认按钮命中高度不得低于 44/48。

### 2.7 卡片

| 平台 | 圆角 | 内边距 | 阴影/Elevation | 背景 | 边框 |
|---|---:|---:|---|---|---|
| iOS | 12pt | 16pt | 微阴影 0-2pt 或无 | `secondarySystemBackground` | 0.5pt 可选 |
| Android | 12dp | 16dp | 1-8dp | `Surface` / `SurfaceContainer` | `OutlineVariant` 可选 |
| 华为鸿蒙 | 12-16vp | 16vp | 轻阴影 | `background_card` 工程基线 | 1vp 可选 |
| 小米/OPPO/vivo/荣耀 | 12-16dp | 16dp | 1-4dp | Surface | 1dp 可选 |

原型落地提示：当前原型为品牌卡片化风格，可保留；但 iOS 端应减少重阴影，Android 端可保留轻 elevation。

### 2.8 悬浮按钮

| 平台 | 尺寸 | 位置 | 阴影 | 展开方式 |
|---|---:|---|---|---|
| iOS | 56×56pt，谨慎使用 | 右下 16pt + safe | 轻阴影 | Menu / Sheet |
| Android | FAB 56×56dp；Small 40；Large 96；Extended 高 56 | 右下 16dp + inset | 6dp 基线 | Speed dial / Bottom Sheet |
| 华为鸿蒙 | 56×56vp | 右下 16-24vp + inset | 轻阴影 | 菜单/半屏 |
| 小米/OPPO/vivo/荣耀 | 56×56dp | 右下 16dp + inset | 4-6dp | 菜单/半屏 |

统一规则：iOS 优先导航栏按钮或底部固定 CTA；Android 发布/创建类入口可使用 FAB。

### 2.9 底部弹窗

| 平台 | 顶部圆角 | 高度 | 拖拽条 | 关闭方式 | 背景 |
|---|---:|---:|---|---|---|
| iOS Action Sheet | 13-28pt | 内容自适应，最大 90% 高 | 可选 36×5pt | 下滑/取消按钮 | material 或 system background |
| Android Bottom Sheet | 28dp | Peek 56-120dp；半屏 50%；全屏 100% | 32×4dp | 下滑/蒙层/返回键 | Surface |
| 华为鸿蒙 | 24-28vp | 50%-100% | 36×4vp | 下滑/返回 | background_card |
| 小米/OPPO/vivo/荣耀 | 24-28dp | 50%-100% | 32×4dp | 下滑/返回 | Surface |

统一规则：底部弹窗必须接入 bottom inset；全面屏设备底部至少保留 16 + inset。

### 2.10 加载状态

| 平台 | 转圈尺寸 | 线宽 | 颜色 | 骨架屏 | 位置 |
|---|---:|---:|---|---|---|
| iOS | 20pt small / 37pt large | 系统默认 | 系统 tint | 圆角 8-12pt，浅灰 shimmer | 居中或按钮内 |
| Android | 40dp CircularProgressIndicator | 4dp | Primary | 圆角 4-12dp，SurfaceVariant shimmer | 居中或列表内 |
| 华为鸿蒙 | 32-40vp | 3-4vp | brand/system | 卡片骨架 | 居中 |
| 小米/OPPO/vivo/荣耀 | 32-40dp | 3-4dp | brand/system | 卡片骨架 | 居中 |

统一规则：超过 400ms 显示加载；超过 8s 给失败/重试；按钮内 loading 保持按钮宽高不跳动。

## 3. 交互手势差异

| 交互 | iOS | 原生 Android | 华为鸿蒙 | 小米 HyperOS | OPPO ColorOS | vivo OriginOS | 荣耀 MagicOS |
|---|---|---|---|---|---|---|---|
| 返回上一级 | 左滑返回 + 左上角返回按钮 | 系统返回键 + 边缘滑动 + 预测返回 | 边缘滑动 + 返回键 | 底部/侧边手势 | 侧滑返回 | 侧滑返回 | 侧滑返回 |
| 下拉刷新 | 系统转圈，触发距离约 80-120pt | Material indicator，触发距离约 64-96dp | 鸿蒙转圈 | HyperOS 风格 | ColorOS 风格 | OriginOS 风格 | MagicOS 风格 |
| 上拉加载 | 自动加载或“加载更多” | 自动加载或“加载更多” | 自动加载 | 自动加载 | 自动加载 | 自动加载 | 自动加载 |
| 长按菜单 | 弱，Haptic Touch 辅助 | 强，上下文菜单 | 强 | 强，兼容传送门 | 强，兼容侧边栏 | 强 | 强，兼容任意门 |
| 多选删除 | 左滑删除优先 | 长按进入多选 | 长按多选 | 长按多选 | 长按多选 | 长按多选 | 长按多选 |
| 快捷操作 | Haptic Touch | 长按 | 长按/跨设备 | 长按识别 | 长按/智能侧边栏 | 长按/原子组件 | 长按/任意门 |
| 触觉反馈 | Selection/Impact/Notification | `HapticFeedbackConstants` | 系统触感 | 系统触感 | 系统触感 | 系统触感 | 系统触感 |

工程要求：iOS 返回必须支持 `interactivePopGestureRecognizer`；Android 接入 `OnBackPressedDispatcher` / Compose BackHandler，并适配预测返回；鸿蒙处理系统返回事件和 `router.back()`。

## 4. 暗黑模式适配

最终原型只保留三态：跟随系统、浅色、深色。Web 层通过 `OWO_APPEARANCE_KEY` 保存偏好，通过 `data-owo-theme` 与 `data-owo-appearance` 写入根节点；原生端应映射为系统主题、浅色主题、深色主题，不再扩展暖色/冷色设置。

| 规范项 | iOS | 原生 Android | 华为鸿蒙 | 小米 HyperOS | OPPO ColorOS | vivo OriginOS | 荣耀 MagicOS |
|---|---|---|---|---|---|---|---|
| 背景色 | `systemBackground`，常见 `#FFFFFF / #000000` | `Surface`，常见 `#FFFFFF / #121212` | `background`，`#FFFFFF / #0A0A0A` 工程基线 | `#FFFFFF / #121212` 工程基线 | `#FFFFFF / #121212` 工程基线 | `#FFFFFF / #121212` 工程基线 | `#FFFFFF / #121212` 工程基线 |
| 卡片背景 | `secondarySystemBackground`，常见 `#F2F2F7 / #1C1C1E` | `SurfaceContainer` / `SurfaceVariant` | `background_card` 工程基线 | SurfaceContainer | SurfaceContainer | SurfaceContainer | SurfaceContainer |
| 主文字色 | `label`，常见 `#000000 / #FFFFFF` | `onSurface` | `text_primary` | `text_primary` | `text_primary` | `text_primary` | `text_primary` |
| 次要文字 | `secondaryLabel` | `onSurfaceVariant` | `text_secondary` | `text_secondary` | `text_secondary` | `text_secondary` | `text_secondary` |
| 占位文字 | `placeholderText` | `onSurfaceVariant` 60% | `text_tertiary` | `text_tertiary` | `text_tertiary` | `text_tertiary` | `text_tertiary` |
| 分割线 | `separator` | `outline` / `outlineVariant` | `divider` | `divider` | `divider` | `divider` | `divider` |
| 实现方式 | SwiftUI `Color(.systemBackground)`；UIKit `UIColor.systemBackground` | `MaterialTheme.colorScheme` + `values-night` | ArkUI token / resource | Android dark resources + 厂商真机测试 | Android dark resources + 厂商真机测试 | Android dark resources + 厂商真机测试 | Android dark resources + 厂商真机测试 |

注意：国产 Android 系统常有全局深色/反色策略。应用未主动适配时，图片、头像、图标、品牌渐变可能被系统算法错误处理。必须主动提供 dark resources、矢量图标 tint、图片 dark variant 或关闭特定图片的自动反色。

## 5. 字体层级规范

| 层级 | iOS SF Pro | Android Roboto | 华为 HarmonyOS Sans | 小米 MiSans | OPPO Sans | vivo Sans | 荣耀 Honor Sans |
|---|---:|---:|---:|---:|---:|---:|---:|
| 页面大标题 | Large Title 34pt Bold | Display Large 57sp | 32fp Bold 工程基线 | 36sp Bold 工程基线 | 34sp Bold 工程基线 | 34sp Bold 工程基线 | 34sp Bold 工程基线 |
| 一级标题 | Title 1 28pt Bold | Headline Large 32sp | 28fp Bold 工程基线 | 28sp Bold 工程基线 | 28sp Bold 工程基线 | 28sp Bold 工程基线 | 28sp Bold 工程基线 |
| 二级标题 | Title 2 22pt Semibold | Headline Medium 28sp | 24fp Semibold 工程基线 | 24sp Semibold 工程基线 | 24sp Semibold 工程基线 | 24sp Semibold 工程基线 | 24sp Semibold 工程基线 |
| 页面标题/导航标题 | Headline 17pt Semibold | Headline Small 24sp / Title Large 22sp | 20fp Semibold 工程基线 | 22sp Semibold 工程基线 | 22sp Semibold 工程基线 | 22sp Semibold 工程基线 | 22sp Semibold 工程基线 |
| 正文 | Body 17pt Regular | Body Large 16sp | 16fp Regular | 16sp Regular | 16sp Regular | 16sp Regular | 16sp Regular |
| 辅助文字 | Subhead 15pt Regular | Body Medium 14sp | 14fp Regular | 14sp Regular | 14sp Regular | 14sp Regular | 14sp Regular |
| 脚注 | Footnote 13pt Regular | Body Small 12sp | 12fp Regular | 12sp Regular | 12sp Regular | 12sp Regular | 12sp Regular |
| 标签/提示 | Caption 1 12pt / Caption 2 11pt | Label Medium 12sp / Label Small 11sp | 11-12fp | 11-12sp | 11-12sp | 11-12sp | 11-12sp |

工程要求：

- iOS：SwiftUI 使用 `.font(.body)`、`.font(.largeTitle)` 等 Dynamic Type；UIKit 使用 `UIFont.preferredFont(forTextStyle:)`。
- Android：Compose 使用 `MaterialTheme.typography`；View 使用 `TextAppearance.Material3.*`。
- 鸿蒙：ArkUI 使用 fp，并跟随系统字体缩放。
- 当前原型已有平台字体 token，但并未替换所有旧字号。开发端不要照抄 10px、10.5px 这类原型小字。

## 6. 折叠屏适配规则

最终原型已补入非侵入式大屏/折叠能力层：`updateOwoAdaptiveRuntime()` 会写入 `data-owo-layout="compact/medium/expanded"`，CSS 提供 `.owo-adaptive-screen[data-owo-adaptive="auto"]`、`.owo-adaptive-list-detail`、`.owo-navigation-rail-slot`、`.owo-foldable-safe`、`.owo-foldable-hinge-mask`。为了不改变当前手机高保真预览，页面不会自动变双栏，业务页面需要显式接入这些类。

| 场景 | 华为 Mate X 系列 | OPPO Find N 系列 | 小米 MIX Fold 系列 | 荣耀 Magic V 系列 | 三星 Galaxy Z 系列 |
|---|---|---|---|---|---|
| 折叠态 | Compact <600vp，单列 | Compact <600dp，单列 | Compact <600dp，单列 | Compact <600dp，单列 | Compact <600dp，单列 |
| 展开态 | ≥600 双栏 List+Detail；≥840 可三栏/Navigation Rail | 同左 | 同左 | 同左 | 同左 |
| 中间折痕 | hinge 区域禁止放 CTA、输入框、主按钮 | 同左 | 同左 | 同左 | Jetpack WindowManager display features |
| 悬停态 | 上下分区，控制区放下半屏 | 同左 | 同左 | 同左 | tabletop posture 上下分区 |
| 视频播放 | 播放区避开折痕，弹幕不跨折痕 | 全屏或上下分区 | 全屏或上下分区 | 全屏或上下分区 | tabletop 上播放下控制 |
| 图片选择器 | 3-4 列网格；详情右侧 | 自适应网格 | 自适应网格 | 自适应网格 | 自适应网格 |
| 导航 | 折叠态 Bottom Nav；展开态 Navigation Rail | 同左 | 同左 | 同左 | 同左 |

断点规则：

- Compact：0-599dp/vp，单列，Bottom Navigation。
- Medium：600-839dp/vp，双栏，List+Detail。
- Expanded：≥840dp/vp，Navigation Rail + List/Detail。
- 所有平台：折痕/铰链矩形区域视为不可交互安全区。

## 7. 各厂商应用商店上架规范

以下表格用于设计交付基线。应用商店控制台规则更新频繁，提交前必须以对应控制台实时校验为准。

| 规范项 | Apple App Store | 华为应用市场 | 小米应用商店 | OPPO 软件商店 | vivo 应用商店 | 荣耀应用市场 |
|---|---:|---:|---:|---:|---:|---:|
| 预览图尺寸 | 6.7" 常用 1290×2796px；6.5" 常用 1242×2688px | 1080×1920px 工程基线 | 1080×1920px 工程基线 | 1080×1920px 工程基线 | 1080×1920px 工程基线 | 1080×1920px 工程基线 |
| 预览图数量 | 最多 10 张 | 最多 5 张工程基线 | 最多 5 张工程基线 | 最多 5 张工程基线 | 最多 5 张工程基线 | 最多 5 张工程基线 |
| 应用图标 | 1024×1024px | 512×512px 工程基线 | 512×512px 工程基线 | 512×512px 工程基线 | 512×512px 工程基线 | 512×512px 工程基线 |
| 应用名称长度 | 30 字符 | 8 个汉字工程基线 | 8 个汉字工程基线 | 8 个汉字工程基线 | 8 个汉字工程基线 | 8 个汉字工程基线 |
| 隐私政策 | 必须 | 必须 | 必须 | 必须 | 必须 | 必须 |

## 8. 跨平台核心差异速查表

| 差异项 | iOS | 原生 Android | 华为鸿蒙 | 小米 HyperOS | OPPO ColorOS |
|---|---|---|---|---|---|
| 导航心智 | Navigation Stack + Tab Bar | Top App Bar + Navigation Bar/Drawer | Navigation + Tabs，强调跨设备 | Android 基础 + HyperOS 系统手势/模糊 | Android 基础 + ColorOS 动态取色/流体云场景 |
| 返回 | 左滑返回强预期 | 系统返回键/边缘返回/预测返回 | 系统返回 + ArkUI router | 侧滑/底部手势 | 侧滑返回 |
| 点击最小值 | 44×44pt | 48×48dp | 48×48vp | 48×48dp | 48×48dp |
| 底部安全区 | Home Indicator 34pt | 手势条高度不固定 | 手势栏 16-24vp 工程基线 | 16-20dp 工程基线 | 12-16dp 工程基线 |
| 字体 | SF Pro / PingFang | Roboto / Noto | HarmonyOS Sans | MiSans | OPPO Sans |
| 暗色策略 | 语义色自动切换 | Material colorScheme | ArkUI token/资源 | 系统深色 + 厂商强制反色测试 | 系统深色 + 厂商强制反色测试 |
| 反馈 | Haptic + opacity/scale | Ripple/StateLayer + Snackbar | 触感 + ArkUI 反馈 | Ripple/系统触感 | Ripple/系统触感 |
| 大屏 | iPad size class | WindowSizeClass / WindowInsets | 折叠屏与跨设备 | 折叠屏与手势栏差异 | 折叠屏与侧边栏场景 |
| 实时活动/胶囊 | Live Activities / Dynamic Island | 通知/系统 surfaces | 实况窗/服务卡片场景 | 灵动脑门场景 | 流体云场景 |

## 9. Web 原型到原生 App 的分别调整

当前 HTML 是 Web 高保真统一原型，不是三套原生 UI。为避免再出现“改 2-7 却全局样式变化过大”的问题，原生化时按以下最小原则拆分：

| 原型元素 | iOS 调整 | Android 调整 | 鸿蒙调整 | 是否必须改 |
|---|---|---|---|---|
| 外观设置 | 映射为系统/浅色/深色；使用语义色 | 映射为 system/light/dark；使用 colorScheme | 映射为 ArkUI 资源主题 | 必须 |
| 顶部安全区 | 使用 `safeAreaInset(edge:.top)` / UIKit safeArea | 使用 `WindowInsets.statusBars` / edge-to-edge | 使用窗口安全区能力 | 必须 |
| 底部导航 | `TabView` / `UITabBarController`，49pt + safe | Material `NavigationBar`，80dp + inset | ArkUI `Tabs` + safe area | 必须 |
| 小图标按钮 | 视觉可保留，命中扩到 44pt | 视觉可保留，命中扩到 48dp | 视觉可保留，命中扩到 48vp | 必须 |
| 颜色 token | `UIColor` / SwiftUI semantic Color | Material `ColorScheme` | ArkUI resource color | 必须 |
| 字体 token | Dynamic Type | fontScale / Typography | fp + 系统缩放 | 必须 |
| 卡片阴影 | 降低阴影，偏系统层次 | 可用 1-3dp elevation | 轻阴影/卡片资源色 | 建议 |
| 折叠屏 | iPad/regular width 双栏 | WindowSizeClass + WindowManager | 展开态 List+Detail | 严重场景必须 |
| 图片暗色版 | 提供 dark asset 或 tint | night drawable / vector tint | dark resource | 必须 |

## 10. 最小可修改清单

只改致命和严重问题，且尽量不改变视觉风格时，按以下顺序处理：

1. 暗黑模式：保留最终原型的三态外观设置，只做系统/浅色/深色；原生端全部改用语义色或主题 token，补图片暗色版本。
2. 点击区域：所有按钮、Tab、图标、checkbox、列表行命中区补到 iOS 44×44pt、Android/鸿蒙 48×48dp/vp；视觉尺寸不必同步放大。
3. 安全区：删除固定顶部/底部避让值；统一接入 status bar、cutout、navigation bar、Home Indicator、折叠铰链 inset。
4. 字体：正文和控件文本换成 Dynamic Type / Material Typography / ArkUI fp；低于 11 的文本不承载关键信息。
5. 平台组件：iOS 用 NavigationStack、TabView、Alert、ActionSheet；Android 用 TopAppBar、NavigationBar、Dialog、BottomSheet、Snackbar；鸿蒙用 ArkUI Navigation、Tabs、Dialog、Sheet。
6. 折叠屏：对列表详情页接入双栏模板；主操作按钮、输入框和弹窗避开折痕区域；≥840 显示导航 rail。
7. 颜色对比：单独检查品牌紫 `#9F97EB`、三级灰 `#999999`、白色文字叠品牌色等组合；正文对比达到 4.5:1，大字号/图标达到 3:1。

## 11. 实现映射

| 能力 | SwiftUI / UIKit | Jetpack Compose / View | ArkTS / ArkUI |
|---|---|---|---|
| 主题三态 | `ColorScheme?` + `@Environment(\.colorScheme)`；UIKit traitCollection | `isSystemInDarkTheme()` + user setting；`MaterialTheme(colorScheme=...)` | 系统深浅色监听 + resource token |
| 安全区 | `safeAreaInset`、`GeometryReader.safeAreaInsets`、UIKit `safeAreaInsets` | `WindowInsets.statusBars/navigationBars/displayCutout` | ArkUI 安全区/窗口 inset |
| 字体缩放 | SwiftUI text styles；`UIFontMetrics` | sp + `MaterialTheme.typography` | fp + 系统字体缩放 |
| 命中区 | `.frame(minWidth:44,minHeight:44)` / `contentShape` | `Modifier.minimumInteractiveComponentSize()` | 外层容器 48vp + 点击事件 |
| 返回 | `NavigationStack`；UIKit interactive pop | BackHandler / OnBackPressedDispatcher / predictive back | 系统返回事件 + router |
| 折叠 | iPad size class | WindowSizeClass + Jetpack WindowManager | 窗口宽度 + 折痕/悬停能力 |
| 反馈 | Haptic + system animation | Ripple/StateLayer + Snackbar | ArkUI toast/dialog/触感能力 |

## 12. 参考来源

- Apple Human Interface Guidelines: <https://developer.apple.com/design/human-interface-guidelines/>
- Apple Human Interface Guidelines - Typography: <https://developer.apple.com/design/human-interface-guidelines/typography>
- Apple Human Interface Guidelines - Layout: <https://developer.apple.com/design/human-interface-guidelines/layout>
- Apple Human Interface Guidelines - SF Symbols: <https://developer.apple.com/sf-symbols/>
- Material Design 3: <https://m3.material.io/>
- Material Design 3 - Typography: <https://m3.material.io/styles/typography/overview>
- Material Design 3 - Color System: <https://m3.material.io/styles/color/system/overview>
- Material Design 3 - Navigation Bar: <https://m3.material.io/components/navigation-bar/overview>
- Android Developers - Window Insets: <https://developer.android.com/develop/ui/views/layout/edge-to-edge>
- Android Developers - Large Screens and Foldables: <https://developer.android.com/guide/topics/large-screens>
- Android Developers - Jetpack WindowManager: <https://developer.android.com/develop/ui/compose/layouts/adaptive/foldables>
- HarmonyOS Design: <https://developer.huawei.com/consumer/en/design/>
- App Store Connect Screenshot Specifications: <https://developer.apple.com/help/app-store-connect/reference/screenshot-specifications>
