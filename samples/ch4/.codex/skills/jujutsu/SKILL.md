---
name: jujutsu
description: This Skill collects detailed procedures for AI agents to use Jujutsu (`jj`) as the version control system safely and consistently in this project. The standing rules live in `AGENTS.md`; this Skill records the concrete operating procedures and decision criteria.
---

# Jujutsu Operations Skill

## When to Use This Skill

- You want to check the current working state or history
- You want to start a new change
- You want to perform a rebase / squash / split / restore
- You want to resolve a conflict
- You want to create, move, or delete a bookmark
- You want to specify a target with a revset
- You want to create a PR
- You want to handle errors such as stale / immutable

## Prerequisites

This project uses Jujutsu as its VCS.

- Using raw `git` commands is generally prohibited
- The exceptions are `jj git ...` and the `gh` CLI
- Do not use read-only commands like `git log` either; replace them with `jj` commands

## Terminology Mapping

Do not bring Git's terms over as-is.

| Git term               | In Jujutsu                              |
| ---------------------- | --------------------------------------- |
| commit (unit of work)  | change                                  |
| branch                 | bookmark                                |
| HEAD                   | `@` (the working copy)                  |
| staging                | No such concept                         |
| unstaged / uncommitted | No such concept                         |
| stash                  | Basically unnecessary; use `jj new` if needed |
| `git add`              | Unnecessary                             |
| `git commit --amend`   | Unnecessary                             |

### Key Concepts

1. The moment you save a file, your edits are part of the current change
2. A change is a unit of work, and a revision is a snapshot of it
3. When you change a parent, descendant changes are rebased automatically
4. A conflict is recorded as a first-class state
5. Every operation is kept in the operation log, so you can go back with `jj undo` or `jj op restore` when needed
6. **Automatic formatting (`jj fix`)**: This project has `jj fix` set up. Because it is registered as a Stop hook, it runs when a task finishes and retroactively reformats the code across the mutable changes. Even if `jj diff` shows unintended formatting changes, accept them as long as they conform to the project's rules.

## Basic Policy for diff and log

### diff

Always add `--git` to `jj diff` / `jj show` / `jj log -p`.

```bash
jj diff --git
jj diff --git -r @-
jj show --git
jj log -p --git
```

Prohibited examples:

```bash
jj diff
jj diff -r @-
jj show
jj log -p
```

### Suppressing Snapshots in Read-Only Operations

Every time a command runs, jj automatically creates a snapshot of the working copy. If the AI checks state carelessly, there is a risk that operations in another process cause a conflict in the operation log. Therefore, for purely read-only operations performed **while no files have been modified**, add `--ignore-working-copy`.

```bash
# ✅ Read-only: when investigating without modifying any files
jj log --ignore-working-copy
jj log --ignore-working-copy -r 'main..@'
jj diff --git --ignore-working-copy -r @-
jj bookmark list --ignore-working-copy

# ❌ When you must NOT add it: checking state right after modifying files
#    (because the latest snapshot needs to be reflected)
jj status          # do not add --ignore-working-copy right after a change
jj diff --git      # do not add --ignore-working-copy right after a change

# ℹ️ When you only want to take a snapshot
jj util snapshot
```

**Rule of thumb:** If you have just created, edited, or deleted a file, do not add `--ignore-working-copy`. In all other cases—when you are "merely checking a known state"—add it.

### log

For ordinary checks, use `jj log` with the graph.
Use `--no-graph` and `-T` only when you want to extract information programmatically.

```bash
jj log --ignore-working-copy
jj log --ignore-working-copy --no-graph -T 'change_id.short() ++ " " ++ description.first_line() ++ "\n"'
jj log --ignore-working-copy --no-graph -T 'commit_id.short() ++ " " ++ bookmarks ++ "\n"' -r 'bookmarks()'
```

## Custom Commands and Dependencies

The following alias is registered in Jujutsu's per-repository configuration.

```toml
[aliases]
push = [
  "util", "exec", "--",
  "bash", "-c",
  "jj fix && mise run pre-push && jj git push \"$@\"",
  ""
]
```

* **Using `jj push`**: When pushing changes to the remote, always use this alias. Internally it runs `jj fix` and the `pre-push` task via `mise`.
* **Dependency**: Running `jj push` requires `mise`.

## Basic Workflow

### 1. Checking State

```bash
jj status
jj log --ignore-working-copy
jj diff --git
jj diff --git --ignore-working-copy -r @-
jj evolog --ignore-working-copy
jj op log --ignore-working-copy
```

Uses:

- `jj status`: Check the working-copy state
- `jj log --ignore-working-copy`: Grasp the history and stack structure
- `jj diff --git`: Check the current diff (right after a change)
- `jj diff --git --ignore-working-copy -r @-`: Check the diff of the previous change (read-only)
- `jj evolog --ignore-working-copy`: Check the evolution of the current change
- `jj op log --ignore-working-copy`: Check the operation history

### 2. Starting Work

Look at the state of the current `@` and decide whether to use `describe` or `new`.

Procedure:

1. Check the current change with `jj log --ignore-working-copy -r @`.
2. If the description is empty and the diff is also empty, start working using that `@`.
3. In that case, use `jj describe -m "<description>"`.
4. If work is already in progress, or there is a description or diff, use `jj new -m "<description>"`.
5. Write the description in Conventional Commits format.

Example:

```bash
jj log --ignore-working-copy -r @
jj describe -m "feat: add search form"
```

Or:

```bash
jj new -m "fix: handle empty input"
```

