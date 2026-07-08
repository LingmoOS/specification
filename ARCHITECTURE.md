# Lingmo Desktop Architecture

> A modern, modular Linux desktop environment built on Qt 6 and KWin.

---

## Phase 1: Analysis of Existing Desktop Environments

### KDE Plasma

**Architecture**: KDE Plasma is built on ~80+ KDE Frameworks libraries, with a shell
(plasma-workspace) separate from desktop applets (plasma-desktop). KWin provides the
compositor. KDED runs background services. Configuration uses KConfig (INI-like files).

**Repository strategy**: Extreme fragmentation. 80+ frameworks repos, 20+ Plasma repos,
200+ application repos. Each KDE Framework (e.g., KCoreAddons, KConfig, KWidgetsAddons)
is in its own repository, often containing only 1-3 classes. This makes understanding
the full system difficult for new contributors.

**Plugin architecture**: KPlugin metadata system uses JSON .desktop files for service
discovery. Applets are plugins loaded into containments. Well-designed but complex.

**Session management**: startkde script → KSMServer → systemd user session. KDED modules
start as part of the session.

**Advantages**:
- Mature and battle-tested (25+ years)
- KWin is the most capable Wayland compositor on Linux
- KPlugin system is well-designed
- Extremely flexible and customizable
- KConfig is simple and effective

**Disadvantages**:
- Over-fragmentation: 80+ framework repos is not maintainable at scale
- KDE Frameworks dependencies are too granular
- Technical debt from Qt4/Qt5 era
- Inconsistent UX across different applications
- QWidgets vs QQuick dualism not fully resolved (two styling systems)
- Kirigami doesn't fully address the convergence problem
- Configuration overload overwhelms users

---

### GNOME

**Architecture**: Monolithic gnome-shell (JavaScript) on mutter compositor. Session
managed by gnome-session. Settings use GSettings (dconf backend). Extensions hook
into shell internals via unofficial JS API. Applications use libadwaita.

**Repository strategy**: ~30-40 core repos. Medium-sized per functional area.

**Plugin architecture**: Shell extensions (JS, loaded by gnome-shell). Fragile — they
break on every minor release. No binary plugin API for the shell.

**Session management**: gnome-session → GDM → systemd user session.

**Advantages**:
- Consistent design language (libadwaita)
- Good Wayland support
- Strong accessibility infrastructure
- Well-polished user experience
- Flatpak integration

**Disadvantages**:
- Extension API is fragile and unofficial (monkey-patching)
- Limited customization (intentionally)
- JavaScript in the shell causes performance issues
- High memory usage
- GTK focus limits toolkit choice
- Mutter lacks advanced compositor features (tiling, scripting)
- Hard to fork or deeply customize

---

### Deepin Desktop (DDE)

**Architecture**: Qt-based with DTK (Deepin Tool Kit) providing UI framework. dde-shell
for the shell, dde-dock for panel, dde-desktop for desktop, dde-control-center for
settings, dde-daemon for services. Uses KWin as compositor (dde-kwin fork).

**Repository strategy**: ~30 repos for full desktop. Medium-sized. DTK split across
4-5 repos (DTKCore, DTKWidget, DTKQuick, DTKGui).

**Plugin architecture**: dde-dock plugins (C++), dde-control-center plugins (QML),
dde-shell plugins (QML). Multiple incompatible plugin systems.

**Session management**: startdde → D-Bus activation → systemd user services.

**Advantages**:
- Beautiful, consistent visual identity
- DTK provides Qt Widgets + QML integration
- Good out-of-box experience
- KWin integration for compositing
- Input method integration for CJK languages

**Disadvantages**:
- DTK has dual parallel implementations (DWidget vs DQuick) = duplicated maintenance
- DTK Widgets subclass every Qt widget → fragile, hard to update
- QML components lag behind Widgets in features
- Some components tied to Deepin OS distribution
- Limited international community
- Documentation is Chinese-only for some components
- Dark theme support incomplete in Widgets path

---

### COSMIC (Pop!_OS)

**Architecture**: Rust-based with Iced toolkit. COSMIC Epoch compositor, cosmic-applets
for widgets, COSMIC design tokens for theming.

**Repository strategy**: ~15 repos for core desktop. Well-organized.

**Plugin architecture**: cosmic-applets (Iced widgets), service type system for
discovery.

**Session management**: cosmic-services (service manager written in Rust).

**Advantages**:
- Modern Rust codebase (memory safe)
- Consistent design tokens
- Performance-oriented
- Clean architecture from the start
- Well-documented design decisions

**Disadvantages**:
- Very new, not yet stable
- Iced toolkit is immature (less than Qt)
- Limited ecosystem
- Small community
- Not Qt-based — can't leverage Qt application ecosystem
- Must reinvent many wheels (text rendering, accessibility, etc.)

---

### Cinnamon

**Architecture**: GNOME 3 fork. Cinnamon shell (CJS), muffin compositor (mutter fork),
nemo file manager (Nautilus fork).

**Repository strategy**: ~10 repos for core desktop.

**Plugin architecture**: Applets (JS, Python, C++), desklets, extensions.

**Advantages**:
- Traditional desktop metaphor (familiar)
- Stable and reliable
- Good performance on modest hardware

**Disadvantages**:
- Old GNOME technology base
- muffin lags behind mutter significantly
- Python applets cause performance issues
- Design feels dated
- GTK3 bound (not GTK4)
- Less active development

---

### XFCE

**Architecture**: Modular GTK2/GTK3 components. xfwm4 compositor, xfce4-panel,
xfdesktop, thunar file manager.

**Repository strategy**: ~15 repos.

**Advantages**:
- Lightweight and fast
- Stable codebase
- Simple and easy to maintain
- Good for older hardware

**Disadvantages**:
- Visually dated
- Limited features
- GTK2/3 split in codebase
- No Wayland support (in progress)
- Limited compositor features

---

### Key Lessons for Lingmo Desktop

| Lesson | Source | Application |
|--------|--------|-------------|
| Avoid over-fragmentation | KDE | Merge libraries into functional units (not 1 class per repo) |
| Avoid dual UI implementations | Deepin | Single theme token system shared by QWidgets and QML |
| Don't monkey-patch for extensions | GNOME | First-class plugin API with stable ABI |
| Don't reinvent Qt | COSMIC | Use Qt 6, don't write a new toolkit |
| Don't fork the compositor | Cinnamon, Deepin | Use KWin via its plugin system |
| Design tokens are essential | COSMIC | LingmoTheme drives all visual rendering |
| KWin is the right compositor | KDE, Deepin | Mature, capable, Wayland-native |

---

## Phase 2: Complete Component List

### Legend

- **[S]** — System service (daemon)
- **[L]** — Library  
- **[A]** — Application (user-facing)
- **[P]** — Plugin / extension
- **[D]** — Data / assets
- **[T]** — Tool

### Desktop Shell

| Component | Type | Description |
|-----------|------|-------------|
| Panel (Main Bar) | A | Primary panel with app menu, task manager, system tray, clock |
| Desktop Surface | A | Wallpaper rendering, desktop icons, right-click context menu |
| Overview | A | Window overview, workspace switcher, application grid |
| Notification Center | A | Notification history, do-not-disturb, notification groups |
| OSD | A | On-screen display (volume, brightness, media, etc.) |
| System Tray | P | StatusNotifierItem host area |
| App Menu | A | Application launcher / start menu |
| Task Manager | P | Window management in panel |
| Quick Settings | A | Expandable system controls in panel |
| Runner (Alt+F2) | A | Quick command/search dialog |
| Window Decorations | P | KWin decoration plugin (Lingmo theme) |
| KWin Scripts | P | Window behavior, placement, focus, animations |
| KWin Effects | P | Custom visual effects |
| Lock Screen Integration | P | ext-session-lock protocol client |

### Session

