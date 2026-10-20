---
name: backups
summary: Laptop restic backup to Google Drive per user, git snapshots, and the server off-site copy
status: active
owner: shared
host: mikaela, roberta
repos: /opt/dotfiles, ~/Desktop/perso/projects/home-server
folder: ~/claude/issues/ciel
verified: 2026-10-10
source: /opt/dotfiles/backup/README.md, notes/history.md (2026-10-04 Drive, 2026-10-09 issues backup, 2026-10-10 audit, fixes, rename), nexus audit-2026-10-10.md, dotfiles commits 975b42b 387c6a8
---

# Backups

Laptop: each user backs up its own personal data with restic to Google Drive through rclone
(client-side encrypted, Google sees ciphertext). Server: a nightly job copies the latest game
snapshot off-site to the same Drive. Procedures, setup steps, restore commands and the full
include logic: `/opt/dotfiles/backup/README.md` (the owner of how-to). Device summary:
`machines/mikaela.md`; server side: `machines/roberta.md` and `topics/home-server.md`.

## Facts

- Where things live (laptop): scripts and units in `/opt/dotfiles/backup/` (`restic-backup.sh`,
  `git-snapshot.sh`, `git-restore.sh`, `system-secrets.sh`, `includes.txt`, `excludes.txt`,
  `never.txt`, `docs-only.txt`, `git-full.txt`). Shared units, per-user timers
  (`restic-backup.timer` every 3 h, `restic-maintain.timer` weekly). Staged copies and state in
  `~/.local/state/backup/` (included in the backup).
- Repo and host are pinned per user in `~/.config/backup/restic.env` (rom: `gdrive:backups/camille-rom`,
  host label `mikaela`). Reason: the default repo path and the forget grouping derived from
  `/etc/hostname`, so the 2026-10-10 hostname rename would have split the history. No automatic
  `init` any more. The repo keeps its old path on purpose.
- The restic password is `~/.config/restic/password` (never readable by agents). It is the only key
  to the repo. Recovery copies of rom's: Bitwarden (item "restic backup mikaela-rom", checked by
  paste-back at the 2026-10-20 change) and `sops/mikaela-rom.yaml` in dotfiles (age key in
  Vaultwarden). rclone remote is named `gdrive`.
- Retention: 24 hourly, 14 daily, 8 weekly, 12 monthly, 5 yearly; weekly prune and a 5% data check.
  Failures and "Drive under 2 GiB free after a backup" raise a desktop notification.
- `backup/never.txt`: paths never backed up in any form (restic, git-snapshot patch, bundle or tar).
  It holds the media library's app data and the media vault, by the user's rule of 2026-10-10, and
  since 2026-10-20 the Claude transcripts of sessions about them (`~/.claude/projects/*camille*`,
  `*media-library*`). Other transcripts stay in the backup (user, 2026-10-20).
- The restic password is protected from agents' reads, not from their use: `restic-backup.sh` and
  `restic` are on the agent allow list, so an agent session can open the repo (it did on 2026-10-20
  for the content review) without ever seeing the password.
- Git repos with a remote are not uploaded: `git-snapshot.sh` records remote, branch, HEAD, plus
  bundle of unpushed commits, patch of uncommitted changes, tar of untracked files. Gitignored files
  of those repos are in no backup (by choice for the media app's `.env` and Caddy data; the mc-modpack
  `.env` is the same case). Unpushed detection only checks the current branch.
- `docs-only.txt`: of `~/claude/issues` only small documents (md txt html json csv yml toml py sh lua,
  up to 1 MiB, hidden dirs skipped) are rsynced to `~/.local/state/backup/docs/` before each backup.
  A stable staging path, not per-file `--files-from`, because `--group-by host,paths` would make each
  changed list its own snapshot group and forget would never prune.
