---
name: github-issue-fix
description: Fix upstream GitHub issues and pull request CI failures with GitHub CLI. Use when Codex is asked to fix one or more GitHub issues, triage open issues, prepare upstream pull requests, repair failing PR checks, resolve merge conflicts or needs-rebase PRs, handle maintainer review feedback or automated review comments, follow project contribution rules, distinguish code failures from permission/external CI gates, or generate a manual PR handoff when automatic PR creation is unavailable.
---

# GitHub Issue Fix

## Core Workflow

Use this skill for open-source upstream issue repair, PR creation, PR CI repair, and PR conflict repair.

1. Preserve local work. Check `git status --short`, current branch, remotes, and authentication before editing.
2. Read project instructions first: `AGENTS.md`, `CONTRIBUTING*`, `.github/PULL_REQUEST_TEMPLATE*`, `.github/workflows/`, pre-commit configs, test configs, and language/tooling files.
3. Fetch issue or PR metadata with `gh`: title, body, labels, assignees, linked PRs, comments, reviews, inline review threads, checks, statuses, and timeline signals.
4. Classify before coding:
   - `skip`: closed, duplicate, maintainer-owned, already fixed by a PR, or explicitly not wanted.
   - `needs-info`: insufficient reproduction, unclear expected behavior, private artifacts required, or likely design/RFC work.
   - `candidate`: narrow root cause, no active owner or existing fix, and at least one local/static/test/CI validation path.
5. Implement the smallest mergeable fix. Avoid unrelated refactors, formatting churn, broad architecture changes, or test weakening.
6. Add focused regression tests where practical. If hardware or large-model validation is required, still add a deterministic unit/meta/static check when possible.
7. Run the PR Quality Gate before committing.
8. Commit with project-required sign-off or trailers. For DCO projects, use `git commit -s`.
9. Push to a fork and create/update one PR per independent fix. Use a local HTML handoff if PR creation cannot be automated.

Industry pattern check: treat issue fixing as an issue-to-branch-to-PR loop with explicit review and validation feedback. Modern coding-agent flows assign or label issues, create a PR from an isolated branch, run tests/linters before handoff, respond to PR comments and inline review threads, and rely on humans or repository policy for merge and privileged CI gates.

Read `references/pr-quality-gate.md` before opening or updating a PR. Read `references/gh-workflow.md` before using `gh` for triage, fork setup, or PR updates.

## Scope Guard

Apply this guard before editing and again before pushing:

1. State the issue contract in one sentence: the exact bug, review comment, CI failure, or contributor pain being fixed.
2. List the files that must change and why. If a file is not needed to satisfy the issue contract, leave it untouched.
3. Keep formatting changes limited to files already being edited, unless the formatter is the explicit issue.
4. Split unrelated cleanup into follow-up issues or PRs. Do not combine build-script cleanup, workflow cleanup, docs cleanup, and code refactors unless the issue explicitly requires that bundle.
5. Before committing, inspect `git diff --stat` and `git diff`. Revert or split any drive-by changes, broad rewrites, generated churn, or style-only edits unrelated to the issue contract.
6. If review feedback asks for a narrower approach, reduce the implementation first. Prefer one small step inside an existing job/script over adding new jobs, files, abstractions, or cross-cutting rewrites.

## Engineering Issue Archetypes

When discovering or fixing issues, classify them into one or more archetypes and use the matching validation path:

1. Build script usability:
   - Symptoms: confusing defaults, slow local loops, undocumented modes, shell behavior that differs across Linux/macOS/Windows/WSL.
   - Fix shape: improve CLI modes/help, preserve CI behavior, make changed-file handling explicit, and keep the script dependency-light.
   - Validate with shell syntax checks, help output, fake-command simulations in a temporary repo, and line-ending checks on Windows.
2. Build performance and dependency efficiency:
   - Symptoms: repeated downloads, unpinned or hardcoded upstream refs, unnecessary full rebuilds, cache misses, heavyweight setup for narrow checks.
   - Fix shape: make refs/config centralized, preserve override precedence, improve cache keys, and split independent build stages only when it reduces work.
   - Validate with dry-run/build logs, cache-key reasoning, dependency graph checks, and a narrow CI equivalent.
