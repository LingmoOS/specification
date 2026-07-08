# Installation Specification

| | |
|---|---|
| **Document ID** | SPEC-REPO-004 |
| **Title** | Installation Specification |
| **Version** | 1.0.0 |
| **Status** | Draft |
| **Category** | Repository |
| **Applies to** | All Lingmo Desktop implementation repositories |
| **Supersedes** | SPEC-REPO-001 §6.6–§6.7, SPEC-REPO-003 §7.2–§7.5 |
| **Cross-ref** | SPEC-PLATFORM-001 (Filesystem Layout), SPEC-REPO-003 (CMake Organization), SPEC-REPO-002 (Header Organization), SPEC-REPO-001 (Repository Layout), SPEC-CORE-001 (Naming Convention) |

---

## 1. Scope

### 1.1 Applicability

This specification applies to EVERY repository within the Lingmo Desktop project that produces installable artifacts. It defines:

- Which artifacts each repository type MUST install
- Which install component each artifact belongs to
- The mandatory CMake install patterns for each artifact type
- Permissions and ownership requirements
- Versioning conventions for installed artifacts
- Optional artifact installation rules
- Mapping between install components and distribution packages
- Verification methods for correct installation

### 1.2 Exceptions

| Repository | Exception |
|---|---|
| `lingmo-specs` | Exempt from all sections (no installable artifacts) |
| `lingmo-assets` | Exempt from Sections 2.1–2.5 (no compiled artifacts). Sections 2.6–2.10 apply. |
| `lingmo-sdk` | Exempt from Section 2.5 (no public library targets). Sections 2.9–2.10 apply. |

### 1.3 Supersession Notice

Section 6.6 (Install Rules), 6.7 (Export and Package Config) of SPEC-REPO-001 and Sections 7.2 (install(TARGETS)), 7.3 (install(FILES) and install(DIRECTORY)), 7.4 (Export and Package Config Generation), and 7.5 (Install Components) of SPEC-REPO-003 are superseded by this document. The concrete patterns and mappings defined here take precedence.

---

## 2. Installable Artifacts

### 2.1 Executables

[REQ-INST-001] Every executable target MUST be installed. The destination depends on the executable type (SPEC-PLATFORM-001 §3):

```cmake
# User-facing executable
install(TARGETS lingmo-file-manager
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    COMPONENT runtime
)

# Internal executable (daemon, shell component)
install(TARGETS lingmo-panel
    RUNTIME DESTINATION ${CMAKE_INSTALL_LIBEXECDIR}
    COMPONENT runtime
)
```

[REQ-INST-002] Executable targets MUST specify `RUNTIME DESTINATION`. `LIBRARY` and `ARCHIVE` destinations MUST NOT be specified for executables.

[REQ-INST-003] Every executable MUST have the executable permission bit set (Section 5.1).

### 2.2 Shared Libraries

[REQ-INST-004] Public shared libraries MUST be installed with `LIBRARY DESTINATION` set to `${CMAKE_INSTALL_LIBDIR}`:

```cmake
install(TARGETS LingmoTheme
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    COMPONENT runtime
)
```

[REQ-INST-005] Public shared libraries MUST specify `PUBLIC_HEADER` to install matching headers:

```cmake
install(TARGETS LingmoTheme
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/LingmoTheme
    COMPONENT runtime
)
```

[REQ-INST-006] Private shared libraries (internal to Lingmo, not for external consumers) MUST be installed to `${CMAKE_INSTALL_LIBDIR}/lingmo/`:

```cmake
install(TARGETS LingmoShellPrivate
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}/lingmo
    COMPONENT runtime
)
```

[REQ-INST-007] Every shared library MUST set `VERSION` and `SOVERSION` properties (Section 6.1).

[REQ-INST-008] Every shared library MUST provide a CMake package config file (Section 2.12).

### 2.3 Static Libraries

[REQ-INST-009] Static libraries MUST be installed with `ARCHIVE DESTINATION`:

```cmake
install(TARGETS LingmoTheme
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    COMPONENT devel
)
```

[REQ-INST-010] Static libraries MUST be installed to the `devel` component. They MUST NOT be installed to the `runtime` component.

[REQ-INST-011] A repository MUST NOT produce both a shared and a static variant of the same library. If both are needed, the shared library is the default; the static library is built only when `BUILD_STATIC_LIBS` is `ON`.

### 2.4 Plugins

[REQ-INST-012] Plugin MODULE libraries MUST be installed to `/usr/lib/lingmo/plugins/<type>/` per SPEC-PLATFORM-001 §9.1:

```cmake
install(TARGETS lingmo_applet_clock
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}/lingmo/plugins/applets
    COMPONENT plug-applets
)
```

[REQ-INST-013] The plugin type directory MUST match the plugin type as declared in the plugin metadata (SPEC-PLATFORM-001 §9.1):

