# Repository Layout Specification

| | |
|---|---|
| **Document ID** | SPEC-REPO-001 |
| **Title** | Repository Layout Specification |
| **Version** | 1.0.0 |
| **Status** | Stable |
| **Applies to** | All Lingmo Desktop implementation repositories |
| **Supersedes** | None |

---

## 1. Scope

### 1.1 Applicability

This specification applies to EVERY repository within the Lingmo Desktop project
that produces source code artifacts (shared libraries, executables, plugins,
QML modules, or application data).

### 1.2 Exceptions

The following repositories are EXEMPT from this specification:

- `lingmo-desktop-release`: A meta-repository containing only release
  manifests and scripts. It follows its own layout defined in that
  repository's README.
- `lingmo-assets`: A data-only repository containing theme files, icons,
  wallpapers, cursors, and sounds. Section 4 (Public Headers) and Section 6
  (Build System) do not apply.
- `lingmo-sdk`: The SDK repository MAY deviate from Section 6.3 (Target Naming)
  because its primary artifacts are documentation and templates rather than
  compiled libraries.
- `lingmo-specs`: The specification repository is exempt from all sections
  of this specification.

All other repositories MUST conform to this specification.

### 1.3 Enforcement

Conformance is verified by:

- Automated CI checks (see Section 10)
- Repository audits performed before each desktop release

A repository that fails conformance MUST be fixed before it can be included
in a release.

---

## 2. Repository Naming

### 2.1 Name Format

Every repository name MUST match the following pattern:

```
lingmo-<component>
```

Where `<component>` is a lowercase identifier composed of:

- Letters (`a-z`)
- Digits (`0-9`)
- Hyphens (`-`)

The component name MUST NOT begin or end with a hyphen.
The component name MUST NOT contain consecutive hyphens.

### 2.2 Prefix Requirement

Every repository MUST use the `lingmo-` prefix. This prefix identifies the
repository as part of the Lingmo Desktop project in public listings, package
registries, and third-party tooling.

### 2.3 Reserved Component Names

The following component names are RESERVED and MUST NOT be used:

| Reserved Name | Rationale |
|--------------|-----------|
| `desktop` | Reserved for the desktop meta-package |
| `api` | Reserved for API documentation repositories |
| `spec` | Reserved for the specification repository |
| `sdk` | Reserved for the SDK repository |
| `release` | Reserved for release coordination |
| `test` | Reserved for integration test repositories |
| `meta` | Reserved for meta-repositories |

### 2.4 Abbreviation Rules

Abbreviations in component names MUST be used with caution. An abbreviation
SHOULD only be used when:

1. The full form would exceed 40 characters, OR
2. The abbreviation is universally recognized (e.g., `ipc`, `dbus`, `qml`, `osd`)

When an abbreviation is used, it MUST maintain the same casing and spelling
across ALL repositories. The `GLOSSARY.md` in the specs repository lists all
approved abbreviations.

### 2.5 Examples of Valid Names

```
lingmo-shell-core
lingmo-shell-surfaces
lingmo-power-manager
lingmo-control-center
lingmo-file-manager
lingmo-settings-daemon
lingmo-nm-applet           # "nm" is an approved abbreviation
lingmo-ipc                  # "ipc" is an approved abbreviation
```

### 2.6 Examples of Invalid Names

```
my-custom-tool              # Missing "lingmo-" prefix
LingmoTheme                 # Wrong case and separator
lingmo_daemon               # Underscore instead of hyphen
lingmo-shell--core          # Consecutive hyphens
lingmo-desktop              # Reserved name
lingmo-very-long-component-name-that-exceeds-forty  # More than 40 chars
```

---

## 3. Mandatory Directory Layout

### 3.1 Top-Level Directory Tree

Every repository MUST have the following top-level structure:

```
<repository-root>/
├── CMakeLists.txt
├── README.md
├── LICENSE
├── CHANGELOG.md
├── .gitignore
├── src/
│   ├── CMakeLists.txt
│   └── ...
├── tests/
│   ├── CMakeLists.txt
│   └── ...
├── debian/
│   ├── changelog
│   ├── control
│   ├── copyright
│   └── rules
├── .github/
│   └── workflows/
│       ├── build.yml
│       ├── test.yml
│       └── package.yml
└── docs/
    └── adr/
```

