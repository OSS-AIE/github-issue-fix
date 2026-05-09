---
name: github-issue-fix
description: Fix upstream GitHub issues and pull request CI failures with GitHub CLI. Use when Codex is asked to fix one or more GitHub issues, triage open issues, prepare upstream pull requests, repair failing PR checks, handle maintainer review feedback, follow project contribution rules, or generate a manual PR handoff when automatic PR creation is unavailable.
---

# GitHub Issue Fix

## Core Workflow

Use this skill for open-source upstream issue repair, PR creation, and PR CI repair.

1. Preserve local work. Check `git status --short`, current branch, remotes, and authentication before editing.
2. Read project instructions first: `AGENTS.md`, `CONTRIBUTING*`, `.github/PULL_REQUEST_TEMPLATE*`, `.github/workflows/`, pre-commit configs, test configs, and language/tooling files.
3. Fetch issue or PR metadata with `gh`: title, body, labels, assignees, linked PRs, comments, reviews, checks, and timeline signals.
4. Classify before coding:
   - `skip`: closed, duplicate, maintainer-owned, already fixed by a PR, or explicitly not wanted.
   - `needs-info`: insufficient reproduction, unclear expected behavior, private artifacts required, or likely design/RFC work.
   - `candidate`: narrow root cause, no active owner or existing fix, and at least one local/static/test/CI validation path.
5. Implement the smallest mergeable fix. Avoid unrelated refactors, formatting churn, broad architecture changes, or test weakening.
6. Add focused regression tests where practical. If hardware or large-model validation is required, still add a deterministic unit/meta/static check when possible.
7. Run the PR Quality Gate before committing.
8. Commit with project-required sign-off or trailers. For DCO projects, use `git commit -s`.
9. Push to a fork and create/update one PR per independent fix. Use a local HTML handoff if PR creation cannot be automated.

Read `references/pr-quality-gate.md` before opening or updating a PR. Read `references/gh-workflow.md` before using `gh` for triage, fork setup, or PR updates.

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
2. Also inspect PR comments and automated reviews; CI may be blocked by a bot comment rather than code.
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
7. Re-run the closest local equivalent, push fixes, and re-check PR status.

## Fork and PR Rules

1. Use `gh auth status` and `git remote -v` to confirm access.
2. Prefer a fork remote owned by the user or organization. Create it with `gh repo fork` only when appropriate.
3. Start from the upstream default branch unless updating an existing PR branch.
4. Keep commits signed according to project rules.
5. Use project title prefixes and PR template sections.
6. If automatic PR creation fails, generate a manual handoff using `scripts/generate_pr_report.py`.

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