| Plugin Type | Install Subdirectory | Component |
|---|---|---|
| `applet` | `applets` | `plug-applets` |
| `settings-panel` | `settings-panels` | `plug-settings-panels` |
| `search-provider` | `search-providers` | `plug-search-providers` |
| `thumbnail-provider` | `thumbnail-providers` | `plug-thumbnail-providers` |
| `content-extractor` | `content-extractors` | `plug-content-extractors` |
| `file-preview` | `file-preview` | `plug-file-preview` |
| `kwin-decoration` | — (see REQ-INST-014) | `plug-kwin-decorations` |

[REQ-INST-014] KWin decoration plugins MUST be installed to KWin's plugin directory:

```cmake
install(TARGETS lingmo_decoration
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}/qt6/plugins/org.kde.kwin.decoration
    COMPONENT plug-kwin-decorations
)
```

[REQ-INST-015] Plugin metadata files (`.lingmo-plugin` JSON) MUST be installed alongside the plugin:

```cmake
install(FILES org.lingmo.applet.digital-clock.lingmo-plugin
    DESTINATION ${CMAKE_INSTALL_DATADIR}/lingmo/plugins/metadata
    COMPONENT plug-applets
)
```

### 2.5 QML Modules

[REQ-INST-016] QML modules MUST be installed to `/usr/lib/qt6/qml/` per SPEC-PLATFORM-001 §4.3:

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
    INSTALL_DIRECTORY ${CMAKE_INSTALL_LIBDIR}/qt6/qml/LingmoUI/Controls
)
```

[REQ-INST-017] The `INSTALL_DIRECTORY` argument to `qt_add_qml_module` MUST point to the correct path under `${CMAKE_INSTALL_LIBDIR}/qt6/qml/`.

[REQ-INST-018] QML module targets MUST be installed to the `runtime` component:

```cmake
set_target_properties(LingmoUI_Controls PROPERTIES
    INSTALL_COMPONENT runtime
)
```

### 2.6 Headers

[REQ-INST-019] Public headers MUST be installed using `install(DIRECTORY)` per SPEC-REPO-002 §11.4:

```cmake
install(DIRECTORY include/
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
    FILES_MATCHING
        PATTERN "*.h"
    COMPONENT devel
)
```

[REQ-INST-020] Private headers (`_p.h`) and internal headers (`detail/`) MUST NOT be installed. The install rule MUST exclude them:

```cmake
install(DIRECTORY include/
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
    FILES_MATCHING
        PATTERN "*.h"
    PATTERN "detail" EXCLUDE
    COMPONENT devel
)
```

[REQ-INST-021] The export header (`<Target>Export.h`) and version header (`<Target>Version.h`) MUST be installed alongside other public headers. They MUST NOT be excluded.

### 2.7 Resources

[REQ-INST-022] Qt resource files compiled into libraries or executables via `.qrc` are part of the binary and MUST NOT be installed separately.

[REQ-INST-023] Standalone resource files (icons, images, UI files not compiled into binaries) MUST be installed to `${CMAKE_INSTALL_DATADIR}/lingmo/<component>/`:

```cmake
install(DIRECTORY resources/
    DESTINATION ${CMAKE_INSTALL_DATADIR}/lingmo/<component>
    COMPONENT data
)
```

### 2.8 Translations

[REQ-INST-024] Compiled translation files (`.qm`) MUST be installed to `${CMAKE_INSTALL_DATADIR}/lingmo/locale/<appId>/`:

```cmake
qt_add_translations(LingmoTheme
    TS_FILES
        translations/lingmo-theme_de.ts
        translations/lingmo-theme_fr.ts
    QM_FILES_OUTPUT_VARIABLE qm_files
)
install(FILES ${qm_files}
    DESTINATION ${CMAKE_INSTALL_DATADIR}/lingmo/locale/lingmo-theme
    COMPONENT data
)
```

[REQ-INST-025] Translation source files (`.ts`) MUST NOT be installed. Only compiled `.qm` files are installed.

### 2.9 Documentation

[REQ-INST-026] API documentation (Doxygen HTML output) MUST be installed to `${CMAKE_INSTALL_DATADIR}/doc/lingmo/<component>/api/html/`:

```cmake
install(DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/docs/api/html/
    DESTINATION ${CMAKE_INSTALL_DATADIR}/doc/lingmo/<component>/api/html
    COMPONENT doc
)
```

[REQ-INST-027] Package-level documentation (README, changelog, copyright) MUST be installed following distribution conventions:

```cmake
install(FILES README.md
    DESTINATION ${CMAKE_INSTALL_DATADIR}/doc/lingmo-<component>
    COMPONENT doc
)
```

### 2.10 Examples

[REQ-INST-028] Example source code MUST be installed to `${CMAKE_INSTALL_DATADIR}/lingmo/examples/<component>/`:

```cmake
install(DIRECTORY examples/
    DESTINATION ${CMAKE_INSTALL_DATADIR}/lingmo/examples/<component>
    COMPONENT examples
)
```

[REQ-INST-029] Examples MUST be installed to the `examples` component. They MUST NOT be installed to `runtime` or `devel`.

[REQ-INST-030] Examples MUST NOT be compiled or built as part of the normal build. They are source-only artifacts.

### 2.11 Configuration Files

[REQ-INST-031] System configuration files MUST be installed to `${CMAKE_INSTALL_SYSCONFDIR}/lingmo/`:

```cmake
install(FILES lingmo-theme.conf
    DESTINATION ${CMAKE_INSTALL_SYSCONFDIR}/lingmo
    COMPONENT runtime
)
```

[REQ-INST-032] Default configuration files (fallback values, shipped with package) MUST be installed to `${CMAKE_INSTALL_DATADIR}/lingmo/defaults/`:

```cmake
install(FILES lingmo-theme.defaults.conf
    DESTINATION ${CMAKE_INSTALL_DATADIR}/lingmo/defaults
    COMPONENT runtime
)
```

### 2.12 CMake Package Config Files

[REQ-INST-033] Every library target MUST install its CMake package config files to `${CMAKE_INSTALL_LIBDIR}/cmake/<TargetName>/`:

```cmake
install(EXPORT LingmoThemeTargets
    FILE LingmoThemeTargets.cmake
    NAMESPACE Lingmo::
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/LingmoTheme
    COMPONENT devel
)