- Firefox is not signed in to Sync (user's choice): `stage_firefox` copies `logins.json`,
  `logins-backup.json`, `logins.db`, `key4.db` and `bookmarkbackups/` of each profile to
  `~/.local/state/backup/firefox/<profile>/`. `places.sqlite` is skipped (locked by Firefox).
- Wi-Fi secrets (root-only): `sudo backup/system-secrets.sh`, age-encrypted with a passphrase
  the user chose; output lands in `~/.local/state/backup/`.
- Drive: one 15 GiB personal account shared by the laptop repo, the server off-site copy and a few
  zips. Deletions skip the Drive trash (`RCLONE_DRIVE_USE_TRASH=false`); the server job empties
  the trash every run at 04:30. The agents' auto-mode classifier refuses Drive deletions
  (`rclone cleanup`, `deletefile`, `purge`): the user runs those by hand.
- Server off-site: `offsite-backup.timer` (04:30) restic-copies only the latest Minecraft snapshot
  into `/backup/minecraft-offsite` (keep-last 1) and rclone-syncs it, plus the Vaultwarden, Ciel,
  worlds-archive and Terraria repos and Camille's encrypted metadata exports (`camille-export.timer`, ciphertext only, since 2026-10-14), to `gdrive:backups/home-server` (`scripts/offsite-backup.sh`). Details: `topics/home-server.md` and the home-server README.
- Restore tested 2026-10-10: `restic-backup.sh restore latest` into scratch took 45 s (496 MiB),
  byte-identical to live except files changed since. `git-restore.sh` reproduced HEAD and uncommitted
  changes on three repos but has a bug (Open).
- Not backed up on purpose or by gap: browser profiles other than the Firefox files, `~/.ollama`,
  `~/Downloads`, `~/.config/mc-server` (see Open), `~/.gitconfig`.

## Open

- [x] (2026-10-18, work ran it; 2 snapshots, timers enabled, dotfiles d1be649 for `sops/mikaela-work.yaml`) work has a backup. Decided 2026-10-18 (user): same Google Drive as rom, repo `gdrive:backups/mikaela-work`. work: in a real terminal, `~/claude/issues/set-up/pending/work-backup-setup.sh` (fixed 2026-10-18 after a sign-in with an empty token; run it only after a job folder's pending ssh move script, see that job's docs, so no job key lands in the backup) (rclone sign-in with rom's Google account, restic password typed or generated, init, first backup, timers, work's sops age key and `sops/mikaela-work.yaml`); save the password and the age key in Vaultwarden when it shows them; then work's Claude commits the sops file. (Scratch timer enabled, group `dots` active and no `dotfiles-backup` folder: checked 2026-10-18) (since 2026-10-10)
- [ ] Own Google OAuth client id for rclone (https://rclone.org/drive/#making-your-own-client-id), for rom's and work's `gdrive` remotes: rclone 1.75 says its shared client id is being retired during 2026, and work's first backup (2026-10-18) hit its shared rate limit (403 rateLimitExceeded, restic retried and succeeded). Needs the Google Cloud console in Firefox, then `rclone config update gdrive client_id=... client_secret=...` and a reconnect per user (since 2026-10-18)
- [x] (2026-10-13, dotfiles) `git-restore.sh` returns to the recorded commit and keeps a patch that fails (`.git/restore-failed.patch`, WARN line); tested on throwaway repos (`backup/tests/git-restore.test.sh`)
- [x] (2026-10-13 19:36, rom) system-secrets.sh re-run; age generated the passphrase (Enter at the prompt): it must go to Vaultwarden, it is the only way to open the file
- [x] (2026-10-20, rom ran `~/claude/issues/mikaela/pending/restic-change-password.sh`; dotfiles aa169b1)
  restic password changed, old key removed, copy in Bitwarden, sops file updated; new password opens
  the repo (85 snapshots)
- [x] (2026-10-20, dotfiles d74fa9c) content review: media-app transcripts into `never.txt`;
  `.claude/file-history`, `plugins`, `telemetry`, `.statusline-cache` excluded; transcripts and
  `paste-cache` kept. New snapshot 902 to 723 MiB, 0 matching transcripts
- [ ] rom: decide whether to scrub the media-app transcripts from the snapshots taken before
  2026-10-20 (`restic rewrite --exclude` then prune, as on 2026-10-10; cannot be undone) (since 2026-10-20)
- [ ] `~/.config/mc-server/home-server-restic.pw` is not backed up; decide what in `~/.config`, `~/.ollama`, `~/Downloads` is disposable (since 2026-10-10)
- [ ] Separate project: back up every GitHub repo and its commits (since 2026-10-10)
- [ ] Rehearse a restore of roberta's backups on another machine, later (rom, 2026-10-13); with it, write where the restic password's recovery copy lives (system audit finding 3) (since 2026-10-10)
- [x] (2026-10-10, home-server) Server backups: worlds-archive weekly, Vaultwarden daily plus off-site; Prometheus, Grafana, Dockge declared disposable; server secrets copied into the laptop backup

## Log

- 2026-09-12: restic live for rom, every 3 h; old Drive tarball sorted and deleted; `system-secrets` written.
- 2026-09-19: weekly maintain starts failing (exit 11). Root cause found 2026-10-10: a stale lock from a backup killed on 2026-09-13; backups kept working (non-exclusive lock) but nothing was pruned or checked, and the failure notification was ignored three weeks. This incident is the motivation for Ciel (`topics/ciel.md`).
- 2026-10-04: Drive full, server off-site failed with storageQuotaExceeded. Causes: the whole Minecraft restic repo (7.3 GiB, +2.5 GiB/day from pre-generation) was synced nightly, and pruned packs filled the Drive trash (5.3 GiB). Fix: off-site copies the latest snapshot only, no trash, trash emptied daily. Result 3.8 GiB used. Server repo `scripts/offsite-backup.sh`.
- 2026-10-09: `~/claude/issues` added through docs-only staging. dotfiles 56809a1.
- 2026-10-10 audit (read-only, `audit-2026-10-10.md`): backups ran, maintain broken, restore never tested, work unprotected, Drive the only copy, gaps listed under Open.
- 2026-10-10: `maintain` runs `restic unlock` first; backup, forget and check use `--retry-lock 30m`; clean-up run removed 123 of 161 snapshots, repo 773 to 329 MiB, check clean. Firefox staging added; notification left click focuses the sender. dotfiles 975b42b.
- 2026-10-10: user decisions: the media app's `.env` and Caddy data stay out of the backup; no git-snapshot rewrite (a separate GitHub backup project instead); Vaultwarden wanted for passwords.
- 2026-10-10: media app data purged from restic. Cause: snapshot of 2026-09-13 contained the app's repo folder (it had no remote until 09-14) and later snapshots held a git-snapshot tar of its untracked files, against the user's no-backup rule. 36 snapshots rewritten with `restic rewrite --exclude`, pruned with `--max-unused 0`, `restic find` returns nothing. `never.txt` added and used by restic and git-snapshot. dotfiles 387c6a8.
- 2026-10-10: hostname camille to mikaela; 39 snapshots relabelled host `mikaela`; repo path unchanged.
- 2026-10-20: Drive 5.6 of 15 GiB used (plus 0.7 GiB Gmail/Photos): `backups/home-server` 5.04 GiB
  (minecraft-latest 3.0, archive 2.0, the rest under 20 MiB each), `backups/camille-rom` 0.41,
  `mc-modpack` 0.15, `backups/mikaela-work` 0.01, `pulls` under 1 MiB. rom's restic password changed.
