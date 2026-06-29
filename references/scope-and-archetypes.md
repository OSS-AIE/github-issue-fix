# Scope Guard and Engineering Issue Archetypes

Use this reference when the requested fix may sprawl across files, when review asks for a narrower implementation, or when discovering candidate issues in a repository.

## Scope Guard

Apply this guard before editing and again before pushing:

1. State the issue contract in one sentence: the exact bug, review comment, CI failure, or contributor pain being fixed.
2. List the files that must change and why. If a file is not needed to satisfy the issue contract, leave it untouched.
3. Keep formatting changes limited to files already being edited, unless the formatter is the explicit issue.
4. Split unrelated cleanup into follow-up issues or PRs. Do not combine build-script cleanup, workflow cleanup, docs cleanup, and code refactors unless the issue explicitly requires that bundle.
5. Before committing, inspect `git diff --stat` and `git diff`. Revert or split any drive-by changes, broad rewrites, generated churn, or style-only edits unrelated to the issue contract.
6. If review feedback asks for a narrower approach, reduce the implementation first. Prefer one small step inside an existing job/script over adding new jobs, files, abstractions, or cross-cutting rewrites.

## Engineering Issue Archetypes

When discovering or fixing issues, classify them into one or more archetypes and use the matching validation path.

### Build Script Usability

Symptoms: confusing defaults, slow local loops, undocumented modes, shell behavior that differs across Linux/macOS/Windows/WSL.

Fix shape: improve CLI modes/help, preserve CI behavior, make changed-file handling explicit, and keep the script dependency-light.

Validate with shell syntax checks, help output, fake-command simulations in a temporary repo, and line-ending checks on Windows.

### Build Performance And Dependency Efficiency

Symptoms: repeated downloads, unpinned or hardcoded upstream refs, unnecessary full rebuilds, cache misses, heavyweight setup for narrow checks.

Fix shape: make refs/config centralized, preserve override precedence, improve cache keys, and split independent build stages only when it reduces work.

Validate with dry-run/build logs, cache-key reasoning, dependency graph checks, and a narrow CI equivalent.

### Unit-Test Efficiency And Reliability

Symptoms: slow test discovery, broad test selection, flaky environment probes, CPU/mock paths leaking into hardware tests, or hardware jobs doing avoidable setup.

Fix shape: tighten test selection, bound probes with clear failure modes, distinguish CPU/mock from hardware-routed tests, and avoid silently weakening tests.

Validate with focused pytest selection, static routing checks, timeout/fallback simulations, and CI selector output.

### Software Architecture And Build Dependency Analysis

Symptoms: unclear module boundaries, hidden import-time side effects, circular imports, duplicated dependency declarations, or build graph surprises.

Fix shape: document the dependency path, move shared constants/config to the owning layer, reduce import side effects, and avoid adding global mutable state.

Validate with import checks, build graph or package metadata inspection, targeted type/lint checks, and existing architecture docs.

### Duplicate Include, Import, Or Header Cleanup

Symptoms: repeated C/C++ headers, duplicate Python imports, unused includes after refactors, or order-sensitive import blocks.

Fix shape: remove only duplicates proven unused, preserve required ordering and platform guards, and avoid whole-file include sorting unless configured.

Validate with compiler/linter output, `ruff check` or include tooling, and targeted builds/tests for touched modules.

### CI And Workflow Correctness

Symptoms: duplicated workflow steps, hardcoded matrix values, inconsistent variable names, new jobs that only compute data for one downstream job, or stale review comments after workflow refactors.

Fix shape: prefer a step in the existing owning job, centralize one source of truth, keep output names semantic, and preserve manual/permission gates.

Validate with YAML parsing, `actionlint` when available, local script extraction tests, and `gh pr checks` after pushing.

### Documentation Contributor Experience

Symptoms: setup instructions that force full clone/env setup for a lint-only task, platform-specific commands presented as universal, or missing verification commands.

Fix shape: separate minimal lint/docs setup from full development setup, include clone/context steps only where needed, and keep commands copy-pasteable.

Validate rendered docs, link checks when practical, and platform notes for Linux/macOS/Windows/WSL.
