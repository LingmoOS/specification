# Specification Authoring Standard

| | |
|---|---|
| **Document ID** | SPEC-CORE-002 |
| **Title** | Specification Authoring Standard |
| **Version** | 1.0.0 |
| **Status** | Stable |
| **Category** | Documentation |
| **Applies to** | All Lingmo Desktop specification documents |
| **Supersedes** | None |

---

## 1. Document Metadata

### 1.1 Required Header Table

Every specification MUST begin with a header table containing the following
fields in the order listed. The table MUST use Markdown's GFM table syntax.

[REQ-META-001] Every specification MUST contain a header table.

[REQ-META-002] The header table MUST contain exactly these rows, in order:

| Row | Content |
|-----|---------|
| **Document ID** | Identifier per Section 1.2 |
| **Title** | Full title of the specification |
| **Version** | Semantic version (MAJOR.MINOR.PATCH) |
| **Status** | One of: Draft, Review, Accepted, Stable, Deprecated |
| **Category** | Specification category per Section 1.3 |
| **Applies to** | Scope of applicability |
| **Supersedes** | Document IDs this spec replaces (or "None") |
| **Cross-ref** | Related specification IDs (OPTIONAL row) |

### 1.2 Document ID Format

[REQ-META-003] Document IDs MUST follow the format:

```
SPEC-<CATEGORY>-<NNN>
```

Where:
- `<CATEGORY>` is an uppercase category code from Section 1.3
- `<NNN>` is a zero-padded three-digit sequential number

[REQ-META-004] Document IDs MUST NOT be reused. If a specification is
superseded, the new specification receives a new ID. The superseded ID
remains retired.

### 1.3 Category Codes

[REQ-META-005] Every specification MUST be assigned exactly one category
code from the following table:

| Code | Specification Area | Examples |
|------|-------------------|----------|
| `REPO` | Repository Standards | Repository Layout |
| `CORE` | Core Infrastructure & Cross-cutting | Naming, Authoring Standard |
| `PLATFORM` | Platform Specifications | FS Layout, IPC, Session |
| `UI` | UI & Visual Design | Theme Tokens, Color System |
| `LINGMOUI` | LingmoUI Framework | Widget Spec, API Design |
| `PLUGIN` | Plugin System | Plugin Architecture, Security |
| `SDK` | Developer SDK | App SDK, Theme SDK |
| `APP` | Application Guidelines | Startup, Logging, CLI |
| `CODING` | Coding Standards | C++, QML, Rust, Go |
| `PACKAGING` | Packaging | Debian, RPM |
| `CI` | CI/CD | Workflows, Tests, Release |
| `DOCS` | Documentation | Standards, ADR |

### 1.4 Version Numbering

[REQ-META-006] Version numbers MUST follow semantic versioning:

```
MAJOR.MINOR.PATCH
```

[REQ-META-007] Version changes MUST follow these rules:

| Change Type | Increment | Examples |
|-------------|-----------|----------|
| Clarification, no normative change | PATCH | 1.0.0 → 1.0.1 |
| New normative requirements, backward compatible | MINOR | 1.0.0 → 1.1.0 |
| Breaking normative change (MUST removed or contradicted) | MAJOR | 1.0.0 → 2.0.0 |

### 1.5 Status Values

[REQ-META-008] A specification's status MUST be one of:

| Status | Meaning |
|--------|---------|
| **Draft** | Under development, subject to change, not yet authoritative |
| **Review** | Complete, submitted for peer review, may change |
| **Accepted** | Approved by the architecture team, ready for implementation |
| **Stable** | Released as part of the specification system, changes require RFC |
| **Deprecated** | Superseded, maintained for historical reference only |

[REQ-META-009] The status MUST appear in the header table and MUST match
the current lifecycle state (see Section 16).

### 1.6 Template

[REQ-META-010] Every specification MUST follow the template defined in
`TEMPLATE.md`. The template is normative and deviations require an
explicit rationale documented in the specification's introduction.

---

## 2. Abstract

[REQ-ABS-001] Every specification MUST include an abstract immediately
after the header table, separated by a horizontal rule (`---`).

[REQ-ABS-002] The abstract MUST be at most 200 words.

[REQ-ABS-003] The abstract MUST state:

1. What the specification defines
2. Who the specification is for
3. Why the specification exists (problem it solves)

[REQ-ABS-004] The abstract MUST NOT contain normative language (MUST,
SHOULD, MAY, etc.). Normative language belongs in the body.

[REQ-ABS-005] The abstract MUST NOT contain references to other
specifications. References belong in the Scope or References sections.

[REQ-ABS-006] The abstract SHOULD be written in a single paragraph.
If multiple paragraphs are needed, two is the maximum.

**Good abstract:**

> This specification defines the naming conventions used across all
> Lingmo Desktop repositories. It is intended for all developers
> contributing to the Lingmo ecosystem. A single unified naming
> convention reduces cognitive load, prevents inconsistencies across
> repositories, and ensures that automated tooling can rely on
> predictable identifiers.

**Bad abstract:**

> This document is about naming. Names are important. You should
> read this before you write any code. (Too vague, uses "should"
> incorrectly, no clear scope.)

---

## 3. Scope

[REQ-SCOPE-001] Every specification MUST contain a Scope section.

[REQ-SCOPE-002] The Scope section MUST contain three subsections:

- **Included**: What the specification covers
- **Excluded**: What the specification explicitly does NOT cover
- **Non-goals**: Outcomes that are intentionally out of scope

[REQ-SCOPE-003] Each subsection MUST be a bullet list or a short paragraph.
The "Excluded" and "Non-goals" subsections MUST be present even if empty
("None. This specification covers all aspects of ...").

[REQ-SCOPE-004] The Scope section MUST NOT contain requirements.
Requirements appear in the body sections.

**Example:**

> ### 3.1 Included
>
> - Source directory layout for all repositories
> - Build system organization (CMake)
> - Test directory structure
> - Packaging directory conventions
>
> ### 3.2 Excluded
>
> - Coding style (covered by SPEC-CODING-001)
> - API design (covered by SPEC-LINGMOUI-003)
>
> ### 3.3 Non-goals
>
> - Defining a project management methodology
> - Specifying release cadence

---

## 4. Terminology

[REQ-TERM-001] If a specification introduces terms that are not defined
in the project GLOSSARY, it MUST define them in a Terminology section.

[REQ-TERM-002] The Terminology section MUST appear before any section
that uses the defined terms.

[REQ-TERM-003] Terms defined in the specification MUST use the following
format:

```markdown
<term>
:   <definition>
```

Example:

```markdown
Applet
:   A plugin that provides a visual widget displayed on a panel or
    the desktop surface. Applets implement the `Lingmo::Applet`
    interface.
```

[REQ-TERM-004] A term that is defined in the project GLOSSARY MUST be
marked with an asterisk on first use: `plugin*`.

[REQ-TERM-005] A specification SHOULD NOT redefine a term that already
exists in the GLOSSARY. If a more specific definition is needed, the
specification MUST note the relationship: "This specification uses
`plugin` in a more restricted sense (see GLOSSARY for the general
definition)."

[REQ-TERM-006] Terms defined in the Terminology section MUST be
capitalized consistently throughout the specification. Once a term is
defined, the exact same capitalization MUST be used in every reference.

---

## 5. Requirements

### 5.1 Requirement Format

[REQ-REQ-001] Every normative requirement MUST be expressed as an
independent sentence containing at least one RFC 2119 keyword.

[REQ-REQ-002] The RFC 2119 keywords recognized in Lingmo specifications
are:

| Keyword | Meaning |
|---------|---------|
| MUST | Absolute requirement |
| MUST NOT | Absolute prohibition |
| REQUIRED | Synonym for MUST |
| SHOULD | Strong recommendation (exceptions exist) |
| SHOULD NOT | Strong recommendation against |
| RECOMMENDED | Synonym for SHOULD |
| MAY | Permitted, optional |
| OPTIONAL | Synonym for MAY |

[REQ-REQ-003] Keywords MUST be written in uppercase and MUST NOT be
escaped or formatted differently (no bold, no code).

[REQ-REQ-004] Each requirement MUST be separated from surrounding text
by blank lines.

[REQ-REQ-005] Requirements SHOULD be grouped by topic into subsections.
Each subsection SHOULD contain no more than 15 requirements.

### 5.2 Requirement Identification

