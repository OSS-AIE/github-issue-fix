# PR Quality Gate

Use this gate before opening or updating an upstream PR.

## Project Standards

Inspect project rules before editing or submitting:

- `AGENTS.md` or equivalent agent instructions.
- `CONTRIBUTING*`, `DEVELOPMENT*`, `docs/contributing*`.
- `.github/PULL_REQUEST_TEMPLATE*`, `.github/ISSUE_TEMPLATE*`.
- `.github/workflows/` for CI names and required jobs.
- `.pre-commit-config.yaml`, `pyproject.toml`, `tox.ini`, `noxfile.py`, `package.json`, `Cargo.toml`, `go.mod`, or equivalent tooling files.

Prefer official docs or repository files over memory. If standards may have changed, verify them.

## Review-Risk Self-Check

Before PR creation, check for common review failures:

- Empty or whitespace-only user-facing error messages should fall back to a safe default.
- Optional config/model metadata should handle `None` where generic wrappers, test doubles, or fallback paths can reach it.
- Fake/meta/custom-op implementations should avoid mutating tracing or forward-context state while computing shapes.
- Use normalized runtime attributes when existing code already derives them from raw config.
- Streaming or incremental APIs must emit only the new delta unless the API contract explicitly expects accumulated content.
- Parsers, tool-call builders, and streaming adapters must flush buffered final content and preserve partial state across chunks.
- Backend-specific behavior should stay scoped to the backend or feature gate that needs it.
- Validation should cover both global configuration and per-request overrides when both can reach the changed path.
- Keep API-compatible behavior unless the issue explicitly requires a behavior change.
- Avoid broad refactors, unrelated formatting, and drive-by cleanup.
- Avoid speculative AI-generated PRs that only rephrase code, add brittle checks, or solve a broader problem than the issue describes.
- Add a regression test for every review-risk fix that can be tested without special hardware.

## Local Validation

Run the narrowest reliable commands first:

```bash
git diff --check
python -m ruff format --check <changed_python_files>
python -m ruff check <changed_python_files>
python -m py_compile <changed_python_files>
pytest -q <targeted_test_file_or_node>
```

Adapt commands to the project stack:

- Python: pytest, ruff, mypy, pyright, tox, nox, pre-commit.
- JavaScript/TypeScript: npm/pnpm/yarn test, lint, typecheck, build.
- Rust: cargo fmt --check, clippy, test.
- Go: gofmt, go test, go vet.
- Docs: docs build or markdown lint when docs change.

If full checks cannot run because of OS, missing hardware, network, or dependency limits, record the exact blocker and run static or syntax checks.

## PR Metadata

Use the project's PR template. Include:

- Issue link with `Fixes #...` when appropriate.
- Root cause and minimal fix summary.
- Test plan and test result.
- Hardware/CI-only limitations.
- AI-assistance disclosure when material changes were AI-assisted and the project asks for it.
- DCO sign-off or required trailers.

## New-Contributor CI Gates

Some projects do not run full CI for new contributors until a maintainer trusts the PR.

Recognize this as a permission gate when logs mention:

- a required `ready`, `verified`, or similar label,
- maintainer approval before running workflows,
- author must have a minimum number of merged PRs,
- forked PR workflows are skipped for security.

Do not repeatedly push code for a label gate. After local checks and review-risk fixes are complete, add a concise maintainer-facing comment asking for the appropriate label or approval.

## Review Feedback Gate

Before declaring CI fixed, inspect human and automated review feedback:

- Inline code comments on outdated commits may already be fixed. Compare each comment's commit SHA with the current PR head before acting.
- Treat security, correctness, race, resource leak, compatibility, and test-coverage comments as blocking unless clearly false positive.
- Do not satisfy a review by only changing prose when the comment describes behavior.
- When a comment is fixed, mention the exact validation command in the PR update or reply.
- When a comment is false positive, explain why using project facts and code references, not opinion.

## External Status Gate

Some required statuses are owned by services outside GitHub Actions.

Classify external statuses separately from code failures when they are:

- documentation hosting previews,
- merge queues or automerge bots,
- CLA/DCO/signature checks,
- trusted-contributor or organization-member checks,
- hardware lab or nightly queues.

Fix repository-controlled failures locally. For gates that require credentials, labels, membership, or maintainer action, stop after local validation and leave a concise handoff.
