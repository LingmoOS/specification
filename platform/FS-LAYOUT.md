# Filesystem Layout Specification

| | |
|---|---|
| **Document ID** | SPEC-PLATFORM-001 |
| **Title** | Filesystem Layout Specification |
| **Version** | 1.0.0 |
| **Status** | Draft |
| **Category** | Platform |
| **Applies to** | All Lingmo Desktop packages for Debian and Fedora |
| **Supersedes** | SPEC-REPO-001 §9.3–§9.6, SPEC-REPO-003 §7.1 |
| **Cross-ref** | ARCHITECTURE-v3.md, SPEC-REPO-001 (Repository Layout), SPEC-REPO-003 (CMake Organization), SPEC-CORE-001 (Naming Convention) |

---

## 1. Scope

### 1.1 Applicability

This specification defines the canonical filesystem hierarchy for Lingmo Desktop on Linux. It applies to:

- All packages produced by Lingmo Desktop repositories (`.deb` for Debian, `.rpm` for Fedora)
- All runtime paths used by Lingmo Desktop processes
- All data, configuration, cache, and state locations
- All future Lingmo Desktop components

### 1.2 Platforms Covered

| Distribution | Base Paths | Package Format |
|---|---|---|
| Debian (≥ 12) | `/usr/` prefix, Debian policy | `.deb` |
| Fedora (≥ 39) | `/usr/` prefix, Fedora packaging guidelines | `.rpm` |

### 1.3 Exceptions

- **Flatpak / Snap / container installations**: These runtimes MAY remap paths according to their own isolation model. Lingmo Desktop components running inside containers MUST NOT assume the host paths defined here.
- **Development builds from source**: Developers MAY use a custom `CMAKE_INSTALL_PREFIX`. The relative paths defined here remain valid regardless of the prefix.

### 1.4 Supersession Notice

Section 9.3 (Installation Paths) through 9.6 (Runtime File Ownership) of SPEC-REPO-001 and Section 7.1 (GNUInstallDirs) of SPEC-REPO-003 are superseded by this document. The concrete path values defined here take precedence over the abstract `GNUInstallDirs` variables.

---

## 2. FHS Compliance

### 2.1 Compliance Statement

[REQ-FS-001] Lingmo Desktop SHALL conform to the Filesystem Hierarchy Standard (FHS) 3.0 wherever practical.

[REQ-FS-002] All Lingmo-managed directories SHALL be placed under the following FHS-compliant roots:

| FHS Root | Lingmo Usage | GNUInstallDirs Variable |
|---|---|---|
| `/usr/bin/` | User-facing executables | `CMAKE_INSTALL_BINDIR` |
| `/usr/libexec/` | Internal executables | `CMAKE_INSTALL_LIBEXECDIR` |
| `/usr/lib/` | Shared libraries | `CMAKE_INSTALL_LIBDIR` |
| `/usr/include/` | Public headers | `CMAKE_INSTALL_INCLUDEDIR` |
| `/usr/share/` | Architecture-independent data | `CMAKE_INSTALL_DATADIR` |
| `/etc/` | System-wide configuration | `CMAKE_INSTALL_SYSCONFDIR` |
| `/var/cache/` | Persistent cache | — |
| `/run/` | Runtime state | — |
| `/tmp/` | Temporary files | — |

### 2.2 Intentional Deviations

[REQ-FS-003] The following deviations from FHS are documented and justified:

| FHS Rule | Lingmo Practice | Justification |
|---|---|---|
| `/usr/libexec/` is not in FHS 3.0 (optional) | `/usr/libexec/` is REQUIRED | De-facto standard on Debian and Fedora. Avoids polluting `/usr/bin/` with internal executables. |
| `/etc/opt/` for add-on packages | `/etc/lingmo/` directly | Lingmo Desktop is the primary desktop environment, not an add-on. `/etc/lingmo/` is consistent with KDE and GNOME conventions. |
| `/var/cache/` for application cache | `~/.cache/lingmo/` per XDG Base Dir | User-specific cache MUST NOT be shared system-wide. XDG Base Directory Specification takes precedence for per-user data. |
| `/var/log/` for logs | `~/.local/state/lingmo/log/` per XDG Base Dir | User session logs are per-user. System-level logs go to journald. |

### 2.3 XDG Base Directory Compliance

[REQ-FS-004] Lingmo Desktop SHALL follow the XDG Base Directory Specification for all per-user paths:

| XDG Variable | Default | Lingmo Use |
|---|---|---|
| `$XDG_CONFIG_HOME` | `~/.config/` | Per-user config |
| `$XDG_DATA_HOME` | `~/.local/share/` | Per-user data |
| `$XDG_CACHE_HOME` | `~/.cache/` | Per-user cache |
| `$XDG_STATE_HOME` | `~/.local/state/` | Per-user state |
| `$XDG_RUNTIME_DIR` | `/run/user/$UID/` | Per-user runtime files |

[REQ-FS-005] Lingmo Desktop SHALL NOT hardcode `~/.config/`, `~/.local/`, or `~/.cache/`. Every per-user path MUST be resolved through the corresponding XDG environment variable.

---

## 3. Executables

### 3.1 User-Facing Executables

[REQ-FS-006] User-facing executables (applications that users invoke from a terminal, launcher, or `.desktop` file) MUST be installed to `/usr/bin/`.

| Executable | Purpose | Source Repository |
|---|---|---|
| `/usr/bin/lingmo-control-center` | System settings application | `lingmo-control-center` |
| `/usr/bin/lingmo-file-manager` | File manager | `lingmo-file-manager` |
| `/usr/bin/lingmo-terminal` | Terminal emulator | `lingmo-terminal` |
| `/usr/bin/lingmo-text-editor` | Text editor | `lingmo-text-editor` |
| `/usr/bin/lingmo-image-viewer` | Image viewer | `lingmo-image-viewer` |
| `/usr/bin/lingmo-software` | Software center | `lingmo-software` |
| `/usr/bin/lingmo-info` | System information | `lingmo-info` |

