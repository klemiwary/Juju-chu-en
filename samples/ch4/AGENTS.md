## Version Control —  Required Procedure

This repository uses Jujutsu (`jj`). Use `jj` terminology and commands; do not run raw `git` commands. `jj git ...` and `gh` are allowed.

Before editing, run `jj status` to snapshot and inspect working-copy commit (`@`). Reuse `@` only when it has neither a description nor a diff; otherwise create a new change. Write change descriptions in English using Conventional Commits.
Always use `--git` with `jj diff`, `jj show`, and `jj log -p`. Use `--ignore-working-copy` for purely read-only inspection only after the working copy is known to be snapshotted.

Do not run interactive `jj split`, `jj resolve`, `jj diffedit`, or `jj arrange`. After history-changing operations, run `jj status` and resolve any conflicts before continuing.
Do not move bookmarks unless publishing changes. PRs target `main` unless instructed otherwise, and stacked changes must not be squashed.

For non-trivial revsets, conflicts, bookmark operations, history rewriting or recovery, fetch, push, and PR creation, consult the `jujutsu` Skill at `.agents/skills/jujutsu/SKILL.md`.
