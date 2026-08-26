# Jujutsu (jj) Rules for AI Agents

Standard jj semantics are assumed knowledge. What follows is only what an agent cannot infer from the tool itself: this repository's permissions, its aliases, and the conventions the team has settled on.

---

## Important Notes for AI Agents

### Command Constraints

`.claude/settings.json` denies these outright. Never plan a workflow around them; if one is genuinely needed, describe the command and let the user run it.

| Denied        | Why                                                                                 |
| ------------- | ----------------------------------------------------------------------------------- |
| `git` (all)   | Risks corrupting jj state. `jj git ...` and `gh` remain allowed.                    |
| `jj resolve`  | Opens the interactive merge editor. Resolve conflicts by editing the files instead. |
| `jj diffedit` | Opens the interactive diff editor.                                                  |
| `jj arrange`  | Interactive TUI.                                                                    |

`jj split` is allowed, but **always pass filesets and `-m`. Never run it bare, and never pass `-i` or `--tool`** because these actions trigger an interactive diff editor.

These prompt for confirmation every time, so save them for the end of a task rather than firing them mid-flow: `jj push`, `jj bookmark delete`, `jj bookmark forget`, `jj bookmark track`, `jj bookmark untrack`.

### What jj Changes About the Workflow

- **No staging, no stash, no "unsaved change".** Saving a file puts it in the working-copy commit (`@`) that instant. Never ask the user whether a change should be included — it already is.
- **Bookmarks are not branches and do not follow `@`** — and they do not need to. Leave them where they are while you work; there is nothing to keep in sync. Position one only when it is about to be used (see "Positioning a Bookmark").
- **Automatic formatting (`jj fix`)**: This project has `jj fix` set up. Because it is registered as a Stop hook, it runs when a task finishes and retroactively reformats the code across the mutable changes. Even if `jj diff` shows unintended formatting changes, accept them as long as they conform to the project's rules.

### Rules for diff Output

Always pass `--git` to `jj diff`, `jj show`, and `jj log -p`. Without it the output cannot express file additions, deletions, renames, and permission changes accurately, and an agent's reading of the diff degrades accordingly.

### Suppressing Snapshots in Read-Only Operations

Every jj command snapshots the working copy, which risks an operation-log conflict when another process is working in the same repo. Add `--ignore-working-copy` to purely read-only inspection — `jj log`, `jj diff`, `jj bookmark list`, `jj evolog`, `jj op log`.

**Never add it right after creating, editing, or deleting a file**: that is exactly when the snapshot has to be recorded. Use `jj util snapshot` if you want one without running anything else.

### Choosing the Right Log Output

Read `jj log`'s graph as-is for ordinary state checks. To extract values programmatically, add `--no-graph` and `-T`:

```bash
jj log --ignore-working-copy --no-graph -T 'change_id.short() ++ " " ++ description.first_line() ++ "\n"'
```

---

## Custom Commands and Dependencies

`jj push` is a repository alias, not the bare command: it runs `jj fix && mise run pre-push && jj git push`. Always publish through it, and note that it requires `mise`.

---

## Basic Workflow

### 1. Starting Work

Adjust your actions according to the state of the working-copy commit (`@`):

1. Run `jj status` — **without** `--ignore-working-copy`, the one exception to the rule above. The user may have edited files since the last jj command, and skipping the snapshot would report those edits as absent.
2. **If `@` is empty** (NO description AND NO diff) → **reuse it.** Set its description with `jj describe -m "<description>"` and work there. **Do not create a new change.** A change created by `jj new` is itself empty, so it falls under this rule too.
3. Otherwise (`@` already has a description or a diff) → `jj new -m "<description>"`.
4. Write the description in English, in Conventional Commits format.

### 2. Splitting a Change

```bash
jj split -m "<description for the extracted change>" <path>...
```

Filesets are what keep it out of the diff editor, `-m` out of the description editor. Hunk-level splits need that editor, so hand those to the user.

### 3. Resolving Conflicts

Edit the marked-up files directly, then confirm with `jj status` that the conflict is gone. Saving the file _is_ the resolution; there is no `git add` equivalent to run afterwards.

jj's conflict markers differ from Git's: the `%%%%%%%` block is a diff against the base, expressed with `-`/`+` lines, while the `+++++++` block is the other side's content verbatim.

### 4. Positioning a Bookmark

Move a bookmark only when asked to push, or asked to open a PR with the push step left implicit. Otherwise leave bookmarks alone — a change does not need one to exist.

Never point one at `@` unexamined: after `jj new` it is empty and undescribed, and `jj git push` refuses a commit whose description is empty. Target the newest ancestor of `@` that has both a diff and a description, and read it back before moving:

```bash
jj log --ignore-working-copy --no-graph \
  -r 'heads(::@ & ~empty() & ~description(exact:""))' \
  -T 'change_id.short() ++ " " ++ description.first_line() ++ "\n"'

jj bookmark create <name> -r 'heads(::@ & ~empty() & ~description(exact:""))'  # the first time
jj bookmark move <name> --to 'heads(::@ & ~empty() & ~description(exact:""))'  # afterwards
```

**Do not use `jj bookmark advance`.** It takes its target from `revsets.bookmark-advance-to`, so the landing point is decided by repository configuration rather than by the command you wrote. Spell the target out at the call site.

### 5. Syncing with the Remote

```bash
jj git fetch                  # fetch the latest state
jj push -b <bookmark-name>    # publish (alias — see above)
```

---

## PR Creation Workflow

- The base is always `main` unless instructed otherwise. No confirmation needed.
- **Never squash.** Keep each change as-is so the history stays traceable.
- Pass the title explicitly with `--title` in Conventional Commits format, derived from the description of the bookmark's lead change. Do not rely on the title `gh` derives.

```bash
jj git fetch
jj log --ignore-working-copy
jj bookmark list --all --ignore-working-copy   # then position it — see above
jj bookmark track <bookmark-name>@origin       # only if untracked
jj push -b <bookmark-name>                     # skip if local and remote already match
gh pr create --base main --head <bookmark-name> --title "<type>: <summary>" --body "<body>"
```

---

## Troubleshooting

**"The working copy is stale"** — a human and an agent worked in the repo in parallel, or an external tool changed files. Run `jj workspace update-stale`, then confirm with `jj status`.