| Component | Type | Description |
|-----------|------|-------------|
| Session Manager | S | Session lifecycle, startup sequence, crash recovery |
| Session Config | S | Environment setup, autostart management |
| Screen Locker | S | Wayland screen locker (ext-session-lock) |
| User Switcher | S | Fast user switching |
| Display Manager Greeter | A | SDDM theme (Lingmo-branded) |

### System Services (Daemon)

| Component | Type | Description |
|-----------|------|-------------|
| Settings Daemon | S | Apply settings changes, monitor hardware (GSettings-style but with our config) |
| Power Manager | S | UPower integration, battery monitoring, suspend/hibernate |
| Display Manager Service | S | Display configuration (KScreen/KWin integration), night light, color management |
| Input Manager | S | Input devices (keyboard, mouse, touchpad, tablet, touchscreen) |
| Audio Manager | S | PipeWire/WirePlumber integration, volume control, audio profiles |
| Media Key Handler | S | Keyboard media keys → actions |
| Thumbnail Generator | S | Thumbnail generation for file manager |
| Removable Devices | S | udisks2 integration, mount/unmount notifications |
| Time Manager | S | NTP/date/time configuration |
| Printing Service | S | CUPS integration |
| Locale Monitor | S | Language/locale change handling |
| Power Profile | S | Performance/power profile switching |

### Control Center (Settings)

| Component | Type | Description |
|-----------|------|-------------|
| Control Center Application | A | Modular settings application with plugin panels |
| Appearance | P | Theme, wallpaper, fonts, icons, cursors |
| Display | P | Monitor arrangement, resolution, scaling, refresh rate |
| Power | P | Power settings, battery, screen lock |
| Input Devices | P | Mouse, touchpad, keyboard, shortcuts, tablet |
| Sound | P | Audio devices, volumes, input/output, profiles |
| Network | P | Wi-Fi, Ethernet, VPN, proxy |
| Bluetooth | P | Bluetooth devices, pairing, file transfer |
| Printers | P | Printer setup, queues, defaults |
| Users | P | User accounts, avatars, passwords |
| Date & Time | P | Time zone, NTP, clock format |
| Language & Region | P | Language, locale, formats, input methods |
| Default Applications | P | Default apps for web, mail, music, files, etc. |
| Online Accounts | P | Google, Nextcloud, Microsoft account integration |
| Accessibility | P | Sticky keys, slow keys, screen reader, zoom, contrast |
| Notifications | P | Per-app notification settings, do-not-disturb |
| Search | P | Search providers, file indexing settings |
| Privacy & Security | P | Location, camera, microphone permissions |
| Updates | P | System update management (PackageKit frontend) |
| About | P | System information, hardware details |
| Removable Devices | P | Auto-mount settings, media handling |
| Session | P | Startup applications, desktop effects |
| Firewall | P | Firewall configuration (firewalld) |
| Color | P | ICC color profiles for displays |
| Shortcuts | P | Custom keyboard shortcuts |

### Core Applications

| Component | Type | Description |
|-----------|------|-------------|
| File Manager | A | Full file manager (icon, list, split, dual-pane), tabs, search |
| Text Editor | A | Code/text editor, syntax highlighting, multiple tabs |
| Terminal Emulator | A | Terminal, tabs, profiles, search, transparency |
| Image Viewer | A | Image viewing, zoom, pan, slideshow, basic editing |
| Document Viewer | A | PDF, EPUB, comic book (CBR/CBZ), DjVu reader |
| Calculator | A | Basic, scientific, programmer, currency converter |
| Clock & Calendar | A | World clock, alarms, timers, calendar with CalDAV |
| Screenshot | A | Screen region/window/full capture, annotation, delayed capture |
| Screen Recorder | A | Video recording, audio source selection, region selection |
| Archive Manager | A | Create/extract/view archives (zip, tar, 7z, rar) |
| System Monitor | A | Process viewer, resource graphs, filesystem info |
| Disk Utility | A | Disk info, partitioning, formatting, SMART data |
| Software Center | A | App discovery (AppStream), PackageKit install/remove/update |
| Font Viewer | A | Font preview, install, manage |
| Diagnostic Center | A | System logs, crash reports, hardware info collection |
| First Run Wizard | A | Initial setup (language, keyboard, privacy, welcome tour) |
| About | A | System summary (OS, hardware, desktop version) |

### Core Libraries

| Component | Type | Description |
|-----------|------|-------------|
| LingmoTheme | L | Design tokens (colors, typography, metrics, shadows, animations) |
| LingmoStyle (QStyle) | L | QProxyStyle for Qt Widgets themed rendering |
| LingmoControls (QML) | L | QML controls module (Button, TextField, ComboBox, etc.) |
| LingmoWindow | L | Custom window frame helpers, titlebar |
| LingmoEffects | L | Blur, shadow, animation utilities |
| LingmoIcon | L | Icon theme loading, rendering, fallback |
| LingmoConfig | L | Configuration backend (INI/JSON/Keyfile with file watching) |
| LingmoIPC | L | D-Bus helpers, service discovery, message bus |
| LingmoPlugin | L | Plugin metadata, discovery, loading (JSON + shared library) |
| LingmoService | L | Process/service lifecycle management |
| LingmoCrash | L | Crash handler, backtrace, coredump integration |
| LingmoLogger | L | Structured logging, log levels, journald integration |
| LingmoFileUtils | L | File operations, MIME detection, URI handling |
| LingmoProcess | L | Process spawning, monitoring, output capture |

### Search & Indexing

| Component | Type | Description |
|-----------|------|-------------|
| File Indexer | S | Recursive file content/metadata indexer |
| Search Service | S | D-Bus search service, query handling |
| Full-Text Engine | L | Xapian/LMDB-based full-text search |
| Metadata Extractors | P | Pluggable extractors (PDF, images, music, documents) |
| Search Provider API | L | Registry for search providers (apps, files, settings, calculator) |
| Shell Search Interface | A | Unified search in overview and runner |

### Network Management

| Component | Type | Description |
|-----------|------|-------------|
| Network Applet | A | Wi-Fi, Ethernet, VPN status in panel |
| Network Settings | P | Full network configuration UI |
| Bluetooth Applet | A | Bluetooth status, device management in panel |
| Bluetooth Settings | P | Bluetooth configuration panel |
| VPN Manager | S | WireGuard/OpenVPN connection management |
| Proxy Configuration | S | System proxy settings |
| Firewall Settings | P | firewalld frontend (panel + control center) |

### Integration

| Component | Type | Description |
|-----------|------|-------------|
| Portal Backend | S | xdg-desktop-portal-lingmo: FileChooser, Screenshot, RemoteDesktop, ScreenCast, Wallpaper, Notification, OpenURI, Print, Camera, Location, Account, Email, Trash, Background, Secrets |
| PolKit Agent | A | PolicyKit authentication dialog (LingmoUI themed) |
| MIME Manager | L | MIME type ↔ application associations |
| Default App Manager | L | XDG default application configuration |
| URL Scheme Handler | L | Protocol handler registration (mailto, tel, etc.) |
| Calamares Modules | P | Calamares installer integration modules |
| Autostart Manager | L | XDG autostart directory monitoring and launch |
| Flatpak Integration | S | Flatpak app installation, update, permission management |
| AppStream Integration | L | AppStream metadata parsing for software center |

### Theme & Assets

| Component | Type | Description |
|-----------|------|-------------|
| Default Theme | D | Lingmo design theme (JSON tokens + QML theme components) |
| Light Theme | D | Light color variant |
| Dark Theme | D | Dark color variant |
| High Contrast Theme | D | Accessibility high-contrast variant |
| Icon Theme | D | Full icon set (apps, actions, devices, places, mimetypes, emblems) |
| Cursor Theme | D | Animated cursor set |
| Sound Theme | D | System sounds (notification, startup, shutdown, error) |
| Wallpaper Collection | D | Default wallpapers (static + animated/parallax) |
| SDDM Theme | D | Login screen theme |
| Boot Splash | D | Plymouth theme |
| GTK Theme | D | GTK3/4 theme matching Lingmo design (for non-Qt apps) |

