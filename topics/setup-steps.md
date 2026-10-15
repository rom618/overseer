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

## kuro onboarding plan (rom's answers 2026-10-15)

Decided: keep the current Linux Mint install (cleaned, not reinstalled); Cinnamon stays, dotfiles
is for mikaela only; rom is the only user; no backup for now; laptop agents reach kuro over ssh
(the gpu-queue sends jobs that way).

Prep (agent, before the evening): the machine age key and the Ciel agent install move from
dotfiles `setup-machine.sh` (Arch, mikaela only) into nier `setup-machine.sh` (any Linux); a sops
file for kuro, encrypted to kuro's machine key and rom's admin key, holding its Ciel agent token,
aether key and Ciel read token.

Evening, in order:
1. rom at kuro: Mint updates and reboot; `hostnamectl set-hostname kuro`; apt `git curl age gh
   openssh-server`; `gh auth login`.
2. rom at kuro: Tailscale install, `sudo tailscale up --operator=$USER`, browser sign-in; key
   expiry off in the admin console; no `--accept-routes` (kuro stays at home).
3. rom at kuro, one command (rom 2026-10-15: "a simple script"; nier `bootstrap/join.sh`):
   `gh repo clone rom618/nier /tmp/nier && bash /tmp/nier/bootstrap/join.sh`. It clones overseer
   and nier to `/opt`, runs nier's install (machine age key made where dotfiles is absent), allows
   the laptop's key `id_ed25519_kuro` (ssh alias `kuro`, sshd keys only), prints the age public
   key, then waits for the Ciel device token.
4. Laptop agent over ssh: `ciel-hub device add kuro rom`, token to kuro's
   `~/.config/ciel/device-token` (join.sh moves it to `/etc/ciel/token` and installs the agent);
   the aether key `kuro-rom` to `~/.config/aether/key`; a Ciel read token to `~/.config/ciel/token`;
   `nier register --role desktop --owner rom`. Recovery copies in a sops file: later.
8. kuro: Claude Code installed, one test request through aether; Firefox "Sign in with Ciel".
9. A Claude session on kuro (`~/claude/issues/kuro`): clean-up and malware check, look-only
   first (packages, package sources, autostart, timers, browser extensions, open ports), then
   ClamAV and rkhunter, report; nothing removed without rom's yes. Second monitor: OS language and
   power saving by the agent, the monitor's own menu language and eco setting by rom.

## Open

- [ ] kuro onboarding (plan above): prep first (agent), then the evening steps with rom at kuro (since 2026-10-15)
- [ ] Fold the one-off `pending/` scripts that describe lasting state into `setup-machine.sh` or `user-setup.sh` (hostname, ssh layout, tailnet join), and delete the rest (since 2026-10-13)
- [ ] One "secrets restore" step: every secret a reset needs listed in one place with where its recovery copy lives, so a reset is "install, restore secrets, run setup". rom 2026-10-13: passwords and API keys people use go to Bitwarden (Vaultwarden); open question: if a device is lost, what has to be revoked and what restored, and where each piece lives. Later, keep every machine secret in sops files in the repos; decided 2026-10-13 (rom): one age key per machine, its public key recorded with the machine when it registers in Ciel, the laptop re-encrypts for each (`sops updatekeys`), a lost device only exposes its own secrets; a rebuilt machine gets its new key, is added, then `bin/hs push` writes every `.env` from the sops file. Roberta done 2026-10-13 (home-server b0b5c83: `secrets/roberta.yaml`, `bin/hs secrets-pull`, README Secrets; both keys checked to open it). Laptop done 2026-10-13 (dotfiles `sops/mikaela-rom.yaml`, `bootstrap/user-sops.py`, system-secrets.sh to the admin key). Ciel records each machine's age public key since 2026-10-13 (ciel 515932f; roberta reported; mikaela since rom's setup-machine.sh run; every machine has its own key in Ciel, separate from rom's admin key, which belongs to the person). Left: kuro, whose setup then needs only: install, setup-machine.sh (machine key reported), the laptop adds that key to the files kuro needs and runs `sops updatekeys`, then kuro applies them; Camille's `.env` kept out until rom decides. ssh keys go into sops too, with their keyring passphrases (rom 2026-10-13; dotfiles `bootstrap/user-sops.py`); passwords people type stay in Vaultwarden; first fix the stale laptop copy of roberta's secrets (system audit finding 4) (since 2026-10-13)
- [ ] A per-user first-login script that does the manual items above in order and checks each (ssh keys registered, backup works, Ciel token, aether key) (since 2026-10-13)

## Log

- 2026-10-13: first count, from the scripts and the TODO, by the agent during the night session.
