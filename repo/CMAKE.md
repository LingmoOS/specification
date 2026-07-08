# CMake Organization Specification

| | |
|---|---|
| **Document ID** | SPEC-REPO-003 |
| **Title** | CMake Organization Specification |
| **Version** | 1.0.0 |
| **Status** | Draft |
| **Category** | Repository |
| **Applies to** | All Lingmo Desktop implementation repositories |
| **Supersedes** | SPEC-REPO-001 Sections 6, 7, 9.3 |
| **Cross-ref** | SPEC-REPO-001 (Repository Layout), SPEC-REPO-002 (Header Organization), SPEC-CORE-001 (Naming Convention) |

---

## 1. Scope

### 1.1 Applicability

This specification applies to EVERY repository within the Lingmo Desktop project that contains a `CMakeLists.txt` file, including:

- Repositories that produce shared libraries
- Repositories that produce executables (daemons, shell components, tools)
- Repositories that produce plugins (applets, decorations, effects)
- Repositories that produce QML modules
- Header-only repositories
- Test-only or example-only repositories

### 1.2 Exceptions

The following repositories are EXEMPT from specific sections:

| Repository | Exempt Sections |
|---|---|
| `lingmo-specs` | All sections (no build system) |
| `lingmo-assets` | Sections 4, 5, 6, 9, 13 (data-only, no compiled code) |
| `lingmo-sdk` | Section 4.3, 7.4 (template-based, not typical targets) |

### 1.3 Supersession Notice

Sections 6 (Build System), 7 (Tests), and 9.3 (Installation Paths) of SPEC-REPO-001 are superseded by this document. All repositories MUST migrate to the rules defined herein within one major release cycle.

---

## 2. Minimum Requirements

### 2.1 CMake Version

[REQ-CMAKE-001] Every repository MUST specify `cmake_minimum_required(VERSION 3.21)` as the first substantive line of the top-level `CMakeLists.txt`.

[REQ-CMAKE-002] The `cmake_minimum_required` directive MUST appear before any `project()` call, any `include()` directive, or any `add_*` command.

### 2.2 Required Policies

[REQ-CMAKE-003] Every repository MUST set the following CMake policies explicitly at the top of `CMakeLists.txt`, immediately after `cmake_minimum_required`:

```cmake
cmake_policy(SET CMP0077 NEW)   # option() honors normal variables
cmake_policy(SET CMP0092 NEW)   # MSVC flags without /W3 override
cmake_policy(SET CMP0128 NEW)   # install() generates --target warnings correctly
```

[REQ-CMAKE-004] Additional policies SHOULD be set to `NEW` explicitly rather than relying on CMake defaults. Policy settings MUST be documented with a rationale comment.

### 2.3 Supported Generators

[REQ-CMAKE-005] Every repository MUST support the following CMake generators:

| Generator | Platform | Status |
|---|---|---|
| Ninja | Linux, macOS, Windows | REQUIRED |
| Unix Makefiles | Linux | REQUIRED |
| Visual Studio 17 2022 | Windows (MSVC) | OPTIONAL |

[REQ-CMAKE-006] The build system MUST NOT rely on generator-specific behavior. Any code path that is generator-specific MUST be guarded by a `${CMAKE_GENERATOR}` condition with a documented rationale.

### 2.4 Supported Compilers

[REQ-CMAKE-007] Every repository MUST support compilation with these toolchains:

| Compiler | Minimum Version | Target Platform |
|---|---|---|
| GCC | 12 | Linux |
| Clang | 16 | Linux, macOS |
| MSVC | Visual Studio 17 2022 | Windows |

[REQ-CMAKE-008] Compiler-specific workarounds MUST be isolated in a `cmake/CompilerSettings.cmake` module and MUST NOT appear inline in target `CMakeLists.txt` files.

### 2.5 Language Standards

[REQ-CMAKE-009] Every repository MUST enforce C++20 with the following block, appearing before any `add_subdirectory` call:

```cmake
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)
```

[REQ-CMAKE-010] Individual targets MUST NOT override `CMAKE_CXX_STANDARD`. If a target requires a different standard, it MUST be explicitly justified and approved by the architecture team.

---

## 3. Project Structure

### 3.1 Top-Level CMakeLists.txt

[REQ-CMAKE-011] The top-level `CMakeLists.txt` MUST follow this exact structure, in order:

```
// Line 1:   cmake_minimum_required(VERSION 3.21)
// Line 2:   cmake_policy(...)  (optional, but RECOMMENDED)
// Block:    project(...)
// Block:    C++ standard setup
// Block:    GNUInstallDirs include
// Block:    Options and variables
// Block:    find_package calls for external dependencies
// Block:    add_subdirectory calls for internal subdirectories
// Block:    CPack include (if packaging)
```

[REQ-CMAKE-012] Blank lines MUST separate each block. Each block MUST have a section comment:

```cmake
# ── Dependencies ──────────────────────────────────────────
find_package(Qt6 REQUIRED COMPONENTS Core Gui)

# ── Subdirectories ────────────────────────────────────────
add_subdirectory(src)
if(BUILD_TESTING)
    add_subdirectory(tests)
endif()
```

