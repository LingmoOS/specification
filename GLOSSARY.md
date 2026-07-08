# Lingmo Desktop — Glossary of Terms

| | |
|---|---|
| **Document ID** | SPEC-CORE-003 |
| **Title** | Lingmo Desktop Glossary |
| **Version** | 1.0.0 |
| **Status** | Stable |
| **Category** | Core Infrastructure & Cross-cutting |
| **Applies to** | All Lingmo Desktop specification documents, source code, and documentation |
| **Supersedes** | None |
| **Cross-ref** | SPEC-CORE-002 (Authoring Standard) |

---

## 1. Purpose

This document defines the canonical terminology for the Lingmo Desktop project. All specification documents, source code comments, commit messages, and developer communication MUST use these terms with the meanings defined here. A single shared vocabulary prevents ambiguity across 47 repositories and ensures that requirements are interpreted consistently.

---

## 2. Scope

### 2.1 Included

- All technical terms used across Lingmo Desktop specifications
- Acronyms and abbreviations with their expanded forms
- Cross-references to specification documents that define terms in greater detail
- Terms specific to the Lingmo Desktop architecture, build system, runtime, and ecosystem

### 2.2 Excluded

- General software engineering terms that are not Lingmo-specific (e.g., "variable", "function", "class")
- Terms defined exclusively within a single specification document (those belong in that document's Terminology section)
- Brand names and marketing terminology

### 2.3 Non-goals

- Providing a tutorial or introduction to desktop environment concepts
- Defining implementation-specific internal names that are not part of the public specification surface

---

## 3. Terminology Conventions

Terms in this glossary use the following format:

```markdown
<term>
:   <definition>

    > *Cross-ref*: <related specification or document>
    > *See also*: <related term>
```

On first use in any specification document, a term defined here MUST be marked with an asterisk: `session*`.

---

## 4. Glossary of Terms

### A

Abstract
:   A concise summary at the beginning of a specification document that states what the specification defines, who it is for, and why it exists.

    > *Cross-ref*: SPEC-CORE-002 §2 (Abstract requirements)

ADR (Architecture Decision Record)
:   A document that records a significant architectural decision, including its context, the alternatives considered, the chosen solution, and the consequences. ADRs are stored in `docs/adr/` within each repository.

    > *Cross-ref*: SPEC-DOCS-002 (ADR process)

Applet
:   A plugin that provides a visual widget displayed on a panel, the desktop surface, or the overview. Applets implement the `Lingmo::Applet` interface and are loaded at runtime by shell surface processes. Examples: application launcher, task manager, system tray, clock.

    > *Cross-ref*: SPEC-PLUGIN-001 (Plugin System)

Architecture Document
:   The highest-level technical description of the Lingmo Desktop system. Architecture documents take precedence over specification documents in case of conflict.

    > *Cross-ref*: ARCHITECTURE-v3.md

Autostart
:   The mechanism by which applications are launched automatically when a user session begins. Lingmo Desktop supports both the XDG Autostart specification (`~/.config/autostart/`) and its own autostart directory.

### C

CMake Target
:   A named build artifact defined in CMake (executable, shared library, static library, or custom target). Targets are the unit of dependency and installation in the Lingmo build system.

    > *Cross-ref*: SPEC-REPO-003 (CMake organization)

Component
:   One of the 47 implementation repositories in the Lingmo Desktop project. Each component has a name following the `lingmo-<component>` pattern and produces one or more build artifacts.

    > *Cross-ref*: SPEC-REPO-001 (Repository Layout), SPEC-CORE-001 (Naming Convention)

Compliance Checklist
:   An appendix in every specification document that enumerates every requirement with its verification method, used for conformance testing.

    > *Cross-ref*: SPEC-CORE-002 §15.3

Conformance Level
:   A tier indicating how fully an implementation satisfies a specification's requirements. Levels include Full Compliance, Substantial Compliance, and Partial Compliance.

    > *Cross-ref*: SPEC-CORE-002 §15.2

### D

D-Bus
:   The inter-process communication (IPC) system used by Lingmo Desktop for communication between processes. Both the session bus (for desktop services) and system bus (for system services) are used. Lingmo Desktop defines its own D-Bus interfaces under the `org.lingmo.*` namespace.

    > *Cross-ref*: SPEC-PLATFORM-005 (IPC Specification)

Daemon
:   A background process that provides a system service. Daemons run for the duration of the user session, have no visible UI, and communicate via D-Bus. Examples: settings daemon, power manager, display manager.

    > *Cross-ref*: SPEC-PLATFORM-006 (Service Specification)

Display Manager
:   The login screen process that authenticates users and starts the desktop session. Lingmo Desktop uses SDDM (Simple Desktop Display Manager) as its display manager.

    > *Cross-ref*: SPEC-PLATFORM-004 (Session Specification)

### I

IPC (Inter-Process Communication)
:   The mechanism by which Lingmo processes exchange data and control signals. The primary IPC mechanism is D-Bus. Shell core additionally uses Unix domain sockets for communication with shell surface processes.

    > *Cross-ref*: SPEC-PLATFORM-005 (IPC Specification)

### K

KWin
:   The Wayland compositor used by Lingmo Desktop. Lingmo Desktop does NOT fork or modify KWin source code. All integration is through documented plugin interfaces: decoration plugins, scripting API, effect plugins, KWayland protocols, and D-Bus API.

    > *Cross-ref*: ARCHITECTURE-v3.md §1 (Layer 1)

### L

Layer
:   A structural level in the Lingmo Desktop architecture. Layers define dependency direction: each layer may depend only on the same or lower layers. The layers are: System Layer (0), KWin (1), Qt 6 Frameworks (2), Lingmo Core Libraries (3), Lingmo Services & Shell (4), Lingmo Applications (5).

    > *Cross-ref*: ARCHITECTURE-v3.md §1

Lingmo Core Libraries
:   The set of foundational shared libraries (Layer 3) that all Lingmo processes depend on: `lingmo-theme`, `lingmo-config`, `lingmo-plugin`, `lingmo-ipc`, `lingmo-utils`, `lingmo-style`, `lingmo-controls`, `lingmo-effects`, `lingmo-icon`, `lingmo-window`.

    > *Cross-ref*: ARCHITECTURE-v3.md §1 (Layer 3)

Lingmo Desktop
:   The complete desktop environment project. Encompasses all 47 implementation repositories, the specification system, and the integrated user experience.

### M

Module
:   A logical unit of functionality within a component. In C++ context, a module corresponds to a subdirectory under `src/` that produces a separate build target. In QML context, a module corresponds to a set of QML types exposed under a single import URI (e.g., `LingmoUI.Controls`).

### N

Normative
:   Expressing a requirement using RFC 2119 keywords (MUST, SHOULD, MAY, etc.). Normative content is authoritative and binding on implementations.

Informative
:   Providing explanation, context, or examples without expressing requirements. Informative content is not binding.

    > *Cross-ref*: SPEC-CORE-002 §5 (Requirements)

### O

OSD (On-Screen Display)
:   A transient overlay showing feedback for user actions such as volume changes, brightness adjustments, media playback state, or keyboard toggle state. The OSD is implemented as a D-Bus-activated process that appears for at most 10 seconds then exits.

    > *Cross-ref*: ARCHITECTURE-v3.md §2.5

### P

Panel
:   A shell surface that provides a bar (typically at the top or bottom of the screen) containing applets such as the application launcher, system tray, clock, and virtual desktop switcher. One panel instance runs per monitor.

    > *Cross-ref*: ARCHITECTURE-v3.md §2.3

Plugin
:   A dynamically loaded shared library that extends Lingmo Desktop functionality at runtime. Plugin types include applets, KWin decorations, KWin scripts, KWin effects, and theme engines. All plugins use the `lingmo-plugin` library for loading and lifecycle management.

    > *Cross-ref*: SPEC-PLUGIN-001 (Plugin System), SPEC-PLUGIN-002 (Plugin Security)

Portal
:   A D-Bus service that provides sandboxed applications with controlled access to desktop resources (file chooser, screen capture, wallpaper, etc.). Lingmo Desktop implements the freedesktop.org Portal API.

Protocol
:   A set of rules governing communication between two processes. Lingmo Desktop uses D-Bus protocols (standardized interface definitions), Wayland protocols (wlr-layer-shell, xdg-shell, ext-session-lock, etc.), and a custom socket-based protocol between shell core and shell surfaces.

### Q

QML Module
:   A set of QML types and JavaScript resources packaged for import by QML applications. Lingmo Desktop provides QML modules under the `LingmoUI.*` namespace, installed to `${QMLDIR}/LingmoUI/`.

    > *Cross-ref*: SPEC-CORE-001 (Naming Convention)

QStyle / LingmoStyle
:   The Qt Widgets styling plugin that implements the visual appearance of standard Qt widgets according to Lingmo Desktop's design language. LingmoStyle is installed as a Qt style plugin and selected at application startup.

    > *Cross-ref*: SPEC-LINGMOUI-003 (QStyle Specification)

### R

RFC (Request for Comments)
:   A formal review process for proposing significant changes to specifications or architecture. RFCs follow a defined template and require approval from specified stakeholders.

    > *Cross-ref*: SPEC-DOCS-002 (ADR process)

Requirement ID
:   A unique identifier for a normative requirement, formatted as `REQ-<PREFIX>-<NNN>`. Requirement IDs are immutable once a specification reaches Accepted status.

    > *Cross-ref*: SPEC-CORE-002 §5.2

### S

SDDM (Simple Desktop Display Manager)
:   The display manager used by Lingmo Desktop. It provides the login screen, authenticates users, and starts the user session via systemd.

sd_notify
:   A systemd protocol by which a service signals its readiness state (`READY=1`) or status changes to the service manager. Lingmo Desktop uses `Type=notify` for critical services that must report readiness before dependent services start.

    > *Cross-ref*: SPEC-PLATFORM-003 (Process Lifecycle)

Session / User Session
:   The lifetime of a user's logged-in desktop environment, from login (started by SDDM) to logout. The session encompasses all user processes, services, and shell components.

    > *Cross-ref*: SPEC-PLATFORM-004 (Session Specification)

Session Manager (lingmo-session)
:   The first process started by the display manager. The session manager coordinates the startup sequence, monitors component health, orchestrates shutdown, and manages crash recovery. It is PID 1 of the user session slice.

    > *Cross-ref*: ARCHITECTURE-v3.md §2.1

Shell Core (lingmo-shell-core)
:   The central process that owns window state, virtual desktop state, applet configuration, and KWin interaction. It is the only process that communicates directly with KWin via D-Bus. All shell surface processes route KWin-related operations through the shell core.

    > *Cross-ref*: ARCHITECTURE-v3.md §2.2

Shell Surface
:   A process that provides visual UI elements: panel, desktop, overview. Each shell surface is a separate process that connects to the shell core via a Unix socket. Shell surfaces load applets as plugins.

    > *Cross-ref*: ARCHITECTURE-v3.md §2.3

Specification / Spec
:   A normative document that defines requirements for some aspect of the Lingmo Desktop project. Every specification follows the template defined in SPEC-CORE-002 and lives in the `specs/` repository.

    > *Cross-ref*: SPEC-CORE-002 (Authoring Standard), SPEC-ROADMAP.md

systemd Unit
:   A systemd service file that defines how a Lingmo process is managed: its type, startup ordering, restart policy, resource limits, and dependencies. Lingmo Desktop uses user-scope systemd units (`systemctl --user`).

### T

Theme
:   A collection of design tokens, assets, and configuration files that define the visual appearance of Lingmo Desktop. Themes include color palettes, typography settings, icon sets, cursor themes, and sound schemes. Themes follow a defined JSON schema and support inheritance and fallback.

    > *Cross-ref*: SPEC-UI-002 (Theme Specification)

Theme Token
:   A named design value (color, spacing, radius, shadow, animation parameter, etc.) that is used consistently across all UI components. Tokens are the atomic units of the theme system.

    > *Cross-ref*: SPEC-UI-003 (Theme Tokens)

### W

Wayland
:   The display protocol used by Lingmo Desktop. KWin acts as the Wayland compositor. Lingmo Desktop uses standard Wayland protocols (xdg-shell, wlr-layer-shell, ext-session-lock, etc.) and does not define custom Wayland protocols except where absolutely necessary.

    > *Cross-ref*: SPEC-PLATFORM-007 (Wayland Protocols)

### X

XDG
:   Cross-Desktop Group (freedesktop.org) standards that Lingmo Desktop follows. Includes XDG Base Directory Specification (`XDG_CONFIG_HOME`, `XDG_DATA_HOME`, `XDG_CACHE_HOME`), XDG Autostart, XDG Desktop Entry Specification, and others.

---

## 5. Acronyms and Abbreviations

The following acronyms and abbreviations are approved for use in Lingmo Desktop specifications and source code. Any abbreviation not listed here MUST be written in full on first use.

| Abbreviation | Full Form | Usage Notes |
|---|---|---|
| ADR | Architecture Decision Record | |
| API | Application Programming Interface | |
| ABI | Application Binary Interface | |
| CI | Continuous Integration | |
| D-Bus | Desktop Bus | Always hyphenated and capitalized |
| IME | Input Method Engine | |
| IPC | Inter-Process Communication | |
| JSON | JavaScript Object Notation | |
| KWin | KWin Window Manager | Proper name, not an acronym |
| OSD | On-Screen Display | |
| PID | Process Identifier | |
| PII | Personally Identifiable Information | |
| QML | Qt Modeling Language | |
| RFC | Request for Comments | |
| SDK | Software Development Kit | |
| SDDM | Simple Desktop Display Manager | |
| SVG | Scalable Vector Graphics | |
| UI | User Interface | |
| URI | Uniform Resource Identifier | |
| URL | Uniform Resource Locator | |
| WCAG | Web Content Accessibility Guidelines | |
| XDG | Cross-Desktop Group | Originally X Desktop Group |
| XML | Extensible Markup Language | |

---

## 6. Compliance

A specification document conforms to this glossary if:

| ID | Requirement | Verification |
|---|---|---|
| REQ-GLOSSARY-001 | Every term marked with an asterisk on first use in any specification MUST be defined in this glossary. | Manual review during spec review |
| REQ-GLOSSARY-002 | No specification MAY redefine a term that is already defined in this glossary. If a more specific definition is needed, the specification MUST note the relationship. | Manual review |
| REQ-GLOSSARY-003 | The exact spelling, capitalization, and hyphenation defined here MUST be used throughout all specifications. | Case-sensitive text search |
| REQ-GLOSSARY-004 | Approved abbreviations from Section 5 MUST be used in preference to any alternative abbreviation. | Manual review |
| REQ-GLOSSARY-005 | An abbreviation not listed in Section 5 MUST be written in full on first use in every specification document, followed by the abbreviation in parentheses. | Automated check during build |
| REQ-GLOSSARY-006 | This glossary MUST be kept in sync with the architecture documents. When a new component, service, or concept is added to the architecture, a corresponding glossary entry MUST be added. | Review before each spec release |

---

## 7. Compliance Checklist

| Requirement ID | Description | Verification Method | Pass/Fail |
|---|---|---|---|
| REQ-GLOSSARY-001 | Asterisk-marked terms defined in glossary | Manual review | |
| REQ-GLOSSARY-002 | No redefinition of glossary terms | Manual review | |
| REQ-GLOSSARY-003 | Consistent spelling and capitalization | Automated text search | |
| REQ-GLOSSARY-004 | Approved abbreviations used | Manual review | |
| REQ-GLOSSARY-005 | Unlisted abbreviations expanded | Automated check | |
| REQ-GLOSSARY-006 | Glossary synced with architecture | Pre-release audit | |

---

## 8. Version History

| Version | Date | Changes |
|---|---|---|
| 1.0.0 | 2026-07-08 | Initial glossary |

---

> *Cross-ref*: SPEC-CORE-002 (Authoring Standard), SPEC-CORE-001 (Naming Convention), SPEC-ROADMAP.md