### 3. Checking for Conflicts After a Modifying Operation

Immediately after a modifying operation such as `jj rebase`, `jj new`, or `jj squash`, always run `jj status`.

```bash
jj rebase -s @ -d main
jj status
```

`jj` does not stop an operation even when a conflict occurs.
So if you move on without looking at `jj status`, you risk continuing to work while still carrying a conflict.

Example of a sign of a conflict:

```text
The change has 2 conflicts:
  src/main.rs    2-sided conflict
```

### 4. Bookmark Operations

Unlike Git branches, bookmarks do not move automatically. Operate on them explicitly when needed.

```bash
jj bookmark create <name> -r @
jj bookmark move <name> -t @
jj bookmark list --ignore-working-copy
jj bookmark delete <name>
```

Uses:

- Create a new bookmark
- Move an existing bookmark to the current change
- Check the list
- Delete an unneeded bookmark

### 5. Splitting and Restoring Changes

#### Splitting

When a change grows too large, split it.

```bash
jj split -r <revision>
```

#### Restoring

Use `restore` when you want to recover part of the state from another revision.

```bash
jj restore --from <revision> <path>
```

#### Restoring to a Past Version

You can also look at `evolog` and go back to a past state of the same change.

```bash
jj evolog --ignore-working-copy -r <change-id>
jj restore --from <change-id>/1 --to <change-id>
```

Notes:

- `<change-id>/0` is the latest version
- `<change-id>/1` is the previous version
- Check with `jj evolog` before running it

### 6. Amending and Undoing History

```bash
jj undo
jj op restore <operation-id>
jj abandon @
```

Uses:

- `jj undo`: Undo the most recent operation
- `jj op restore <operation-id>`: Return to any operation point
- `jj abandon @`: Discard the current change

You may operate without fear of mistakes, but when your intent is unclear, check with `jj op log` before going back.

## Resolving Conflicts

In Jujutsu, even when a conflict occurs, the change is recorded in that state.
Resolve it with the following procedure.

1. Identify the conflicting files with `jj status`.
2. Open the file and directly edit the sections containing conflict markers.
3. Arrange the content correctly and save.
4. Confirm with `jj status` that the conflict is gone.

Format of the conflict markers:

```text
<<<<<<<
%%%%%%%
-removed line
+added line
+++++++
content from the other side
>>>>>>>
```

Meaning:

- The `%%%%%%%` block: the diff from the base
- The `+++++++` block: the other side's content itself

Notes:

- No `git add` like in Git is needed
- Once you save, `jj` automatically detects the resolution

## Syncing with the Remote

```bash
jj git fetch
jj push -b <bookmark-name>
```

- `jj git fetch`: Fetch the latest state from the remote
- `jj push -b <bookmark-name>`: Push the bookmark
(Note: `jj push` is an alias command that internally runs `jj git push`)

## revset Cheat Sheet

### Basic Syntax

| Syntax              | Meaning                                  |
| ------------------- | ---------------------------------------- |
| `@`                 | The current change                       |
| `@-`                | The previous change                      |
| `@--`               | The change two before                    |
| `<bookmark>`        | Bookmark name                            |
| `<bookmark>@origin` | A remote bookmark                        |
| `main..@`           | The set of changes from `main` to the current one |
| `empty()`           | Empty changes                            |
| `<change-id>/n`     | The version n generations back of the same change |

### Handy Patterns

| Pattern                             | Purpose                            |
| ----------------------------------- | ---------------------------------- |
| `trunk()..@`                        | The entire stack from main to the current change |
| `mine() & mutable() & ~empty()`     | List of your in-progress changes   |
| `conflict()`                        | Changes that contain conflicts     |
| `bookmarks() & ~remote_bookmarks()` | Bookmarks not yet pushed           |

Examples:

```bash
jj log -r 'trunk()..@'
jj log -r 'conflict()'
jj log -r 'mine() & mutable() & ~empty()'
```

## PR Creation Workflow

### Basic Rules

- Unless instructed otherwise, the base for a PR is `main`
- Do not squash multiple changes
- You do not need to confirm the following every time

  * Whether to include the change → it is always in the current change
  * Whether to base it on `main` → `main` unless stated otherwise
  * Whether to squash → no

### Steps

```bash
jj git fetch
jj log --ignore-working-copy
jj bookmark list --all --ignore-working-copy
jj bookmark track <bookmark-name>@origin
jj push -b <bookmark-name>
gh pr create --base main --head <bookmark-name>
```

Notes:

- First check whether the bookmark exists
- If it is untracked, `track` it
- Judging from `jj log` or `bookmark list --all`, if the local and remote bookmarks point to the same revision, it has already been pushed, so no re-push is needed

## Troubleshooting

### "The working copy is stale"

This happens when a human and an AI touch the same repository in parallel, or when an external tool rewrites files.

Fix:

```bash
jj workspace update-stale
jj status
```

### "Commit XXXX is immutable"

The target of `describe` or `squash` is among immutable revisions, such as `main@origin` or its ancestors.

Fix:

1. Check that the target specified with `-r` is correct.
2. Redo the operation against a mutable change.
3. If needed, check your current position with `jj log`.

## Final Notes

1. `jj` has no concept of staging.
2. Changes in the working directory are tracked automatically.
3. Running `jj status` after a modifying operation is mandatory.
4. It can coexist with `.git`, but perform operations through `jj`.
5. Do not use `git` commands as a general rule, because they risk corrupting state.
6. When in doubt, look at `jj log` / `jj status` / `jj op log` before acting.
