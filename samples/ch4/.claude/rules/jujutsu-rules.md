# Jujutsu (jj) Rules for AI Agents

This project uses **Jujutsu (jj)** for version control. AI agents must strictly follow the rules below, and **using `git` commands is generally prohibited** (except for the `jj git` subcommands and the `gh` CLI).

---

## Important Notes for AI Agents

### Terminology Mapping

The meanings of terms differ between Jujutsu and Git. AI agents must rigorously observe the distinctions below.

| Git term                 | Jujutsu term                                  |
| ------------------------ | --------------------------------------------- |
| commit (as a noun)       | change                                        |
| branch                   | bookmark                                       |
| staging                  | (no such concept)                             |
| unstaged / uncommitted   | (no such concept)                             |
| HEAD                     | `@` (the working copy)                        |
| stash                    | (no such concept; use `jj new` instead)       |
| `git add`                | (unnecessary; automatic snapshot)             |
| `git commit --amend`     | (unnecessary; changes to `@` are reflected automatically) |

### Fundamental Differences from Git

1. **There is no such thing as an "unsaved change"**: the moment you save a file, it is automatically included in the current change. There is no need to ask, "Do you want to include this change?"
2. **change and revision**:
   - **change**: A unit of work. It has a unique, immutable **change ID**, while its contents are mutable.
   - **revision**: A snapshot of a change. A new revision is created every time you edit, but the change ID never changes.
3. **Automatic rebase**: When you change a change's parent, its descendant changes are rebased automatically. There is no need to manage a rebase chain by hand.
4. **First-class conflicts**: Even when a conflict occurs, the operation is not interrupted; it is recorded as a change that contains the conflict. You can resolve it later.
5. **Operation log**: Every operation is recorded, and you can return to any point with `jj undo` / `jj op restore`. You may operate without fear of mistakes.
6. **Automatic formatting (`jj fix`)**: This project has `jj fix` set up. Because it is registered as a Stop hook, it runs when a task finishes and retroactively reformats the code across the mutable changes. Even if `jj diff` shows unintended formatting changes, accept them as long as they conform to the project's rules.

### Rules for diff Output

**When running `jj diff`, `jj show`, or `jj log -p`, always add the `--git` flag.**

Specifying this option makes the diff output use the Git format automatically.

```bash
# Correct
jj diff --git
jj diff --git -r @-
jj diff --git --from main --to @

# Prohibited
jj diff           # without --git is prohibited
jj diff -r @-     # without --git is prohibited
```

With a Git-format diff, file additions, deletions, renames, and permission changes are all represented accurately, which greatly improves the accuracy with which an AI agent can analyze the diff.

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

# ℹ️  When you only want to take a snapshot
jj util snapshot
```

**Rule of thumb:** If you have just created, edited, or deleted a file, do not add `--ignore-working-copy`. In all other cases—when you are "merely checking a known state"—add it.

### Choosing the Right Log Output

For ordinary state checks, use `jj log`'s graph display as-is. When you need to extract specific information programmatically, make use of `--no-graph` and `--template` (`-T`).

```bash
# Ordinary check (with graph)
jj log --ignore-working-copy