### 3.2 Directory Classification

Each directory is classified as one of:

- **REQUIRED**: MUST exist in every repository
- **CONDITIONAL**: MUST exist IF the repository produces artifacts of that type
- **RECOMMENDED**: SHOULD exist in most repositories
- **OPTIONAL**: MAY exist, no requirement
- **FORBIDDEN**: MUST NOT exist

### 3.3 Directory Requirements

| Path | Class | Condition |
|------|-------|-----------|
| `CMakeLists.txt` | REQUIRED | |
| `README.md` | REQUIRED | |
| `LICENSE` | REQUIRED | |
| `CHANGELOG.md` | REQUIRED | |
| `.gitignore` | REQUIRED | |
| `src/` | REQUIRED | |
| `src/CMakeLists.txt` | REQUIRED | If `src/` exists |
| `tests/` | REQUIRED | |
| `tests/CMakeLists.txt` | REQUIRED | If `tests/` exists |
| `tests/data/` | RECOMMENDED | If tests require external data |
| `debian/` | REQUIRED | |
| `debian/changelog` | REQUIRED | If `debian/` exists |
| `debian/control` | REQUIRED | If `debian/` exists |
| `debian/copyright` | REQUIRED | If `debian/` exists |
| `debian/rules` | REQUIRED | If `debian/` exists |
| `rpm/` | REQUIRED | |
| `.github/` | REQUIRED | |
| `.github/workflows/` | REQUIRED | If `.github/` exists |
| `.github/workflows/build.yml` | REQUIRED | |
| `.github/workflows/test.yml` | REQUIRED | |
| `.github/workflows/package.yml` | REQUIRED | |
| `docs/` | RECOMMENDED | |
| `docs/adr/` | RECOMMENDED | |
| `include/` | CONDITIONAL | If the repository produces a public library |
| `tools/` | OPTIONAL | |
| `examples/` | OPTIONAL | |
| `cmake/` | OPTIONAL | |
| `scripts/` | OPTIONAL | |
| `translations/` | RECOMMENDED | If the repository ships translatable strings |
| `qml/` | CONDITIONAL | If the repository produces QML modules |

### 3.4 Forbidden Top-Level Entries

The following MUST NOT appear at the top level of any repository:

| Entry | Rationale |
|-------|-----------|
| `build/` | Build artifacts are generated OUTSIDE the source tree |
| `build-*` | Any directory matching `build-*` |
| `.vscode/` | IDE configuration must be user-local |
| `.idea/` | IDE configuration must be user-local |
| `*.user` | Qt Creator user files |
| `*.autosave` | Editor autosave files |
| `__pycache__/` | Python cache files |
| `node_modules/` | Node.js dependencies |
| `*.pyc` | Compiled Python bytecode |

These entries MUST appear in `.gitignore` if they could be generated in the
repository directory during development.

### 3.5 Symbolic Links

Symbolic links within the repository:

- MUST NOT point outside the repository root
- MUST NOT create cycles
- SHOULD be avoided unless required for backward compatibility
- MUST be documented in README.md if used

---

## 4. Public Headers

### 4.1 Applicability

This section applies ONLY to repositories that produce a shared library with
a public API. Repositories that produce only executables or plugins MAY
omit public headers.

### 4.2 Header Location

Public headers MUST reside in either:

1. A top-level `include/` directory, OR
2. The `src/` directory alongside implementation files

A repository MUST choose ONE of these two approaches and MUST NOT mix them.

### 4.3 Installation Directory

All public headers MUST install to:

```
${CMAKE_INSTALL_INCLUDEDIR}/Lingmo/
```

The basename of each installed header MUST match the QObject or namespace
it declares. For example, `LingmoTheme.h` installs to `<prefix>/include/Lingmo/LingmoTheme.h`.

### 4.4 Include Guard

Every public header MUST use `#pragma once` as its include guard.
The `#ifndef` / `#define` / `#endif` pattern MUST NOT be used.

Rationale: `#pragma once` is supported by all compilers used in Lingmo Desktop
(GCC, Clang, MSVC) and eliminates the risk of guard name collisions across
47 repositories.

### 4.5 Namespace and Class Naming

All public C++ declarations MUST be in the `Lingmo` namespace.

Nested namespaces MUST be avoided. If they are necessary, they MUST be
declared as `inline namespace` to support versioning.

