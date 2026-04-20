---
status: accepted
date: 2026-04-15
decision-makers: Aleksandar Buza
consulted: null
informed: null
---

# SDLC Workflow

## Context and Problem Statement

The project needs a coherent, end-to-end software development lifecycle (SDLC) that addresses seven tightly coupled concerns:

1. How commit messages are written.
2. How versions and changelogs are produced.
3. How contributors integrate changes.
4. How branches are named.
5. How quality gates run before code leaves a developer's machine.
6. How the commit-message and branch-name conventions are enforced identically in local and CI environments.
7. How the tools that execute the above are provisioned and pinned across every contributor workstation and CI runner.

These concerns are interdependent. The commit message format drives automated versioning; the branching strategy determines what the versioning tool reads; branch names communicate intent and are themselves machine-parsed; pre-commit checks enforce the formats everything else relies on; the validation tool is the single authority on whether a message or branch name is well-formed; and the toolchain provisioning layer determines whether any of the above runs identically on a laptop and in CI. Treating the seven concerns as a single decision avoids internal contradictions, makes the workflow easy to describe end-to-end, and ensures that a change to one concern (for example, replacing the validation tool) is reviewed against all the others it touches.

This ADR covers the full lifecycle from the moment a contributor creates a branch to the moment a new version is tagged on `main`. It does not cover the organisation of source code, release-artefact publishing, or runtime deployment, which are addressed separately.

## Decision Drivers

* **Machine-readable commit intent.** Commit messages MUST clearly communicate intent and be parseable without natural-language heuristics so that automated changelog generation and semantic versioning can be driven directly from Git history.
* **Deterministic versioning.** Version increments MUST be derived from commit history, not from manual decisions, so that every release is reproducible and auditable.
* **Dual-runnable releases.** The release process MUST be fully automatable in CI AND runnable on a developer's machine without a different setup, so that developers can verify a release end-to-end locally before touching CI configuration.
* **No ecosystem-specific runtime for SDLC tooling.** The SDLC toolchain MUST NOT require Node.js, Ruby, or any other ecosystem that is not already a first-class project dependency, so that the tooling cost is bounded by the project's own stack.
* **Deployable `main`.** The branching workflow MUST keep `main` always in a releasable state and require code review via pull requests before any change lands.
* **Linear, machine-readable history.** Each merged pull request MUST produce exactly one clean commit on `main` so that the versioning tool has an unambiguous input and history browsing remains straightforward.
* **Consistent, parseable branch names.** Branch names MUST follow a consistent format that communicates intent at a glance and that tooling (dashboards, automations, branch protection) can reason about mechanically.
* **Pre-push feedback.** Formatting, security, quality, commit-message, and branch-name issues MUST be caught before code leaves the developer's machine, shortening the feedback loop and reducing noise in code review.
* **Shared check configuration.** Check configuration MUST be language-agnostic, version-controlled, and shared identically across all contributors and CI so that local and CI results cannot diverge.
* **Single-command onboarding.** Onboarding friction MUST be low: after cloning, one command should be sufficient to reach a working environment with all tools installed at the pinned versions and all Git hooks registered.
* **Reproducible toolchain.** Tool versions MUST be pinned and provisioned through a single, contributor-neutral mechanism so that the workflow behaves identically on every workstation and CI runner.

## Considered Options

The workflow is decomposed into seven concerns. Options are listed per concern; the resulting combination is evaluated as a whole in Decision Outcome.

### Commit message specification