# Extracting specific information (machine-readable format)
jj log --ignore-working-copy --no-graph -T 'change_id.short() ++ " " ++ description.first_line() ++ "\n"'
jj log --ignore-working-copy --no-graph -T 'commit_id.short() ++ " " ++ bookmarks ++ "\n"' -r 'bookmarks()'
```

---

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

---

## Basic Workflow

### 1. Checking State and Diffs

```bash
jj status                                  # Check working-copy state (run as-is right after a change)
jj log --ignore-working-copy               # Show history as a graph
jj diff --git                              # Diff of the current working copy (right after a change)
jj diff --git --ignore-working-copy -r @-  # Diff of the previous change (read-only)
jj evolog --ignore-working-copy            # Evolution history of the current change
jj op log --ignore-working-copy            # Show the operation log
```

### 2. Starting Work

Adjust your actions according to the state of the current change (`@`). The decision procedure is as follows:

1. Check the current change with `jj log --ignore-working-copy -r @`.
2. **If the current change is empty** (it has NO description AND NO diff) → **reuse it**. Set its description with `jj describe -m "<description>"` and do your work in this change. **Do NOT create a new change in this case.** Note that a change created by `jj new` is itself empty, so it also falls under this rule.
3. Otherwise (the current change already has a description or a diff) → create a new change with `jj new -m "<description>"`.
4. Write the description in Conventional Commits format..

### 3. Checking for Conflicts After a Modifying Operation

**After any modifying operation such as `jj rebase`, `jj new`, or `jj squash`, always check for conflicts with `jj status`.** Because Jujutsu does not interrupt an operation when a conflict occurs, there is a risk of unknowingly continuing your work.

```bash
# Always run this after a modifying operation
jj status

# If the output contains lines like the following, a conflict exists:
#   The change has 2 conflicts:
#     src/main.rs    2-sided conflict
```

If you detect a conflict, resolve it before continuing (see the "Resolving Conflicts" section for the procedure).

### 4. Bookmark Operations

Unlike Git branches, bookmarks must be moved manually.

```bash
jj bookmark create <name> -r @          # Create a new bookmark (-r specifies the target revision)
jj bookmark move <name> -t @            # Move an existing bookmark to the current change
jj bookmark list --ignore-working-copy  # List bookmarks
jj bookmark delete <name>               # Delete a bookmark
```

### 5. Splitting and Restoring Changes

```bash
# Split a change into several (used to fix a change whose scope is too large)
jj split -r <revision>

# Restore a specific file from another revision
jj restore --from <revision> <path>

# Return a change to a previous state (use a past version found via evolog)
jj evolog --ignore-working-copy -r <change-id>    # Check past versions
jj restore --from <change-id>/1 --to <change-id>  # Restore to the previous state
```

> **Note:** In the `<change-id>/n` notation, `xyz/0` refers to the latest version and `xyz/1` to the previous one. Use it after checking the evolution history with `jj evolog`.

### 6. Amending and Undoing History

- **`jj undo`**: If you make a mistake, use it without hesitation to return to the previous state.
- **`jj op restore <operation-id>`**: Return to a specific operation. You can find the operation ID with `jj op log`.
- **`jj abandon @`**: Discard the current change itself.

### 7. Resolving Conflicts

In Jujutsu, even when a conflict occurs the operation is not interrupted; the change is recorded with conflict markers inserted. AI agents should resolve it using the following procedure.

1. Identify the conflicting files with `jj status`.
2. Open the relevant file and **directly edit the sections containing conflict markers, rewriting them into the correct state**.

Jujutsu's conflict markers use a different format from Git's:

```
<<<<<<<
%%%%%%%
-removed line
+added line
+++++++
content from the other side
>>>>>>>
```

- The `%%%%%%%` block: diff format. It expresses the change from the base with `-`/`+`.
- The `+++++++` block: snapshot format. It shows the other side's content as-is.

3. Save the file. Because `jj` automatically detects the resolution, no operation equivalent to `git add` is needed.
4. Confirm with `jj status` that the conflict is gone.

### 8. Syncing with the Remote

```bash
jj git fetch                    # Fetch the latest state from the remote
jj push -b <bookmark-name>      # Push the bookmark to the remote
                                # (Note: `jj push` is an alias command)
