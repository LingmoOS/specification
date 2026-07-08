# Lingmo Desktop Architecture v2

> 基于"长期维护、独立发布、职责单一、可复用、可测试"原则重新划分

---

## 总览

**47 个仓库**，按子系统分组。

每个仓库满足以下条件：

- **长期维护**：职责边界清晰，不会随需求变化而膨胀或需要拆分
- **独立发布**：有独立的版本号和发布节奏，关键修复不阻塞其他组件
- **职责单一**：只做一件事，且做这一件事所需的全部代码在仓库内
- **可复用**：对外提供稳定的 API，其他组件或第三方开发者可按需依赖
- **可测试**：依赖边界明确，可独立编写单元测试和集成测试

---

## 1. UI 框架层 (6 个仓库)

UI 框架是 Lingmo Desktop 对第三方开发者暴露的核心界面层。必须保证第三方开发者
可以按需选择依赖，避免引入不需要的组件。

---

### lingmo-theme

**职责**：设计令牌系统（Design Token System）。定义桌面全局的视觉原子：
色板（主色/强调色/语义色）、字体（字体族/字号/字重/行高）、间距/边距、
圆角半径、阴影定义（模糊半径/偏移/颜色）、动效时长/缓动曲线、高度层级
（elevation/z-depth）。提供 C++ 类和 QML singleton 两种 API。

**为什么需要独立**：主题令牌是整个桌面视觉一致性的基石。所有组件（Widgets 风格、
QML 控件、窗口装饰器、GTK 主题、SDDM 主题）都依赖它。主题的 API 版本变更
会影响整个生态，需要独立发版和语义化版本控制。

**依赖**：
- 构建时：Qt 6 (Core, Gui, Qml, Quick)
- 运行时：Qt 6

**发布策略**：语义化版本。major 变更（如删除/重命名令牌）需协调所有消费者。

**为什么不合并到其他仓库**：
- 不合并到 `lingmo-style`：因为 QML 控件、窗口装饰器、GTK 主题也需要令牌，
  不能强制它们引入 QStyle 的依赖链
- 不合并到 `lingmo-controls`：Widgets 应用不需要 QML 依赖

---

### lingmo-style

**职责**：Qt Widgets 的 QProxyStyle 实现。从 `lingmo-theme` 读取令牌，
渲染所有标准 Qt 控件：按钮、输入框、滚动条、滑块、复选框、单选按钮、
标签页、菜单、工具提示、工具栏、进度条、分组框、微调框、组合框等。

**为什么需要独立**：
- 纯 Qt Widgets 应用只需要设置 `Lingmo::StyleManager::setStyle()` 即可
  获得 Lingmo 外观，无需引入任何 QML 依赖
- QStyle 的实现逻辑（绘制代码、像素度量、样式提示）变化频繁，
  但与令牌定义无关

**依赖**：
- 构建时：Qt 6 (Core, Gui, Widgets, Svg)，lingmo-theme
- 运行时：Qt 6，lingmo-theme

**发布策略**：补丁版本可频繁发布（修复特定控件的绘制问题）。

**为什么不合并到 lingmo-theme**：主题令牌 API 稳定（semver major 很少），
而 QStyle 实现细节迭代快。合并会导致主题消费者被迫接受频繁的 ABI 变更。

---

### lingmo-controls

**职责**：Qt Quick QML 控件模块。命名空间 `LingmoUI.Controls`。
提供：Button, TextField, TextArea, ComboBox, Switch, CheckBox, RadioButton,
TabBar, TabButton, Slider, ProgressBar, ScrollBar, ScrollView, SpinBox,
Dial, ToolTip, Menu, MenuItem, RangeSlider, ToolBar, PageIndicator 等。
所有控件从 `lingmo-theme` 读取设计令牌渲染。

**为什么需要独立**：
- QML 应用只需 `import LingmoUI.Controls`，无需引入 Qt Widgets 相关库
- QML 控件的开发节奏和 Widgets 不一致（QML 更依赖 Qt Quick 的版本演进）
- QML 控件的测试框架不同（qmllint + qmltest）

**依赖**：
- 构建时：Qt 6 (Core, Gui, Qml, Quick, QmlModels)，lingmo-theme
- 运行时：Qt 6，lingmo-theme

**发布策略**：跟随 Qt 6 minor 版本发布，利用 Qt 新特性时同步。

**为什么不合并到 lingmo-theme**：控件逻辑（状态管理、交互、焦点处理）
与令牌数据是不同层次的关注点，合并会模糊职责边界。

---

### lingmo-effects

**职责**：视觉效果库。提供平台无关的 API 来实现：实时模糊
（若 KWin 支持则使用 KDE 的 kwin blur，否则回退到软件模糊）、
阴影渲染（九宫格或 GPU blur-based）、高度层叠（elevation transition）、
背景效果（acrylic/mica-like 半透明）、涟漪效果、高亮效果。
C++ 库 + QML 导入 `LingmoUI.Effects`。

**为什么需要独立**：
- 视觉效果依赖平台特定的合成器能力（KWin 的 blur、custom effect hooks），
  实现逻辑复杂且与具体平台耦合
- 应用的视觉效果需求迭代快（新的过渡动画、新的背景材质），独立发版不会
  影响主题令牌或控件的稳定性
- 对平台能力的检测和回退逻辑需要独立测试

**依赖**：
- 构建时：Qt 6 (Core, Gui, Qml, Quick, Widgets)，lingmo-theme，
  KWin::libkwineffects（可选，启用硬件加速模糊时）
- 运行时：Qt 6，lingmo-theme

**发布策略**：可按需发版，新增效果类型为 minor，效果 bug 修复为 patch。

**为什么不合并到其他仓库**：既不是令牌定义（lingmo-theme），也不是控件
（lingmo-controls），也不是风格引擎（lingmo-style）。它是平台相关的视觉
渲染层，有自己独特的依赖链（KWin 效果头文件）。

---

### lingmo-icon

**职责**：图标主题引擎。加载 XDG 标准图标主题目录，实现图标查找和回退链
（`apps/` → `actions/` → `places/` → `devices/` 等）。支持 SVG 符号化
着色（根据当前主题色动态重着色），支持 HiDPI 图标选择，支持图标缓存
（QIconEngine 实现），支持从主题目录加载自定义覆盖图标。

**为什么需要独立**：
- 图标引擎不依赖于特定 UI 框架（同时服务于 Widgets 和 QML 应用）
- 图标查找逻辑（fallback chain、theme inheritance）本身是复杂的搜索算法，
  需要独立测试
- 第三方应用只需要图标查找、不需要 Lingmo 控件或主题令牌即可集成

**依赖**：
- 构建时：Qt 6 (Core, Gui, Svg, SvgWidgets)
- 运行时：Qt 6，XDG 数据目录中的图标主题

**发布策略**：独立于主题令牌。图标引擎很少需要 major 变更。

---

### lingmo-window

**职责**：窗口管理辅助库。提供：无边框窗口支持（隐藏系统标题栏）、
自定义标题栏组件（可嵌入搜索框/工具栏/标签页）、窗口阴影管理、
最小化/最大化/关闭按钮组（与 lingmo-decoration 中的 KWin 装饰器保持一致
样式）、窗口圆角裁剪。Qt Widgets 的 `Lingmo::Window`（QMainWindow 子类）、
QML 的 `LingmoUI.Window` 模块。

**为什么需要独立**：
- 窗口管理辅助是应用层与窗口管理器之间的桥梁，属于"应用开发工具"而
  非"UI 控件"。应用开发者只有在需要自定义窗口时才引入此库
- 窗口行为的变更（如圆角区域的支持、阴影渲染方式的平台差异）需要独立于
  控件库迭代

**依赖**：
- 构建时：Qt 6 (Core, Gui, Qml, Quick, Widgets, X11Extras)，lingmo-theme
- 运行时：Qt 6，lingmo-theme

**发布策略**：可独立发版。窗口管理相关的平台兼容性修复频繁。

**为什么不合并到 lingmo-effects**：两者都是视觉效果相关，但窗口管理涉及
平台 API（X11 extents、Wayland xdg-decoration 协议），而 effects 是
纯粹的图形渲染。不同的维护领域和测试策略。

---

## 2. 基础设施层 (4 个仓库)

基础设施层提供所有桌面组件和第三方应用所需的通用服务。每个库职责严格界定。

---

### lingmo-config

**职责**：配置系统。键值存储，支持 INI/JSON/Keyfile 格式。层级配置：系统级
（`/etc/lingmo/`）、用户级（`~/.config/lingmo/`）、会话级（临时覆盖）。
文件监听（inotify/ReadDirectoryChanges）实现实时配置同步。可选的 schema
验证机制（定义键的类型、默认值、取值范围）。分组配置（`[group]/key`）。
支持 QML `LingmoConfig` 绑定（配置值变化自动更新 UI）。

**为什么需要独立**：
- 配置系统是桌面环境的核心基础设施，几乎每个组件都使用
- API 需要保持长期稳定，语义化版本号独立演进
- 配置格式和存储后端（INI vs JSON vs DConf）的变更不影响其他组件

**依赖**：
- 构建时：Qt 6 (Core)
- 运行时：Qt 6

**发布策略**：semver。向后兼容的配置格式是承诺。major 版本仅当存储格式
发生不兼容变更时发布。

**为什么不合并到 lingmo-utils**：配置系统有自己的文件格式规范、schema
验证引擎、文件监听机制，是独立的子系统。合并到工具库会导致工具库的
依赖和版本管理复杂化。

---

### lingmo-plugin

**职责**：插件系统。JSON 元数据发现（在标准路径 `/usr/lib/lingmo/plugins/`
或 `~/.local/lib/lingmo/plugins/` 下扫描 `.lingmo-plugin` JSON 文件）、
共享库加载（QLibrary）、版本检查（插件声明的 API 版本与加载器预期版本匹配）、
依赖解析（插件 A 依赖插件 B）、延迟加载、服务类型注册
（applet/search-provider/settings-panel/extractor 等类型）。

**为什么需要独立**：
- 插件系统是桌面模块化和可扩展性的基础架构。API 必须稳定且版本化
- 插件的版本契约（API/ABI 兼容性声明）需要独立于任何特定插件类型管理
- 第三方开发者需要明确的插件开发指南和稳定的加载器

**依赖**：
- 构建时：Qt 6 (Core)
- 运行时：Qt 6

**发布策略**：版本号与插件 API 版本绑定。breaking change 时 major 版本递进，
同时要求所有插件生产者更新。

