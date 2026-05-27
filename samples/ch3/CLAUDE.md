## Version Control — Required Procedure

This project uses **Jujutsu (jj)** for version control. For the detailed rules, see `.claude/rules/jujutsu-rules.md`.

**Before you begin editing code, always perform the following steps:**

1. Check the current change with `jj log --ignore-working-copy -r @`.
2. If the description is empty and the diff is empty (an `empty` change) → set a description with `jj describe -m "<description>"` and begin work.
3. Otherwise (work already in progress, or already finished) → create a new change with `jj new -m "<description>"`.
4. Write the description in Conventional Commits format.

**Prohibited:** Direct use of `git` commands (the `jj git` subcommands and the `gh` CLI are allowed).