### 4.6 Export Macro

Every repository that produces a shared library MUST define an export macro:

```cpp
// In a public header or a build-system-generated header:
#define LINGMOTHEME_EXPORT Q_DECL_EXPORT   // On Windows
#define LINGMOTHEME_EXPORT                  // On other platforms
```

The macro name MUST follow the pattern `LINGMO<MODULE>_EXPORT` where
`<MODULE>` is the uppercased module name without the `Lingmo` prefix.

Examples:

| Module Name | Export Macro |
|-------------|-------------|
| `LingmoTheme` | `LINGMOTHEME_EXPORT` |
| `LingmoConfig` | `LINGMOCONFIG_EXPORT` |
| `LingmoShellCore` | `LINGMOSHELLCORE_EXPORT` |

The export macro MUST be defined in a file named `<Module>Export.h` in the
same directory as the public headers.

### 4.7 Visibility

All declarations that are part of the public API MUST be annotated with the
export macro.

Declarations that are NOT part of the public API MUST NOT be annotated with
the export macro. They MUST use `Q_DECL_HIDDEN` or be placed in a `detail`
namespace to prevent symbol export.

### 4.8 Deprecation

Deprecated public API MUST be annotated with:

```cpp
LINGMOTHEME_EXPORT [[deprecated("Use newFunction() instead")]]
void oldFunction();
```

The deprecation message MUST include the replacement API or a reference to
the relevant section in the API documentation.

### 4.9 Header Structure

Every public header MUST follow this structure, in order:

1. `#pragma once`
2. Include of the export macro header
3. Required Qt includes
4. Required Lingmo includes
5. Forward declarations
6. Class/function declarations

Blank lines MUST separate each section.

---

## 5. Source Layout

### 5.1 Private Headers

Private headers (declarations used across translation units within a library
but not exposed to users) MUST be named with a `_p.h` suffix:

```
LingmoTheme_p.h
LingmoConfig_p.h
```

Private headers MUST be placed alongside the implementation files in `src/`.
They MUST NOT be installed.

### 5.2 Internal Headers

Headers that are internal to a single translation unit or a tightly coupled
set of files MUST be placed in a `detail/` subdirectory:

```
src/detail/hash_helpers.h
src/detail/platform_detect.h
```

Internal headers MUST NOT be installed. They MUST NOT be included by any
file outside the repository.

### 5.3 Implementation Files

Implementation files (`.cpp`) MUST be placed in `src/`.

The file name SHOULD match the class or module it implements:

```
src/LingmoTheme.cpp           # Implements Lingmo::Theme
src/LingmoTheme_p.cpp         # Implements private data for Lingmo::Theme
```

### 5.4 Generated Code

Files that are generated by a build step (moc, uic, rcc, protobuf, etc.)
MUST be placed in the build directory, NOT in the source directory.

If generated code MUST be committed to the repository (e.g., for bootstrapping),
it MUST be placed in a `generated/` subdirectory within `src/`:

```
src/generated/
```

The method of generation MUST be documented in a comment at the top of each
committed generated file.

### 5.5 QML Modules

If the repository produces QML modules, the QML source files MUST be in a
`qml/` directory. The directory structure within `qml/` MUST mirror the
QML import URI:

```
qml/
└── LingmoUI/
    ├── Controls/
    │   ├── qmldir
    │   ├── Button.qml
    │   └── ButtonPrivate.qml
    └── Window/
        ├── qmldir
        └── Window.qml
```

The `qmldir` file MUST exist in each QML module directory.

QML files prefixed with a capital letter are public types.
QML files prefixed with a lowercase letter or containing `Private` are
private types and MUST NOT be imported by external code.

### 5.6 Resource Files

Qt resource files (`.qrc`) MUST be placed in the directory that corresponds
to the virtual file system they define.

For example, if a resource file maps files to `:/lingmo/theme/`, the `.qrc`
file SHOULD be at `src/theme/theme.qrc`.

Resource files MUST NOT use the `:/` prefix to override Qt built-in resources.

### 5.7 Translation Source Files

Translation source files (`.ts`) MUST be placed in a `translations/` directory,
NOT inside `src/`.

Translation project files (`.pro`) for managing translation files SHOULD be
at the repository root.

---

## 6. Build System

### 6.1 CMake Version

Every repository MUST require CMake 3.21 or later.