**为什么不合并到 lingmo-config**：配置系统读取的是静态数据，插件系统管理
的是可执行代码。安全审计要求不同（插件加载涉及沙箱、路径校验、签名验证），
分开更安全。

---

### lingmo-ipc

**职责**：进程间通信 + 服务生命周期管理。D-Bus 封装层：服务注册与发现、
信号发射、方法调用、属性绑定、introspection 辅助。systemd 集成：
sd-bus 接口（更快）、`sd_notify` 就绪通知（systemd watchdog 健康检查）、
服务启动/停止/重启辅助。

**为什么需要独立**：
- D-Bus 通信是所有桌面组件交换信息的基础。需要在系统级统一封装
- 服务生命周期管理（守护进程启动顺序、崩溃重启、健康检查）是独立于
  业务逻辑的基础设施

**依赖**：
- 构建时：Qt 6 (Core, DBus)，glib-2.0（用于 sd-bus 的 glib 主循环集成）
- 运行时：Qt 6，systemd（可选，用于 watchdog 和 socket 激活）

**发布策略**：minor 版本用于新增通信模式，patch 用于 bug 修复。

**为什么不合并到 lingmo-utils**：IPC 和服务管理是桌面环境的独有需求，
与通用文件/进程工具（lingmo-utils）是不同的领域。合并会让工具库产生
对 D-Bus 和 systemd 的不必要依赖。

---

### lingmo-utils

**职责**：通用工具库。日志（结构化日志、分级过滤、journald 集成、日志轮转）、
崩溃处理（信号处理器、backtrace 生成、coredump 触发、崩溃报告生成）、
文件操作（递归复制/移动/删除、进度回调、冲突处理）、MIME 类型检测
（shared-mime-info 封装）、进程管理（环境控制、输出捕获、退出码处理、
进程监视）、XDG 路径工具（BaseDir、autostart 解析）。

**为什么需要独立**：
- 纯工具函数集合，不依赖任何桌面特定服务
- 所有组件（UI、shell、daemon、apps）都需要但不属于任何一个业务领域
- 每个子模块都有独立的测试需求和迭代节奏

**依赖**：
- 构建时：Qt 6 (Core)，glib-2.0（用于 gschema 和 locale 检测），
  libmagic（MIME），libunwind（backtrace）
- 运行时：Qt 6

**发布策略**：频繁 patch 修复，minor 新增功能，major 极少。

**为什么不合并到其他基础设施库**：工具库应当零假设零耦合。如果合并到
lingmo-ipc，则工具库的消费者就不得不引入 D-Bus 依赖。工具库应当保持
最少的可选依赖。

---

## 3. 桌面外壳层 (8 个仓库)

外壳层由多个独立进程组成，通过进程间通信协作。这是现代桌面环境的最佳实践
（每个进程故障隔离，不会导致整个桌面崩溃）。

---

### lingmo-shell-core

**职责**：外壳核心进程。不渲染任何界面。职责：
- 状态管理（当前工作区、焦点窗口、活动应用）
- IPC 中心枢纽（panel/desktop/overview/notification/OSD 进程之间的通信网关）
- KWin 协调：通过 D-Bus 与 KWin 通信（窗口列表、工作区管理、窗口操作）
- Applet 加载协调：向 panel/desktop 进程分发 applet 实例
- KWin 脚本管理：加载/卸载 KWin JavaScript 行为脚本
- Wayland 协议支持：注册和实现 Lingmo 自定义 Wayland 协议

**为什么需要独立**：
- 核心进程是桌面外壳中最关键的部分，崩溃会导致整个桌面不可用。
  需要最小化其代码量以降低崩溃概率
- 其他外壳组建（panel/desktop/overview）可以独立重启而不影响核心状态
- 核心进程不需要图形渲染能力，可以去图形化运行

**依赖**：
- 构建时：Qt 6 (Core, DBus, Gui)，lingmo-theme，lingmo-config，
  lingmo-plugin，lingmo-ipc，lingmo-utils
- 运行时：Qt 6，lingmo-theme，lingmo-config，lingmo-plugin，lingmo-ipc，
  lingmo-utils，kwin (D-Bus API)

**发布策略**：patch 频繁（崩溃修复），minor 用于新增协议或 API。

**为什么不合并到其他外壳仓库**：
- 不合并到 lingmo-shell-surfaces：核心进程不是渲染进程，不含任何 QML/OpenGL。
  分离后 panel/desktop 的重启不影响核心状态
- 不合并到 lingmo-ipc：核心进程包含的是桌面状态逻辑（窗口管理、工作区），
  不是 IPC 基础设施

---

### lingmo-shell-surfaces

**职责**：表面进程（一个二进制，三种运行模式）。使用 wlr-layer-shell 协议
创建 Wayland 层表面。包含三个入口：

1. **Panel（面板）**：主面板，支持上下左右四个位置。内部通过 applet 接口
   加载和管理 applets（应用菜单、任务管理器、系统托盘、时钟、音量、电池等）。
   支持多面板（多显示器场景各面板独立配置）。

2. **Desktop（桌面）**：桌面表面。壁纸渲染（静态/幻灯片/动态/视差）、
   桌面图标（文件图标 + 应用图标）、桌面右键菜单（新建文件/更改壁纸/
   桌面设置）。

3. **Overview（概览）**：覆盖表面。窗口缩略图（通过 ext-foreign-toplevel 协议）、
   工作区切换器、应用网格、统一搜索入口。

**为什么需要独立**：
- 这三种表面共享 applet 加载框架、配置管理和 KWin 通信（通过 shell-core）。
  它们之间的差异主要是 QML 布局不同，而非架构差异
- 放在同一仓库便于共享模式代码和测试基础设施。拆分三个仓库会导致
  大量重复的 applet host 代码

**依赖**：
- 构建时：Qt 6 (Core, Gui, Qml, Quick, DBus, WaylandClient)，
  lingmo-theme，lingmo-controls，lingmo-effects，lingmo-icon，
  lingmo-window，lingmo-config，lingmo-plugin，lingmo-ipc，lingmo-utils
- 运行时：Qt 6，lingmo-theme，lingmo-controls，lingmo-effects，lingmo-icon，
  lingmo-window，lingmo-config，lingmo-plugin，lingmo-ipc，lingmo-utils

**发布策略**：一个版本号对应三个二进制。壁纸引擎新增格式或面板布局新增
位置选项为 minor。

**为什么不拆分**：三个表面使用同样的 applet 插槽系统、同样的配置库、
同样的 IPC 通道到 shell-core。拆分只会重复这些基础设施。职责单一体现在
"表面层的渲染和交互"这个层面，而不是"具体是哪种表面"。

---

### lingmo-applets

**职责**：Applet 插件框架 + 内置 applet 集合。提供：
- `Lingmo::Applet` 基类定义（初始化、配置界面、生命周期：activate/deactivate/
  suspend/resume）
- `Lingmo::AppletContext`：applet 运行环境（配置读写、shell-core IPC 访问、
  屏幕信息、多显示器信息）
- 内置 applets（作为子目录，每个可独立编译为插件 .so）：
  - 应用菜单（Application Menu）：XDG 应用目录、分类浏览、搜索、收藏/常用
  - 任务管理器（Task Manager）：窗口列表、图标模式/文字模式、分组、缩略图预览
  - 系统托盘（System Tray）：StatusNotifierItem host
  - 数字时钟（Digital Clock）：时间/日期显示、日历弹窗、时区切换
  - 电池（Battery）：百分比、图标、充电状态、时间预估
  - 音量（Volume）：音量滑块、输出设备切换、静音
  - 网络（Network Indicator）：信号图标、连接状态（权限交由 lingmo-network 管理）
  - 快速设置（Quick Settings）：可展开的面板区域，容纳 Wi-Fi、蓝牙、飞行模式、
    夜间模式、勿扰模式、性能模式的快捷开关
  - 托盘间距（Spacer）、分隔符（Separator）

**为什么需要独立**：
- Applet 插件 API 是面向第三方开发者的公共接口，必须拥有独立版本号和
  ABI 兼容性保证
- 内置 applets 的增删不影响外壳进程的版本号。新增一个 applet 类型不应当
  触发面板或桌面进程重新编译
- 第三方开发的 applet 只需要依赖 `lingmo-applets` 库，不应引入整个面板
  或桌面的依赖

**依赖**：
- 构建时：Qt 6 (Core, Gui, Qml, Quick, DBus)，lingmo-theme，lingmo-controls，
  lingmo-icon，lingmo-config，lingmo-ipc，lingmo-plugin，lingmo-utils
- 运行时：Qt 6，lingmo-theme，lingmo-controls，lingmo-icon，lingmo-config，
  lingmo-ipc，lingmo-plugin，lingmo-utils

**发布策略**：
- `liblingmo-applets`（applet 框架库）：semver，ABI 兼容是承诺
- 内置 applets 作为子包发布，可独立更新

**为什么不合并到 lingmo-shell-surfaces**：Applet API 需要独立版本化，
与面板/桌面/概览的发布节奏解耦。第三方 applet 开发者不应该因为面板进程
的版本升级而被强制重新编译 applet。

---

### lingmo-notification

**职责**：通知系统。实现 `org.freedesktop.Notifications` D-Bus 接口，
支持通知优先级（低/普通/紧急）、操作按钮、通知超时、通知替换与合并。
通知中心 UI：作为独立的 Wayland 层表面显示，展示通知历史、通知分组、
每应用通知权限管理、勿扰模式（支持定时启用）。通知历史持久化到
journald。

**为什么需要独立**：
- 通知服务是有明确定义的 D-Bus 标准接口的独立服务。它不属于外壳
  （不需要渲染时不占用资源），也不属于守护进程（它是用户界面服务）
- 通知中心可以独立于桌面外壳更新。第三方通知提供者也依赖稳定的接口

**依赖**：
- 构建时：Qt 6 (Core, Gui, Qml, Quick, DBus, WaylandClient)，
  lingmo-theme，lingmo-controls，lingmo-window，lingmo-config，
  lingmo-ipc，lingmo-utils
- 运行时：Qt 6，lingmo-theme，lingmo-controls，lingmo-window，
  lingmo-config，lingmo-ipc，lingmo-utils

**发布策略**：新增通知功能（如图形气泡样式、通知分类）为 minor，
兼容 `org.freedesktop.Notifications` 规范为 patch。

**为什么不合并到 lingmo-shell-core**：通知服务有自己的进程生命周期。
它不需要在桌面未运行时运行，但它是 D-Bus 激活的服务，不依赖于外壳进程
是否存活。

