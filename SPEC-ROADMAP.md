# Lingmo Desktop — Specification Roadmap

> This document defines the complete specification system for Lingmo Desktop.
>
> It is NOT a specification itself. It is a roadmap that describes every
> specification document that must be written, their relationships,
> priorities, and estimated effort.
>
> **Status**: Living document
> **Version**: 1.0
> **Last Updated**: 2026-07-08

---

## Table of Contents

1. [Introduction](#introduction)
2. [Specification Repository Structure](#specification-repository-structure)
3. [Specification Document Lifecycle](#specification-document-lifecycle)
4. [Document Dependencies](#document-dependencies)
5. [Reading Order](#reading-order)
6. [Priority and Effort](#priority-and-effort)
7. [Complete Document Catalog](#complete-document-catalog)
8. [Specification Versioning](#specification-versioning)
9. [Maintenance Guide](#maintenance-guide)

---

## Introduction

### Purpose

This document serves as the master index for the Lingmo Desktop specification
system. It defines every specification document that must be written, the
relationships between them, the order in which they should be written and read,
and their priority relative to the implementation timeline.

### Scope

This roadmap covers the specification system only. It does not cover:

- Architecture decisions (covered in ARCHITECTURE-v3.md and individual ADRs)
- Implementation details (covered in repository source code)
- Project management or release planning

### Audience

- **Specification authors**: Use this roadmap to understand what to write next
- **Implementors**: Use this to find the authoritative specification for any
  aspect of Lingmo Desktop
- **Reviewers**: Use this to understand the relationships between specifications
  when reviewing changes
- **Third-party developers**: Use the reading order to understand the system

### Repository

All specification documents live in a single repository:

```
https://github.com/lingmo-desktop/specs
```

This is a separate repository from all implementation repositories.
It is the single source of truth for all normative requirements.

Individual implementation repositories reference spec documents by URL
and section number in their own documentation.

---

## Specification Repository Structure

```
specs/
├── README.md                         ← This file (the roadmap)
├── TEMPLATE.md                       ← Template for writing specs
├── GLOSSARY.md                       ← Common terms and definitions
│
├── platform/                         ← System-level specifications
│   ├── FS-LAYOUT.md                  # Filesystem hierarchy
│   ├── RUNTIME-LAYOUT.md             # Runtime process layout
│   ├── PROCESS-LIFECYCLE.md          # Process lifecycle requirements
│   ├── SESSION-SPEC.md               # Session manager requirements
│   ├── IPC-SPEC.md                   # Inter-process communication
│   ├── SERVICE-SPEC.md               # Service requirements and conventions
│   └── WAYLAND-PROTOCOLS.md          # Wayland protocol usage
│
├── ui/                               ← Visual design specifications
│   ├── DESIGN-LANGUAGE.md            # Design philosophy and principles
│   ├── THEME-SPEC.md                 # Theme system architecture
│   ├── THEME-TOKENS.md               # Complete token definitions
│   ├── COLOR-SYSTEM.md               # Color system and semantics
│   ├── TYPOGRAPHY.md                 # Typography system
│   ├── ICON-SPEC.md                  # Icon theme and icon engine
│   ├── ANIMATION-SPEC.md             # Animation and motion
│   ├── ACCESSIBILITY-SPEC.md         # Accessibility requirements
│   └── WINDOW-BEHAVIOR.md            # Window management policies
│
├── lingmoui/                         ← LingmoUI framework specifications
│   ├── WIDGET-SPEC.md                # Qt Widget rendering requirements
│   ├── QML-CONTROL-SPEC.md           # QML control requirements
│   ├── QSTYLE-SPEC.md                # QStyle implementation requirements
│   ├── COMPONENT-NAMING.md           # Naming conventions for components
│   ├── API-DESIGN.md                 # API design rules for LingmoUI
│   └── BINARY-COMPAT.md              # Binary compatibility requirements
│
├── plugins/                          ← Plugin system specifications
│   ├── SYSTEM.md                     # Plugin system architecture and ABI
│   └── SECURITY.md                   # Plugin sandboxing and security
│
├── sdk/                              ← Developer SDK specifications
│   ├── APP-SDK.md                    # Application SDK requirements
│   ├── PLUGIN-SDK.md                 # Plugin SDK requirements
│   ├── THEME-SDK.md                  # Theme creation SDK requirements
│   └── EXTENSION-SDK.md              # Extension API requirements
│
├── app-guidelines/                   ← Application development guidelines
│   ├── STARTUP.md                    # Application startup requirements
│   ├── CONFIG.md                     # Configuration conventions
│   ├── LOGGING.md                    # Logging requirements
│   ├── TRANSLATION.md                # Internationalization requirements
│   ├── CRASH-HANDLING.md             # Crash handling requirements
│   ├── SETTINGS.md                   # Settings storage conventions
│   ├── CLI.md                        # Command-line interface conventions
│   └── IPC-USAGE.md                  # IPC usage guidelines for apps
│
├── repo/                             ← Repository standards
│   ├── LAYOUT.md                     # Repository directory layout
│   ├── NAMING.md                     # Naming conventions for all artifacts
│   ├── CMAKE.md                      # CMake organization requirements
│   ├── INSTALL.md                    # Install rules and paths
│   ├── HEADERS.md                    # Public and private header conventions
│   └── TESTS.md                      # Test structure and requirements
│
├── coding/                           ← Coding standards
│   ├── CPP.md                        # C++ coding standards
│   ├── QML.md                        # QML coding standards
│   ├── RUST.md                       # Rust coding standards
│   └── GO.md                         # Go coding standards
│
├── packaging/                        ← Packaging specifications
│   ├── DEBIAN.md                     # Debian packaging requirements
│   ├── RPM.md                        # Fedora RPM packaging requirements
│   └── PATHS.md                      # Install path specifications
│
├── ci/                               ← CI/CD specifications
│   ├── WORKFLOWS.md                  # CI workflow requirements
│   ├── TEST-REQUIREMENTS.md          # Testing requirements
│   ├── COVERAGE.md                   # Code coverage requirements
│   ├── STATIC-ANALYSIS.md            # Static analysis requirements
│   └── RELEASE.md                    # Release workflow requirements
│
└── docs/                             ← Documentation standards
    ├── STANDARDS.md                  # Documentation style and format
    ├── ADR.md                        # Architecture Decision Record process
    ├── CHANGELOG.md                  # Changelog policy
    └── RELEASE-NOTES.md              # Release notes policy
```

**Total: 61 specification documents** (including README, TEMPLATE, GLOSSARY)

Every document below `specs/` follows the same format defined in `TEMPLATE.md`.

---

## Specification Document Lifecycle

### Document Format

Every specification document follows `TEMPLATE.md`:

```
┌──────────────────────────────────────────────────────────┐
│  Header Section                                           │
│  - Title                                                  │
│  - Version (MAJOR.MINOR)                                  │
│  - Status (Draft / Stable / Deprecated)                   │
│  - Last Updated                                            │
│  - Authors                                                 │
├──────────────────────────────────────────────────────────┤
│  1. Purpose                                               │
│  2. Scope (what this spec does and does not cover)        │
│  3. Normative References (to other specs or standards)    │
│  4. Terminology (definitions specific to this spec)       │
│  5. Specification (normative requirements)                │
│  6. Examples (illustrative, non-normative)                │
│  7. Version History                                       │
└──────────────────────────────────────────────────────────┘
```

### Lifecycle States

```
Draft ──→ Stable ──→ Deprecated
  │          │            │
  │          │            └──→ Removed (after 2 major versions)
  │          │
  │          └──→ Draft (when a breaking revision is needed)
  │
  └──→ Abandoned
```

- **Draft**: Under development, subject to change, not yet authoritative
- **Stable**: Approved, authoritative, changes require RFC
- **Deprecated**: Superseded, maintained for legacy reference only

### Change Process

1. **Minor changes** (clarifications, non-normative corrections): Direct PR with
   two approvals from specification editors
2. **Major changes** (new requirements, modified requirements): Must go through
   the RFC process defined in `docs/ADR.md`
3. **Breaking changes** (backward-incompatible requirements): Must be announced
   at least 6 months in advance, must go through ADR, must update the spec
   version's MAJOR number

---

## Document Dependencies

### Dependency Graph

The following diagram shows which documents reference which other documents.
An arrow from A to B means "A references B as a normative reference."

```
                             TEMPLATE.md
                                │
                                ▼
                ┌───────────────────────────────┐
                │     GLOSSARY.md                │
                │  (referenced by all docs)      │
                └───────────────┬───────────────┘
                                │
       ┌────────────────────────┼──────────────────────────┐
       │                        │                          │
       ▼                        ▼                          ▼
 ┌───────────┐           ┌────────────┐            ┌──────────────┐
 │ repo/     │           │ coding/    │            │ packaging/   │
 │ LAYOUT.md │           │ CPP.md     │            │ DEBIAN.md    │
 │ CMAKE.md  │           │ QML.md     │            │ RPM.md       │
 │ NAMING.md │           │ RUST.md    │            │ PATHS.md     │
 │ INSTALL.md│           │ GO.md      │            └──────────────┘
 │ HEADERS.md│           └────────────┘                    │
 │ TESTS.md  │                │                           │
 └───────────┘                │                           │
       │                      │                           │
       ▼                      ▼                           ▼
 ┌───────────┐           ┌────────────┐            ┌──────────────┐
 │ ci/       │           │ lingmoui/  │            │ platform/    │
 │ WORKFLOWS │           │ API-DESIGN │            │ FS-LAYOUT    │
 │   .md     │           │ BINARY-    │            │ RUNTIME-     │
 │ TEST-     │           │  COMPAT.md │            │  LAYOUT.md   │
 │ REQUIRE   │           │ WIDGET-SPEC│            │ PROCESS-     │
 │ MENTS.md  │           │ QML-CONTROL│            │  LIFECYCLE   │
 │ COVERAGE  │           │  -SPEC.md  │            │ SESSION-SPEC │
 │  .md      │           │ QSTYLE-    │            │ IPC-SPEC     │
 │ STATIC-   │           │  SPEC.md   │            │ SERVICE-SPEC │
 │  ANALYSIS │           │ COMPONENT- │            └──────────────┘
 │ RELEASE.md│           │  NAMING.md │                    │
 └───────────┘           └────────────┘                     │
       │                      │                             │
       │                      ▼                             │
       │              ┌──────────────┐                      │
       │              │ ui/          │                      │
       └──────────────│ DESIGN-      │                      │
                      │  LANGUAGE.md │                      │
                      │ THEME-SPEC   │                      │
                      │ THEME-TOKENS │                      │
                      │ COLOR-SYSTEM │                      │
                      │ TYPOGRAPHY   │                      │
                      │ ICON-SPEC    │                      │
                      │ ANIMATION-   │                      │
                      │  SPEC.md     │                      │
                      │ ACCESSIBILITY                      │
                      │ WINDOW-      │                      │
                      │  BEHAVIOR.md │                      │
                      └──────────────┘                      │
                            │                              │
                            ▼                              ▼
                      ┌──────────────┐              ┌──────────────┐
                      │ plugins/     │              │ sdk/         │
                      │ SYSTEM.md    │              │ APP-SDK.md   │
                      │ SECURITY.md  │              │ PLUGIN-SDK   │
                      └──────────────┘              │ THEME-SDK    │
                            │                       │ EXTENSION-SDK│
                            ▼                       └──────────────┘
                      ┌──────────────┐
                      │ app-         │
                      │  guidelines/ │
                      │ STARTUP.md   │
                      │ CONFIG.md    │
                      │ LOGGING.md   │
                      │ TRANSLATION  │
                      │ CRASH-       │
                      │  HANDLING.md │
                      │ SETTINGS.md  │
                      │ CLI.md       │
                      │ IPC-USAGE.md │
                      └──────────────┘
```

### Dependency Rules for Specifications

1. A specification MAY reference any document in the same directory or any
   lower-level directory (alphabetically by directory name: coding < docs <
   lingmoui < packaging < platform < plugins  <repo  =sdk < app-guidelines <
   ci < ui)

2. A specification MUST NOT reference a document that depends on it
   (no circular references).

3. A specification MUST list all normative references in its "Normative References"
   section.

4. A specification SHOULD list informative references (other specs that provide
   useful context but are not required for conformance) separately.

---

## Reading Order

### For New Contributors

Read in this order to understand the complete system:

```
Phase 1 (Foundation — read these first):
  1.  GLOSSARY.md
  2.  repo/LAYOUT.md
  3.  repo/NAMING.md
  4.  repo/CMAKE.md
  5.  coding/CPP.md
  6.  coding/QML.md

Phase 2 (System architecture):
  7.  platform/FS-LAYOUT.md
  8.  platform/RUNTIME-LAYOUT.md
  9.  platform/SESSION-SPEC.md
  10. platform/IPC-SPEC.md
  11. platform/SERVICE-SPEC.md
  12. platform/PROCESS-LIFECYCLE.md

Phase 3 (Visual system):
  13. ui/DESIGN-LANGUAGE.md
  14. ui/THEME-SPEC.md
  15. ui/THEME-TOKENS.md
  16. ui/COLOR-SYSTEM.md
  17. ui/TYPOGRAPHY.md
  18. ui/ICON-SPEC.md
  19. ui/ANIMATION-SPEC.md
  20. ui/ACCESSIBILITY-SPEC.md
  21. ui/WINDOW-BEHAVIOR.md

Phase 4 (UI framework):
  22. lingmoui/API-DESIGN.md
  23. lingmoui/COMPONENT-NAMING.md
  24. lingmoui/BINARY-COMPAT.md
  25. lingmoui/WIDGET-SPEC.md
  26. lingmoui/QML-CONTROL-SPEC.md
  27. lingmoui/QSTYLE-SPEC.md

Phase 5 (Extensions and integration):
  28. plugins/SYSTEM.md
  29. plugins/SECURITY.md
  30. sdk/APP-SDK.md
  31. sdk/PLUGIN-SDK.md
  32. sdk/THEME-SDK.md

Phase 6 (Application development):
  33. app-guidelines/STARTUP.md
  34. app-guidelines/CONFIG.md
  35. app-guidelines/CLI.md
  36. app-guidelines/LOGGING.md
  37. app-guidelines/TRANSLATION.md
  38. app-guidelines/CRASH-HANDLING.md
  39. app-guidelines/SETTINGS.md
  40. app-guidelines/IPC-USAGE.md

Phase 7 (Infrastructure):
  41. repo/INSTALL.md
  42. repo/HEADERS.md
  43. repo/TESTS.md
  44. packaging/DEBIAN.md
  45. packaging/RPM.md
  46. packaging/PATHS.md
  47. ci/WORKFLOWS.md
  48. ci/TEST-REQUIREMENTS.md
  49. ci/COVERAGE.md
  50. ci/STATIC-ANALYSIS.md
  51. ci/RELEASE.md

Phase 8 (Documentation):
  52. docs/STANDARDS.md
  53. docs/ADR.md
  54. docs/CHANGELOG.md
  55. docs/RELEASE-NOTES.md

Phase 9 (Advanced):
  56. sdk/EXTENSION-SDK.md
  57. coding/RUST.md
  58. coding/GO.md
  59. platform/WAYLAND-PROTOCOLS.md
```

### For Specific Tasks

| Task | Documents to Read |
|------|------------------|
| Write a new application | app-guidelines/*, sdk/APP-SDK.md, ui/THEME-TOKENS.md, lingmoui/API-DESIGN.md |
| Write a new applet plugin | plugins/SYSTEM.md, sdk/PLUGIN-SDK.md, lingmoui/COMPONENT-NAMING.md |
| Create a custom theme | ui/THEME-SPEC.md, ui/THEME-TOKENS.md, ui/COLOR-SYSTEM.md |
| Add a new daemon | platform/SERVICE-SPEC.md, platform/IPC-SPEC.md, repo/CMAKE.md |
| Add a new QML control | lingmoui/QML-CONTROL-SPEC.md, lingmoui/API-DESIGN.md, coding/QML.md |
| Fix a packaging bug | packaging/DEBIAN.md, packaging/RPM.md, packaging/PATHS.md, repo/INSTALL.md |
| Review a PR for API compliance | coding/CPP.md, lingmoui/API-DESIGN.md, lingmoui/BINARY-COMPAT.md |

---

## Priority and Effort

### Priority Definitions

| Priority | Label | Meaning |
|----------|-------|---------|
| P0 | Foundation | Required before ANY implementation can begin |
| P1 | First Release | Must be stable for the first desktop release |
| P2 | Pre-1.0 | Must be stable before version 1.0 |
| P3 | Post-1.0 | Required for ecosystem growth after 1.0 |
| P4 | Future | Desirable but no timeline |

### Effort Scale

| Size | Person-days | Description |
|------|-------------|-------------|
| XS | 1 day | Single topic, well-understood, few requirements |
| S | 2-3 days | Requires some research, one author |
| M | 4-5 days | Requires cross-team input, significant research |
| L | 6-10 days | Complex topic, multiple drafts, extensive review |
| XL | 11-20 days | Cross-cutting impact, requires RFC process |

### Complete Document Catalog

#### specs/ (root directory)

| Document | Priority | Effort | Dependencies | Description |
|----------|----------|--------|--------------|-------------|
| `README.md` | P0 | XS | none | This roadmap document |
| `TEMPLATE.md` | P0 | XS | none | Template for writing new specs |
| `GLOSSARY.md` | P0 | XS | none | Common terms and definitions |

---

#### platform/ (7 documents — system-level)

| # | Document | Priority | Effort | Key Dependencies | Summary |
|---|----------|----------|--------|-----------------|---------|
| 1 | `FS-LAYOUT.md` | P0 | M | ARCHITECTURE-v3.md | Filesystem hierarchy: all install paths, config paths, cache paths, runtime paths. MUST, SHOULD, MAY for every directory |
| 2 | `RUNTIME-LAYOUT.md` | P1 | M | FS-LAYOUT.md | Runtime structure: socket paths, PID file locations, lock file conventions, temporary file conventions |
| 3 | `PROCESS-LIFECYCLE.md` | P1 | M | RUNTIME-LAYOUT.md, SERVICE-SPEC.md | Process states, startup ordering, shutdown ordering, watchdog requirements, readiness protocol (sd_notify) |
| 4 | `SESSION-SPEC.md` | P1 | M | PROCESS-LIFECYCLE.md, IPC-SPEC.md | Session startup sequence (timing), shutdown sequence, crash recovery state machine, multi-seat support |
| 5 | `IPC-SPEC.md` | P1 | M | SERVICE-SPEC.md | D-Bus interface design rules, message size limits, serialization format, timeout requirements, delivery guarantees |
| 6 | `SERVICE-SPEC.md` | P1 | M | FS-LAYOUT.md | Service requirements: D-Bus activation vs systemd activation, readiness, restart policies, logging requirements, resource limits |
| 7 | `WAYLAND-PROTOCOLS.md` | P2 | S | IPC-SPEC.md | Which wlr-* protocols are used, custom protocol XML files, protocol version requirements, interface version negotiation |

---

#### ui/ (9 documents — visual design)

| # | Document | Priority | Effort | Key Dependencies | Summary |
|---|----------|----------|--------|-----------------|---------|
| 8 | `DESIGN-LANGUAGE.md` | P2 | M | GLOSSARY.md | Design principles (e.g., "content first", "progressive disclosure"), visual hierarchy rules, density guidelines, brand usage rules |
| 9 | `THEME-SPEC.md` | P2 | M | COLOR-SYSTEM.md, TYPOGRAPHY.md | Theme file format (JSON schema), theme loading order, fallback chain, theme inheritance, dark/light/high-contrast requirements |
| 10 | `THEME-TOKENS.md` | P2 | L | GLOSSARY.md | Every design token defined: name, type, expected range, semantic meaning. Token categories: palette, typography, spacing, radius, shadow, animation, elevation, opacity, icon. No implementation — pure data spec |
| 11 | `COLOR-SYSTEM.md` | P2 | L | THEME-TOKENS.md | Color roles and their usage, contrast ratios (WCAG AA/AAA), dark/light mapping, accent color rules, semantic color usage, blur/transparency requirements |
| 12 | `TYPOGRAPHY.md` | P2 | L | THEME-TOKENS.md | Font families per platform, size scale (modular scale), weight mapping, line height ratios, role assignments (body/heading/monospace/UI), font feature requirements |
| 13 | `ICON-SPEC.md` | P2 | M | THEME-TOKENS.md | Icon sizes for each context, icon naming (XDG compliance), symbolic icon coloring rules, fallback chain order, animation requirements for cursors |
| 14 | `ANIMATION-SPEC.md` | P2 | S | THEME-TOKENS.md | Duration table for every interaction type, easing curve definitions, spring physics parameters, choreography rules (stagger, simultaneous), reduced-motion requirements |
| 15 | `ACCESSIBILITY-SPEC.md` | P2 | S | COLOR-SYSTEM.md, TYPOGRAPHY.md | WCAG 2.1 AA/AAA requirements per component type, keyboard navigation requirements, screen reader integration (AT-SPI2), focus indicator requirements, color-blind safe palettes |
| 16 | `WINDOW-BEHAVIOR.md` | P3 | M | IPC-SPEC.md | Window placement policies, focus model (click-to-focus/focus-follows-mouse), multi-monitor behavior, virtual desktop behavior, window snapping rules, window maximize/minimize/close behavior |

---

#### lingmoui/ (6 documents — UI framework)

| # | Document | Priority | Effort | Key Dependencies | Summary |
|---|----------|----------|--------|-----------------|---------|
| 17 | `WIDGET-SPEC.md` | P2 | M | THEME-TOKENS.md, API-DESIGN.md | Every Qt Widget must be listed with its themed appearance. For each widget: which QStyle elements control it, minimum/maximum sizes, spacing, font roles, color roles, focus behavior |
| 18 | `QML-CONTROL-SPEC.md` | P2 | M | THEME-TOKENS.md, API-DESIGN.md | Every QML control must be listed with its visual specification. For each control: states (normal/hover/pressed/disabled/focused), size constraints, layout rules, interaction patterns |
| 19 | `QSTYLE-SPEC.md` | P2 | M | THEME-TOKENS.md, API-DESIGN.md | Every QStyle::PrimitiveElement, ControlElement, and ComplexControl that LingmoStyle must implement. Drawing requirements for each: antialiasing rules, padding, gradient/flat color choices |
| 20 | `COMPONENT-NAMING.md` | P1 | XS | GLOSSARY.md | Naming conventions for QML types, C++ classes, enum values, QML properties, CSS-like selectors. Prefix rules, suffix rules, namespace allocation. Prevents naming inconsistencies across 47 repos |
| 21 | `API-DESIGN.md` | P1 | M | COMPONENT-NAMING.md, BINARY-COMPAT.md | Rules for designing public APIs: when to use Q_PROPERTY vs getter/setter, enum vs flags, QObject hierarchy rules, signal/slot design patterns, include file conventions |
| 22 | `BINARY-COMPAT.md` | P1 | S | API-DESIGN.md | Rules for maintaining binary compatibility: when new virtual methods can be added, when methods can be deprecated, how to use d-pointer, export macros, inline method rules, template export rules |

---

#### plugins/ (2 documents — plugin system)

| # | Document | Priority | Effort | Key Dependencies | Summary |
|---|----------|----------|--------|-----------------|---------|
| 23 | `SYSTEM.md` | P1 | M | API-DESIGN.md, BINARY-COMPAT.md | Plugin metadata JSON schema, discovery paths and order, loading algorithm (symbol resolution, version checking), lifecycle state machine (discovered → loaded → initialized → running → shutdown), ABI requirements for interface classes, plugin versioning scheme |
| 24 | `SECURITY.md` | P1 | L | SYSTEM.md | Plugin sandboxing requirements: in-process vs out-of-process, seccomp filter rules for out-of-process plugins, filesystem access restrictions, network access restrictions, user confirmation requirements for permission elevation, signing and verification requirements |

---

#### sdk/ (4 documents — developer SDK)

| # | Document | Priority | Effort | Key Dependencies | Summary |
|---|----------|----------|--------|-----------------|---------|
| 25 | `APP-SDK.md` | P2 | M | API-DESIGN.md, lingmoui/*, ui/* | What the application SDK provides: Lingmo::Application class API, Lingmo::Window class API, Lingmo::Config scope API. Requirements for SDK stability. Deprecation policy for SDK APIs |
| 26 | `PLUGIN-SDK.md` | P2 | M | plugins/SYSTEM.md | What the plugin SDK provides: base classes for each plugin type, CMake macros for plugin building, template files. Requirements for plugin SDK versioning. Plugin packaging requirements |
| 27 | `THEME-SDK.md` | P2 | L | ui/* | How to create a Lingmo theme: theme file structure, JSON schema, validation tools, testing requirements, distribution packaging. Requirements for theme fallback and partial theme support |
| 28 | `EXTENSION-SDK.md` | P4 | L | plugins/SECURITY.md, sdk/APP-SDK.md | Long-term extension API: how third-party developers extend Lingmo Desktop beyond plugins. Extension manifest format, extension distribution, extension lifecycle, permission model |

---

#### app-guidelines/ (8 documents — application conventions)

| # | Document | Priority | Effort | Key Dependencies | Summary |
|---|----------|----------|--------|-----------------|---------|
| 29 | `STARTUP.md` | P2 | M | sdk/APP-SDK.md | Application startup sequence: Lingmo::Application initialization order, command-line parsing, single-instance detection, D-Bus registration, plugin loading, window creation, ready signal. Startup time budget |
| 30 | `CONFIG.md` | P1 | S | repo/LAYOUT.md | Configuration file location, format (INI/JSON), schema conventions, default value location, per-user vs per-system vs per-session config, migration path for config format changes |
| 31 | `LOGGING.md` | P1 | M | GLOSSARY.md | Log category naming, log level usage rules (trace/debug/info/warn/error/fatal), log message format, PII redaction requirements, journald integration, log file fallback, log rotation |
| 32 | `TRANSLATION.md` | P1 | M | GLOSSARY.md | Translation file locations, Qt Linguist workflow, translation domains, context markers, plural form handling, right-to-left language support, script/locale tags |
| 33 | `CRASH-HANDLING.md` | P1 | M | PROCESS-LIFECYCLE.md | Crash handler installation requirements, signal list to catch, backtrace generation format, crash report file format, crash report location, user consent requirements for crash reporting |
| 34 | `SETTINGS.md` | P1 | XS | CONFIG.md | Settings panel integration: how applications expose settings to LingmoControlCenter, settings schema format, settings change notification protocol, default settings location |
| 35 | `CLI.md` | P1 | S | CONFIG.md | Standard CLI options that all Lingmo applications MUST support (--help, --version, --theme, --config, --safe-mode, --verbose, --platform, --display). Option parsing rules. Exit codes. Output format conventions |
| 36 | `IPC-USAGE.md` | P2 | M | IPC-SPEC.md | How applications should use IPC: permitted D-Bus interfaces to call, prohibited calls (no direct KWin D-Bus access), D-Bus timeout recommendations, async call patterns, fallback when service unavailable |

---

#### repo/ (6 documents — repository standards)

| # | Document | Priority | Effort | Key Dependencies | Summary |
|---|----------|----------|--------|-----------------|---------|
| 37 | `LAYOUT.md` | P0 | S | none | Standard directory layout for every repository: top-level CMakeLists.txt, src/, tests/, tools/, debian/, rpm/, .github/, docs/, cmake/, examples/. Purpose of each directory. Files that MUST exist |
| 38 | `NAMING.md` | P0 | XS | LAYOUT.md | Repository naming conventions (lingmo-<component>), binary naming, library naming (libLingmo<Module>.so), package naming, RPM package naming, QML module naming, header naming |
| 39 | `CMAKE.md` | P0 | S | LAYOUT.md, NAMING.md | CMake minimum version, project() requirements, C++ standard requirements, find_package requirements, target_* requirements, install() requirements, export requirements. Template CMakeLists.txt files for libraries, QML modules, executables, plugins, tests |
| 40 | `INSTALL.md` | P0 | S | CMAKE.md, FS-LAYOUT.md | Install rules for each target type: libraries (libdir), executables (libexecdir), plugins (libdir/lingmo/plugins/), QML modules (libdir/lingmo/qml/), headers (includedir/Lingmo/), config files (sysconfdir/lingmo/), data files (datadir/lingmo/) |
| 41 | `HEADERS.md` | P0 | XS | NAMING.md, CMAKE.md | Public header conventions: include guard format, namespace conventions, #include order, forward declaration rules, doxygen requirements, private header naming (*_p.h), detail/ directory conventions |
| 42 | `TESTS.md` | P0 | S | CMAKE.md, HEADERS.md | Test directory structure, test naming conventions, test framework requirements (Qt Test for C++, qmltest for QML), test environment requirements, mock conventions, CI test integration |

---

#### coding/ (4 documents — coding standards)

| # | Document | Priority | Effort | Key Dependencies | Summary |
|---|----------|----------|--------|-----------------|---------|
| 43 | `CPP.md` | P0 | M | HEADERS.md | C++ standard version (C++20), naming conventions (classes, methods, variables, namespaces, macros), formatting (clang-format rules), memory ownership model, threading rules, error handling (exceptions vs error codes), logging requirements, Qt best practices |
| 44 | `QML.md` | P1 | M | API-DESIGN.md | QML coding conventions: property naming, signal naming, import statement ordering, anchor usage vs layout, component file naming, JavaScript coding conventions within QML, performance rules (binding avoidance, model optimization) |
| 45 | `RUST.md` | P4 | S | GLOSSARY.md | Rust version (2024 edition), integration with Qt (CXX bridge or similar), naming conventions, error handling (Result/Option), async usage, panic safety, FFI conventions |
| 46 | `GO.md` | P4 | S | GLOSSARY.md | Go version, integration with D-Bus, naming conventions, error handling, concurrency rules, package organization, FFI with C |

---

#### packaging/ (3 documents — packaging)

| # | Document | Priority | Effort | Key Dependencies | Summary |
|---|----------|----------|--------|-----------------|---------|
| 47 | `DEBIAN.md` | P1 | S | INSTALL.md, CMAKE.md | Debian source package format (`3.0 (native)`), control file fields, dependencies (Build-Depends, Depends), rules file (dh), symbols file requirements, library packaging guidelines, lintian overrides policy |
| 48 | `RPM.md` | P1 | S | INSTALL.md, CMAKE.md | RPM spec file structure, Name/Version/Release conventions, BuildRequires/Requires, %files section, %post/%postun for ldconfig, %check for tests, Fedora packaging guidelines compliance |
| 49 | `PATHS.md` | P1 | S | FS-LAYOUT.md, INSTALL.md | Complete path mapping table: CMake variable → filesystem path → Debian path → RPM path → internal path (e.g., `${CMAKE_INSTALL_LIBEXECDIR}` → `/usr/libexec` → Debian package path → RPM %_libexecdir). One canonical source of truth for all path decisions |

---

#### ci/ (5 documents — CI/CD)

| # | Document | Priority | Effort | Key Dependencies | Summary |
|---|----------|----------|--------|-----------------|---------|
| 50 | `WORKFLOWS.md` | P1 | M | CMAKE.md, TEST-REQUIREMENTS.md | Required GitHub Actions workflow files. Triggers (push, PR, schedule, tag). Job matrix (arch, config, distro). Required steps. Artifact upload requirements. Secret management. Deployment targets |
| 51 | `TEST-REQUIREMENTS.md` | P1 | M | CMAKE.md | Test types required (unit, integration, UI, screenshot, Wayland, X11). Minimum pass thresholds. Test time budgets. Flaky test handling. Required test infrastructure (virtual displays, KWin headless) |
| 52 | `COVERAGE.md` | P3 | S | TEST-REQUIREMENTS.md | Code coverage targets by component type (library ≥ 80%, application ≥ 60%). Coverage tool requirements (gcov/lcov). Coverage reporting format. Coverage gate requirements for CI. Exemption process |
| 53 | `STATIC-ANALYSIS.md` | P3 | S | CPP.md, QML.md | Required tools (clang-tidy, clang-format, qmllint, cmake-lint, cppcheck). Rule configuration per repository. CI integration. Warning-as-error policy. Analyzer version pinning |
| 54 | `RELEASE.md` | P2 | S | WORKFLOWS.md, packaging/* | Release workflow triggers (tag push). Branch protection rules. Package generation steps. Signature requirements. Artifact publishing targets (GitHub Releases, APT repo, COPR). Release notes generation |

---

#### docs/ (4 documents — documentation standards)

| # | Document | Priority | Effort | Key Dependencies | Summary |
|---|----------|----------|--------|-----------------|---------|
| 55 | `STANDARDS.md` | P0 | S | GLOSSARY.md | Markdown formatting rules (cmark-gfm), heading structure, link conventions, code block language tags, image conventions, table formatting, note/warning/caution callouts, cross-reference format (`[SECTION.md#section-name]`) |
| 56 | `ADR.md` | P0 | XS | GLOSSARY.md | Architecture Decision Record format: Number, Title, Status, Context, Decision, Consequences, Compliance. Numbering scheme (0001-9999). Storage location (docs/adr/ in each repo). Review process. Supersession rules |
| 57 | `CHANGELOG.md` | P0 | XS | GLOSSARY.md | Changelog format (Keep a Changelog v1.1.0), change categories (Added, Changed, Deprecated, Removed, Fixed, Security), version header format, release date format, unreleased section policy, credit policy |
| 58 | `RELEASE-NOTES.md` | P2 | S | ADR.md, CHANGELOG.md | Release notes format, what must be included (breaking changes, new features, deprecations, bug fixes, known issues), per-component vs unified notes, translation requirements, distribution format |

---

### Effort Summary by Priority

| Priority | Count | Total Days | Equivalent Person-Months |
|----------|-------|-----------|------------------------|
| P0 | 14 documents | 20 days | 1.0 |
| P1 | 21 documents | 70 days | 3.5 |
| P2 | 16 documents | 86 days | 4.3 |
| P3 | 5 documents | 20 days | 1.0 |
| P4 | 3 documents | 18 days | 0.9 |
| **Total** | **59 documents** | **214 days** | **10.7** |

### Recommended Writing Timeline

With 3 specification authors working in parallel:

```
Month 1 (P0):
  Author A: repo/ (LAYOUT, NAMING, CMAKE, INSTALL, HEADERS, TESTS) = 6 S = 15 days
  Author B: coding/ (CPP, QML) + platform/ (FS-LAYOUT) = 3 M = 15 days
  Author C: specs/ (README, TEMPLATE, GLOSSARY) + docs/ (STANDARDS, ADR, CHANGELOG) = 4 XS + 3 S = 10 days
            + platform/ (RUNTIME-LAYOUT) = 1 S = 3 days
  → P0 complete: 15 docs in 1 month

Month 2-3 (P1):
  Author A: platform/ (PROCESS-LIFECYCLE, SESSION-SPEC, IPC-SPEC, SERVICE-SPEC) = 4 M = 20 days
            + packaging/ (DEBIAN, RPM, PATHS) = 3 S = 6 days
  Author B: lingmoui/ (API-DESIGN, BINARY-COMPAT, COMPONENT-NAMING) = 2 M + 1 XS = 10 days
            + plugins/ (SYSTEM, SECURITY) = 1 M + 1 L = 12 days
  Author C: app-guidelines/ (CONFIG, LOGGING, TRANSLATION, CRASH-HANDLING, SETTINGS, CLI) = 4 S + 2 M = 22 days
            + ci/ (WORKFLOWS, TEST-REQUIREMENTS) = 2 M = 10 days
  → P1 complete: 21 docs in 2 months

Month 4-5 (P2):
  Author A: ui/ (THEME-SPEC, THEME-TOKENS, COLOR-SYSTEM) = 1 M + 2 L = 24 days
  Author B: ui/ (TYPOGRAPHY, ICON-SPEC, ANIMATION-SPEC, DESIGN-LANGUAGE, WINDOW-BEHAVIOR) = 3 M + 2 S = 21 days
            + ui/ (ACCESSIBILITY-SPEC) = 1 S = 3 days
  Author C: lingmoui/ (WIDGET-SPEC, QML-CONTROL-SPEC, QSTYLE-SPEC) = 3 M = 15 days
            + sdk/ (APP-SDK, PLUGIN-SDK, THEME-SDK) = 2 M + 1 L = 20 days
  → P2 complete: 16 docs in 2 months

→ Total: 5 months for all P0-P2 documents (51 documents)
→ Remaining: P3-P4 documents (8 docs) can be written incrementally post-1.0
```

---

## Complete Document Catalog

### Overview Table

| Dir | # | Document | Version Depends On | Writing When | Review Required From |
|-----|---|----------|-------------------|-------------|---------------------|
| root | R1 | README.md | v1 | First | All architects |
| root | R2 | TEMPLATE.md | v1 | First | Documentation team |
| root | R3 | GLOSSARY.md | v1 | First | All teams |
| platform | P1 | FS-LAYOUT.md | v1 | P0 | Release engineers, packagers |
| platform | P2 | RUNTIME-LAYOUT.md | v1 | P1 | Release engineers |
| platform | P3 | PROCESS-LIFECYCLE.md | v1 | P1 | System engineers |
| platform | P4 | SESSION-SPEC.md | v1 | P1 | Session team |
| platform | P5 | IPC-SPEC.md | v1 | P1 | IPC/system engineers |
| platform | P6 | SERVICE-SPEC.md | v1 | P1 | Service developers |
| platform | P7 | WAYLAND-PROTOCOLS.md | v1 | P2 | Compositor team |
| ui | U1 | DESIGN-LANGUAGE.md | v1 | P2 | Design team |
| ui | U2 | THEME-SPEC.md | v1 | P2 | Design team, theme developers |
| ui | U3 | THEME-TOKENS.md | v1 | P2 | Design team |
| ui | U4 | COLOR-SYSTEM.md | v1 | P2 | Design team, a11y team |
| ui | U5 | TYPOGRAPHY.md | v1 | P2 | Design team |
| ui | U6 | ICON-SPEC.md | v1 | P2 | Design team, icon artists |
| ui | U7 | ANIMATION-SPEC.md | v1 | P2 | Design team, motion designers |
| ui | U8 | ACCESSIBILITY-SPEC.md | v1 | P2 | a11y team, design team |
| ui | U9 | WINDOW-BEHAVIOR.md | v1 | P3 | Shell team |
| lingmoui | L1 | WIDGET-SPEC.md | v1 | P2 | UI framework team |
| lingmoui | L2 | QML-CONTROL-SPEC.md | v1 | P2 | UI framework team |
| lingmoui | L3 | QSTYLE-SPEC.md | v1 | P2 | UI framework team |
| lingmoui | L4 | COMPONENT-NAMING.md | v1 | P1 | All teams |
| lingmoui | L5 | API-DESIGN.md | v1 | P1 | All library maintainers |
| lingmoui | L6 | BINARY-COMPAT.md | v1 | P1 | Library maintainers |
| plugins | PL1 | SYSTEM.md | v1 | P1 | Plugin framework team |
| plugins | PL2 | SECURITY.md | v1 | P1 | Security team |
| sdk | S1 | APP-SDK.md | v1 | P2 | SDK team, app developers |
| sdk | S2 | PLUGIN-SDK.md | v1 | P2 | Plugin developers |
| sdk | S3 | THEME-SDK.md | v1 | P2 | Theme developers |
| sdk | S4 | EXTENSION-SDK.md | v1 | P4 | SDK team |
| app-guidelines | A1 | STARTUP.md | v1 | P2 | App developers |
| app-guidelines | A2 | CONFIG.md | v1 | P1 | All developers |
| app-guidelines | A3 | LOGGING.md | v1 | P1 | All developers |
| app-guidelines | A4 | TRANSLATION.md | v1 | P1 | All developers, translators |
| app-guidelines | A5 | CRASH-HANDLING.md | v1 | P1 | All developers |
| app-guidelines | A6 | SETTINGS.md | v1 | P1 | App developers |
| app-guidelines | A7 | CLI.md | v1 | P1 | All developers |
| app-guidelines | A8 | IPC-USAGE.md | v1 | P2 | App developers |
| repo | R1 | LAYOUT.md | v1 | P0 | All maintainers |
| repo | R2 | NAMING.md | v1 | P0 | All maintainers |
| repo | R3 | CMAKE.md | v1 | P0 | All maintainers, release eng |
| repo | R4 | INSTALL.md | v1 | P0 | All maintainers, release eng |
| repo | R5 | HEADERS.md | v1 | P0 | All maintainers |
| repo | R6 | TESTS.md | v1 | P0 | QA team, all maintainers |
| coding | C1 | CPP.md | v1 | P0 | C++ developers |
| coding | C2 | QML.md | v1 | P1 | QML developers |
| coding | C3 | RUST.md | v1 | P4 | Rust developers |
| coding | C4 | GO.md | v1 | P4 | Go developers |
| packaging | PK1 | DEBIAN.md | v1 | P1 | Release engineers |
| packaging | PK2 | RPM.md | v1 | P1 | Release engineers |
| packaging | PK3 | PATHS.md | v1 | P1 | Release engineers, all devs |
| ci | CI1 | WORKFLOWS.md | v1 | P1 | CI engineers |
| ci | CI2 | TEST-REQUIREMENTS.md | v1 | P1 | QA team |
| ci | CI3 | COVERAGE.md | v1 | P3 | QA team |
| ci | CI4 | STATIC-ANALYSIS.md | v1 | P3 | All maintainers |
| ci | CI5 | RELEASE.md | v1 | P2 | Release engineers |
| docs | D1 | STANDARDS.md | v1 | P0 | All contributors |
| docs | D2 | ADR.md | v1 | P0 | Architects |
| docs | D3 | CHANGELOG.md | v1 | P0 | All maintainers |
| docs | D4 | RELEASE-NOTES.md | v1 | P2 | Release engineers |

---

## Specification Versioning

### Per-Document Version

Each specification document carries its own version:

```
Version: MAJOR.MINOR

MAJOR: A requirement in the specification has changed in a way that
       makes previously conforming implementations non-conforming.
       Example: "The timeout MUST be 5 seconds" → "The timeout MUST be 10 seconds"

MINOR: A new requirement has been added in a backward-compatible way,
       or a clarification has been made that does not change conformance.
       Example: Adding "The timeout SHOULD be documented in the help text."
```

### Specification System Version

The specification system as a whole has a version:

```
Specs vX.Y

X: Any document's MAJOR version has changed
Y: Any document's MINOR version has changed
```

This version is tracked in `specs/VERSION`.

### Repository Tagging

When a new version of the specification system is released:

```
git tag specs-v1.0
```

Each tag corresponds to a snapshot where all documents are consistent.

### Required Version Numbers

Every specification document MUST include in its header:

```markdown
**Version**: 1.0         # MAJOR.MINOR of this document
**Specs Version**: 1.0   # The specs/ VERSION file value at time of writing
**Status**: Draft        # Draft | Stable | Deprecated
```

### Cross-Document References

When a specification references another specification, it MUST reference
a specific version range:

```
Normative References:
- FS-LAYOUT.md >= 1.0 < 2.0  (filesystem layout requirements)
- API-DESIGN.md >= 2.0 < 3.0 (API design rules)
```

This prevents silent breakage when a referenced specification changes.

---

## Maintenance Guide

### Who Can Change a Specification

| Action | Required Approval |
|--------|------------------|
| Minor clarification (no normative change) | Spec editor + one team lead |
| Normative MINOR change (new requirements) | Spec editor + affected team leads |
| Normative MAJOR change (breaking) | All team leads + architecture team + RFC |
| Status change (Draft → Stable) | Architecture team approval |
| Status change (Stable → Deprecated) | Architecture team + replacement spec ready |
| Remove deprecated spec | Architecture team + 1 year after deprecation |

### Review Cadence

| Activity | Frequency |
|----------|-----------|
| All specs reviewed for consistency | Every desktop release (2x/year) |
| P0 specs reviewed | Quarterly |
| P1-P2 specs reviewed | Biannually |
| P3-P4 specs reviewed | Annually |
| Spec system version updated | When any document changes |

### Transitioning to a New Major Version

When a MAJOR version change is needed (breaking change):

1. Create a Draft of the new version in a separate branch
2. Document what changed and why in the version history
3. Set a deprecation date for the old version
4. Announce on the developer mailing list
5. Provide migration guidance
6. After the deprecation date, change the old spec to Deprecated status
7. After one more release cycle, remove the deprecated spec

### What Happens When Architecture Conflicts with Specs

The hierarchy of authority is:

```
ARCHITECTURE documents > Specification documents > Implementation code
```

If a conflict is found:

1. The specification document takes precedence over implementation code
   (implementation must change to match the spec)
2. If a specification contradicts the architecture document, the architecture
   document takes precedence (the specification must be revised)

---

## Index of Terms

See `GLOSSARY.md` for a complete list of Lingmo Desktop terminology.

Key terms used in this roadmap:

| Term | Meaning |
|------|---------|
| Specification | A normative document that defines requirements |
| Normative | Expresses a requirement (MUST/SHOULD/MAY) |
| Informative | Provides information, not a requirement |
| P0-P4 | Priority levels (Foundation → Future) |
| XS-XL | Effort estimates (Extra Small → Extra Large) |
| RFC | Request for Comments (design review process) |
| ADR | Architecture Decision Record |

---

> This roadmap is version 1.0 of the Lingmo Desktop specification system plan.
>
> All architectural decisions about the specification system should be
> recorded as ADRs in `docs/adr/`.