```

---

## Revision Specification Syntax (revset)

| Syntax                | Meaning                                              |
| --------------------- | ---------------------------------------------------- |
| `@`                   | The current change                                   |
| `@-`                  | The previous change                                  |
| `@--`                 | The change two before                                |
| `<bookmark>`          | Specify by bookmark name                             |
| `<bookmark>@origin`   | A remote bookmark                                    |
| `main..@`             | The set of all changes from `main` to the current change |
| `empty()`             | Changes with empty contents                          |
| `<change-id>/n`       | The version n generations back of a change (0 is the latest) |

---

## Handy revset Patterns

| Pattern                                     | Purpose                                |
| ------------------------------------------- | -------------------------------------- |
| `trunk()..@`                                | The entire stack from main to the current change |
| `mine() & mutable() & ~empty()`             | List of your in-progress changes       |
| `conflict()`                                | Changes that have conflicts            |
| `bookmarks() & ~remote_bookmarks()`         | Bookmarks not yet pushed               |

```bash
# Examples
jj log --ignore-working-copy -r 'trunk()..@'
jj log --ignore-working-copy -r 'conflict()'
```

---

## PR Creation Workflow

### Basic Rules

1. **The target is always `main`** — Unless instructed otherwise, the target branch for a PR is `main`. No confirmation needed.
2. **No squashing** — Do not combine multiple changes into one. To preserve the traceability of the change history, keep each change as-is.
3. **Things that need no confirmation** — You do not need to ask the user about the following every time:
   - "Should I include this change?" → It is always included.
   - "Should I merge into main?" → main, unless instructed otherwise.
   - "Should I squash?" → No.
4. **The PR title follows Conventional Commits** — The PR title must follow the Conventional Commits format (e.g. `feat: ...`, `fix: ...`). Always pass it explicitly with `--title` rather than relying on the title auto-derived by `gh`.

### PR Creation Steps

```bash
# 1. Fetch the latest state from the remote
jj git fetch

# 2. Check the current state
jj log --ignore-working-copy

# 3. Confirm that the bookmark is set (including the remote)
jj bookmark list --all --ignore-working-copy

# 4. If the bookmark is untracked, start tracking it
jj bookmark track <bookmark-name>@origin

# 5. Push to the remote (only if not yet pushed or if there are updates)
#    Judging from jj log or bookmark list --all, if the local and remote bookmarks
#    point to the same revision, no re-push is needed
jj push -b <bookmark-name>

# 6. Create the PR with the GitHub CLI
#    The PR title MUST follow Conventional Commits (e.g. "feat: ...", "fix: ...").
#    Do not rely on the auto-derived title; pass it explicitly with --title,
#    derived from the description of the lead change of the bookmark.
gh pr create --base main --head <bookmark-name> --title "<type>: <summary>" --body "<body>"
```

---

## Troubleshooting

### "The working copy is stale" Error

This occurs when a human and an AI are working in parallel in the same repository, or when an external tool has modified files. If you see this error, run the following to resync.

```bash
jj workspace update-stale
```

Afterward, confirm with `jj status` that the working copy is in a normal state.

### "Commit XXXX is immutable" Error

If this error appears when you run `describe` or `squash`, the target of the operation is among the protected revisions (`main@origin` and its ancestors). Check that the revision specified with the `-r` option is correct, and redo the operation against a mutable change.

---

## Points to Note

1. **Automatic saving**: jj automatically tracks changes in the working directory. No explicit `add` is needed.
2. **Immutable history**: By default, `trunk()` and its ancestors are immutable. Local mutable changes can be edited freely.
3. **Handling conflicts**: jj can record changes that contain conflicts. You can resolve them later, but **always check with `jj status` after a modifying operation**.
4. **Git compatibility**: It can coexist with a `.git` directory. Use `jj fetch` / `jj push` to integrate with Git remotes.
5. **Transmitting the change ID**: The change ID is also transmitted to the remote as a Git commit header (`change-id`).
6. **glob patterns**: String patterns in revsets and the like are interpreted as globs by default. For partial matches, use the `substring:` prefix.
7. **Restrictions on git commands**: `git` commands are generally prohibited because they risk corrupting state. The `gh` CLI uses `git` internally, but it is allowed. Replace read-only `git` operations (such as `git log`) with `jj log` as well.
