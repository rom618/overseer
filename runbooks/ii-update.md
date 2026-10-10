# Updating illogical-impulse (end-4 upstream) on mikaela

Never run `./setup install` or `./setup exp-update` by hand. Use
`/opt/dotfiles/bootstrap/ii-update.sh [--deps] [--dry-run]`:

- pulls upstream and 3-way merges every changed upstream file against the commit in
  `bootstrap/ii.commit`, copies new files in;
- runs exp-update in "keep" mode for per-user files, with `bootstrap/ii.updateignore` protecting
  `custom/` and generated files;
- re-links with `link.sh --adopt` and commits one reviewable change;
- `--deps` runs upstream's own dependency step (the metapackages in `packages-aur.txt`), so their
  dependency graph is never hand-maintained.

`bootstrap/ii-diff.sh` lists what is customized relative to upstream. Edits made directly to
upstream quickshell files (no override layer exists there) show up there and may need
re-applying after a merge.

If a manual upstream step is ever unavoidable: confirm with the user, commit `/opt/dotfiles`
first, and review `git -C /opt/dotfiles diff` afterwards.
