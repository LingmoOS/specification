# Lingmo Desktop — Technical Architecture

> Version 3.0 — Production Architecture
>
> This document describes how Lingmo Desktop actually works at the system level.
> Every design decision is justified. No generic descriptions.

---

## Table of Contents

1. [Layered Architecture](#1-layered-architecture)
2. [Runtime Process Model](#2-runtime-process-model)
3. [IPC Design](#3-ipc-design)
4. [Plugin System](#4-plugin-system)
5. [Theme System](#5-theme-system)
6. [Application Framework](#6-application-framework)
7. [Dependency Rules](#7-dependency-rules)
8. [Versioning](#8-versioning)
9. [Build System](#9-build-system)
10. [Testing](#10-testing)
11. [Release Engineering](#11-release-engineering)
12. [Long-term Evolution](#12-long-term-evolution)

---

## 1. Layered Architecture

### Layer Diagram

```
 ┌──────────────────────────────────────────────────────────────────┐
 │                                                                  │
 │                     LINGMO APPLICATIONS                          │
 │  file-manager  text-editor  terminal  image-viewer  document     │
 │  utilities  capture  archive  system-tools  software  info       │
 │  control-center  apps-common                                      │
 │                                                                  │
 └──────────────────────────────┬───────────────────────────────────┘
                                │
 ┌──────────────────────────────▼───────────────────────────────────┐
 │                     LINGMO SHELL & SERVICES                      │
 │                                                                  │
 │  ┌──────────────┐  ┌─────────────────┐  ┌────────────────────┐  │
 │  │  Shell       │  │  Daemons         │  │  Integration       │  │
 │  │              │  │                  │  │                    │  │
 │  │ shell-core   │  │ settings-daemon  │  │ portals            │  │
 │  │ surfaces     │  │ power-manager    │  │ polkit             │  │
 │  │ applets      │  │ display          │  │ flatpak            │  │
 │  │ notification │  │ input            │  │                    │  │
 │  │ osd          │  │ audio            │  └────────────────────┘  │
 │  │ runner       │  │ thumbnails       │                          │
 │  │ decoration   │  │ print            │  ┌────────────────────┐  │
 │  │ scripts      │  │ network          │  │  Search            │  │
 │  │              │  │ bluetooth        │  │  Indexer           │  │
 │  │              │  │                  │  │  Search Service    │  │
 │  └──────────────┘  └─────────────────┘  │  Providers         │  │
 │                                          └────────────────────┘  │
 └──────────────────────────────┬───────────────────────────────────┘
                                │
 ┌──────────────────────────────▼───────────────────────────────────┐
 │                     LINGMO CORE LIBRARIES                        │
 │                                                                  │
 │  ┌────────────┐  ┌──────────┐  ┌────────┐  ┌──────────────────┐ │
 │  │ lingmo-theme│  │ lingmo-  │  │ lingmo- │  │ lingmo-utils     │ │
 │  │            │  │ config   │  │ plugin  │  │                  │ │
 │  └────────────┘  └──────────┘  └────────┘  └──────────────────┘ │
 │  ┌────────────┐  ┌──────────┐  ┌────────┐  ┌──────────────────┐ │
 │  │ lingmo-style│  │ lingmo-  │  │ lingmo- │  │ lingmo-ipc       │ │
 │  │ (QStyle)   │  │ controls │  │ effects │  │                  │ │
 │  └────────────┘  └──────────┘  └────────┘  └──────────────────┘ │
 │  ┌────────────┐  ┌──────────┐                                    │
 │  │ lingmo-icon│  │ lingmo-  │                                    │
 │  │            │  │ window   │                                    │
 │  └────────────┘  └──────────┘                                    │
 └──────────────────────────────┬───────────────────────────────────┘
                                │
 ┌──────────────────────────────▼───────────────────────────────────┐
 │                     Qt 6 FRAMEWORKS                              │
 │  Core  Gui  Widgets  Qml  Quick  DBus  Svg  Network  Concurrent │
 │  WaylandClient  WaylandServer  TextToSpeech                      │
 └──────────────────────────────┬───────────────────────────────────┘
                                │
 ┌──────────────────────────────▼───────────────────────────────────┐
 │                     KWIN (COMPOSITOR)                            │
 │                                                                  │
 │  Decoration API  Scripting API  Effect API  Wayland Protocols   │
 │                                                                  │
 └──────────────────────────────┬───────────────────────────────────┘
                                │
 ┌──────────────────────────────▼───────────────────────────────────┐
 │                     SYSTEM LAYER                                 │
 │                                                                  │
 │  systemd  D-Bus  Wayland  X11  PipeWire  NetworkManager  BlueZ  │
 │  UPower  udisks2  CUPS  colord  GeoClue  PackageKit  polkitd    │
 │                                                                  │
 └──────────────────────────────────────────────────────────────────┘
```

### Layer Definitions

#### Layer 0: System Layer

The operating system services that Lingmo Desktop depends on but does not control.

| Service | Purpose | Required | Integration Point |
|---------|---------|----------|-------------------|
| systemd | Service management, logind, journald | Yes | sd-bus API, unit files |
| D-Bus | Inter-process communication | Yes | Session bus + system bus |
| Wayland | Display protocol (KWin as compositor) | Yes | wlr-layer-shell, xdg-shell |
| X11 | Legacy display protocol (via XWayland) | Optional | XWayland (KWin manages) |
| PipeWire | Audio + video capture | Yes | libpipewire, pw-dbus |
| NetworkManager | Network configuration | Yes | libnm D-Bus API |
| BlueZ | Bluetooth stack | Optional | org.bluez D-Bus API |
| UPower | Power management information | Yes | org.freedesktop.UPower |
| udisks2 | Disk management | Yes | org.freedesktop.UDisks2 |
| CUPS | Printing | Optional | libcups, IPP |
| colord | Color management | Optional | org.freedesktop.ColorManager |
| GeoClue | Location services | Optional | org.freedesktop.GeoClue2 |
| PackageKit | Package management | Yes | libpackagekit-qt6 |
| polkitd | Authorization | Yes | org.freedesktop.PolicyKit1 |

#### Layer 1: KWin (Compositor)

KWin is the compositor. Lingmo Desktop does NOT modify KWin source code.
All integration is through documented plugin interfaces:

- **Decoration plugin** (`lingmo-decoration`): Window titlebar + borders
- **Scripting API** (`lingmo-shell-scripts`): Window behavior JavaScript
- **Effect plugins** (future): Visual effects (loaded by KWin)
- **D-Bus API**: Window management, virtual desktops, screen info
- **KWayland protocols**: wlr-layer-shell (panels), ext-session-lock (lock screen),
  zwlr-screencopy, ext-foreign-toplevel

#### Layer 2: Qt 6 Frameworks

Standard Qt 6 modules. LingmoUI adds style and controls on top of Qt,
it does not replace or modify Qt internals.

#### Layer 3: Lingmo Core Libraries

These are the foundational libraries that everything else depends on.

| Library | Dependency Type | Consumers |
|---------|----------------|-----------|
| lingmo-theme | Shared library (.so) | All processes |
| lingmo-config | Shared library (.so) | All processes |
| lingmo-plugin | Shared library (.so) | Shell, daemons, apps |
| lingmo-ipc | Shared library (.so) | Daemons, shell, apps |
| lingmo-utils | Shared library (.so) | All processes |
| lingmo-style | QStyle plugin | Widgets apps |
| lingmo-controls | QML module | QML apps |
| lingmo-effects | Shared library + QML | Shell, apps |
| lingmo-icon | Shared library | Shell, apps |
| lingmo-window | Shared library + QML | Apps |

#### Layer 4: Lingmo Services & Shell

All running processes. This layer is where the desktop environment
actually operates. See Section 2 for process details.

#### Layer 5: Lingmo Applications

Core applications. Each is a standalone process. See v2 architecture
for the complete list (12 repos).

### Layer Rules

1. Each layer may only depend on the same or lower layers.
2. No layer may depend on KWin directly except through the documented
   plugin interfaces (decoration, scripts, effects).
3. System services (daemons) must NOT link against any UI library
   (neither Qt Widgets nor Qt Quick). They communicate with UI via IPC.
4. Applications must NOT link against shell libraries.
5. Libraries in Layer 3 must NOT link against libraries in Layer 4.
6. The theme library (lingmo-theme) must NOT depend on Qt Quick or Qt Widgets.
   It depends only on Qt Core and Qt Gui.

---

## 2. Runtime Process Model

### Process Map

```
 ┌──────────────────┐     ┌──────────────────┐     ┌──────────────────┐
 │  Session Manager  │────▶│  KWin             │     │  Screen Locker   │
 │  (lingmo-session)  │     │  (kwin_wayland)   │     │  (lingmo-lockscreen)
 │  PID: 1 in user   │     │  PID: fixed       │◀────│  ext-session-lock│
 │  slice            │     │  display server   │     │  on lock         │
 └──────────────────┘     └──────────────────┘     └──────────────────┘
         │                                                        ▲
         │ starts                                                  │
         ▼                                                        │
 ┌──────────────────┐     ┌──────────────────┐     ┌──────────────────┐
 │  Settings Daemon  │     │  Power Manager    │     │  Display Manager │
 │  (lingmo-settings)│     │  (lingmo-power)   │     │  (lingmo-display) │
 │  always-running   │     │  always-running   │     │  always-running  │
 └──────────────────┘     └──────────────────┘     └──────────────────┘
         │                                                       
         │ starts after daemons are ready                        
         ▼                                                       
 ┌──────────────────┐     ┌──────────────────┐     ┌──────────────────┐
 │  Shell Core       │────▶│  Shell Surfaces   │     │  Notification    │
 │  (lingmo-core)    │     │  (lingmo-panel)    │     │  (lingmo-notify)  │
 │  no rendering     │     │  (lingmo-desktop)  │     │                  │
 └──────────────────┘     │  (lingmo-overview)  │     └──────────────────┘
                          └──────────────────┘             ▲
                                                            │
 ┌──────────────────┐     ┌──────────────────┐              │
 │  Audio Manager    │     │  Input Manager    │             │
 │  (lingmo-audio)   │────▶│  (lingmo-input)   │────┐        │
 └──────────────────┘     └──────────────────┘    │        │
                                                   │        │
 ┌──────────────────┐     ┌──────────────────┐    │        │
 │  OSD              │     │  Runner           │    │        │
 │  (lingmo-osd)     │     │  (lingmo-runner)  │    │        │
 │  dbus-activated   │     │  dbus-activated   │    │        │
 └──────────────────┘     └──────────────────┘    │        │
                                                   │        │
 ┌──────────────────┐     ┌──────────────────┐    │        │
 │  Network          │     │  Bluetooth        │    │        │
 │  (lingmo-network) │     │  (lingmo-bluetooth)│    │        │
 └──────────────────┘     └──────────────────┘    │        │
                                                   ▼        ▼
 ┌──────────────────┐     ┌──────────────────┐  ┌──────────────────┐
 │  Print Service    │     │  Thumbnails       │  │  Search Indexer  │
 │  (lingmo-print)   │     │  (lingmo-thumbs)   │  │  (lingmo-indexer)│
 │  dbus-activated   │     │  dbus-activated   │  │  always-running  │
 └──────────────────┘     └──────────────────┘  └──────────────────┘

 ┌──────────────────┐     ┌──────────────────┐
 │  Portal Backend   │     │  PolKit Agent     │
 │  (lingmo-portals)  │     │  (lingmo-polkit)  │
 │  dbus-activated   │     │  dbus-activated   │
 └──────────────────┘     └──────────────────┘
```

### Process Details

#### 2.1 Session Manager

| Property | Value |
|----------|-------|
| Executable | `/usr/libexec/lingmo-session` |
| systemd service | `lingmo-session.service` (user) |
| Type | `notify` (sd_notify readiness) |
| Startup | Started by display manager (SDDM) via `lingmo-session.target` |
| Lifetime | Entire user session (login → logout) |
| Restart policy | `always` (restart-delay: 2s, max-retries: 5, then fallback to failsafe) |
| Crash recovery | On crash: (1) retry 5 times with 2s delay, (2) start failsafe session (only xterm + wm), (3) show recovery dialog |
| Memory model | Owns: session state (environment, autostart list, component health). Shares: nothing via library. Communicates via D-Bus |
| Dependencies | systemd (logind), KWin (starts it), Lingmo IPC library |

Startup sequence:

```
1. SDDM authenticates user
2. SDDM starts `lingmo-session.target` (systemd unit)
3. lingmo-session begins:
   a. Set up environment (XDG_* variables, LINGMO_* variables, locale)
   b. Start KWin if not running (`kwin_wayland --replace`)
   c. Wait for KWin readiness (D-Bus: org.kde.KWin / ready)
   d. Start core daemons in order (settings-daemon, power, display, input, audio)
   e. Wait for daemon readiness (each sd_notify)
   f. Start shell core (lingmo-shell-core)
   g. Wait for shell core readiness
   h. Start shell surfaces (panel, desktop)
   i. Start notification service, search indexer
   j. Launch autostart applications (XDG autostart + Lingmo autostart)
   k. If first login: launch First Run Wizard
   l. Signal readiness (sd_notify READY=1)
4. Session is now active
```

Shutdown sequence:

```
1. User logs out / system shutdown requested
2. lingmo-session sends SIGTERM to autostart apps (graceful: 5s timeout)
3. lingmo-session sends SIGTERM to shell surfaces → notification
4. lingmo-session sends SIGTERM to shell core
5. lingmo-session sends SIGTERM to daemons (reverse order)
6. lingmo-session stops KWin
7. lingmo-session signals systemd (stop complete)
```

#### 2.2 Shell Core

| Property | Value |
|----------|-------|
| Executable | `/usr/libexec/lingmo-shell-core` |
| systemd service | `lingmo-shell-core.service` (user) |
| Type | `notify` |
| Startup | Started by lingmo-session after daemons are ready |
| Lifetime | Entire user session |
| Restart policy | `on-failure` (restart-delay: 1s). Shell surfaces reconnect on restart |
| Crash recovery | Shell core crash → surfaces lose IPC → enter "broken" mode (show "shell recovering" overlay) → core restarts (≤1s) → surfaces reconnect → restoration complete |
| Memory model | Owns: window state (position, workspace), applet configuration, input focus state, virtual desktop state, KWin script state. Shares: nothing (all state communicated via IPC) |

The shell core is the only process that communicates directly with KWin via D-Bus.
All other processes (panel, desktop, overview) route KWin-related operations
through the shell core.

Internal architecture:

```
shell-core/
├── main.cpp
├── core/
│   ├── SessionManager.cpp      # Coordinates with lingmo-session
│   ├── WindowManager.cpp        # KWin D-Bus wrapper
│   ├── VirtualDesktopManager.cpp
│   ├── FocusManager.cpp
│   └── ScreenManager.cpp        # Multi-monitor coordination
├── ipc/
│   ├── IPCServer.cpp            # Local socket server for surfaces
│   ├── DBusService.cpp          # org.lingmo.Shell D-Bus interface
│   └── Protocol.cpp             # Protocol definition for IPC
├── plugins/
│   ├── ScriptManager.cpp        # Load/unload KWin scripts
│   └── EffectManager.cpp        # Load/unload KWin effects
└── applets/
    └── AppletRegistry.cpp       # Known applet types → surface assignment
```

#### 2.3 Shell Surfaces (Panel, Desktop, Overview)

| Property | Value |
|----------|-------|
| Executable | `/usr/libexec/lingmo-panel`, `/usr/libexec/lingmo-desktop`, `/usr/libexec/lingmo-overview` |
| systemd service | `lingmo-panel@.service`, `lingmo-desktop.service`, `lingmo-overview.service` (all user) |
| Type | `simple` |
| Startup | Started by shell core after IPC connection established |
| Lifetime | Shell surface: entire session. Panel: per-monitor instance. Desktop: one per session. Overview: on-demand (when activated) |
| Restart policy | Panel: `on-failure` (restart-delay: 500ms). Desktop: `on-failure`. Overview: no restart (launched on demand) |
| Crash recovery | Panel crashes → loss of panel UI → automatic restart ≤500ms → applet state restored from IPC cache. Desktop crashes → wallpapers disappear → restart → icons reloaded. Overview crash → overview closed → relaunched on next activation |

Each surface process connects to the shell core via a Unix socket for IPC.
The protocol is a simple message-passing format (CBOR or flatbuffers) over
the socket.

```
shell-core ──Socket──▶ lingmo-panel (monitor 0)
shell-core ──Socket──▶ lingmo-panel (monitor 1) 
shell-core ──Socket──▶ lingmo-desktop
shell-core ──Socket──▶ lingmo-overview (when active)
```

Surface processes load applets as shared library plugins via `lingmo-plugin`.

#### 2.4 Notification Service

| Property | Value |
|----------|-------|
| Executable | `/usr/libexec/lingmo-notification` |
| systemd service | `lingmo-notification.service` (user) |
| Type | `dbus` (D-Bus activation) |
| Startup | D-Bus activated on first notification request |
| Lifetime | As long as notifications exist. After 30s idle, exits (auto-shutdown). D-Bus reactivates on next notification |
| Restart policy | `on-failure` |
| Memory model | Owns: notification history (journald-backed), per-app settings, do-not-disturb state. Shares: notification data via D-Bus (org.freedesktop.Notifications) |

Implements `org.freedesktop.Notifications` on the session bus.

#### 2.5 OSD

| Property | Value |
|----------|-------|
| Executable | `/usr/libexec/lingmo-osd` |
| systemd service | D-Bus activated (no explicit unit) |
| Type | `dbus` |
| Startup | D-Bus activation on `org.lingmo.OSD.Show()` |
| Lifetime | ≤ 10 seconds (shows OSD, then auto-exits) |
| Restart policy | Not needed (per-invocation process) |
| Crash recovery | OSD crash → user doesn't see feedback → next OSD request creates new process (no recovery needed) |

The OSD process receives a JSON payload on D-Bus containing:
- Type (volume/brightness/media/airplane/keyboard/notifications)
- Value (0-100 for sliders, or action string for media)
- Source display (integer, for multi-monitor)

#### 2.6 Runner

| Property | Value |
|----------|-------|
| Executable | `/usr/libexec/lingmo-runner` |
| systemd service | D-Bus activated |
| Type | `dbus` |
| Startup | D-Bus activation on Alt+F2 (keyboard shortcut invokes `org.lingmo.Runner.Show()`) |
| Lifetime | As long as the dialog is open. Exits when dialog is dismissed |
| Restart policy | Not needed |
| Crash recovery | If crash while dialog is open → dialog disappears → user presses Alt+F2 again → new process |

#### 2.7 Settings Daemon

| Property | Value |
|----------|-------|
| Executable | `/usr/libexec/lingmo-settings-daemon` |
| systemd service | `lingmo-settings-daemon.service` (user) |
| Type | `notify` |
| Startup | Started by lingmo-session (first daemon to start) |
| Lifetime | Entire session |
| Restart policy | `on-failure` (restart-delay: 1s, max-retries: 10, then `fail`) |
| Crash recovery | Full restart. Watches for changes (inotify on config dirs) and re-applies. State is in config files, not in memory |

Submodules (all in one process, no sub-processes):

```
settings-daemon/
├── main.cpp
├── core/
│   ├── ConfigMonitor.cpp       # inotify on ~/.config/lingmo/ and /etc/lingmo/
│   ├── XSettingsManager.cpp    # XSettings for XWayland apps
│   ├── GSettingsBridge.cpp     # Sync Lingmo config → gsettings
│   └── EnvManager.cpp          # Manage ~/.config/environment.d/
├── keyboards/
│   ├── KeyboardManager.cpp     # Layout switching, repeat rate
│   └── LayoutDBus.cpp          # org.lingmo.SettingsDaemon.Keyboard
├── fonts/
│   └── FontManager.cpp         # Fontconfig integration
├── gtk/
│   └── GTKThemeBridge.cpp      # Generate GTK settings.ini
├── scaling/
│   └── ScalingManager.cpp      # Qt/Wayland scaling environment
└── removable/
    └── RemovableManager.cpp    # udisks2 monitor, mount/unmount
```

#### 2.8 Power Manager

| Property | Value |
|----------|-------|
| Executable | `/usr/libexec/lingmo-power` |
| systemd service | `lingmo-power.service` (user) |
| Type | `notify` |
| Startup | After settings-daemon |
| Lifetime | Entire session |
| Restart policy | `on-failure` |

Key thread: poll UPower D-Bus every 5 seconds for battery changes.
Main loop: Qt event loop with D-Bus connection.

#### 2.9 Display Manager

| Property | Value |
|----------|-------|
| Executable | `/usr/libexec/lingmo-display` |
| systemd service | `lingmo-display.service` (user) |
| Type | `notify` |
| Restart policy | `on-failure` |
| Integration | KScreen (libkscreen) for monitor detection. colord for ICC profiles |

Hotplug handling: udev monitor → 500ms debounce timer → query KScreen →
apply saved config or extend current layout.

#### 2.10 Input Manager

| Property | Value |
|----------|-------|
| Executable | `/usr/libexec/lingmo-input` |
| systemd service | `lingmo-input.service` (user) |
| Type | `notify` |
| Restart policy | `on-failure` |

#### 2.11 Audio Manager

| Property | Value |
|----------|-------|
| Executable | `/usr/libexec/lingmo-audio` |
| systemd service | `lingmo-audio.service` (user) |
| Type | `notify` |
| Restart policy | `on-failure` |

#### 2.12 Thumbnail Service

| Property | Value |
|----------|-------|
| Executable | `/usr/libexec/lingmo-thumbnails` |
| systemd service | `lingmo-thumbnails.service` (user) |
| Type | `dbus` |
| Startup | D-Bus activated on first thumbnail request |
| Lifetime | Idle timeout: 30 seconds after last request |
| Restart policy | `on-failure` |

Thread pool model:
- Main thread: D-Bus listener, request queue management
- Worker threads (default: CPU count / 2): parallel thumbnail generation
- Each worker runs content extractors

#### 2.13 Print Service

| Property | Value |
|----------|-------|
| Executable | `/usr/libexec/lingmo-print` |
| systemd service | `lingmo-print.service` (user) |
| Type | `dbus` |
| Startup | D-Bus activated |

#### 2.14 Network & Bluetooth Applets

| Property | Value |
|----------|-------|
| Executable | `/usr/libexec/lingmo-network-applet` |
| systemd service | Started by shell-core (not systemd) |
| Startup | Shell core launches as child process after surfaces are ready |
| Lifetime | Entire session |
| Restart policy | Shell core monitors and restarts on-failure |

These are panel applets that also run background monitoring processes.
The network applet monitors NetworkManager D-Bus signals. The Bluetooth
applet monitors BlueZ D-Bus signals.

#### 2.15 Portal Backend

| Property | Value |
|----------|-------|
| Executable | `/usr/libexec/xdg-desktop-portal-lingmo` |
| systemd service | `xdg-desktop-portal-lingmo.service` (user) |
| Type | `dbus` |
| Startup | D-Bus activated by xdg-desktop-portal |
| Lifetime | As long as portal requests are being processed |
| Restart policy | `on-failure` |

#### 2.16 PolKit Agent

| Property | Value |
|----------|-------|
| Executable | `/usr/libexec/lingmo-polkit-agent` |
| systemd service | `lingmo-polkit-agent.service` (user) |
| Type | `dbus` |
| Startup | D-Bus activated when authentication is required |
| Lifetime | Shows dialog, exits after authentication result |
| Restart policy | `on-failure` |

#### 2.17 Search Indexer

| Property | Value |
|----------|-------|
| Executable | `/usr/libexec/lingmo-indexer` |
| systemd service | `lingmo-indexer.service` (user) |
| Type | `notify` |
| Startup | After shell core, before autostart apps |
| Lifetime | Entire session |
| Restart policy | `on-failure` |
| I/O priority | `Idle` class (not competitive with interactive I/O) |
| CPU scheduling | `Idle` policy (lowest priority) |

#### 2.18 Screen Locker

| Property | Value |
|----------|-------|
| Executable | `/usr/libexec/lingmo-lockscreen` |
| systemd service | Not a systemd service. Started by KWin when locking |
| Startup | KWin sends DBus signal → lingmo-lockscreen starts |
| Lifetime | Until unlocked (user provides correct password) |
| Restart policy | KWin restarts the locker if it crashes while locked |
| Memory model | Owns: everything (standalone process). Isolated from all other processes |

Security constraints:
- seccomp: block all non-essential syscalls (no `open` on user files,
  no `connect` to network, no `fork` except for PAM)
- no access to user session D-Bus bus
- no access to Wayland surfaces other than the lock surface
- PAM conversation via a dedicated socket to pam_unix

---

## 3. IPC Design

### 3.1 Communication Map

```
                            SESSION BUS (D-Bus)
 ┌─────────────────────────────────────────────────────────────────────┐
 │                                                                     │
 │  org.lingmo.Shell ←→ org.lingmo.SettingsDaemon                     │
 │  (window state)     (config change signals)                         │
 │                                                                     │
 │  org.lingmo.Power  ──▶  org.lingmo.OSD                             │
 │  (battery events)      (show volume/brightness OSD)                │
 │                                                                     │
 │  org.lingmo.Audio  ──▶  org.lingmo.OSD                             │
 │  (volume change)       (show volume OSD)                           │
 │                                                                     │
 │  org.lingmo.Input  ──▶  org.lingmo.OSD                             │
 │  (media keys)          (show media OSD)                            │
 │                                                                     │
 │  org.lingmo.Display ──▶  org.lingmo.Shell                          │
 │  (display hotplug)     (re-layout panels)                          │
 │                                                                     │
 │  org.lingmo.Search  ◀──  org.lingmo.Shell (through runner)        │
 │  (query results)       (search query)                               │
 │                                                                     │
 │  org.lingmo.Network ◀──  org.lingmo.Shell                          │
 │  (connection state)    (request network list)                       │
 │                                                                     │
 │  Org.freedesktop.     ◀──  Any app                                  │
 │  Notifications         (send notification)                          │
 │       │                                                             │
 │       └──▶  org.lingmo.Notification (our impl)                     │
 │                                                                     │
 └─────────────────────────────────────────────────────────────────────┘

                            SYSTEM BUS (D-Bus)
 ┌─────────────────────────────────────────────────────────────────────┐
 │                                                                     │
 │  org.kde.KWin ←→ org.lingmo.ShellCore                             │
 │  (window management)    (window list, desktops, operations)         │
 │                                                                     │
 │  org.freedesktop.      ◀──  org.lingmo.Power                       │
 │  UPower                  (battery status, suspend/hibernate)        │
 │                                                                     │
 │  org.freedesktop.      ◀──  org.lingmo.Network                     │
 │  NetworkManager          (connection state, scan, connect)          │
 │                                                                     │
 │  org.bluez             ◀──  org.lingmo.Bluetooth                   │
 │  (device management)                                                │
 │                                                                     │
 │  org.freedesktop.      ◀──  org.lingmo.SessionManager              │
 │  login1                  (session management, user switching)       │
 │                                                                     │
 │  org.freedesktop.      ◀──  org.lingmo.Print                       │
 │  CUPS                    (printer list, job management)             │
 │                                                                     │
 └─────────────────────────────────────────────────────────────────────┘

                            LOCAL SOCKETS (Shell Internal)
 ┌─────────────────────────────────────────────────────────────────────┐
 │                                                                     │
 │  shell-core: /run/user/$UID/lingmo/shell.sock                      │
 │       │                                                            │
 │       ├── panel 0 (monitor 0)                                      │
 │       ├── panel 1 (monitor 1)                                      │
 │       ├── desktop                                                   │
 │       └── overview                                                  │
 │                                                                     │
 │  Protocol: CBOR-encoded messages over Unix stream socket            │
 │  Message types: AppletEvent, WindowEvent, InputEvent, ConfigEvent   │
 │                                                                     │
 └─────────────────────────────────────────────────────────────────────┘
```

### 3.2 D-Bus Interface Specifications

#### 3.2.1 org.lingmo.Shell (Session Bus)

```
Service: org.lingmo.Shell
Path: /org/lingmo/Shell
Interface: org.lingmo.Shell

Methods:
    uint32[] GetWindows()
        → Array of window IDs managed by the shell

    void FocusWindow(uint32 id)
    void CloseWindow(uint32 id)
    void MinimizeWindow(uint32 id)
    void MaximizeWindow(uint32 id)
    void MoveWindow(uint32 id, int32 x, int32 y)
    void ResizeWindow(uint32 id, uint32 width, uint32 height)

    uint32[] GetVirtualDesktops()
    void SwitchToDesktop(uint32 id)
    void CreateDesktop(string name)
    void RemoveDesktop(uint32 id)
    void RenameDesktop(uint32 id, string name)

    string GetCurrentDesktop()
    void SetCurrentDesktop(string id)

    void ShowOverview()
    void HideOverview()
    void ShowRunner()

    void LockScreen()

    ActivationState GetActivationState(uint32 windowId)

Signals:
    void WindowAdded(uint32 id, string appId, string title)
    void WindowRemoved(uint32 id)
    void WindowChanged(uint32 id, string property, variant value)
    void ActiveWindowChanged(uint32 id)
    void VirtualDesktopSwitched(uint32 from, uint32 to)
    void VirtualDesktopAdded(uint32 id, string name)
    void VirtualDesktopRemoved(uint32 id)

Properties:
    read uint32 ActiveWindow
    read uint32[] Windows
    read uint32[] VirtualDesktops
    read string CurrentDesktopName
    read uint32 CurrentDesktopIndex
    read uint32 MonitorCount
```

#### 3.2.2 org.lingmo.SettingsDaemon (Session Bus)

```
Service: org.lingmo.SettingsDaemon
Path: /org/lingmo/SettingsDaemon
Interface: org.lingmo.SettingsDaemon

Methods:
    void Apply(string key, variant value)
        → Signal the daemon to apply a config change system-wide
    string[] GetKeyboardLayouts()
    void SetKeyboardLayouts(string[] layouts)
    string FindKeyboardLayout(string language)
    uint32 GetRepeatDelay()
    uint32 GetRepeatRate()
    void SetRepeatDelay(uint32 ms)
    void SetRepeatRate(uint32 Hz)
    double GetFontScale()
    double GetHinting()
    string GetFontFamily()
    void SetFontScale(double scale)
    void SetFontHinging(double hinting)

Signals:
    void KeyboardLayoutChanged(string[] layouts)
    void FontConfigurationChanged(variant config)
    void ScaleFactorChanged(double scale)
    void GTKThemeChanged(string theme)

Interface: org.lingmo.SettingsDaemon.Removable
Methods:
    void Mount(string udi)
    void Unmount(string udi)
    void Eject(string udi)
    string[] GetRemovableDevices()
```

#### 3.2.3 org.lingmo.OSD (Session Bus)

```
Service: org.lingmo.OSD
Path: /org/lingmo/OSD
Interface: org.lingmo.OSD

Methods:
    void Show(string type, variant value, uint32 monitor)
        type: "volume" | "brightness" | "media" | "airplane" |
              "keyboard" | "notification" | "microphone"
        value: 0-100 for slider types, string for media
        monitor: screen index for multi-monitor
```

#### 3.2.4 org.lingmo.Power (Session Bus)

```
Service: org.lingmo.Power
Path: /org/lingmo/Power
Interface: org.lingmo.Power

Methods:
    void Suspend()
    void Hibernate()
    void Sleep()
    double GetBatteryPercent()
    uint32 GetBatteryState()  // 0=unknown, 1=charging, 2=discharging, 3=full
    double GetTimeToEmpty()
    double GetTimeToFull()
    string GetPowerProfile()  // "performance" | "balanced" | "power-saver"
    void SetPowerProfile(string profile)
    uint32 GetLidAction(bool onBattery)
    uint32 GetPowerButtonAction()

Signals:
    void BatteryPercentChanged(double percent)
    void BatteryStateChanged(uint32 state)
    void PowerProfileChanged(string profile)
    void LidClosed()
    void LidOpened()
    void LowBatteryWarning(double percent)
    void CriticalBatteryWarning(double percent)
```

#### 3.2.5 org.lingmo.Audio (Session Bus)

```
Service: org.lingmo.Audio
Path: /org/lingmo/Audio
Interface: org.lingmo.Audio

Methods:
    double GetVolume()
    bool GetMute()
    void SetVolume(double volume)
    void SetMute(bool mute)
    double GetVolumePercent()
    void SetVolumePercent(double percent)
    void CycleOutput(int direction)
    string[] GetOutputDevices()
    string GetActiveOutputDevice()
    void SetActiveOutputDevice(string id)
    string[] GetInputDevices()
    string GetActiveInputDevice()
    void SetActiveInputDevice(string id)
    void SetApplicationVolume(string appId, double volume)
    string[] GetApplications()

Signals:
    void VolumeChanged(double volume, bool muted)
    void OutputDeviceChanged(string id)
    void InputDeviceChanged(string id)
    void ApplicationAdded(string appId, string name)
    void ApplicationRemoved(string appId)
    void ApplicationVolumeChanged(string appId, double volume)
```

#### 3.2.6 org.lingmo.Input (Session Bus)

```
Service: org.lingmo.Input
Path: /org/lingmo/Input
Interface: org.lingmo.Input

Methods:
    double GetTouchpadSpeed()
    void SetTouchpadSpeed(double speed)
    bool GetTouchpadEnabled()
    void SetTouchpadEnabled(bool enabled)
    bool GetTouchpadTapToClick()
    void SetTouchpadTapToClick(bool enabled)
    double GetMouseSpeed()
    void SetMouseSpeed(double speed)
    bool GetMouseNaturalScroll()
    void SetMouseNaturalScroll(bool natural)
    uint32 GetKeyboardRepeatDelay()
    uint32 GetKeyboardRepeatRate()
    string[] GetKeyboardLayouts()
    void SetKeyboardLayouts(string[] layouts)
    void NextKeyboardLayout()

Signals:
    void KeyboardLayoutChanged(string newLayout)
    void TouchpadEnabledChanged(bool enabled)
    void MouseSettingsChanged()
```

#### 3.2.7 org.lingmo.Search (Session Bus)

```
Service: org.lingmo.Search
Path: /org/lingmo/Search
Interface: org.lingmo.Search

Methods:
    SearchResult[] Search(string query, uint32 maxResults, string[] categories)
        categories: "files" | "apps" | "settings" | "calculator" | "commands"
    void OpenResult(string resultId, string provider)
    string[] GetProviders()

Signals:
    void IndexingProgress(double percent)
    void IndexingComplete()
```

### 3.3 Internal Shell Protocol (Unix Sockets)

For communication between shell-core and surface processes,
Unix sockets are used instead of D-Bus for performance and simplicity.

Protocol: CBOR (RFC 8949) encoded messages over Unix stream sockets.

```
Message Format:
┌────────────────────────────────────────────┐
│ uint32: message_length (network byte order)│
├────────────────────────────────────────────┤
│ CBOR-encoded message body                  │
│ {                                          │
│   "type": "AppletEvent",                   │
│   "seq": 142,                              │
│   "body": { ... }                          │
│ }                                          │
└────────────────────────────────────────────┘
```

Message types:

| Type | Source → Target | Description |
|------|----------------|-------------|
| `AppletLoaded` | Surface → Core | Applet plugin has been loaded |
| `AppletUnloaded` | Surface → Core | Applet plugin unloaded |
| `AppletEvent` | Surface ↔ Core | Applet-specific event |
| `WindowFocus` | Core → Surface | Active window changed (update task bar) |
| `DesktopSwitch` | Core → Surface | Virtual desktop changed (update pager) |
| `WindowAdded` | Core → Surface | New window opened (add to task bar) |
| `WindowRemoved` | Core → Surface | Window closed (remove from task bar) |
| `MonitorChanged` | Core → All | Screen layout changed (re-layout surfaces) |
| `ConfigChanged` | Core → All | Configuration reload required |
| `SurfaceReady` | Surface → Core | Surface is ready for applets |
| `Command` | Core → Surface | Execute action (lock, logout, etc.) |

### 3.4 How Processes Find Each Other

| Target | Discovery Method | Identifier |
|--------|-----------------|------------|
| System services | D-Bus well-known names | `org.freedesktop.NetworkManager` etc. |
| Lingmo daemons | D-Bus well-known names | `org.lingmo.PowerManager` etc. |
| Shell core | D-Bus well-known name | `org.lingmo.Shell` |
| Shell surfaces | Unix socket path | `/run/user/$UID/lingmo/shell.sock` |
| KWin | D-Bus well-known name | `org.kde.KWin` |
| Desktop portals | D-Bus well-known name | `org.freedesktop.portal.Desktop` |
| xdg-desktop-portal | D-Bus activation | `org.freedesktop.portal.Desktop` |

### 3.5 Shared Memory

Shared memory is used in two cases to avoid copying large data:

1. **Thumbnail images**: The thumbnail service writes generated thumbnails
   to the standard Freedesktop thumbnail cache (`~/.cache/thumbnails/`).
   Consumers (file manager, image viewer) read directly from the cache file.
   No shared memory IPC needed — filesystem cache is sufficient.

2. **Screenshot sharing**: The screenshot portal uses PipeWire's shared
   memory DMA-BUF mechanism. The Portal backend does NOT copy pixel data;
   instead, it passes DMA-BUF file descriptors via the PipeWire protocol.

No other shared memory is needed. D-Bus message sizes are kept under 2MB
(the D-Bus spec limit). For larger data (thumbnails, screenshots), filesystem
or PipeWire DMA-BUF transport is used.

---

## 4. Plugin System

### 4.1 Design Goals

1. **One plugin infrastructure for all Lingmo plugin types.**
   No more separate plugin loaders for applets, settings panels, search
   providers, etc. All use `lingmo-plugin`.

2. **ABI stability across minor versions.**
   A plugin compiled for Lingmo 1.0 must work on Lingmo 1.5 without
   recompilation.

3. **No code generation.**
   No moc-like code generators for plugin metadata. Metadata is a JSON
   file installed alongside the plugin binary.

4. **Forward compatibility.**
   A Lingmo 2.0 system can load a Lingmo 1.x plugin if the plugin
   interface major version is within the supported range.

### 4.2 Plugin Metadata Format

Each plugin installs a `.lingmo-plugin` JSON file to
`$prefix/share/lingmo/plugins/` or `~/.local/share/lingmo/plugins/`.

```json
{
    "id": "org.lingmo.applet.digital-clock",
    "type": "applet",
    "version": "2.1.0",
    "api": {
        "name": "org.lingmo.applet",
        "version": "3.0"
    },
    "name": {
        "default": "Digital Clock",
        "zh_CN": "数字时钟",
        "de": "Digitale Uhr"
    },
    "description": "Displays the current date and time",
    "category": "utility",
    "icon": "clock",
    "library": "liblingmo_applet_digital_clock.so",
    "author": "Lingmo Desktop Team",
    "website": "https://lingmo.org/applets/clock",
    "minimumQtVersion": "6.5",
    "dependencies": [
        { "id": "org.lingmo.applet", "version": ">=3.0 <4.0" }
    ],
    "runtimeData": {
        "singleInstance": true,
        "configurable": true,
        "configWidget": "DigitalClockConfig.qml"
    }
}
```

Metadata fields:

| Field | Required | Description |
|-------|----------|-------------|
| `id` | Yes | Reverse domain name, globally unique |
| `type` | Yes | Plugin type: `applet`, `settings-panel`, `search-provider`, `thumbnail-provider`, `content-extractor`, `file-preview` |
| `version` | Yes | Plugin version (semver, used for display only) |
| `api.name` | Yes | Plugin interface identifier |
| `api.version` | Yes | Plugin interface version (semver range compatibility) |
| `library` | Yes | Shared library filename (without path) |
| `name` | Yes | Human-readable name (i18n) |
| `description` | No | Human-readable description (i18n) |
| `icon` | No | Icon name from icon theme |

### 4.3 Plugin Discovery

Discovery order (first match wins):

1. `$LINGMO_PLUGINS_PATH` (environment variable, for development)
2. `~/.local/lib/lingmo/plugins/` (user-installed plugins)
3. `~/.local/share/lingmo/plugins/` (user-installed metadata)
4. `/usr/lib/lingmo/plugins/` (system-installed plugins)
5. `/usr/share/lingmo/plugins/` (system-installed metadata)

Discovery algorithm:

```
scan_directory(path):
    for each .lingmo-plugin file in path/metadata/:
        metadata = parse_json(file)
        if verify_plugin(metadata):
            register_plugin(metadata)
            // library is loaded lazily on first use
```

Verification:

1. JSON schema validation
2. `api.name` matches a known plugin interface
3. `api.version` is within the compatible range of the known interface
4. Library file exists in the library directory
5. Library exports the expected factory symbol

### 4.4 Plugin Loading (Lazy)

```
load_plugin(metadata):
    library = QLibrary(path + metadata.library)
    if not library.load():
        error("Cannot load plugin: " + library.errorString())
        return null

    factory = library.resolve("lingmo_plugin_factory")
    if not factory:
        error("Plugin does not export factory")
        library.unload()
        return null

    instance = factory()
    plugin = qobject_cast<PluginInterface>(instance)
    if not plugin:
        error("Factory returned incompatible object")
        delete instance
        library.unload()
        return null

    return plugin
```

### 4.5 Plugin Interfaces

All plugin interfaces inherit from `Lingmo::Plugin`:

```cpp
// lingmo-plugin/include/Lingmo/Plugin.h
namespace Lingmo {

class Plugin : public QObject {
    Q_OBJECT
public:
    struct Metadata {
        QString id;
        QString type;
        QString version;
        QString apiName;
        QString apiVersion;
        QMap<QString, QString> name;
        QMap<QString, QString> description;
        QString category;
        QString icon;
    };

    Plugin(QObject* parent = nullptr);
    virtual ~Plugin();

    virtual Metadata metadata() const = 0;
    virtual bool initialize() = 0;
    virtual void shutdown() = 0;

    // Provided by the plugin loader:
    static Plugin* load(const QString& id);
    static QList<Plugin*> allOfType(const QString& type);
};

} // namespace Lingmo
```

#### 4.5.1 Applet Interface (lingmo-applets)

```cpp
namespace Lingmo {

class Applet : public Plugin {
    Q_OBJECT
public:
    // Lifecycle
    virtual QQuickItem* createItem(QQuickItem* parent = nullptr) = 0;
    virtual QWidget* createWidget(QWidget* parent = nullptr) = 0;
    // At least one must return non-null. Both for dual-rendering applets.

    // Configuration
    virtual QObject* config() const { return nullptr; }
    virtual QQuickItem* createConfigItem(QQuickItem* parent = nullptr) {
        Q_UNUSED(parent); return nullptr;
    }

    // Constraints
    virtual QStringList supportedLocations() const { return {"top", "bottom", "left", "right"}; }
    virtual QSize minimumSize() const { return {32, 32}; }
    virtual QSize maximumSize() const { return {4096, 4096}; }

signals:
    void titleChanged(const QString& title);
    void iconChanged(const QString& iconName);
    void tooltipChanged(const QString& tooltip);
};

} // namespace Lingmo
```

#### 4.5.2 Settings Panel Interface (lingmo-control-center)

```cpp
namespace Lingmo {

class SettingsPanel : public Plugin {
    Q_OBJECT
public:
    virtual QWidget* createWidget(QWidget* parent = nullptr) = 0;
    virtual bool needsApply() const { return false; }
    virtual void apply() {}
    virtual void reset() {}
    virtual void cancel() {}
    virtual QString title() const = 0;
    virtual QString category() const = 0;  // "appearance", "hardware", "system", etc.
    virtual QString icon() const = 0;
    virtual int sortOrder() const { return 100; }

    // Search
    virtual QStringList keywords() const { return {}; }
    virtual QString helpUrl() const { return {}; }
};

} // namespace Lingmo
```

#### 4.5.3 Search Provider Interface (lingmo-search)

```cpp
namespace Lingmo {

struct SearchResult {
    QString id;
    QString title;
    QString description;
    QString iconName;
    double relevance;  // 0.0 to 1.0
    QString category;  // "files", "apps", "settings", etc.
    QVariantMap data;  // Provider-specific data
};

class SearchProvider : public Plugin {
    Q_OBJECT
public:
    virtual QString name() const = 0;
    virtual double priority() const { return 0.5; }
    virtual void search(const QString& query, int maxResults,
                        std::function<void(QList<SearchResult>)> callback) = 0;
    virtual void activate(const SearchResult& result) = 0;

signals:
    void resultsReady(const QList<SearchResult>& results);
};

} // namespace Lingmo
```

#### 4.5.4 Content Extractor Interface (lingmo-search)

```cpp
namespace Lingmo {

class ContentExtractor : public Plugin {
    Q_OBJECT
public:
    virtual QStringList supportedMimeTypes() const = 0;
    virtual bool canExtract(const QString& filePath) const;
    virtual ExtractionResult extract(const QString& filePath,
                                     const QByteArray& data) = 0;
};

struct ExtractionResult {
    QString plainText;
    QVariantMap metadata;  // title, author, date, etc.
    QImage thumbnail;
    bool success;
    QString errorMessage;
};

} // namespace Lingmo
```

#### 4.5.5 Thumbnail Provider Interface (lingmo-thumbnails)

```cpp
namespace Lingmo {

class ThumbnailProvider : public Plugin {
    Q_OBJECT
public:
    virtual QStringList supportedMimeTypes() const = 0;
    virtual bool canHandle(const QString& filePath) const;
    virtual QImage generate(const QString& filePath, const QSize& size) = 0;
    virtual bool isRemoteCapable() const { return false; }  // Can handle remote files?
};

} // namespace Lingmo
```

### 4.6 Plugin Binary ABI Compatibility

Strategy for ABI stability:

1. **Interfaces are pure virtual classes** with no member variables.
   The vtable layout is the only ABI concern.

2. **New virtual methods are always appended.**
   ```
   class Applet : public Plugin {        // API version 3.0
       virtual QQuickItem* createItem(...) = 0;
       // --- version 3.0 boundary ---
   };

   class Applet : public Plugin {        // API version 3.1 (ABI compatible)
       virtual QQuickItem* createItem(...) = 0;
       virtual QWidget* createWidget(...) = 0;  // new, appended
       // --- version 3.1 boundary ---
   };
   ```
   This preserves ABI: old plugins have nullptr for new virtuals.
   The infrastructure must handle nullptr returns from new methods.

3. **Capability querying via `qobject_cast` instead of interface bloating.**
   ```cpp
   // API 3.0
   class Applet : public Plugin { ... };

   // API 4.0 (new interface, separate)
   class ExtendedApplet : public Plugin {
       virtual bool supportsTouchGesture() const = 0;
   };

   // Consumer code:
   Applet* applet = loadPlugin(id);
   if (auto* extended = qobject_cast<ExtendedApplet*>(applet)) {
       // Use new capabilities
   }
   ```

4. **Removal policy**: A virtual method that has been deprecated can be
   removed only after 2 major versions (e.g., deprecated in API v5,
   removed in API v7). Removal is done by making the method non-virtual
   with a default implementation.

### 4.7 Plugin Sandboxing

Current: Plugins run in-process with the host. No sandbox.

Future: Optional out-of-process plugins via `lingmo-plugin-host`:

```
Host Process                    Plugin Host Process
┌────────────┐                 ┌────────────────────┐
│ Panel      │──IPC Socket────▶│ Plugin Loader      │
│            │                 │   ┌──────────────┐ │
│ Needs      │                 │   │ Plugin Instance│ │
│ Plugin     │                 │   └──────────────┘ │
│ Rendering  │◀──Shared Mem───│ Output: QImage     │
└────────────┘                 └────────────────────┘
```

This is opt-in: plugins declare `"sandbox": "process"` in metadata.
Default is `"sandbox": "none"` (in-process).

### 4.8 Plugin Types Summary

| Plugin Type | Interface Class | Host | API Identifier | Discovery Path |
|-------------|----------------|------|----------------|----------------|
| Applet | `Lingmo::Applet` | `lingmo-shell-surfaces` | `org.lingmo.applet` | `lingmo/applets/` |
| Settings Panel | `Lingmo::SettingsPanel` | `lingmo-control-center` | `org.lingmo.settings-panel` | `lingmo/settings-panels/` |
| Search Provider | `Lingmo::SearchProvider` | `lingmo-search` | `org.lingmo.search-provider` | `lingmo/search-providers/` |
| Content Extractor | `Lingmo::ContentExtractor` | `lingmo-search` | `org.lingmo.extractor` | `lingmo/extractors/` |
| Thumbnail Provider | `Lingmo::ThumbnailProvider` | `lingmo-thumbnails` | `org.lingmo.thumbnail-provider` | `lingmo/thumbnail-providers/` |

---

## 5. Theme System

### 5.1 Architecture

```
Theme files (JSON)
       │
       ▼
┌─────────────────────────────────────┐
│  Lingmo::Theme (C++ class)          │
│  - Parses JSON into typed structs   │
│  - Singleton: Theme::instance()     │
│  - Q_PROPERTY bindings              │
│  - change signal on theme switch    │
└─────────────────────────────────────┘
       │                 │
       ▼                 ▼
┌──────────────┐  ┌──────────────────────┐
│ LingmoStyle  │  │  LingmoTheme QML     │
│ (QProxyStyle) │  │  singleton type      │
│              │  │                      │
│ Reads tokens  │  │  Every QML control   │
│ for drawing   │  │  reads tokens for    │
│ every widget  │  │  styling             │
└──────────────┘  └──────────────────────┘
       │                 │
       ▼                 ▼
┌──────────────────────────────────────┐
│  Identical visual output             │
│  Same colors, same radii, same       │
│  shadows, same fonts, same spacing   │
└──────────────────────────────────────┘
```

### 5.2 Theme File Format

Theme files are JSON. Located in:
- System: `$prefix/share/lingmo/themes/<name>/theme.json`
- User: `~/.local/share/lingmo/themes/<name>/theme.json`
- Default: `$prefix/share/lingmo/themes/lingmo-default/theme.json`

```json
{
    "meta": {
        "name": "Lingmo Default",
        "version": "1.0",
        "author": "Lingmo Design Team",
        "compatible": ">=1.0 <2.0"
    },

    "palette": {
        "window": {
            "background": "#eff0f1",
            "foreground": "#232629",
            "alternateBackground": "#e0e1e3",
            "link": "#2980b9",
            "visitedLink": "#7f8c8d"
        },
        "button": {
            "background": "#f0f1f2",
            "foreground": "#232629",
            "hover": "#e4e5e6",
            "pressed": "#c8c9ca",
            "disabled": "#b0b1b3",
            "disabledForeground": "#8a8b8d",
            "checked": "#d0d1d2"
        },
        "input": {
            "background": "#ffffff",
            "foreground": "#232629",
            "border": "#b0b1b3",
            "focusBorder": "#3daee9",
            "placeholder": "#8a8b8d",
            "disabledBackground": "#e0e1e3"
        },
        "accent": {
            "normal": "#3daee9",
            "hover": "#2e9ad4",
            "pressed": "#1f86bf",
            "disabled": "#7fc4ed",
            "onAccent": "#ffffff"
        },
        "semantic": {
            "success": "#27ae60",
            "warning": "#f39c12",
            "error": "#e74c3c",
            "info": "#3498db"
        },
        "surface": {
            "card": "#ffffff",
            "popup": "#ffffff",
            "tooltip": "#f0f1f2",
            "overlay": "rgba(0, 0, 0, 0.4)"
        },
        "decoration": {
            "titlebar": "#eff0f1",
            "titlebarInactive": "#e0e1e3",
            "titlebarForeground": "#232629",
            "border": "#b0b1b3",
            "shadow": "rgba(0, 0, 0, 0.3)"
        }
    },

    "typography": {
        "families": {
            "primary": "Noto Sans, Noto Sans CJK SC, system-ui",
            "monospace": "JetBrains Mono, Noto Sans Mono, monospace",
            "ui": "Noto Sans, system-ui"
        },
        "sizes": {
            "caption": 10,
            "body_small": 11,
            "body": 13,
            "body_large": 14,
            "subtitle": 15,
            "title": 18,
            "heading": 22,
            "heading_large": 28,
            "display": 36
        },
        "weights": {
            "light": 300,
            "normal": 400,
            "medium": 500,
            "semibold": 600,
            "bold": 700
        },
        "lineHeights": {
            "tight": 1.2,
            "normal": 1.5,
            "relaxed": 1.75
        }
    },

    "spacing": {
        "xxs": 2,
        "xs": 4,
        "sm": 8,
        "md": 12,
        "lg": 16,
        "xl": 24,
        "xxl": 32,
        "xxxl": 48
    },

    "radius": {
        "none": 0,
        "xs": 2,
        "sm": 4,
        "md": 6,
        "lg": 8,
        "xl": 12,
        "xxl": 16,
        "full": 9999,
        "widget": 4,
        "popup": 6,
        "dialog": 8,
        "window": 0
    },

    "shadow": [
        { "elevation": 0, "size": 0, "blur": 0, "offsetX": 0, "offsetY": 0, "color": "rgba(0,0,0,0)", "spread": 0 },
        { "elevation": 1, "size": 1, "blur": 2, "offsetX": 0, "offsetY": 1, "color": "rgba(0,0,0,0.12)", "spread": 0 },
        { "elevation": 2, "size": 1, "blur": 4, "offsetX": 0, "offsetY": 2, "color": "rgba(0,0,0,0.15)", "spread": 0 },
        { "elevation": 3, "size": 2, "blur": 8, "offsetX": 0, "offsetY": 4, "color": "rgba(0,0,0,0.18)", "spread": 0 },
        { "elevation": 4, "size": 3, "blur": 16, "offsetX": 0, "offsetY": 8, "color": "rgba(0,0,0,0.20)", "spread": 0 },
        { "elevation": 5, "size": 4, "blur": 24, "offsetX": 0, "offsetY": 12, "color": "rgba(0,0,0,0.22)", "spread": 0 }
    ],

    "animation": {
        "duration": {
            "instant": 0,
            "faster": 80,
            "fast": 120,
            "normal": 200,
            "slow": 350,
            "slower": 500,
            "slowest": 800
        },
        "easing": {
            "default": "ease-in-out-quad",
            "enter": "ease-out-cubic",
            "exit": "ease-in-cubic",
            "spring": "spring(120, 10)",
            "bounce": "ease-out-bounce"
        }
    },

    "elevation": {
        "base": 0,
        "raised": 1,
        "overlay": 2,
        "dialog": 3,
        "popup": 3,
        "tooltip": 4,
        "dropdown": 3,
        "notification": 4
    },

    "opacity": {
        "disabled": 0.4,
        "inactive": 0.7,
        "ghost": 0.3
    },

    "icon": {
        "defaultSize": 16,
        "symbolicColor": "#232629",
        "disabledColor": "#8a8b8d",
        "activeColor": "#3daee9"
    }
}
```

### 5.3 C++ Theme API

```cpp
// lingmo-theme/include/Lingmo/Theme.h
namespace Lingmo {

class Theme : public QObject {
    Q_OBJECT

    // Palette properties
    Q_PROPERTY(ColorPalette palette READ palette NOTIFY paletteChanged)
    Q_PROPERTY(Color windowColor READ windowColor NOTIFY paletteChanged)
    Q_PROPERTY(Color windowTextColor READ windowTextColor NOTIFY paletteChanged)
    Q_PROPERTY(Color accentColor READ accentColor NOTIFY paletteChanged)

    // Scale factor (for HiDPI)
    Q_PROPERTY(double scaleFactor READ scaleFactor NOTIFY scaleFactorChanged)

public:
    static Theme* instance();

    // --- Palette ---
    struct ColorPalette {
        struct Window { QColor background; QColor foreground; QColor alternateBackground; };
        struct Button { QColor background; QColor foreground; QColor hover; QColor pressed; };
        struct Input { QColor background; QColor foreground; QColor border; QColor focusBorder; };
        struct Accent { QColor normal; QColor hover; QColor pressed; QColor onAccent; };
        struct Semantic { QColor success; QColor warning; QColor error; QColor info; };
        struct Surface { QColor card; QColor popup; QColor tooltip; QColor overlay; };
        struct Decoration { QColor titlebar; QColor titlebarInactive; QColor titlebarForeground; QColor border; QColor shadow; };

        Window window;
        Button button;
        Input input;
        Accent accent;
        Semantic semantic;
        Surface surface;
        Decoration decoration;
    };

    ColorPalette palette() const;

    // Convenience accessors
    QColor windowColor() const;
    QColor windowTextColor() const;
    QColor accentColor() const;
    QColor successColor() const;
    QColor warningColor() const;
    QColor errorColor() const;
    QColor infoColor() const;

    // --- Typography ---
    struct Typography {
        struct Families { QString primary; QString monospace; QString ui; };
        struct Sizes { int caption; int body_small; int body; int body_large; int subtitle; int title; int heading; int heading_large; int display; };
        struct Weights { int light; int normal; int medium; int semibold; int bold; };
        struct LineHeights { double tight; double normal; double relaxed; };
    };

    Typography typography() const;
    QFont font(const QString& sizeRole = "body") const;
    QFont monoFont(const QString& sizeRole = "body") const;

    // --- Spacing ---
    struct Spacing { int xxs; int xs; int sm; int md; int lg; int xl; int xxl; int xxxl; };
    Spacing spacing() const;
    int spacing(const QString& role) const;  // "sm" → 8

    // --- Corner Radii ---
    struct CornerRadii { int none; int xs; int sm; int md; int lg; int xl; int xxl; int full; int widget; int popup; int dialog; int window; };
    CornerRadii radius() const;
    int radius(const QString& role) const;

    // --- Shadow ---
    struct ShadowDef { int elevation; int size; int blur; int offsetX; int offsetY; QColor color; int spread; };
    ShadowDef shadow(int elevation) const;
    QList<ShadowDef> allShadows() const;

    // --- Animation ---
    struct Animation { int instant; int faster; int fast; int normal; int slow; int slower; int slowest; QEasingCurve defaultEasing; };
    Animation animation() const;
    int duration(const QString& role) const;
    QEasingCurve easing(const QString& role) const;

    // --- Elevation ---
    int elevation(const QString& role) const;  // "dialog" → 3

    // --- Icon ---
    int iconSize(const QString& role = "default") const;
    QColor symbolicIconColor() const;

    // --- Theme management ---
    void setTheme(const QString& name);
    QString currentTheme() const;
    QStringList availableThemes() const;

signals:
    void paletteChanged();
    void typographyChanged();
    void themeChanged();
    void scaleFactorChanged();

private:
    void load(const QString& path);
    void reload();  // Called when files change

    struct Private;
    QScopedPointer<Private> d;
};

} // namespace Lingmo
```

### 5.4 QML Theme Integration

```qml
// lingmo-theme/qml/LingmoTheme.qml (singleton)
import QtQml 2.0

pragma Singleton
QtObject {
    readonly property QtObject palette: QtObject {
        readonly property color window: _theme.windowColor
        readonly property color windowText: _theme.windowTextColor
        readonly property color accent: _theme.accentColor
        readonly property color accentHover: _theme.palette.accent.hover
        readonly property color success: _theme.successColor
        readonly property color warning: _theme.warningColor
        readonly property color error: _theme.errorColor
        readonly property color info: _theme.infoColor
        // ... all palette colors exposed
    }

    readonly property QtObject typography: QtObject {
        readonly property font body: _theme.font("body")
        readonly property font bodySmall: _theme.font("body_small")
        readonly property font heading: _theme.font("heading")
        readonly property font monospace: _theme.monoFont("body")
        // ... all typography roles
    }

    readonly property QtObject spacing: QtObject {
        readonly property int sm: _theme.spacing("sm")
        readonly property int md: _theme.spacing("md")
        readonly property int lg: _theme.spacing("lg")
        // ...
    }

    readonly property QtObject radius: QtObject {
        readonly property int widget: _theme.radius("widget")
        readonly property int popup: _theme.radius("popup")
        readonly property int dialog: _theme.radius("dialog")
        // ...
    }

    readonly property int animationDuration: _theme.duration("normal")

    property real scaleFactor: _theme.scaleFactor

    readonly property QtObject shadow: QtObject {
        function forElevation(elevation) {
            return _theme.shadow(elevation)
        }
    }

    signal themeChanged()
}
```

Usage in QML controls:
```qml
import LingmoTheme 1.0 as Theme
import LingmoUI.Controls 1.0

Button {
    background: Rectangle {
        color: hovered ? Theme.palette.accentHover : Theme.palette.accent
        radius: Theme.radius.widget
    }
    contentItem: Text {
        color: Theme.palette.accent.onAccent
        font: Theme.typography.body
    }
}
```

### 5.5 QStyle Theme Integration

The `lingmo-style` QProxyStyle reads `Lingmo::Theme::instance()` for all
drawing operations:

```cpp
// lingmo-style/src/LingmoStyle.cpp (conceptual)
void LingmoStyle::drawPrimitive(PrimitiveElement element,
                                const QStyleOption* option,
                                QPainter* painter,
                                const QWidget* widget) const
{
    auto* theme = Theme::instance();

    switch (element) {
    case PE_PanelButtonCommand: {
        // Draw button background using theme colors
        auto bg = (option->state & State_Sunken)
            ? theme->palette().button.pressed
            : (option->state & State_MouseOver)
                ? theme->palette().button.hover
                : (option->state & State_Enabled)
                    ? theme->palette().button.background
                    : QColor(); // disabled handled by style sheet

        int radius = theme->radius("widget") * theme->scaleFactor();
        painter->setRenderHint(QPainter::Antialiasing);
        painter->setBrush(bg);
        painter->setPen(Qt::NoPen);
        painter->drawRoundedRect(option->rect.adjusted(1, 1, -1, -1), radius, radius);
        break;
    }
    // ... all other primitive, control, and complex element drawing
    }
}
```

### 5.6 Icon Theme Integration

```cpp
// lingmo-icon/include/Lingmo/IconLoader.h
namespace Lingmo {

class IconLoader : public QObject {
    Q_OBJECT
public:
    static IconLoader* instance();

    QIcon loadIcon(const QString& name, const QString& category = "actions",
                   int size = 16) const;

    // Symbolic icon: returns a QIcon that renders using the theme's
    // symbolic icon color. This is how the system tray and sidebar
    // icons get their theme-matching color.
    QIcon loadSymbolicIcon(const QString& name, const QString& category = "actions",
                           int size = 16,
                           const QColor& color = Theme::instance()->symbolicIconColor()) const;

    // Returns the full path to the icon file, for cases where the
    // consumer wants to load the icon themselves.
    QString iconPath(const QString& name, int size = 16) const;

signals:
    void themeChanged();
};

} // namespace Lingmo
```

### 5.7 Theme Switching Flow

```
User selects theme "Lingmo Dark" in Control Center
    │
    ▼
lingmo-control-center writes ~/.config/lingmo/lingmo-theme.conf:
    [Theme]
    name=Lingmo Dark
    accent=#3daee9
    │
    ▼
lingmo-settings-daemon (ConfigMonitor) detects file change
    │
    ▼
lingmo-settings-daemon applies system-wide:
    1. Adds GTK theme override to gsettings
    2. Calls Lingmo::Theme::instance()->setTheme("Lingmo Dark")
    3. Emits D-Bus signal: org.lingmo.SettingsDaemon.ThemeChanged
    │
    ▼
All processes with Lingmo::Theme instance receive:
    Theme::instance()->themeChanged() signal
    │
    ├── QStyle: repaints all widgets → QProxyStyle re-reads colors
    ├── QML controls: theme bindings update → UI redraws
    ├── Window decorations: KWin decoration plugin re-reads tokens
    └── Shell surfaces: re-apply palette to all rendered elements
```

### 5.8 Cursor Theme

Cursor theme is set via `XCURSOR_THEME` environment variable, managed by
`lingmo-settings-daemon`:

```
Config key: input/cursor/theme = "lingmo-cursors"
Config key: input/cursor/size = 24

↓

lingmo-settings-daemon writes:
    ~/.config/environment.d/lingmo-cursor.conf:
        XCURSOR_THEME=lingmo-cursors
        XCURSOR_SIZE=24
```

XWayland apps pick up the environment variables. Wayland apps use the cursor
provided by KWin, which reads the same environment.

### 5.9 Sound Theme

Sound files are in `$prefix/share/lingmo/sounds/`. The sound theme is
selected via config:

```
Config key: sounds/theme = "lingmo"
Config key: sounds/systemSounds = true
```

The `lingmo-notification` process loads sound files from the theme directory
and plays them via libpipewire (or via `QSoundEffect` as fallback).

Sound file naming convention (following the XDG Sound Theme spec):

```
<context>/<event>.<ext>
e.g.:
    notifications/message.oga
    notifications/error.oga
    system/login.oga
    system/logout.oga
    system/startup.oga
    system/shutdown.oga
    device/plugged.oga
    device/unplugged.oga
```

---

## 6. Application Framework

### 6.1 LingmoApplication Base Class

Every Lingmo application uses either `Lingmo::Application` (Widgets)
or the `LingmoUI.Window` QML module (QML).

The framework handles:

```cpp
// lingmo-window/include/Lingmo/Application.h
namespace Lingmo {

class Application : public QApplication {
    Q_OBJECT
public:
    Application(int& argc, char** argv, const QString& appId);
    ~Application();

    // --- Setup (called automatically in constructor) ---
    // 1. Install LingmoStyle as the application style
    //     QApplication::setStyle(new LingmoStyle());
    // 2. Install crash handler (Lingmo::CrashHandler)
    // 3. Initialize Lingmo::Theme singleton
    // 4. Set application metadata:
    //     QApplication::setApplicationName(appId)
    //     QApplication::setOrganizationName("LingmoDesktop")
    //     QApplication::setApplicationVersion(version)
    //     QApplication::setDesktopFileName("org.lingmo." + appId + ".desktop")
    // 5. Set icon theme search path to include Lingmo icon theme
    // 6. Install translator (auto-detect system locale)
    // 7. Set up Qt logging to Lingmo::Logger

    // --- Accessors ---
    static Application* instance();
    QString appId() const;

    // --- Window factory ---
    // Creates a window with Lingmo theming (frameless, shadows, etc.)
    Lingmo::Window* createWindow(QWidget* content = nullptr);

    // --- Configuration ---
    // Returns a Config instance scoped to this application
    // Config file: ~/.config/lingmo/<appId>/config.ini
    Lingmo::Config* config() const;

    // --- Translator ---
    void installTranslator(const QString& domain, const QString& dir = QString());
    // Translations are in: $prefix/share/lingmo/<appId>/translations/

    // --- Command line ---
    // Parses standard Lingmo command-line options:
    // --help, --version, --platform, --display, --theme, --config
    QCommandLineParser* commandLine() const;

    // --- Unique instance ---
    // If "singleInstance" is set in the .desktop file, ensures only
    // one instance runs. Secondary instances send their command line
    // to the primary via D-Bus.
    bool ensureSingleInstance();
    void raiseExistingInstance();
};

} // namespace Lingmo
```

### 6.2 QML Application Entry Point

For QML applications, the entry point is similar but configured in QML:

```qml
// main.qml
import QtQuick
import LingmoUI.Window 1.0
import LingmoUI.Controls 1.0

LingmoWindow {
    id: app
    title: "My App"
    width: 800
    height: 600

    Component.onCompleted: {
        // Application framework is automatically initialized
        // by the QML engine's C++ plugin (lingmo-window)
    }
}
```

The C++ `main.cpp` for a QML app:

```cpp
#include <Lingmo/Application.h>

int main(int argc, char* argv[]) {
    Lingmo::Application app(argc, argv, "my-app");
    // app.setup() is automatic in constructor

    // Load QML
    QQmlApplicationEngine engine;
    engine.load(QUrl("qrc:/main.qml"));

    return app.exec();
}
```

### 6.3 Config Storage

Each application gets a scoped config via `lingmo-config`:

```
File: ~/.config/lingmo/<appId>/config.ini
      ~/.config/lingmo/<appId>/config.json  (alternative if JSON preferred)

Example:
    ~/.config/lingmo/file-manager/config.ini
        [General]
        showHiddenFiles=false
        viewMode=icon
        sortColumn=name
        sortOrder=ascending
        windowWidth=1024
        windowHeight=768
```

### 6.4 Translation

Translation files are in QM format (Qt Linguist), stored in:

```
$prefix/share/lingmo/<appId>/translations/
    <appId>_zh_CN.qm
    <appId>_de.qm
    <appId>.qm  (fallback: English)
```

The `Lingmo::Application` constructor automatically:

1. Loads the system locale (from `LANG`/`LC_MESSAGES`)
2. Loads the corresponding `.qm` file from the translation directory
3. Installs the translator

For shared libraries (lingmo-apps-common), translations are in:

```
$prefix/share/lingmo/lingmo-apps-common/translations/
```

### 6.5 Crash Handling

```cpp
namespace Lingmo {

class CrashHandler {
public:
    // Install signal handlers for SIGSEGV, SIGABRT, SIGFPE, SIGILL, SIGBUS
    static void install(const QString& appId);

    // Generate backtrace
    static QString backtrace(int maxDepth = 64);

    // Write crash report to journald + file
    static void writeCrashReport(const QString& appId, const QString& backtrace);
};

} // namespace Lingmo
```

Crash report location: `~/.local/share/lingmo/crash-reports/<appId>-<timestamp>.crash`

Format:
```
Application: org.lingmo.file-manager
Version: 1.2.3
Qt Version: 6.5.3
Signal: SIGSEGV (11)
Timestamp: 2026-07-08T14:30:00Z

Backtrace:
0: 0x7f1234567890 /usr/lib/libLingmoFileManager.so (_ZN6Lingmo12FileManager12someMethodEv+0x30)
1: 0x7f1234567890 /usr/lib/libQt6Core.so (_ZN6QObject5eventEP6QEvent+0x50)
...

Registers:
RAX: 0x0000000000000000  RBX: 0x00007f1234567890
...
```

Crash reports are visible in the `lingmo-info` app (Crash Report Viewer).

### 6.6 Logging

```cpp
namespace Lingmo {

// Log categories
Q_DECLARE_LOGGING_CATEGORY(app)
Q_DECLARE_LOGGING_CATEGORY(ui)
Q_DECLARE_LOGGING_CATEGORY(backend)

// Usage:
// qCWarning(app) << "File not found:" << path;
// qCInfo(backend) << "Indexing started";
// qCDebug(ui) << "Paint event for widget:" << widget->objectName();

} // namespace Lingmo
```

Log output goes to:
1. `stderr` (when running from terminal)
2. `journald` (when running as systemd service)
3. `~/.local/share/lingmo/logs/<appId>.log` (file fallback, rotated at 10MB)

### 6.7 Command Line Interface

Standard CLI options for all Lingmo applications:

```
$ lingmo-files --help
Usage: lingmo-files [options] [path]

Options:
  --version                    Show version information
  --theme <name>               Start with specific theme
  --config <file>              Use alternative config file
  --platform <platform>        Qt platform plugin (wayland, xcb)
  --display <display>          X11 display (for XWayland)
  --safe-mode                  Disable plugins, use default theme
  -v, --verbose                Verbose output
  -h, --help                   Show this help
```

### 6.8 Plugin Loading in Applications

Applications can load plugins using `Lingmo::Plugin::load()`:

```cpp
#include <Lingmo/Plugin.h>

// Load a specific plugin by ID
auto* plugin = Lingmo::Plugin::load("org.lingmo.my-app.extension");

// Load all plugins of a type
auto* panels = Lingmo::Plugin::allOfType("my-app-plugin");

// Access plugin metadata
qDebug() << plugin->metadata().name;
```

---

## 7. Dependency Rules

### 7.1 Layer Rules

Strict compile-time linking rules:

```
Layer 0: Qt 6, KWin
    │
Layer 1: lingmo-theme, lingmo-utils
    │  (may depend on Layer 0 only)
    │
Layer 2: lingmo-config, lingmo-plugin, lingmo-ipc
    │  (may depend on Layer 0 + Layer 1)
    │
Layer 3: lingmo-style, lingmo-controls, lingmo-effects, lingmo-icon, lingmo-window
    │  (may depend on Layer 0 + Layer 1. NOT Layer 2)
    │
Layer 4: lingmo-apps-common, lingmo-applets
    │  (may depend on Layer 0-3)
    │
Layer 5 (Services): All daemon repos
    │  (may depend on Layer 0-2. NOT Layer 3, 4)
    │
Layer 6 (Shell): All shell repos
    │  (may depend on Layer 0-4. NOT Layer 5)
    │
Layer 7 (Applications): All application repos
    │  (may depend on Layer 0-4. NOT Layer 5, 6)
    │
Layer 8 (SDK): lingmo-sdk
    │  (may depend on Layer 0-4)
```

### 7.2 Specific Constraints

| Constraint | Reasoning | Enforcement |
|------------|-----------|-------------|
| Services must NOT link against lingmo-style | Services run headless. Linking against QStyle would load Qt Widgets, which must never happen on a headless process | CMake: `find_package(LingmoStyle)` is not called in service repos. CI: check imported targets in generated binaries |
| Services must NOT link against lingmo-controls | Same reasoning: no QML in services | CMake: no `find_package(Qt6Quick)` in service CMakeLists.txt |
| Apps must NOT link against lingmo-shell-core | Apps are independent of the shell. They communicate via D-Bus, not via library calls | CI: check shared library dependencies of app binaries |
| lingmo-theme must NOT depend on Qt Quick or Qt Widgets | lingmo-theme is the most foundational library. It must be usable without any UI framework | CMake: only `find_package(Qt6 COMPONENTS Core Gui)` |
| lingmo-theme must NOT depend on lingmo-config | Themes should not require a config backend. They load from files directly | CMake: no `${LingmoConfig_LIBRARIES}` in lingmo-theme |
| Shell surfaces must NOT directly call KWin D-Bus | All KWin communication goes through shell-core. Shell surfaces only talk to shell-core | Architectural design: enforced by code review |
| Portal backend must NOT depend on any specific app | Portal interfaces must be self-contained. No import of lingmo-file-manager etc. | CI: check for cross-repo include paths |

### 7.3 Runtime vs Compile-time Dependencies

A process can communicate with another process at runtime without a compile-time
dependency:

```
Runtime communication channels:
    D-Bus               → no compile-time dependency needed
    Local sockets       → no compile-time dependency needed
    Shared memory       → no compile-time dependency needed
    Wayland protocols   → protocol XML shared, but no library link

Example:
    lingmo-power-manager → D-Bus → lingmo-osd
    (no library link)            (no library link)

    This is correct: power has NO compile-time dependency on OSD.
```

### 7.4 Enforcement

Dependency rules are enforced at three levels:

1. **CMake**: Each repo's `CMakeLists.txt` explicitly declares dependencies.
   If a forbidden dependency is added, the build fails (missing package).

2. **CI**: A CI job runs `ldd` / `objdump -p` on all produced binaries and
   checks that no forbidden shared libraries are linked.

3. **Code Review**: Architecture team reviews all `CMakeLists.txt` changes
   for cross-layer dependency violations.

---

## 8. Versioning

### 8.1 Shared Library Versioning

Each library follows semantic versioning:

```
MAJOR.MINOR.PATCH  (e.g., 1.3.2)

MAJOR: Incompatible ABI change
    → SO version incremented
    → All consumers must recompile
    → Announcement: 3 months before release
    → Deprecation: remove deprecated APIs that were marked in last major

MINOR: Backward-compatible feature addition
    → SO version unchanged
    → New public API added
    → Existing API unchanged
    → New virtual methods appended to plugin interfaces (see section 4.6)

PATCH: Backward-compatible bug fix
    → SO version unchanged
    → No new public API
    → No virtual method additions
    → Only behavior changes (bug fixes, performance improvements)
```

SO version = MAJOR (e.g., `libLingmoTheme.so.1.0.0` → SONAME `libLingmoTheme.so.1`)

### 8.2 Repository Versioning

Each repository has its own `RELEASE_VERSION` file containing the current version.
The version is set in the top-level `CMakeLists.txt`:

```cmake
project(LingmoTheme VERSION 1.3.2)
```

The version follows the library version (if the repo produces a library) or
the application version (if the repo produces an application).

### 8.3 Plugin Interface Versioning

Plugin interfaces have their own version, independent of the implementing library:

```
Format: MAJOR.MINOR
Interface identifier: "org.lingmo.applet" @ "3.1"

MAJOR: Incompatible interface change (virtual methods reordered/removed)
MINOR: New virtual methods appended (ABI compatible)
```

Plugin metadata declares which API version it requires:
```json
"api": { "name": "org.lingmo.applet", "version": "3.1" }
```

Plugin loader checks:
```
major_version_diff = api_major(plugin) - api_major(host)
if major_version_diff != 0:
    REJECT plugin (ABI incompatible)
if minor_version(plugin) > minor_version(host):
    ACCEPT plugin, but new methods will not be called (runtime check via qobject_cast)
if minor_version(plugin) < minor_version(host):
    ACCEPT plugin, host may call new methods → plugin must have null overrides
```

### 8.4 API Stability Policy

| API Category | Stability Promise | Deprecation Period |
|-------------|-------------------|-------------------|
| `Lingmo::Theme` | Stable (LTS) | 2 major versions |
| `Lingmo::Style` | Stable (LTS) | 2 major versions |
| `Lingmo::Application` | Stable (LTS) | 2 major versions |
| `Lingmo::Config` | Stable | 1 major version |
| `Lingmo::Plugin` | Stable | 2 major versions |
| Plugin interfaces | Stable | 2 major versions |
| `Lingmo::Window` | Experimental (first 2 years) | 1 major version |
| `Lingmo::Effects` | Experimental (first 2 years) | 1 major version |
| `Lingmo::IconLoader` | Stable | 1 major version |
| Internal APIs (in detail/) | No stability promise | N/A |

### 8.5 Deprecation Process

1. **Mark deprecated**: Add `[[deprecated]]` attribute + documentation note
2. **Announce**: Release notes mention the deprecation
3. **Remove after**: Deprecated APIs are removed after the declared number
   of major versions

Example deprecation timeline:
```
Version 1.0: API introduced
Version 2.0: API marked deprecated
Version 3.0: API marked deprecated, new alternative available
Version 4.0: API removed (breaks anyone still using it, announced for 3 releases)
```

### 8.6 Application Versioning

Applications follow their own versioning scheme, not tied to the desktop release:

```
Format: MAJOR.MINOR.PATCH
Example: lingmo-files 2.1.0

Applications do NOT promise ABI stability (they are executables, not libraries).
API stability is not relevant for applications.
```

### 8.7 Desktop Release Versioning

The coordinated desktop release has its own version:

```
Year.Release (e.g., "Lingmo Desktop 2026.1", "Lingmo Desktop 2026.2")
```

Each desktop release pins compatible versions of all 47 repositories.
A compatibility matrix is maintained:

```
Lingmo Desktop 2026.1:
    lingmo-theme >= 1.2.0 < 2.0.0
    lingmo-shell-core >= 1.0.0 < 2.0.0
    lingmo-file-manager >= 1.1.0 < 2.0.0
    ...
```

---

## 9. Build System

### 9.1 CMake Layout (Standard per Repository)

```
<repo>/
├── CMakeLists.txt
├── cmake/
│   ├── Find<Dep>.cmake              # Custom find modules if needed
│   └── LingmoCompilerSettings.cmake # Shared compiler flags
├── src/
│   ├── CMakeLists.txt
│   ├── module1/
│   │   ├── CMakeLists.txt
│   │   ├── LingmoModule1.h          # Public header
│   │   ├── LingmoModule1_p.h        # Private header
│   │   └── LingmoModule1.cpp
│   └── module2/
├── qml/                              # QML modules (if applicable)
│   └── LingmoUI/
│       └── Controls/
│           ├── qmldir
│           └── *.qml
├── tests/
│   ├── CMakeLists.txt
│   ├── tst_Module1Test.cpp
│   └── tst_Module2Test.cpp
├── tools/                             # CLI tools if any
├── debian/                            # Debian packaging
├── rpm/                               # Fedora RPM spec
├── .github/workflows/
│   ├── build.yml
│   ├── test.yml
│   └── package.yml
├── .cmake-format.yaml
├── .clang-format
├── .clang-tidy
├── RELEASE_VERSION
└── README.md
```

### 9.2 Top-level CMakeLists.txt Template

```cmake
cmake_minimum_required(VERSION 3.21)

project(LingmoTheme
    VERSION 1.2.0
    DESCRIPTION "Lingmo Desktop Theme Library"
    HOMEPAGE_URL "https://lingmo.org"
    LANGUAGES CXX
)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# Compiler settings
include(cmake/LingmoCompilerSettings.cmake)

# Dependencies
find_package(Qt6 6.5 REQUIRED COMPONENTS Core Gui)

# Subdirectories
add_subdirectory(src)

if(BUILD_TESTING)
    enable_testing()
    add_subdirectory(tests)
endif()
```

### 9.3 Library CMakeLists.txt Template

```cmake
# src/CMakeLists.txt

set(SOURCES
    Theme.cpp
    ColorPalette.cpp
    Typography.cpp
)

set(HEADERS
    Lingmo/Theme.h
    Lingmo/ColorPalette.h
    Lingmo/Typography.h
)

add_library(LingmoTheme SHARED ${SOURCES} ${HEADERS})

target_include_directories(LingmoTheme
    PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
        $<INSTALL_INTERFACE:include>
    PRIVATE
        ${CMAKE_CURRENT_SOURCE_DIR}/detail
)

target_link_libraries(LingmoTheme PUBLIC Qt6::Core Qt6::Gui)

# Version the shared library
set_target_properties(LingmoTheme PROPERTIES
    VERSION ${PROJECT_VERSION}
    SOVERSION ${PROJECT_VERSION_MAJOR}
    EXPORT_NAME LingmoTheme
)

# Install
install(TARGETS LingmoTheme
    EXPORT LingmoThemeTargets
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
)

install(FILES ${HEADERS}
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/Lingmo
)

install(EXPORT LingmoThemeTargets
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/LingmoTheme
    NAMESPACE Lingmo::
)

# Config file
install(FILES
    ${CMAKE_SOURCE_DIR}/cmake/LingmoThemeConfig.cmake
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/LingmoTheme
)
```

### 9.4 QML Module CMakeLists.txt Template

```cmake
# For QML modules (lingmo-controls, etc.)
qt_add_qml_module(LingmoUIControls
    URI LingmoUI.Controls
    VERSION 1.0
    QML_FILES
        Button.qml
        TextField.qml
        Slider.qml
        # ...
    RESOURCES
        qmldir
)

target_link_libraries(LingmoUIControls
    PUBLIC Qt6::Qml Qt6::Quick Lingmo::Theme
)
```

### 9.5 Export Structure

Each library exports:

```
# For consumers (third-party app developers):
find_package(LingmoTheme REQUIRED)
target_link_libraries(my_app PRIVATE Lingmo::Theme)

# CMake config files are installed to:
${CMAKE_INSTALL_LIBDIR}/cmake/LingmoTheme/
    LingmoThemeConfig.cmake
    LingmoThemeConfigVersion.cmake
    LingmoThemeTargets.cmake
    LingmoThemeTargets-<config>.cmake

# Usage from the build directory:
# LingmoTheme_DIR=/path/to/lingmo-theme/build
```

### 9.6 Install Layout

```
/usr/
├── bin/
│   └── lingmo-example-binary
├── lib/
│   ├── libLingmoTheme.so.1          # SONAME link
│   ├── libLingmoTheme.so.1.2.0      # Actual library
│   ├── libLingmoTheme.so             # Development symlink
│   ├── cmake/LingmoTheme/
│   │   ├── LingmoThemeConfig.cmake
│   │   ├── LingmoThemeConfigVersion.cmake
│   │   └── LingmoThemeTargets.cmake
│   └── lingmo/
│       ├── plugins/                   # Shared libraries for plugins
│       └── qml/                       # QML modules
│           └── LingmoUI/
│               └── Controls/
│                   ├── qmldir
│                   └── *.qml
├── libexec/
│   ├── lingmo-shell-core
│   ├── lingmo-panel
│   ├── lingmo-desktop
│   ├── lingmo-overview
│   ├── lingmo-settings-daemon
│   ├── lingmo-power
│   ├── lingmo-display
│   ├── lingmo-input
│   ├── lingmo-audio
│   ├── lingmo-thumbnails
│   ├── lingmo-print
│   ├── lingmo-notification
│   ├── lingmo-osd
│   ├── lingmo-runner
│   ├── lingmo-session
│   ├── lingmo-lockscreen
│   ├── lingmo-indexer
│   ├── xdg-desktop-portal-lingmo
│   └── lingmo-polkit-agent
├── include/
│   └── Lingmo/
│       ├── Theme.h
│       ├── Config.h
│       └── ...
└── share/
    ├── lingmo/
    │   ├── themes/
    │   │   └── lingmo-default/
    │   │       └── theme.json
    │   ├── wallpapers/
    │   ├── icons/
    │   ├── sounds/
    │   ├── plugins/
    │   │   ├── applets/              # .lingmo-plugin metadata
    │   │   ├── settings-panels/
    │   │   ├── search-providers/
    │   │   └── extractors/
    │   ├── translations/
    │   └── sddm/
    ├── icons/hicolor/                # Icon theme (XDG spec)
    ├── sddm/themes/                   # SDDM login theme
    └── plymouth/themes/               # Plymouth boot splash
```

### 9.7 Build Types

| Build Type | Description |
|-----------|-------------|
| `Debug` | No optimization, full debug symbols, assertions enabled |
| `Release` | Full optimization (-O2), no debug symbols, no assertions |
| `RelWithDebInfo` | Full optimization + debug symbols (for profiling) |
| `Sanitized` | Debug + AddressSanitizer + UndefinedBehaviorSanitizer |

CI builds in `RelWithDebInfo` for tests and `Release` for packaging.

---

## 10. Testing

### 10.1 Test Pyramid

```
                    ╱╲
                   ╱  ╲
                  ╱ UI ╲
                 ╱ tests╲
                ╱────────╲
               ╱Integration╲
              ╱   tests     ╲
             ╱────────────────╲
            ╱   Unit tests      ╲
           ╱    (per repo)        ╲
          ╱─────────────────────────╲
         ╱  Static analysis + lint   ╲
        ╱  (clang-tidy, clang-format) ╲
       ╱────────────────────────────────╲
```

### 10.2 Unit Tests

Every C++ class with non-trivial logic has unit tests.

Framework: Qt Test (QTestLib, part of Qt).

Location: `tests/` directory in each repository.

```cpp
// tests/tst_ThemeTest.cpp
#include <QtTest>
#include <Lingmo/Theme.h>

class tst_Theme : public QObject {
    Q_OBJECT

private slots:
    void initTestCase() {
        // Set up a test environment with known theme files
    }

    void testSingleton() {
        auto* theme = Lingmo::Theme::instance();
        QVERIFY(theme != nullptr);
    }

    void testPaletteColors() {
        auto palette = Lingmo::Theme::instance()->palette();
        QCOMPARE(palette.window.background, QColor("#eff0f1"));
    }

    void testRadiusRoles() {
        QCOMPARE(Lingmo::Theme::instance()->radius("widget"), 4);
        QCOMPARE(Lingmo::Theme::instance()->radius("dialog"), 8);
    }
};

QTEST_MAIN(tst_Theme)
```

Target: 80%+ code coverage for library code, 60%+ for application code.

### 10.3 Integration Tests

Cross-repo tests that verify IPC between components.

```cpp
// tests/integration/tst_SettingsDaemon.cpp
// Requires: lingmo-settings-daemon running in test mode

class tst_SettingsDaemon : public QObject {
    Q_OBJECT

private slots:
    void testApplyKeyboardLayout() {
        // Connect to the daemon's D-Bus interface
        QDBusInterface iface("org.lingmo.SettingsDaemon",
                             "/org/lingmo/SettingsDaemon",
                             "org.lingmo.SettingsDaemon");

        // Apply a layout change
        QVERIFY(iface.call("SetKeyboardLayouts", {"us", "de"}).isValid());

        // Verify the system actually changed
        // (check output of `setxkbmap -query` or similar)
    }
};
```

Integration tests run in a dedicated CI pipeline with a virtual display
(see section 10.7).

### 10.4 UI Tests

Framework: Qt Test GUI testing (QTest::mouseClick, etc.).

```cpp
// tests/tst_ButtonTest.cpp
#include <QtTest/QtTest>
#include <Lingmo/Controls/Button.h>

class tst_Button : public QObject {
    Q_OBJECT

private slots:
    void testClick() {
        Lingmo::Button button("Click me");
        button.show();
        QVERIFY(QTest::qWaitForWindowExposed(&button));

        QSignalSpy spy(&button, &QPushButton::clicked);
        QTest::mouseClick(&button, Qt::LeftButton);
        QCOMPARE(spy.count(), 1);
    }
};
```

### 10.5 Screenshot Regression Tests

Each UI component has reference screenshots stored in the repository.
Tests render the component and compare against the reference.

```
tests/screenshots/
├── references/
│   ├── button-default.png
│   ├── button-hover.png
│   ├── button-pressed.png
│   ├── textfield-empty.png
│   ├── textfield-focused.png
│   └── ...
├── tst_ScreenshotComparison.cpp
└── CMakeLists.txt
```

Comparison tolerance: 0.1% pixel difference allowed (for anti-aliasing
differences across platforms). Anything above triggers a failure and
uploads the diff image as a CI artifact.

### 10.6 Wayland Tests

For components that require a Wayland compositor:

```bash
# Headless KWin instance for testing
kwin_wayland --headless --virtual-size 1920x1080 &
sleep 2  # Wait for KWin to initialize

# Run tests in the virtual Wayland session
WAYLAND_DISPLAY=wayland-0 ctest --test-dir build

# Take screenshot of virtual display for verification
kwin_screenshot --output /tmp/test-screenshot.png

# Cleanup
kill %1
```

This is used by:
- `lingmo-shell-surfaces` (panel/desktop/overlay layout)
- `lingmo-notification` (notification bubble positioning)
- `lingmo-osd` (OSD overlay rendering)

### 10.7 X11 Tests

For components that support X11 fallback:

```bash
Xvfb :99 -screen 0 1920x1080x24 &
export DISPLAY=:99
ctest --test-dir build
kill %1
```

### 10.8 Performance Benchmarks

Framework: Google Benchmark (or Qt Test benchmarking).

```cpp
// tests/benchmarks/bm_IconLoading.cpp
#include <benchmark/benchmark.h>
#include <Lingmo/IconLoader.h>

static void BM_IconLoading(benchmark::State& state) {
    auto* loader = Lingmo::IconLoader::instance();
    for (auto _ : state) {
        auto icon = loader->loadIcon("document-open", "actions", 32);
        benchmark::DoNotOptimize(icon);
    }
}
BENCHMARK(BM_IconLoading);

// Startup time benchmark
static void BM_AppStartup(benchmark::State& state) {
    for (auto _ : state) {
        int argc = 1;
        char* argv[] = {const_cast<char*>("test")};
        Lingmo::Application app(argc, argv, "benchmark");
        benchmark::DoNotOptimize(app);
    }
}
BENCHMARK(BM_AppStartup);
```

Benchmarks are run weekly, not on every commit. Results are tracked
in a dashboard to detect regressions.

### 10.9 CI Test Matrix

```
.pipeline:
  - lint (clang-tidy, clang-format, qmllint, cmake-lint)
  - build (Debug + tests)
  - build (Release + tests)
  - unit-tests (ctest --output-on-failure)
  - integration-tests (requires KWin virtual display)
  - screenshot-tests (requires Wayland virtual display)
  - package-deb (cpack -G DEB)
  - package-rpm (cpack -G RPM)
```

### 10.10 Continuous Integration

Each repository has three GitHub Actions workflows:

1. **build.yml**: On every push and PR.
   - Checkout → Install deps → CMake configure → Build → Unit tests

2. **test.yml**: On every push to main, nightly.
   - Build → Integration tests → Screenshot regression → Performance benchmarks
   - Runs on both `ubuntu-24.04` and the matrix of architectures

3. **package.yml**: On tags (v*) and release branches.
   - Build for multiple distros → Generate .deb and .rpm → Upload artifacts

---

## 11. Release Engineering

### 11.1 Git Branching Model

```
main
  │
  ├── dev/<feature>         # Feature branches (merge → main via PR)
  ├── fix/<bug>             # Bug fix branches
  │
  ├── stable/2026.1         # Release branch (from main, only bug fixes)
  │     └── v2026.1.0       # Tag (release)
  │     └── v2026.1.1       # Tag (patch)
  │
  └── stable/2026.2         # Next release branch
        └── v2026.2.0       # Tag
```

### 11.2 Release Cadence

| Release Type | Cadence | Description |
|-------------|---------|-------------|
| Major desktop release | 2 per year (April, October) | Coordinated release of all repos |
| Patch release | As needed (monthly) | Bug fixes only, no new features |
| Security release | Within 48 hours of disclosure | Critical fixes, minimal changes |
| Service pack | Quarterly | Cumulative patches for stable branch |

### 11.3 Release Process

```
1. Freeze: main branch is frozen for feature commits
2. Testing: full test suite + manual QA (1 week)
3. Branch: create stable/YYYY.MM from main
4. Stabilize: only bug fixes on stable branch (2 weeks)
5. Tag: vYYYY.MM.0 on stable branch
6. Package: CI generates .deb and .rpm packages
7. Publish: upload to GitHub Releases + package repositories
8. Announce: release notes, blog post, social media
9. Unfreeze: main branch opens for next release features
```

### 11.4 Package Generation

Each repository generates independent packages using CPack.

#### Debian Packages

```bash
# In each repo:
mkdir -p debian
# debian/control, debian/rules, debian/changelog, debian/compat, etc.
# CI runs:
dpkg-buildpackage -us -uc
# Or equivalently with CPack:
cpack -G DEB
```

Package naming:
```
liblingmo-theme1          # Runtime library (SONAME versioned)
liblingmo-theme-dev       # Development headers
lingmo-shell-core         # Shell core binary
lingmo-panel              # Panel binary
lingmo-settings-daemon    # Daemon binary
lingmo-theme-default      # Theme data (arch: all)
```

#### Fedora RPM Packages

```bash
# In each repo:
mkdir -p rpm
# rpm/*.spec file
# CI runs:
rpmbuild -ba rpm/*.spec
# Or equivalently with CPack:
cpack -G RPM
```

### 11.5 GitHub Actions Structure

```yaml
# .github/workflows/build.yml (per repo)
name: Build

on: [push, pull_request]

jobs:
  build:
    strategy:
      matrix:
        os: [ubuntu-24.04, ubuntu-24.04-arm64]
        config: [Debug, Release]

    runs-on: ${{ matrix.os == 'ubuntu-24.04-arm64' && 'ubuntu-24.04' || matrix.os }}
    # For arm64: use QEMU or self-hosted runner

    steps:
      - uses: actions/checkout@v4

      - name: Install Dependencies
        run: |
          sudo apt-get update
          sudo apt-get install -y $(cat debian/control | grep Build-Depends | \
            sed 's/Build-Depends://' | tr ',' ' ')

      - name: Configure
        run: cmake -B build -DCMAKE_BUILD_TYPE=${{ matrix.config }}

      - name: Build
        run: cmake --build build --parallel $(nproc)

      - name: Test
        run: ctest --test-dir build --output-on-failure
```

### 11.6 Release Synchronization

A meta-repository `lingmo-desktop-release` manages the coordinated release:

```
lingmo-desktop-release/
├── manifest.json            # Pinned versions of all 47 repos
├── sync-release.py          # Script to update all repos to release versions
├── check-compatibility.py   # Verify all pinned versions are compatible
└── RELEASE_NOTES.md
```

`manifest.json` format:
```json
{
    "release": "2026.1",
    "date": "2026-04-15",
    "repositories": {
        "lingmo-theme": { "version": "1.2.0", "ref": "v1.2.0" },
        "lingmo-shell-core": { "version": "1.0.0", "ref": "v1.0.0" },
        "lingmo-file-manager": { "version": "2.1.0", "ref": "v2.1.0" },
        ...
    }
}
```

### 11.7 Artifact Publishing

Packages are published to:

1. **GitHub Releases**: Each repository's GitHub Releases page
2. **Debian repository**: `https://packages.lingmo.org/debian/` (APT repo)
3. **Fedora COPR**: `https://copr.fedorainfracloud.org/coprs/lingmo/`
4. **Flatpak**: Lingmo applications on Flathub (future)

---

## 12. Long-term Evolution

### 12.1 Five-Year Roadmap (2026–2031)

| Year | Focus Areas |
|------|-------------|
| 2026 | Desktop shell stabilization, app ecosystem, Wayland-only default |
| 2027 | Accessibility overhaul, performance optimization, Flatpak integration |
| 2028 | Mobile/tablet adaptation, remote desktop, cloud accounts |
| 2029 | HDR/color management, next-gen compositor features, AI integration |
| 2030 | Sixth-gen UI refresh, Wayland-only (X11 dropped), new plugin APIs |

### 12.2 Repositories Expected to Grow

| Repository | Growth Direction | Expected 5-Year Size |
|------------|-----------------|---------------------|
| `lingmo-shell-surfaces` | New applet types, evolution of panel/desktop/overview UI | 2x |
| `lingmo-search` | More extractors, AI-powered search, offline NLP | 3x |
| `lingmo-portals` | New portal interfaces, security hardening | 2x |
| `lingmo-settings-daemon` | More hardware support (printers, scanners, game controllers) | 2x |
| `lingmo-audio` | Spatial audio, pro-audio features, dynamic routing UI | 2x |
| `lingmo-display` | HDR support, color-accurate workflows, multi-GPU | 3x |
| `lingmo-control-center` | New settings panels, convergence settings (tablet) | 1.5x |
| `lingmo-file-manager` | Cloud file integration, tags, advanced search UI | 1.5x |
| `lingmo-terminal` | GPU-accelerated rendering, split pane improvements, sixel | 1.5x |

### 12.3 Stable Repositories (Rarely Change)

| Repository | Reason for Stability |
|------------|---------------------|
| `lingmo-theme` | Design token format stabilizes after first release. New tokens added via minor versions, never major |
| `lingmo-style` | QProxyStyle becomes stable after all Qt widgets are covered. Only bug fixes after year 2 |
| `lingmo-plugin` | Plugin API is designed for forward compatibility. Interface additions via new interface types, not API changes |
| `lingmo-decoraation` | KWin Decoration API is stable. Style follows theme tokens, no feature growth |
| `lingmo-utils` | Utility functions reach completeness quickly. Bug fixes only |
| `lingmo-icon` | Icon theme engine stabilizes. Only compatibility fixes for new Qt versions |

### 12.4 Future Expansion Paths (without breaking existing APIs)

1. **New plugin types** (add to `lingmo-plugin`):
   - `FilePreview` (quick preview in file manager)
   - `InputMethod` (custom IME integration)
   - `NotificationAction` (custom notification action handlers)
   - `SessionSaveHandler` (participate in session save/restore)
   - These are NEW interface identifiers, not modifications to existing ones

2. **New portal interfaces** (add to `lingmo-portals`):
   - Follow the xdg-desktop-portal upstream additions
   - Each new portal interface is a separate C++ class implementing
     the portal's D-Bus interface

3. **New Wayland protocols** (new protocol XML files):
   - lingmo-virtual-keyboard (on-screen keyboard protocol)
   - lingmo-touch-gesture (custom touch gesture events)
   - lingmo-wallpaper (set wallpaper from compositor level)
   - These are new XML files, not modifications to existing protocols

4. **New application types** (new repositories):
   - Media player (lingmo-media-player)
   - Weather app (lingmo-weather, as a data provider + UI)
   - Dictionary (lingmo-dictionary, offline + online)
   - Maps (lingmo-maps, OpenStreetMap integration)
   - These are new repos following the same application framework conventions

### 12.5 Deprecation and Removal Path

```
Phase 1 (2028): Drop X11-only features
    - Remove X11-specific code from all components
    - Controls: drop X11-specific rendering paths
    - Shell: drop X11-specific window management
    - KWin: continue using XWayland for legacy apps

Phase 2 (2029): Remove backward compatibility shims
    - Remove Qt 5 compatibility code
    - Remove legacy config format support
    - Remove deprecated plugin interfaces (api major +2)

Phase 3 (2030): Major API refresh
    - lingmo-theme API v2: simplified token structure
    - Plugin interfaces v2: error-handling improvements
    - All old APIs maintained for one more major version
      with deprecation warnings
```

### 12.6 Breaking Change Policy

A breaking change MUST:

1. Be announced at least 6 months in advance
2. Be discussed in a public RFC (GitHub Discussion)
3. Have a migration guide
4. Have the old API available for at least one major version
5. Be accompanied by automated migration tools where possible

### 12.7 Technology Watchlist

Technologies to monitor for future adoption:

| Technology | Why Watch | Potential Impact |
|-----------|-----------|-----------------|
| Wayland ext-* protocols | New Wayland protocols for idle inhibit, session lock, etc. | Replace custom implementations |
| PipeWire new features | On-demand audio routing, Bluetooth LE Audio | Audio manager improvements |
| Vulkan rendering in Qt | Qt 6's graphics abstraction | hardware-accelerated UI |
| Flatpak portals v2 | Next-gen sandbox integration | Lingmo portals evolution |
| systemd extensions | systemd user services improvements | Session manager simplification |
| Machine learning on-device (ONNX) | Local AI for search, photo tagging | lingmo-search, lingmo-file-manager |
| AV1 encoding | Royalty-free video codec | lingmo-capture updates |

---

> This document describes version 3.0 of the Lingmo Desktop architecture.
>
> It is intended to be a living document, updated as the desktop evolves.
> When making significant changes to any component, update the relevant
> section of this document.
>
> Lingmo Desktop Architecture Team