---

### lingmo-osd

**职责**：屏幕 OSD（On-Screen Display）进程。轻量级进程，
由 D-Bus 信号激活（`org.lingmo.OSD`）。显示短暂覆盖：
音量变化、亮度变化、媒体播放控制（上一首/播放/暂停/下一首）、
飞行模式切换、键盘布局切换、勿扰模式开关、性能模式切换。
每条 OSD 自动超时消失（1.5 秒）。使用 wlr-layer-shell 协议
在最顶层渲染。

**为什么需要独立**：
- OSD 进程生命周期极短（创建后显示 1.5 秒即退出）。它不是常驻进程
- D-Bus 激活的进程按需启动，零资源占用
- OSD 的 UI 极其简单（一个弹出条），不需要复杂的 applet 框架或
  通知中心的依赖

**依赖**：
- 构建时：Qt 6 (Core, Gui, Qml, Quick, DBus, WaylandClient)，
  lingmo-theme，lingmo-controls，lingmo-ipc
- 运行时：Qt 6，lingmo-theme，lingmo-controls，lingmo-ipc

**发布策略**：随整体桌面发版即可，极少需要独立发版。但保持独立的理由
是职责纯粹和进程边界清晰。

**为什么不合并到 lingmo-notification**：通知和 OSD 是两种不同的交互模式。
通知是持久性的、有历史的、用户可主动查看的。OSD 是短暂性的、瞬时的、
用户无需交互的。合并后代码可以共享（都是 D-Bus 触发的 UI），但职责
和测试向量完全不同。

---

### lingmo-runner

**职责**：Alt+F2 运行命令/快速搜索对话框。D-Bus 激活的进程，按需启动。
支持插件化的结果提供者：命令执行（Shell 命令）、计算器（表达式求值）、
文件搜索（通过 lingmo-search 服务）、应用启动（XDG desktop files）、
网页搜索（调用默认浏览器）、书签、终端命令。结果显示为列表，
可键盘导航。选中项执行对应操作。

**为什么需要独立**：
- 与 OSD 类似，按需启动的进程，不常驻
- 搜索提供者的插件架构（通过 lingmo-plugin）需要独立的插件发现路径
  和版本管理
- 结果提供者的开发和测试独立于面板开发

**依赖**：
- 构建时：Qt 6 (Core, Gui, Qml, Quick, DBus, WaylandClient)，
  lingmo-theme，lingmo-controls，lingmo-icon，lingmo-config，
  lingmo-plugin，lingmo-ipc，lingmo-utils
- 运行时：Qt 6，lingmo-theme，lingmo-controls，lingmo-icon，lingmo-config，
  lingmo-plugin，lingmo-ipc，lingmo-utils

**发布策略**：新增结果提供者类型为 minor，搜索算法改进为 patch。

**为什么不合并到 lingmo-shell-surfaces**：Runner 是独立的应用程序，
不是外壳表面。它不共享 applet 框架，不参与面板/桌面布局。
合并会引入不必要的 applet 依赖。

---

### lingmo-decoration

**职责**：KWin 窗口装饰 C++ 插件。使用 KWin Decoration API
（`KDecoration2::Decoration`）实现 Lingmo 风格的窗口装饰：
标题栏（是否显示、可选的 CSD 支持）、窗口按钮（关闭/最小化/最大化/全屏/
窗口菜单）、窗口边框（宽度、颜色、圆角半径与 lingmo-theme 一致）、
窗口阴影（使用 KWin 内置阴影引擎，大小与 blur 匹配 lingmo-theme 的 shadow
令牌）、双击标题栏最大化、右键标题栏窗口菜单。

**为什么需要独立**：
- KWin 装饰插件是 KWin 生态中的 C++ 插件，有特定的 ABI 要求
  （KDecoration2 API 版本）。它不依赖于外壳的任何部分
- 装饰插件的版本必须与 KWin API 的版本匹配，有独立的兼容性约束
- 窗口装饰样式非常容易被用户定制，独立的仓库便于社区贡献新的
  装饰主题

**依赖**：
- 构建时：Qt 6 (Core, Gui, Widgets)，lingmo-theme，KWin::libkwindecoration
- 运行时：Qt 6，lingmo-theme，kwin (加载插件)

**发布策略**：只有当 KWin 的 Decoration API 变更或 Lingmo 主题令牌
视觉更新时才发版。

**为什么不合并到 lingmo-shell-core**：装饰插件不运行在外壳进程中，
它运行在 KWin 进程中。不同的进程边界、不同的 API 契约、不同的安全上下文。
合并到外壳仓库会导致错误的架构分层。

---

### lingmo-shell-scripts

**职责**：KWin JavaScript 行为脚本集合。通过 KWin 脚本系统加载，
实现桌面行为逻辑：
- 窗口放置策略（智能放置/居中/最大化/手动指定）
- 焦点策略（点击聚焦/跟随鼠标/严格跟随鼠标）
- 窗口捕捉区域（触摸屏边缘/快捷键激活的平铺布局：1/2 屏幕、1/3 屏幕、
  网格布局）
- 工作区切换动画（滑动/淡入淡出/3D 立方体）
- 应用启动反馈（任务栏闪烁/启动动画）
- 窗口透明度/阴影规则（如对特定应用禁用阴影）
- 工作区管理（快捷键切换、鼠标滚轮切换）

**为什么需要独立**：
- KWin 脚本是 JavaScript 纯数据文件，没有编译步骤。独立仓库意味着
  可以绕过整个 C++ CI 流程直接发布行为更新
- 行为变更非常频繁（用户反馈窗口布局不好用，快速调整放置策略），
  需要与 C++ 仓库不同的发布节奏
- 社区贡献者可以 fork 并修改行为而不需要理解 C++ 编译

**依赖**：
- 构建时：无
- 运行时：kwin (加载 JS 脚本)

**发布策略**：可以独立于所有 C++ 仓库高频发布。甚至可以通过脚本更新
频道每日推送新行为。

**为什么不合并到 lingmo-shell-core**：脚本的迭代周期是小时到天，
C++ 代码的迭代周期是天到周。不同的开发流程、不同的测试要求、不同的
发布管道。同一个仓库中混入 C++ 和 JS 会让开发体验变差（CMake 和脚本格式
混在一起）。

---

## 4. 会话管理层 (2 个仓库)

---

### lingmo-session

**职责**：会话管理器。作为系统d user service 启动。职责：
- 启动序列编排：按依赖顺序启动桌面组件（lingmo-daemon 系列 →
  lingmo-shell-core → lingmo-shell-surfaces → lingmo-notification）
- 崩溃检测与恢复：如果外壳崩溃，自动重启；如果核心守护进程崩溃，
  弹出恢复对话框
- 关闭/重启/注销处理：按依赖逆序优雅终止组件
- 自动启动管理：实现 XDG Autostart 规范（`~/.config/autostart/`）
  和 Lingmo 自动启动目录
- 会话环境设置：XDG 基本目录、Lingmo 环境变量（`LINGMO_*`）、
  locale/语言、键盘布局设置
- 首次运行检测：检测是否为首次登录，若为首登录则启动 lingmo-first-run
- 会话状态保存/恢复（简化版：保存打开的应用列表，恢复时重新启动它们）

**为什么需要独立**：
- 会话管理器是系统d 管理的用户服务，在其他桌面组件启动之前运行。
  它需要最小化依赖以降低启动失败概率
- 所有组件都可能崩溃，但会话管理器必须存活。它不能依赖可能崩溃的
  任何 UI 组件

**依赖**：
- 构建时：Qt 6 (Core, DBus)，lingmo-ipc，lingmo-utils，lingmo-config
- 运行时：Qt 6，lingmo-ipc，lingmo-utils，lingmo-config，systemd

**发布策略**：patch 频繁（启动顺序调整、崩溃恢复逻辑改进）。

**为什么不合并到 lingmo-shell-core**：会话管理器是外壳核心进程的管理者。
如果它们在同一进程，会话管理器崩溃就无法管理外壳核心了。不同的进程边界
是实现崩溃自愈的先决条件。

---

### lingmo-lockscreen

**职责**：屏幕锁定器。实现 `ext-session-lock` Wayland 协议（KWin 已支持）。
作为单独进程运行：
- PAM 身份验证（`system-auth` 栈）
- 锁屏 UI：时钟（大字体显示时间/日期）、用户头像和用户名、密码输入框、
  解锁按钮、紧急操作（切换用户、关机/重启）、媒体控制（当前播放曲目信息
  和控制按钮）
- 安全加固：seccomp 系统调用过滤、当锁定失效时自动锁定、
  无敏感信息泄漏
- 自动锁定：闲置超时后自动锁定（超时时间来自电源管理配置）
- 锁定动画：壁纸虚化/变暗过渡

**为什么需要独立**：
- 锁屏是安全关键组件。独立的进程意味着即使外壳被攻破，锁屏仍然安全
- 锁屏需要最少的依赖。它不访问文件系统、不加载用户配置、不运行插件
- 独立的代码库便于安全审计和形式化验证

**依赖**：
- 构建时：Qt 6 (Core, Gui, Qml, Quick, DBus, WaylandClient)，
  lingmo-theme，lingmo-controls，lingmo-effects，lingmo-utils，
  pam (libpam)，libseccomp
- 运行时：Qt 6，lingmo-theme，lingmo-controls，lingmo-effects，
  lingmo-utils

**发布策略**：安全修复（patch）优先于功能新增。任何安全相关变更都需要
独立审计和签名。

**为什么不合并到 lingmo-session**：锁屏有自己的进程生命周期（在锁定时
由 KWin 触发），不由会话管理器创建。不同的入口点、不同的执行上下文、
不同的安全策略。

---

## 5. 系统服务层 (7 个仓库)

每个服务守护进程作为一个独立的 systemd user service 运行。拥有独立的
D-Bus 接口、独立的构建依赖、独立的测试套件、独立的发布周期。

---

### lingmo-settings-daemon

**职责**：核心设置守护进程。监控全局配置变更并在系统中应用：
- 键盘布局变更 → 调用 `setxkbmap` / Wayland 键盘布局 API
- 字体配置变更 → 更新 fontconfig 配置，通知应用
- GTK 主题同步 → 通过 gsettings 桥将 Lingmo 主题令牌映射到 GTK 主题
- XSettings 管理 → 更新 XSettings（用于 XWayland 应用的 GTK/Qt 主题同步）
- 界面缩放变更 → 设置 `QT_WAYLAND_FORCE_FRACTIONAL_HI`DPI`_SCALE` 等环境变量
- 环境文件更新 → 写入 `~/.config/environment.d/`（systemd 用户会话环境）
- 可移动设备管理：监控 udisks2 事件 → 自动挂载通知 → 桌面上显示设备图标 →
  安全弹出