[REQ-REQ-006] Every requirement MUST have a unique requirement ID.

[REQ-REQ-007] Requirement IDs MUST follow the format:

```
REQ-<PREFIX>-<NNN>
```

Where:
- `<PREFIX>` is a 3–6 character uppercase abbreviation for the section
  (e.g., `META`, `SCOPE`, `TERM`, `REQ`, `EX`, `STYLE`)
- `<NNN>` is a zero-padded three-digit sequential number within that prefix

[REQ-REQ-008] Requirement IDs MUST be enclosed in square brackets at the
beginning of the requirement:

```markdown
[REQ-REQ-001] Every normative requirement MUST be expressed as an
independent sentence containing at least one RFC 2119 keyword.
```

[REQ-REQ-009] Requirement IDs MUST NOT change once a specification
reaches Accepted status. When a requirement is removed, its ID is retired
with a note: `[REQ-XXX-NNN] — Removed in version X.Y.Z`.

### 5.3 Requirement Types

[REQ-REQ-010] Requirements are classified into three types:

| Type | Keyword | Convention |
|------|---------|-----------|
| Absolute | MUST / MUST NOT | No exceptions allowed |
| Conditional | SHOULD / SHOULD NOT | Exceptions require documented justification |
| Permissive | MAY | Optional, no justification needed |

[REQ-REQ-011] If a requirement applies only under certain conditions,
the condition MUST be stated first:

```markdown
[REQ-REQ-011] If a repository contains C++ code, it MUST have a
`.clang-format` file at the repository root.
```

### 5.4 Requirement Tables

[REQ-REQ-012] Related requirements MAY be grouped into tables when a
tabular format improves readability. Table-based requirements MUST
include a requirement label for each row.

```markdown
[REQ-REQ-012] The following requirements apply to header files:

| ID | Entity | Rule |
|----|--------|------|
| REQ-HEADER-001 | Public headers | MUST use `#pragma once` |
| REQ-HEADER-002 | Private headers | MUST use `_p.h` suffix |
| REQ-HEADER-003 | Internal headers | MUST reside in `detail/` directory |
```

### 5.5 Requirement Traceability

[REQ-REQ-013] Every requirement MUST be traceable to at least one
verification method.

[REQ-REQ-014] The verification method MUST be documented in the
Compliance section (Section 15) and MAY be noted inline:

```markdown
[REQ-REQ-014] Every repository MUST have a `CMakeLists.txt` file.
(*Verification*: CI check listing root directory contents.)
```

---

## 6. Examples

### 6.1 When to Include Examples

[REQ-EX-001] A specification SHOULD include examples for every
requirement that is not self-evident.

[REQ-EX-002] Examples MUST be clearly labeled as non-normative.
The label "Example" or "Counterexample" MUST appear at the start.

[REQ-EX-003] Examples MUST NOT use RFC 2119 keywords. If a requirement
appears inside an example, it MUST be in a comment or illustration
context.

### 6.2 Good and Bad Examples

[REQ-EX-004] Examples that illustrate correct usage MUST be labeled
with a checkmark:

```markdown
**Example 1 (✓): Correct repository name**

    lingmo-shell-core
```

[REQ-EX-005] Examples that illustrate incorrect usage MUST be labeled
with a cross:

```markdown
**Example 2 (✗): Incorrect repository name**

    lingmo_shell_core   # Uses underscores instead of hyphens
    LingmoShellCore     # Uses PascalCase instead of kebab-case
```

### 6.3 Counterexamples

[REQ-EX-006] Counterexamples (patterns that violate a requirement)
SHOULD be used to clarify common mistakes.

[REQ-EX-007] Every counterexample MUST reference the requirement it
violates:

```markdown
**Counterexample (violates REQ-NAMING-001):**

    My_App        # Violation: Missing "lingmo-" prefix
    lingmo-App    # Violation: Mixed case in component