The `cmake_minimum_required` directive MUST appear as the first substantive
line of the top-level `CMakeLists.txt`.

### 6.2 Project Declaration

The `project()` call MUST specify:

```cmake
project(Lingmo<Component>
    VERSION <major.minor.patch>
    DESCRIPTION "<Short description>"
    HOMEPAGE_URL "https://lingmo.org"
    LANGUAGES CXX
)
```

The project name MUST use PascalCase with the `Lingmo` prefix.
The project name MUST NOT contain hyphens. Hyphens are converted to
PascalCase boundaries.

Examples:

| Repository | CMake Project Name |
|-----------|-------------------|
| `lingmo-shell-core` | `LingmoShellCore` |
| `lingmo-power-manager` | `LingmoPowerManager` |
| `lingmo-file-manager` | `LingmoFileManager` |

### 6.3 Target Naming

CMake targets MUST follow these rules:

| Artifact Type | Target Name Pattern | Example |
|--------------|--------------------|---------|
| Shared library | `Lingmo<Module>` | `LingmoTheme` |
| Executable (system) | `lingmo-<component>` | `lingmo-panel` |
| Executable (tool) | `lingmo-<component>-<tool>` | `lingmo-shell-core-inspect` |
| Plugin library | `lingmo_<type>_<name>` | `lingmo_applet_clock` |
| Test executable | `tst_<TestName>` | `tst_ThemeTest` |
| Benchmark | `bm_<BenchmarkName>` | `bm_IconLoading` |

### 6.4 C++ Standard

Every repository MUST set:

```cmake
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)
```

These MUST appear in the top-level `CMakeLists.txt` before any
`add_subdirectory` call.

### 6.5 Build Output Directory

Build output MUST go to a directory that is a sibling of the repository
root, not a child:

```
# CORRECT:
../build-lingmo-shell-core/

# INCORRECT:
build/
<repo-root>/build/
```

The build directory MUST be ignored by Git (in `.gitignore`).

### 6.6 Install Rules

Every library target MUST specify:

```cmake
install(TARGETS Lingmo<Module>
    EXPORT Lingmo<Module>Targets
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
)
```

Every executable target MUST specify:

```cmake
install(TARGETS lingmo-<component>
    RUNTIME DESTINATION ${CMAKE_INSTALL_LIBEXECDIR}
)
```

Rationale for `LIBEXECDIR` for daemons and shell components:
These executables are not intended to be launched directly by users.
They are launched by systemd, D-Bus activation, or the session manager.
Putting them in `LIBEXECDIR` separates them from user-facing executables.

### 6.7 Export and Package Config

Every library target MUST export its build targets:

```cmake
install(EXPORT Lingmo<Module>Targets
    FILE Lingmo<Module>Targets.cmake
    NAMESPACE Lingmo::
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/Lingmo<Module>
)
```

A `Lingmo<Module>Config.cmake` file MUST be installed alongside the targets
file to support `find_package(Lingmo<Module>)`.

### 6.8 Dependencies

Dependencies MUST be declared with `find_package` BEFORE the corresponding
`add_subdirectory` for the dependent target.

Build-time-only dependencies (test frameworks, code generators) MUST use:

```cmake
find_package(Qt6 REQUIRED COMPONENTS Test)
```

And the dependency MUST be linked only to the test target:

```cmake
target_link_libraries(tst_ThemeTest PRIVATE Qt6::Test)
```

### 6.9 Finding Lingmo Dependencies

Lingmo libraries MUST be found using:

```cmake
find_package(Lingmo<Module> REQUIRED)
```

The `REQUIRED` keyword MUST be used for all direct dependencies.
The `OPTIONAL_COMPONENTS` keyword MAY be used for optional features.

### 6.10 Compiler Warnings

Every repository MUST enable the following compiler warning settings:

```cmake
if(MSVC)
    add_compile_options(/W4 /WX)
else()
    add_compile_options(-Wall -Wextra -Wpedantic -Werror)
endif()
```

These MUST appear in the top-level `CMakeLists.txt` or in a sourced
compiler settings file in `cmake/`.

### 6.11 Custom CMake Modules

Custom CMake modules (`.cmake` files) MUST be placed in the `cmake/`
directory.

If a custom module provides a `find_package` integration, it MUST be
named `Find<Package>.cmake`.

