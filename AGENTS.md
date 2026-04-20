# AGENTS.md

**This document sets forth the required rules, standards, and workflow all agents must follow when generating or modifying content in this repository.**

_The keywords "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" used herein are interpreted as specified in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119.txt)._

---

## Table of Contents

[Purpose](#purpose)
[Core Principles](#core-principles)
[Character Set](#character-set)
[Formatting Rules](#formatting-rules)
[Compliance](#compliance)
[General rules](#general-rules)
[Workflow](#workflow)
[Task Management](#task-management)

---

## Purpose

This document defines the rules, workflows, and guardrails every agent (human or AI) MUST follow when authoring or modifying content in this repository.
If a rule in this file conflicts with instructions elsewhere, this file wins unless an ADR explicitly supersedes it.

---

## Core Principles

**Simplicity First**: Make every change as simple as possible. Impact minimal code.
**No Laziness**: Find root cause. No temporary fixes. Senior developer standards.
**Minimal Impact**: Changes should only touch what's necessary. Avoid introducing bugs.

---

## Character Set

Files are stored as UTF-8, but the content MUST restrict itself to the printable ASCII range (U+0020 to U+007E) plus line feed (LF, U+000A).

The following are PROHIBITED everywhere, including in code comments, commit messages, and documentation:

- Smart quotes (`U+2018`, `U+2019`, `U+201C`, `U+201D`).
- Typographic dashes (en-dash `U+2013`, em-dash `U+2014`).
- Ellipsis (`U+2026`). Use three ASCII periods (`...`) if absolutely needed.
- Non-breaking, zero-width, or any other invisible Unicode whitespace (`U+00A0`, `U+200B`, etc.).
- Control characters other than LF.
- Emoji and pictographic symbols.

When copying text from external sources, strip non-ASCII characters before committing.

---

## Formatting Rules

- Files MUST be free of hidden or non-printable characters.
- Use simple, consistent Markdown. Prefer tables for structured data and fenced code blocks for commands.
- All Markdown tables MUST have the same number of columns in every row, and column widths MUST be uniform throughout the table.
- Relative links between repository files MUST use relative paths (e.g., `doc/adr/0001-use-conventional-commits.md`), not absolute URLs.
- `.editorconfig` governs all files.

---

## Compliance

- Any violation of these rules MUST be corrected prior to committing.
- Failure to comply MAY result in rejection of the generated content.
- Run `mise run distro:hygiene` to verify compliance before committing.

---

## General rules

- Do NOT commit secrets, credentials, tokens, or API keys.

---

## Workflow

### 1. Plan Mode Default
- Enter plan mode for ANY non-trivial task (3+ steps or architectural decisions).
- If something goes sideways, STOP and re-plan immediately - don't keep pushing.
- Use plan mode for verification steps, not just building.
- Write detailed specs upfront to reduce ambiguity.

### 2. Subagent strategy
- Use subagents liberally to keep main context window clean.
- Offload research, exploration, and parallel analysis to subagents.
- For complex problems, throw more compute at it via subagents.
- One task per subagent for focused execution.

### 3. Self-Improvement Loop
- After any correction from the user: update `tasks/lessions.md` with the pattern.
- Write rules for yourself that prevent the same mistake.
- Ruthlessly iterate on these lessons until mistake rate drops.
- Review lessons at session start for relevant projects.

### 4. Verification Before Done
- Never mark a task complete without proving it works.
- Diff behavior between main and your changes when relevant.
- Ask yourself: "Would a staff engineer approve this?"
- Run tests, check logs, demonstrate correctness.

### 5. Demand Elegance (Balanced)
- For non-trivial changes: pause and ask "is there a more elegant way?".
- If a fix feels hacky: "Knowing everything I know now, implement the elegant solution".
- Skip this for simple, obvious fixes - don't over-engineer.
- Challenge your own work before presenting it.

### 6. Autonomous Bug Fixing
- When given a bug report: just fix it. Don't ask for hand-holding.
- Point at logs, errors, failing tests - then resolve them.
- Zero context switching required from the user.
- Go fix failing CI tests without being told how.

## Task Management

1. **Plan First**: Write plan to `tasks/todo.md` with checkable items
2. **Verify Plan**: Check in before starting implementation
3. **Track Progress**: Mark items complete as you go
4. **Explain Changes**: High-level summary of each step
5. **Document Results**: Add review section to `tasks/todo.md`
6. **Capture Lessons**: Update `tasks/lessons.md` after corrections

---