### SDK & Developer Tools

| Component | Type | Description |
|-----------|------|-------------|
| Public API Headers | D | Stable public C++ headers for LingmoUI and LingmoCore |
| QML Import Documentation | D | QML type documentation for LingmoUI modules |
| Plugin Development Kit | T | Templates, examples, CMake macros for plugin development |
| UI Inspector | T | Developer tool for inspecting LingmoUI properties |
| AppStream Generator | T | Tool to generate AppStream metadata for Lingmo apps |
| CI Templates | D | GitHub Actions templates for Lingmo app developers |

### Infrastructure

| Component | Type | Description |
|-----------|------|-------------|
| D-Bus Service Files | D | D-Bus activation files for system services |
| systemd Service Files | D | systemd user/system service files for Lingmo components |
| PAM Configuration | D | PAM configuration for lock screen and authentication |
| Wayland Protocol Files | D | Lingmo-specific Wayland protocol XML definitions |
| Environment Setup | D | Profile scripts, environment variables |
| XSession Entry | D | .desktop file for display manager session selection |

---

## Phase 3: Merge Rationale

### Merging Decisions

#### LingmoShell Combines: Panel, Desktop, Overview, Notifications, Window Decorations, KWin Scripts

These are all part of a single shell process. The panel, desktop, and overview
share the containment/applet infrastructure. Notifications are rendered by the
shell. Window decorations are KWin plugins that share the same design tokens.
KWin scripts define behavior that the shell needs to coordinate with.

Splitting these into separate repos would introduce:
- Duplicated applet/containment infrastructure
- Version synchronization problems
- Inconsistent design across shell surfaces
- Unnecessary build complexity

#### LingmoDaemon Combines: Settings Daemon, Power, Display, Input, Audio, Thumbnails, Time, Printing

All are background services that start early in the session and run continuously.
They share:
- Common D-Bus service infrastructure
- Common logging and error handling
- Common privilege management
- Common activation/startup patterns
- They all depend on LingmoCore

Splitting them would mean each daemon reimplements the same service framework.

#### LingmoApplications Combines: All Core Applications

The core apps share:
- LingmoUI for consistent theming
- Common dialog components (file dialogs, about dialogs, settings dialogs)
- Common utilities (file operations, thumbnail loading, MIME handling)
- Translation infrastructure
- CI/build system
- Release coordination

The individual apps are well-separated internally with their own subdirectories,
tests, and translations. They can still be built and packaged independently
(CMakeLists.txt per app).

#### LingmoControlCenter Combines: All Settings Panels

The control center is a single application hosting plugin panels. Each panel is
a plugin loaded at runtime. The app provides:
- Window frame and navigation
- Search across all settings
- Change tracking (apply/reset)
- Panel discovery and loading

Panels can be developed independently as plugins. They share the same plugin
infrastructure from LingmoCore.

#### LingmoAssets Combines: All Theme Files

Icon themes, cursor themes, sound themes, and wallpapers are all data files.
They're versioned together to ensure visual consistency. GTK theme depends on
the same color tokens. SDDM theme and boot splash use the same brand colors.

Separate repos would require version coordination across repos for a single
visual update.

### Why NOT to merge further

**LingmoUI** separate from **LingmoCore**: Third-party app developers need the UI
framework without pulling in desktop infrastructure (IPC, plugin system, services).
LingmoUI depends only on Qt 6. LingmoCore depends on LingmoUI + Qt 6. This lets
external Qt apps adopt Lingmo styling with minimal dependencies.

**LingmoSession** separate from **LingmoShell**: The session manager runs before the
shell (systemd user service). It must start even if the shell crashes. The screen
locker is a separate process (security isolation). Mixing these with the shell
would create security and reliability issues.

**LingmoNetwork** separate from **LingmoControlCenter**: Network management has
its own background service (NetworkManager + our wrappers), its own panel applet,
and its own settings panel. The panel applet must run in the shell, and the
settings panel is a control center plugin, but the core logic (connection
management, VPN, proxy) is substantial enough for its own repo.

**LingmoSearch** separate from **LingmoDaemon**: Search includes a heavy background
indexer (file system scanning, content extraction), which has different resource
requirements than other daemon services. The indexer can be paused/stopped
independently. The search infrastructure is also used by the file manager and
the shell overview, making it a cross-cutting concern.

**LingmoIntegration** separate from **LingmoCore**: Portal backends are
xdg-desktop-portal D-Bus services — they run as separate processes. The PolKit
agent is a separate process. These have different lifecycle and security
requirements than core libraries. Mixing them would introduce unnecessary
dependencies for library consumers.

---

## Phase 4: Final Repository Architecture

---

### Repository: LingmoUI

**Purpose**: UI framework for Lingmo Desktop. The only component third-party
developers need for visual consistency. Zero desktop infrastructure dependencies.

**Dependencies**
- Build: Qt 6 (Core, Gui, Qml, Quick, Widgets, Svg)
- Runtime: Qt 6

**Internal Modules**

| Module | Responsibility |
|--------|----------------|
| LingmoTheme | Token system: color palettes, typography, spacing, corner radii, shadows, elevations, animations (durations, easing). Loads tokens from JSON/QML theme files at standard paths. Provides C++ and QML singleton APIs. |
| LingmoStyle | QProxyStyle implementation that reads LingmoTheme tokens for all standard Qt Widget rendering: buttons, inputs, scrollbars, sliders, checkboxes, radio buttons, tabs, menus, tooltips, toolbars, frames, progress bars, group boxes, splitters, spin boxes, dials, combo boxes |
| LingmoControls (QML Import) | QML module `LingmoUI.Controls`: Button, TextField, TextArea, ComboBox, Switch, CheckBox, RadioButton, TabBar, TabButton, Slider, ProgressBar, ScrollBar, ScrollView, SpinBox, Dial, ToolTip, Menu, MenuItem, ToolBar, RangeSlider |
| LingmoWindow | Window helpers: frameless window, titlebar widget, headerbar, window shadow, minimize/maximize/close buttons. Qt Widgets: `LingmoWindow` subclass of QMainWindow. QML: `LingmoUI.Window` |
| LingmoEffects | Blur (platform blur where available), shadow (9-patch or blur-based), elevation layering, transition utilities, ripple effect, highlight effect |
| LingmoIcon | Icon theme engine: loads from XDG icon themes, fallback chain, symbolic icon coloring to match theme palette, lazy loading with cache |
| LingmoAnimation | Easing curves, duration presets, spring animations, parallel/sequential animation helpers |

**Output Packages**

| Package | Contents |
|---------|----------|
| `liblingmo-ui` | Shared library (LingmoTheme, LingmoWindow, LingmoIcon, LingmoEffects, LingmoAnimation) |
| `lingmo-style` | QStyle plugin for Qt Widgets |
| `qml-module-lingmo-ui-controls` | QML LingmoUI.Controls module |
| `qml-module-lingmo-ui-window` | QML LingmoUI.Window module |
| `qml-module-lingmo-ui-effects` | QML LingmoUI.Effects module |
| `liblingmo-ui-dev` | Headers, CMake config files, pkg-config files |
| `lingmo-ui-themes-core` | Default theme JSON/QML files |

**Public API**

```
C++:
  Lingmo::Theme (singleton, property-based)
  Lingmo::Theme::palette() -> ColorPalette
  Lingmo::Theme::typography() -> Typography
  Lingmo::Theme::spacing() -> Metrics
  Lingmo::Theme::radius() -> CornerRadii
  Lingmo::Theme::shadow(elevation) -> Shadow
  Lingmo::Theme::animation(duration) -> qreal
  Lingmo::Window (QMainWindow subclass with Lingmo theming)
  Lingmo::IconLoader (icon lookup with fallback)
  Lingmo::StyleManager (configure QStyle)

QML:
  import LingmoUI.Controls 1.0
  import LingmoUI.Window 1.0
  import LingmoUI.Effects 1.0
  LingmoTheme (attached property / singleton)
```