### 3.2 Subdirectory Organization

[REQ-CMAKE-013] Source subdirectories MUST be structured as follows:

| Subdirectory | CMake Command | Condition |
|---|---|---|
| `src/` | `add_subdirectory(src)` | REQUIRED if repo has code |
| `tests/` | `add_subdirectory(tests)` | CONDITIONAL: `if(BUILD_TESTING)` |
| `tools/` | `add_subdirectory(tools)` | OPTIONAL |
| `examples/` | `add_subdirectory(examples)` | CONDITIONAL: `if(BUILD_EXAMPLES)` |
| `cmake/` | No `add_subdirectory` (config file directory) | OPTIONAL |

[REQ-CMAKE-014] The top-level `CMakeLists.txt` MUST NOT contain target definitions (`add_library`, `add_executable`). All targets MUST be defined in the directory corresponding to their source code.

### 3.3 Module Inclusion

[REQ-CMAKE-015] Custom CMake modules (`*.cmake` files) MUST be placed in the top-level `cmake/` directory.

[REQ-CMAKE-016] The `cmake/` directory MUST be added to `CMAKE_MODULE_PATH`:

```cmake
list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")
```

[REQ-CMAKE-017] Each reusable CMake module MUST have a single responsibility. A module that defines multiple unrelated functions MUST be split.

### 3.4 Reusable CMake Modules

[REQ-CMAKE-018] The following standard modules MUST be provided in `cmake/` across all repositories (via a shared `lingmo-cmake` repository or vendored copies):

| Module | Purpose |
|---|---|
| `LingmoInstallDirs.cmake` | Installation path conventions (common across all repos) |
| `LingmoCompilerSettings.cmake` | Compiler flags, warnings, visibility |
| `LingmoSanitizers.cmake` | Sanitizer support (ASan, UBSan, LSan) |
| `LingmoPCH.cmake` | Precompiled header support |
| `LingmoQt.cmake` | Qt integration helpers |
| `LingmoPlugin.cmake` | Plugin target helpers |
| `LingmoExport.cmake` | Export header generation |
| `LingmoPackaging.cmake` | CPack configuration |

[REQ-CMAKE-019] Custom `Find<Package>.cmake` modules MUST follow the CMake standard Find-module format and MUST set both `<Package>_FOUND` and the canonical `<Package>::<Target>` imported targets.

---

## 4. Targets

### 4.1 Target Naming

[REQ-CMAKE-020] All CMake target names MUST follow the patterns defined in SPEC-CORE-001 §8.1 and SPEC-REPO-001 §6.3.

| Artifact Type | Pattern | Example |
|---|---|---|
| Shared library | `Lingmo<Module>` | `LingmoTheme` |
| Static library | `Lingmo<Module>` | `LingmoTheme` (no separate convention) |
| Object library | `Lingmo<Module>_obj` | `LingmoTheme_obj` |
| Executable (system) | `lingmo-<component>` | `lingmo-panel` |
| Executable (tool) | `lingmo-<component>-<tool>` | `lingmo-shell-core-inspect` |
| Plugin library | `lingmo_<type>_<name>` | `lingmo_applet_clock` |
| Test executable | `tst_<TestName>` | `tst_Theme` |
| Benchmark | `bm_<BenchmarkName>` | `bm_IconLoading` |
| Interface (header-only) | `Lingmo<Module>` | `LingmoUtils` |

### 4.2 Library Targets

[REQ-CMAKE-021] Every shared library target MUST specify:

```cmake
add_library(LingmoTheme SHARED
    include/LingmoTheme/Theme.h
    include/LingmoTheme/ThemeSettings.h
    src/LingmoTheme/Theme.cpp
    src/LingmoTheme/ThemeSettings.cpp
    src/LingmoTheme/private/Theme_p.h
)
```

[REQ-CMAKE-022] Source files MUST be listed explicitly in `add_library` and `add_executable` calls. The `GLOB` or `FILE(GLOB ...)` command MUST NOT be used for source collection.

> *Rationale*: `GLOB` does not automatically detect new files. Developers must re-run CMake after adding files, which is error-prone and breaks IDE integration.

[REQ-CMAKE-023] Library target properties MUST be set in the same `CMakeLists.txt` where the target is created, using `set_target_properties`.

### 4.3 Executable Targets

[REQ-CMAKE-024] System executables (daemons, shell components) MUST be installed to `${CMAKE_INSTALL_LIBEXECDIR}` per SPEC-REPO-001 §6.6.

[REQ-CMAKE-025] User-facing executables (applications, tools) MUST be installed to `${CMAKE_INSTALL_BINDIR}`.

[REQ-CMAKE-026] Every executable target MUST define `WIN32_EXECUTABLE` only on Windows:

```cmake
if(WIN32)
    set_target_properties(lingmo-panel PROPERTIES WIN32_EXECUTABLE TRUE)
endif()
```

### 4.4 Plugin Targets

[REQ-CMAKE-027] Plugin libraries MUST be defined as `MODULE` libraries:

```cmake
add_library(lingmo_applet_clock MODULE
    src/ClockApplet.cpp
    src/ClockApplet.h
)
```

