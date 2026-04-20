# Architecture Decision Records

This directory contains the Architecture Decision Records (ADRs) for this project.

ADRs are written using [MADR 4.0.0](https://adr.github.io/madr/) (Markdown Architectural Decision Records).
The template for new ADRs is in [_index.md](_index.md).

## Creating a new ADR

- Copy `doc/adr/_index.md` to `doc/adr/NNNN-short-title.md` using the next sequential number.
- Fill in all required frontmatter fields: `status` (use `proposed` initially), `date` (YYYY-MM-DD), `decision-makers`.
- Complete all mandatory sections: Context and Problem Statement, Considered Options, and Decision Outcome.
- Add a row to the index table in `doc/adr/README.md`.
- Submit the new ADR as a pull request for review.

### ADR statuses

| Status       | Meaning                                              |
|--------------|------------------------------------------------------|
| `proposed`   | Under discussion, not yet binding                    |
| `accepted`   | Binding - MUST be followed                           |
| `deprecated` | No longer recommended but not actively removed       |
| `superseded` | Replaced by a newer ADR (reference it in the footer) |
| `rejected`   | Considered and explicitly not adopted                |

## Records

| ID                                                         | Title                                           | Status     | Date       | Decision-makers |
|------------------------------------------------------------|-------------------------------------------------|------------|------------|-----------------|
| [0000](0000-use-madr-for-architecture-decision-records.md) | Use MADR for Architecture Decision Records      | `proposed` | 2026-04-15 | Aleksandar Buza |
| [0001](0001-sdlc-workflow.md)                              | Use mise for Development Environment Management | `proposed` | 2026-04-15 | Aleksandar Buza |