---

### Repository: LingmoCore

**Purpose**: Desktop infrastructure libraries used by all Lingmo components.
Depends on LingmoUI for visual integration but provides non-visual desktop
services.

**Dependencies**
- Build: Qt 6 (Core, DBus, Xml), LingmoUI, glib-2.0 (for gsettings bridge)
- Runtime: Qt 6, LingmoUI

**Internal Modules**

| Module | Responsibility |
|--------|----------------|
| LingmoConfig | Configuration system: key-value storage in INI/JSON/Keyfile format, file watching for live updates, schema-based validation, layered config (system + user + override) |
| LingmoIPC | D-Bus wrappers: service registration, signal emission, method calls, property binding, introspection helpers |
| LingmoPlugin | Plugin system: JSON metadata discovery in standard paths, shared library loading (QLibrary), version checking, dependency resolution, lazy loading, service type registration |
| LingmoService | Service lifecycle: start/stop/restart helpers, systemd integration (sd-bus), readiness notification (sd_notify), watchdog |
| LingmoCrash | Crash handling: signal handlers (SIGSEGV, SIGABRT, etc.), backtrace generation (backward-cpp or libunwind), coredump triggering, crash report generation |
| LingmoLogger | Structured logging: log levels (trace/debug/info/warn/error/fatal), category-based filtering, journald integration, log file rotation |
| LingmoFileUtils | File operations: copy/move/delete with progress, recursive operations, MIME type detection via shared-mime-info, URI handling, path utilities |
| LingmoProcess | Process management: spawning with environment control, output capture, exit code handling, process monitoring, sandbox helpers |
| LingmoSystemInfo | System information: OS release, kernel version, hardware detection (CPU, GPU, memory, disks), desktop environment identification |

**Output Packages**

| Package | Contents |
|---------|----------|
| `liblingmo-core` | Shared library |
| `qml-module-lingmo-core` | QML imports for config and IPC |
| `liblingmo-core-dev` | Headers, CMake config files |

**Public API**

```
C++:
  Lingmo::Config (key-value store with groups)
  Lingmo::Config::group("name")
  Lingmo::ConfigWatcher (monitor key changes)
  Lingmo::DBus (service, interface wrappers)
  Lingmo::Plugin (metadata, loader)
  Lingmo::PluginLoader (discover, load, instantiate)
  Lingmo::Service (lifecycle helpers)
  Lingmo::CrashHandler (install signal handlers)
  Lingmo::Logger (log macros, categories)
  Lingmo::FileUtils (file operations)
  Lingmo::Process (process launch)
  Lingmo::SystemInfo (OS/hardware info)
```

---

### Repository: LingmoShell

**Purpose**: The desktop shell — everything visible on screen except application
windows. One process (lingmo-shell) with multiple components.

**Dependencies**
- Build: Qt 6 (Core, Gui, Qml, Quick, Widgets, DBus), LingmoUI, LingmoCore, KWin::libkwineffects, KWin::libkwinglutils, KWaylandServer, KF6::ConfigQt
- Runtime: Qt 6, LingmoUI, LingmoCore, kwin

**Internal Modules**

| Module | Responsibility |
|--------|----------------|
| **Shell Core** | Main process, applet containment infrastructure, plugin loading, Wayland surface management (wlr-layer-shell), window management coordination with KWin via D-Bus, screen management |
| **Panel** | Configurable panels (position: top/bottom/left/right, length: fill/centered), panel configuration UI, applet hosting |
| **Desktop** | Wallpaper rendering (static, slideshow, animated), desktop icon manager (file-based or app-based), desktop context menu |
| **Overview** | Window overview (thumbnails via ext-foreign-toplevel), workspace switcher, application grid, search integration |
| **Notification Center** | org.freedesktop.Notifications implementation, notification history (journald-backed), notification groups, do-not-disturb, per-app notification settings |
| **App Menu** | Application launcher: categorized grid, search, frequent/recent apps, XDG desktop-file-based |
| **Task Manager** | Window list in panel: icons, titles, grouping, thumbnail previews, window operations (close, minimize, maximize) |
| **Quick Settings** | Expandable panel area: volume, brightness, network, Bluetooth, power profile, night light, do-not-disturb |
| **System Tray** | StatusNotifierItem host: icons, tooltips, context menus, right-to-left ordering |
| **OSD** | On-screen display overlays for volume, brightness, media playback, keyboard layout, airplane mode |
| **Runner** | Alt+F2 dialog: command execution, calculator, file search, web search plugins |
| **Window Decorations** | KWin decoration plugin (C++): titlebar, borders, buttons, shadow, CSD/SSD support |
| **KWin Scripts** | JavaScript scripts: window placement (smart/maximized/centered), focus policy (click-to-focus/focus-follows-mouse), desktop grid switching animation, window snap zones, application-to-desktop assignment |
| **KWin Effects** | Optional C++ effects: application launch feedback, smooth workspace switch, window minimize animation |

**Output Packages**

| Package | Contents |
|---------|----------|
| `lingmo-shell` | Main shell binary |
| `lingmo-shell-applets` | Built-in applets (battery, clock, volume, weather, network status) |
| `lingmo-kwin-decoration` | KWin decoration plugin |
| `lingmo-kwin-scripts` | KWin JavaScript scripts |
| `lingmo-kwin-effects` | Optional KWin effects plugins |

---

### Repository: LingmoSession

**Purpose**: Session lifecycle, display manager integration, screen locking.

**Dependencies**
- Build: Qt 6 (Core, Gui, Qml, Quick, DBus), LingmoUI, LingmoCore, KF6::ConfigQt, PAM, libseccomp (for locker sandbox)
- Runtime: Qt 6, LingmoUI, LingmoCore, kwin, sddm (for DM), PAM

**Internal Modules**

| Module | Responsibility |
|--------|----------------|
| **Session Manager** | Systemd user service: start session in order (daemon → shell → apps), health monitoring (systemd watchdog), session save/restore, logout/shutdown/reboot dialog, crash detection with automatic shell restart |
| **Screen Locker** | Wayland ext-session-lock client: PAM authentication, clock display, media controls, unlock animation, security hardening (sandboxed, seccomp), idle-based locking |
| **User Switcher** | Fast user switching via logind: switch session, multi-seat support |
| **Session Config** | Environment setup: XDG directories, Lingmo environment variables, locale, Xauthority management, autostart (.desktop files via XDG autostart spec) |
| **SDDM Theme** | SDDM login screen theme using LingmoUI components |

**Output Packages**

| Package | Contents |
|---------|----------|
| `lingmo-session` | Session manager binary |
| `lingmo-lockscreen` | Screen locker binary |
| `lingmo-session-config` | Environment files, autostart spec |
| `lingmo-sddm-theme` | SDDM theme files |
| `lingmo-session-services` | systemd service units |

---

### Repository: LingmoDaemon

**Purpose**: All background system services that run as part of the session.

**Dependencies**
- Build: Qt 6 (Core, DBus, Network, Xml), LingmoCore, LingmoUI (for OSD), UPower (client lib), PipeWire (client lib), udisks2 (client lib), KScreen (libkscreen), KF6::ConfigQt, libpulse (or pipewire-qt), libcups, NetworkManager (client lib), BlueZ (client lib)
- Runtime: Qt 6, LingmoCore, LingmoUI, systemd, D-Bus

**Internal Modules (each is a separate binary/service)**

