## Jujutsu Operating Rules

This repository uses Jujutsu (`jj`) for version control.
Always work using Jujutsu's concepts and commands rather than Git's.
For detailed procedures, see `.agents/skills/jujutsu/SKILL.md`.

### Critical Rules

- Using raw `git` commands is **generally prohibited**
  - Exceptions: `jj git ...` and the `gh` CLI
- Do not run `jj split` / `jj resolve` / `jj diffedit` / `jj arrange` (they are interactive commands)
- Even for read-only purposes, do not use `git log` or similar; use `jj log` and the like instead
- Always pass `--git` to `jj diff` / `jj show` / `jj log -p`
- Immediately after any modifying operation such as `jj rebase` / `jj new` / `jj squash`, always run `jj status` to check for conflicts
- Unless instructed otherwise, the base for a PR is `main`
- Do not squash stacked changes

### Terminology

Think in Jujutsu's terms, not Git's.

- commit (the unit of work) → change
- branch → bookmark
- HEAD → `@` (the working copy)
- There is no concept of staging / unstaged / uncommitted
- `git add` is unnecessary
- `git commit --amend` is unnecessary
- When you need the equivalent of stash, use `jj new` instead

### Principles for Starting Work

**Before you begin editing code, always perform the following steps:**

1. Check the current change with `jj log --ignore-working-copy -r @`.
2. **If the current change is empty** (it has NO description AND NO diff) → **reuse it**. Set its description with `jj describe -m "<description>"` and do your work in this change. **Do NOT create a new change in this case.** Note that a change created by `jj new` is itself empty, so it also falls under this rule. Setting a description with `jj describe` does NOT make the change non-empty; keep working in the same change.
3. Otherwise (the current change already has a diff) → create a new change with `jj new -m "<description>"`.
4. Write the description in Conventional Commits format.

### Basic Inspection Commands

Prefer the following commands as needed:

```bash
jj status
jj log --ignore-working-copy
jj diff --git --ignore-working-copy
jj evolog --ignore-working-copy
jj op log --ignore-working-copy
```

**Note:** Except when the agent is inspecting files it has just modified itself, it is recommended to add `--ignore-working-copy` to purely read-only operations.

## Principles for Conflicts

In Jujutsu, an operation is not interrupted even when a conflict occurs.
For that reason, you must not move on without checking `jj status` after a modifying operation.

If there is a conflict, resolve it by editing the files directly, then verify again with `jj status`.
No operation equivalent to `git add` is needed.

## Principles for Remotes and PRs

- For syncing with remotes, use `jj git fetch` / `jj push -b <bookmark-name>`
(Note: `jj push` is an alias command that internally runs `jj git push`)
- To create a PR, use `gh pr create --base main --head <bookmark-name> --title "<type>: <summary>" --body "<body>"`. The PR title MUST follow Conventional Commits (e.g. `feat: ...`, `fix: ...`); always pass it explicitly with `--title` rather than relying on the title auto-derived by `gh`.
- Unlike a branch, a bookmark does not move automatically; operate on it explicitly as needed

## Additional Notes

When performing any detailed Jujutsu operation in this project, you must consult the `jujutsu` Skill.
In situations that require a Jujutsu-specific judgment, you must not proceed before consulting the `jujutsu` Skill.

Examples of such cases:
- revset
- Resolving conflicts
- Bookmark operations
- rebase / squash / split / restore / abandon / undo / op restore
- Handling errors such as stale / immutable
- fetch / push / creating PRs
