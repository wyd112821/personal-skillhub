---
name: finishing-branch
description: "Finish completed work on a Git feature branch with an explicit local merge and cleanup choice. Use when implementation is complete and the user wants to wrap up, merge, or remove a development branch or linked worktree: inspect branch and base state, summarize changes, present exactly two merge options, execute the selected option, and require separate consent only when an abnormal or forceful deletion would be necessary."
---

# Finishing Branch

Report state first. Never merge or delete before the user selects an option.

## 1. Inspect and summarize

Read the current state:

```bash
git status --short --branch
git branch --show-current
git worktree list --porcelain
git diff --stat
```

Determine the base branch from evidence in this order:

1. Use an explicit base named by the user, repository instructions, or the recorded task/worktree creation context.
2. For a linked worktree, treat branches checked out in the primary or sibling worktrees as base candidates. Compare each plausible candidate with the feature branch:

   ```bash
   git merge-base <candidate> <feature-branch>
   git rev-list --left-right --count <candidate>...<feature-branch>
   ```

   Prefer the branch the feature was actually forked from. A sibling branch that shared the feature's starting commit and has since advanced independently is stronger evidence than the remote default branch.
3. Only when task context and local worktree topology provide no plausible base, use the remote default branch; then fall back to an existing `main` or `master`.

The remote default branch is a fallback, not proof of the feature branch's parent. Never select it solely because it exists when a different local branch plausibly owns the worktree's development context. Commit ancestry and divergence are supporting evidence, not definitive intent; if two candidates remain plausible, report the evidence and ask only for the base branch before continuing.

Summarize:

- Current branch and worktree path
- Base branch and the evidence used to identify it
- Changed files and the purpose of the changes
- Verification already run and its result

If the feature worktree has uncommitted changes, report them and stop. Ask the user to commit or explicitly choose another disposition before offering merge options.

## 2. Present exactly two options

Use this prompt with the detected base branch:

```text
当前分支收尾，你想怎么处理？
1. 合并回 <base-branch>，保留当前 worktree 和分支
2. 合并回 <base-branch>，并删除当前 worktree 和分支
请选择一个选项。
```

Wait for the selection. Do not add PR, discard, or open-ended alternatives unless the user asks for them.

## 3. Merge after selection

Locate the checkout that has the base branch checked out. Before merging, require that checkout to be clean.

Merge the feature branch into the base branch there. Do not pull, rebase, force-push, or resolve conflicts automatically unless the user requested it. If the merge conflicts or verification fails, report the state and stop without cleanup.

After a successful merge:

- Option 1: report success and leave the feature branch and worktree untouched.
- Option 2: move outside the feature worktree, run `git worktree remove "<worktree-path>"`, then run `git branch -d "<feature-branch>"`.

The selection of option 2 authorizes normal worktree and branch deletion; do not ask twice. If cleanup would require `--force`, `-D`, deletion of uncommitted files, or removal of a worktree not under the repository's `.worktrees/` directory, stop and request explicit confirmation for that abnormal risk.

Finish by reporting the merge result and whether the worktree and branch were retained or removed.