install(FILES
    "${CMAKE_CURRENT_BINARY_DIR}/LingmoThemeConfig.cmake"
    "${CMAKE_CURRENT_BINARY_DIR}/LingmoThemeConfigVersion.cmake"
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/LingmoTheme
    COMPONENT devel
)
```

[REQ-INST-034] CMake package config files MUST be installed to the `devel` component.

### 2.13 pkg-config Files

[REQ-INST-035] pkg-config files (`.pc`) MUST be installed to `${CMAKE_INSTALL_LIBDIR}/pkgconfig/`:

```cmake
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/LingmoTheme.pc
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig
    COMPONENT devel
)
```

[REQ-INST-036] pkg-config files are OPTIONAL for libraries that provide CMake config files. They MUST be provided if the library is expected to be consumed by non-CMake build systems.

### 2.14 Desktop Integration Files

[REQ-INST-037] Desktop integration files MUST be installed to their standardized locations per SPEC-PLATFORM-001 §8:

```cmake
# Desktop file
install(FILES org.lingmo.file-manager.desktop
    DESTINATION ${CMAKE_INSTALL_DATADIR}/applications
    COMPONENT runtime
)

# D-Bus service
install(FILES org.lingmo.Notification.service
    DESTINATION ${CMAKE_INSTALL_DATADIR}/dbus-1/services
    COMPONENT runtime
)

# systemd user unit
install(FILES lingmo-shell-core.service
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/systemd/user
    COMPONENT runtime
)

# AppStream metadata
install(FILES org.lingmo.file-manager.appdata.xml
    DESTINATION ${CMAKE_INSTALL_DATADIR}/metainfo
    COMPONENT runtime
)
```

[REQ-INST-038] All desktop integration files MUST be installed to the `runtime` component.

### 2.15 systemd Service Files

[REQ-INST-039] systemd user service units MUST be installed to `${CMAKE_INSTALL_LIBDIR}/systemd/user/`:

```cmake
install(FILES lingmo-session.service lingmo-session.target
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/systemd/user
    COMPONENT runtime
)
```

[REQ-INST-040] systemd service files MUST be installed to the `runtime` component.

---

## 3. Install Components

### 3.1 Mandatory Components

[REQ-INST-041] Every repository MUST define installs for at least the `runtime` component. All other components are OPTIONAL depending on the artifacts produced.

[REQ-INST-042] The following components are defined for the Lingmo Desktop project:

| Component | Contents | Installed By Default | Separately Installable |
|---|---|---|---|
| `runtime` | Shared libraries, executables, plugins, QML modules, config files, desktop integration files, systemd units, translations, resources | Yes | Yes |
| `devel` | Public headers, CMake config files, pkg-config files, static libraries, export headers, version headers | No (depends: yes with devel) | Yes |
| `data` | Architecture-independent data files (icons, sounds, wallpapers), resource files, default configs | Yes | Yes |
| `doc` | API documentation, changelogs, README, copyright files | No | Yes |
| `examples` | Example source code | No | Yes |
| `plug-<type>` | Plugin libraries and metadata (one component per plugin type) | No (depends: conditional) | Yes |
| `sdk` | SDK resources, templates, CMake helper modules | No | Yes |

### 3.2 Component Dependencies

[REQ-INST-043] Install components MUST declare their dependencies on other components:

| Component | Depends On | Rationale |
|---|---|---|
| `devel` | `runtime` | Headers require the corresponding runtime library |
| `examples` | `runtime`, `devel` | Examples need both library and headers to build |
| `doc` | (none) | Documentation is standalone |
| `plug-<type>` | `runtime` | Plugins depend on the runtime they extend |
| `sdk` | (none) | SDK resources are standalone |

### 3.3 CMake Component Specification

[REQ-INST-044] Every `install()` command MUST specify a `COMPONENT` argument. Install commands without a component are FORBIDDEN.

[REQ-INST-045] The component name MUST match one of the values from Section 3.1. Custom component names are FORBIDDEN unless approved by the architecture team.

[REQ-INST-046] Example of correct component usage:

```cmake
install(TARGETS LingmoTheme
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR} COMPONENT runtime
    PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/LingmoTheme COMPONENT devel
)
install(FILES org.lingmo.theme.desktop
    DESTINATION ${CMAKE_INSTALL_DATADIR}/applications COMPONENT runtime
)
```

---

## 4. CMake Installation Rules

### 4.1 Mandatory Commands

[REQ-INST-047] Every repository MUST include `GNUInstallDirs` before any `install()` command:

```cmake
include(GNUInstallDirs)
```

[REQ-INST-048] All destination paths in `install()` commands MUST use `GNUInstallDirs` variables. Literal paths are FORBIDDEN:

```cmake
# CORRECT
install(TARGETS LingmoTheme LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR} COMPONENT runtime)