### 3.2 Internal Executables

[REQ-FS-007] Internal executables (daemons, shell components, services not intended for direct user invocation) MUST be installed to `/usr/libexec/`.

| Executable | Purpose | Source Repository |
|---|---|---|
| `/usr/libexec/lingmo-session` | Session manager (PID 1 of user slice) | `lingmo-session` |
| `/usr/libexec/lingmo-shell-core` | Shell core process | `lingmo-shell-core` |
| `/usr/libexec/lingmo-panel` | Shell panel surface | `lingmo-shell` |
| `/usr/libexec/lingmo-desktop` | Shell desktop surface | `lingmo-shell` |
| `/usr/libexec/lingmo-overview` | Shell overview surface | `lingmo-shell` |
| `/usr/libexec/lingmo-settings-daemon` | Settings daemon | `lingmo-settings-daemon` |
| `/usr/libexec/lingmo-power` | Power manager daemon | `lingmo-power-manager` |
| `/usr/libexec/lingmo-display` | Display manager daemon | `lingmo-display-manager` |
| `/usr/libexec/lingmo-input` | Input manager daemon | `lingmo-input-manager` |
| `/usr/libexec/lingmo-audio` | Audio manager daemon | `lingmo-audio-manager` |
| `/usr/libexec/lingmo-notification` | Notification service | `lingmo-notification` |
| `/usr/libexec/lingmo-osd` | On-screen display | `lingmo-osd` |
| `/usr/libexec/lingmo-runner` | Command runner (Alt+F2) | `lingmo-runner` |
| `/usr/libexec/lingmo-thumbnails` | Thumbnail generation service | `lingmo-thumbnails` |
| `/usr/libexec/lingmo-print` | Print service | `lingmo-print` |
| `/usr/libexec/lingmo-indexer` | Search indexer | `lingmo-indexer` |
| `/usr/libexec/lingmo-lockscreen` | Screen locker | `lingmo-lockscreen` |
| `/usr/libexec/lingmo-polkit-agent` | PolicyKit authentication agent | `lingmo-polkit` |
| `/usr/libexec/lingmo-network-applet` | Network monitor applet process | `lingmo-network` |
| `/usr/libexec/lingmo-bluetooth-applet` | Bluetooth monitor applet process | `lingmo-bluetooth` |
| `/usr/libexec/xdg-desktop-portal-lingmo` | Portal backend | `lingmo-portals` |

### 3.3 Helper Programs

[REQ-FS-008] Helper programs (invoked by other Lingmo components, not by users directly, and not long-running daemons) MUST be installed to `/usr/libexec/`.

| Executable | Purpose | Source Repository |
|---|---|---|
| `/usr/libexec/lingmo-apply-settings` | Apply configuration changes | `lingmo-settings-daemon` |
| `/usr/libexec/lingmo-screen-grab` | Screenshot/recording helper | `lingmo-portals` |

### 3.4 Command-Line Tools

[REQ-FS-009] Developer-facing CLI tools MUST be installed to `/usr/bin/`.

| Executable | Purpose | Source Repository |
|---|---|---|
| `/usr/bin/lingmo-inspect` | Inspect running services | `lingmo-devtools` |
| `/usr/bin/lingmo-theme-preview` | Preview theme changes | `lingmo-devtools` |
| `/usr/bin/lingmo-config-editor` | Edit config files from CLI | `lingmo-devtools` |

---

## 4. Shared Libraries

### 4.1 Public Libraries

[REQ-FS-010] Public shared libraries (exported API, linked by external consumers) MUST be installed to `/usr/lib/` (or the appropriate `$prefix/lib/` for multilib).

| Library | SONAME | Source Repository |
|---|---|---|
| `/usr/lib/libLingmoTheme.so.1` | `libLingmoTheme.so.1` | `lingmo-theme` |
| `/usr/lib/libLingmoConfig.so.1` | `libLingmoConfig.so.1` | `lingmo-config` |
| `/usr/lib/libLingmoPlugin.so.1` | `libLingmoPlugin.so.1` | `lingmo-plugin` |
| `/usr/lib/libLingmoIPC.so.1` | `libLingmoIPC.so.1` | `lingmo-ipc` |
| `/usr/lib/libLingmoUtils.so.1` | `libLingmoUtils.so.1` | `lingmo-utils` |
| `/usr/lib/libLingmoWindow.so.1` | `libLingmoWindow.so.1` | `lingmo-window` |
| `/usr/lib/libLingmoIcon.so.1` | `libLingmoIcon.so.1` | `lingmo-icon` |
| `/usr/lib/libLingmoEffects.so.1` | `libLingmoEffects.so.1` | `lingmo-effects` |

### 4.2 QStyle Plugin

[REQ-FS-011] The Lingmo QStyle plugin MUST be installed to the Qt style plugin directory:

```
/usr/lib/qt6/plugins/styles/lingmo-style.so
```

### 4.3 QML Modules

[REQ-FS-012] QML modules MUST be installed to `/usr/lib/qt6/qml/LingmoUI/`:

```
/usr/lib/qt6/qml/LingmoUI/
├── Controls/
│   ├── qmldir
│   ├── Button.qml
│   ├── libLingmoUI_Controls.so
│   └── ...
├── Window/
│   ├── qmldir
│   ├── Window.qml
│   └── libLingmoUI_Window.so
└── ...
```

[REQ-FS-013] The `LingmoTheme` QML singleton MUST be installed alongside other QML modules:

```
/usr/lib/qt6/qml/LingmoTheme/qmldir
```

### 4.4 Private Libraries

[REQ-FS-014] Libraries that are internal to a single component and NOT part of the public API MUST be installed alongside their plugins:

```
/usr/lib/lingmo/plugins/applets/liblingmo_applet_clock.so
```