[REQ-CMAKE-028] Plugin targets MUST NOT be installed to the default library directory. They MUST be installed to:

```cmake
install(TARGETS lingmo_applet_clock
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}/lingmo/plugins/applets
)
```

### 4.5 QML Module Targets

[REQ-CMAKE-029] QML modules MUST be defined using `qt_add_qml_module`:

```cmake
qt_add_qml_module(LingmoUI_Controls
    URI LingmoUI.Controls
    VERSION 1.0
    QML_FILES
        Button.qml
        Slider.qml
    SOURCES
        LingmoControlsPlugin.cpp
    RESOURCES
        resources.qrc
)
```

[REQ-CMAKE-030] The `qt_add_qml_module` target name MUST follow the pattern `LingmoUI_<Module>` where `<Module>` matches the QML module name with dots replaced by underscores.

### 4.6 Interface (Header-Only) Targets

[REQ-CMAKE-031] Header-only libraries MUST be defined as `INTERFACE` targets:

```cmake
add_library(LingmoUtils INTERFACE)
target_include_directories(LingmoUtils
    INTERFACE
        "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>"
        "$<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>"
)
```

[REQ-CMAKE-032] Interface targets MUST NOT list source files. They MUST specify `INTERFACE` libraries and include directories only.

### 4.7 Target Properties

[REQ-CMAKE-033] Every non-test library target MUST set:

```cmake
set_target_properties(LingmoTheme PROPERTIES
    CXX_VISIBILITY_PRESET hidden
    VISIBILITY_INLINES_HIDDEN ON
    EXPORT_NAME LingmoTheme
)
```

[REQ-CMAKE-034] The `EXPORT_NAME` property MUST match the target name without the `Lingmo` prefix. This ensures the CMake config file exports use the `Lingmo::` namespace.

---

## 5. Dependencies

### 5.1 find_package Rules

[REQ-CMAKE-035] All external dependencies MUST be declared via `find_package` BEFORE any `add_subdirectory` call.

[REQ-CMAKE-036] The `REQUIRED` keyword MUST be used for mandatory dependencies:

```cmake
find_package(Qt6 REQUIRED COMPONENTS Core Gui)
find_package(LingmoTheme REQUIRED)
```

[REQ-CMAKE-037] Optional dependencies MUST use `OPTIONAL_COMPONENTS`:

```cmake
find_package(Qt6 COMPONENTS Test QUIET)
```

[REQ-CMAKE-038] Every `find_package` call MUST specify both the package name and, for multi-component packages, the `COMPONENTS` keyword.

### 5.2 target_link_libraries Rules

[REQ-CMAKE-039] Dependencies MUST be linked with the minimum necessary visibility:

| Visibility | Semantics | Used For |
|---|---|---|
| `PUBLIC` | Propagated to consumers | Types exposed in public API |
| `PRIVATE` | Not propagated | Implementation details |
| `INTERFACE` | Header-only propagation | Header-only libraries |

[REQ-CMAKE-040] The `PUBLIC` keyword MUST be used ONLY for dependencies whose types appear in public headers. All other dependencies MUST use `PRIVATE`.

[REQ-CMAKE-041] Plugin targets MUST use `PRIVATE` linkage for all dependencies. Plugins are loaded at runtime and their dependencies MUST NOT propagate.

### 5.3 Forbidden Patterns

[REQ-CMAKE-042] The following CMake dependency patterns are FORBIDDEN:

| Pattern | Example | Problem |
|---|---|---|
| `link_directories` | `link_directories(/usr/lib/foo)` | Non-portable, breaks find_package |
| `target_link_libraries` with absolute path | `target_link_libraries(foo /usr/lib/libbar.so)` | Non-portable, no dependency resolution |
| `include_directories` | `include_directories(/usr/include/foo)` | Global scope, not target-specific |
| `add_definitions` | `add_definitions(-DFOO=1)` | Global scope, pollutes all targets |
| Manual `-I` flags | `set(CMAKE_CXX_FLAGS "-I/usr/include/foo")` | Breaks target-level include management |

[REQ-CMAKE-043] All dependency paths MUST be expressed through CMake targets (`target_include_directories`, `target_link_libraries`). Raw directory or file paths are FORBIDDEN.

### 5.4 Interface Dependencies

[REQ-CMAKE-044] When a library depends on another Lingmo library that is used only in private implementation, the dependency MUST be `PRIVATE`:

```cmake
target_link_libraries(LingmoShellCore
    PRIVATE
        Lingmo::Theme
        Lingmo::Config
)
```

[REQ-CMAKE-045] When a library exposes types from another library in its public API, the dependency MUST be `PUBLIC`:

```cmake
target_link_libraries(LingmoShellCore
    PUBLIC
        Lingmo::IPC
    PRIVATE
        Lingmo::Theme
)
```

### 5.5 System Dependencies

[REQ-CMAKE-046] System libraries (pthread, dl, m, rt) MUST be linked via CMake's built-in targets:

```cmake
target_link_libraries(LingmoTheme PRIVATE Threads::Threads)
```

[REQ-CMAKE-047] The `find_library` command SHOULD NOT be used for system libraries. Use `find_package` with the appropriate CMake module.

---

## 6. Qt Integration