# FORBIDDEN
install(TARGETS LingmoTheme LIBRARY DESTINATION /usr/lib COMPONENT runtime)
```

[REQ-INST-049] The `install(TARGETS)` command MUST be used for all CMake targets (executables, libraries, plugins). The `install(FILES)` and `install(DIRECTORY)` commands MUST be used for all non-target artifacts (headers, data, config, docs).

[REQ-INST-050] Generator expressions in install destinations MUST be avoided. If unavoidable, they MUST be documented with a rationale comment.

### 4.2 Export Registration

[REQ-INST-051] Every shared library target MUST register with `install(EXPORT ...)` per SPEC-REPO-003 §7.4:

```cmake
install(EXPORT LingmoThemeTargets
    FILE LingmoThemeTargets.cmake
    NAMESPACE Lingmo::
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/LingmoTheme
    COMPONENT devel
)
```

[REQ-INST-052] The `NAMESPACE` argument to `install(EXPORT)` MUST be `Lingmo::`.

[REQ-INST-053] Plugin and executable targets MUST NOT register with `install(EXPORT)`. Only library targets produce CMake package config files.

### 4.3 Build Interface vs Install Interface

[REQ-INST-054] Include directories MUST use generator expressions to differentiate build and install paths:

```cmake
target_include_directories(LingmoTheme
    PUBLIC
        "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>"
        "$<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/LingmoTheme>"
)
```

[REQ-INST-055] The `BUILD_INTERFACE` generator expression MUST reference the source directory path. The `INSTALL_INTERFACE` generator expression MUST reference the installed path relative to `${CMAKE_INSTALL_INCLUDEDIR}`.

---

## 5. Permissions

### 5.1 Executable Permissions

[REQ-INST-056] Executables installed to `/usr/bin/` and `/usr/libexec/` MUST have permissions `0755` (owner rxw, group rx, other rx).

[REQ-INST-057] CMake sets the executable bit automatically for `RUNTIME` targets. No manual permission manipulation is REQUIRED.

### 5.2 Library Permissions

[REQ-INST-058] Shared libraries MUST have permissions `0644` (owner rw, group r, other r). They MUST NOT have the executable bit set.

[REQ-INST-059] Static libraries MUST have permissions `0644`.

[REQ-INST-060] Plugin libraries MUST have permissions `0644`. They MUST NOT have the executable bit set.

### 5.3 Data and Header Permissions

[REQ-INST-061] All data files (translations, resources, config files, desktop files, systemd units) MUST have permissions `0644`.

[REQ-INST-062] Public header files MUST have permissions `0644`.

[REQ-INST-063] Documentation files MUST have permissions `0644`.

### 5.4 Generated File Permissions

[REQ-INST-064] Files generated by the build system (export headers, version headers, MOC output) acquire the permissions of their parent directory. No special handling is REQUIRED.

[REQ-INST-065] CMake package config files and pkg-config files generated by the build MUST have permissions `0644`.

### 5.5 Ownership

[REQ-INST-066] All files installed to system directories (`/usr/`, `/etc/`) MUST be owned by `root:root`.

[REQ-INST-067] Per-user files (installed to `$XDG_*` directories during runtime) MUST be owned by the running user. These files are created by Lingmo processes at runtime, not by package installation.

---

## 6. Versioned Artifacts

### 6.1 SONAME

[REQ-INST-068] Every shared library MUST set `VERSION` and `SOVERSION` properties:

```cmake
set_target_properties(LingmoTheme PROPERTIES
    VERSION ${PROJECT_VERSION}
    SOVERSION 1
)
```

[REQ-INST-069] The `SOVERSION` MUST be an integer (`1`, `2`, ...) that increments only when the public ABI breaks (SPEC-REPO-002 §6.1).

[REQ-INST-070] The `VERSION` SHOULD match the project version (`major.minor.patch`).

[REQ-INST-071] CMake automatically creates the required symlinks (`libLingmoTheme.so → libLingmoTheme.so.1 → libLingmoTheme.so.1.0.0`). Manual symlink creation is FORBIDDEN.

### 6.2 Version Headers

[REQ-INST-072] Every library MUST install a version header (`<Target>Version.h`) that defines version macros from the CMake project version (SPEC-REPO-002 §2.4).

[REQ-INST-073] The version header MUST be auto-generated by the build system, not hand-maintained.

### 6.3 Package Config Versioning

[REQ-INST-074] CMake package config version files MUST be generated using `write_basic_package_version_file` with `COMPATIBILITY SameMajorVersion` (SPEC-REPO-003 §7.4).

### 6.4 Plugin Versioning

[REQ-INST-075] Plugin shared libraries MUST NOT use SONAME versioning. The plugin API version is declared in the `.lingmo-plugin` metadata JSON and checked at runtime.

[REQ-INST-076] Plugin library filenames MUST NOT contain version suffixes:

```
/usr/lib/lingmo/plugins/applets/liblingmo_applet_clock.so    # CORRECT
/usr/lib/lingmo/plugins/applets/liblingmo_applet_clock.so.1  # FORBIDDEN
```

---

## 7. Optional Components

### 7.1 Examples

[REQ-INST-077] Examples MUST be built and installed only when `BUILD_EXAMPLES` is `ON` (default: `OFF`):

```cmake
option(BUILD_EXAMPLES "Build example programs" OFF)
if(BUILD_EXAMPLES)
    add_subdirectory(examples)
