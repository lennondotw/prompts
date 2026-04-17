---
name: commit-and-merge
description: Commit, push, open/update a PR, merge (with automerge when safe), and monitor until merged and post-merge CI is green. For GitHub repos via the `gh` CLI. Use when asked to commit, push, open a PR, enable automerge, or babysit a PR to green — including hands-off merging that self-heals on CI failures.
---

# Commit and Merge

Ship changes end-to-end on GitHub via `gh`. Stay attached until merged — automerge is not fire-and-forget.

## Workflow

1. **Review and batch**
   - `git status` + `git diff` (staged and unstaged); read the last ~10 commits for style.
   - Split diffs over ~400 meaningful lines or with mixed concerns into focused commits/PRs.

2. **Commit**
   - Conventional `type(scope): description`. Format touched files per the repo's toolchain first. No AI attribution.

3. **Rebase and push**
   - `git fetch origin && git rebase origin/<base>` before pushing. Stale base wastes CI and fails late at merge.
   - `git push -u origin HEAD` on first push.

4. **Open/update the PR**
   - **Target branch**: follow `AGENTS.md` / `CONTRIBUTING.md`; if silent, use the base of recent merged PRs (`gh pr list --state merged --limit 5 --json baseRefName`). `defaultBranchRef` is often release-only — don't assume it.
   - If scope evolved, update title and description. Stale titles are a bug.

5. **Merge (automerge when possible)**
   - Strategy: match repo policy (branch protection or recent merged PRs). Otherwise try `--rebase` → `--squash` → `--merge`.
   - `gh pr merge --auto --rebase` (or `--squash` / `--merge`) needs auto-merge enabled on the repo. Without required status checks in branch protection it merges immediately without waiting for CI — fall back to manual: wait for green, then `gh pr merge --rebase` (or the chosen strategy).
   - If base advances and conflicts appear: `gh pr update-branch --rebase <pr>` (or rebase locally and push). `gh pr update-branch` without `--rebase` creates a merge commit on your branch.

6. **Monitor until merged**
   - Prefer streams: `gh pr checks <pr> --watch`, `gh run watch <run-id> --exit-status`.
   - Fallback polling (no run-id yet, watching merge state, or `--watch` unavailable):
     - Estimate ETA from recent runs (`gh run list --workflow <name> --limit 10 --json startedAt,updatedAt,conclusion`); default ~1 min if unknown.
     - First check near ETA (don't poll every 5s for a 5-min job); then exponential backoff, single-sleep capped ~2 min.
     - If total wait exceeds ~2× ETA, stop and investigate (queued runner, misconfigured required check, stuck approval).
   - Done only when `state=MERGED`.

7. **On failure, pick deliberately**
   - `gh run view <run-id> --log-failed`, then choose one:
     (a) fix in place — clear local bug;
     (b) scope down — drop a commit or split the PR;
     (c) revert / close — especially if post-merge CI broke base.
   - Don't default to (a) just because the PR is already open.

8. **Post-merge CI**
   - Watch base-branch workflows (`gh run watch <run-id> --exit-status`). On failure, revert first to keep base green, then follow up in a new PR.

## Guardrails

- Never force-push to protected branches.
- Never `--amend` a pushed commit, never `--no-verify`, unless explicitly requested.
- If the same step fails twice after honest fixes, stop and report — don't thrash.
