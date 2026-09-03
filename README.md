# github-issue-fix

Codex skill for fixing upstream GitHub issues and pull request failures with GitHub CLI.

Use it when Codex needs to:

- triage one or more upstream GitHub issues,
- repair pull request CI failures,
- address PR review comments,
- resolve merge conflicts or rebase existing PRs,
- prepare reviewer-friendly PR descriptions and handoff notes.

Install with Codex skill installer:

```bash
python3 ~/.codex/skills/.system/skill-installer/scripts/install-skill-from-github.py \
  --repo OSS-AIE/github-issue-fix \
  --path . \
  --name github-issue-fix
```

The skill itself is in [SKILL.md](SKILL.md).

For build-system usability, dependency, performance, reproducibility, or
supply-chain work, install the shared build-engineering skill as well:

```bash
python3 ~/.codex/skills/.system/skill-installer/scripts/install-skill-from-github.py \
  --repo OSS-AIE/github-issue-fix \
  --path skills/refactor-build-system \
  --name refactor-build-system
```

`github-issue-fix` keeps ownership of GitHub issue, PR, review, and Actions
operations. `refactor-build-system` supplies the platform-neutral analysis and
minimal-change contract. The GitCode adapter uses the same shared skill while
retaining its own CLI, PR, and Pipeline workflow.