### 6.1 Automatic MOC

[REQ-CMAKE-048] Every target that contains Q_OBJECT macros MUST enable AUTOMOC:

```cmake
set_target_properties(LingmoTheme PROPERTIES
    AUTOMOC ON
)
```

[REQ-CMAKE-049] AUTOMOC MUST be enabled at the target level, not globally:

```cmake
set(CMAKE_AUTOMOC ON)        # FORBIDDEN — global scope
set_target_properties(... PROPERTIES AUTOMOC ON)  # CORRECT
```

[REQ-CMAKE-050] The `AUTOMOC_MOC_NAMESPACE` property SHOULD be set to `Lingmo`:

```cmake
set_target_properties(LingmoTheme PROPERTIES
    AUTOMOC ON
    AUTOMOC_MOC_NAMESPACE Lingmo
)
```

### 6.2 Automatic UIC

[REQ-CMAKE-051] AUTOUIC MUST be enabled for targets that use Qt Widgets `.ui` files:

```cmake
set_target_properties(LingmoControlPanel PROPERTIES AUTOUIC ON)
```

[REQ-CMAKE-052] UI files MUST be placed in the same directory as the source files that use them. They MUST NOT be placed in a separate `ui/` directory.

### 6.3 Automatic RCC

[REQ-CMAKE-053] AUTORCC MUST be enabled for targets that use Qt resource files:

```cmake
set_target_properties(LingmoTheme PROPERTIES AUTORCC ON)
```

[REQ-CMAKE-054] Resource files (`.qrc`) MUST be listed as source files in `add_library` or `add_executable`. AUTORCC processes them automatically.

### 6.4 qt_add_qml_module

[REQ-CMAKE-055] QML modules MUST use `qt_add_qml_module` (available in Qt 6.4+). The older `qt5_add_qml_module` or manual QML type registration MUST NOT be used.

[REQ-CMAKE-056] The `qt_add_qml_module` call MUST specify all QML source files, C++ source files, and resource files explicitly. The `QML_FILES` argument MUST list every `.qml` file in the module.

### 6.5 Translations

[REQ-CMAKE-057] Translation files (`.ts`) MUST be managed with `qt_add_translations` or `qt_add_translation`:

```cmake
qt_add_translations(LingmoTheme
    TS_FILES
        translations/lingmo-theme_de.ts
        translations/lingmo-theme_fr.ts
)
```

[REQ-CMAKE-058] Translation source files MUST reside in a `translations/` directory per SPEC-REPO-001 §5.7.

### 6.6 Resources

[REQ-CMAKE-059] Qt resource files MUST NOT use the `:/` prefix to override Qt built-in resources (SPEC-REPO-001 §5.6).

[REQ-CMAKE-060] Resource files SHOULD use the `Lingmo` namespace prefix for all internal paths: `:/lingmo/theme/icons/`.

---

## 7. Installation

### 7.1 GNUInstallDirs

[REQ-CMAKE-061] Every repository MUST include `GNUInstallDirs` at the top of the top-level `CMakeLists.txt`:

```cmake
include(GNUInstallDirs)
```

[REQ-CMAKE-062] All install destinations MUST use `GNUInstallDirs` variables. Hardcoded paths MUST NOT appear in `install()` commands.

### 7.2 install(TARGETS)

[REQ-CMAKE-063] Library targets MUST use this install pattern:

```cmake
install(TARGETS LingmoTheme
    EXPORT LingmoThemeTargets
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/LingmoTheme
)
```

[REQ-CMAKE-064] System executable targets MUST use `LIBEXECDIR`:

```cmake
install(TARGETS lingmo-panel
    RUNTIME DESTINATION ${CMAKE_INSTALL_LIBEXECDIR}
)
```

### 7.3 install(FILES) and install(DIRECTORY)

[REQ-CMAKE-065] Public headers MUST be installed using `install(DIRECTORY)` per SPEC-REPO-002 §11.4.

[REQ-CMAKE-066] Configuration files MUST be installed to `${CMAKE_INSTALL_SYSCONFDIR}/lingmo/`.

[REQ-CMAKE-067] Data files MUST be installed to `${CMAKE_INSTALL_DATADIR}/lingmo/<component>/`.

### 7.4 Export and Package Config Generation

[REQ-CMAKE-068] Every library target MUST export its targets:

```cmake
install(EXPORT LingmoThemeTargets
    FILE LingmoThemeTargets.cmake
    NAMESPACE Lingmo::
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/LingmoTheme
)
```

[REQ-CMAKE-069] Every library MUST provide a `Lingmo<Module>Config.cmake` file that supports `find_package(Lingmo<Module>)`. This file MUST be auto-generated using `configure_package_config_file` or a hand-written template.

[REQ-CMAKE-070] The config file template MUST be placed in `cmake/Lingmo<Module>Config.cmake.in`:

```cmake
@PACKAGE_INIT@

include("${CMAKE_CURRENT_LIST_DIR}/LingmoThemeTargets.cmake")

check_required_components(LingmoTheme)
```

[REQ-CMAKE-071] The config file version MUST be auto-generated using `write_basic_package_version_file`:

```cmake
write_basic_package_version_file(
    "${CMAKE_CURRENT_BINARY_DIR}/LingmoThemeConfigVersion.cmake"
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY SameMajorVersion
)
```

[REQ-CMAKE-072] The `SameMajorVersion` compatibility mode MUST be used for all Lingmo libraries.

### 7.5 Install Components

[REQ-CMAKE-073] The `COMPONENT` argument MUST be specified in every `install()` command:

| Component | Content | Installable Separately |
|---|---|---|
| `runtime` | Shared libraries, executables | Yes |
| `devel` | Headers, CMake config files, static libraries | Yes |
| `plug-<type>` | Plugin libraries | Yes |
| `data` | Data files, translations | Yes |
| `doc` | Documentation | Yes |

[REQ-CMAKE-074] Example:

```cmake
install(TARGETS LingmoTheme
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR} COMPONENT runtime
    PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/LingmoTheme COMPONENT devel
)
```

---

## 8. Build Types

### 8.1 Required Build Types

[REQ-CMAKE-075] Every repository MUST support the following build types:

| Build Type | Flags | Use Case |
|---|---|---|
| `Debug` | `-O0 -g` | Development, debugging |
| `Release` | `-O2 -DNDEBUG` | Production builds |
| `RelWithDebInfo` | `-O2 -g -DNDEBUG` | Production with debug symbols |
| `MinSizeRel` | `-Oz -DNDEBUG` | Space-constrained deployments |

### 8.2 Project Default

[REQ-CMAKE-076] The default build type MUST be `RelWithDebInfo` if none is specified:

```cmake
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
    set(CMAKE_BUILD_TYPE RelWithDebInfo CACHE STRING "Build type" FORCE)
endif()
```

### 8.3 Multi-Config Generators

[REQ-CMAKE-077] For multi-configuration generators (Visual Studio, Xcode), the `CMAKE_CONFIGURATION_TYPES` variable MUST be constrained to the four required types:

```cmake
set(CMAKE_CONFIGURATION_TYPES "Debug;Release;RelWithDebInfo;MinSizeRel"
    CACHE STRING "Configuration types" FORCE)
```

---

## 9. Compiler Options

### 9.1 Warning Flags

[REQ-CMAKE-078] Every repository MUST enable strict compiler warnings:

```cmake
if(MSVC)
    add_compile_options(/W4 /WX /permissive- /utf-8)
else()
    add_compile_options(-Wall -Wextra -Wpedantic -Werror)
endif()
```

[REQ-CMAKE-079] Individual targets MUST NOT override the global warning level. Target-specific warning suppressions MUST use `target_compile_options` with `PRIVATE` visibility and an `INTERFACE` suppression file for public headers.

### 9.2 Visibility

[REQ-CMAKE-080] Every shared library target MUST set hidden visibility (SPEC-REPO-002 §5.5):

```cmake
set_target_properties(LingmoTheme PROPERTIES
    CXX_VISIBILITY_PRESET hidden
    VISIBILITY_INLINES_HIDDEN ON
)
```

### 9.3 Sanitizers

[REQ-CMAKE-081] The `cmake/LingmoSanitizers.cmake` module MUST provide an `enable_sanitizers()` function that supports:

| Sanitizer | Flag | Build Type Requirement |
|---|---|---|
| AddressSanitizer | `-fsanitize=address` | Debug only |
| UndefinedBehaviorSanitizer | `-fsanitize=undefined` | Debug only |
| LeakSanitizer | `-fsanitize=leak` | Debug only (Linux) |

[REQ-CMAKE-082] Sanitizer support MUST be opt-in via a CMake option:

```cmake
option(LINGMO_ENABLE_SANITIZERS "Enable sanitizers" OFF)
```

### 9.4 Link-Time Optimization

[REQ-CMAKE-083] LTO MUST be enabled for Release builds:

```cmake
if(CMAKE_BUILD_TYPE STREQUAL "Release" OR CMAKE_BUILD_TYPE STREQUAL "MinSizeRel")
    include(CheckIPOSupported)
    check_ipo_supported(RESULT result)
    if(result)
        set(CMAKE_INTERPROCEDURAL_OPTIMIZATION ON)
    endif()
endif()
```

[REQ-CMAKE-084] LTO MUST be opt-out per-target for targets that are incompatible with LTO:

```cmake
set_target_properties(lingmo-plugin-loader PROPERTIES INTERPROCEDURAL_OPTIMIZATION OFF)
```

### 9.5 Precompiled Headers

[REQ-CMAKE-085] The `cmake/LingmoPCH.cmake` module MUST provide a reusable PCH setup:

```cmake
lingmo_pch(LingmoTheme
    PUBLIC
        <LingmoTheme/Theme.h>
        <QtCore/QObject>
    PRIVATE
        <QtCore/QDebug>
)
```

[REQ-CMAKE-086] PCH headers MUST be chosen to maximize cache reuse. PCH files MUST NOT include headers that change frequently during development.

### 9.6 Platform-Specific Handling

[REQ-CMAKE-087] Platform-specific compiler options MUST be isolated in `cmake/CompilerSettings.cmake`:

```cmake
# cmake/CompilerSettings.cmake
if(CMAKE_CXX_COMPILER_ID STREQUAL "MSVC")
    add_compile_options(/utf-8 /Zc:preprocessor /Zc:__cplusplus)
elseif(CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
    add_compile_options(-fcolor-diagnostics)
endif()
```

---

## 10. Testing

### 10.1 CTest Integration

[REQ-CMAKE-088] Every repository MUST enable CTest using the following pattern in the top-level `CMakeLists.txt`:

```cmake
if(BUILD_TESTING)
    enable_testing()
    add_subdirectory(tests)
endif()
```

[REQ-CMAKE-089] The `BUILD_TESTING` option MUST NOT be redefined. It is provided by the `CTest` module when `enable_testing()` is called.

### 10.2 Test Target Rules

[REQ-CMAKE-090] Test executables MUST NOT be installed. They MUST be built only when `BUILD_TESTING` is enabled.

[REQ-CMAKE-091] Each test executable MUST be linked with the test framework explicitly:

```cmake
add_executable(tst_Theme tst_Theme.cpp)
target_link_libraries(tst_Theme PRIVATE LingmoTheme Qt6::Test)
```

[REQ-CMAKE-092] Test target names MUST be unique within the repository (SPEC-REPO-001 §7.4).

### 10.3 Test Registration

[REQ-CMAKE-093] Every test MUST be registered with CTest using `add_test`:

```cmake
add_test(NAME tst_Theme COMMAND tst_Theme)
```

[REQ-CMAKE-094] Tests that require a display MUST use a timeout and a `SKIP_RETURN_CODE`:

```cmake
add_test(NAME tst_WaylandSurface_display COMMAND tst_WaylandSurface_display)
set_tests_properties(tst_WaylandSurface_display PROPERTIES
    SKIP_RETURN_CODE 77
    TIMEOUT 30
)
```

[REQ-CMAKE-095] Tests that require a display MUST use `QSKIP` when the display is unavailable, returning exit code 77.

### 10.4 Benchmark Registration

[REQ-CMAKE-096] Benchmarks MUST be registered with CTest as "benchmark" tests:

```cmake
add_test(NAME bm_IconLoading COMMAND bm_IconLoading)
set_tests_properties(bm_IconLoading PROPERTIES
    LABELS "benchmark"
    TIMEOUT 120
)
```

### 10.5 Test Timeouts

[REQ-CMAKE-097] Every `add_test` call MUST specify a timeout appropriate to the test:

| Test Type | Default Timeout |
|---|---|
| Unit test | 30 seconds |
| Integration test | 120 seconds |
| Benchmark | 300 seconds |
| Display test | 30 seconds |
| Screenshot test | 60 seconds |

[REQ-CMAKE-098] Timeouts MUST be set using `set_tests_properties`:

```cmake
set_tests_properties(tst_Theme PROPERTIES TIMEOUT 30)
```

---

## 11. Packaging Support

### 11.1 CPack Configuration

[REQ-CMAKE-099] The top-level `CMakeLists.txt` MAY include CPack support for generating packages directly:

```cmake
if(LINGMO_ENABLE_PACKAGING)
    include(CPack)
endif()
```

[REQ-CMAKE-100] When CPack is used, the following variables MUST be set:

```cmake
set(CPACK_PACKAGE_NAME "lingmo-<component>")
set(CPACK_PACKAGE_VERSION ${PROJECT_VERSION})
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "Lingmo Desktop - <component>")
set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/LICENSE")
```

### 11.2 Install Components for Packaging

[REQ-CMAKE-101] The install component system (Section 7.5) MUST be used to enable per-component packaging:

| Component | Debian Package | RPM Package |
|---|---|---|
| `runtime` | `lingmo-<component>` | `lingmo-<component>` |
| `devel` | `lingmo-<component>-dev` | `lingmo-<component>-devel` |
| `data` | `lingmo-<component>-data` | `lingmo-<component>-data` |
| `doc` | `lingmo-<component>-doc` | `lingmo-<component>-doc` |

### 11.3 Debian Packaging Support

[REQ-CMAKE-102] The `debian/` directory structure MUST follow SPEC-REPO-001 §9.1. The `rules` file MUST call CMake through `dh`:

```makefile
%:
    dh $@

override_dh_auto_configure:
    dh_auto_configure -- \
        -DCMAKE_BUILD_TYPE=RelWithDebInfo \
        -DBUILD_TESTING=ON

override_dh_auto_test:
    dh_auto_test -- \
        --output-on-failure
```

### 11.4 RPM Packaging Support

[REQ-CMAKE-103] The `rpm/` spec file MUST use CMake macros:

```spec
%build
%cmake -DBUILD_TESTING=ON
%cmake_build

%install
%cmake_install

%check
%ctest --output-on-failure
```

---

## 12. IDE Support

### 12.1 Compile Commands

[REQ-CMAKE-104] Every repository MUST set `CMAKE_EXPORT_COMPILE_COMMANDS` to `ON`:

```cmake
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
```

This enables language servers (clangd), IDEs (CLion, VS Code), and static analyzers to parse the build correctly.

### 12.2 Qt Creator