3. Unit-test efficiency and reliability:
   - Symptoms: slow test discovery, broad test selection, flaky environment probes, CPU/mock paths leaking into hardware tests, or hardware jobs doing avoidable setup.
   - Fix shape: tighten test selection, bound probes with clear failure modes, distinguish CPU/mock from hardware-routed tests, and avoid silently weakening tests.
   - Validate with focused pytest selection, static routing checks, timeout/fallback simulations, and CI selector output.
4. Software architecture and build dependency analysis:
   - Symptoms: unclear module boundaries, hidden import-time side effects, circular imports, duplicated dependency declarations, or build graph surprises.
   - Fix shape: document the dependency path, move shared constants/config to the owning layer, reduce import side effects, and avoid adding global mutable state.
   - Validate with import checks, build graph or package metadata inspection, targeted type/lint checks, and existing architecture docs.
5. Duplicate include/import/header cleanup:
   - Symptoms: repeated C/C++ headers, duplicate Python imports, unused includes after refactors, or order-sensitive import blocks.
   - Fix shape: remove only duplicates proven unused, preserve required ordering and platform guards, and avoid whole-file include sorting unless configured.
   - Validate with compiler/linter output, `ruff check` or include tooling, and targeted builds/tests for touched modules.
6. CI/workflow correctness:
   - Symptoms: duplicated workflow steps, hardcoded matrix values, inconsistent variable names, new jobs that only compute data for one downstream job, or stale review comments after workflow refactors.
   - Fix shape: prefer a step in the existing owning job, centralize one source of truth, keep output names semantic, and preserve manual/permission gates.
   - Validate with YAML parsing, `actionlint` when available, local script extraction tests, and `gh pr checks` after pushing.
7. Documentation contributor experience:
   - Symptoms: setup instructions that force full clone/env setup for a lint-only task, platform-specific commands presented as universal, or missing verification commands.
   - Fix shape: separate minimal lint/docs setup from full development setup, include clone/context steps only where needed, and keep commands copy-pasteable.
   - Validate rendered docs, link checks when practical, and platform notes for Linux/macOS/Windows/WSL.

## Multi-Issue Workflow

When given multiple issue links or numbers:

1. Normalize every item to `<owner>/<repo>#<number>`.
2. Fetch metadata and search existing PRs for each issue.
3. Present or record a concise triage table.
4. Fix only `candidate` issues unless the user explicitly overrides.
5. Use one branch and one PR per independent root cause.
6. If the user explicitly asks for parallel agents, split independent candidate issues across agents with disjoint branches and write scopes.

Batch output should include issue number, decision, reason, branch, PR URL or handoff path, validation, and residual risks.

## Issue Fix Procedure

1. Read the issue, reproduction, logs, labels, assignees, maintainer comments, linked PRs, and project contribution rules.
2. Skip if a maintainer or contributor is clearly working on it or a linked PR already resolves it.
3. Locate related code and tests before changing files.
4. Reproduce locally when feasible. If not feasible due to GPU, OS, secrets, private models, or scale, state that clearly and proceed only when the code path is narrow enough for static/unit/CI validation.
5. Explain the root cause briefly while implementing.
6. Make minimal changes in the relevant module.
7. Add/update tests that would fail before the fix when practical.
8. Run changed-file checks and the closest local behavior checks.
9. Prepare a reviewer-friendly PR body with purpose, root cause, test plan, test result, limitations, and AI-assistance disclosure when appropriate.

## PR CI Fix Procedure

When given a PR link/number or asked to fix current-branch CI:

1. Inspect failing checks with `gh pr checks` and job logs with `gh run view --job --log`.
2. Also inspect PR comments, inline review comments, review summaries, status contexts, and automated reviews; CI may be blocked by a bot comment or external status rather than code.
3. Classify each failure:
   - permission/label gate,
   - lint/format,
   - typing,
   - pytest/unit/integration,
   - docs,
   - build/package,
   - platform or hardware,
   - flaky/environmental,
   - automated-review feedback.
4. Fix only failures caused by the PR. Do not hide failures by skipping tests, loosening assertions, or deleting checks unless the test itself is demonstrably wrong.
5. For permission or new-contributor gates, confirm the exact log message. If you cannot add the needed label, leave a maintainer-facing comment after local checks are clean.
6. For automated-review feedback, decide whether each point is a real defect, false positive, or optional polish. Fix real defects with tests.
7. Check whether comments are stale by comparing comment commit IDs with the current PR head before changing code.
8. Re-run the closest local equivalent, push fixes, and re-check PR status.
9. If all local checks are clean but a permission, trusted-contributor, label, or external-service gate remains, leave a precise maintainer-facing note instead of pushing no-op commits.