* [Conventional Commits 1.0.0](https://www.conventionalcommits.org/en/v1.0.0/).
* [Angular commit message guidelines](https://github.com/angular/angular/blob/main/CONTRIBUTING.md#-commit-message-format).
* Freeform commit messages.

### Versioning and changelog

* [cocogitto](https://github.com/cocogitto/cocogitto).
* [semantic-release](https://github.com/semantic-release/semantic-release).
* [release-please](https://github.com/googleapis/release-please).
* Manual versioning.

### Branching strategy

* [GitHub Flow](https://docs.github.com/en/get-started/using-github/github-flow).
* [Git Flow](https://nvie.com/posts/a-successful-git-branching-model/).
* [Trunk-Based Development](https://trunkbaseddevelopment.com/).

### Branch naming specification

* [Conventional Branch](https://github.com/conventional-branch/conventional-branch).
* Ad-hoc branch names.
* Ticket-id-only branch names (e.g. `JIRA-123`).

### Local check enforcement

* [pre-commit](https://pre-commit.com/).
* [Husky](https://typicode.github.io/husky/).
* Manual conventions.
* CI-only checks.

### Validation tool

* [commit-check](https://github.com/commit-check/commit-check).
* [conventional-pre-commit](https://github.com/compilerla/conventional-pre-commit) combined with a custom branch-name regex and a separate CI PR-title check.
* [commitlint](https://commitlint.js.org/).

### Toolchain provisioning

* [mise](https://mise.jdx.dev/) combined with [uv](https://docs.astral.sh/uv/) for Python tooling.
* Direct installation of each tool via the operating system package manager or the tool's upstream installer.
* A project-local bootstrap script that downloads tool binaries to a versioned cache.

## Decision Outcome

The chosen workflow consists of seven coordinated choices. Each choice addresses one concern, and every choice was evaluated against the others to ensure there are no contradictions.

1. **Commit messages follow [Conventional Commits 1.0.0](https://www.conventionalcommits.org/en/v1.0.0/).** A lightweight, vendor-neutral specification that makes commit intent explicit and maps directly onto Semantic Versioning. It is the most widely adopted option, has a stable 1.0.0 release, and is natively understood by `cocogitto` and `commit-check`.
2. **Versioning and changelog are driven by [cocogitto](https://github.com/cocogitto/cocogitto).** A self-contained, language-agnostic CLI built around Conventional Commits and SemVer that runs identically in CI and locally. It ships as a single statically linked binary with no ecosystem-specific runtime dependency, which satisfies the "no ecosystem-specific runtime" driver better than `semantic-release` or `release-please`.
3. **Branching follows [GitHub Flow](https://docs.github.com/en/get-started/using-github/github-flow) with squash-only merges.** The simplest strategy that keeps `main` deployable, enforces code review through pull requests, and produces exactly one clean commit per PR for `cocogitto` to parse.
4. **Branch names follow [Conventional Branch](https://github.com/conventional-branch/conventional-branch).** Branches MUST use one of three shapes: `<type>/<short-description>`, `<type>/<ISSUE-ID>/<short-description>`, or `<type>/<USERNAME>/<short-description>`. The permitted types are the Conventional Commits types (`feat`, `fix`, `chore`, `docs`, `style`, `refactor`, `perf`, `test`, `ci`, `build`) plus `hotfix` and `release`.
5. **Local check enforcement is provided by [pre-commit](https://pre-commit.com/).** The de facto standard for polyglot Git hook management, configured in a single version-controlled `.pre-commit-config.yaml`. `pre-commit` itself is installed as a Python package via `uv` (see choice 7).
6. **Conventional Commits and Conventional Branch are validated by [commit-check](https://github.com/commit-check/commit-check).** A single tool that covers commit message format, branch naming, committer identity, and PR-title validation from one configuration file (`.commit-check.yml`). It is installed as a Python package via `uv` and wired into `.pre-commit-config.yaml` as a `language: system` local hook on the `commit-msg` and `pre-push` stages. The same binary runs in CI, guaranteeing that local and CI rules cannot drift apart.
7. **The SDLC toolchain is provisioned by `mise` and `uv`.** `mise` installs `cocogitto`, `python`, and `uv` at pinned versions declared in committed `mise` configuration files at the repository root. `uv` installs `pre-commit` and `commit-check` as Python packages, declared in `pyproject.toml` and locked in `uv.lock`. No SDLC tool is installed through any other mechanism, and contributors never install any of these tools globally.

These seven choices reinforce each other. The `commit-check` binary installed by `uv` validates both Conventional Commits and Conventional Branch locally via `pre-commit`; the same binary runs in CI and validates the PR title that GitHub Flow turns into the squash commit on `main`; `cocogitto` reads those squash commits on `main` to bump versions and regenerate changelogs. Branch names, commit types, and PR titles share a single vocabulary, so intent is consistent from the moment a branch is created through to the final changelog entry. Because every tool in this chain is installed deterministically through `mise` and `uv`, the workflow behaves identically on a laptop and in CI.

### Consequences

* **Good**, because commit types (`feat`, `fix`, `chore`, etc.) make the nature of each change immediately clear and drive automated MAJOR/MINOR/PATCH bumps without manual classification.
* **Good**, because version numbers and `CHANGELOG.md` files are derived deterministically from commit history: no manual release notes, no subjective version decisions, and no human error at the point of release.
* **Good**, because `cog bump --auto` can be executed either by a developer locally or in CI via the `cocogitto-action` GitHub Action; `cocogitto` ships as a single binary with no runtime dependency, so the two execution paths produce byte-equivalent output.
* **Good**, because GitHub Flow is trivial to onboard: branch from `main`, open a PR, squash-merge, deploy. `main` is always production-ready.
* **Good**, because squash merging produces exactly one commit per PR, keeping history linear and making the PR title the canonical input that `cocogitto` reads.
* **Good**, because Conventional Branch names share vocabulary with Conventional Commits, so contributors internalise one set of types for branches, commits, and PR titles; reviewers can spot mismatches (e.g. a `fix/` branch that opens a `feat:` PR) at a glance.
* **Good**, because `pre-commit` catches issues before they enter the repository, reducing CI failures and review noise. The same hooks can be re-run in CI via `pre-commit run --all-files` for identical results.
* **Good**, because `commit-check` provides a single configuration surface for Conventional Commits, Conventional Branch, and committer-identity checks, eliminating the split between `conventional-pre-commit` (messages only) and a hand-rolled branch-name regex.
* **Good**, because `commit-check` runs as a `language: system` local hook wired to the `uv`-installed binary, so the pre-commit path, the ad-hoc CLI path, and the CI path all resolve to the same executable.
* **Good**, because `mise` pins `cocogitto`, `python`, and `uv` at exact versions committed to the repository, and `uv` pins `pre-commit` and `commit-check` at exact versions via `uv.lock`. Every contributor and every CI run uses the same bytes.
* **Good**, because every SDLC tool is installed through the same two-step mechanism (`mise install` followed by `uv sync`), executed as part of the `distro:bootstrap` mise task described below. Onboarding is a single command.
* **Bad**, because the Conventional Commits discipline becomes load-bearing: a malformed commit or PR title can suppress or mis-classify a release. This is mitigated by the `commit-msg` and `pre-push` pre-commit hooks (local) and the `commit-check-action` (CI) enforcing the format on every commit and every PR.
* **Bad**, because contributors must run `mise run distro:bootstrap` once after cloning to install tools and register pre-commit hooks. The bootstrap task is the single documented entry point for this.
* **Bad**, because GitHub Flow does not prescribe a release branching strategy for projects that must maintain multiple active release lines in parallel. This is acceptable for the current scope of the project; if the situation changes, this ADR will be superseded.
* **Bad**, because individual commit history within a branch is lost after squash merge. The PR itself is the durable record of intra-branch context, which is a deliberate trade-off in favour of a clean, linear `main`.
* **Neutral**, because `cocogitto` has a smaller plugin ecosystem than `semantic-release`, but its built-in scope (commit validation, bump, changelog, tagging) already covers the full release workflow without plugins.
* **Neutral**, because `commit-check` is a Python tool. Since `python` and `uv` are already managed by `mise`, no system-level Python is required.

### Confirmation

The workflow is confirmed as active when all of the following are true:

* `pyproject.toml` declares `pre-commit` and `commit-check` as development dependencies, and `uv.lock` is committed with the resolved versions pinned.
* `.pre-commit-config.yaml` is present at the repository root and wires `commit-check` as a `language: system` local hook on the `commit-msg` stage (commit messages) and the `pre-push` stage (branch names), invoking the `commit-check` binary provided by the project's `uv`-managed virtual environment.
* `.commit-check.yml` is present at the repository root and configures the Conventional Commits rules for messages and PR titles, and the Conventional Branch regex for head branch names.
* `mise run distro:bootstrap` installs `cocogitto`, `python`, and `uv`; runs `uv sync` to install `pre-commit` and `commit-check`; and runs `pre-commit install --hook-type commit-msg --hook-type pre-push` so that the hooks are registered in the local Git repository.
* CI runs `pre-commit run --all-files` and the `commit-check-action` on every pull request, covering PR title, head branch name, and all commits on the branch.
* GitHub repository settings have "Allow squash merging" enabled, and "Allow merge commits" and "Allow rebase merging" disabled, making squash the only permitted merge strategy.
* Branch protection rules require at least one approving review, passing CI checks, and a PR title matching Conventional Commits before merge.
* A `PULL_REQUEST_TEMPLATE.md` is present in `.github/` to guide contributors on the expected PR title and description format.
* `cog.toml` is present at the repository root. On every merge to `main`, the `cocogitto-action` job runs `cog bump --auto`, producing new Git tags, updated `CHANGELOG.md` files, and version increments matching the highest-impact commit type since the last release.
* Every commit on `main` has a message in Conventional Commits format, verifiable with `cog check`.

## Pros and Cons of the Options

### Commit message specification

#### Conventional Commits 1.0.0

<https://www.conventionalcommits.org/en/v1.0.0/>

Commit message format:

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

Common types: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `chore`, `ci`, `build`. Breaking changes are indicated by `BREAKING CHANGE:` in the footer or by appending `!` after the type or scope (e.g. `feat!:`, `feat(api)!:`).

* Good, because it is a vendor-neutral open specification with a stable 1.0.0 release.
* Good, because it maps directly onto Semantic Versioning: `fix` -> PATCH, `feat` -> MINOR, `BREAKING CHANGE` -> MAJOR.
* Good, because it has broad ecosystem support (`cocogitto`, `commit-check`, `conventional-changelog`, and numerous others).
* Neutral, because it requires tooling or discipline to enforce consistently. The combination of `pre-commit` and `commit-check` chosen below addresses this directly.

#### Angular commit message guidelines

<https://github.com/angular/angular/blob/main/CONTRIBUTING.md#-commit-message-format>

* Good, because Conventional Commits is directly derived from this format, so migration in either direction is trivial.
* Bad, because it is scoped to the Angular project and framed as Angular-specific guidance rather than a general-purpose standard.
* Bad, because it lacks a versioned specification document, which makes it difficult to reference normatively or to pin a conformance target.

#### Freeform commit messages

* Good, because there is zero overhead or convention to learn.
* Bad, because history becomes inconsistent and hard to scan across contributors and over time.
* Bad, because downstream automation (changelog, versioning) is not possible without parsing unstructured text.
* Bad, because the intent of a commit is not immediately apparent from the subject line.

### Versioning and changelog

#### cocogitto

<https://github.com/cocogitto/cocogitto>

* Good, because it is a single statically linked binary with no ecosystem-specific runtime dependency: it works in any CI environment and locally, installed through `mise`.
* Good, because it covers the full release workflow from one CLI: commit validation (`cog check`), bump (`cog bump --auto`), changelog (`cog changelog`), and tagging.
* Good, because `cog bump --auto` determines the next version automatically from commit history; developers can also run it manually with `--patch`, `--minor`, or `--major` overrides when an exceptional bump is required.
* Good, because it is purpose-built for Conventional Commits and Semantic Versioning with no plugin configuration required for core use.
* Neutral, because its plugin and hook ecosystem is smaller than `semantic-release`'s; this is acceptable because its built-in scope already covers the full release workflow for this project.

#### semantic-release

<https://github.com/semantic-release/semantic-release>

* Good, because it is widely adopted with a large plugin ecosystem (GitHub Releases, npm, Docker, and others).
* Good, because the entire release process runs in a single CI step.
* Bad, because it requires Node.js in the CI environment even for non-JavaScript projects, which contradicts the "no ecosystem-specific runtime for SDLC tooling" driver.
* Bad, because it is effectively CI-only: there is no straightforward path to running a release from a developer's machine with the same configuration and output.
* Bad, because its plugin-based architecture adds configuration overhead for non-npm projects.

#### release-please

<https://github.com/googleapis/release-please>

* Good, because it automates versioning from Conventional Commits.
* Good, because it works well in GitHub Actions-centric workflows via the `release-please-action`.
* Bad, because it follows a PR-based release model (it opens a release PR) rather than releasing directly on merge, which adds an extra manual review step that is not required by the current workflow.
* Bad, because it requires Node.js, which contradicts the same driver as `semantic-release`.
* Neutral, because the PR-based model provides an explicit review checkpoint before release, which may be desirable in some workflows but is not required here.

#### Manual versioning

* Good, because it requires no tooling setup.
* Bad, because version decisions are subjective and inconsistent across contributors.
* Bad, because changelog maintenance is time-consuming and is often neglected under pressure.
* Bad, because human error can result in incorrect semver increments or missed releases.

### Branching strategy

#### GitHub Flow

<https://docs.github.com/en/get-started/using-github/github-flow>

Core rules adopted for this project:

1. `main` is always deployable.
2. New work is done on a descriptively named branch created from `main`, following the Conventional Branch specification.
3. A pull request is opened as early as possible to facilitate discussion, using the repository pull request template.
4. The pull request title MUST follow the Conventional Commits format because it becomes the squash commit message on `main`.
5. Changes are reviewed and CI must pass before merging.
6. Pull requests are ALWAYS merged using squash merge. Merge commits and rebase merges are disabled at the repository level.
7. Once merged to `main`, the change can be deployed immediately.

* Good, because it has minimal rules and is trivial to onboard new contributors.
* Good, because pull requests are a first-class part of the workflow, not an afterthought.
* Good, because it maps directly onto GitHub's branch protection and CI integration features.
* Neutral, because it assumes a single production environment or that `main` tracks the latest release; this is acceptable for the current scope.

#### Git Flow

<https://nvie.com/posts/a-successful-git-branching-model/>

* Good, because it provides explicit branches for features, releases, hotfixes, and maintenance, which suits projects with scheduled release cycles and long-lived support branches.
* Bad, because it introduces significant complexity: two long-lived branches (`main` and `develop`) plus several supporting branch types.
* Bad, because merging overhead and branch synchronisation costs increase with team size.
* Bad, because the author of Git Flow later acknowledged that it is not suitable for continuously delivered software.

#### Trunk-Based Development

<https://trunkbaseddevelopment.com/>

* Good, because it enforces the shortest possible integration cycle: all commits land on trunk quickly.
* Good, because it eliminates the merge conflicts that accumulate on long-lived branches.
* Bad, because it relies heavily on feature flags and a mature CI pipeline to keep trunk stable when incomplete features are committed directly.
* Neutral, because GitHub Flow is a pragmatic middle ground between Git Flow and Trunk-Based Development that retains short-lived branches and pull requests without imposing feature-flag discipline.

### Branch naming specification

#### Conventional Branch

<https://github.com/conventional-branch/conventional-branch>

Branch name format. One of the following shapes, chosen based on context:

```
<type>/<short-description>
<type>/<ISSUE-ID>/<short-description>
<type>/<USERNAME>/<short-description>
```

Rules:

1. The `<type>` segment is one of the Conventional Commits types (`feat`, `fix`, `chore`, `docs`, `style`, `refactor`, `perf`, `test`, `ci`, `build`) plus `hotfix` and `release`.
2. The `<short-description>` segment uses lowercase letters, digits, and hyphens only. Spaces, underscores, and camelCase are not permitted.
3. The `<ISSUE-ID>` segment is the identifier used by the project's issue tracker (e.g. `JIRA-123`, `GH-456` for GitHub Issues, `LIN-789` for Linear). Case is preserved from the tracker.
4. The `<USERNAME>` segment is the contributor's handle on the Git host (e.g. `alekbuza`). It is used only in forked repositories or when the work has no corresponding tracked issue (spikes, experiments, quick fixes).
5. No trailing or consecutive slashes. Exactly one segment (issue ID or username) sits between the type and the short description when the three-segment form is used.

When to use which shape:

* **`<type>/<short-description>`** is the default for small, self-contained work where a ticket reference adds no value.
* **`<type>/<ISSUE-ID>/<short-description>`** is required whenever the work maps to a tracked issue. The slash-separated form is preferred over appending the ID to the description because Git hosts and CLI tools display branches as a directory tree, so all branches for a given ticket group together under one node (e.g. `feat/JIRA-123/login-form` and `feat/JIRA-123/login-form-analytics` both sit under `feat/JIRA-123/`). This naturally supports multiple branches per ticket, for example a spike followed by the main implementation, or a front-end branch and a back-end branch sharing one ticket.
* **`<type>/<USERNAME>/<short-description>`** is used in forked repositories (where the convention helps distinguish contributor branches when fetching from multiple remotes) or when a contributor is working without an issue tracker reference.

Examples:

* `feat/add-login-form` - simple feature, no ticket.
* `feat/JIRA-123/add-login-form` - feature tracked in Jira; peers under `feat/JIRA-123/` can hold related follow-up work.
* `fix/GH-456/authentication-bug` - bug fix tracked as GitHub Issue #456.
* `feat/alekbuza/experimental-cache-layer` - spike by contributor `alekbuza` with no tracked issue.
* `fix/alekbuza/typo-in-readme` - fork contribution without a ticket reference.
* `hotfix/critical-security-patch`, `release/v1.2.0` - type-prefixed operational branches.

* Good, because it is a vendor-neutral, lightweight specification that mirrors the semantics of Conventional Commits at the branch level.
* Good, because branch names immediately communicate intent to reviewers and CI dashboards without requiring them to read the diff.
* Good, because the slash-separated issue-id form exposes a directory-like structure in Git UIs, so multiple branches per ticket group naturally under one node.
* Good, because the format is issue-tracker-agnostic: Jira, GitHub Issues, Linear, GitLab, and others all fit the same pattern without changes to the spec.
* Good, because the username form gives forked-repo contributors and untracked-work branches a consistent home while still conveying type.
* Good, because the format is trivial to validate with a regex, enabling both local hooks and CI checks.
* Good, because sharing vocabulary with Conventional Commits reduces cognitive load: contributors learn one set of types for branches, commits, and PR titles.
* Neutral, because it requires enforcement (a pre-commit hook and a CI check) to prevent drift; the regex-based validation is straightforward to configure.

#### Ad-hoc branch names

* Good, because there is zero convention to learn.
* Bad, because branch purpose is opaque until someone opens the PR or reads the commits.
* Bad, because tooling cannot route or filter branches by type (for example, "show all open fix branches").
* Bad, because inconsistent naming creates friction in code review and operational dashboards.

#### Ticket-id-only branch names (e.g. `JIRA-123`)

* Good, because every branch is unambiguously tied to a tracked work item.
* Bad, because the name alone conveys nothing about the nature of the change: reviewers must look up the ticket.
* Bad, because branches created without a ticket (experiments, spikes, tiny fixes) have no naming convention to fall back on.
* Bad, because tight coupling to a specific issue tracker makes migration painful if the tracker changes.

### Local check enforcement

#### pre-commit

<https://pre-commit.com/>

* Good, because it supports hooks written in any language (Python, shell, Docker, and others) without requiring contributors to manage those runtimes themselves.
* Good, because hooks are pinned to specific versions in `.pre-commit-config.yaml`, making checks reproducible.
* Good, because it has a large ecosystem of existing hook repositories covering formatting, linting, secrets detection, YAML and JSON validation, and more.
* Good, because `pre-commit run --all-files` makes it easy to apply all checks retroactively or in CI with identical behaviour.
* Good, because `local` hooks of `language: system` allow `uv`-installed binaries (such as `commit-check`) to be wired in without duplicating the install path.
* Neutral, because contributors must explicitly register the hooks after cloning; the `distro:bootstrap` mise task described in More Information does this on their behalf, so no manual step is required.

#### Husky

<https://typicode.github.io/husky/>

* Good, because it integrates tightly with npm and Node.js projects and is simple to configure in JavaScript ecosystems.
* Good, because hooks are committed to the repository and activated automatically via the `prepare` npm lifecycle script.
* Bad, because it requires Node.js and npm to be present, which contradicts the "no ecosystem-specific runtime for SDLC tooling" driver.
* Bad, because hook scripts are plain shell: contributors must install and manage any required linters or formatters themselves.
* Bad, because there is no built-in isolation of hook dependencies, which allows version drift between contributors.
* Bad, because the hook ecosystem is not standardised; each team assembles scripts manually rather than pulling from a shared registry.

#### Manual conventions

* Good, because there is zero tooling overhead.
* Bad, because compliance depends entirely on individual discipline; enforcement is impossible.
* Bad, because check commands and versions vary between contributors, producing inconsistent results.
* Bad, because issues surface in CI or review instead of locally, which lengthens the feedback loop.

#### CI-only checks

* Good, because no local setup is required from contributors.
* Bad, because the feedback loop is long: contributors must push and wait for CI to learn about trivial issues.
* Bad, because CI time and cost increase as checks accumulate.
* Bad, because bad commits can enter the repository temporarily until CI fails.

### Validation tool

#### commit-check

<https://github.com/commit-check/commit-check>

`commit-check` is a multi-purpose validator that covers commit message format, branch name, committer name and email, merge base, and PR title in a single configuration file (`.commit-check.yml`). It is distributed as a Python package and is installed via `uv`, exposing a CLI that is used both as a pre-commit hook (`language: system` local hook) and as the `commit-check-action` GitHub Action.

* Good, because it enforces Conventional Commits AND Conventional Branch from one tool and one configuration file, eliminating the split between `conventional-pre-commit` (messages only) and a hand-rolled branch-name regex.
* Good, because the same binary runs as a pre-commit hook, an ad-hoc CLI, and a GitHub Action, guaranteeing identical validation locally and in CI.
* Good, because it validates PR titles in CI (via `commit-check-action`), closing the loop on GitHub Flow's requirement that the squash commit (derived from the PR title) conforms to Conventional Commits.
* Good, because additional checks (author/email, merge-base, signoff) can be enabled later from the same configuration without introducing new tools.
* Good, because it is vendor-neutral and issue-tracker-agnostic: regex-configurable to fit any Conventional Branch variant, including the `<type>/<ISSUE-ID>/<short-description>` and `<type>/<USERNAME>/<short-description>` shapes adopted here.
* Neutral, because it is a Python tool. Since `python` and `uv` are already managed by `mise` (see choice 7 in Decision Outcome), contributors do not need any Python setup beyond what the toolchain provides.

#### conventional-pre-commit combined with a custom branch-name regex

* Good, because `conventional-pre-commit` is narrowly focused and well-known in the Conventional Commits ecosystem.
* Bad, because it only validates commit messages: branch-name enforcement requires a separate hand-maintained regex hook and an independent CI check, doubling the configuration surface.
* Bad, because there is no built-in PR-title validator, so a third mechanism is needed in CI.
* Bad, because local and CI rules can drift apart when implemented as three separate mechanisms.

#### commitlint

<https://commitlint.js.org/>

* Good, because it has a large ecosystem of shared configs and plugins around Conventional Commits.
* Bad, because it requires Node.js, which contradicts the "no ecosystem-specific runtime for SDLC tooling" driver already adopted for the versioning toolchain (`cocogitto`).
* Bad, because it focuses on commit messages; branch-name and committer validation require extra tooling.
* Bad, because its configuration surface (JavaScript config files, plugin packages) is heavier than a single `.commit-check.yml`.

### Toolchain provisioning

#### mise combined with uv

<https://mise.jdx.dev/> and <https://docs.astral.sh/uv/>

* Good, because `mise` pins `cocogitto`, `python`, and `uv` at exact versions committed to the repository, and `uv` pins `pre-commit` and `commit-check` at exact versions via `uv.lock`. No SDLC tool version is implicit.
* Good, because the full toolchain is installed through a single bootstrap task (`mise run distro:bootstrap`, described in More Information), which also registers pre-commit hooks. Onboarding is one command.
* Good, because the same bootstrap path is used by CI (via `jdx/mise-action` followed by `uv sync`), which rules out "works locally but not in CI" failures for SDLC tooling.
* Good, because scoping `uv`-managed tools to Python packages keeps the dependency graph shallow: only `pre-commit` and `commit-check` are currently installed this way, both narrowly focused.
* Neutral, because contributors must install `mise` itself before the bootstrap task can run; this is a one-time step documented in the repository.

#### Direct installation via OS package manager or upstream installer

* Good, because it does not require learning a version manager.
* Bad, because tool versions drift between contributors and between local and CI environments.
* Bad, because each tool has a different installation path, which makes onboarding a long checklist instead of a single command.
* Bad, because no single file records the required tool versions, so upgrades and audits are ad-hoc.

#### Project-local bootstrap script

* Good, because it removes the external dependency on `mise` and `uv`.
* Bad, because it reimplements version pinning, cache management, and platform handling that `mise` and `uv` already solve.
* Bad, because maintenance and security review of the script become project work that adds no product value.
* Bad, because it has no ecosystem: no shared tool registry, no native CI integration, no standard upgrade path.

## More Information

### Specifications and tools

* Conventional Commits 1.0.0: <https://www.conventionalcommits.org/en/v1.0.0/>
* Conventional Branch specification: <https://github.com/conventional-branch/conventional-branch>
* GitHub Flow documentation: <https://docs.github.com/en/get-started/using-github/github-flow>
* GitHub Flow specification: <https://githubflow.github.io/>
* cocogitto documentation: <https://docs.cocogitto.io/>
* cocogitto GitHub Action: <https://github.com/cocogitto/cocogitto-action>. Handles the Git configuration and token permissions required for tagging and pushing releases.
* pre-commit documentation: <https://pre-commit.com/>
* pre-commit hook registry: <https://pre-commit.com/hooks.html>
* commit-check: <https://github.com/commit-check/commit-check>. Validates commit messages, branch names, committer identity, and PR titles against configurable rules.
* commit-check-action: <https://github.com/commit-check/commit-check-action>. The GitHub Action that runs `commit-check` on pull requests in CI.
* mise: <https://mise.jdx.dev/>
* uv: <https://docs.astral.sh/uv/>

### Toolchain and installation layout

The SDLC toolchain consists of exactly five tools, installed in two layers. This ADR introduces no further tools.

| Tool           | Installed by | Pinned in                          | Purpose                                                      |
| -------------- | ------------ | ---------------------------------- | ------------------------------------------------------------ |
| `cocogitto`    | `mise`       | `mise.dev.toml` / `mise.ci.toml`   | Conventional Commits validation, semver bumping, changelogs. |
| `python`       | `mise`       | `mise.dev.toml` / `mise.ci.toml`   | Runtime for `uv`-managed Python packages.                    |
| `uv`           | `mise`       | `mise.dev.toml` / `mise.ci.toml`   | Python package and environment manager.                      |
| `pre-commit`   | `uv`         | `pyproject.toml` + `uv.lock`       | Git hook framework.                                          |
| `commit-check` | `uv`         | `pyproject.toml` + `uv.lock`       | Conventional Commits and Conventional Branch validator.      |

Future SDLC tooling (additional formatters, linters, security scanners) MUST be wired in through the same two layers: `mise` for standalone binaries, `uv` for Python packages. No SDLC tool is installed through any other mechanism.

### Enforcement

* `commit-check` is the single tool responsible for validating Conventional Commits and Conventional Branch across local and CI environments. It is configured in `.commit-check.yml` at the repository root.
* Locally, `commit-check` is wired into `.pre-commit-config.yaml` as a `language: system` local hook on the `commit-msg` stage (commit messages) and the `pre-push` stage (branch names). The hook invokes the `commit-check` binary provided by the project's `uv`-managed virtual environment, so the pre-commit path, the ad-hoc CLI path, and the CI path resolve to the same executable.
* In CI, `commit-check-action` runs on every pull request and validates the PR title, head branch name, and all commits on the branch, using the same `.commit-check.yml` configuration.
* The configured branch-name regex accepts all three shapes defined in the Conventional Branch specification:

  ```
  ^(feat|fix|chore|docs|style|refactor|perf|test|ci|build|hotfix|release)/([A-Z][A-Z0-9]+-[0-9]+/|[a-z0-9][a-z0-9-]*/)?[a-z0-9]+(-[a-z0-9]+)*$
  ```

  This matches `<type>/<short-description>`, `<type>/<ISSUE-ID>/<short-description>`, and `<type>/<USERNAME>/<short-description>`, rejecting any other shape. The uppercase-prefixed alternative in the middle group matches issue IDs (e.g. `JIRA-123`, `GH-456`); the lowercase-prefixed alternative matches usernames. The middle group is optional, which admits the two-segment shape.
* Additional `pre-commit` hooks (formatting, linting, secret scanning, and so on) will be added as the project grows, all wired in through the same `.pre-commit-config.yaml` and installed through the same two-layer toolchain.

### Release workflow

On every merge to `main`, the `cocogitto-action` CI job runs automatically and:

1. Inspects the commit history since the last release to detect which packages have new commits (the packages set is read from `cog.toml`; single-package repositories treat the whole repository as one package).
2. Determines the appropriate version bump for each affected package based on commit types (`fix` -> PATCH, `feat` -> MINOR, `BREAKING CHANGE` -> MAJOR).
3. Creates a new Git tag for each bumped package.
4. Regenerates the `CHANGELOG.md` file for each bumped package with the new entries.

The same `cog bump --auto` command is also runnable on a developer's machine and produces identical output. No manual version management or changelog editing is required or permitted.

### mise configuration layout

`mise` configuration is split across four files at the repository root, so that SDLC tools never leak between contexts:

| File              | Committed | Purpose                                                                         |
| ----------------- | --------- | ------------------------------------------------------------------------------- |
| `mise.toml`       | yes       | Clean base configuration shared by all contexts. Minimal by design.             |
| `mise.dev.toml`   | yes       | Local development tools, including `cocogitto`, `python`, and `uv`.             |
| `mise.ci.toml`    | yes       | CI tools loaded by `jdx/mise-action`. Includes the SDLC toolchain used by CI.   |
| `mise.local.toml` | no        | Per-contributor local overrides and secrets. Listed in `.gitignore`.            |

CI workflows use the `jdx/mise-action` GitHub Action with `mise.ci.toml` as the configuration file, so CI installs exactly the pinned versions declared in the repository and never reaches for locally-scoped tooling.

### Bootstrap task

The `distro:bootstrap` task is the single entry point for setting up a local development environment after cloning. It is implemented as a mise file task (a standalone script under `.config/mise/tasks/distro/bootstrap`), so it is readable, reviewable, and directly executable without going through `mise`. The task MUST perform the following steps in order:

1. `mise install` - install `cocogitto`, `python`, `uv`, and any other tools declared in `mise.dev.toml` at pinned versions.
2. `uv sync` - install `pre-commit` and `commit-check` into the project's `uv`-managed virtual environment at the versions pinned in `uv.lock`.
3. `pre-commit install --hook-type commit-msg --hook-type pre-push` - register the Git hooks so that subsequent `git commit` and `git push` operations trigger the configured checks without any further manual step.

Contributors MUST run `mise run distro:bootstrap` once after cloning; no other SDLC setup steps should be required or documented outside of this task.

### Scope boundary

If a future ADR introduces release branches or multi-line maintenance, the branching section of this ADR will be superseded. The commit-message, branch-naming, validation, and toolchain sections are expected to remain stable. Any change to how tools are provisioned (for example replacing `mise` or `uv`) is, by definition, a change to this ADR.