```

### 6.4 Code Examples

[REQ-EX-008] Code examples MUST specify the language in a fenced code
block declaration:

````markdown
```cpp
Lingmo::Theme* theme = Lingmo::Theme::instance();
QColor accent = theme->accentColor();
```
````

[REQ-EX-009] Code examples SHOULD be syntactically correct and
compilable unless the purpose is to illustrate an error.

### 6.5 Diagrams

[REQ-EX-010] Diagrams MUST include a caption and alt text:

```markdown
![Session startup sequence](images/session-startup.png)
*Figure 1: Session startup sequence showing process initialization order*
```

[REQ-EX-011] Diagrams SHOULD be provided in a vector format (SVG).
Raster formats (PNG) are acceptable only when vector is impractical.

[REQ-EX-012] The source file for every diagram MUST be committed
alongside the rendered output, in a `docs/images/` directory within
the specification repository.

---

## 7. Rationale

[REQ-RAT-001] A specification SHOULD include a rationale for every
requirement that is likely to be questioned or that represents a
significant design decision.

[REQ-RAT-002] Rationale MUST be formatted as a `> ` blockquote:

```markdown
[REQ-RAT-002] All public headers MUST use `#pragma once`.
> *Rationale*: `#pragma once` is supported by all target compilers
> (GCC, Clang, MSVC). It eliminates the risk of include guard name
> collisions that would be difficult to debug across 47 repositories.
```

[REQ-RAT-003] Rationale that extends beyond a single paragraph SHOULD
be placed in a separate subsection titled `### Rationale` within the
relevant section.

[REQ-RAT-004] Rationale is informative, not normative. The requirement
stands independently of the rationale.

---

## 8. Compatibility

[REQ-COMPAT-001] If a specification deprecates or supersedes a previous
specification, it MUST include a Compatibility section.

[REQ-COMPAT-002] The Compatibility section MUST document:

- What changed (between versions or between superseded and current spec)
- Migration guidance for implementations
- Deprecation timeline (when old behavior is removed)
- How to detect incompatibilities

[REQ-COMPAT-003] The compatibility section MUST use paragraph labels:

```markdown
### 8.1 Backward Compatibility
This specification is backward compatible with version 1.0. Implementations
conforming to 1.0 automatically conform to 1.1.

### 8.2 Migration from 1.0
Implementations that use the `oldMethod()` API MUST replace it with
`newMethod()` before version 2.0 removes the old API.
```

---

## 9. Security

[REQ-SEC-001] A specification MUST include a Security section if any
requirement in the specification has security implications.

[REQ-SEC-002] Security implications include, but are not limited to:

- Authentication and authorization
- Input validation
- Privilege escalation
- Inter-process communication security
- Data confidentiality
- Denial of service

[REQ-SEC-003] The Security section MUST describe:

1. The security-relevant requirement (by ID)
2. The threat model (what is being protected and from whom)
3. The security mechanism (how the requirement addresses the threat)
4. Residual risks (what the requirement does NOT protect against)

[REQ-SEC-004] The Security section MUST appear after Compatibility and
before Performance.

---

## 10. Performance

[REQ-PERF-001] A specification MUST include a Performance section if any
requirement has implications for startup time, memory usage, CPU usage,
disk I/O, or network I/O.

[REQ-PERF-002] Performance requirements SHOULD be expressed as concrete
measurements rather than vague goals:

```markdown
[REQ-PERF-002] The plugin loading mechanism MUST complete discovery of
all installed plugins within 100 milliseconds on a system with 100
installed plugins.
```

Not:

```markdown
The plugin loading mechanism SHOULD be fast.   // Vague, untestable
```

---

## 11. Accessibility

[REQ-A11Y-001] A specification MUST include an Accessibility section if
any requirement affects the user interface or user interaction.

[REQ-A11Y-002] Accessibility requirements MUST reference WCAG 2.1
success criteria where applicable:

```markdown
[REQ-A11Y-002] The color contrast ratio between text and background
MUST be at least 4.5:1 for normal text (WCAG 2.1 AA, SC 1.4.3).
```

---

## 12. Internationalization

[REQ-I18N-001] A specification MUST include an Internationalization
section if any requirement involves:

- User-visible text
- Date, time, or number formatting
- Character encoding
- Text input (keyboard, IME)
- Bi-directional text
- Locale-dependent behavior

[REQ-I18N-002] The Internationalization section MUST specify the
locales that MUST be supported and the level of support required.

---

## 13. References

### 13.1 Reference Sections

[REQ-REF-001] Every specification MUST contain a References section
at the end of the document (before the Version History).