[REQ-FS-015] Private shared libraries (used by multiple components within Lingmo but not exposed to external consumers) MUST be installed to `/usr/lib/lingmo/`:

```
/usr/lib/lingmo/libLingmoShellPrivate.so.1
```

---

## 5. Shared Data

### 5.1 Icons

[REQ-FS-016] Lingmo icon themes MUST be installed to `/usr/share/icons/` per the XDG Icon Theme Specification.

```
/usr/share/icons/lingmo-cursors/
/usr/share/icons/lingmo-icons/
```

### 5.2 Desktop Themes

[REQ-FS-017] Lingmo Desktop themes (color palettes, design tokens, metadata) MUST be installed to:

```
/usr/share/lingmo/themes/
├── lingmo-light/
│   ├── theme.json
│   ├── palette.json
│   └── metadata.json
├── lingmo-dark/
│   ├── theme.json
│   ├── palette.json
│   └── metadata.json
└── lingmo-high-contrast/
    └── ...
```

### 5.3 Wallpapers

[REQ-FS-018] Wallpaper images MUST be installed to:

```
/usr/share/lingmo/wallpapers/
├── default/
│   └── default.jpg
└── <additional-wallpapers>/
```

[REQ-FS-019] Per-user wallpapers MUST be stored under `$XDG_DATA_HOME/lingmo/wallpapers/`.

### 5.4 Sounds

[REQ-FS-020] Sound theme files MUST be installed to:

```
/usr/share/lingmo/sounds/
├── lingmo/
│   ├── notifications/
│   │   ├── message.oga
│   │   └── error.oga
│   ├── system/
│   │   ├── login.oga
│   │   └── logout.oga
│   └── device/
│       ├── plugged.oga
│       └── unplugged.oga
```

### 5.5 Translations

[REQ-FS-021] Translation files (`.qm` compiled from `.ts`) MUST be installed to:

```
/usr/share/lingmo/locale/
├── <appId>/
│   ├── <appId>_zh_CN.qm
│   ├── <appId>_de.qm
│   └── <appId>.qm
```

[REQ-FS-022] Shared-library translations MUST be installed under:

```
/usr/share/lingmo/locale/lingmo-apps-common/
```

### 5.6 Templates

[REQ-FS-023] Template files for new documents (used by file manager "Create New" menu) MUST be installed to:

```
/usr/share/lingmo/templates/
├── Text File.txt
├── Directory/
└── ...
```

### 5.7 Examples

[REQ-FS-024] Example code and configuration samples MUST be installed to:

```
/usr/share/lingmo/examples/
├── <component>/
│   ├── CMakeLists.txt
│   └── main.cpp
```

### 5.8 Metadata

[REQ-FS-025] Plugin metadata files (`.lingmo-plugin` JSON) MUST be installed to:

```
/usr/share/lingmo/plugins/metadata/
├── org.lingmo.applet.digital-clock.lingmo-plugin
├── org.lingmo.applet.task-manager.lingmo-plugin
└── ...
```

---

## 6. Configuration

### 6.1 System Configuration

[REQ-FS-026] System-wide configuration files (shipped by packages, read-only) MUST be installed to:

```
/etc/lingmo/
├── lingmo-theme.conf           # Default theme selection
├── lingmo-session.conf         # Session manager defaults
├── lingmo-settings-daemon.conf # Default settings
├── lingmo-power.conf           # Power management defaults
├── lingmo-display.conf         # Display management defaults
├── lingmo-input.conf           # Input device defaults
├── lingmo-audio.conf           # Audio defaults
└── xdg-desktop-portal-lingmo.conf  # Portal backend defaults
```

[REQ-FS-027] System configuration files MUST be installed by the `devel` or `runtime` component of the corresponding package. They MUST NOT be marked as conffiles in Debian if they represent default values that are overwritten by user configuration.

### 6.2 User Configuration

[REQ-FS-028] Per-user configuration MUST be stored under `$XDG_CONFIG_HOME/lingmo/`:

```
$XDG_CONFIG_HOME/lingmo/
├── lingmo-theme.conf           # User theme selection
├── lingmo-session.conf         # User session overrides
├── lingmo-shell.conf           # Shell settings
├── lingmo-settings-daemon.conf # User settings overrides
├── <appId>/
│   └── config.ini              # Per-application config
└── environment.d/
    ├── lingmo-cursor.conf      # XCURSOR_THEME, XCURSOR_SIZE
    └── lingmo-session.conf     # LINGMO_* environment variables
```

[REQ-FS-029] Application-specific configuration files MUST be placed in `$XDG_CONFIG_HOME/lingmo/<appId>/`. The `<appId>` matches the application identifier from the `.desktop` file (e.g., `file-manager`, `text-editor`).

### 6.3 Default Configuration

[REQ-FS-030] Default configuration values (shipped in packages, used when no user or system override exists) MUST be installed to:

```
/usr/share/lingmo/defaults/
├── <component>.conf
└── ...
```

### 6.4 Configuration Override Priority

[REQ-FS-031] Configuration resolution MUST follow this priority order (highest first):

1. `$XDG_CONFIG_HOME/lingmo/<name>.conf` (user override)
2. `/etc/lingmo/<name>.conf` (system override)
3. `/usr/share/lingmo/defaults/<name>.conf` (package defaults)

[REQ-FS-032] A key set in a lower-priority file MUST be overridden by the same key in a higher-priority file. Partial overrides are supported: files are NOT replaced wholesale; individual keys are merged.

### 6.5 Migration

[REQ-FS-033] When configuration file paths change between major versions, the old path MUST be symlinked or migrated automatically on first login after upgrade. Migration MUST be handled by `lingmo-settings-daemon`.

---

## 7. Runtime Data

### 7.1 Cache

[REQ-FS-034] Per-user cache data MUST be stored under `$XDG_CACHE_HOME/lingmo/`:

```
$XDG_CACHE_HOME/lingmo/
├── icon-cache/                 # Generated icon caches
├── fontconfig/                 # Fontconfig cache (fc-cache)
├── thumbnails/                 # Freedesktop thumbnail cache
│   ├── normal/
│   └── large/
└── <component>/
    └── ...                     # Component-specific caches
```

[REQ-FS-035] System-wide cache (for services that run before user login) MUST be stored under `/var/cache/lingmo/`:

```
/var/cache/lingmo/
└── ...                         # Only used if a service runs pre-session
```

### 7.2 Logs

[REQ-FS-036] Lingmo Desktop processes MUST log to the systemd journal. Process-level log files MUST NOT be written to disk under normal operation.

[REQ-FS-037] If journald is unavailable, fallback log files MUST be written to:

```
$XDG_STATE_HOME/lingmo/log/
├── session.log
├── shell-core.log
└── <component>.log
```

[REQ-FS-038] Crash reports MUST be written to:

```
$XDG_STATE_HOME/lingmo/crash/
└── <timestamp>-<appId>.crash
```

### 7.3 Sockets

[REQ-FS-039] Unix domain sockets MUST be created under `$XDG_RUNTIME_DIR/lingmo/`:

```
/run/user/$UID/lingmo/
├── shell.sock                  # Shell core ↔ surface IPC
└── ...
```

[REQ-FS-040] The `$XDG_RUNTIME_DIR/lingmo/` directory MUST be created by `lingmo-session` at session start with permissions `0700`.

### 7.4 PID Files

[REQ-FS-041] PID files MUST NOT be used. Lingmo Desktop uses systemd service management; systemd tracks PIDs automatically. Legacy PID files are FORBIDDEN.

### 7.5 Temporary Files

[REQ-FS-042] Temporary files (short-lived, process-scoped) MUST use the standard temporary directory resolved through `QStandardPaths::TempLocation` or `$TMPDIR`:

```
/tmp/lingmo-<appId>-<random>/
```

[REQ-FS-043] Lingmo-specific temporary directories MUST be prefixed with `lingmo-` to avoid collisions.

### 7.6 State Files

[REQ-FS-044] Persistent runtime state (data that must survive reboots but is not user configuration) MUST be stored under:

```
$XDG_STATE_HOME/lingmo/
├── session.state              # Session state (restored on next login)
├── shell.state                # Shell state (window positions, desktops)
└── <component>/
    └── state.bin
```

### 7.7 Locks

[REQ-FS-045] Lock files MUST be stored under `$XDG_RUNTIME_DIR/lingmo/locks/`:

```
/run/user/$UID/lingmo/locks/
├── <appId>.lock               # Single-instance lock
└── ...
```

[REQ-FS-046] Lock files MUST be automatically removed when the creating process exits. `flock()` or `QDtLock` MUST be used instead of stale-PID-detection patterns.

---

## 8. Desktop Integration

### 8.1 Desktop Files

[REQ-FS-047] Application `.desktop` files MUST be installed to `/usr/share/applications/` per the XDG Desktop Entry Specification:

```
/usr/share/applications/
├── org.lingmo.file-manager.desktop
├── org.lingmo.terminal.desktop
├── org.lingmo.text-editor.desktop
├── org.lingmo.control-center.desktop
└── ...
```

[REQ-FS-048] Desktop file identifiers MUST use reverse-domain-name format: `org.lingmo.<appId>.desktop`.

### 8.2 AppStream

[REQ-FS-049] AppStream metadata MUST be installed to `/usr/share/metainfo/`:

```
/usr/share/metainfo/
├── org.lingmo.file-manager.appdata.xml
├── org.lingmo.terminal.appdata.xml
└── ...
```

### 8.3 D-Bus

[REQ-FS-050] D-Bus service activation files MUST be installed to `/usr/share/dbus-1/services/`:

```
/usr/share/dbus-1/services/
├── org.lingmo.Notification.service
├── org.lingmo.OSD.service
├── org.lingmo.Runner.service
├── org.lingmo.Thumbnails.service
├── org.lingmo.Print.service
└── ...
```

[REQ-FS-051] D-Bus system service files (for services on the system bus) MUST be installed to `/usr/share/dbus-1/system-services/`:

```
/usr/share/dbus-1/system-services/
└── ...
```

### 8.4 PolicyKit

[REQ-FS-052] PolicyKit action definitions MUST be installed to `/usr/share/polkit-1/actions/`:

```
/usr/share/polkit-1/actions/
├── org.lingmo.policy
└── ...
```

### 8.5 systemd User Units

[REQ-FS-053] systemd user service units MUST be installed to `/usr/lib/systemd/user/`:

```
/usr/lib/systemd/user/
├── lingmo-session.service
├── lingmo-session.target
├── lingmo-shell-core.service
├── lingmo-panel@.service
├── lingmo-desktop.service
├── lingmo-overview.service
├── lingmo-settings-daemon.service
├── lingmo-power.service
├── lingmo-display.service
├── lingmo-input.service
├── lingmo-audio.service
├── lingmo-notification.service
├── lingmo-thumbnails.service
├── lingmo-print.service
├── lingmo-indexer.service
├── lingmo-polkit-agent.service
├── xdg-desktop-portal-lingmo.service
└── ...
```

[REQ-FS-054] systemd system service units (for boot-time or system-wide services) MUST be installed to `/usr/lib/systemd/system/`:

```
/usr/lib/systemd/system/
└── ... (if any)
```

### 8.6 Portals

[REQ-FS-055] Portal backend definition files MUST be installed to `/usr/share/xdg-desktop-portal/portals/`:

```
/usr/share/xdg-desktop-portal/portals/
├── lingmo.portal
└── ...
```

### 8.7 MIME Packages

[REQ-FS-056] MIME type definition files MUST be installed to `/usr/share/mime/packages/`:

```
/usr/share/mime/packages/
├── lingmo-<component>.xml
└── ...
```

### 8.8 GSettings Schemas

[REQ-FS-057] GSettings XML schemas MUST be installed to `/usr/share/glib-2.0/schemas/`:

```
/usr/share/glib-2.0/schemas/
├── org.lingmo.<component>.gschema.xml
└── ...
```

### 8.9 GTK Theme Integration

[REQ-FS-058] GTK theme files (generated or shipped for GTK application integration) MUST be installed to:

```
/usr/share/themes/Lingmo/
├── gtk-3.0/
│   └── gtk.css
├── gtk-4.0/
│   └── gtk.css
└── index.theme
```

---

## 9. Plugins

### 9.1 Plugin Library Installation

[REQ-FS-059] All Lingmo plugin types MUST be installed under `/usr/lib/lingmo/plugins/<type>/`:

```
/usr/lib/lingmo/plugins/
├── applets/
│   ├── liblingmo_applet_clock.so
│   ├── liblingmo_applet_taskmanager.so
│   └── ...
├── settings-panels/
│   ├── liblingmo_settings_appearance.so
│   ├── liblingmo_settings_keyboard.so
│   └── ...
├── search-providers/
│   ├── liblingmo_search_files.so
│   ├── liblingmo_search_apps.so
│   └── ...
├── thumbnail-providers/
│   ├── liblingmo_thumb_images.so
│   └── ...
├── content-extractors/
│   ├── liblingmo_extract_text.so
│   └── ...
├── file-preview/
│   └── ...
└── kwin-decorations/
    └── liblingmo_decoration.so
```

[REQ-FS-060] New plugin types (added in future versions) MUST follow the same pattern: `/usr/lib/lingmo/plugins/<type>/`.

[REQ-FS-061] KWin decoration plugins are an exception: they MUST be installed to KWin's plugin directory:

```
/usr/lib/qt6/plugins/org.kde.kwin.decoration/
└── liblingmo_decoration.so
```

### 9.2 Plugin Metadata

[REQ-FS-062] Plugin metadata JSON files MUST be installed to `/usr/share/lingmo/plugins/metadata/` (Section 5.8).

### 9.3 User-Installed Plugins

[REQ-FS-063] User-installed plugins (not managed by the system package manager) MUST be placed under:

```
$XDG_DATA_HOME/lingmo/plugins/
├── metadata/                   # .lingmo-plugin files
└── libraries/                  # .so files
```

### 9.4 Plugin Discovery Order

[REQ-FS-064] The plugin discovery system MUST search in this order (first match wins):

1. `$LINGMO_PLUGINS_PATH` (environment variable, development override)
2. `$XDG_DATA_HOME/lingmo/plugins/` (user-installed)
3. `/usr/lib/lingmo/plugins/` (system-installed libraries)
4. `/usr/share/lingmo/plugins/` (system-installed metadata)

---

## 10. Development Files

### 10.1 Headers

[REQ-FS-065] Public C++ headers MUST be installed to `/usr/include/Lingmo/`:

```
/usr/include/Lingmo/
├── Theme.h
├── Config.h
├── Plugin.h
├── IPC.h
├── <Module>/
│   ├── <Module>.h
│   └── ...
└── ...
```

[REQ-FS-066] Per-module headers MUST be in subdirectories per SPEC-REPO-002:

```
/usr/include/LingmoTheme/
├── LingmoTheme.h
├── LingmoThemeExport.h
├── Theme.h
└── ...
```

### 10.2 CMake Package Config

[REQ-FS-067] CMake package configuration files MUST be installed to `/usr/lib/cmake/`:

```
/usr/lib/cmake/
├── LingmoTheme/
│   ├── LingmoThemeConfig.cmake
│   ├── LingmoThemeConfigVersion.cmake
│   └── LingmoThemeTargets.cmake
├── LingmoConfig/
│   └── ...
└── ...
```

### 10.3 pkg-config

[REQ-FS-068] pkg-config files (.pc) MUST be installed to `/usr/lib/pkgconfig/`:

```
/usr/lib/pkgconfig/
├── LingmoTheme.pc
├── LingmoConfig.pc
├── LingmoPlugin.pc
├── LingmoIPC.pc
└── ...
```

[REQ-FS-069] Each `.pc` file MUST specify `Libs`, `Cflags`, and `Requires` correctly for both static and dynamic linking.

### 10.4 SDK Resources

[REQ-FS-070] SDK resource files (CMake templates, project wizards, helper scripts) MUST be installed to:

```
/usr/share/lingmo/sdk/
├── cmake/
│   ├── LingmoLibrary.cmake
│   ├── LingmoExecutable.cmake
│   └── LingmoPlugin.cmake
├── templates/
│   ├── app-widgets/
│   ├── app-qml/
│   └── plugin-applet/
└── ...
```

### 10.5 Documentation

[REQ-FS-071] API documentation MUST be installed to:

```
/usr/share/doc/lingmo/<component>/
├── api/html/
│   └── index.html
└── ...
```

[REQ-FS-072] Package-level documentation (README, changelog, copyright) MUST follow distribution conventions:

| Distribution | Location |
|---|---|
| Debian | `/usr/share/doc/lingmo-<component>/` |
| Fedora | `/usr/share/doc/lingmo-<component>/` |

---

## 11. Security

### 11.1 Writable Directories

[REQ-FS-073] The following directories MUST be writable by the owning user only:

| Directory | Permissions | Ownership |
|---|---|---|
| `$XDG_CONFIG_HOME/lingmo/` | `0700` | User |
| `$XDG_CACHE_HOME/lingmo/` | `0700` | User |
| `$XDG_STATE_HOME/lingmo/` | `0700` | User |
| `$XDG_RUNTIME_DIR/lingmo/` | `0700` | User |

[REQ-FS-074] The following directories MUST be writable only by root:

| Directory | Permissions | Ownership |
|---|---|---|
| `/etc/lingmo/` | `0755` | `root:root` |
| `/usr/lib/lingmo/` | `0755` | `root:root` |
| `/usr/share/lingmo/` | `0755` | `root:root` |

### 11.2 Read-Only Directories

[REQ-FS-075] The following directories MUST NOT be written to by any Lingmo process at runtime:

| Directory | Rationale |
|---|---|
| `/usr/bin/` | System-managed executables |
| `/usr/lib/` | System-managed libraries |
| `/usr/include/` | System-managed headers |
| `/usr/share/applications/` | System-managed desktop entries |
| `/usr/share/icons/` | System-managed icon themes |

### 11.3 Executable Permissions

[REQ-FS-076] Executables in `/usr/bin/` and `/usr/libexec/` MUST have permissions `0755`.

[REQ-FS-077] Shared libraries MUST have permissions `0644`. They MUST NOT have the executable bit set.

[REQ-FS-078] Plugin libraries MUST have permissions `0644`. They MUST NOT have the executable bit set.

### 11.4 Runtime Isolation

[REQ-FS-079] The `$XDG_RUNTIME_DIR/lingmo/` directory MUST be created with permissions `0700` and ownership `$UID:$UID`. No other user may access Lingmo runtime files.

[REQ-FS-080] Socket files under `$XDG_RUNTIME_DIR/lingmo/` MUST be created with permissions `0600`.

[REQ-FS-081] Lock files under `$XDG_RUNTIME_DIR/lingmo/locks/` MUST be created with permissions `0600`.

---

## 12. Compatibility

### 12.1 Debian Differences

[REQ-FS-082] On Debian, the following paths differ from the generic FHS:

| Generic Path | Debian Path | Reason |
|---|---|---|
| `/usr/libexec/` | `/usr/libexec/` (same) | Debian ≥ 12 uses `/usr/libexec/` |
| `/usr/lib/cmake/` | `/usr/lib/cmake/` (same) | CMake's default |
| `/usr/lib/pkgconfig/` | `/usr/lib/pkgconfig/` (same) | pkg-config default |

No substantive differences for Lingmo Desktop on Debian ≥ 12. For Debian 11 (obsolete), `/usr/libexec/` did not exist; packages SHOULD fall back to `/usr/lib/` for executables.

### 12.2 Fedora Differences

[REQ-FS-083] On Fedora, the following paths differ from Debian:

| Debian Path | Fedora Path | Notes |
|---|---|---|
| `/usr/lib/qt6/plugins/styles/` | `/usr/lib64/qt6/plugins/styles/` | Fedora uses `/usr/lib64/` for 64-bit libraries |
| `/usr/lib/lingmo/plugins/` | `/usr/lib64/lingmo/plugins/` | Same multilib distinction |
| `/usr/lib/cmake/` | `/usr/lib64/cmake/` | 64-bit cmake modules |

[REQ-FS-084] The Fedora `%_libdir` macro expands to `/usr/lib64/` on x86_64 and `/usr/lib/` on x86. CMake packages MUST use `${CMAKE_INSTALL_LIBDIR}` which resolves correctly on both.

### 12.3 Multiarch Considerations

[REQ-FS-085] On Debian multiarch systems, library paths MUST use the architecture-qualified triplet:

```
/usr/lib/<arch-triplet>/
```

[REQ-FS-086] Multiarch libraries (e.g., 32-bit compatibility layers) MUST NOT ship Lingmo Desktop plugins. All Lingmo plugins are 64-bit only on x86_64.

### 12.4 Future Compatibility

[REQ-FS-087] All Lingmo-managed directories under `/usr/` MUST use a `lingmo/` subdirectory. This ensures that Lingmo paths do not collide with other projects' files when installed on the same system.

[REQ-FS-088] No Lingmo package MAY install files directly into `/usr/share/` without a `lingmo/`, `Lingmo/`, or similarly namespaced subdirectory, except for standardized locations that require otherwise (desktop files, AppStream, D-Bus, systemd, icons, MIME, GSettings, PolicyKit). Those standardized locations are enumerated in Section 8.

---

## 13. Compliance Checklist