endif()
```

[REQ-INST-078] Example installs MUST specify `COMPONENT examples`.

### 7.2 Developer Tools

[REQ-INST-079] Developer CLI tools (Section 2.1) MUST be installed to `${CMAKE_INSTALL_BINDIR}` with `COMPONENT devel`:

```cmake
install(TARGETS lingmo-inspect
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    COMPONENT devel
)
```

### 7.3 SDK Resources

[REQ-INST-080] SDK resources (CMake templates, project wizards, helper scripts) MUST be installed to `${CMAKE_INSTALL_DATADIR}/lingmo/sdk/`:

```cmake
install(DIRECTORY cmake/ templates/
    DESTINATION ${CMAKE_INSTALL_DATADIR}/lingmo/sdk
    COMPONENT sdk
)
```

### 7.4 Benchmarks

[REQ-INST-081] Benchmark executables MUST NOT be installed. They MUST be built only for CI and local development:

```cmake
if(BUILD_TESTING)
    add_executable(bm_IconLoading bm_IconLoading.cpp)
    # NO install() for benchmark targets
endif()
```

### 7.5 Test Executables

[REQ-INST-082] Test executables MUST NOT be installed (SPEC-REPO-003 §10.2). Any `install()` call targeting a test target is FORBIDDEN.

---

## 8. Packaging Integration

### 8.1 Component-to-Package Mapping

[REQ-INST-083] The install component system MUST map to distribution packages as follows:

| Install Component | Debian Package | Fedora RPM Package |
|---|---|---|
| `runtime` | `lingmo-<component>` | `lingmo-<component>` |
| `devel` | `lingmo-<component>-dev` | `lingmo-<component>-devel` |
| `data` | `lingmo-<component>-data` | `lingmo-<component>-data` |
| `doc` | `lingmo-<component>-doc` | `lingmo-<component>-doc` |
| `examples` | `lingmo-<component>-examples` | `lingmo-<component>-examples` |
| `plug-<type>` | `lingmo-<component>-<type>` | `lingmo-<component>-<type>` |
| `sdk` | `lingmo-sdk` | `lingmo-sdk` |

[REQ-INST-084] The `runtime` component is the base package. All other component packages MUST declare a dependency on the `runtime` package.

### 8.2 Debian Packaging Integration

[REQ-INST-085] Debian `rules` files MUST use `dh_install` with component-based installation:

```makefile
override_dh_auto_configure:
    dh_auto_configure -- \
        -DCMAKE_BUILD_TYPE=RelWithDebInfo \
        -DBUILD_TESTING=ON

override_dh_install:
    dh_install --list-missing

override_dh_installinit:
    # systemd user units are handled by dh_systemd_enable
    true
```

[REQ-INST-086] Debian package dependencies MUST be computed from the CMake component dependencies:

| Debian Package | Depends |
|---|---|
| `lingmo-theme` | (direct deps) |
| `lingmo-theme-dev` | `lingmo-theme` (= ${binary:Version}) |
| `lingmo-theme-data` | `lingmo-theme` (>= ${source:Version}) |

### 8.3 Fedora RPM Packaging Integration

[REQ-INST-087] RPM spec files MUST use `%cmake_install` with component-based installation:

```spec
%install
%cmake_install

%files
%{_libdir}/libLingmoTheme.so.*

%files devel
%{_includedir}/LingmoTheme/
%{_libdir}/cmake/LingmoTheme/
%{_libdir}/libLingmoTheme.so

%files data
%{_datadir}/lingmo/themes/
```

[REQ-INST-088] RPM subpackage declarations MUST follow the component mapping in Section 8.1.

### 8.4 Inter-Package File Ownership

[REQ-INST-089] A package MUST NOT own files that belong to another package. Shared directories (e.g., `/usr/lib/lingmo/`) MUST be owned by the `lingmo-desktop` base package.

[REQ-INST-090] The `lingmo-desktop` meta-package MUST own the following directories:

```
/usr/lib/lingmo/
/usr/lib/lingmo/plugins/
/usr/lib/qt6/qml/LingmoUI/
/usr/share/lingmo/
```

---

## 9. Verification

### 9.1 Build-Time Verification

[REQ-INST-091] Every repository MUST include a CI step that verifies the install tree:

```yaml
- name: Verify install tree
  run: |
    cmake --install build --prefix /tmp/install-root
    test -d /tmp/install-root/usr/lib
    test -f /tmp/install-root/usr/lib/libLingmoTheme.so