[REQ-REF-002] The References section MUST be divided into two subsections:

- **Normative References**: Documents that contain requirements upon
  which this specification depends
- **Informative References**: Documents that provide additional context
  but are not required for conformance

### 13.2 Reference Format

[REQ-REF-003] Every reference MUST include:

- Document title
- Document ID (if a Lingmo specification)
- Version or version constraint
- URL (optional, but RECOMMENDED for external references)

[REQ-REF-004] Normative references MUST specify a version constraint:

```markdown
- [SPEC-REPO-001]: Repository Layout Specification, version >= 1.0.0
```

[REQ-REF-005] When a normative reference has multiple versions, the
requirement applies to the newest version within the specified constraint.

### 13.3 Cross-Specification References

[REQ-REF-006] When referencing another specification's requirement,
the reference MUST include the document ID and requirement ID:

```markdown
As specified in [SPEC-REPO-001 §4.2], public headers MUST use
`#pragma once` (REQ-HEADER-001).
```

---

## 14. Appendix

[REQ-APP-001] A specification MAY include appendices for supplementary
material that is too detailed for the main body.

[REQ-APP-002] Appendices MUST be labeled alphabetically:

```
## Appendix A: Full Token Reference Table
## Appendix B: Migration Examples
```

[REQ-APP-003] Each appendix MUST be referenced from the main body.
Unreferenced appendices MUST NOT exist.

[REQ-APP-004] Appendices MAY include:

- Reference tables (e.g., all color token values)
- Historical notes (e.g., why a particular design was chosen)
- Extended examples
- Checklists (see Section 19)

[REQ-APP-005] Appendices MUST NOT contain normative requirements.
All normative requirements MUST appear in the main body.

---

## 15. Compliance

### 15.1 Compliance Statement

[REQ-CONFORM-001] Every specification MUST include a Compliance section
that describes how an implementation proves conformance.

[REQ-CONFORM-002] The Compliance section MUST specify, for each
requirement or group of requirements:

- Verification method (automated check, code review, manual inspection)
- Verification tool (if automated)
- Pass/fail criteria

### 15.2 Conformance Levels

[REQ-CONFORM-003] A specification SHOULD define conformance levels:

| Level | Name | Meaning |
|-------|------|---------|
| 1 | Full Compliance | All MUST and SHOULD requirements satisfied |
| 2 | Substantial Compliance | All MUST requirements satisfied; documented exceptions for SHOULD |
| 3 | Partial Compliance | All critical MUST requirements satisfied; non-critical MUST and SHOULD have documented exceptions |

### 15.3 Compliance Checklist

[REQ-CONFORM-004] Every specification MUST include a compliance checklist
as its final appendix. The checklist MUST enumerate every requirement
in the specification in a format suitable for manual or automated
verification.

[REQ-CONFORM-005] Each checklist item MUST include:

- Requirement ID
- Brief description
- Verification method
- Pass/fail indicator

---

## 16. Specification Lifecycle

### 16.1 States

[REQ-LIFE-001] Every specification MUST pass through the following
states in order:

```
Draft → Review → Accepted → Stable → (Deprecated → Obsolete)
```

[REQ-LIFE-002] A specification MAY transition from Review back to Draft
if major changes are requested during review.

[REQ-LIFE-003] A specification MAY transition from Stable to Draft if
a major revision is initiated. The previous Stable version remains
available for reference.

[REQ-LIFE-004] A specification MUST NOT skip states. Transitions are:

| From | To | Trigger |
|------|----|---------|
| Draft | Review | Author submits for review |
| Review | Accepted | Review board approves |
| Review | Draft | Major changes requested |
| Accepted | Stable | Specification is released |
| Stable | Draft | Major revision initiated |
| Stable | Deprecated | Superseded by newer spec |
| Deprecated | Obsolete | 2 years after deprecation |

### 16.2 State Requirements

[REQ-LIFE-005] A specification in Draft state MUST include a notice at
the top:

```markdown
> **Status**: Draft — This specification is under development.
> Requirements may change without notice. Do not implement against
> this version.
```

[REQ-LIFE-006] A specification in Deprecated state MUST include a notice
at the top:

```markdown
> **Status**: Deprecated — This specification is superseded by
> [SPEC-XXXX-NNN]. It is maintained for historical reference only.
> New implementations MUST follow [SPEC-XXXX-NNN].
```

---

## 17. Change Control

### 17.1 Change Types

[REQ-CHANGE-001] Changes to specifications are classified as:

| Class | Definition | Version Impact | Approval |
|-------|------------|---------------|----------|
| Editorial | Spelling, grammar, formatting | PATCH | Single editor |
| Clarification | Non-normative explanation | PATCH | Single editor |
| Addition | New normative requirement, backward compatible | MINOR | Review board |
| Revision | Existing requirement modified, backward compatible | MINOR | Review board |
| Breaking | MUST requirement removed or contradicted | MAJOR | Architecture team + RFC |

### 17.2 Change Process

[REQ-CHANGE-002] Editorial and clarification changes MAY be submitted
as direct pull requests without prior discussion.

[REQ-CHANGE-003] Addition and revision changes MUST be submitted as
pull requests and MUST receive approval from at least two members of
the specification review board.

[REQ-CHANGE-004] Breaking changes MUST follow the RFC process defined
in `docs/ADR.md`. The RFC MUST:

1. Explain why the breaking change is necessary
2. Document the migration path
3. Provide a timeline (minimum 6 months from announcement to removal)

### 17.3 Version History

[REQ-CHANGE-005] Every specification MUST include a Version History
table at the end of the document, before the References.

[REQ-CHANGE-006] The Version History table MUST include:

| Version | Date | Author | Changes |
|---------|------|--------|---------|

[REQ-CHANGE-007] Every change to the specification MUST be recorded
in the Version History, regardless of change class.

---

## 18. Writing Style

### 18.1 General Rules

[REQ-STYLE-001] Sentences MUST NOT exceed 40 words.

[REQ-STYLE-002] Paragraphs MUST NOT exceed 7 sentences.

[REQ-STYLE-003] Section depth MUST NOT exceed 4 levels:

```markdown
# Level 1 (Document title)
## Level 2 (Major section)
### Level 3 (Subsection)
#### Level 4 (Sub-subsection)
```

[REQ-STYLE-004] The document title MUST be an H1 (`#`). Every
specification MUST have exactly one H1.