| ID | Requirement | Verification Method |
|---|---|---|
| REQ-FS-001 | FHS 3.0 compliance | Package lintian/rpmlint check |
| REQ-FS-002 | Paths use GNUInstallDirs variables | CMake install rule scan |
| REQ-FS-003 | Deviations documented and justified | Manual review |
| REQ-FS-004 | XDG Base Dir for per-user paths | Source scan for hardcoded paths |
| REQ-FS-006 | User executables in /usr/bin/ | Package file listing |
| REQ-FS-007 | Internal executables in /usr/libexec/ | Package file listing |
| REQ-FS-010 | Public libs in /usr/lib/ | Package file listing |
| REQ-FS-011 | QStyle in Qt plugin dir | Package file listing |
| REQ-FS-012 | QML modules under /usr/lib/qt6/qml/LingmoUI/ | Package file listing |
| REQ-FS-016 | Icons in /usr/share/icons/ | Package file listing |
| REQ-FS-017 | Themes in /usr/share/lingmo/themes/ | Package file listing |
| REQ-FS-020 | Sounds in /usr/share/lingmo/sounds/ | Package file listing |
| REQ-FS-021 | Translations in /usr/share/lingmo/locale/ | Package file listing |
| REQ-FS-026 | System config in /etc/lingmo/ | Package file listing |
| REQ-FS-028 | User config under $XDG_CONFIG_HOME/lingmo/ | Source scan for config paths |
| REQ-FS-031 | Config priority order (user > system > defaults) | Code review of config loader |
| REQ-FS-034 | Cache under $XDG_CACHE_HOME/lingmo/ | Source scan |
| REQ-FS-036 | Logging via journald | Source scan |
| REQ-FS-039 | Sockets under $XDG_RUNTIME_DIR/lingmo/ | Source scan |
| REQ-FS-041 | PID files forbidden | Grep for PID file patterns |
| REQ-FS-044 | State under $XDG_STATE_HOME/lingmo/ | Source scan |
| REQ-FS-045 | Locks under $XDG_RUNTIME_DIR/lingmo/locks/ | Source scan |
| REQ-FS-047 | Desktop files in /usr/share/applications/ | Package file listing |
| REQ-FS-049 | AppStream in /usr/share/metainfo/ | Package file listing |
| REQ-FS-050 | D-Bus services in /usr/share/dbus-1/services/ | Package file listing |
| REQ-FS-053 | systemd user units in /usr/lib/systemd/user/ | Package file listing |
| REQ-FS-055 | Portal defs in /usr/share/xdg-desktop-portal/portals/ | Package file listing |
| REQ-FS-059 | Plugins in /usr/lib/lingmo/plugins/<type>/ | Package file listing |
| REQ-FS-061 | KWin decorations in KWin plugin dir | Package file listing |
| REQ-FS-064 | Plugin discovery order | Code review of plugin loader |
| REQ-FS-065 | Headers in /usr/include/Lingmo/ | Package file listing |
| REQ-FS-067 | CMake config in /usr/lib/cmake/ | Package file listing |
| REQ-FS-073 | User dirs 0700 | Package postinst check |
| REQ-FS-074 | System dirs 0755 root | Package file listing |
| REQ-FS-076 | Executables 0755 | Package file listing |
| REQ-FS-077 | Libraries 0644 | Package file listing |
| REQ-FS-079 | Runtime dir 0700 $UID | Unit test |
| REQ-FS-082 | Debian paths correct | Package build test on Debian |
| REQ-FS-084 | Fedora paths use %_libdir | Package build test on Fedora |
| REQ-FS-087 | All /usr/ paths under lingmo/ namespace | Package file listing scan |

---

## Appendix A: Complete Installation Tree

```
/usr/
├── bin/                                  # User-facing executables
│   ├── lingmo-control-center
│   ├── lingmo-file-manager
│   ├── lingmo-terminal
│   ├── lingmo-text-editor
│   ├── lingmo-image-viewer
│   ├── lingmo-software
│   └── ...
├── lib/                                  # Shared libraries
│   ├── libLingmoTheme.so.1
│   ├── libLingmoConfig.so.1
│   ├── libLingmoPlugin.so.1
│   ├── libLingmoIPC.so.1
│   ├── libLingmoUtils.so.1
│   ├── libLingmoWindow.so.1
│   ├── libLingmoIcon.so.1
│   ├── libLingmoEffects.so.1
│   ├── cmake/                            # CMake config files
│   │   ├── LingmoTheme/
│   │   ├── LingmoConfig/
│   │   └── ...
│   ├── pkgconfig/                        # pkg-config files
│   │   ├── LingmoTheme.pc
│   │   └── ...
│   ├── qt6/
│   │   ├── plugins/
│   │   │   ├── styles/lingmo-style.so
│   │   │   └── org.kde.kwin.decoration/liblingmo_decoration.so
│   │   └── qml/
│   │       └── LingmoUI/
│   │           ├── Controls/
│   │           ├── Window/
│   │           └── ...
│   └── lingmo/
│       ├── libLingmoShellPrivate.so.1
│       └── plugins/
│           ├── applets/
│           ├── settings-panels/
│           ├── search-providers/
│           ├── thumbnail-providers/
│           ├── content-extractors/
│           └── file-preview/
├── libexec/                              # Internal executables
│   ├── lingmo-session
│   ├── lingmo-shell-core
│   ├── lingmo-panel
│   ├── lingmo-desktop
│   ├── lingmo-overview
│   ├── lingmo-settings-daemon
│   ├── lingmo-power
│   ├── lingmo-display
│   ├── lingmo-input
│   ├── lingmo-audio
│   ├── lingmo-notification
│   ├── lingmo-osd
│   ├── lingmo-runner
│   ├── lingmo-thumbnails
│   ├── lingmo-print
│   ├── lingmo-indexer
│   ├── lingmo-lockscreen
│   ├── lingmo-polkit-agent
│   ├── lingmo-network-applet
│   ├── lingmo-bluetooth-applet
│   └── xdg-desktop-portal-lingmo
├── include/                              # Public headers
│   ├── Lingmo/
│   │   ├── Theme.h
│   │   ├── Config.h
│   │   └── ...
│   └── LingmoTheme/
│       ├── Theme.h
│       └── ...
└── share/                                # Architecture-independent data
    ├── applications/                     # Desktop files
    │   ├── org.lingmo.file-manager.desktop
    │   └── ...
    ├── metainfo/                         # AppStream metadata
    │   └── org.lingmo.file-manager.appdata.xml
    ├── dbus-1/
    │   └── services/                     # D-Bus service activation
    ├── polkit-1/
    │   └── actions/                      # PolicyKit definitions
    ├── systemd/
    │   └── user/                         # User service units
    ├── xdg-desktop-portal/
    │   └── portals/                      # Portal backend defs
    ├── mime/
    │   └── packages/                     # MIME type defs
    ├── glib-2.0/
    │   └── schemas/                      # GSettings schemas
    ├── icons/                            # Icon themes
    │   ├── lingmo-cursors/
    │   └── lingmo-icons/
    ├── themes/                           # GTK themes
    │   └── Lingmo/
    └── lingmo/                           # Lingmo-specific data
        ├── themes/                       # Desktop themes
        ├── wallpapers/                   # Wallpaper images
        ├── sounds/                       # Sound themes
        ├── locale/                       # Translations
        ├── templates/                    # Document templates
        ├── examples/                     # Code examples
        ├── defaults/                     # Default configs
        ├── plugins/
        │   └── metadata/                 # Plugin metadata JSON
        └── sdk/                          # SDK resources
            ├── cmake/
            └── templates/

/etc/
└── lingmo/                               # System configuration
    ├── lingmo-theme.conf
    ├── lingmo-session.conf
    ├── lingmo-settings-daemon.conf
    ├── lingmo-power.conf
    ├── lingmo-display.conf
    ├── lingmo-input.conf
    ├── lingmo-audio.conf
    └── xdg-desktop-portal-lingmo.conf
```

