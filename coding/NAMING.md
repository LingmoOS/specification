# Naming Convention Specification

| | |
|---|---|
| **Document ID** | SPEC-CORE-001 |
| **Title** | Naming Convention Specification |
| **Version** | 1.0.0 |
| **Status** | Stable |
| **Applies to** | All Lingmo repositories and artifacts |
| **Supersedes** | SPEC-REPO-001 Sections 2 and 6.2–6.3 |
| **Cross-ref** | SPEC-REPO-001 (Repository Layout), SPEC-LINGMOUI-001 (API Design) |

---

## 1. General Principles

### 1.1 Primary Principles

The following principles govern all naming decisions in the Lingmo ecosystem:

**P1 — Readability**. Readability takes precedence over brevity. A name that
must be abbreviated is a name that was not well chosen.

**P2 — Consistency**. The same concept MUST have the same name in every
repository, language, and protocol. If a feature is called `activeWindow` in
C++, it MUST be called `activeWindow` in QML and `ActiveWindow` in D-Bus
(following each language's casing rules).

**P3 — Stability**. A public name, once released, MUST NOT be renamed. If a
name proves problematic, the old name MUST be deprecated and the new name
introduced alongside it for at least two major versions.

**P4 — Uniqueness**. Every named entity MUST have exactly one canonical name
within its scope. Aliases are permitted for backward compatibility but MUST
be clearly documented as deprecated.

**P5 — Self-description**. A name SHOULD convey the purpose or semantics of
the named entity without requiring context beyond the scope. `FileManager`
is better than `Manager`, `NetworkManager` is better than `Nm`.

**P6 — Avoiding abbreviations**. Abbreviations MUST be avoided unless the
abbreviated form is more widely recognized than the full form (e.g., `IPC`,
`DBus`, `URL`, `HTML`). The approved abbreviations list in Section 13.1 is
the exclusive set of permitted abbreviations.

**P7 — Vendor prefixing**. The `Lingmo` vendor prefix MUST be used where
visibility into shared namespaces is required (D-Bus, desktop entries,
environment variables). The prefix MUST NOT be used where the namespace is
already project-scoped (C++ within the `Lingmo` namespace, QML within a
`LingmoUI` module).

### 1.2 Terminology

Throughout this specification, the following terminology is used:

| Term | Meaning |
|------|---------|
| PascalCase | First letter of each word capitalized: `ThisIsPascalCase` |
| camelCase | First word lowercase, subsequent words capitalized: `thisIsCamelCase` |
| snake_case | Words separated by underscores, all lowercase: `this_is_snake_case` |
| SCREAMING_SNAKE_CASE | Words separated by underscores, all uppercase: `THIS_IS_SCREAMING` |
| kebab-case | Words separated by hyphens, all lowercase: `this-is-kebab-case` |
| Upper-Kebab-Case | Words separated by hyphens, first letter of each word uppercase: `This-Is-Upper-Kebab` |

---

## 2. Repository Names

### 2.1 Format

Repository names are specified in SPEC-REPO-001 Section 2. The canonical
rule is:

```
lingmo-<component>
```

Where `<component>`:
- MUST consist of lowercase letters (`a-z`), digits (`0-9`), and hyphens (`-`)
- MUST NOT begin or end with a hyphen
- MUST NOT contain consecutive hyphens
- MUST NOT exceed 40 characters
- MUST NOT be a reserved name (listed in SPEC-REPO-001 Section 2.3)

### 2.2 Component Derivation from Repository Purpose

The component name SHOULD be derived from the primary artifact:

| Primary Artifact | Component Name Example |
|-----------------|----------------------|
| Shared library `LingmoTheme` | `lingmo-theme` |
| Executable `lingmo-shell-core` | `lingmo-shell-core` |
| Plugin directory `applets/clock` | `lingmo-applet-clock` |

### 2.3 Forbidden Name Patterns

The following patterns MUST NOT appear in repository names:

| Pattern | Example (Invalid) | Reason |
|---------|------------------|--------|
| Underscore | `lingmo_shell` | Hyphen-only |
| Mixed case | `lingmo-Shell` | Lowercase-only |
| Digits at start | `lingmo-2d` | Letter-start required |
| Consecutive hyphens | `lingmo-shell--core` | Single hyphens only |
| Trailing hyphen | `lingmo-shell-` | Incomplete component |
| Prefix other than `lingmo-` | `kde-theme` | Wrong vendor |

---

## 3. Library Names

### 3.1 Shared Library File Name

Shared library file names MUST follow the pattern:

```
libLingmo<Module>.so.<major>.<minor>.<patch>
```

| Part | Rule |
|------|------|
| `lib` | Literal prefix (Unix convention) |
| `Lingmo` | Literal vendor prefix |
| `<Module>` | PascalCase module name (e.g., `Theme`, `Config`) |
| `.<major>.<minor>.<patch>` | Version suffix (semver) |

**Examples**:
- `libLingmoTheme.so.1.2.0`
- `libLingmoConfig.so.1.0.3`

### 3.2 SONAME

The SONAME MUST be:

```
libLingmo<Module>.so.<major>
```

**Examples**:
- `libLingmoTheme.so.1`
- `libLingmoConfig.so.1`

### 3.3 CMake Package Name

The CMake `find_package` name MUST be:

```
Lingmo<Module>
```

| Repository | CMake Package Name | `find_package` Call |
|------------|-------------------|---------------------|
| `lingmo-theme` | `LingmoTheme` | `find_package(LingmoTheme REQUIRED)` |
| `lingmo-config` | `LingmoConfig` | `find_package(LingmoConfig REQUIRED)` |

### 3.4 CMake Target Name

The CMake target (with namespace) MUST be:

```
Lingmo::<Module>
```

| Repository | CMake Target | Linking |
|------------|-------------|---------|
| `lingmo-theme` | `Lingmo::Theme` | `target_link_libraries(myapp PRIVATE Lingmo::Theme)` |
| `lingmo-config` | `Lingmo::Config` | `target_link_libraries(myapp PRIVATE Lingmo::Config)` |

### 3.5 pkg-config Name

The pkg-config name MUST be:

```
Lingmo<Module>
```

File: `Lingmo<Module>.pc`

### 3.6 Static Library

Static library files MUST follow the same naming as shared libraries, with
the exception of the suffix:

```
libLingmo<Module>.a
```

Static libraries MUST be suffixed with `_static` in the CMake target name:

```
Lingmo::<Module>Static
```

---

## 4. C++ Naming

### 4.1 Namespaces

All Lingmo C++ declarations MUST be in the `Lingmo` namespace.

```cpp
namespace Lingmo {
    // All public API lives here
}
```

Nested namespaces:

| Pattern | Usage | Example |
|---------|-------|---------|
| `Lingmo` | All public API | `Lingmo::Theme` |
| `Lingmo::Detail` | Internal implementation (not public) | `Lingmo::Detail::hashHelper` |
| `Lingmo::Plugin` | Plugin interface types | `Lingmo::Plugin::Applet` |

The `Lingmo::Detail` namespace MUST NOT be relied upon by external code.
It provides no stability guarantee.

### 4.2 Classes

Class names MUST use PascalCase with the `Lingmo` prefix.

```cpp
class LingmoTheme;
class LingmoConfig;
class LingmoShellCore;
```

The `Lingmo` prefix is REQUIRED for all classes that are part of a public
API. It is OPTIONAL for private implementation classes (which are in the
`Detail` namespace or have no external visibility).

Classes that inherit from QObject MUST:
- End with the class name (no `Object` suffix)
- Declare `Q_OBJECT` in the private section

```cpp
class LingmoTheme : public QObject {
    Q_OBJECT
public:
    // ...
};
```

### 4.3 Structs

Structs MUST use PascalCase. Structs SHOULD only be used for passive data
containers (POD or near-POD types). Structs with methods that maintain
invariants SHOULD be classes instead.

```cpp
struct ColorPalette {
    QColor background;
    QColor foreground;
    QColor accent;
};
```

Structs MUST NOT have constructors or destructors unless required for
initialization. Structs MUST NOT have virtual methods.

### 4.4 Enumerations

Enumerations MUST use `enum class` (scoped enumerations). Plain `enum` MUST
NOT be used.

```cpp
enum class WindowState {
    Normal,
    Minimized,
    Maximized,
    FullScreen
};
```

Enum values MUST use PascalCase.
Enum type names MUST use PascalCase.

Enum values MUST NOT include the enum type name as a prefix.
Qt-style prefixes (like `WindowStateNormal`) MUST NOT be used.

```cpp
// CORRECT
enum class ColorRole { Window, Button, Accent };
// INCORRECT
enum class ColorRole { ColorRoleWindow, ColorRoleButton };
```

### 4.5 Functions and Methods

Free functions and member methods MUST use camelCase.

```cpp
class LingmoTheme {
public:
    QColor accentColor() const;
    void setAccentColor(const QColor& color);
};

namespace Detail {
    QByteArray generateHash(const QString& input);
}
```

### 4.6 Getters and Setters

Getter methods MUST use the property name without `get` prefix:

```cpp
QColor accentColor() const;
int spacing() const;
```

Setter methods MUST use the `set` prefix:

```cpp
void setAccentColor(const QColor& color);
void setSpacing(int pixels);
```

Boolean getters MAY use the `is` or `has` prefix:

```cpp
bool isEnabled() const;
bool hasSelection() const;
```

### 4.7 Variables

Local variables MUST use camelCase.

```cpp
int windowCount = 0;
QString currentFilePath;
```

Member variables MUST NOT use any prefix. Disambiguation from local
variables is achieved through `this->` when necessary.

```cpp
class LingmoTheme {
    QColor accentColor;  // NOT m_accentColor, NOT accentColor_
    int spacing;
};
```

Rationale: The `m_` prefix is a legacy Qt 4 convention. Modern C++ code in
the Lingmo ecosystem omits it, relying on `this->` for disambiguation.
This matches the Qt 6 style and reduces visual noise.

Static member variables MUST NOT use any prefix.

```cpp
class LingmoTheme {
    static Theme* s_instance;  // ALLOWED (s_ prefix for statics)
};
```

The `s_` prefix MAY be used for static member variables to distinguish
them from instance variables.

### 4.8 Constants

Constants MUST be declared as `constexpr` or `const`.

Named constants (file-scope or class-scope) MUST use PascalCase:

```cpp
constexpr int DefaultIconSize = 16;
constexpr QStringView ProductName = u"Lingmo";
```

Rationale: PascalCase for constants follows the pattern established by
Qt 6 (e.g., `Qt::Key_Escape`, `QWindow::Animated`).

### 4.9 Templates

Template type parameters MUST use PascalCase. Single uppercase letters
(`T`, `U`, `V`) are acceptable for simple generic code.

```cpp
template<typename T>
T clamp(const T& value, const T& min, const T& max);

template<typename ValueType>
class Cache {
    ValueType get(const QString& key);
};
```

Template template parameters MUST use PascalCase with a `Tag` suffix.

### 4.10 Macros

Preprocessor macros MUST use SCREAMING_SNAKE_CASE with the `LINGMO_` prefix.

```cpp
#define LINGMO_VERSION_MAJOR 1
#define LINGMO_VERSION_MINOR 0
#define LINGMO_DEPRECATED [[deprecated]]
```

Macros that control export visibility MUST follow the pattern:

```
LINGMO<MODULE>_EXPORT
```

Where `<MODULE>` is the uppercased module name without the `Lingmo` prefix.

```cpp
#define LINGMOTHEME_EXPORT Q_DECL_EXPORT
#define LINGMOCONFIG_EXPORT Q_DECL_EXPORT
```

Macros that are internal to a single repository MUST use the repository's
module name:

```cpp
// In lingmo-shell-core:
#define LINGMOSHELLCORE_EXPORT Q_DECL_EXPORT
```

### 4.11 Private Implementation Classes

Private implementation classes (d-pointer) MUST follow:

- Class name: `<PublicClass>Private` (PascalCase)
- Header file: `<PublicClass>_p.h`
- The `Private` suffix MUST be appended directly to the public class name
- The private class MUST be declared in the `Lingmo` namespace

```cpp
// LingmoTheme_p.h
namespace Lingmo {
class LingmoThemePrivate {
public:
    QColor accentColor;
    int spacing;
    // ...
};
}
```

The public class MUST hold the private implementation as a
`QScopedPointer` or `std::unique_ptr`:

```cpp
// LingmoTheme.h
class LingmoTheme {
    // ...
private:
    QScopedPointer<LingmoThemePrivate> d;
};
```

### 4.12 Files

C++ source files MUST be named according to the primary class they contain:

| Entity | File Name |
|--------|-----------|
| Class `LingmoTheme` | `LingmoTheme.h`, `LingmoTheme.cpp` |
| Class `LingmoTheme` private | `LingmoTheme_p.h`, `LingmoTheme_p.cpp` |
| Namespace `Detail` helpers | `detail/hash_helpers.h` |
| Test for `LingmoTheme` | `tst_LingmoTheme.cpp` |
| Benchmark for icon loading | `bm_IconLoading.cpp` |

Public headers MUST use PascalCase matching the class name.
Private headers MUST use the `_p.h` suffix.
Internal detail headers MUST use snake_case and reside in a `detail/`
directory.

### 4.13 Directories

Source directories MUST use camelCase or PascalCase matching the module
structure.

```
src/
├── LingmoTheme.cpp
├── LingmoTheme_p.h
├── detail/
│   ├── color_utils.h
│   └── font_metrics.h
├── LingmoConfig.cpp
└── LingmoConfig_p.h
```

---

## 5. QML Naming

### 5.1 Module URI

QML module URIs MUST use PascalCase with dot separators.

```
LingmoUI.Controls
LingmoUI.Window
LingmoUI.Effects
LingmoCore
```

The module URI MUST begin with `LingmoUI` for UI components or `Lingmo`
for non-UI components.

### 5.2 QML Types

QML type names MUST use PascalCase.

```qml
Button
TextField
ComboBox
Switch
```

QML types that wrap a C++ class SHOULD use the same name as the C++ class,
minus the `Lingmo` prefix (if applicable), unless ambiguity would result.

### 5.3 Attached Properties

Attached property type names MUST use PascalCase.

```qml
Item {
    LingmoTheme.palette: ...
}
```

### 5.4 Singletons

Singleton type names MUST use PascalCase.

```qml
pragma Singleton
QtObject {
    id: LingmoTheme
    // ...
}
```

### 5.5 Properties

QML property names MUST use camelCase.

```qml
property int iconSize: 16
property color accentColor: "#3daee9"
property bool isEnabled: true
```

### 5.6 Signals

Signal names MUST use camelCase. Signals that indicate a state change
SHOULD use past tense:

```qml
signal clicked()
signal toggled()
signal activated()
```

### 5.7 Object IDs

QML `id` values MUST use camelCase.

```qml
Button {
    id: submitButton
}

TextField {
    id: userNameInput
}
```

The `id` SHOULD reflect the type of the object (e.g., `submitButton` for
a `Button`, `userNameInput` for a `TextField`).

### 5.8 Enum Declarations in QML

Enums in QML MUST use PascalCase for the type and PascalCase for values.

```qml
enum class Status {
    Ready,
    Loading,
    Error
};
```

### 5.9 QML File Names

QML files that define a public type MUST use PascalCase matching the
type name:

```
Button.qml
TextField.qml
ComboBox.qml
```

QML files that are internal helpers MUST use camelCase or a leading
lowercase letter:

```
buttonHelper.qml
privateUtils.js
```

### 5.10 JavaScript Files

JavaScript files within QML modules MUST use camelCase:

```
dateUtils.js
stringFormatter.js
colorParser.js
```

---

## 6. Rust Naming

### 6.1 Crates

Rust crate names MUST use kebab-case without the `lingmo-` prefix (the
prefix is implied by the repository).

| Repository | Crate Name |
|------------|-----------|
| `lingmo-search-engine` | `search-engine` |
| `lingmo-portals` | `portals` |

Crate names MUST match the last two segments of the repository name
(component only, without `lingmo-`).

### 6.2 Modules

Module names MUST use snake_case.

```rust
mod file_indexer;
mod query_parser;
mod search_result;
```

### 6.3 Traits

Trait names MUST use PascalCase.

```rust
pub trait SearchProvider {
    fn search(&self, query: &str) -> Vec<SearchResult>;
}
```

### 6.4 Structs

Struct names MUST use PascalCase.

```rust
pub struct SearchResult {
    pub id: String,
    pub title: String,
    pub relevance: f64,
}
```

### 6.5 Enums

Enum names MUST use PascalCase. Enum variants MUST use PascalCase.

```rust
pub enum IndexerState {
    Idle,
    Indexing { progress: f64 },
    Paused,
    Error(String),
}
```

### 6.6 Functions

Function names MUST use snake_case.

```rust
pub fn create_index(path: &Path) -> Result<Index, IndexerError>;
pub fn search_query(index: &Index, query: &str) -> Vec<SearchResult>;
```

### 6.7 Constants

Constants MUST use SCREAMING_SNAKE_CASE.

```rust
pub const MAX_RESULTS: usize = 100;
pub const DEFAULT_INDEX_PATH: &str = "/tmp/lingmo-search-index";
```

---

## 7. Go Naming

### 7.1 Packages

Package names MUST use lowercase, single-word names.

```go
package service
package dbus
package config
```

Multi-word package names MUST use concatenation without separators:

```go
package servicemanager   // NOT service_manager, NOT service-manager
package powerprofile
```

### 7.2 Exported Names

Exported names (visible outside the package) MUST use PascalCase.

```go
type PowerManager struct { ... }
func NewPowerManager() *PowerManager { ... }
const MaxBatteryPercent = 100
```

### 7.3 Unexported Names

Unexported names (package-private) MUST use camelCase.

```go
type batteryState struct { ... }
func readDeviceInfo() (*DeviceInfo, error) { ... }
```

### 7.4 Interfaces

Interface names MUST use PascalCase. Single-method interfaces SHOULD
use the method name with the `-er` suffix convention.

```go
type PowerProvider interface {
    BatteryPercent() (float64, error)
}

// Single-method interface:
type Stringer interface {
    String() string
}
```

### 7.5 File Names

Go file names MUST use snake_case:

```
power_manager.go
battery_monitor.go
```

Test files MUST follow Go convention:

```
power_manager_test.go
```

---

## 8. D-Bus Naming

### 8.1 Reverse Domain

The Lingmo reverse domain for D-Bus is:

```
org.lingmo
```

All Lingmo D-Bus identifiers MUST use this domain.

### 8.2 Service Names

D-Bus service names MUST follow the pattern:

```
org.lingmo.<Component>
```

| Component | Service Name |
|-----------|-------------|
| Shell core | `org.lingmo.Shell` |
| Settings daemon | `org.lingmo.SettingsDaemon` |
| Power manager | `org.lingmo.Power` |
| Audio manager | `org.lingmo.Audio` |

Service names MUST use PascalCase for each segment after `org.lingmo.`.

### 8.3 Interface Names

D-Bus interface names MUST follow the pattern:

```
org.lingmo.<Component>.<Interface>
```

| Component | Interface | Full Name |
|-----------|-----------|-----------|
| Shell | Core | `org.lingmo.Shell.Core` |
| Shell | WindowManager | `org.lingmo.Shell.WindowManager` |
| SettingsDaemon | Keyboard | `org.lingmo.SettingsDaemon.Keyboard` |

Interface names MUST use PascalCase for each segment after `org.lingmo.`.

### 8.4 Object Paths

D-Bus object paths MUST follow the pattern:

```
/org/lingmo/<Component>/<Object>
```

| Component | Object | Path |
|-----------|--------|------|
| Shell | | `/org/lingmo/Shell` |
| Power | Battery | `/org/lingmo/Power/Battery` |

Object paths MUST use PascalCase for each segment.

### 8.5 Methods, Signals, Properties

D-Bus method names MUST use PascalCase.

```xml
<method name="GetWindows">
<method name="FocusWindow">
```

D-Bus signal names MUST use PascalCase.

```xml
<signal name="WindowAdded">
<signal name="BatteryPercentChanged">
```

D-Bus property names MUST use PascalCase.

```xml
<property name="ActiveWindow" type="u" access="read">
<property name="Volume" type="d" access="readwrite">
```

### 8.6 Error Names

D-Bus error names MUST follow the pattern:

```
org.lingmo.<Component>.Error.<ErrorName>
```

Error names MUST use PascalCase.

```
org.lingmo.Shell.Error.InvalidWindowId
org.lingmo.Power.Error.NoBattery
```

### 8.7 Bus Activation Files

D-Bus activation service files MUST be named:

```
org.lingmo.<Component>.service
```

Files MUST be installed to:

- System bus: `${CMAKE_INSTALL_DATADIR}/dbus-1/system-services/`
- Session bus: `${CMAKE_INSTALL_DATADIR}/dbus-1/services/`

---

## 9. Plugin Naming

### 9.1 Plugin IDs

Plugin IDs MUST use reverse domain notation:

```
org.lingmo.<type>.<name>
```

| Type | Name | Plugin ID |
|------|------|-----------|
| `applet` | digital-clock | `org.lingmo.applet.digital-clock` |
| `applet` | task-manager | `org.lingmo.applet.task-manager` |
| `settings-panel` | display | `org.lingmo.settings-panel.display` |
| `settings-panel` | network | `org.lingmo.settings-panel.network` |
| `search-provider` | files | `org.lingmo.search-provider.files` |
| `extractor` | pdf | `org.lingmo.extractor.pdf` |

The `<type>` portion MUST be one of the registered plugin types from
the plugin system specification. The `<name>` portion MUST use kebab-case.

### 9.2 Metadata Keys

Plugin metadata JSON keys MUST use camelCase.

```json
{
    "id": "org.lingmo.applet.clock",
    "apiName": "org.lingmo.applet",
    "apiVersion": "3.0",
    "singleInstance": true,
    "configurable": true
}
```

### 9.3 Plugin Directories

Plugin shared libraries MUST be installed to:

```
${CMAKE_INSTALL_LIBDIR}/lingmo/plugins/<type>/
```

Where `<type>` is the plural form of the plugin type:

```
${libdir}/lingmo/plugins/applets/
${libdir}/lingmo/plugins/settings-panels/
${libdir}/lingmo/plugins/search-providers/
${libdir}/lingmo/plugins/extractors/
${libdir}/lingmo/plugins/thumbnail-providers/
```

### 9.4 Entry Point Symbol

The entry point symbol for every plugin shared library MUST be:

```cpp
extern "C" Lingmo::Plugin* lingmo_plugin_factory();
```

This symbol MUST be exported with default visibility.

### 9.5 Version Fields

Version fields in plugin metadata MUST use semantic versioning format:

```
MAJOR.MINOR.PATCH
```

Example: `"1.2.0"`.

Comparison MUST be done numerically, not as strings.
Only `MAJOR` and `MINOR` MUST be used for API compatibility checks.
`PATCH` is informational.

---

## 10. Configuration

### 10.1 Environment Variables

Environment variables MUST use SCREAMING_SNAKE_CASE with the `LINGMO_` prefix.

```
LINGMO_HOME
LINGMO_CONFIG_DIR
LINGMO_PLUGIN_PATH
LINGMO_THEME
LINGMO_SCALE_FACTOR
LINGMO_DEBUG
LINGMO_SESSION
```

Internal environment variables (not intended for user configuration)
MUST use the `_` prefix after `LINGMO_`:

```
LINGMO_INTERNAL_XDG_CONFIG_HOME
LINGMO_INTERNAL_DEVELOPER_MODE
```

### 10.2 Configuration Keys

Configuration key names MUST use camelCase. Hierarchical keys MUST use
forward slash as separator:

```
appearance/theme
appearance/accentColor
input/keyboard/repeatRate
input/touchpad/tapToClick
power/battery/saverProfile
```

Group names MUST use PascalCase or camelCase. Keys within a group MUST
use camelCase.

### 10.3 JSON Keys

JSON object keys MUST use camelCase.

```json
{
    "themeName": "lingmo-dark",
    "accentColor": "#3daee9",
    "showHiddenFiles": false,
    "windowWidth": 1024
}
```

JSON keys that correspond to configuration keys MUST match exactly.

### 10.4 YAML Keys

YAML keys MUST use camelCase, matching JSON conventions:

```yaml
themeName: lingmo-dark
accentColor: "#3daee9"
windowSettings:
  width: 1024
  height: 768
```

### 10.5 INI Groups and Keys

INI group names MUST use PascalCase. INI keys MUST use camelCase.

```ini
[General]
themeName = lingmo-dark
accentColor = #3daee9

[Input]
keyboardLayout = us
touchpadEnabled = true
```

### 10.6 Command-Line Options

Short options MUST use a single letter:

```
-v
-h
-t
```

Long options MUST use kebab-case:

```
--verbose
--help
--theme-name
--safe-mode
--config-file
```

Long option names MUST NOT use underscores.
Long option names MUST NOT use camelCase.

Options that take a value MUST use `=` in the long form:

```
--theme-name=lingmo-dark
--config-file=/etc/lingmo.conf
```

---

## 11. Desktop Integration

### 11.1 Desktop Entry IDs

Desktop entry file names MUST use the reverse domain pattern:

```
org.lingmo.<app>.desktop
```

| Application | Desktop Entry |
|-------------|---------------|
| File Manager | `org.lingmo.file-manager.desktop` |
| Terminal | `org.lingmo.terminal.desktop` |
| Settings | `org.lingmo.control-center.desktop` |

### 11.2 AppStream IDs

AppStream component IDs MUST match the desktop entry ID without the
`.desktop` suffix:

```
org.lingmo.file-manager
org.lingmo.terminal
org.lingmo.control-center
```

### 11.3 MIME Types

Lingmo-specific MIME types MUST use:

```
application/x-lingmo-<name>
```

| Name | Full MIME Type |
|------|---------------|
| Lingmo theme package | `application/x-lingmo-theme` |
| Lingmo plugin bundle | `application/x-lingmo-plugin` |

### 11.4 Icons

Icon names MUST use kebab-case:

```
document-open
application-exit
computer-laptop
input-mouse
preferences-system
```

Icons that are Lingmo-specific MUST use the `lingmo-` prefix:

```
lingmo-logo
lingmo-menu
lingmo-workspace
```

### 11.5 systemd Service Names

systemd service unit names MUST use kebab-case:

```
lingmo-session.service
lingmo-shell-core.service
lingmo-power.service
lingmo-settings-daemon.service
lingmo-display.service
```

### 11.6 Polkit Action IDs

Polkit action identifiers MUST use reverse domain notation:

```
org.lingmo.<domain>.<action>
```

| Domain | Action | Full ID |
|--------|--------|---------|
| power | suspend | `org.lingmo.power.suspend` |
| power | hibernate | `org.lingmo.power.hibernate` |
| session | switch-user | `org.lingmo.session.switch-user` |
| network | modify-connections | `org.lingmo.network.modify-connections` |

### 11.7 Portal IDs

The portal backend implementation ID MUST be:

```
org.freedesktop.impl.portal.Lingmo
```

---

## 12. Documentation

### 12.1 Specification Documents

Specification files MUST use Upper-Kebab-Case:

```
FS-LAYOUT.md
THEME-TOKENS.md
PROCESS-LIFECYCLE.md
```

### 12.2 Architecture Decision Records

ADR files MUST use the pattern:

```
NNNN-title-with-dashes.md
```

| Example | Description |
|---------|-------------|
| `0001-use-kwin-compositor.md` | First ADR |
| `0012-theme-token-format.md` | Twelfth ADR |

The number MUST be zero-padded to four digits.

### 12.3 RFC Documents

RFC files MUST use the pattern:

```
rfc-NNNN-title-with-dashes.md
```

| Example |
|---------|
| `rfc-0001-plugin-sandboxing.md` |

### 12.4 General Documentation

Documentation files SHOULD use PascalCase:

```
GettingStarted.md
ArchitectureOverview.md
ContributingGuide.md
```

### 12.5 README

The top-level README MUST be named `README.md` (all caps, no variation).
This is a convention that GitHub and other forges recognize natively.

### 12.6 Diagram Files

Diagram source files MUST use kebab-case:

```
session-startup-sequence.puml
process-architecture.drawio
theme-data-flow.png
```

### 12.7 Example Files

Example source files MUST follow the naming conventions of their language
as defined in the relevant sections of this document.

---

## 13. Forbidden Patterns

### 13.1 General Forbidden Patterns

The following naming patterns MUST NOT be used anywhere in the Lingmo
ecosystem:

| Pattern | Example (Invalid) | Reason |
|---------|------------------|--------|
| Hungarian notation | `strName`, `nCount` | Not used in Qt/C++ |
| Trailing underscore | `name_`, `count_` | Reserved by C++ standard |
| Leading underscore (unscoped) | `_name`, `__name` | Reserved by C++ standard |
| Mixed camelCase and snake_case | `active_windowCount` | Inconsistent |
| Names differing only by case | `activewindow` vs `activeWindow` | Readability hazard |
| Abbreviations not in the approved list | `calc` for `calculator`, `cfg` for `config` | Ambiguous |
| Single-letter names (except loop counters) | `int a; QString b;` | Obscures meaning |

### 13.2 Language-Specific Forbidden Patterns

**C++**:
- `class` names with non-alphabetic first character
- Macros without `LINGMO_` prefix (exception: include guards MAY use
  the path-based convention instead)
- `using namespace` in public headers
- `enum` (unscoped) — MUST use `enum class`

**QML**:
- QML types with lowercase first letter
- Property names with PascalCase
- Signal names with PascalCase (exception: signals that bridge to
  C++ MUST match the C++ method name)

**Rust**:
- `pub const` in snake_case (MUST use SCREAMING_SNAKE_CASE)
- Function names in PascalCase
- Type names in snake_case

**Go**:
- Package names with underscores
- File names with spaces or hyphens in `_test.go` suffix context

**D-Bus**:
- Hyphens in interface, method, signal, or property names
- Lowercase method names (D-Bus convention requires PascalCase)

### 13.3 Approved Abbreviations

The following abbreviations are APPROVED for use in all contexts. Any
abbreviation not in this list MUST be expanded to its full form.

| Abbreviation | Full Form | Notes |
|-------------|-----------|-------|
| API | Application Programming Interface | |
| ASCII | American Standard Code for Information Interchange | |
| ABI | Application Binary Interface | |
| ADR | Architecture Decision Record | |
| CLI | Command-Line Interface | |
| CPU | Central Processing Unit | |
| CSD | Client-Side Decoration | |
| CSS | Cascading Style Sheets | |
| CSV | Comma-Separated Values | |
| D-Bus | Desktop Bus | |
| DNS | Domain Name System | |
| GPU | Graphics Processing Unit | |
| GTK | GIMP Toolkit | |
| GUI | Graphical User Interface | |
| HTTP | Hypertext Transfer Protocol | |
| HTTPS | HTTP Secure | |
| ICC | International Color Consortium | |
| IDE | Integrated Development Environment | |
| IME | Input Method Editor | |
| INI | Initialization (file format) | |
| IPC | Inter-Process Communication | |
| IPP | Internet Printing Protocol | |
| JSON | JavaScript Object Notation | |
| MIME | Multipurpose Internet Mail Extensions | |
| MPRIS | Media Player Remote Interfacing Specification | |
| NTP | Network Time Protocol | |
| OCR | Optical Character Recognition | |
| OSD | On-Screen Display | |
| PAM | Pluggable Authentication Modules | |
| PNG | Portable Network Graphics | |
| PII | Personally Identifiable Information | |
| PID | Process Identifier | |
| PolKit | Policy Kit | |
| QML | Qt Meta-Object Language | |
| RAM | Random Access Memory | |
| RFC | Request for Comments | |
| RPM | RPM Package Manager (recursive) | |
| SDK | Software Development Kit | |
| SDDM | Simple Desktop Display Manager | |
| SMART | Self-Monitoring, Analysis and Reporting Technology | |
| SMTP | Simple Mail Transfer Protocol | |
| SSD | Server-Side Decoration | |
| SVG | Scalable Vector Graphics | |
| TCP | Transmission Control Protocol | |
| UDI | Universal Device Identifier | |
| UDP | User Datagram Protocol | |
| UI | User Interface | |
| URI | Uniform Resource Identifier | |
| URL | Uniform Resource Locator | |
| UUID | Universally Unique Identifier | |
| VPN | Virtual Private Network | |
| XML | Extensible Markup Language | |
| YAML | YAML Ain't Markup Language | |

Repository-specific abbreviations (approved for use within a single
repository's scope only):

| Abbreviation | Full Form | Repository |
|-------------|-----------|------------|
| NM | Network Manager | `lingmo-network` |
| BT | Bluetooth | `lingmo-bluetooth` |

---

## 14. Compliance Checklist

### 14.1 Repository Names

- [ ] Repository name matches `lingmo-<component>` pattern
- [ ] Component name is all lowercase
- [ ] Component name uses hyphens only (no underscores)
- [ ] Component name does not contain reserved words
- [ ] Component name ≤ 40 characters
- [ ] Component name does not start or end with hyphen

### 14.2 Library Names

- [ ] Shared library name starts with `libLingmo`
- [ ] SONAME is `libLingmo<Module>.so.<major>`
- [ ] CMake `find_package` name is `Lingmo<Module>`
- [ ] CMake target name is `Lingmo::<Module>`
- [ ] pkg-config name is `Lingmo<Module>`

### 14.3 C++

- [ ] All public declarations are in `Lingmo` namespace
- [ ] Class names use PascalCase with `Lingmo` prefix
- [ ] `enum class` is used (not plain `enum`)
- [ ] Enum values use PascalCase without type-prefix
- [ ] Methods and functions use camelCase
- [ ] Getters match property name (no `get` prefix)
- [ ] Member variables have no prefix
- [ ] Macros use `LINGMO_` prefix in SCREAMING_SNAKE_CASE
- [ ] Export macros match `LINGMO<MODULE>_EXPORT` pattern
- [ ] Private headers use `_p.h` suffix
- [ ] Private implementation classes use `Private` suffix

### 14.4 QML

- [ ] Module URI starts with `LingmoUI` or `Lingmo`
- [ ] QML types use PascalCase
- [ ] Properties use camelCase
- [ ] Signals use camelCase
- [ ] Object IDs use camelCase
- [ ] Public QML files match type name (PascalCase)

### 14.5 Rust

- [ ] Crate name matches repository component (kebab-case)
- [ ] Module names use snake_case
- [ ] Trait names use PascalCase
- [ ] Struct names use PascalCase
- [ ] Enum variants use PascalCase
- [ ] Function names use snake_case
- [ ] Constants use SCREAMING_SNAKE_CASE

### 14.6 Go

- [ ] Package names are lowercase, single-word
- [ ] Exported names use PascalCase
- [ ] Unexported names use camelCase
- [ ] File names use snake_case

### 14.7 D-Bus

- [ ] Service name starts with `org.lingmo.`
- [ ] Interface name starts with `org.lingmo.<Component>.`
- [ ] Object path starts with `/org/lingmo/`
- [ ] Methods, signals, properties use PascalCase
- [ ] Error names match `org.lingmo.<Component>.Error.<Name>`

### 14.8 Plugins

- [ ] Plugin ID matches `org.lingmo.<type>.<name>` pattern
- [ ] Metadata JSON keys use camelCase
- [ ] Library installed to correct type subdirectory
- [ ] Entry point symbol is `lingmo_plugin_factory`
- [ ] Version field is semantic versioning

### 14.9 Configuration

- [ ] Environment variables use `LINGMO_` prefix in SCREAMING_SNAKE_CASE
- [ ] JSON keys use camelCase
- [ ] INI groups use PascalCase
- [ ] INI keys use camelCase
- [ ] Long CLI options use kebab-case

### 14.10 Desktop Integration

- [ ] Desktop entry ID starts with `org.lingmo.`
- [ ] AppStream ID matches desktop entry (without `.desktop`)
- [ ] Icon names use kebab-case
- [ ] systemd service names use kebab-case
- [ ] Polkit action IDs start with `org.lingmo.`

### 14.11 Documentation

- [ ] Specification files use Upper-Kebab-Case
- [ ] ADR files use `NNNN-title-with-dashes.md` pattern
- [ ] General documentation uses PascalCase
- [ ] README is named `README.md`

### 14.12 General

- [ ] No Hungarian notation in any codebase
- [ ] All abbreviations are in the approved list
- [ ] No names differing only by case within the same scope
- [ ] No single-letter names except loop counters

---

## Version History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0.0 | 2026-07-08 | Lingmo Specification Team | Initial specification |

## References

- [SPEC-REPO-001]: Repository Layout Specification, Sections 2, 6.2–6.3
- [RFC 2119]: Key words for use in RFCs to Indicate Requirement Levels
- [Qt 6 Coding Conventions]: <https://wiki.qt.io/Qt_Coding_Style>
- [Rust API Guidelines]: <https://rust-lang.github.io/api-guidelines/>
- [Go Code Review Comments]: <https://go.dev/wiki/CodeReviewComments>
- [D-Bus Specification]: <https://dbus.freedesktop.org/doc/dbus-specification.html>
- [Keep a Changelog]: <https://keepachangelog.com/>