[REQ-STYLE-005] Major sections MUST be H2 (`##`). Subsection MUST be
H3 (`###`). Sub-subsections MUST be H4 (`####`).

[REQ-STYLE-006] Lists MUST NOT exceed 12 items. Lists with more than
12 items MUST be converted to a table.

[REQ-STYLE-007] Tables MUST have a header row. Tables MUST use the
GFM pipe syntax.

### 18.2 Voice and Tense

[REQ-STYLE-008] Specifications MUST use active voice:

```markdown
The implementation MUST reject invalid input.  // CORRECT
Invalid input MUST be rejected.                // AVOID (passive)
```

[REQ-STYLE-009] Specifications MUST use present tense:

```markdown
The function returns the current time.         // CORRECT
The function will return the current time.     // INCORRECT (future)
```

### 18.3 Terminology Rules

[REQ-STYLE-010] The following words and phrases MUST NOT appear in
specifications:

```
obviously, clearly, simply, just, easy, trivial, basically,
essentially, actually, of course, note that, it should be noted
```

Rationale: These words add no information and imply judgment.

[REQ-STYLE-011] The following abbreviations MUST be expanded on first
use in each specification:

| Abbreviation | Expansion |
|-------------|-----------|
| e.g. | for example |
| i.e. | that is |
| etc. | and similar |
| vs. | versus |
| w.r.t. | with respect to |

[REQ-STYLE-012] Contractions MUST NOT be used:

```markdown
do not     // CORRECT
don't      // INCORRECT

cannot     // CORRECT (single word)
can't      // INCORRECT

it is      // CORRECT
it's       // INCORRECT (also ambiguous with possessive)
```

### 18.4 Capitalization

[REQ-STYLE-013] RFC 2119 keywords (MUST, SHOULD, MAY, etc.) MUST be
fully uppercase.

[REQ-STYLE-014] Defined terms from the Terminology section MUST be
capitalized consistently throughout the document.

[REQ-STYLE-015] Section titles MUST use sentence case:

```markdown
## 5. Repository naming rules       // CORRECT
## 5. Repository Naming Rules       // INCORRECT (title case)
```

[REQ-STYLE-016] Table headers MUST use sentence case:

```markdown
| Requirement ID | Rule                              |
|--------------|-----------------------------------|
| REQ-NAME-001 | Repository names MUST be lowercase |
```

### 18.5 Lists

[REQ-STYLE-017] Bullet lists MUST use `-` (hyphen) as the marker.

[REQ-STYLE-018] Ordered lists MUST use `1.` (Arabic numeral followed
by period).

[REQ-STYLE-019] List items MUST be complete sentences or valid sentence
fragments. All items in a list MUST use the same grammatical structure.

### 18.6 Cross-References

[REQ-STYLE-020] Cross-references MUST use the section symbol (`§`):

```markdown
As documented in §5.2, requirement IDs MUST follow the REQ- format.
See also [SPEC-REPO-001 §3].
```

[REQ-STYLE-021] Cross-references to specific requirements MUST use the
requirement ID:

```markdown
The naming rules (REQ-NAMING-001 through REQ-NAMING-012) apply to
all repositories.
```

### 18.7 Code Formatting

[REQ-STYLE-022] Inline code MUST use backticks:

```markdown
Use the `Lingmo::Theme::instance()` method.
```

[REQ-STYLE-023] Code blocks MUST use fenced code blocks with a language
declaration:

````markdown
```cpp
#include <Lingmo/Theme.h>
```
````

### 18.8 Notes and Warnings

[REQ-STYLE-024] Informative notes MUST use the following format:

```markdown
> *Note*: This section does not apply to header-only libraries.
```

[REQ-STYLE-025] Warnings MUST use the following format:

```markdown
> **Warning**: Violating this requirement may cause undefined behavior
> in the window manager.
```

### 18.9 Document Structure

[REQ-STYLE-026] Every specification MUST follow this section order:

```
1. Document Metadata (header table + abstract)
2. Scope
3. Terminology (if needed)
4. [Specification body sections]

   The body sections MUST appear in a logical order determined by
   the spec author. Within the body, requirements are grouped by topic.

5. Compatibility (if needed)
6. Security (if needed)
7. Performance (if needed)
8. Accessibility (if needed)
9. Internationalization (if needed)
10. References
11. Appendix A: Compliance Checklist

    The checklist MUST be the final appendix.

12. Version History
```

---

## 19. Compliance Checklist

Every specification MUST pass the following checklist before reaching
Accepted status.

### 19.1 Structure

- [ ] Document ID follows `SPEC-<CATEGORY>-<NNN>` format
- [ ] Header table contains all required fields (REQ-META-001)
- [ ] Status value is valid (REQ-META-008)
- [ ] Abstract is present and ≤ 200 words (REQ-ABS-001, REQ-ABS-002)
- [ ] Template from `TEMPLATE.md` is followed (REQ-META-010)

### 19.2 Scope

- [ ] Scope section exists (REQ-SCOPE-001)
- [ ] Included, Excluded, and Non-goals subsections exist (REQ-SCOPE-002)

### 19.3 Terminology

- [ ] All undefined terms are defined (REQ-TERM-001)
- [ ] GLOSSARY terms are marked with asterisk (REQ-TERM-004)
- [ ] GLOSSARY terms are not redefined (REQ-TERM-005)
- [ ] Terms are capitalized consistently (REQ-TERM-006)

### 19.4 Requirements

- [ ] Every requirement uses at least one RFC 2119 keyword (REQ-REQ-001)
- [ ] Keywords are uppercase (REQ-REQ-003)
- [ ] Every requirement has a unique ID (REQ-REQ-006)
- [ ] Requirement IDs follow `REQ-<PREFIX>-<NNN>` format (REQ-REQ-007)
- [ ] Requirement IDs are enclosed in square brackets (REQ-REQ-008)

### 19.5 Examples

- [ ] Non-obvious requirements have examples (REQ-EX-001)
- [ ] Examples are labeled as non-normative (REQ-EX-002)
- [ ] Code examples specify language (REQ-EX-008)
- [ ] Diagrams have captions and alt text (REQ-EX-010)

### 19.6 Rationale

