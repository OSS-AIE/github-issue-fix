# GitHub CLI Workflow

Use `gh` for issue/PR metadata, CI logs, fork setup, and PR creation.

## Preflight

```bash
git status --short
git branch --show-current
git remote -v
gh auth status
gh repo view <owner>/<repo> --json nameWithOwner,defaultBranchRef,viewerPermission
```

Do not overwrite unrelated local changes. Use a fresh branch or worktree for each independent fix.

## Issue Metadata

```bash
gh issue view <number-or-url> --repo <owner>/<repo> \
  --json number,title,state,author,labels,assignees,body,comments,url

gh pr list --repo <owner>/<repo> --state all --search "<issue-number-or-title>" \
  --json number,title,state,url,author,labels,isDraft,mergedAt,body
```

Skip signals:

- Closed, duplicate, or not planned.
- Maintainer says they are working on it.
- Existing PR clearly resolves it.
- Missing data prevents a code-level hypothesis.

Candidate signals:

- Narrow bug and visible failing code path.
- No active owner.
- Test or static validation is possible.
- Hardware-only reproduction but small deterministic code change can be reviewed and CI-tested.

## Branch and PR

```bash
git fetch <upstream-remote>
git switch -c codex/issue-<number>-<slug> <upstream-remote>/<default-branch>
git add <changed_files>
git commit -s -m "[Bugfix] <summary>"
git push -u <fork-remote> codex/issue-<number>-<slug>
gh pr create --repo <owner>/<repo> --base <default-branch> \
  --head <fork-owner>:codex/issue-<number>-<slug> \
  --title "<project-style title>" \
  --body-file <body-file>
```

Use one branch and PR per independent root cause.

## CI Logs

```bash
gh pr checks <pr-number> --repo <owner>/<repo> \
  --json name,state,bucket,link,workflow,description,startedAt,completedAt

gh run view <run-id> --repo <owner>/<repo> --job <job-id> --log
```

If `gh pr checks` links to a failed job, open the job log before editing. Classify the failure using `references/pr-quality-gate.md`.

## Maintainer Gate Comment

When CI is blocked by a permission label:

```text
I pushed a follow-up addressing review feedback and updated the PR description with validation details. Local changed-file checks pass. The remaining check appears to be a new-contributor gate requiring a maintainer label/approval that I cannot add from this fork. If the fix looks appropriate, could a maintainer please trigger full CI?
```