| Module | Responsibility |
|--------|----------------|
| **lingmo-settings-daemon** | Monitors settings changes and applies them. Keyboard layout switching, font configuration, GTK theme sync (via gsettings bridge), XSettings |
| **lingmo-power** | UPower client: battery status, charge thresholds, suspend/hibernate/sleep handling, power profiles (performance/balanced/power-saver), lid switch action, low battery warnings, adaptive brightness |
| **lingmo-display** | KScreen client: monitor hotplug, resolution/refresh rate, scaling, arrangement, night light (blue light filter), color profiles (colord/ICC), Wayland output configuration |
| **lingmo-input** | Input device management: keyboard (repeat rate, layout switching), mouse (speed, acceleration, natural scroll), touchpad (tap-to-click, gestures, disable-while-typing), graphics tablet, touchscreen calibration, input device hotplug |
| **lingmo-audio** | PipeWire client: device enumeration, default sink/source, volume control, mute state, audio profiles (analog/digital/HDMI), application audio streams, spatial audio |
| **lingmo-media** | Media key handler: keyboard media keys → MPRIS player actions (play/pause/next/prev), OSD display |
| **lingmo-thumbnails** | Thumbnail generation service: D-Bus service, parallel generation, caching (via thumbnails spec), extractors for images, videos, PDF, office documents, raw photos |
| **lingmo-time** | NTP/time management: time zone detection (geoclue or /etc/timezone), NTP enable/disable, manual time setting, time auto-sync monitoring |
| **lingmo-printing** | CUPS client: printer discovery, driverless printing (IPP Everywhere), default printer management, job monitoring |
| **lingmo-removables** | udisks2 client: mount/unmount automation, mount location management, system tray integration for removable media |

**Output Packages**

| Package | Contents |
|---------|----------|
| `lingmo-daemon-core` | Common daemon library (shared startup, logging, D-Bus registration) |
| `lingmo-settings-daemon` | Settings daemon binary |
| `lingmo-power` | Power management daemon and OSD client |
| `lingmo-display` | Display management daemon |
| `lingmo-input` | Input device daemon |
| `lingmo-audio` | Audio management daemon |
| `lingmo-media` | Media key handler daemon |
| `lingmo-thumbnails` | Thumbnail generator daemon |
| `lingmo-time` | Time management daemon |
| `lingmo-printing` | Printing daemon |
| `lingmo-removables` | Removable devices daemon |

---

### Repository: LingmoControlCenter

**Purpose**: Central settings application with plugin-based panels.

**Dependencies**
- Build: Qt 6 (Core, Gui, Qml, Quick, Widgets, DBus), LingmoUI, LingmoCore, LingmoDaemon (for some panel DBus interfaces)
- Runtime: Qt 6, LingmoUI, LingmoCore, LingmoDaemon

**Internal Modules**

| Module | Responsibility |
|--------|----------------|
| **Control Center App** | Main window: navigation sidebar, search across all panels, breadcrumb, settings hierarchy, change tracking (apply/reset), keyboard navigation, plugin discovery |
| **Appearance Panel** | Theme switcher, wallpaper selection, font settings, icon/cursor theme selection, accent color, dark/light mode |
| **Display Panel** | Monitor arrangement (drag-and-drop), resolution, refresh rate, scale, orientation, night light settings |
| **Power Panel** | Power profile selection, screen blank timeout, idle behavior, battery charge limit, lid close action |
| **Input Panel** | Mouse speed/acceleration, touchpad gestures, keyboard repeat rate, layout switching, shortcuts manager |
| **Sound Panel** | Output/input device selection, volume, balance, audio profile, application volumes, system sounds on/off |
| **Network Panel** | Wi-Fi (network list, connect, saved networks), Ethernet, VPN (add/configure/remove), proxy settings, hotspot |
| **Bluetooth Panel** | Device list, pairing, connection management, file transfer settings |
| **Printers Panel** | Add printer (auto-detect, IPP, manual), manage print queues, set default |
| **Users Panel** | User list, password change, avatar, account type, auto-login |
| **Date & Time Panel** | Time zone map, NTP toggle, manual time/date, format preferences (12h/24h, date format) |
| **Language Panel** | Language selection, locale format, input method framework (IBus/Fcitx), keyboard layout per language |
| **Default Apps Panel** | Default browser, mail, music player, video player, image viewer, file manager, terminal |
| **Online Accounts Panel** | Account type selection (Google, Nextcloud, Microsoft), OAuth flow, service enable/disable (mail, calendar, contacts, files) |
| **Accessibility Panel** | Screen reader (Orca), large text, high contrast theme, zoom (magnifier), sticky keys, slow keys, bounce keys, mouse keys |
| **Notifications Panel** | Do-not-disturb schedule, per-app notification categories (silent/alert/banner/lock screen), notification history limit |
| **Search Panel** | Search provider order, file indexing on/off, index exclusions |
| **Privacy Panel** | Camera/mic/location permissions, crash reporting consent, usage data collection |
| **Updates Panel** | Check for updates, update categories (system, apps, firmware), update settings |
| **About Panel** | System info: OS name/version, kernel, CPU, memory, graphics, disk, desktop version, hardware model |
| **Removables Panel** | Auto-mount settings, media handling actions (CD/DVD/camera/phone) |
| **Shortcuts Panel** | Custom shortcut list, add/remove/reset, conflict detection, capture keystroke |
| **Firewall Panel** | Firewall zones, port management, application rules, connection blocking |
| **Online Accounts Panel** | Add, manage, remove online accounts; sync scope per account |
| **Session Panel** | Startup applications list (add/remove/enable/disable), desktop session settings |

**Output Packages**

| Package | Contents |
|---------|----------|
| `lingmo-control-center` | Main settings application |
| `lingmo-control-center-panels` | Built-in settings panels as plugins |
| `lingmo-control-center-dev` | Panel plugin development headers and macros |

---

### Repository: LingmoApplications

**Purpose**: Core desktop applications.

**Dependencies**
- Build: Qt 6 (Core, Gui, Qml, Quick, Widgets, DBus, Svg, Network, Concurrent, TextToSpeech, Positioning), LingmoUI, LingmoCore, LingmoSearch (for file manager), LingmoDaemon (for system monitor), KF6::ConfigQt, Poppler (PDF), DjVuLibre, libzip, libarchive, libpipewire (recorder), freetype, fontconfig, PackageKit-Qt6
- Runtime: Qt 6, LingmoUI, LingmoCore, LingmoDaemon

**Internal Modules (each is an application)**

| Application | Responsibility |
|-------------|----------------|
| **Lingmo Files** | File manager: icon/grid/list/column views, split pane, tabs, tree sidebar, places, search (recursive, filtered), rename dialog, properties, bookmarks, network locations (SFTP/SMB/DAV), trash, archive integration, bulk rename, file conflict resolution |
| **Lingmo Edit** | Text editor: syntax highlighting (QSyntaxHighlighter or KSyntaxHighlighting), code folding, line numbers, bracket matching, multiple cursors, find/replace (regex), session save/restore, file change detection |
| **Lingmo Terminal** | Terminal emulator: QML-based terminal widget (QProcess + VT parser or VTE/libvterm), tabs, split panes, profiles (colors, font, transparency), search, hyperlink detection, custom key bindings, true color support, sixel/kitty protocol |
| **Lingmo Image** | Image viewer: thumbnail strip, zoom/pan/fit, fullscreen, slideshow, EXIF display, basic edits (crop, rotate, resize, color adjust), RAW format support |
| **Lingmo Documents** | Document viewer: PDF with Poppler, EPUB with Qt WebEngine or custom renderer, comic books (CBR/CBZ/ZIP), DjVu, continuous/single-page scrolling, table of contents, search, annotations, bookmarks, text selection, printing |
| **Lingmo Calc** | Calculator: basic mode, scientific mode (trig, log, constants), programmer mode (hex/bin/dec, bit operations), currency converter (API-based rates), expression history, unit converter |
| **Lingmo Clock** | Clock: world clocks (add cities, time zone map), alarms (systemd timer integration), timers, stopwatch, calendar (month/week/day views, CalDAV support via online accounts) |
| **Lingmo Capture** | Screenshot + screen recorder: region/window/screen capture, delay timer, annotation (arrows, text, shapes, highlights, blur), sharing (Imgur/local save), screen recording (PipeWire screencast portal, MP4/WebM, audio source selection) |
| **Lingmo Archive** | Archive manager: create (zip/tar/tar.gz/tar.bz2/7z), extract (all formats via libarchive), browse archive contents, incremental extraction, password-protected archives, drag-and-drop |
| **Lingmo Monitor** | System monitor: process list (tree/table, filter, search, kill/renice), CPU/memory/disk/network graphs, filesystem usage, GPU monitoring (NVML/amdgpu), service management (systemd), startup applications |
| **Lingmo Disk** | Disk utility: device list, partition table view, create/delete/resize partitions (util-linux/libblockdev), format (mkfs.*), mount/unmount, SMART data display, disk benchmarks, ISO writing |
| **Lingmo Software** | Software center: AppStream categories, search, featured apps, installed apps view, updates, Flatpak/APT/RPM backends via PackageKit, app ratings, screenshots, app permissions (Flatpak) |
| **Lingmo Fonts** | Font viewer: installed fonts list (grid/list), preview (sample text, all sizes, all styles), font comparison, install font file, font information (metadata, glyphs) |
| **Lingmo Info** | System info: OS, kernel, desktop environment, CPU (model, cores, frequency), memory (total/used), graphics (GPU model, driver, OpenGL version), disks, network interfaces, benchmarks |
| **Lingmo First Run** | First-run wizard: language selection, keyboard layout, privacy settings (crash reporting, location), online accounts setup, welcome tour, wallpaper selection |