**为什么需要独立**：设置守护进程是第一个启动的守护进程，需要尽快就绪。
它在后台持续运行，监控配置变化。它的逻辑（将配置值转换为系统操作）是
独特且独立于其他守护进程的。

**依赖**：
- 构建时：Qt 6 (Core, DBus)，lingmo-config，lingmo-utils，glib-2.0
  （用于 gsettings），udisks2 (libudisks2 或 D-Bus)
- 运行时：Qt 6，lingmo-config，lingmo-utils

**发布策略**：新增对某个设置的响应操作为 minor，现有响应操作的 bug 修复
为 patch。

**为什么不合并到其他守护进程**：设置守护进程处理的是"配置变更"事件。
其他守护进程（如电源管理、显示管理）处理的是"硬件事件"。不同的触发源、
不同的应用模式、不同的测试策略。

---

### lingmo-power-manager

**职责**：电源管理。UPower 客户端（通过 D-Bus 或 libupower-glib）：
电池状态监控（百分比、充放电、健康度、估计剩余时间）、功耗曲线切换
（性能/均衡/节能）、合盖操作（挂起/休眠/关闭屏幕/无操作）、
挂起/休眠触发（基于空闲、基于按键、基于合盖）、低电量警告（渐进式：
第一次 20%、第二次 10%、临界 5%）、自适应亮度（可选，通过传感器读取
/usr/lib/sensorfw 或 iio-sensor-proxy）、DPMS 控制（显示器超时关闭）。

**为什么需要独立**：电源管理涉及硬件交互（ACPI、传感器），需要持续运行。
如果崩溃，需要自动重启（systemd Restart=always）。独立的守护进程保证
电源管理不因其他守护进程的问题而中断。

**依赖**：
- 构建时：Qt 6 (Core, DBus)，lingmo-config，lingmo-ipc，lingmo-utils，
  UPower (libupower-glib 或 D-Bus 接口)，systemd (sd-bus 用于 inhibit)
- 运行时：Qt 6，lingmo-config，lingmo-ipc，lingmo-utils，upowerd

**发布策略**：电源管理策略调整（如新的省电策略）为 minor。

---

### lingmo-display

**职责**：显示管理。KScreen 集成（libkscreen）：显示器热插拔检测（udev）、
分辨率/刷新率/缩放/排列（拖拽布局）、显示配置保存和恢复（每显示器/每
连接类型记忆）、夜间模式（定时或手动启用的蓝光过滤，集成了类似于
redshift 的色温调整）、ICC 色彩管理配置文件加载（colord）、
Wayland 输出配置（`zwlr_output_configuration` 协议封装）。

**为什么需要独立**：显示管理有大量平台相关代码（KScreen/XRandR/Wayland
输出协议），需要针对每种配置场景进行测试。显示错误（如分辨率设置不当）
可能导致用户无法操作，需要独立的错误恢复逻辑。

**依赖**：
- 构建时：Qt 6 (Core, DBus, Gui)，lingmo-config，lingmo-ipc，
  lingmo-utils，libkscreen，colord (libcolord)，lcms2
- 运行时：Qt 6，lingmo-config，lingmo-ipc，lingmo-utils

**发布策略**：新增显示器特性（如可变刷新率适配）为 minor。

---

### lingmo-input

**职责**：输入设备管理。支持：键盘（重复率/延时、布局切换、自定义组合键、
LED 指示）、鼠标（速度/加速度、自然滚动、按钮映射、滚轮速度）、
触摸板（点击触击、多指手势、边沿滚动、禁用输入即打字）、
绘图板（压力感应按钮自定义、映射区域）、触摸屏校准（通过 libinput 或
直接触控屏协议）。媒体键处理：监听键盘媒体键（XF86Audio*、XF86MonBrightness*
等）→ 发送 MPRIS D-Bus 信号给媒体播放器 → 触发 OSD 显示。

**为什么需要独立**：输入设备是用户直接交互的入口。输入设置错误会立即
影响用户体验。不同的输入设备类型有不同的测试需求（触摸板手势需要硬件
或无头测试环境）。

**依赖**：
- 构建时：Qt 6 (Core, DBus)，lingmo-config，lingmo-ipc，lingmo-utils，
  libinput (插件化后端)，systemd (logind 用于座位和会话)
- 运行时：Qt 6，lingmo-config，lingmo-ipc，lingmo-utils

**发布策略**：新增输入设备支持（如游戏手柄映射）为 minor。

---

### lingmo-audio

**职责**：音频管理。PipeWire 客户端（通过 libpipewire 或 pipewire-qt）：
音频设备枚举（声卡、HDMI、蓝牙耳机、USB 音频）、默认音源/输出切换、
音量控制（百分比 + 分贝）、静音状态管理、音频配置文件选择（模拟/数字/
HDMI/环绕声）、应用音频流路由（每个应用独立音量 + 输出设备）、
空间音频支持（Dolby Atmos / DTS:X）、蓝牙音频编解码器选择（SBC/AAC/aptX/LDAC）。

**为什么需要独立**：音频服务是多媒体体验的核心。PipeWire 的图形化拓扑
管理和流路由逻辑复杂，需要专注的开发和测试。音频问题（无声音、声道错乱）
是用户最敏感的问题之一。

**依赖**：
- 构建时：Qt 6 (Core, DBus)，lingmo-config，lingmo-ipc，lingmo-utils，
  libpipewire，spa (PipeWire SPA 库)
- 运行时：Qt 6，lingmo-config，lingmo-ipc，lingmo-utils，pipewire，
  wireplumber

**发布策略**：新增音频配置文件支持（如新的 Surround Sound 格式）
为 minor。

**为什么不把媒体键合并到这里**：媒体键处理的是键盘输入（输入设备范畴），
虽然它触发的是媒体播放指令（与音频相关），但它的数据流是：键盘硬件 →
内核输入子系统 → libinput → D-Bus（MPRIS）。它不涉及 PipeWire 或音频
设备管理。放在 lingmo-input 更准确。

---

### lingmo-thumbnails

**职责**：缩略图生成服务。D-Bus 激活的服务，按需启动和退出。
并行生成缩略图（QThreadPool 或工作进程池）。缓存管理（遵循 Freedesktop
Thumbnail Managing Standard：`~/.cache/thumbnails/` 下的 normal/large/x-large/
xx-large 目录，失败缓存标记以减少重试）。可插拔的内容提取器：
图像（JPEG/PNG/WebP/GIF/BMP/HEIF/RAW 通过 libraw）、视频（FFmpeg）、
PDF（Poppler）、Office 文档（LibreOffice 或直接解析 ODF/DOCX）、
音频专辑封面（taglib）、字体预览。

**为什么需要独立**：缩略图生成是 CPU 密集型操作，需要后台处理。
从文件管理器和图像查看器等调用时，不需要阻塞或影响调用者的性能。
作为独立服务可以根据负载调整并行度。

**依赖**：
- 构建时：Qt 6 (Core, DBus, Gui, Concurrent)，lingmo-plugin（提取器插件），
  lingmo-utils，libpng，libjpeg-turbo，libwebp，FFmpeg (libavcodec/libavformat)，
  poppler (可选)，taglib (可选)
- 运行时：Qt 6，lingmo-plugin，lingmo-utils

**发布策略**：新增提取器格式为 minor，提取算法优化为 patch。

**为什么不合并到 lingmo-settings-daemon**：缩略图服务是 CPU 密集型工作，
不适合持续运行的核心守护进程（可能影响系统响应性）。两者有不同的资源
配置需求（core 设置 I/O priority low、CPU nice high）。

---

### lingmo-print

**职责**：打印管理。CUPS 集成：自动打印机发现（IPP Everywhere、Bonjour）、
打印机配置（选项：纸张尺寸、双面、彩色/黑白、质量）、打印队列监控
（作业状态、取消、暂停）、默认打印机管理、驱动安装辅助（搜索推荐驱动）、
PDF 打印输出（打印到文件）。提供 D-Bus 接口供控制中心和文件管理器调用。

**为什么需要独立**：打印管理依赖 CUPS，而 CUPS 是一个具有自己复杂状态
模型和事件系统的外部系统。打印服务的崩溃不应影响桌面核心功能。

**依赖**：
- 构建时：Qt 6 (Core, DBus)，lingmo-ipc，lingmo-utils，CUPS (libcups)
- 运行时：Qt 6，lingmo-ipc，lingmo-utils，cupsd

**发布策略**：新增打印协议支持（如 IPP 3.0）为 minor。

**为什么不合并到文件管理器**：打印不仅从文件管理器触发，也从文本编辑器、
文档查看器、图像查看器等应用触发。集中化的打印服务保证所有应用使用
同样的打印配置和体验。

---

## 6. 网络管理层 (2 个仓库)

---

### lingmo-network

**职责**：网络管理。NetworkManager 前端：
- 面板 applet：Wi-Fi 信号强度指示、网络列表/扫描/连接/断开、以太网状态、
  VPN 管理（OpenVPN/WireGuard 配置文件导入、启停状态、自动连接）、
  飞行模式开关、网络代理（手动/自动/PAC）
- 后台 D-Bus 服务：通过 NetworkManager D-Bus API 监控连接状态。
  提供简化的接口供控制中心网络面板和面板 applet 使用
- 连接编辑器：Wi-Fi（SSID/安全类型/密码/自动连接/优先级）、
  有线（IPv4/IPv6/DNS/DHCP/802.1X）、VPN（OpenVPN/WireGuard/L2TP/PPTP 配置）
- 热点功能：创建 Wi-Fi 热点并与文件管理器/远程桌面集成

**为什么需要独立**：
- 网络管理是整个桌面中最复杂且与外部服务（NetworkManager）耦合最深的组件。
  独立的仓库允许专注于网络相关的开发和测试
- NetworkManager 的接口变更（D-Bus API 版本更新）需要网络组件快速响应，
  不应阻塞桌面其他部分的版本发布

**依赖**：
- 构建时：Qt 6 (Core, Gui, Qml, Quick, DBus, Network)，lingmo-theme，
  lingmo-controls，lingmo-icon，lingmo-config，lingmo-ipc，lingmo-utils，
  NetworkManager (libnm)
