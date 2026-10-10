---
name: dotfiles
summary: /opt/dotfiles, the shared desktop config repo: install, link, VM test, ii-update, packages, system files
status: active
owner: shared
host: mikaela
repos: /opt/dotfiles
folder: none
verified: 2026-10-10
source: /opt/dotfiles (README, system/README.md, bootstrap/, git log), notes/history.md 2026-09-12 to 2026-10-10
---

# dotfiles

`/opt/dotfiles` (remote rom618/config) holds the desktop for both accounts on mikaela plus the
backup tooling. Owned `rom:dots`, group writable. Layout and link table: `overseer/machines/mikaela.md`.
How to run the scripts: `/opt/dotfiles/README.md`. Agent config is a separate repo, `/opt/nier`.

## Facts

- `bootstrap/links.manifest` is the single source of what is linked where; `link.sh` applies it
  for the calling user (`--dry-run`, `--adopt`). `sudo -u work link.sh` works without a user bus.
- `link.sh` enables per-user timers by symlink (restic-*, hypr-session-save) and starts them when
  a user bus is reachable. `systemctl disable --now` on a unit also removes its unit-file symlink;
  rerun `link.sh` to recreate it.
- `install.sh` (fresh Arch): packages, both users linked, sddm, then asks whether to set up
  `/opt/nier` (`NONINTERACTIVE` skips unless `CLAUDE_CONFIG=1`).
- VM test: `bootstrap/test/vm-install-test.sh [phase2]` builds a throwaway qemu Arch VM
  (3 cores, 8 GB, about 35 min after the base image; 15 min stall watchdog), runs `install.sh` from a
  read-only 9p share, leaves report and SDDM screenshot in `~/.cache/dotfiles-vmtest/`. Run it
  through `detach.sh`. Last pass 2026-09-12.
- Packages: `packages.txt` (pacman) and `packages-aur.txt`; the pacman hook in
  `system/etc/pacman.d/hooks/` keeps them in sync after every pacman/yay run
  (`update-packages.sh` refreshes by hand). `packages-aur.txt` still lists the old
  `illogical-impulse-*` names that upstream renamed `mainstream-*`; `install.sh` tolerates it.
- ii-update workflow (upstream end-4): `bootstrap/ii-update.sh [--deps] [--dry-run]`, never
  `./setup`. State: `ii.commit` (upstream commit synced to), `ii.updateignore` (custom/, generated
  colors, fish_variables). Procedure: `overseer/runbooks/ii-update.md`. `ii-diff.sh` lists what
  differs from upstream. Quickshell has no override layer, so local edits to upstream QML files
  (see `topics/quickshell-bar.md`) must be re-applied if a merge drops them.
- `system/` is installed by `sudo bootstrap/setup-machine.sh` (idempotent, `install` copy): sddm
  and SilentSDDM theme config, login avatars, pacman hook, Firefox `policies.json` (previous
  session, local CA trust), logind `power-button.conf` (HandlePowerKey=ignore) and `lid.conf`.
  `systemd-timesyncd` is enabled there too. Not tracked yet: Ollama drop-in (see Open).
- `/opt/dotfiles/claude` and `/opt/claude-config` are untracked, gitignored links to `/opt/nier`
  kept only until `work` relinks.
- Pre-commit hook runs gitleaks. Other sessions often hold uncommitted work here: stage own paths
  (`/opt/nier/tools/commit-precise.sh`), never `git add -A`.
- Wallpapers: `wallpapers/`, with `/opt/wallpapers` linked to it; the SDDM background is copied
  into the theme by setup-machine.

## Open

- [ ] Track `/etc/systemd/system/ollama.service.d/override.conf` in `system/` (since 2026-10-10)
- [ ] `packages-aur.txt` metapackage names: fixed by the next `ii-update.sh --deps` (since 2026-09-12)
- [ ] work: run `/opt/nier/bootstrap/link.sh`, then remove the old links (since 2026-10-10)
- [ ] Stale text: `system/README.md` says lid.conf needs a reboot (it was applied) (since 2026-10-10)
- [ ] Uncommitted work of other sessions: `bootstrap/packages.txt`, `config/fish/config.fish`
      (an alias with a hard-coded home path) (since 2026-10-10)

## Log

- 2026-09-12: VM install test passed end to end (`bootstrap/test/`); ii-update workflow, backup
  scripts, system/ and wallpapers tracked in the repo.
- 2026-09-13: recurring hazard recorded: `scripts/commit.sh` runs `git add -A` and scoops up other
  sessions' edits; `commit-precise.sh` stages named paths only.
- 2026-09-24: Claude Code config split out to its own repo (now `/opt/nier`); `gpu-usage.py` moved
  here because the bar uses it (`config/hypr/custom/scripts/`).
- 2026-10-09: `lid.conf` moved into `system/` so setup-machine installs it; retired units
  (`ai-key-listener`, old session-save service) removed from the manifest, scripts stay unwired;
  both `link.sh --dry-run` clean (dotfiles d2c5a59).
- 2026-10-10: repo pointers renamed to `/opt/nier`; backup never-list added (387c6a8, see backups).
