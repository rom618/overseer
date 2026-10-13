---
name: setup-steps
summary: How many manual steps a reset or a new device costs today, per device and user, and how to cut them
status: active
owner: shared
host: mikaela
repos: /opt/dotfiles, /opt/nier, home-server
folder: ~/claude/issues/ciel
verified: 2026-10-13
source: dotfiles bootstrap/install.sh and user-setup.sh, nier bootstrap/install.sh, home-server host/setup.sh, overseer TODO "Work account", the pending/ scripts of the ciel issue folder (2026-10-10 to 2026-10-12)
---

# Setup steps

Goal (rom, 2026-10-12): a reset or a lost device costs as few manual steps as possible.

## Facts

- Laptop from a fresh Arch install: one command (`dotfiles/bootstrap/install.sh`, 9 scripted
  steps) plus the prompts it cannot avoid: sudo password, the upstream ii installer's questions
  (interactive unless `NONINTERACTIVE=1`), work's password, the optional agent config.
- Then by hand, outside any script: ssh keys per purpose (`nier/tools/ssh-key-new.sh`, one run per
  key, then registering each public key on GitHub, roberta, the forge), the restic backup (rclone
  login to Google Drive, `~/.config/backup/restic.env`, the restic password), Surfshark login,
  Tailscale join (`pending/mikaela-tailscale-join.sh`, a browser sign-in), the Ciel agent token
  (`/etc/ciel/token`, then `setup-machine.sh` again), the user's Ciel API token, the aether key
  (`hs aether-key-save`), Firefox profile and extensions, the vault passphrase.
- A second user (work, measured 2026-10-10 to 2026-10-12): `user-setup.sh` (7 scripted steps) plus
  about 8 manual items (overseer TODO "Work account"): password reset, ssh keys and their
  registration, ssh-agent, tailnet operator, Ciel token, restic remote, aether key.
- Server (roberta): `bin/hs push` runs `host/setup.sh` (idempotent, everything from git) plus the
  secrets that never enter git: Cloudflare token, playit key, Surfshark WireGuard key, Camille's box
  passphrase, Ciel DSNs (`hs camille-dsn`, `hs aether-dsn`), Tailscale login, approving the subnet
  route and disabling key expiry in the admin console.
- 14 one-off scripts were handed to rom from the ciel issue folder's `pending/` in three days: each
  is a step a future reset would repeat unless it is folded into a setup script.

## Open

- [ ] Fold the one-off `pending/` scripts that describe lasting state into `setup-machine.sh` or `user-setup.sh` (hostname, ssh layout, tailnet join), and delete the rest (since 2026-10-13)
- [ ] One "secrets restore" step: every secret a reset needs listed in one place with where its recovery copy lives, so a reset is "install, restore secrets, run setup". rom 2026-10-13: later, keep every machine secret and ssh key in one encrypted store (sops files encrypted with an age key proposed; passwords people type stay in Vaultwarden); first fix the stale laptop copy of roberta's secrets (system audit finding 4) (since 2026-10-13)
- [ ] A per-user first-login script that does the manual items above in order and checks each (ssh keys registered, backup works, Ciel token, aether key) (since 2026-10-13)

## Log

- 2026-10-13: first count, from the scripts and the TODO, by the agent during the night session.