- 运行时：Qt 6，lingmo-theme，lingmo-controls，lingmo-icon，lingmo-config，
  lingmo-ipc，lingmo-utils，NetworkManager

**发布策略**：新增连接类型支持（如新的 VPN 协议）为 minor。

---

### lingmo-bluetooth

**职责**：蓝牙管理。BlueZ 客户端：
- 面板 applet：蓝牙开关、设备列表（已配对/可发现）、连接/断开、
  设备电池电量显示、文件传输（发送/接收进度）
- 配对代理：D-Bus 配对代理实现（配对确认弹窗、PIN 码输入/显示、
  一致配对确认）
- 文件传输：通过 OBEX (OBEX Push 和 FTP) 发送和接收文件，
  接收文件存储到下载目录
- 设备浏览：浏览已连接蓝牙设备上的文件（FTP 配置文件）
- 蓝牙 Tethering：与网络优先级集成（蓝牙网络共享）

**为什么需要独立**：
- 蓝牙的 D-Bus 接口（org.bluez）非常复杂，高频变更。独立的仓库允许
  专注的维护和测试
- 蓝牙硬件状态不稳定（适配器丢失、设备断开），需要健壮的连接重试逻辑
  和错误处理，独立于其他网络功能

**依赖**：
- 构建时：Qt 6 (Core, Gui, Qml, Quick, DBus)，lingmo-theme，lingmo-controls，
  lingmo-icon，lingmo-config，lingmo-ipc，lingmo-utils，bluez (libbluetooth
  或 D-Bus 接口)
- 运行时：Qt 6，lingmo-theme，lingmo-controls，lingmo-icon，lingmo-config，
  lingmo-ipc，lingmo-utils，bluetoothd

**发布策略**：新增配置文件支持（如 LE Audio、LC3 编解码器）为 minor。

**为什么不合并到 lingmo-network**：网络和蓝牙使用完全不同的底层协议栈
（NetworkManager vs BlueZ），不同的硬件管理系统，不同的测试要求。
合并会引入蓝牙不需要的 NetworkManager 依赖，反之亦然。

---

## 7. 搜索层 (1 个仓库)

---

### lingmo-search

**职责**：桌面搜索。包含：
- **索引器守护进程**（lingmo-indexer）：文件系统监视（inotify/fanotify），
  增量索引（监听变更事件比全量扫描更高效），内容提取（通过可插拔提取器），
  低优先级 I/O（ionice + nice），索引暂停/恢复（当系统忙碌/电池供电时）
- **搜索引擎库**：Xapian 封装（创建/更新索引，BM25 排名，查询解析，
  模糊匹配，分面过滤（文件类型/日期范围/文件大小））
- **内容提取器插件**：通过 lingmo-plugin 加载。纯文本、PDF（Poppler）、
  Microsoft Office（libreofficekit 或内部解析）、OpenDocument、
  HTML（Qt WebEngine 或 libxml2）、图片元数据（exiv2）、音频元数据（taglib）、
  视频元数据（FFmpeg）、存档（libarchive，索引文件列表）
- **搜索提供者插件**：文件（来自索引）、应用（XDG desktop 文件）、
  设置（控制中心面板标签）、计算器（表达式求值）、终端命令（$PATH 查找）、
  网页搜索（默认浏览器的搜索引擎）

**为什么需要独立**：
- 搜索系统包含一个守护进程（索引器）和一个库（搜索引擎），两者紧密耦合。
  拆分会导致版本同步问题
- Xapian 索引格式向前兼容但有时需要升级。索引版本管理需要搜索组件
  整体考虑
- 内容提取器依赖大量第三方库（poppler、libreoffice、ffmpeg、exiv2、taglib），
  这些依赖与桌面的其他部分无关

**依赖**：
- 构建时：Qt 6 (Core, DBus, Concurrent)，lingmo-plugin，lingmo-config，
  lingmo-ipc，lingmo-utils，xapian-core，exiv2，taglib，ffmpeg (libavformat)，
  poppler (可选)
- 运行时：Qt 6，lingmo-plugin，lingmo-config，lingmo-ipc，lingmo-utils，
  xapian-core

**发布策略**：索引模式变更为 minor，提取器插件新增格式为 patch。

**为什么不拆分**：搜索组件虽然包含多个子系统（索引器、引擎、提取器、提供者），
但它们共享同一份索引文件、同一份 Xapian 数据库、同一套配置。拆分会导致
索引锁定冲突、版本不匹配、API/ABI 对齐问题。完整性校验需要在统一版本下
保证。

---

## 8. 控制中心 (1 个仓库)

---

### lingmo-control-center

**职责**：设置中心应用。所有的设置面板作为内置模块（不是外部插件）：
外观、显示、电源、输入设备、声音、网络、蓝牙、打印机、用户账户、时间/日期、
语言和区域、默认应用、在线账户、辅助功能、通知、搜索、隐私、更新、关于、
可移动设备、快捷键、防火墙、会话、桌面效果（22 个面板）。

为什么面板不拆分成外部插件：
- 每个面板共享控制中心的导航框架（侧边栏/搜索栏/面包屑导航）
- 每个面板共享相同的设置变更追踪模式（统一 apply/reset 机制）
- 面板之间可能有交叉引用（如网络面板引用防火墙面板的规则）

**为什么需要独立**：
- 设置应用是桌面中使用频率最高的应用之一，有自己独立的 UI/UX 设计
- 添加或修改一个设置面板不影响其他桌面组件
- 设置应用的国际化（翻译）独立于其他组件

**依赖**：
- 构建时：Qt 6 (Core, Gui, Qml, Quick, Widgets, DBus)，lingmo-theme，
  lingmo-style，lingmo-controls，lingmo-icon，lingmo-window，lingmo-effects，
  lingmo-config，lingmo-plugin，lingmo-ipc，lingmo-utils，libkscreen (显示面板)，
  NetworkManager (libnm，网络面板)，bluez (蓝牙面板)，cups (打印面板)，
  PackageKit-Qt6 (更新面板)，polkit-qt-6 (用户面板)
- 运行时：Qt 6，lingmo-theme，lingmo-style，lingmo-controls，lingmo-icon，
  lingmo-window，lingmo-effects，lingmo-config，lingmo-plugin，lingmo-ipc，
  lingmo-utils

**发布策略**：新增设置面板为 minor，任一面板的 bug 修复为 patch。
受益于单一仓库，用户只需要更新一个包即可获得所有设置改进。

**为什么不拆分成多个面板仓库**：22 个面板是同一应用的不同视图。它们共享
导航框架、搜索索引、设置读写接口、变更追踪机制。拆分到独立的仓库会使
每次"添加一个新设置项"需要协调多个仓库的版本发布。但构建时使用 CMake
选项控制各面板的条件编译（无 NetworkManager 时不编译网络面板）。

---

## 9. 集成层 (3 个仓库)

---

### lingmo-portals

**职责**：xdg-desktop-portal 后端。实现以下 portal 接口：
- FileChooser：打开/保存文件对话框（使用 LingmoUI 风格的文件选择器）
- Screenshot：截图请求（调用 lingmo-capture 或内部实现）
- RemoteDesktop：远程桌面请求（通过 PipeWire + KWin screencast）
- ScreenCast：屏幕录制请求（通过 PipeWire）
- Wallpaper：壁纸设置请求
- Notification：发送通知请求（桥接到 lingmo-notification）
- OpenURI：打开 URI 请求（使用默认应用）
- Print：打印对话框请求（桥接到 lingmo-print）
- Camera：摄像头访问请求（通过 PipeWire 或 v4l2）
- Location：位置信息请求（通过 GeoClue）
- Account：用户账户信息请求
- Email：发送邮件请求
- Trash：移动到回收站请求
- Background：后台运行权限请求（Flatpak 应用）
- Secrets：密钥存储服务（libsecret 兼容接口）

**为什么需要独立**：
- Portal 后端是桌面环境与沙箱应用（Flatpak）之间的桥梁。它是独立于
  桌面外壳的系统服务（由 xdg-desktop-portal D-Bus 激活）
- Portal 接口规范由 freedesktop.org 定义，随规范更新需要快速响应
- Portal 代码逻辑是与安全相关的（权限决策），需要独立的安全审计

**依赖**：
- 构建时：Qt 6 (Core, Gui, Qml, Quick, Widgets, DBus)，lingmo-theme，
  lingmo-controls，lingmo-window，lingmo-config，lingmo-ipc，lingmo-utils，
  xdg-desktop-portal (libportal 或 D-Bus 接口)
- 运行时：Qt 6，lingmo-theme，lingmo-controls，lingmo-window，lingmo-config，
  lingmo-ipc，lingmo-utils，xdg-desktop-portal

**发布策略**：新增 portal 接口实现为 minor，安全修复为 patch。

---

### lingmo-polkit

**职责**：PolicyKit 认证代理。D-Bus 监听 `org.freedesktop.PolicyKit1.AuthenticationAgent`
请求。弹出认证对话框（LingmoUI 风格，显示请求应用名称和图标、
权限描述、密码输入框）。凭证缓存（一段时间内重复验证免输入）。
超时处理（超时未响应自动拒绝）。

**为什么需要独立**：
- PolicyKit 代理是安全敏感组件，与系统权限管理直接相关
- 有严格的可靠性和可用性要求（不能崩溃，崩溃后 systemd 必须能重启）
- 独立的代码库便于安全审计和签名检测

**依赖**：
- 构建时：Qt 6 (Core, Gui, Qml, Quick, DBus)，lingmo-theme，lingmo-controls，
  lingmo-window，lingmo-ipc，lingmo-utils，polkit-qt-6
- 运行时：Qt 6，lingmo-theme，lingmo-controls，lingmo-window，lingmo-ipc，
  lingmo-utils，polkitd

**发布策略**：安全修复优先。认证弹窗 UI 改进为 minor。

**为什么不合并到 lingmo-portals**：Portals 管理的是沙箱权限（应用访问文件/
摄像头/位置），PolKit 管理的是系统级权限（安装软件/修改系统设置）。
不同的权限模型、不同的接口规范、不同的安全审计维度。

---

### lingmo-flatpak

**职责**：Flatpak 集成。提供：
- Flatpak 应用安装/卸载/更新管理（通过 Flatpak 命令行或 D-Bus API）
- Flatpak 应用权限浏览器/编辑器（显示每个应用的 sandbox 权限，
  允许用户修改：文件系统访问、网络、设备、系统总线、会话总线）
