# Errata & Updates for _Juju-chu!—Starting Your Jujutsu × AI Workflow with `jj new`_

Last updated: September 18, 2026

### Please Note

- This errata page is updated as needed.
- Both the ebook and paperback editions may be updated to new versions that incorporate corrections and revisions as needed.
- Page numbers for corrections are based on the initial version of the PDF and paperback. As versions are updated, page numbers may shift relative to those in the version you have.

<br>

### Corrections

- Column in chapter 2 / p.64

```diff
- Column: Jujuts's Roots—What Kind of VCS Is Mercurial?
+ Column: Jujutsu's Roots—What Kind of VCS Is Mercurial?
```

- 3-2-1. Running `jj fix` via Hooks / p.79 / code block

```diff
  [fix.tools.biome]
  command = [ "pnpm", "exec", "biome", "check", "--write" ,"--stdin-file-path",
  "$path" ]
  patterns = ["glob:'**/*.
- {js,mjs,cjs,ts,mts,cts,jsx,tsx,html,css,json,jsonc,yaml,yml,md,mdx}'"]
+ {js,mjs,cjs,ts,mts,cts,jsx,tsx,html,css,json,jsonc}'"]
```

```diff
  [aliases]
  set -e
  lint = ["util", "exec", "--", "sh", "-c", """
- files=$(jj diff -r 'reachable(@, mutable())' --name-only)
+ files=$(jj diff -r 'reachable(@, mutable())' --name-only |
+     while IFS= read -r f; do if [ -f "$f" ]; then printf '%s\\n' "$f"; fi; done)
  js_files=$(echo "$files" | grep -E '\\.(js|mjs|cjs|ts|mts|cts|jsx|
  tsx)$' // true)
```
