---
name: commit-and-merge
description: Commit staged work, push, enable automerge, and actively monitor until the PR is merged and post-merge CI is green. For GitHub repositories using the `gh` CLI. Use when the user asks to commit, push, open a PR, enable automerge, or babysit a PR to merge; especially when they want hands-off merging that still self-heals on CI failures.
---

# Commit and Merge

Ship changes end-to-end on GitHub via `gh`: commit, rebase, push, open/update the PR, merge (with automerge when possible), and monitor until merged plus post-merge CI green. Stay attached — automerge is not fire-and-forget.

## Workflow

1. **Review and batch**
   - `git status` + `git diff` (staged and unstaged); read the last ~10 commits for style.
   - Split large or mixed-concern diffs into focused commits/PRs (>~400 meaningful lines or multiple concerns → stop and split).

2. **Commit**
   - Conventional `type(scope): description`. Format touched files per the repo's toolchain first. No AI attribution.

3. **Rebase and push**
   - `git fetch origin && git rebase origin/<base>` before pushing. CI on a stale base is wasted wall time and fails late at merge.
   - `git push -u origin HEAD` on first push.

4. **Open/update the PR**
   - **Target branch**: follow `AGENTS.md` / `CONTRIBUTING.md`; if silent, use the base of recent merged PRs (`gh pr list --state merged --limit 5 --json baseRefName`). `defaultBranchRef` is often a release-only branch — don't assume it.
   - If scope evolved during iteration, update the title and description. Stale titles are a bug.

5. **Merge (automerge when possible)**
   - Strategy: match repo policy (branch protection or recent merged PRs). If unspecified, try `--rebase` → `--squash` → `--merge` in that order.
   - `gh pr merge --auto <strategy>` requires auto-merge enabled on the repo. Without required status checks in branch protection it will merge immediately without waiting for CI — if so, fall back to manual: wait for green, then `gh pr merge <strategy>`.
   - If base advances during the wait and a conflict appears, refresh: `gh pr update-branch <pr>` (or rebase locally and push).

6. **Monitor until merged**
   - Prefer streams: `gh pr checks <pr> --watch`, `gh run watch <run-id> --exit-status`.
   - Fallback polling (no run-id yet, watching merge state, or `--watch` unavailable):
     - Estimate expected runtime from recent runs (`gh run list --workflow <name> --limit 10 --json createdAt,updatedAt,conclusion`); default ~1 min if unknown.
     - First check near that ETA — don't burn API quota every 5s while a 5-minute job is still running. After that, exponential backoff with single-sleep capped at ~2 min.
     - If total wait exceeds ~2× ETA, stop polling and investigate (queued runner, misconfigured required check, stuck approval) — don't sleep longer.
   - Done only when `state=MERGED`.

7. **On failure, pick deliberately**
   - `gh run view <run-id> --log-failed` to read the cause, then choose one:
     (a) fix in place — clear local bug;
     (b) scope down — drop a commit or split the PR;
     (c) revert / close — especially if post-merge CI broke the base branch.
   - Don't default to (a) just because the PR is already open.

8. **Post-merge CI**
   - Watch the base-branch workflows (`gh run watch <run-id> --exit-status`). If they fail, revert the merge first to keep the base branch green, then open a follow-up fix PR.

## Guardrails

- Never force-push to protected branches.
- Never `--amend` a pushed commit, never `--no-verify`, unless explicitly requested.
- If the same step fails twice after honest fix attempts, stop and report — don't thrash.