[REQ-CMAKE-105] Qt Creator project files (`CMakeLists.txt.user`, `*.files`, `*.includes`) MUST NOT be committed. The `.gitignore` MUST include:

```
*.user
*.autosave
```

### 12.3 CLion

[REQ-CMAKE-106] CLion configuration directories (`.idea/`) MUST NOT be committed. The `.gitignore` MUST include:

```
.idea/
```

### 12.4 VS Code

[REQ-CMAKE-107] VS Code workspace configuration (`.vscode/`) MUST NOT be committed. The `.gitignore` MUST include:

```
.vscode/
```

---

## 13. Performance

### 13.1 Unity (Batch) Builds

[REQ-CMAKE-108] Every repository SHOULD support unity (batch) builds to reduce compilation time for CI and release builds:

```cmake
set(CMAKE_UNITY_BUILD ON)
```

[REQ-CMAKE-109] Unity build MUST be opt-in via CMake option, not enabled by default for Debug builds:

```cmake
option(LINGMO_ENABLE_UNITY_BUILD "Enable unity (batch) builds" OFF)
if(LINGMO_ENABLE_UNITY_BUILD AND NOT CMAKE_BUILD_TYPE STREQUAL "Debug")
    set(CMAKE_UNITY_BUILD ON)
endif()
```

[REQ-CMAKE-110] Files that are incompatible with unity builds (due to name collisions or conflicting declarations) MUST be explicitly excluded:

```cmake
set_target_properties(LingmoTheme PROPERTIES
    UNITY_BUILD_MODE BATCH
)
```

### 13.2 Incremental Builds

[REQ-CMAKE-111] The build system MUST NOT use `GLOB` for source collection (Section 4.2). Explicit source lists ensure that CMake detects changes correctly for incremental builds.

[REQ-CMAKE-112] Header file dependencies MUST be tracked automatically by CMake. Manual `set_source_files_properties` with `OBJECT_DEPENDS` MUST NOT be used.

### 13.3 Object Libraries

[REQ-CMAKE-113] Object libraries SHOULD be used to share compiled code between multiple targets without duplicating compilation:

```cmake
add_library(LingmoTheme_obj OBJECT
    src/LingmoTheme/Theme.cpp
    src/LingmoTheme/ThemeSettings.cpp
)
target_link_libraries(LingmoTheme_obj PRIVATE Qt6::Core)

add_library(LingmoTheme SHARED $<TARGET_OBJECTS:LingmoTheme_obj>)
add_executable(tst_Theme $<TARGET_OBJECTS:LingmoTheme_obj> tst_Theme.cpp)
```

[REQ-CMAKE-114] Object libraries MUST be named with the `_obj` suffix to clearly indicate their purpose.

### 13.4 Dependency Reuse

[REQ-CMAKE-115] Common dependency groups SHOULD be aggregated into CMake interface libraries to reduce repetitive `find_package` and `target_link_libraries` calls:

```cmake
add_library(LingmoBaseDeps INTERFACE)
target_link_libraries(LingmoBaseDeps INTERFACE
    Qt6::Core
    Qt6::Gui
    Qt6::DBus
)
```

---

## 14. Compliance Checklist

### 14.1 Minimum Requirements

| ID | Requirement | Verification Method |
|---|---|---|
| REQ-CMAKE-001 | cmake_minimum_required ≥ 3.21 | Parse CMakeLists.txt |
| REQ-CMAKE-002 | cmake_minimum_required before project() | Parse CMakeLists.txt |
| REQ-CMAKE-003 | Required policies set | Parse CMakeLists.txt |
| REQ-CMAKE-009 | C++20 enforced | Parse CMakeLists.txt |
| REQ-CMAKE-010 | No target-level C++ standard override | Scan for CMAKE_CXX_STANDARD in subdirectories |

### 14.2 Project Structure

| ID | Requirement | Verification Method |
|---|---|---|
| REQ-CMAKE-011 | Top-level structure follows blocks | Repository audit |
| REQ-CMAKE-013 | Subdirectories organized per spec | Directory listing |
| REQ-CMAKE-014 | No targets in top-level CMakeLists.txt | Parse CMakeLists.txt |
| REQ-CMAKE-015 | Custom modules in cmake/ | Directory listing |
| REQ-CMAKE-018 | Standard modules provided | File existence check |

### 14.3 Targets

| ID | Requirement | Verification Method |
|---|---|---|
| REQ-CMAKE-020 | Target names follow naming conventions | Automated check |
| REQ-CMAKE-022 | No GLOB for source collection | Scan CMake files |
| REQ-CMAKE-027 | Plugins as MODULE libraries | Parse CMake files |
| REQ-CMAKE-033 | Visibility properties set | Parse CMake files |

### 14.4 Dependencies

| ID | Requirement | Verification Method |
|---|---|---|
| REQ-CMAKE-035 | find_package before add_subdirectory | Parse CMake files |
| REQ-CMAKE-039 | Minimum visibility (PUBLIC/PRIVATE/INTERFACE) | Automated scan |
| REQ-CMAKE-042 | No forbidden patterns | Automated grep |

### 14.5 Qt Integration

