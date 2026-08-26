## Version Control — Required Procedure

This project uses **Jujutsu (jj)** for version control. For the detailed rules, see `.claude/rules/jujutsu-rules.md`.

**Before you begin editing code**, always settle which change the work belongs to: inspect working-copy commit (`@`), reuse it if it is empty, otherwise open a new one. The decision procedure is in the rules file under "1. Starting Work" of "Basic Workflow" — do not start editing and sort the changes out afterwards.

**Prohibited:** Direct use of `git` commands (the `jj git` subcommands and the `gh` CLI are allowed).
