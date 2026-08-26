---
name: jujutsu
description: Repository-specific procedures for non-trivial Jujutsu operations, including revsets, conflicts, bookmarks, history rewriting or recovery, remote synchronization, and PR creation.
---

# Jujutsu Operations

The standing policy lives in `AGENTS.md`. This skill covers only repository-specific decisions and procedures that are not implied by standard Jujutsu behavior.

## When to Use This Skill

- Writing or evaluating a non-trivial revset
- Resolving conflicts
- Creating, moving, tracking, forgetting, or deleting bookmarks
- Rebasing, squashing, restoring, abandoning, undoing, or restoring an operation
- Splitting a change without a diff editor
- Handling stale or immutable working-copy errors
- Fetching, pushing, or creating a PR

Routine state inspection and starting ordinary work do not require this skill; follow `AGENTS.md` directly.

## Repository Constraints

- Do not run raw `git` commands. `jj git ...` and `gh` are allowed.
- Do not run `jj resolve`, `jj diffedit`, or `jj arrange`; they require interactive interfaces. `jj split` is allowed only with filesets and `-m` — never bare, never `-i`/`--tool`. See "Splitting a Change by File". If a task cannot be completed safely with non-interactive commands, ask the user to perform that part.
- Always add `--git` to `jj diff`, `jj show`, and `jj log -p`.
- After a history-changing operation such as `jj new`, `jj rebase`, `jj squash`, `jj restore`, or `jj abandon`, run `jj status` before continuing.
- Do not squash stacked changes when preparing a PR.

## Automatic Formatting

The Stop hook in `.codex/hooks.json` runs `jj fix` when a task finishes. It can retroactively format files across mutable changes, so accept formatting-only differences when they conform to the repository's formatting rules.

## Snapshot Discipline

Use `--ignore-working-copy` for read-only inspection only when the working copy is already known to be snapshotted. It prevents a new snapshot and can therefore report stale state.

At the start of work, run `jj status` without `--ignore-working-copy`. Likewise, after creating, editing, or deleting a file, inspect the current state without `--ignore-working-copy` so the latest filesystem state is captured.

Typical read-only commands after that snapshot include:

```bash
jj log --ignore-working-copy
jj diff --git --ignore-working-copy -r @-
jj bookmark list --all --ignore-working-copy
jj evolog --ignore-working-copy
jj op log --ignore-working-copy
```

## Repository Push Alias

The repository defines this alias:

```toml
[aliases]
push = [
  "util", "exec", "--",
  "sh", "-c",
  "jj fix && mise run pre-push && jj git push \"$@\"",
  "--"
]
```

Use `jj push` rather than `jj git push` when publishing. It runs `jj fix` and the `pre-push` task first and requires `mise`.

## Splitting a Change by File

```bash
jj split -m "<description for the extracted change>" <path>...
```

Filesets are what keep it out of the diff editor, `-m` out of the text editor. Hunk-level splits need that editor, so hand those to the user.

## Resolving Conflicts

1. Identify the conflicted files with `jj status`.
2. Edit the conflict markers directly into the intended final content.
3. Run `jj status` and confirm that no conflict remains.

Do not run `jj resolve`, and do not run an equivalent of `git add` after editing.

## Positioning a Bookmark for Publication

Do not move bookmarks during ordinary implementation. Position one only when the user asks to push or create a PR, including when the push is implicit in the PR request.

Never assume `@` is the revision to publish. After `jj new`, it may be an empty change. First identify the newest ancestor of `@` that has both content and a description:

```bash
jj log --ignore-working-copy --no-graph \
  -r 'heads(::@ & ~empty() & ~description(exact:""))' \
  -T 'change_id.short() ++ " " ++ description.first_line() ++ "\n"'
```

Read the result back before positioning the bookmark:

```bash
jj bookmark create <name> -r 'heads(::@ & ~empty() & ~description(exact:""))'
jj bookmark move <name> --to 'heads(::@ & ~empty() & ~description(exact:""))'
jj status
```

Use `create` only for a new bookmark and `move` for an existing one. Do not use `jj bookmark advance`; spell out the target revision at the call site.

## Remote and PR Workflow

Unless instructed otherwise, use `main` as the PR base and preserve each change in a stack.

```bash
jj git fetch
jj log --ignore-working-copy
jj bookmark list --all --ignore-working-copy
```

Position the bookmark as described above. If an existing remote bookmark is untracked, track it with `jj bookmark track <name>@origin`. If the local and remote bookmarks differ, publish with:

```bash
jj push -b <name>
```

Create the PR with an explicit Conventional Commits title and a Japanese body:

```bash
gh pr create --base main --head <name> --title "<type>: <summary>" --body "<body>"
```

## Recovery and Troubleshooting

Before `jj undo`, `jj op restore`, `jj restore`, or `jj abandon`, inspect the affected operation or revision and confirm that it is exactly the intended target. Do not discard or rewrite user work based on an assumed target.

### Stale working copy

```bash
jj workspace update-stale
jj status
```

### Immutable revision

Do not bypass immutability. Check the target revision, choose the intended mutable change, and retry the operation there.