**Output Packages**

| Package | Contents |
|---------|----------|
| `lingmo-files` | File manager |
| `lingmo-edit` | Text editor |
| `lingmo-terminal` | Terminal emulator |
| `lingmo-image` | Image viewer |
| `lingmo-documents` | Document viewer |
| `lingmo-calc` | Calculator |
| `lingmo-clock` | Clock/calendar |
| `lingmo-capture` | Screenshot + recorder |
| `lingmo-archive` | Archive manager |
| `lingmo-monitor` | System monitor |
| `lingmo-disk` | Disk utility |
| `lingmo-software` | Software center |
| `lingmo-fonts` | Font viewer |
| `lingmo-info` | System info |
| `lingmo-firstrun` | First-run wizard |
| `lingmo-apps-common` | Shared library (dialogs, widgets used across apps) |

---

### Repository: LingmoNetwork

**Purpose**: Network and Bluetooth management — applet, settings, and background
services.

**Dependencies**
- Build: Qt 6 (Core, Gui, Qml, Quick, DBus, Network), LingmoUI, LingmoCore, NetworkManager-Qt, BlueZ-Qt (or BlueZ client lib)
- Runtime: Qt 6, LingmoUI, LingmoCore, NetworkManager, bluez

**Internal Modules**

| Module | Responsibility |
|--------|----------------|
| **lingmo-nm-applet** | Panel applet: Wi-Fi signal indicator, network list (scan/connect/disconnect), Ethernet status, VPN status, airplane mode toggle, connection details |
| **lingmo-nm-dbus** | D-Bus service wrapping NetworkManager for shell integration: connection state signals, device enumeration, hotspot control |
| **lingmo-bt-applet** | Panel applet: Bluetooth on/off, device list, pairing, connect/disconnect, battery level display |
| **lingmo-bt-dbus** | D-Bus service wrapping BlueZ: device discovery, pairing agent, file transfer (OBEX), browse device files |
| **linvgmo-vpn** | VPN management: OpenVPN/WireGuard connection setup, import/export config files, status monitoring |

**Output Packages**

| Package | Contents |
|---------|----------|
| `lingmo-network-core` | Network management library |
| `lingmo-nm-applet` | Panel network applet |
| `lingmo-bt-applet` | Panel Bluetooth applet |
| `lingmo-vpn` | VPN management tools |

---

### Repository: LingmoSearch

**Purpose**: Full-text and metadata search across files, applications, settings,
and online sources.

**Dependencies**
- Build: Qt 6 (Core, DBus, Concurrent), LingmoCore, Xapian (or Tantivy), libmagic (file type detection), exiv2 (image metadata), taglib (audio metadata), libopenmpt (or similar for document text extraction)
- Runtime: Qt 6, LingmoCore, Xapian

**Internal Modules**

| Module | Responsibility |
|--------|----------------|
| **lingmo-indexer** | Background indexer: filesystem watcher (inotify/fanotify), content extraction for text/pdf/doc/odf/images/audio/video, metadata extraction (EXIF, MP3 tags, document metadata), incremental updates, low-priority I/O (ionice) |
| **lingmo-search-service** | D-Bus search service: accept queries, return ranked results, support for search providers (files, apps, settings, calculator, web, history), result snippets |
| **lingmo-search-engine** | Xapian wrapper: index creation/update, query parser, ranking (BM25/tf-idf), faceted search, filtering (file type, date range, size), fuzzy matching |
| **lingmo-extractors** | Plugin system for content extractors: text (txt), PDF, ODF/DOCX, HTML, images (OCR optional), audio (metadata), video (metadata), archives |
| **lingmo-search-provider** | Plugin system for search result sources: files (from index), applications (XDG desktop files), settings (control center panels), calculator, terminal commands, web search |

**Output Packages**

| Package | Contents |
|---------|----------|
| `lingmo-indexer` | File indexer daemon |
| `lingmo-search-service` | D-Bus search service |
| `lingmo-search-engine` | Shared library for search |
| `lingmo-search-extractors` | Content extractor plugins |
| `lingmo-search-providers` | Built-in search providers |
| `lingmo-search-dev` | Headers for custom extractors/providers |

---

### Repository: LingmoIntegration

**Purpose**: Desktop integration interfaces: portals, authentication, XDG
compliance.

**Dependencies**
- Build: Qt 6 (Core, Gui, Qml, Quick, Widgets, DBus), LingmoUI, LingmoCore, polkit-qt-6 (PolicyKit client), xdg-desktop-portal (headers), flatpak (client lib, optional), AppStream-Qt
- Runtime: Qt 6, LingmoUI, LingmoCore, xdg-desktop-portal, polkitd, flatpak (optional)

**Internal Modules**

| Module | Responsibility |
|--------|----------------|
| **xdg-desktop-portal-lingmo** | xdg-desktop-portal backend implementing: FileChooser (native Lingmo file dialog), Screenshot/RemoteDesktop/ScreenCast (PipeWire/KWin screencast), Wallpaper (Lingmo wallpaper setter), Notification (Lingmo notifications), OpenURI (URI handler dispatch), Print (CUPS dialog), Camera (PipeWire camera), Location (GeoClue), Account (user info), Email/Trash, Background (startup background), Secrets (secret service) |
| **lingmo-polkit-agent** | PolicyKit agent: D-Bus listener for auth requests, LingmoUI-themed authentication dialog, timeout handling, remember-auth |
| **lingmo-defaults** | Default applications manager: XDG MIME apps configuration, MIME type ↔ app associations, scheme handlers (mailto, tel, etc.) |
| **lingmo-xdg** | XDG utilities: desktop file validation, application cache (menu cache), icon lookup fallback |
| **lingmo-calamares** | Calamares installer modules: Lingmo Desktop configuration (theme, default apps, session), user setup integration |
| **lingmo-flatpak** | Flatpak integration: installation/removal, permissions management, update monitoring, portal integration verification |
| **lingmo-appstream** | AppStream: metadata parsing, category management, screenshot downloading, app info caching |

**Output Packages**

| Package | Contents |
|---------|----------|
| `xdg-desktop-portal-lingmo` | Portal backend D-Bus service |
| `lingmo-polkit-agent` | PolicyKit agent |
| `lingmo-defaults` | Default app manager |
| `lingmo-xdg-utils` | XDG utilities library |
| `lingmo-calamares` | Calamares modules |
| `lingmo-flatpak` | Flatpak integration |
| `lingmo-appstream` | AppStream utilities |

---

### Repository: LingmoAssets

