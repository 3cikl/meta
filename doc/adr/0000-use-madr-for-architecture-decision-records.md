---
status: accepted
date: 2026-04-15
decision-makers: Aleksandar Buza
consulted: null
informed: null
---

# Use MADR for Architecture Decision Records

## Context and Problem Statement

The project requires a standardized format for capturing architectural decisions in a way that is lightweight, version-controllable, and accessible to all contributors. Without a consistent format, decisions are scattered across wikis, emails, and pull request discussions, making it difficult to reconstruct the reasoning behind design choices over time.

Which format should be adopted for recording Architecture Decision Records (ADRs) in this repository?

## Decision Drivers

* Decisions must be stored alongside the codebase in version control (Git)
* The format must have a low barrier to entry; contributors should be able to author ADRs without learning a heavyweight methodology
* The format must be human-readable in plain text (Markdown)
* The format should support structured reasoning: context, options considered, and chosen outcome with justification
* Tooling support (linters, generators) is desirable but not required
* The format should scale from small teams to larger organisations without structural changes

## Considered Options

* MADR (Markdown Architectural Decision Records)
* Nygard-style ADR (original format by Michael Nygard)
* RFC (Request for Comments)
* Arc42 Decision Section
* No formal ADR process

## Decision Outcome

Chosen option: **MADR**, because it best satisfies all decision drivers, particularly the requirement for structured multi-option analysis, which is absent from the Nygard format and not mandated by the other options. It is plain Markdown, version-control-native, and has an active community with tooling (adr-tools, Log4brains). Its template is the basis for this project's ADR index (`doc/adr/_index.md`).

### Consequences

* Good, because all architectural decisions are co-located with the code they describe and travel with every branch and tag
* Good, because the structured template (Context -> Options -> Outcome -> Pros/Cons) enforces consistent quality across authors
* Good, because MADR is tooling-agnostic: plain Markdown renders correctly in GitHub, GitLab, and any static site generator
* Good, because optional sections (Decision Drivers, Pros/Cons, Confirmation) allow low-ceremony decisions to stay concise while complex ones can be fully elaborated
* Neutral, because contributors unfamiliar with ADRs will need a brief onboarding to the convention (mitigated by the `_index.md` template)
* Neutral, because this ADR pins the project to MADR 4.0.0; adopting a future version of the spec requires a superseding ADR
* Bad, because MADR does not enforce any workflow (approval gates, status transitions); process discipline must come from team convention or CI tooling
* Bad, because maintaining ADRs adds process overhead (authoring, reviewing, indexing) that may not be justified until the project has multiple active contributors
* Bad, because ADRs can become stale if there is no process to periodically review and update their statuses

### Confirmation

* Compliance is verified during pull request review: the reviewer checks that new ADRs follow the `doc/adr/_index.md` template, are placed in `doc/adr/NNNN-<slug>.md`, and that the index in `doc/adr/README.md` is updated
* ADR status (`proposed -> accepted -> deprecated / superseded`) must be kept current via PR review

## Pros and Cons of the Options

### MADR (Markdown Architectural Decision Records)

* Good, because it is pure Markdown; zero tooling required to read or write
* Good, because it is version-control-native and works with standard Git workflows (PRs, blame, history)
* Good, because structured multi-option comparison is a first-class concern, reducing the risk of undocumented alternatives
* Good, because the template is modular; optional sections can be omitted for trivial decisions
* Good, because the community actively maintains templates, tooling (Log4brains, adr-tools), and examples
* Neutral, because numbering (`0001-`, `0002-`) is a convention rather than an enforced constraint
* Bad, because it has no built-in approval workflow; status is a free-text frontmatter field

### Nygard-style ADR (Original Michael Nygard Format)

* Good, because it is the original and most widely recognised ADR format
* Good, because its minimal five-section structure is easy to learn and quick to write
* Neutral, because many teams extend it informally, which produces inconsistency over time
* Bad, because it does not include a structured section for considered alternatives, making the reasoning less auditable
* Bad, because there is no built-in mechanism for documenting decision drivers or trade-off analysis

### RFC (Request for Comments)

* Good, because it is extremely flexible; no mandatory structure
* Good, because it suits open-ended, community-reviewed decisions well (open source, cross-team standards)
* Neutral, because some teams add their own templates on top of the RFC convention, converging toward MADR anyway
* Bad, because the convention favors prose-heavy, discussion-oriented documents over concise, structured decision records
* Bad, because without an enforced template, multi-option comparison is rarely included, making it easy to justify a decision without documenting rejected alternatives
* Bad, because RFCs are typically long documents, which discourages their use for routine architectural decisions

### Arc42 Decision Section

* Good, because arc42 is widely known in enterprise and consulting contexts (especially in Germany/DACH)
* Good, because decisions live inside a broader, structured documentation framework covering the full architecture
* Neutral, because arc42 section 9 does not prescribe a specific per-decision format; teams typically combine it with Nygard or MADR internally
* Bad, because the arc42 decision section is designed as part of a broader documentation framework; using it in isolation loses the cross-referencing benefits that justify its complexity
* Bad, because it is document-oriented rather than record-oriented; individual decisions are harder to discover, link, and version independently

### No formal ADR process

* Good, because it has zero process overhead; contributors focus entirely on code
* Good, because it requires no onboarding or tooling
* Neutral, because decisions can still be captured informally in commit messages and PR descriptions
* Bad, because there is no structured audit trail; reconstructing the reasoning behind past decisions requires searching through scattered PR discussions, commit messages, and tribal knowledge
* Bad, because onboarding new contributors is harder without a discoverable record of why things are the way they are
* Bad, because decisions are easily revisited without awareness of prior analysis, leading to repeated debates

## More Information

This ADR is necessarily self-referential: the MADR format was adopted provisionally in order to write this record. Rejection of this ADR would require migrating existing records to the chosen alternative format.

Y-Statements were considered but excluded from the options list because they serve a complementary purpose (single-sentence decision summaries) rather than functioning as a standalone decision record format. A Y-Statement summary can be added to the top of any MADR record without conflict.

* MADR 4.0.0 specification and examples: https://adr.github.io/madr/
* Log4brains (MADR-based ADR portal): https://github.com/thomvaill/log4brains
* arc42 template: https://arc42.org/download
* Original Nygard ADR post: https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions
* Y-Statements: https://medium.com/olzzhas/y-statements-10eb07b5a177
* Comparison of ADR formats: https://adr.github.io/#existing-adr-templates
* Revisit this decision if the team scales significantly or adopts a documentation platform (e.g., Confluence) that warrants a format with deeper integration.