## PR Conflict Repair Procedure

Use this when a PR is marked `CONFLICTING`, `DIRTY`, `needs-rebase`, or the user says a submitted PR has merge conflicts.

1. Inspect PR state:
   - `gh pr view <pr> --json mergeable,mergeStateStatus,headRefName,headRepositoryOwner,baseRefName,labels`
   - `gh pr checks <pr>` to avoid mixing conflict repair with unrelated CI failures.
2. Work on the existing PR branch, preferably in an isolated worktree. Do not use `git reset --hard` or discard unrelated user work.
3. Fetch both upstream and fork remotes:
   - `git fetch <upstream> <base-branch>`
   - `git fetch <fork> <head-branch>`
4. Prefer a clean rebase onto the upstream base branch when the branch is small and linear:
   - `git switch <head-branch>`
   - `git rebase <upstream>/<base-branch>`
5. If rebase conflicts occur:
   - Resolve conflict markers by preserving the PR's intended behavior and the upstream version's newer structure.
   - Use `git diff --name-only --diff-filter=U` to list unresolved files.
   - Inspect upstream changes around each conflict before editing.
   - Re-run focused static and behavior checks for every touched area.
   - Continue with `git rebase --continue` only after the index is clean.
6. If the conflict is large or the upstream code invalidates the original approach, stop and classify as `needs-redesign` instead of forcing a stale patch.
7. Push with `--force-with-lease` only after a successful rebase, because updating an existing PR branch requires rewriting the branch tip.
8. Re-check `mergeable` and `mergeStateStatus`, then comment with a short conflict-resolution summary when useful.

## Fork and PR Rules

1. Use `gh auth status` and `git remote -v` to confirm access.
2. Prefer a fork remote owned by the user or organization. Create it with `gh repo fork` only when appropriate.
3. Start from the upstream default branch unless updating an existing PR branch.
4. Keep commits signed according to project rules.
5. Use project title prefixes and PR template sections.
6. Before opening or updating a PR, inspect the repository's title validation rules, style guide, and PR template. If they disagree, choose a title that satisfies the enforced CI rule first and is as close as possible to the style guide, then explain the choice in the PR body when useful.
7. If automatic PR creation fails, generate a manual handoff using `scripts/generate_pr_report.py`.

## Review-Learning Checklist

Use this short checklist after fixing review feedback or CI failures, especially when several similar PRs are being created:

1. Compare every automated review comment's commit SHA with the current PR head. Treat comments on old commits as stale only after the current diff and CI prove the issue is fixed.
2. For Python changes, run at least `python -m py_compile <changed files>` and the narrowest available linter for touched files, such as `ruff check <file>`, before relying on CI.
3. For shell changes, run `bash -n <script>` and normalize line endings when working from Windows before trusting shell syntax results.
4. For PR title failures, update the title and trigger a new check run by pushing a real or metadata-only signed commit if the repository does not allow reruns.
5. When a fix removes code, scan the nearby block for now-unused variables, comments, imports, and tests. Run a focused search or linter for the touched file before pushing.
6. Update the PR body after review fixes so it describes the final implementation, not the first attempt. Preserve repository-managed footer lines such as version metadata.
7. When changing helper scripts that claim to run on "changed files", verify the tool's default scope. For example, bare `pre-commit run` covers staged files, not unstaged or untracked files. If the intended scope is all local changes, explicitly gather staged, unstaged, and untracked files, exclude deleted files, pass them with `--files`, and simulate the argument list in a temporary git repo when practical.
8. When adding timeouts or fallbacks around hardware/environment probes, distinguish CPU/mock jobs from hardware-routed jobs. Do not silently replace real hardware dependencies with mocks when the selected test path or runner expects hardware; fail loudly with a clear error instead.
9. If CI reports "pre-commit hook(s) made changes", apply or replicate the formatter diff locally before pushing again. Syntax checks such as `bash -n` or `py_compile` are not enough for files governed by auto-format hooks.

## Output Requirements

End issue-fix work with:

- Triage decision and root cause.
- Changed files and fix summary.
- Tests added or changed.
- Validation commands and results.
- Skipped checks or environment limits.
- PR URL or HTML handoff path.

End CI-fix work with:

- Failed checks and classification.
- Root cause for each relevant failure.
- Fix summary and pushed branch.
- Validation commands and results.
- Remaining maintainer label, hardware, or CI-only risk.