**Purpose**: All visual and audio assets for the desktop.

**Dependencies**
- Build: None (data files only, some SVGs may need rendering)
- Runtime: None (loaded by LingmoUI, shell, etc.)

**Contents**

| Directory | Contents |
|-----------|----------|
| `themes/lingmo-default` | Default theme: JSON token files (colors, typography, metrics, radii, shadows, animations), QML theme components (QtQuick imports for custom styling) |
| `themes/lingmo-dark` | Dark variant theme tokens |
| `themes/lingmo-highcontrast` | High contrast accessibility theme |
| `icons/` | Complete icon theme: apps (application icons), actions (toolbar/menu), devices (computers, displays), places (folders, drives), mimetypes (file type icons), emblems (status overlays), status (battery, network, notifications), categories |
| `cursors/` | Cursor theme: all standard XCursor cursors, HiDPI variants, animated cursors |
| `sounds/` | Sound theme: startup, shutdown, login, logout, notification (info, warning, error, message), system alerts, window close, volume change, screenshot, device connect/disconnect |
| `wallpapers/` | Default static wallpapers, live wallpapers (Shaders/QML), slideshow configurations, contest-winner wallpapers |
| `sddm/` | SDDM theme: QML components, background images, configuration |
| `plymouth/` | Plymouth boot splash: theme configuration, script, images, animations |
| `gtk/` | GTK3/4 themes: CSS matching Lingmo design tokens, metacity/metacity-theme (legacy), window manager definitions |

**Output Packages**

| Package | Contents |
|---------|----------|
| `lingmo-theme-default` | Default theme files (JSON tokens + QML) |
| `lingmo-theme-dark` | Dark theme variant |
| `lingmo-theme-highcontrast` | High contrast theme |
| `lingmo-icon-theme` | Icon theme |
| `lingmo-cursor-theme` | Cursor theme |
| `lingmo-sound-theme` | Sound theme |
| `lingmo-wallpapers` | Default wallpapers |
| `lingmo-sddm-theme` | SDDM theme |
| `lingmo-plymouth-theme` | Plymouth theme |
| `lingmo-gtk-theme` | GTK3/4 themes |

---

### Repository: LingmoSDK

**Purpose**: Developer tools, documentation, and templates for third-party
Lingmo application development.

**Dependencies**
- Build: LingmoUI, LingmoCore (headres)
- Runtime: None (documentation and tools)

**Contents**

| Item | Description |
|------|-------------|
| Public API Reference | Generated API docs (Doxygen + QDoc) for LingmoUI and LingmoCore |
| QML Documentation | QML types documentation for all LingmoUI modules |
| Plugin Development Guide | How to write shell applets, control center panels, search providers, content extractors |
| Application Templates | CMake project templates for Qt Widgets and QML Lingmo apps |
| CMake Macros | `lingmo_add_plugin()`, `lingmo_add_applet()`, `lingmo_add_settings_panel()` — reusable CMake functions |
| UI Inspector | Standalone tool: inspect LingmoUI properties of running applications, preview theme changes in real-time, export theme snapshots |
| AppStream Generator | CLI tool: generate AppStream XML metadata for Lingmo applications |
| CI Templates | Reusable GitHub Actions workflows for Lingmo app projects (build, lint, test, package) |
| Code Examples | Full working examples: panel applet, settings panel, LingmoUI application (Widgets and QML), search provider |

**Output Packages**

| Package | Contents |
|---------|----------|
| `lingmo-sdk` | Development tools and documentation |
| `lingmo-sdk-templates` | Project templates and examples |
| `lingmo-sdk-cmake` | CMake macros and modules |
| `lingmo-ui-inspector` | UI inspector tool |

---

## Phase 5: Dependency Graph

### Layer Hierarchy

```
                        ┌─────────────────────────────────────┐
                        │          LingmoApplications          │
                        │  (files, edit, terminal, viewer,     │
                        │   documents, calc, clock, capture,   │
                        │   archive, monitor, disk, software,  │
                        │   fonts, info, firstrun)             │
                        └──────────────────┬──────────────────┘
                                           │
                        ┌──────────────────▼──────────────────┐
                        │           LingmoControlCenter         │
                        │  (settings app + plugin panels)      │
                        └──────────────────┬──────────────────┘
                                           │
            ┌──────────────────────────────┼──────────────────────────────┐
            │                              │                              │
┌───────────▼──────────┐   ┌───────────────▼───────────┐   ┌─────────────▼───────────┐
│     LingmoShell      │   │       LingmoDaemon        │   │    LingmoIntegration    │
│  (panel, desktop,    │   │  (power, display, input    │   │  (portals, polkit, xdg) │
│   overview, notifs,  │   │   audio, thumbnails, etc.) │   │                         │
│   decorations, etc.) │   │                            │   │                         │
└───────────┬──────────┘   └───────────────┬────────────┘   └─────────────┬───────────┘
            │                              │                              │
            │              ┌───────────────┼──────────────┐               │
            │              │               │              │               │
┌───────────▼──────────┐   │   ┌───────────▼──────────┐  │  ┌─────────────▼───────────┐
│    LingmoSession     │   │   │    LingmoNetwork     │  │  │      LingmoSearch       │
│ (session manager,    │   │   │ (networks, bluetooth) │  │  │  (indexer, search svc,  │
│  locker, greeter)    │   │   │                      │  │  │   engine, extractors)   │
└───────────┬──────────┘   │   └──────────────────────┘  │  └─────────────────────────┘
            │              │                              │
            └──────────────┼──────────────────────────────┘
                           │
            ┌──────────────▼──────────────┐
            │         LingmoCore          │
            │  (config, ipc, plugin,       │
            │   service, crash, logger,    │
            │   fileutils, process)        │
            └──────────────┬──────────────┘
                           │
            ┌──────────────▼──────────────┐
            │          LingmoUI            │
            │  (theme, style, controls,    │
            │   window, effects, icon,     │
            │   animation)                 │
            └──────────────┬──────────────┘
                           │
            ┌──────────────▼──────────────┐
            │          Qt 6                │
            │  (Core, Gui, Qml, Quick,    │
            │   Widgets, Svg, DBus, etc.)  │
            └──────────────┬──────────────┘
                           │
            ┌──────────────▼──────────────┐
            │          KWin                │
            │  (compositor, decoration     │
            │   API, scripting API,        │
            │   Wayland protocols)         │
            └─────────────────────────────┘
```

### Data-flow-only Dependencies

These repositories depend on data from LingmoAssets at runtime, not at compile time:

```
LingmoUI           ──loads──▶  LingmoAssets (theme tokens)
LingmoShell        ──loads──▶  LingmoAssets (wallpapers, icons)
LingmoSession      ──loads──▶  LingmoAssets (SDDM theme, lock screen theme)
LingmoControlCenter──loads──▶  LingmoAssets (wallpaper previews, icon previews)
LingmoApplications ──loads──▶  LingmoAssets (icons, wallpapers)
```

### Runtime Dependency Table

| Component | Depends on (runtime) |
|-----------|---------------------|
| **LingmoUI** | Qt 6 |
| **LingmoCore** | Qt 6, LingmoUI |
| **LingmoAssets** | None (data only) |
| **LingmoSession** | Qt 6, LingmoUI, LingmoCore, KWin, PAM, SDDM |
| **LingmoShell** | Qt 6, LingmoUI, LingmoCore, KWin (decoration plugin), KWayland |
| **LingmoDaemon** | Qt 6, LingmoCore, LingmoUI (for OSD), systemd, PipeWire, UPower, udisks2, colord, CUPS, BlueZ |
| **LingmoNetwork** | Qt 6, LingmoUI, LingmoCore, NetworkManager, BlueZ |
| **LingmoSearch** | Qt 6, LingmoCore, Xapian |
| **LingmoIntegration** | Qt 6, LingmoUI, LingmoCore, xdg-desktop-portal, polkitd, Flatpak (optional) |
| **LingmoControlCenter** | Qt 6, LingmoUI, LingmoCore, LingmoDaemon (DBus interfaces) |
| **LingmoApplications** | Qt 6, LingmoUI, LingmoCore, LingmoSearch (file manager), LingmoDaemon (system monitor) |
| **LingmoSDK** | LingmoUI (headers), LingmoCore (headers) |