- Portal 后端正确性验证（在 Flatpak 环境中验证 portal 是否正常工作）
- Flatpak 运行时管理（列出/更新运行时）
- Flatpak 源管理（添加/删除 Flathub 等远程源）

**为什么需要独立**：
- Flatpak 有自己独立的依赖链（flatpak 库、bubblewrap、OSTree），
  与桌面其他组件无共享依赖
- Flatpak 的版本与功能演进独立于桌面开发周期
- 有些用户可能不使用 Flatpak，独立的包允许按需安装

**依赖**：
- 构建时：Qt 6 (Core, DBus)，lingmo-ipc，lingmo-utils，flatpak (libflatpak)，
  ostree (libostree)
- 运行时：Qt 6，lingmo-ipc，lingmo-utils，flatpakd (或 flatpak 命令行)

**发布策略**：Flatpak 规范更新时同步发版。

**为什么不合并到 lingmo-portals**：Portal 后端是被动响应式的（被调用时激活）
，Flatpak 集成是主动管理式的（用户主动管理应用）。两者的代码结构和
生命周期不同。

---

## 10. 应用层 (12 个仓库)

每个核心应用作为一个独立仓库。原因是：
- 每个应用有**独立的依赖链**（文件管理器需要 libmagic/exiv2/taglib，
  文档查看器需要 poppler/libdjvulibre）
- 每个应用有**独立的用户群体**（用户可能只安装文件管理器而不安装计算器）
- 每个应用可以**独立发布版本**（计算器的 bug 修复不应阻塞终端模拟器的发布）
- 每个应用有**独立的测试策略**（终端需要 pseudo-terminal 测试，图片查看器
  需要图像比较测试）
- 每个应用可以由**不同的维护者**维护

---

### lingmo-apps-common

**职责**：应用共享库。提供所有核心应用通用的组件：
- 关于对话框（统一的"关于 Lingmo"样式）
- 设置对话框（统一的偏好设置窗口布局）
- 文件对话框（统一的文件打开/保存，集成 LingmoUI 主题）
- 状态栏组件（进度信息、状态提示）
- 搜索栏组件（统一的搜索输入样式和行为）
- 视图切换器（图标/列表/网格视图切换）
- 翻译工具函数（复数处理、上下文翻译）

**为什么需要独立**：共享库必须有自己的版本号，保证 ABI 兼容性。
如果所有应用从各自仓库拷贝共享代码，将面临维护灾难。

**依赖**：
- 构建时：Qt 6 (Core, Gui, Qml, Quick, Widgets)，lingmo-theme，lingmo-style，
  lingmo-controls，lingmo-icon，lingmo-window，lingmo-config，lingmo-utils
- 运行时：Qt 6，lingmo-theme，lingmo-style，lingmo-controls，lingmo-icon，
  lingmo-window，lingmo-config，lingmo-utils

**发布策略**：新增共享组件为 minor。

---

### lingmo-file-manager

**职责**：文件管理器。视图模式：图标、列表、详细列表、列视图。分栏模式
（左右双栏，独立目录）。标签页。侧边栏（树状目录、书签、快速访问：桌面/
下载/文档/图片/音乐/视频、网络位置、回收站）。搜索（递归目录搜索、
文件名过滤、内容搜索桥接 lingmo-search、搜索历史）。文件操作
（复制/移动/删除/重命名，带进度对话框、冲突解决、错误处理）。属性对话框
（基本信息、权限、所有者、封套程序标签）。书签管理。网络文件系统
（SFTP/WebDAV/SMB/CIFS 通过 GVfs）。回收站管理（Freedesktop Trash 规范、
还原/清空）。批量重命名（规则模式如编号/日期/替换）。文件冲突解决
（保留两者/替换/跳过/重命名）。批量解压与压缩集成。

**依赖**：
- 构建时：Qt 6 (Core, Gui, Qml, Quick, Widgets, DBus)，lingmo-theme，
  lingmo-style，lingmo-controls，lingmo-icon，lingmo-window，lingmo-apps-common，
  lingmo-config，lingmo-ipc，lingmo-utils，libmagic，exiv2，taglib，gvfs (可选)
- 运行时：Qt 6，lingmo-theme，lingmo-style，lingmo-controls，lingmo-icon，
  lingmo-window，lingmo-apps-common，lingmo-config，lingmo-ipc，lingmo-utils

---

### lingmo-text-editor

**职责**：文本编辑器。语法高亮（通过 KSyntaxHighlighting，支持超过
200 种语言定义）。代码折叠。行号显示。括号匹配。自动缩进。
多光标编辑。查找替换（普通文本/正则表达式）。会话恢复（重启后重新打开
之前未保存的标签页）。文件变更外部检测（文件被其他程序修改时的通知和
重新加载选项）。大文件支持（超过 100MB 时使用只读流模式）。
打印（调用 lingmo-print 服务）。

**依赖**：
- 构建时：Qt 6 (Core, Gui, Qml, Quick, Widgets)，lingmo-theme，lingmo-style，
  lingmo-controls，lingmo-icon，lingmo-window，lingmo-apps-common，
  lingmo-utils，KSyntaxHighlighting (KF6::SyntaxHighlighting)
- 运行时：Qt 6，lingmo-theme，lingmo-style，lingmo-controls，lingmo-icon，
  lingmo-window，lingmo-apps-common，lingmo-utils

---

### lingmo-terminal

**职责**：终端模拟器。使用 libvterm（纯终端状态机/解析器）或 VTE（GTK
终端模拟器部件封装）实现终端核心。标签页管理（新建/关闭/拖拽重新排序/
分离为新窗口）。分栏（上下/左右分割，每个分栏独立目录）。配置文件
（多组配色方案、字体、透明度/背景图片、滚动行数、光标样式）。
搜索（反向搜索、增量搜索、大小写敏感/正则）。超链接检测（URL、email
可点击打开）。自定义快捷键。真彩色支持（24 位色彩）。Sixel/Kitty 图像
协议支持。SSH 快捷连接管理器。

**依赖**：
- 构建时：Qt 6 (Core, Gui, Qml, Quick, Widgets)，lingmo-theme，lingmo-style，
  lingmo-controls，lingmo-icon，lingmo-window，lingmo-apps-common，lingmo-utils，
  libvterm（或 vte-ng）
- 运行时：Qt 6，lingmo-theme，lingmo-style，lingmo-controls，lingmo-icon，
  lingmo-window，lingmo-apps-common，lingmo-utils

---

### lingmo-image-viewer

**职责**：图片查看器。缩略图条（可滚动）。缩放（适配窗口/实际大小/
自定义缩放比例/缩放到选区）。全屏模式（通过快捷键或双击，隐藏工具栏）。
幻灯片播放（自定义间隔、过渡动画、随机顺序）。EXIF/XMP/IPTC 元数据
显示面板。基础编辑：裁剪（拖拽选区）、旋转（90°/自定义）、
翻转（水平/垂直）、调整（亮度/对比度/饱和度/色温）。RAW 格式支持
（通过 libraw 解码 CR2/NEF/ARW/DNG 等）。拖放支持（拖出到其他应用）。

**依赖**：
- 构建时：Qt 6 (Core, Gui, Qml, Quick, Widgets)，lingmo-theme，lingmo-style，
  lingmo-controls，lingmo-icon，lingmo-window，lingmo-apps-common，lingmo-utils，
  exiv2，libraw (可选)
- 运行时：Qt 6，lingmo-theme，lingmo-style，lingmo-controls，lingmo-icon，
  lingmo-window，lingmo-apps-common，lingmo-utils

---

### lingmo-document-viewer

**职责**：文档阅读器。PDF（通过 Poppler：渲染、文本选择、超链接、目录、
表单填写、注释、数字签名验证）。EPUB（通过 Qt WebEngine 渲染 HTML 内容
或自定义渲染器）。漫画书（CBR/CBZ/ZIP/RAR，使用 libzip/libarchive 解压，
逐页渲染）。DjVu（通过 DjVuLibre）。视图模式：连续滚动（默认）、
单页模式、双页模式。目录导航（从 PDF 书签或 EPUB 目录提取）。
文本搜索（高亮所有匹配、匹配计数）。注释（高亮、下划线、删除线、
文本框、附注）。书签（每个文档独立保存）。选区复制（选择文本复制到
剪贴板）。打印（通过 lingmo-print D-Bus 服务）。

**依赖**：
- 构建时：Qt 6 (Core, Gui, Qml, Quick, Widgets, WebEngine 可选)，
  lingmo-theme，lingmo-style，lingmo-controls，lingmo-icon，lingmo-window，
  lingmo-apps-common，lingmo-utils，poppler (libpoppler-qt6)，DjVuLibre (可选)，
  libzip，libarchive，libmagic
- 运行时：Qt 6，lingmo-theme，lingmo-style，lingmo-controls，lingmo-icon，
  lingmo-window，lingmo-apps-common，lingmo-utils

---

### lingmo-utilities

**职责**：小型工具应用集合（共享一个仓库因为它们都很小、依赖简单、
变更频率低）。

包含三个独立二进制：

1. **计算器**：基础模式（四则运算、百分比、平方根）、科学模式
   （三角函数、对数、指数、常量）、程序员模式（十六进制/二进制/十进制、
   位运算、字节操作）、货币换算器（通过 API 获取汇率、离线使用缓存汇率）、
   单位换算器（长度/重量/温度/体积/速度/压力等）。表达式历史记录（保留
   最近 100 条）。

2. **时钟/日历**：世界时钟（添加城市、时区地图选择、日光时间指示）、
   闹钟（systemd timer 集成、多闹钟、重复、自定义铃声）、计时器
   （多计时器、预设值、倒计时结束时通知）、秒表（计次、暂停）、
   日历（月视图/周视图/日视图、从在线账户同步 CalDAV 事件）。

3. **字体查看器**：已安装字体列表（网格/列表视图）、字体预览
   （示例文本预览、字号滑块、样式选择、行距控制）、字体详情
   （OpenType 特性、字符映射表、版权信息）、字体安装（拖放 .ttf/.otf 文件，
   安装到用户字体目录）、字体比较（并排比较两种字体）。

**为什么合并成一个仓库**：这三个应用都非常小（每个 < 3000 行），
依赖简单（仅 Qt + LingmoUI），极少需要独立发布。给予各自独立的仓库
只会增加维护成本而不带来实际好处。但它们与文件管理器或终端不共享代码，
所以合并在"工具"类别下是合理的。