---

## 7. Tests

### 7.1 Test Directory

Every repository MUST have a `tests/` directory.

If a repository contains no testable code (e.g., a header-only library), a
`tests/CMakeLists.txt` MUST still exist and SHOULD contain a compile test
that verifies the headers compile without errors.

### 7.2 Test Naming

Test source files MUST be named according to the following patterns:

| Test Type | Pattern | Example |
|-----------|---------|---------|
| C++ unit test | `tst_<TestName>.cpp` | `tst_Theme.cpp` |
| C++ benchmark | `bm_<BenchmarkName>.cpp` | `bm_IconLoading.cpp` |
| QML unit test | `tst_<TestName>.qml` | `tst_Button.qml` |
| Integration test | `tst_<Scenario>.cpp` | `tst_SettingsDaemon.cpp` |

### 7.3 Test Targets

Test executables MUST NOT be installed.
They MUST be built only when `BUILD_TESTING` is enabled:

```cmake
if(BUILD_TESTING)
    enable_testing()
    add_subdirectory(tests)
endif()
```

### 7.4 Test Registration

All tests MUST be registered with CTest:

```cmake
add_test(NAME tst_Theme COMMAND tst_Theme)
```

Test names MUST be unique within the repository.

### 7.5 Test Data

Test data files MUST be placed in `tests/data/`.

Large test data files (over 1 MB) MUST NOT be committed to the repository.
They MUST be downloaded during the build or test step, or generated by a
test fixture.

### 7.6 Screenshot Tests

Screenshot regression test reference images MUST be placed in:

```
tests/screenshots/references/
```

Screenshot test outputs MUST be placed in:

```
tests/screenshots/results/
```

The `results/` directory MUST be ignored by Git (in `.gitignore`).

### 7.7 Test Environment Requirements

Tests MUST NOT assume a running desktop environment.
Tests MUST NOT require a display (Wayland or X11) unless the test name
includes the suffix `_display`:

| Test Name | Display Required |
|-----------|-----------------|
| `tst_Theme` | No |
| `tst_WaylandSurface_display` | Yes |

Tests that require a display MUST skip gracefully if the display is not
available, using `QSKIP`:

```cpp
if (QGuiApplication::platformName() == "offscreen") {
    QSKIP("This test requires a Wayland display");
}
```

---

## 8. Documentation

### 8.1 README.md

Every repository MUST have a `README.md` file at the repository root.

`README.md` MUST contain:

1. **Repository name and purpose**: A one-paragraph description of what the
   repository provides.
2. **Build status badge**: A GitHub Actions workflow badge.
3. **Dependencies**: Build-time and runtime dependencies.
4. **Building**: Build instructions.
5. **Contributing**: Link to the contributing guide.
6. **License**: License identifier (SPDX).
7. **Links**: Links to related repositories.

`README.md` SHOULD contain:

8. **Quick start**: Minimal steps to build and test.
9. **Configuration**: Environment variables or build options.
10. **Directory layout** (if non-standard): Explanation of non-obvious
    directories.

### 8.2 LICENSE

Every repository MUST include a `LICENSE` file at the repository root.

All Lingmo Desktop repositories MUST use the GNU General Public License
v2.0 or later, unless an exception is approved by the project's legal team.

The license file MUST be an exact copy of the canonical GPL-2.0 text
from the SPDX License List.

### 8.3 CHANGELOG.md

Every repository MUST include a `CHANGELOG.md` file.