```

### 9.2 Package Inspection

[REQ-INST-092] Every package MUST be inspected after build to verify:

- All expected files are present
- No unexpected files are present
- File permissions are correct
- SONAMEs are correct
- Symlinks are valid

[REQ-INST-093] Debian packages MUST pass `lintian` with no errors:

```bash
lintian --check-all lingmo-theme_*.deb
```

[REQ-INST-094] RPM packages MUST pass `rpmlint` with no errors:

```bash
rpmlint lingmo-theme-*.rpm
```

### 9.3 Install Tree Validation

[REQ-INST-095] The install tree validation script MUST check:

```bash
# Verify all installed files exist
for f in $(grep -oP 'install\(.*?DESTINATION \K\S+' CMakeLists.txt); do
    test -e "/tmp/install-root/$f" || echo "MISSING: $f"
done

# Verify no unexpected files
while IFS= read -r f; do
    echo "$f" | grep -qE '\.h$|\.so|\.conf|\.desktop|\.service' || echo "UNEXPECTED: $f"
done < <(find /tmp/install-root -type f)
```

[REQ-INST-096] The validation MUST run as part of the CI `package.yml` workflow.

### 9.4 Post-Install Verification

[REQ-INST-097] After installation on a real system, the following MUST be verified:

| Check | Command |
|---|---|
| Library is loadable | `ldconfig -p \| grep LingmoTheme` |
| Headers are findable | `ls /usr/include/LingmoTheme/` |
| CMake package is findable | `cmake --find-package -DNAME=LingmoTheme` |
| D-Bus service is registered | `dbus-send --session --dest=org.freedesktop.DBus --type=method_call --print-reply /org/freedesktop/DBus org.freedesktop.DBus.ListNames \| grep lingmo` |

---

## 10. Compliance Checklist

### 10.1 Executables

| ID | Requirement | Verification Method |
|---|---|---|
| REQ-INST-001 | Executables installed to correct destination | Package file listing |
| REQ-INST-002 | RUNTIME DESTINATION specified | CMake scan |
| REQ-INST-003 | Executable bit set | Package file listing |

### 10.2 Libraries

| ID | Requirement | Verification Method |
|---|---|---|
| REQ-INST-004 | Public libs to CMAKE_INSTALL_LIBDIR | Package file listing |
| REQ-INST-005 | PUBLIC_HEADER specified | CMake scan |
| REQ-INST-006 | Private libs to lib/lingmo | Package file listing |
| REQ-INST-007 | VERSION and SOVERSION set | CMake scan |
| REQ-INST-009 | Static libs to devel component | CMake scan |
| REQ-INST-010 | Static libs not in runtime | CMake scan |

### 10.3 Plugins and QML

| ID | Requirement | Verification Method |
|---|---|---|
| REQ-INST-012 | Plugins to lib/lingmo/plugins/<type>/ | Package file listing |
| REQ-INST-014 | KWin decor to qt6/plugins/org.kde.kwin.decoration/ | Package file listing |
| REQ-INST-015 | Plugin metadata installed | Package file listing |
| REQ-INST-016 | QML modules to lib/qt6/qml/ | Package file listing |
| REQ-INST-018 | QML modules to runtime component | CMake scan |

### 10.4 Headers and Development Files

| ID | Requirement | Verification Method |
|---|---|---|
| REQ-INST-019 | Headers via install(DIRECTORY) | CMake scan |
| REQ-INST-020 | Private/internal headers excluded | CMake scan |
| REQ-INST-033 | CMake config to lib/cmake/<Target>/ | Package file listing |
| REQ-INST-034 | CMake config to devel component | CMake scan |
| REQ-INST-035 | pkg-config files to lib/pkgconfig | Package file listing |

### 10.5 Components

| ID | Requirement | Verification Method |
|---|---|---|
| REQ-INST-041 | runtime component defined | CMake scan |
| REQ-INST-044 | COMPONENT in every install() | Automated CMake scan |
| REQ-INST-045 | Component names from approved list | Automated CMake scan |

### 10.6 CMake Rules

| ID | Requirement | Verification Method |
|---|---|---|
| REQ-INST-047 | GNUInstallDirs included | CMake scan |
| REQ-INST-048 | GNUInstallDirs variables used | CMake scan |
| REQ-INST-049 | install(TARGETS) for targets, install(FILES/DIRECTORY) for others | CMake scan |
| REQ-INST-051 | Library targets exported via install(EXPORT) | CMake scan |
| REQ-INST-052 | NAMESPACE Lingmo:: | CMake scan |

### 10.7 Permissions

| ID | Requirement | Verification Method |
|---|---|---|
| REQ-INST-056 | Executables 0755 | Package file listing |
| REQ-INST-058 | Libraries 0644 | Package file listing |
| REQ-INST-060 | Plugins 0644 | Package file listing |
| REQ-INST-061 | Data files 0644 | Package file listing |
| REQ-INST-066 | Files owned by root:root | Package file listing |

### 10.8 Versioning

| ID | Requirement | Verification Method |
|---|---|---|
| REQ-INST-068 | VERSION and SOVERSION set | CMake scan, readelf |
| REQ-INST-069 | SOVERSION integer | CMake scan |
| REQ-INST-072 | <Target>Version.h installed | File existence check |
| REQ-INST-074 | SameMajorVersion compatibility | CMake scan |

### 10.9 Optional Components

| ID | Requirement | Verification Method |
|---|---|---|
| REQ-INST-077 | Examples guarded by BUILD_EXAMPLES | CMake scan |
| REQ-INST-081 | Benchmarks not installed | CMake scan |
| REQ-INST-082 | Test executables not installed | CMake scan |

### 10.10 Packaging

| ID | Requirement | Verification Method |
|---|---|---|
| REQ-INST-083 | Component-package mapping correct | Package listing vs CMake |
| REQ-INST-089 | No cross-package file ownership | Package audit |
| REQ-INST-091 | CI verifies install tree | CI config inspection |
| REQ-INST-093 | lintian passes | CI output |
| REQ-INST-094 | rpmlint passes | CI output |
| REQ-INST-097 | Post-install verification | Manual test procedure |

---

## Appendix A: Complete Installation Matrix by Artifact Type

| Artifact Type | CMake Command | Destination | Component | Permissions | Versioned |
|---|---|---|---|---|---|
| User-facing executable | `install(TARGETS ... RUNTIME ...)` | `${CMAKE_INSTALL_BINDIR}` | `runtime` | `0755` | No |
| Internal executable | `install(TARGETS ... RUNTIME ...)` | `${CMAKE_INSTALL_LIBEXECDIR}` | `runtime` | `0755` | No |
| Public shared library | `install(TARGETS ... LIBRARY ...)` | `${CMAKE_INSTALL_LIBDIR}` | `runtime` | `0644` | SONAME |
| Private shared library | `install(TARGETS ... LIBRARY ...)` | `${CMAKE_INSTALL_LIBDIR}/lingmo` | `runtime` | `0644` | SONAME |
| Static library | `install(TARGETS ... ARCHIVE ...)` | `${CMAKE_INSTALL_LIBDIR}` | `devel` | `0644` | No |
| Plugin library | `install(TARGETS ... LIBRARY ...)` | `${CMAKE_INSTALL_LIBDIR}/lingmo/plugins/<type>/` | `plug-<type>` | `0644` | No |
| KWin decoration | `install(TARGETS ... LIBRARY ...)` | `${CMAKE_INSTALL_LIBDIR}/qt6/plugins/org.kde.kwin.decoration/` | `plug-kwin-decorations` | `0644` | No |
| QML module | `qt_add_qml_module(... INSTALL_DIRECTORY ...)` | `${CMAKE_INSTALL_LIBDIR}/qt6/qml/<ImportUri>/` | `runtime` | `0644` | No |
| Public headers | `install(DIRECTORY ...)` | `${CMAKE_INSTALL_INCLUDEDIR}` | `devel` | `0644` | No |
| Plugin metadata | `install(FILES ...)` | `${CMAKE_INSTALL_DATADIR}/lingmo/plugins/metadata` | `plug-<type>` | `0644` | No |
| Config files | `install(FILES ...)` | `${CMAKE_INSTALL_SYSCONFDIR}/lingmo` | `runtime` | `0644` | No |
| Default configs | `install(FILES ...)` | `${CMAKE_INSTALL_DATADIR}/lingmo/defaults` | `runtime` | `0644` | No |
| Translations | `install(FILES ... .qm)` | `${CMAKE_INSTALL_DATADIR}/lingmo/locale/<appId>/` | `data` | `0644` | No |
| Resources | `install(DIRECTORY ...)` | `${CMAKE_INSTALL_DATADIR}/lingmo/<component>/` | `data` | `0644` | No |
| Desktop files | `install(FILES ...)` | `${CMAKE_INSTALL_DATADIR}/applications` | `runtime` | `0644` | No |
| D-Bus services | `install(FILES ...)` | `${CMAKE_INSTALL_DATADIR}/dbus-1/services` | `runtime` | `0644` | No |
| systemd user units | `install(FILES ...)` | `${CMAKE_INSTALL_LIBDIR}/systemd/user` | `runtime` | `0644` | No |
| AppStream metadata | `install(FILES ...)` | `${CMAKE_INSTALL_DATADIR}/metainfo` | `runtime` | `0644` | No |
| API docs | `install(DIRECTORY ...)` | `${CMAKE_INSTALL_DATADIR}/doc/lingmo/<component>/api/html` | `doc` | `0644` | No |
| Examples | `install(DIRECTORY ...)` | `${CMAKE_INSTALL_DATADIR}/lingmo/examples/<component>/` | `examples` | `0644` | No |
| CMake config | `install(EXPORT ...)` + `install(FILES ...)` | `${CMAKE_INSTALL_LIBDIR}/cmake/<Target>/` | `devel` | `0644` | Yes |
| pkg-config | `install(FILES ... .pc)` | `${CMAKE_INSTALL_LIBDIR}/pkgconfig` | `devel` | `0644` | Yes |
| SDK resources | `install(DIRECTORY ...)` | `${CMAKE_INSTALL_DATADIR}/lingmo/sdk` | `sdk` | `0644` | No |

---

## Appendix B: Repository → Install Component Mapping

| Repository | runtime | devel | data | doc | examples | plug-* | sdk |
|---|---|---|---|---|---|---|---|
| `lingmo-theme` | libLingmoTheme.so | Headers, CMake config | Themes | ✅ | ✅ | — | — |
| `lingmo-config` | libLingmoConfig.so | Headers, CMake config | — | ✅ | ✅ | — | — |
| `lingmo-plugin` | libLingmoPlugin.so | Headers, CMake config | — | ✅ | — | — | — |
| `lingmo-ipc` | libLingmoIPC.so | Headers, CMake config | — | ✅ | — | — | — |
| `lingmo-utils` | libLingmoUtils.so | Headers, CMake config | — | ✅ | — | — | — |
| `lingmo-window` | libLingmoWindow.so | Headers, CMake config | — | ✅ | ✅ | — | — |
| `lingmo-icon` | libLingmoIcon.so | Headers, CMake config | Icon theme | ✅ | — | — | — |
| `lingmo-effects` | libLingmoEffects.so | Headers, CMake config | — | ✅ | — | — | — |
| `lingmo-style` | QStyle plugin | — | — | ✅ | — | — | — |
| `lingmo-controls` | QML module | — | — | ✅ | ✅ | — | — |
| `lingmo-shell-core` | lingmo-shell-core | — | — | ✅ | — | — | — |
| `lingmo-shell` | panel, desktop, overview | — | — | ✅ | — | — | — |
| `lingmo-session` | lingmo-session | — | — | ✅ | — | — | — |
| `lingmo-settings-daemon` | daemon + configs | — | — | ✅ | — | — | — |
| `lingmo-power-manager` | daemon + configs | — | — | ✅ | — | — | — |
| `lingmo-display-manager` | daemon + configs | — | — | ✅ | — | — | — |
| `lingmo-input-manager` | daemon + configs | — | — | ✅ | — | — | — |
| `lingmo-audio-manager` | daemon + configs | — | — | ✅ | — | — | — |
| `lingmo-notification` | daemon | — | Sounds | ✅ | — | — | — |
| `lingmo-osd` | daemon | — | — | ✅ | — | — | — |
| `lingmo-runner` | daemon | — | — | ✅ | — | — | — |
| `lingmo-thumbnails` | daemon | — | — | ✅ | — | — | — |
| `lingmo-print` | daemon | — | — | ✅ | — | — | — |
| `lingmo-indexer` | daemon | — | — | ✅ | — | — | — |
| `lingmo-lockscreen` | daemon | — | — | ✅ | — | — | — |
| `lingmo-polkit` | daemon | — | PolicyKit actions | ✅ | — | — | — |
| `lingmo-portals` | daemon | — | Portal defs | ✅ | — | — | — |
| All applet plugins | — | — | — | — | — | Metadata + .so | — |
| `lingmo-devtools` | — | CLI tools | — | ✅ | — | — | SDK resources |

---

## Appendix C: Common Installation Mistakes

| Mistake | Problem | Correct Approach |
|---|---|---|
| Using `/usr/lib/` literal path | Non-portable, breaks on Fedora multiarch | Use `${CMAKE_INSTALL_LIBDIR}` |
| Omitting `COMPONENT` in `install()` | Cannot create per-component packages (Debian/RPM) | Always specify `COMPONENT` |
| Installing test executables | Tests included in runtime packages | Never call `install()` for test targets |
| Installing `.ts` files | Source files shipped instead of compiled `.qm` | Install only `.qm` files |
| Using `install(TARGETS)` for data files | Only TARGETS supports LIBRARY/RUNTIME/ARCHIVE | Use `install(FILES)` or `install(DIRECTORY)` |
| Forgetting `PUBLIC_HEADER` | Headers not installed alongside library | Add `PUBLIC_HEADER DESTINATION ...` |
| Hand-writing `<Target>Export.h` | Inconsistent with CMake configuration | Use `generate_export_header` |
| No `SOVERSION` on shared lib | No ABI versioning | Set `VERSION` and `SOVERSION` properties |
| Installing plugin with SONAME | Plugin has unnecessary version symlinks | Do NOT set `SOVERSION` on MODULE libraries |
| Not excluding `detail/` from install | Internal headers leaked in packages | Add `PATTERN "detail" EXCLUDE` |

---

*Cross-ref*: SPEC-PLATFORM-001 (Filesystem Layout), SPEC-REPO-003 (CMake Organization), SPEC-REPO-002 (Header Organization), SPEC-REPO-001 (Repository Layout), SPEC-CORE-001 (Naming Convention)