**依赖**：
- 构建时：Qt 6 (Core, Gui, Qml, Quick, Widgets, Network 用于货币换算)，
  lingmo-theme，lingmo-style，lingmo-controls，lingmo-icon，lingmo-window，
  lingmo-apps-common，lingmo-utils
- 运行时：Qt 6，lingmo-theme，lingmo-style，lingmo-controls，lingmo-icon，
  lingmo-window，lingmo-apps-common，lingmo-utils

---

### lingmo-capture

**职责**：截图 + 屏幕录制。统一界面入口，两种模式切换。

**截图模式**：区域选择（拖拽选择矩形区域、窗口吸附、全屏）、
窗口选择（鼠标悬停窗口高亮并自动捕捉）、延迟定时（3/5/10 秒自定义）、
鼠标指针包含/排除。标注工具（箭头、矩形、椭圆形、自由画笔、文本标注、
高亮、马赛克/模糊、数字序号）、撤销/重做、颜色选择器和线宽选择器。
保存（PNG/JPEG/WebP）、剪贴板直接复制、分享到 imgur 或其他外部服务。

**录制模式**：区域/窗口/全屏录制。音源选择（麦克风、系统音频、两者）。
视频格式（MP4 (H.264/H.265)、WebM (VP8/VP9/AV1)、GIF）、帧率和质量设置。
通过 PipeWire screencast portal (`xdg-desktop-portal`) 实现 Wayland 录制。
录制中显示录制工具栏（暂停、停止、录制时间）。

**依赖**：
- 构建时：Qt 6 (Core, Gui, Qml, Quick, Widgets, DBus, Network, Multimedia)，
  lingmo-theme，lingmo-style，lingmo-controls，lingmo-icon，lingmo-window，
  lingmo-apps-common，lingmo-ipc，lingmo-utils，libpipewire，
  ffmpeg (libavcodec/libavformat/libavutil)
- 运行时：Qt 6，lingmo-theme，lingmo-style，lingmo-controls，lingmo-icon，
  lingmo-window，lingmo-apps-common，lingmo-ipc，lingmo-utils

---

### lingmo-archive

**职责**：归档管理器。支持打开/查看/创建/提取格式：
ZIP (libzip)、tar (libarchive)、tar.gz/tar.bz2/tar.xz (libarchive + 压缩库)、
7z (p7zip 或 libarchive)、rar (libarchive + 非免费解压器，可选)、
ISO 镜像（只读浏览）。创建归档：选择文件/目录、选择归档格式、设置选项
（压缩级别、加密密码、分卷大小、存储路径）。提取归档：提取到指定目录、
提取选中文件、提取到当前目录、提取到自动命名目录。归档浏览器：
树状目录视图、文件筛选和排序、快速预览（提取到临时目录并用关联应用打开）、
属性显示（文件大小、压缩率、修改时间、CRC32）。进度对话框（大文件/大
压缩进度显示、取消操作）。

**依赖**：
- 构建时：Qt 6 (Core, Gui, Qml, Quick, Widgets)，lingmo-theme，lingmo-style，
  lingmo-controls，lingmo-icon，lingmo-window，lingmo-apps-common，lingmo-utils，
  libarchive，libzip，p7zip (可选，调用 7z 二进制)
- 运行时：Qt 6，lingmo-theme，lingmo-style，lingmo-controls，lingmo-icon，
  lingmo-window，lingmo-apps-common，lingmo-utils

---

### lingmo-system-tools

**职责**：系统工具集合。包含两个独立二进制：

1. **系统监视器**：进程管理器（树状/列表视图、进程过滤/搜索、发送信号
   终止/暂停/继续、调整 Nice 值、CPU 占用/内存占用排序、文件描述符查看、
   环境变量查看）。资源图表（CPU 核心使用率、内存/RAM+交换使用、磁盘
   读写速率、网络收发速率、GPU 使用率/显存/温度）。文件系统（挂载点、
   总容量/已用/可用、文件系统类型、使用百分比）。GPU 监控
   （通过 NVML/NVIFR 对于 NVIDIA，通过 libdrm/amdgpu 对于 AMD，
   通过 sysfs 对于 Intel）。systemd 服务管理（服务列表、启动/停止/重启/
   启用/禁用、查看 journal 日志）。

2. **磁盘工具**：磁盘列表（通过 libblockdev/udev 枚举的块设备列表，
   显示设备名称、型号、容量、传输协议、旋转/固态标记）。分区表
   （MBR/GPT 视图，显示分区大小、文件系统、挂载点、标签、UUID）。
   分区操作（创建/删除/调整大小/格式化，通过 util-linux (sfdisk/parted)
   或 libblockdev）。文件系统创建（ext4/Btrfs/XFS/FAT32/NTFS 格式化）。
   挂载/卸载。SMART 数据（通过 libatasmart/smartctl、显示健康状态、
   属性值、离线测试执行）。磁盘基准（顺序/随机读写测试、IOPS 测试、
   延迟测试）。ISO 写入（将 ISO 镜像写入 USB 驱动器）。

**为什么合并成一个仓库**：两个工具都使用同一套系统信息库
（通过 /proc/sysfs/libblockdev 获取数据）。它们面向同一类用户群体
（高级用户/系统管理员）。合并后共享系统信息读取代码、设备枚举代码、
权限提升代码。

**依赖**：
- 构建时：Qt 6 (Core, Gui, Qml, Quick, Widgets, DBus, Charts (Qt Charts
  或自定义绘图))，lingmo-theme，lingmo-style，lingmo-controls，lingmo-icon，
  lingmo-window，lingmo-apps-common，lingmo-config，lingmo-ipc，lingmo-utils，
  libblockdev，libatasmart (可选，SMART)，udev (libudev)，procps-ng (libproc2)
- 运行时：Qt 6，lingmo-theme，lingmo-style，lingmo-controls，lingmo-icon，
  lingmo-window，lingmo-apps-common，lingmo-config，lingmo-ipc，lingmo-utils

---

### lingmo-software

**职责**：软件中心。AppStream 应用目录（通过 AppStream-Qt 解析 `/usr/share/appdata`
和 `/var/lib/flatpak/app/` 中的元数据）。应用分类浏览（图形/开发/办公/网络/
游戏/影音/工具 等）。搜索（按名称/描述/关键词）。特色应用（编辑精选）。
已安装应用管理（列出已安装、批量移除）。更新管理（系统更新和 Flatpak 应用
更新分开显示、批量更新、更新详情查看）。后端：PackageKit（通过 apt/rpm/dnf
处理系统包）、Flatpak（通过 flatpak D-Bus API 或 libflatpak 处理沙箱应用）。
应用详情页（截图轮播、描述/变更日志、评分、评论、依赖关系、文件列表、
Flatpak 权限显示）。应用评级和评论（通过 OCS 协议或 AppStream 评分服务）。
Flatpak 权限编辑（在安装后微调应用的沙箱权限）。

**依赖**：
- 构建时：Qt 6 (Core, Gui, Qml, Quick, Widgets, DBus, Network)，lingmo-theme，
  lingmo-style，lingmo-controls，lingmo-icon，lingmo-window，lingmo-apps-common，
  lingmo-config，lingmo-ipc，lingmo-utils，AppStream-Qt，PackageKit-Qt6，
  flatpak (libflatpak 可选)
- 运行时：Qt 6，lingmo-theme，lingmo-style，lingmo-controls，lingmo-icon，
  lingmo-window，lingmo-apps-common，lingmo-config，lingmo-ipc，lingmo-utils，
  packagekitd

---

### lingmo-info

**职责**：系统信息和诊断工具。包含：

1. **系统信息**：操作系统（发行版名称、版本、内核版本、架构）、
   桌面环境（Lingmo Desktop 版本号、Qt 版本、KWin 版本）、CPU
   （型号名、物理/逻辑核心数、频率、架构、缓存大小）、内存
   （总量/可用、交换总量/可用）、显卡（型号、驱动、OpenGL/Vulkan 版本、
   显存）、磁盘（列出所有磁盘的总容量、分区布局摘要）、网络接口
   （接口名、IP/MAC 地址、连接状态）。

2. **崩溃报告查看器**：查看最近应用崩溃报告（coredumpctl 列表、崩溃时间、
   应用名称、信号类型）、崩溃详情（backtrace、寄存器转储、触发时截图）、
   提交崩溃报告到开发团队、清空崩溃历史。

3. **日志查看器**：systemd journald 日志查看（按时间线/严重级别/服务
   单位过滤、搜索日志条目、日志高亮）。导出日志为文本文件。

**为什么需要独立**：系统信息是用户在寻求帮助时首先查看的页面。
它与系统监视器的"实时数据"不同，它提供的是"静态配置"信息。
它需要访问系统文件（/etc/os-release、/sys/class/dmi/ 等），
需要与 coredumpctl/journalctl 集成。它与管理任务无关，纯粹提供信息。

**依赖**：
- 构建时：Qt 6 (Core, Gui, Qml, Quick, Widgets)，lingmo-theme，lingmo-style，
  lingmo-controls，lingmo-icon，lingmo-window，lingmo-apps-common，lingmo-utils，
  systemd (libsystemd 用于 journal)
- 运行时：Qt 6，lingmo-theme，lingmo-style，lingmo-controls，lingmo-icon，
  lingmo-window，lingmo-apps-common，lingmo-utils

---

## 11. 主题资产层 (1 个仓库)

---

### lingmo-assets

**职责**：Lingmo Desktop 所有视觉和音频资产。纯数据（无代码）。
包含目录：
- `themes/`：Lingmo 默认主题（亮色/暗色/高对比度三套 JSON 令牌文件，
  以及 QML 主题覆盖组件）。注意：令牌的格式规范和加载逻辑在
  lingmo-theme 中，这里只是数据文件
- `icons/`：完整图标主题（遵循 Freedesktop Icon Theme 规范：
  apps/actions/devices/places/mimetypes/emblems/status/categories
  目录。SVG 矢量 + PNG 回退。多尺寸：16/22/24/32/48/64/96/128/256。
  符号化图标支持主题色重着色）
- `cursors/`：光标主题（XCursor 标准动画光标。标准大小支持。HiDPI 兼容
  通过 XCursor 多大小图像）
- `sounds/`：音效主题（启动/关闭/登录/注销/通知错误/通知信息/通知警告/
  系统警报/窗口关闭/音量变化/截图/设备连接/设备断开。Ogg Vorbis 格式，
  短音频片段）
- `wallpapers/`：壁纸集合（静态壁纸 JPEG/PNG（多种分辨率适配），
  动态壁纸（QML ShaderEffect 着色器壁纸，支持参数/时间/交互），
  轮播壁纸 XML 配置，社区壁纸竞赛获奖作品）