---

## Appendix B: Repository-to-Path Mapping

| Repository | Binaries | Libraries | Data |
|---|---|---|---|
| `lingmo-session` | `/usr/libexec/lingmo-session` | — | — |
| `lingmo-shell-core` | `/usr/libexec/lingmo-shell-core` | — | — |
| `lingmo-shell` | `/usr/libexec/lingmo-panel`, `/usr/libexec/lingmo-desktop`, `/usr/libexec/lingmo-overview` | — | — |
| `lingmo-settings-daemon` | `/usr/libexec/lingmo-settings-daemon`, `/usr/libexec/lingmo-apply-settings` | — | `/etc/lingmo/lingmo-settings-daemon.conf` |
| `lingmo-power-manager` | `/usr/libexec/lingmo-power` | — | `/etc/lingmo/lingmo-power.conf` |
| `lingmo-display-manager` | `/usr/libexec/lingmo-display` | — | `/etc/lingmo/lingmo-display.conf` |
| `lingmo-input-manager` | `/usr/libexec/lingmo-input` | — | `/etc/lingmo/lingmo-input.conf` |
| `lingmo-audio-manager` | `/usr/libexec/lingmo-audio` | — | `/etc/lingmo/lingmo-audio.conf` |
| `lingmo-theme` | — | `/usr/lib/libLingmoTheme.so.1` | `/usr/share/lingmo/themes/` |
| `lingmo-config` | — | `/usr/lib/libLingmoConfig.so.1` | — |
| `lingmo-plugin` | — | `/usr/lib/libLingmoPlugin.so.1` | — |
| `lingmo-ipc` | — | `/usr/lib/libLingmoIPC.so.1` | — |
| `lingmo-utils` | — | `/usr/lib/libLingmoUtils.so.1` | — |
| `lingmo-window` | — | `/usr/lib/libLingmoWindow.so.1` | — |
| `lingmo-icon` | — | `/usr/lib/libLingmoIcon.so.1` | `/usr/share/icons/lingmo-icons/` |
| `lingmo-effects` | — | `/usr/lib/libLingmoEffects.so.1` | — |
| `lingmo-style` | — | `/usr/lib/qt6/plugins/styles/lingmo-style.so` | — |
| `lingmo-controls` | — | `/usr/lib/qt6/qml/LingmoUI/Controls/libLingmoUI_Controls.so` | — |
| `lingmo-notification` | `/usr/libexec/lingmo-notification` | — | — |
| `lingmo-osd` | `/usr/libexec/lingmo-osd` | — | — |
| `lingmo-runner` | `/usr/libexec/lingmo-runner` | — | — |
| `lingmo-thumbnails` | `/usr/libexec/lingmo-thumbnails` | — | — |
| `lingmo-print` | `/usr/libexec/lingmo-print` | — | — |
| `lingmo-indexer` | `/usr/libexec/lingmo-indexer` | — | — |
| `lingmo-lockscreen` | `/usr/libexec/lingmo-lockscreen` | — | — |
| `lingmo-polkit` | `/usr/libexec/lingmo-polkit-agent` | — | `/usr/share/polkit-1/actions/` |
| `lingmo-portals` | `/usr/libexec/xdg-desktop-portal-lingmo` | — | `/usr/share/xdg-desktop-portal/portals/` |
| `lingmo-network` | `/usr/libexec/lingmo-network-applet` | — | — |
| `lingmo-bluetooth` | `/usr/libexec/lingmo-bluetooth-applet` | — | — |
| `lingmo-file-manager` | `/usr/bin/lingmo-file-manager` | — | — |
| `lingmo-terminal` | `/usr/bin/lingmo-terminal` | — | — |
| `lingmo-text-editor` | `/usr/bin/lingmo-text-editor` | — | — |
| `lingmo-image-viewer` | `/usr/bin/lingmo-image-viewer` | — | — |
| `lingmo-control-center` | `/usr/bin/lingmo-control-center` | — | — |
| `lingmo-software` | `/usr/bin/lingmo-software` | — | — |
| `lingmo-devtools` | `/usr/bin/lingmo-inspect`, `/usr/bin/lingmo-theme-preview`, `/usr/bin/lingmo-config-editor` | — | `/usr/share/lingmo/sdk/` |

---

## Appendix C: FHS Comparison Table

| Path | FHS 3.0 | Lingmo | Status |
|---|---|---|---|
| `/usr/bin/` | User executables | User-facing Lingmo executables | Compliant |
| `/usr/libexec/` | Not required | Internal Lingmo executables | Deviation (de-facto standard) |
| `/usr/lib/` | Libraries | Lingmo shared libraries | Compliant |
| `/usr/lib/<triplet>/` | Multiarch | Architecture-specific libs | Compliant |
| `/usr/include/` | Headers | Lingmo public headers | Compliant |
| `/usr/share/` | Arch-independent data | Lingmo data, themes, icons | Compliant |
| `/etc/` | Host-specific config | System-wide Lingmo config | Compliant |
| `/var/cache/` | Persistent cache | Pre-session cache data | Compliant |
| `/run/` | Runtime variable data | Session runtime files | Compliant |
| `/tmp/` | Temporary files | Process-scoped temp files | Compliant |

---

*Cross-ref*: ARCHITECTURE-v3.md, SPEC-REPO-001 (Repository Layout), SPEC-REPO-002 (Header Organization), SPEC-REPO-003 (CMake Organization), SPEC-CORE-001 (Naming Convention), SPEC-CORE-002 (Authoring Standard)