### System-Level Dependencies

These are not Lingmo repositories but are required at runtime:

| Dependency | Purpose | Required |
|-----------|---------|----------|
| **KWin** | Compositor / Wayland window manager | Yes |
| **systemd** | System and user service management, logind, journald | Yes |
| **D-Bus** | Inter-process communication | Yes |
| **SDDM** | Display manager (login screen) | Yes |
| **NetworkManager** | Network configuration | Yes |
| **BlueZ** | Bluetooth stack | Optional |
| **PipeWire** | Audio and video (screen capture) | Yes |
| **Upower** | Power management information | Yes |
| **udisks2** | Disk and removable media management | Yes |
| **colord** | Color management (ICC profiles) | Optional |
| **CUPS** | Printing system | Optional |
| **GeoClue** | Location services | Optional |
| **PackageKit** | Package management backend | Yes |
| **Flatpak** | Application sandboxing | Optional |
| **xdg-desktop-portal** | Desktop portal infrastructure | Yes |
| **polkitd** | PolicyKit authorization | Yes |
| **PAM** | Pluggable authentication modules | Yes |
| **Xapian** | Full-text search engine | Yes |
| **AppStream** | Application metadata | Yes |
| **Calamares** | System installer (optional integration) | Optional |

### Circular Dependency Prevention

All dependency edges follow a strict downward direction:

```
LingmoSDK → LingmoUI, LingmoCore
LingmoApplications → LingmoUI, LingmoCore, LingmoSearch, LingmoDaemon
LingmoControlCenter → LingmoUI, LingmoCore, LingmoDaemon, LingmoNetwork
LingmoShell → LingmoUI, LingmoCore, LingmoSession
LingmoIntegration → LingmoUI, LingmoCore
LingmoSearch → LingmoUI, LingmoCore
LingmoNetwork → LingmoUI, LingmoCore
LingmoDaemon → LingmoUI, LingmoCore, LingmoNetwork
LingmoSession → LingmoUI, LingmoCore
LingmoCore → LingmoUI
LingmoUI → Qt 6
```

No repository depends on a repository that depends on it. The only possible
circular concern would be if LingmoDaemon depended on LingmoShell (for OSD
display), but this is resolved by:
1. LingmoDaemon emits D-Bus signals for OSD
2. LingmoShell listens for these signals and displays the OSD
3. No compile-time or link-time dependency from LingmoDaemon to LingmoShell

Similarly, LingmoControlCenter does NOT depend on LingmoSession. The session
manager runs independently. The control center may query session state via
D-Bus, but this is a runtime interface, not a build dependency.

---

## Package Grouping for Distribution

### Meta-Packages

| Package | Contents |
|---------|----------|
| `lingmo-desktop` | Full desktop environment (all repos + recommended system deps) |
| `lingmo-desktop-core` | Minimal: LingmoUI, LingmoCore, LingmoSession, LingmoShell, LingmoDaemon, LingmoControlCenter |
| `lingmo-ui` | UI framework only (for third-party apps): LingmoUI + LingmoAssets theme |

### Debian Packaging Structure

Each repository produces `.deb` packages. The meta-packages would be in a
separate `lingmo-meta` repository or in distribution-specific packaging repos.

```
lingmo-desktop:
  Depends: lingmo-shell, lingmo-session, lingmo-daemon-core,
           lingmo-control-center, lingmo-files, lingmo-terminal,
           lingmo-edit, lingmo-image, lingmo-documents, lingmo-calc,
           lingmo-clock, lingmo-capture, lingmo-archive, lingmo-monitor,
           lingmo-software, lingmo-fonts, lingmo-info,
           lingmo-theme-default, lingmo-icon-theme, lingmo-cursor-theme,
           lingmo-wallpapers, lingmo-sddm-theme, lingmo-gtk-theme,
           lingmo-power, lingmo-display, lingmo-audio,
           xdg-desktop-portal-lingmo, lingmo-polkit-agent,
           lingmo-nm-applet, lingmo-bt-applet,
           sddm, kwin, pipewire, network-manager, upower,
           udisks2, packagekit, polkitd
```

### RPM Packaging Structure

Similar but using RPM spec files in each repository.

---

## CI/CD Strategy (GitHub Actions)

Each repository has:

```
.github/workflows/
├── build.yml          # Build on amd64 + arm64 for Debian + Fedora
├── test.yml           # Run unit tests
├── package.yml        # Generate .deb and .rpm packages
├── lint.yml           # Lint (clang-tidy, clang-format, qmllint)
└── release.yml        # Future: tag-based release automation
```

Workflow structure:

```
build.yml:
  matrix: { arch: [amd64, arm64], distro: [debian, fedora] }
  steps:
    - Checkout + submodules
    - Install build dependencies (distro-specific)
    - CMake configure
    - CMake build
    - Upload build artifacts

test.yml:
  steps:
    - Build + run ctest
    - Upload test results

package.yml:
  steps:
    - Build
    - cpack (Debian: .deb, Fedora: .rpm)
    - Upload packages as artifacts

release.yml:
  on: tag 'v*'
  steps:
    - Build for all architectures
    - Generate packages
    - Create GitHub Release
    - Upload packages
```

---

## Summary

Lingmo Desktop consists of **12 repositories**:

| # | Repository | Lines of Code (est.) | Packages | Primary Language |
|---|-----------|---------------------|----------|-----------------|
| 1 | LingmoUI | ~80K | 6 | C++, QML |
| 2 | LingmoCore | ~40K | 3 | C++ |
| 3 | LingmoShell | ~100K | 4 | C++, QML, JS |
| 4 | LingmoSession | ~30K | 5 | C++, QML |
| 5 | LingmoDaemon | ~80K | 10 | C++ |
| 6 | LingmoControlCenter | ~80K | 3 | C++, QML |
| 7 | LingmoApplications | ~200K | 16 | C++, QML |
| 8 | LingmoNetwork | ~30K | 4 | C++, QML |
| 9 | LingmoSearch | ~40K | 6 | C++ |
| 10 | LingmoIntegration | ~50K | 7 | C++ |
| 11 | LingmoAssets | ~200MB | 10 | Data, QML, CSS, SVG |
| 12 | LingmoSDK | ~20K | 4 | C++, QML, CMake |

**Total: ~730K lines of code + assets across 12 repositories.**

### Key Architectural Decisions

1. **LingmoUI as standalone framework**: Third-party developers adopt only
   LingmoUI + Qt 6, giving apps the Lingmo look without desktop infrastructure.

2. **Unified theming (not dual implementations)**: Unlike Deepin DTK, LingmoUI
   uses a single token system. Qt Widgets render via a QProxyStyle that reads
   these tokens. QML controls read the same tokens. Both paths produce
   identical visual output.

3. **KWin as compositor (not a fork)**: KWin is used via its plugin system
   (decorations, scripts, effects). No fork means we benefit from upstream
   development. This is the same strategy as Deepin but without the custom
   dde-kwin fork.

4. **Medium-sized repositories**: Each repo represents a functional domain
   (UI, shell, session, services, apps, search, network, integration). Avoids
   KDE's over-fragmentation while maintaining modularity.

5. **Plugin-first architecture**: Shell applets, settings panels, search
   providers, content extractors — all use the same LingmoPlugin system from
   LingmoCore. A single plugin mechanism across the entire desktop.

6. **No circular dependencies**: Strict layered architecture ensures repos
   can be built, tested, and released independently.

7. **All repos independently releasable**: Each has CMakeLists.txt, debian/,
   rpm/, and CI. A release can update just one component (e.g., file manager)
   without rebuilding everything.
