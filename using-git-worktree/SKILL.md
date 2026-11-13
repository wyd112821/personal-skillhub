---
name: using-git-worktree
description: "Prepare an isolated Git worktree on explicit request. Use only when the user manually invokes `$using-git-worktree` or explicitly asks to use this named skill; never trigger it automatically for ordinary coding, bug-fix, refactor, or implementation tasks."
---

# Using Git Worktree

Keep the main checkout stable and give each development task an independent directory.

## 1. Detect the current workspace

Before editing any repository file, inspect:

```bash
git rev-parse --show-toplevel
git rev-parse --git-dir
git rev-parse --git-common-dir
git branch --show-current
git status --short
```

Resolve the Git paths before comparing them. If the Git directory differs from the common directory, the current checkout is already a linked worktree. Report its path and branch, then continue there without creating another worktree or asking again.

If the task does not modify repository files, do not offer a worktree.

## 2. Return the decision to the user

In a normal checkout, report the current branch, repository path, and whether the working tree is clean. Ask one closed question:

```text
当前位于 <branch>（<path>），工作区<干净/有未提交修改>。是否为本次任务创建独立 worktree？
```

Wait for the answer. If the user declines, continue in the current checkout. Never create a worktree merely because the task looks large.

## 3. Create the worktree after consent

Use a user-supplied branch name. Otherwise derive a short kebab-case name from the task and state it before creation.

From the repository root:

1. Ensure `.worktrees/` is ignored by `.gitignore` or `.git/info/exclude`; prefer the local exclude file when no project convention exists.
2. Run `git worktree add ".worktrees/<branch>" -b "<branch>"`.
3. Change the working directory to the new worktree and perform all subsequent work there.

If the branch or path already exists, report the collision and ask for a different name. If creation fails because of permissions, request elevated execution when the environment supports it. Otherwise stop and report the failure. Do not silently edit the original checkout.

## 4. Initialize local environment files

Inside the new worktree:

- Copy `.env.example` to `.env` only when the example exists and `.env` does not.
- For a Python project, create `.venv` and install dependencies using the repository's documented or locked package-manager workflow. If none exists, use `python3 -m venv .venv`, then install from `requirements.txt` when present.
- Report setup failures and stop before implementation if the project cannot run reliably.

Finish by reporting the worktree path and branch, then continue the original task from that directory.