| ID | Requirement | Verification Method |
|---|---|---|
| REQ-CMAKE-048 | AUTOMOC ON for Q_OBJECT targets | Parse CMake files |
| REQ-CMAKE-049 | AUTOMOC at target level, not global | Automated scan |
| REQ-CMAKE-055 | qt_add_qml_module for QML modules | Parse CMake files |
| REQ-CMAKE-057 | Translations managed via qt_add_translations | Parse CMake files |

### 14.6 Installation

| ID | Requirement | Verification Method |
|---|---|---|
| REQ-CMAKE-061 | GNUInstallDirs included | Parse CMakeLists.txt |
| REQ-CMAKE-063 | install(TARGETS) uses correct pattern | Parse CMake files |
| REQ-CMAKE-068 | Targets exported | Parse CMake files |
| REQ-CMAKE-069 | Config file provided | File existence check |
| REQ-CMAKE-072 | SameMajorVersion compatibility | Parse version file |
| REQ-CMAKE-073 | COMPONENT specified in install() | Parse CMake files |

### 14.7 Build Types and Compiler Options

| ID | Requirement | Verification Method |
|---|---|---|
| REQ-CMAKE-075 | Four build types supported | Parse CMake files |
| REQ-CMAKE-076 | Default build type RelWithDebInfo | Check CMakeCache |
| REQ-CMAKE-078 | Strict warnings enabled | Parse CMake files |
| REQ-CMAKE-080 | Hidden visibility set | Parse CMake files |
| REQ-CMAKE-083 | LTO enabled for Release | Parse CMake files |

### 14.8 Testing

| ID | Requirement | Verification Method |
|---|---|---|
| REQ-CMAKE-088 | CTest enabled via BUILD_TESTING | Parse CMakeLists.txt |
| REQ-CMAKE-090 | Test executables not installed | Parse install rules |
| REQ-CMAKE-093 | Tests registered with add_test | Parse CMake files |
| REQ-CMAKE-097 | Timeouts set for all tests | Parse CMake files |

### 14.9 IDE and Performance

| ID | Requirement | Verification Method |
|---|---|---|
| REQ-CMAKE-104 | CMAKE_EXPORT_COMPILE_COMMANDS ON | Parse CMakeLists.txt |
| REQ-CMAKE-105 | Qt Creator files gitignored | Check .gitignore |
| REQ-CMAKE-108 | Unity build support | Parse CMake files |

---

## Appendix A: Complete Top-Level CMakeLists.txt Example

```cmake
cmake_minimum_required(VERSION 3.21)

cmake_policy(SET CMP0077 NEW)
cmake_policy(SET CMP0092 NEW)
cmake_policy(SET CMP0128 NEW)

project(LingmoTheme
    VERSION 1.0.0
    DESCRIPTION "Lingmo Desktop - Theme engine"
    HOMEPAGE_URL "https://lingmo.org"
    LANGUAGES CXX
)

# ── C++ Standard ──────────────────────────────────────────
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# ── Install Dirs ──────────────────────────────────────────
include(GNUInstallDirs)

# ── Compiler Settings ─────────────────────────────────────
list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")
include(LingmoCompilerSettings)
include(LingmoSanitizers)
include(LingmoPCH)

# ── Options ───────────────────────────────────────────────
option(BUILD_TESTING "Build tests" ON)
option(LINGMO_ENABLE_SANITIZERS "Enable sanitizers" OFF)
option(LINGMO_ENABLE_UNITY_BUILD "Enable unity builds" OFF)

# ── Dependencies ──────────────────────────────────────────
find_package(Qt6 REQUIRED COMPONENTS Core Gui)
find_package(Qt6 COMPONENTS Test QUIET)

# ── Subdirectories ────────────────────────────────────────
add_subdirectory(src)
if(BUILD_TESTING)
    enable_testing()
    add_subdirectory(tests)
endif()
```

---

## Appendix B: Migration from SPEC-REPO-001

| Old Rule (REPO-001 §6) | New Rule (REPO-003) | Migration Action |
|---|---|---|
| Basic CMake version check | Full minimum requirements with policies | Add `cmake_policy` calls |
| Simple project() call | Project call with full metadata | Add `HOMEPAGE_URL` field |
| GLOB permitted for sources | GLOB forbidden, explicit lists required | Replace `file(GLOB ...)` with explicit file lists |
| No target visibility properties | Hidden visibility REQUIRED | Add `set_target_properties` with visibility |
| No unity build mention | Unity build support REQUIRED | Add `CMAKE_UNITY_BUILD` option |
| No build type defaults | Default to RelWithDebInfo | Add default build type logic |
| No sanitizer support | Sanitizer module REQUIRED | Add `LingmoSanitizers.cmake` |
| No install components | COMPONENT required in install() | Add COMPONENT to every install() |
| No PCH module | PCH module REQUIRED | Add `LingmoPCH.cmake` |
| No compile_commands.json | CMAKE_EXPORT_COMPILE_COMMANDS ON | Add global setting |
| No IDE rules | IDE gitignore rules | Add to .gitignore |

---

*Cross-ref*: SPEC-REPO-001 (Repository Layout), SPEC-REPO-002 (Header Organization), SPEC-CORE-001 (Naming Convention), SPEC-CORE-002 (Authoring Standard)