The format MUST follow the Keep a Changelog specification
(<https://keepachangelog.com/en/1.1.0/>).

Entries MUST be categorized as:

- Added
- Changed
- Deprecated
- Removed
- Fixed
- Security

Each release MUST have a version header linking to the Git tag.
The version header format is:

```markdown
## [1.2.0] - 2026-04-15
```

An `[Unreleased]` section MUST be maintained at the top for changes
since the last release.

### 8.4 API Documentation

Public API MUST be documented using Doxygen-style comments.

Every public function, class, method, and enum MUST have a Doxygen
comment block.

The comment block MUST include:

- `\brief`: One-sentence summary
- `\param`: For each parameter (functions only)
- `\return`: For non-void return values
- `\sa`: Related API (OPTIONAL)

Documentation comments MUST use the `//!` or `///` style (block comments
`/** */` are also acceptable).

### 8.5 Examples

If the repository ships examples, they MUST be in an `examples/` directory.

Each example MUST have its own `CMakeLists.txt` and MUST be buildable
independently of the repository's test suite.

Examples MUST NOT be installed. They MUST be built only when
`BUILD_EXAMPLES` is set (OFF by default).

### 8.6 Architecture Decision Records

Architecture decisions MUST be recorded as ADR files in `docs/adr/`.

ADR files MUST follow the template in `SPEC-DOCS-001` (Documentation
Standards Specification).

---

## 9. Packaging

### 9.1 Debian Packaging

Every repository MUST have a `debian/` directory containing:

| File | Requirement | Description |
|------|-------------|-------------|
| `changelog` | REQUIRED | Debian changelog format |
| `control` | REQUIRED | Package metadata and dependencies |
| `copyright` | REQUIRED | License information (machine-readable) |
| `rules` | REQUIRED | Build rules (dh syntax) |
| `compat` | REQUIRED | debhelper compatibility level |
| `source/format` | REQUIRED | Source package format (`3.0 (native)`) |

The `rules` file MUST use the `dh` sequencer and MUST NOT contain
hardcoded paths.

### 9.2 RPM Packaging

Every repository MUST have an `rpm/` directory containing at least one
`.spec` file.

The spec file MUST follow Fedora packaging guidelines.

The spec file MUST use CMake macros (`%cmake`, `%cmake_build`,
`%cmake_install`) instead of invoking CMake directly.

### 9.3 Installation Paths

All installation paths MUST be relative to `CMAKE_INSTALL_PREFIX`
(default: `/usr`).

The following `GNUInstallDirs` variables MUST be used for path resolution:

| Variable | Default | Usage |
|----------|---------|-------|
| `CMAKE_INSTALL_BINDIR` | `bin` | User-facing executables |
| `CMAKE_INSTALL_LIBEXECDIR` | `libexec` | System executables |
| `CMAKE_INSTALL_LIBDIR` | `lib/` | Shared libraries |
| `CMAKE_INSTALL_INCLUDEDIR` | `include/` | Public headers |
| `CMAKE_INSTALL_DATADIR` | `share/` | Data files |
| `CMAKE_INSTALL_SYSCONFDIR` | `/etc` | Configuration files |

The `GNUInstallDirs` module MUST be included at the top of every
`CMakeLists.txt` that uses install rules:

```cmake
include(GNUInstallDirs)
```

### 9.4 Plugin Installation

Plugin shared libraries MUST be installed to:

```
${CMAKE_INSTALL_LIBDIR}/lingmo/plugins/<type>/
```

Where `<type>` is one of: `applets`, `settings-panels`, `search-providers`,
`extractors`, `thumbnail-providers`.

### 9.5 QML Module Installation

QML modules MUST be installed to:

```
${CMAKE_INSTALL_LIBDIR}/lingmo/qml/<ImportUri>/
```

Where `<ImportUri>` uses forward slashes as path separators:
`LingmoUI/Controls/`.

### 9.6 Runtime File Ownership

A package MUST NOT own files that belong to another package.

If two packages share a directory (e.g., `/usr/lib/lingmo/`), the directory
MUST be owned by the `lingmo-desktop` base package.

---

## 10. CI

### 10.1 Required Workflows

Every repository MUST have three GitHub Actions workflows:

1. **`build.yml`**: Triggered on every push and pull request.
   - Builds in Debug and Release configurations.
   - Runs on both amd64 and arm64 (via QEMU or native runners).
   - Reports build status.

2. **`test.yml`**: Triggered on every push to main and on pull requests.
   - Runs the full test suite.
   - Runs on amd64 only (arm64 tests run weekly).
   - Uploads test results as artifacts on failure.

3. **`package.yml`**: Triggered on tags matching `v*`.
   - Generates `.deb` and `.rpm` packages.
   - Uploads packages as build artifacts.
   - Attaches packages to the GitHub Release.

### 10.2 Workflow Structure

Every workflow MUST:

1. Check out the repository with `actions/checkout@v4`.
2. Install dependencies (from `debian/control` or a documented list).
3. Configure with CMake.
4. Build.
5. Run tests (workflow 2 only).
6. Upload artifacts (workflow 3 only).

Workflows MUST NOT hardcode Ubuntu versions. They MUST use `ubuntu-24.04`
or the latest LTS runner as the default, with a matrix entry for testing
on `ubuntu-24.04` and `ubuntu-latest`.

### 10.3 Build Matrix

The `build.yml` workflow MUST include a matrix:

```yaml
strategy:
  matrix:
    config: [Debug, Release]
    arch: [amd64, arm64]
```

If arm64 runners are not available, the arm64 entries SHOULD use
cross-compilation or QEMU emulation.

### 10.4 Required Checks

The `.github/workflows/build.yml` workflow MUST include the following
steps before the build:

1. **Linting**: clang-format check (if the repo contains C++ code).
2. **Static analysis**: clang-tidy (if the repo contains C++ code).

These checks MUST fail the workflow if violations are found.

For QML code, the workflow MUST include:
3. **qmllint**: QML syntax and convention check.

### 10.5 conformance-check.yml

A fourth workflow, `conformance-check.yml`, MUST exist to verify compliance
with this specification. This workflow is OPTIONAL for individual repos
and SHOULD be maintained in the `lingmo-specs` repository, running against
all repos.

The conformance check MUST verify:

- All REQUIRED files exist (sections 3.1, 9.1)
- All FORBIDDEN files are absent (section 3.4)
- Repository name matches the pattern (section 2.1)
- CMakeLists.txt contains required elements (section 6.1, 6.2, 6.4)

---

## 11. Repository Checklist

The following checklist MUST be completed before a repository can be
released. Each item maps to a `MUST` requirement in this specification.

### 11.1 Repository Identity

- [ ] Repository name matches `lingmo-<component>` pattern
- [ ] Component name is not a reserved name
- [ ] Component name does not exceed 40 characters

### 11.2 Top-Level Files

- [ ] `CMakeLists.txt` exists
- [ ] `README.md` exists
- [ ] `LICENSE` exists and is GPL-2.0-or-later
- [ ] `CHANGELOG.md` exists
- [ ] `.gitignore` exists
- [ ] No `build/` directory in source tree
- [ ] No IDE configuration files in source tree

### 11.3 Source Layout

- [ ] `src/CMakeLists.txt` exists
- [ ] Public headers use `#pragma once`
- [ ] Public headers live in `include/` or `src/` (not both)
- [ ] Export macro defined and used on all public declarations
- [ ] Private headers use `_p.h` suffix
- [ ] Internal headers in `detail/` subdirectory

### 11.4 Build System

- [ ] CMake version ≥ 3.21
- [ ] C++ standard set to C++20
- [ ] Project name uses `Lingmo` prefix and PascalCase
- [ ] `GNUInstallDirs` is included
- [ ] Install rules use `CMAKE_INSTALL_LIBEXECDIR` for system executables
- [ ] Library targets export to CMake package config
- [ ] Compiler warnings enabled and treated as errors

### 11.5 Tests

- [ ] `tests/CMakeLists.txt` exists
- [ ] Tests registered with CTest
- [ ] Tests do not assume a running desktop (unless suffixed `_display`)
- [ ] Test executables are not installed

### 11.6 Packaging

- [ ] `debian/changelog` exists and has current release entry
- [ ] `debian/control` exists and has correct dependencies
- [ ] `debian/copyright` exists and matches LICENSE
- [ ] `debian/rules` exists and uses `dh`
- [ ] `rpm/` directory exists with a `.spec` file

### 11.7 CI

- [ ] `.github/workflows/build.yml` exists
- [ ] `.github/workflows/test.yml` exists
- [ ] `.github/workflows/package.yml` exists
- [ ] `build.yml` runs on amd64 and arm64
- [ ] `build.yml` runs in Debug and Release
- [ ] Linting step is configured (clang-format or qmllint)

### 11.8 Documentation

- [ ] `README.md` contains name, purpose, dependencies, building, license
- [ ] Public API has Doxygen comments
- [ ] `CHANGELOG.md` follows Keep a Changelog format

---

## Version History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0.0 | 2026-07-08 | Lingmo Specification Team | Initial specification |

## References

- [ARCHITECTURE-v3.md]: Lingmo Desktop Technical Architecture, Section 9 (Build System)
- [RFC 2119]: Key words for use in RFCs to Indicate Requirement Levels
- [Keep a Changelog v1.1.0]: <https://keepachangelog.com/en/1.1.0/>
