# Backups and recovery

Owners: laptop how-to `/opt/dotfiles/backup/README.md`; state `/opt/overseer/topics/backups.md`;
server how-to home-server `README.md:70-82, 155-160, 210-211`; script
`home-server/scripts/offsite-backup.sh`.

## The picture

```
 mikaela (per user)                         roberta
 restic-backup.timer (3 h) ---+             /srv/data/<stack> --restic--> /backup/<stack> (2nd SSD)
 restic-maintain (weekly)     |                 minecraft (30 min), vaultwarden (03:00),
                              |                 ciel (03:15), archive (Sun 03:30)
                              v                                   |
            Google Drive, one 15 GB personal account  <-- 04:30 offsite-backup (rclone sync)
            gdrive:backups/camille-rom (rom's laptop repo)   gdrive:backups/home-server/...
```

## What is protected, where

| Data | Copy 1 | Copy 2 (off-site) | Source |
|---|---|---|---|
| rom's laptop files (`includes.txt`) | live disk | restic `gdrive:backups/camille-rom`, every 3 h | `machines/mikaela.md:83-92` |
| Git repos with a remote | GitHub | git-snapshot (remote, HEAD, unpushed bundle, patch, untracked tar) | `backup/README.md:27-35` |
| `~/claude/issues` small documents | live | staged into the restic backup | `backup/docs-only.txt` |
| Firefox logins and bookmarks | profile | staged into the restic backup | `backup/README.md:41-45` |
| Wi-Fi secrets (root) | `/etc/NetworkManager` | `system-secrets.sh` age file, by hand, stale since 2026-09-12 | `topics/backups.md:49-50, 68` |
| Server secrets copy | roberta | `~/.config/home-server/` on the laptop, in the restic backup | `backup/includes.txt:15` |
| Minecraft world | `/backup/minecraft` (2 d full, 7 daily, 4 weekly, 3 monthly) | latest snapshot only | `stacks/minecraft/compose.yaml:81`, `offsite-backup.sh:10-22` |
| Vaultwarden | `/backup/vaultwarden` | whole repo | `offsite-backup.sh:24-27` |
| Ciel database | `/backup/ciel` | whole repo | `offsite-backup.sh:29-32` |
| Worlds archive, Terraria | `/backup/archive`, `/backup/terraria` | whole repos | `offsite-backup.sh:34-42` |

## Not backed up (by choice or gap)

| Data | Why | Source |
|---|---|---|
| Camille (media and app data, laptop vaults, laptop deployment) | rom's rule, `never.txt` | `backup/never.txt:23-28` |
| aether database (account tokens, keys, usage) | decided 2026-10-12: two logins to redo | `topics/aether.md` Open |
| Prometheus history | disposable, Ciel replaces it | home-server README:79-80 |
| `~/.config/aether/key`, `~/.config/ciel/token` | not in `includes.txt`; regenerable from roberta | `backup/includes.txt` |
| `~/.config/mc-server` (a restic password copy), `~/.ollama`, `~/Downloads` | gap, Open | `topics/backups.md:61-62, 69` |
| `~/Downloads/ccflare.zip` (aether's fork history) | in `~/Downloads` | `topics/aether.md` Open |
| work's laptop data | work has no backup yet | `topics/backups.md:66` |
| GitHub itself | separate project, Open | `topics/backups.md:70` |
| Gitignored files of repos (`.env` of apps) | by choice | `topics/backups.md:38-41` |

## The keys that open the backups

| Backup | Key | Where the key lives | Recovery copy |
|---|---|---|---|
| laptop restic (rom) | `~/.config/restic/password` | rom's laptop, never readable by agents | only rom (written down?) |
| server restic repos | `/etc/home-server/restic.pw` | roberta | `~/.config/home-server/` in the laptop backup |
| Drive (both) | rclone OAuth config | `~/.config/rclone` (laptop, in backup), `/etc/home-server/rclone.conf` (roberta) | the Google account |
| Wi-Fi secrets | age passphrase | rom | none |
| Vaultwarden contents | each user's master password | rom, written down offline | none by design (README:159-160) |

The chain: the laptop restic password opens the laptop backup, which holds the server's restic
password and rclone config, which open the server's off-site copies, which hold Vaultwarden.
Vaultwarden is meant to hold the other passwords (`overseer/TODO.md:33-43`). So the one secret
that must survive outside every machine is the laptop restic password (and the Google login).

## Recovery order (derived, not rehearsed)

Lost laptop:
1. Fresh Arch, `dotfiles/bootstrap/install.sh`, nier installer (`topics/setup-steps.md:19-21`).
2. `rclone config` (remote `gdrive`), write the restic password, `restic-backup.sh restore latest`
   (`backup/README.md:52-58`); `git-restore.sh` for repos.
3. New ssh keys per purpose, register them (GitHub, roberta's `authorized_keys` needs console or
   an old key), Surfshark, Tailscale join, Ciel tokens, aether key
   (`topics/setup-steps.md:22-27`).

Lost roberta:
1. New Debian host, clone home-server, `bin/hs push` runs `setup.sh`.
2. Secrets that never enter git: Cloudflare token, playit key, Surfshark key, restic.pw and
   rclone config (from the laptop copy), Ciel and aether DSNs, Tailscale login and route approval
   (`topics/setup-steps.md:31-34`).
3. Restore Vaultwarden and Ciel from `gdrive:backups/home-server`; Minecraft from the latest
   snapshot (README:92-103). aether: relink both accounts, new client keys for every device.
   Camille: nothing to restore.
4. Rebuild and push the images (`bin/deploy` in ciel, aether, camille).

A restore of roberta's backups on another machine has never been rehearsed (Open,
`topics/backups.md:71`). The laptop restore was tested 2026-10-10 (45 s, byte-identical,
`topics/backups.md:58-60`).

## Who notices a failed backup

- Laptop: `backup-failed@.service` desktop notification, and the Ciel agent turns the failed user
  unit into an issue under `mikaela-rom-units`; heartbeats catch missed runs (Ciel README:88-95,
  120-123). The 2026-09 incident (three weeks of ignored failures) is why Ciel exists
  (`topics/backups.md:77`).
- roberta: Ciel heartbeats for every timer (`topics/ciel.md` Log 2026-10-11). If roberta is down,
  its timers do not run and only the laptop's crossed-cloud bar icon shows it (`failure-modes.md`).
- Drive space: a warning under 2 GiB free after a laptop backup (`backup/README.md:23-24`). The
  Drive filled once (2026-10-04) and broke the off-site job (`topics/backups.md:78`).