- [ ] Requirements likely to be questioned have rationale (REQ-RAT-001)
- [ ] Rationale is formatted as blockquote (REQ-RAT-002)

### 19.7 Security, Performance, Accessibility, Internationalization

- [ ] Security section exists if requirements have security implications
- [ ] Performance section exists if requirements have performance implications
- [ ] Accessibility section exists if requirements affect UI
- [ ] Internationalization section exists if requirements involve locale

### 19.8 References

- [ ] Normative references have version constraints (REQ-REF-004)
- [ ] References section is at the end, before Version History (REQ-REF-001)

### 19.9 Compliance

- [ ] Compliance section exists (REQ-CONFORM-001)
- [ ] Compliance checklist exists as the final appendix (REQ-CONFORM-004)

### 19.10 Style

- [ ] No sentence exceeds 40 words (REQ-STYLE-001)
- [ ] No paragraph exceeds 7 sentences (REQ-STYLE-002)
- [ ] Section depth does not exceed 4 levels (REQ-STYLE-003)
- [ ] Active voice is used (REQ-STYLE-008)
- [ ] Present tense is used (REQ-STYLE-009)
- [ ] Forbidden words are absent (REQ-STYLE-010)
- [ ] Abbreviations are expanded on first use (REQ-STYLE-011)
- [ ] Contractions are absent (REQ-STYLE-012)

### 19.11 Lifecycle

- [ ] Status matches the document's lifecycle state (REQ-META-009)
- [ ] Required status notice is present for Draft/Deprecated (REQ-LIFE-005, REQ-LIFE-006)
- [ ] Version History table exists (REQ-CHANGE-005)
- [ ] Version History has all required columns (REQ-CHANGE-006)

---

## Appendix A: Specification Template Reference

The following skeleton shows every required section in order.
Bold sections are REQUIRED. Italic sections are CONDITIONAL.

```markdown
# Title

| | |
|---|---|
| **Document ID** | SPEC-XXXX-NNN |
| **Title** | Full Title |
| **Version** | 1.0.0 |
| **Status** | Draft |
| **Category** | <category> |
| **Applies to** | <scope> |
| **Supersedes** | None |

---

*Abstract*

---

## 1. Scope

### 1.1 Included

### 1.2 Excluded

### 1.3 Non-goals

## 2. Terminology (if needed)

## 3–N. Body Sections

*Requirements grouped by topic, using REQ-XXX-NNN identifiers*

## N+1. Compatibility (if needed)

## N+2. Security (if needed)

## N+3. Performance (if needed)

## N+4. Accessibility (if needed)

## N+5. Internationalization (if needed)

## N+6. References

### Normative References

### Informative References

## N+7. Appendix A: Compliance Checklist

---

## Version History
```

---

## Appendix B: Example Requirement IDs by Section

The following PREFIX values are RECOMMENDED for common section types.
Authors MAY define new prefixes as needed, but they MUST be documented
in the Terminology section.

| Section Type | Recommended Prefix |
|-------------|-------------------|
| Document Metadata | `META` |
| Scope | `SCOPE` |
| Terminology | `TERM` |
| Requirements (general) | `REQ` |
| Naming | `NAME` |
| Layout | `LAYOUT` |
| Build System | `BUILD` |
| Tests | `TEST` |
| Examples | `EX` |
| Rationale | `RAT` |
| Compatibility | `COMPAT` |
| Security | `SEC` |
| Performance | `PERF` |
| Accessibility | `A11Y` |
| Internationalization | `I18N` |
| References | `REF` |
| Appendix | `APP` |
| Compliance | `CONFORM` |
| Lifecycle | `LIFE` |
| Change Control | `CHANGE` |
| Writing Style | `STYLE` |
| Checklist | `CHECK` |

---

## Version History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0.0 | 2026-07-08 | Lingmo Specification Team | Initial specification |

---

## References

### Normative References

- [RFC 2119]: Key words for use in RFCs to Indicate Requirement Levels
- [SPEC-CORE-001]: Naming Convention Specification, version >= 1.0.0

### Informative References

- [SPEC-REPO-001]: Repository Layout Specification
- [SPEC-ROADMAP]: Specification Roadmap
- [ISO/IEC Directives Part 2]: Principles and rules for the structure and drafting of ISO and IEC documents