- `sddm/`：SDDM 登录主题（QML 组件集合，调用 LingmoUI 样式，
  背景图片集成，会话/用户选择和密码输入 UI）
- `plymouth/`：Plymouth 开机启动主题（脚本 + 图片 + 动画配置，
  文字模式回退）
- `gtk/`：GTK3/4 主题（CSS 文件匹配 Lingmo 设计令牌颜色/圆角/间距/阴影。
  metacity/metacity-theme-3.xml 兼容传统窗口装饰

**为什么需要独立**：
- 资产是数据文件，不需要编译。独立的仓库意味着设计者和社区贡献者
  可以用纯 Git 工作流（不需要 CMake 或编译器）
- 资产仓库的大小远大于代码仓库（图标主题可能 > 50MB），频繁更新
  图标不应该触发任何 CI 构建流程
- 资产可以独立于桌面版本发布（新壁纸可以随时添加），版本号由
  文件系统路径管理

**依赖**：无（运行时被各个组件读取）

**发布策略**：不适用版本号。通过 Git 标签标记兼容的桌面版本。

---

## 12. 开发者工具层 (1 个仓库)

---

### lingmo-sdk

**职责**：Lingmo 开发者工具和文档。提供给第三方应用开发者。
包含：
- **API 文档**：lingmo-theme、lingmo-style、lingmo-controls、lingmo-icon、
  lingmo-window、lingmo-effects、lingmo-applets、lingmo-config、lingmo-plugin
  的 Doxygen + QDoc 生成的 API 参考文档
- **QML 文档**：所有 LingmoUI QML 模块的 QML 类型文档（Button、TextField、
  Theme singleton 等），带截图和代码示例
- **插件开发向导**：如何开发 shell applet、settings panel、search provider、
  content extractor 的完整指南
- **项目模板**：Qt Widgets 应用 CMake 模板、QML 应用 CMake 模板、
  shell applet 模板、settings panel 模板、search provider 模板
- **CMake 宏**：`lingmo_add_plugin()`、`lingmo_add_applet()`、
  `lingmo_add_settings_panel()`、`lingmo_add_search_provider()`、
  `lingmo_add_extractor()` — 统一的 CMake 模块
- **UI 审查工具**（lingmo-ui-inspector）：独立工具，附加到运行中的
  Lingmo 应用，显示当前应用的 LingmoTheme 令牌值，实时修改令牌并预览
  效果，导出当前主题快照，控件的焦点层/布局边界可视化
- **AppStream 生成器**：CLI 工具，从 CMake 项目文件生成
  AppStream XML 元数据
- **CI 模板**：GitHub Actions 模板文件，适用于 Lingmo 第三方应用
  项目的构建/测试/打包

**为什么需要独立**：
- SDK 的发布节奏完全独立于桌面本身。当有新 API 发布时，SDK 随之更新，
  但反之不必成立
- SDK 的消费者是外部应用开发者，不是桌面组件

**依赖**：
- 构建时：lingmo-theme，lingmo-style，lingmo-controls，lingmo-icon，
  lingmo-window，lingmo-effects，lingmo-applets，lingmo-config，lingmo-plugin
  （仅头文件）
- 运行时：无（文档、模板、脚本）

**发布策略**：新增一个 API 的文档为 minor。模板和 CMake 宏可随时更新。

---

## 依赖关系总图

```
                         ┌──────────────────────────────────────────────────────┐
                         │                   应用层 (12 仓库)                      │
                         │  lingmo-file-manager  lingmo-text-editor             │
                         │  lingmo-terminal  lingmo-image-viewer               │
                         │  lingmo-document-viewer  lingmo-utilities           │
                         │  lingmo-capture  lingmo-archive                     │
                         │  lingmo-system-tools  lingmo-software               │
                         │  lingmo-info  lingmo-apps-common                    │
                         └───────────────────────┬──────────────────────────────┘
                                                 │
                         ┌───────────────────────▼──────────────────────────────┐
                         │  lingmo-control-center  (settings)                   │
                         └───────────────────────┬──────────────────────────────┘
                                                 │
          ┌──────────────────────────────────────┼──────────────────────────────────────┐
          │                                      │                                      │
          ▼                                      ▼                                      ▼
┌─────────────────────┐  ┌─────────────────────────────┐  ┌─────────────────────────────┐
│   外壳层 (8 仓库)    │  │   系统服务层 (7 仓库)          │  │   集成层 (3 仓库)            │
│                     │  │                             │  │                             │
│ lingmo-shell-core   │  │ lingmo-settings-daemon      │  │ lingmo-portals              │
│ lingmo-shell-surfaces│  │ lingmo-power-manager       │  │ lingmo-polkit               │
│ lingmo-applets      │  │ lingmo-display              │  │ lingmo-flatpak              │
│ lingmo-notification │  │ lingmo-input                │  │                             │
│ lingmo-osd          │  │ lingmo-audio                │  └─────────────────────────────┘
│ lingmo-runner       │  │ lingmo-thumbnails            │
│ lingmo-decoration   │  │ lingmo-print                │
│ lingmo-shell-scripts│  └──────────────┬──────────────┘
└──────────┬──────────┘                 │
           │                            │
           │    ┌───────────────────────┼───────────────────────┐
           │    │                       │                       │
           ▼    ▼                       ▼                       ▼
┌──────────────────────┐  ┌─────────────────────┐  ┌─────────────────────┐
│  lingmo-search        │  │  lingmo-network      │  │  lingmo-bluetooth    │
│  (搜索层)             │  │  (网络层)             │  │  (蓝牙层)             │
└──────────┬───────────┘  └─────────────────────┘  └─────────────────────┘
           │
           └──────────────────┬──────────────────────┐
                              │                      │
         ┌────────────────────▼──────┐  ┌─────────────▼─────────────┐
         │   lingmo-session          │  │  lingmo-lockscreen        │
         │   (会话管理层)              │  │  (锁屏)                   │
         └────────────────┬──────────┘  └─────────────┬─────────────┘
                          │                           │
                          └──────────┬────────────────┘
                                     │
         ┌───────────────────────────▼───────────────────────────────┐
         │               lingmo-core (基础设施层, 4 仓库)              │
         │  lingmo-config  lingmo-plugin  lingmo-ipc  lingmo-utils   │
         └───────────────────────────┬───────────────────────────────┘
                                     │
         ┌───────────────────────────▼───────────────────────────────┐
         │               lingmo-ui (UI 框架层, 6 仓库)                │
         │  lingmo-theme  lingmo-style  lingmo-controls              │
         │  lingmo-effects  lingmo-icon  lingmo-window               │
         └───────────────────────────┬───────────────────────────────┘
                                     │
         ┌───────────────────────────▼───────────────────────────────┐
         │                         Qt 6                              │
         └───────────────────────────┬───────────────────────────────┘
                                     │
         ┌───────────────────────────▼───────────────────────────────┐
         │                       KWin                                │
         │            (透过 decoration/script/effect 插件集成)         │
         └───────────────────────────────────────────────────────────┘
```

### 数据依赖（无编译依赖）

```
lingmo-shell-surfaces  ──reads──▶  lingmo-assets (壁纸)
lingmo-shell-core      ──reads──▶  lingmo-shell-scripts (加载 KWin 脚本)
lingmo-notification    ──reads──▶  lingmo-assets (通知音效)
lingmo-lockscreen      ──reads──▶  lingmo-assets (锁屏壁纸)
lingmo-control-center  ──reads──▶  lingmo-assets (壁纸预览、图标预览)
所有应用               ──reads──▶  lingmo-assets (图标)
lingmo-session         ──reads──▶  lingmo-assets (登录主题)
```

### 无循环依赖

每个仓库只依赖下层仓库。不存在 A 依赖 B 且 B 依赖 A 的情况。

---

## 发布策略总览

| 层级 | 发布频率 | 版本策略 | 说明 |
|------|---------|---------|------|
| lingmo-ui (6 仓库) | 按需 | semver | API 稳定承诺，major 极小 |
| lingmo-core (4 仓库) | 按需 | semver | 基础设施，向后兼容要求高 |
| lingmo-shell (8 仓库) | 随桌面版本 | semver | shell-scripts 可高频独立 |
| lingmo-session (2 仓库) | 随桌面版本 | semver | 锁屏安全补丁优先 |
| lingmo-daemon (7 仓库) | 随桌面版本 | semver | 可独立修补关键问题 |
| lingmo-network (2 仓库) | 随桌面版本 | semver | 协议栈 API 变化时响应 |
| lingmo-search | 随桌面版本 | semver | 索引格式升级需协调 |
| lingmo-control-center | 随桌面版本 | semver | 面板随应用共享发布节奏 |
| lingmo-integration (3 仓库) | 按需 | semver | 门户规范更新即时响应 |
| lingmo-apps (12 仓库) | 按需 | semver | 每个应用可独立发版 |
| lingmo-assets | 随时 | 无版本 | Git tag 标记兼容桌面版 |
| lingmo-sdk | 按需 | semver | API 更新后同步更新 |

所有仓库支持**同时发布**（统一定时发布桌面版本，如 Lingmo Desktop 3.0）
和**独立发布**（关键安全修复可以绕过统一发布窗口立即发布）。

---

## 总结

47 个仓库的划分遵循以下决策原则：

1. **UI 框架 (6 个)**：面向第三方开发者，各自按需选择依赖
2. **基础设施 (4 个)**：核心服务内聚但彼此解耦，每个都有独立的 API 契约
3. **外壳 (8 个)**：故障隔离（不同进程），开发隔离（JS 脚本 vs C++ 插件 vs shell 核心）
4. **会话 (2 个)**：安全关键组件最小化信任边界
5. **系统服务 (7 个)**：不同的硬件依赖（PipeWire/UPower/CUPS）要求不同的
   构建和测试环境
6. **网络 (2 个)**：协议栈差异（NetworkManager vs BlueZ）
7. **搜索 (1 个)**：整体性高（索引/引擎/提取器共享索引文件）
8. **控制中心 (1 个)**：单一应用，插件化面板
9. **集成 (3 个)**：安全域不同（沙箱权限 vs 系统权限 vs 应用管理）
10. **应用 (12 个)**：每个应用有不同的依赖链、用户群体、测试策略、
    维护者
11. **资产 (1 个)**：纯数据，无代码
12. **SDK (1 个)**：面向外部开发者的工具和文档
