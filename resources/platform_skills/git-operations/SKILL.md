---
name: git-operations
description: >-
  Git safety and operational guidance for Scion agents. Covers working tree
  resets, rebase pitfalls, and general git behavior that agents should know
  regardless of workspace mode. For workspace-mode-specific git rules, see
  the git-sandbox skill.
---

# Git Operations

General git safety rules for Scion agents. These apply regardless of
workspace sharing mode — for mode-specific guidance (worktree isolation,
shared-plain index hazards, clone-per-agent remotes), see the `git-sandbox`
skill.

## Working Tree Reset Safety

When cleaning a working tree, **use `git clean -fd`, not `git clean -fdx`**.

- `git clean -fd` removes untracked files but **respects `.gitignore`** — `.scion/`, agent state, and ignored directories survive.
- `git clean -fdx` deliberately defeats `.gitignore` and **deletes everything** not tracked by git, including `.scion/`, `downloads/`, and any local state.

The `-x` flag is not a stronger clean — it is a different operation. Default to `-fd`; use `-x` only with a specific reason and after verifying nothing irreplaceable is in an ignored directory.

**`downloads/` is an inbox, not storage.** Files downloaded into your container are visible only inside it and invisible to every other agent. Move anything worth keeping to `/scion-volumes/scratchpad/` (shared across all agents) promptly. A `downloads/` file that has not been drained to the scratchpad is one command from gone.

## Rebase After Deletion

**When a PR deletes a file, a rebase will tell you about edits to that file and will not tell you about new references to it.**

A concurrent edit to the deleted file raises a modify/delete conflict and halts the rebase — that case is handled, because it demands a decision. A concurrent edit that adds a *reference* to the deleted file **from a different file** produces no conflict at all: git sees two disjoint changes, reports "Successfully rebased," and leaves a dangling reference.

After rebasing a deletion, **grep for the deleted name**. Any inbound-reference count taken before the rebase is stale by construction.

## Shallow Clones

Agent clones may be shallow (`git rev-parse --is-shallow-repository` prints `true`). History-dependent commands then give wrong or confusing results.

- Before `merge-base`, `merge-tree`, `log` archaeology, or diffing against an old SHA, deepen: `git fetch --unshallow`, or `git fetch origin main --deepen=200` for a bounded fetch.
- To get a pull request's head: `git fetch origin pull/<n>/head` (then inspect `FETCH_HEAD`).

## Comparing Against the Base Branch

Don't swap files in your working tree with `git stash` or `git checkout` to see the base version; it is easy to lose or mix up changes.

- Read a single file: `git show origin/main:path/to/file > "$scratch/file.base"`.
- For builds, tests or before/after measurements, use a throwaway worktree outside the repo: `git worktree add "$scratch/base" origin/main`, and `git worktree remove "$scratch/base"` when done. Take baselines there *before* you start editing.

## Branch and Staging Discipline

- **Stay on the branch you were assigned** (see the `gitStatus` context or `git branch --show-current`). Do not create or switch to another branch unless asked.
- **Stage explicit paths** (`git add path/a path/b`), not `git add -A` or `git add .`. Check `git status --short` first: tools leave untracked backup, report and generated files around. Keep backups and scratch output outside tracked directories (e.g. in the scratchpad).
