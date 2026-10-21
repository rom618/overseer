---
name: home-server
summary: roberta as a project: Docker stacks deployed with bin/hs push, audited 2026-10-10, to be split personal/work
status: active
owner: shared
host: roberta
repos: ~/Desktop/perso/projects/home-server
folder: ~/claude/issues/ciel
verified: 2026-10-11
source: ~/Desktop/perso/projects/home-server/README.md, machines/roberta.md, nexus audit-2026-10-10.md sections 9 and 10, notes/history.md 2026-10-04
---

# Home server (roberta)

The repo `home-server` (GitHub rom618/home-server) defines the machine and its Docker stacks;
nothing is configured by hand. Hardware, address, key and hard rules: `machines/roberta.md`.
How to run, deploy, restore and the per-stack gotchas: the repo's `README.md` (its only owner).
Personal game content on it (Minecraft, Terraria) is documented in 9s; here only the stacks.

## Facts

- Stacks (`stacks/`): minecraft (+ minecraft-backup), playit (tunnels, host network), bluemap,
  portal (start page), monitoring (prometheus, node-exporter, cadvisor, mc-monitor), vaultwarden,
  camille, ciel, aether, proxy, registry, terraria (stopped). Web apps only through
  https://*.wuling.dev (no LAN ports since 2026-10-11, except qBittorrent 8090, also at
  https://qbit.wuling.dev); host firewall lets only ssh reach host processes. Data `/srv/data/<stack>`,
  restic repos `/backup/<stack>`, checkout `/srv/home-server`.
- Deploy: edit and commit in the repo on the laptop, then `bin/hs push` (git push into the
  server's checked-out branch, then idempotent `host/setup.sh` owning timezone, swap, sleep
  settings, Docker, unattended security updates, timers, secrets generation). The push refuses if
  files were edited on the server.
- Secrets: `stacks/*/.env` (0600, generated, not in git), `/etc/home-server/restic.pw`, the rclone
  config; they exist only on the box. `bin/hs secrets` shows them to the owner. Never copy values
  into a doc.
- Monitoring: Prometheus keeps 365 days; Ciel's charts show node, container and game metrics
  (Dockge retired 2026-10-12, Grafana 2026-10-13). Wall power is not measured.
- Public access: the Flybox 4G is behind CGNAT, so nothing can be port-forwarded; players reach the
  servers through playit.gg tunnels, and every player appears with a playit address (no IP bans).
- Backups: local restic repos per stack; off-site copy of the latest game snapshot to Google Drive
  (see `topics/backups.md`). Restic on the Minecraft repo only through the backup container
  (`docker exec minecraft-backup restic`), because a root restic leaves root-owned locks that break
  the next backup. The off-site job runs as uid 1000, never root on `/backup/minecraft`.
- Timers: `offsite-backup` (04:30), `mc-restart` (10:00), `archive-backup` (Sundays 03:30),
  `vaultwarden-backup`, `ciel-backup`, `backup-budget`; `terraria-backup` is disabled while the
  Terraria stack is stopped.
- Personal and work today: games, playit, BlueMap and the Drive account are personal; Docker,
  monitoring, the `server` user, the `hs` tooling and the off-site script are shared; one
  restic password covers everything.
- Memory (2026-10-17): 12 of 31 GB used after Minecraft moved from a fixed 16 GB heap (18 GB
  used) to a 4-12 GB heap that shrinks when idle, 16 GB limit (7 GB right after boot; home-server
  README "Minecraft"); BlueMap 2.2 GB, qBittorrent 1.6 GB.

## Open

- [ ] Bug (2026-10-15): `hs push` silently reverts any secret set on the box since the last `hs secrets-pull` (`host/secrets.py apply` writes `secrets/roberta.yaml` over the `.env`); `hs mephisto-password`, `mephisto-key`, `clipboard-pin`, the `*-dsn` and `*-aether-key` commands never update the record. It undid rom's Mephisto password reset (9s mephisto). Fix to decide with rom: `hs push` refuses when `secrets.py check` finds the box newer, and each secret-writing `hs` command runs `secrets-pull` itself (since 2026-10-15)
- [ ] rom's list: stacks to add: Forgejo (GitHub replica, TODO), Pi-hole (network topic) and two of rom's own apps (9s); rom's own apps get French names (docs topic, naming) (since 2026-10-13)
- [ ] rom's list: auto-deploy CI once the local forge (Forgejo, GitLab or GitHub runners) is up: a push to `main` of home-server, Ciel and aether deploys it on roberta (today by hand: `bin/hs push`, `bin/deploy`), so the runner is never behind its repo (2026-10-17: home-server f236230, the burn runner's stop check, waited for a manual `bin/hs push`) (since 2026-10-17)
- [ ] Split into personal and work servers (roberta shared only for now): own restic password, repo, Drive account and proxy; parameterize `offsite-backup.sh` (since 2026-10-10)
- [ ] Hardening left on purpose: sudo NOPASSWD until the scoped agent user exists (the docker group is
  root-equivalent anyway, decision 2026-10-11); cAdvisor privileged (no port, Ciel replaces it) (since 2026-10-11)
- [x] (2026-10-11, camille session) Camille 8081 closed, AirPlay over https
- [ ] Docker repo in unattended-upgrades or a monthly upgrade; pin image versions (2026-10-13: monitoring stack pinned, newest releases at least two weeks old, no daily pulls; left: bluemap (pinning means a downgrade from 5.29), mc-backup in the minecraft stack and the pot provider in camille's, not touched at night) and prune old ones (since 2026-10-10)
- [ ] Disk clean-up (2026-10-13: 15.8 GB freed, untagged images left by deploys, two old tModLoader tags, Vaultwarden 1.33.2, build cache over 24 h; the `/var` part is done, see Log 2026-10-17): left: `/srv/staging`, leftover networks, logs, a disk budget per data dir (since 2026-10-10)
- [ ] rom: go for the one-time move of containerd's image store to `/srv` (`scripts/containerd-move.sh`, run detached as its header says): every container stops for a few minutes, aether too (running agent sessions wait and retry), Minecraft is saved and stopped first; best with no players on. Until then `/var` is protected by the daily prune and Ciel's disk alert only. After it: check `/var` use and the Hub. `docker-prune.timer` runs (checked 2026-10-20: last run 2026-10-20 04:40, Result=success; `/var` at 56%) (since 2026-10-18)
- [x] (2026-10-10) Terraria backup timer disabled by setup.sh while the stack is stopped
- [ ] Ethernet instead of Wi-Fi if possible (many retries, no UPS, dead battery) (README memory figures fixed 2026-10-13) (since 2026-10-10)
- [ ] Agent access: agents should run as a scoped `agent` user, not the NOPASSWD `server` (since 2026-10-10)
- [ ] rom's list: see what the Raspberry Pis can run (Ciel hub, Vaultwarden, Pi-hole are candidates) (since 2026-10-12)
- [ ] rom's list: optimize performance on every server and PC (measure first: Ciel charts, milestone 7) (since 2026-10-12)
- [x] (2026-10-14, home-server b4a5f4b, deployed) Hub: status dots through same-origin `/probe/<name>` (a 502 shows down), page renamed Hub; every probe answered live (200, or 403 from Camille and clipboard: up). Red on a stopped app not seen live yet
- [ ] rom's list (2026-10-14): rename the home-server repo (not the machine roberta); rom has no name yet. When picked: GitHub repo, local clone path, `watched-repos.txt`, topic front matter `repos:` and every doc link (since 2026-10-14)
- [x] (2026-10-15, every site done) rom's list (2026-10-14): an icon per site (Ciel, Camille, Mephisto, aether, clipboard, Emile) so Add to Home Screen shows it: a web manifest and an apple-touch-icon in each app; Emile has one; aether has one (aether 02301899, deployed 2026-10-14); Camille has one (camille 9722702, deployed 2026-10-14); Ciel has one (ciel e23f8d7, deployed 2026-10-14); clipboard already ships one (icon-180, manifest); Hub has one (home-server 99f1e08); Mephisto has one (mephisto 7a326f2, deployed 2026-10-15) (since 2026-10-14)
- [x] (2026-10-20, rom: "yes done", tried on the iPhone) rom (2026-10-15): pull down at the top to reload, in every Home Screen app (no address bar there): Hub done (home-server 9145634, standalone mode only; rom to try it on the iPhone); Ciel live (ciel 1629249); Camille live (camille 5818868, in the deploys of 2026-10-16 and later); clipboard live (clipboard 596ba60, not armed while a draft, a send or the Devices sheet is open); Mephisto live (mephisto 7a326f2). Left: rom tries each on the iPhone (since 2026-10-15)
- [x] Apps' own `bin/deploy` scripts open their registry tunnel on a fixed laptop port (5056, 5057): two deploys at once fail (Mephisto, 2026-10-15, worked on retry). Done 2026-10-21: each run picks a free local port, the exit trap closes the tunnel and drops the temporary tag; two tunnels at once tested against roberta. Commits: mephisto 724da70, camille 23582c3, routiner c1c939e, clipboard a642bf5, emile 7d1b60e, pulls f9edf52, ciel 6b02491, aether 6bd3db1b (the aether-courier worktree, branch courier, still has the old script until it merges main)
- [ ] rom: roberta offline since 2026-10-21 11:52 CEST (Log). Someone at home checks: is the laptop on (fans, lights), is the Flybox online (other devices have internet)? Then power it on or reboot it. Once back, an agent reads `last -x | head`, `journalctl --list-boots`, `journalctl -b -1 -n 100` (clean shutdown or cut off mid-log) and `journalctl -b -1 -k | grep -iE 'thermal|mce|oom|wlo|wlan'` and fills in the cause (since 2026-10-21)
- [ ] An alert when roberta itself goes down: Ciel runs on roberta, so nothing alerted on 2026-10-21; needs a watcher outside the house (mikaela, a Pi, or a free uptime service on a playit or tailnet probe) (since 2026-10-21)

## Log

- 2026-09-27: CPU capped at 85% (80 C instead of 93-99 C under load); daily restart and Chunky tuning recorded in the README.
- 2026-10-04: off-site job reshaped after the Drive filled (latest snapshot only, no trash, daily trash empty); see `topics/backups.md`.
- 2026-10-10 inventory (read-only ssh plus the repo): no drift between repo and server in units or stacks. Docker, kernel and container list as in the README.
- 2026-10-10 security audit: LAN-reachable and unprotected: no firewall, `server` has passwordless sudo, sshd on defaults (password login allowed), Dockge on 0.0.0.0:5001 with docker.sock and no auth, cAdvisor privileged, node-exporter :9100, Grafana anonymous; playit uses host networking, so a wrong tunnel could expose anything. Not exposed to the internet today (CGNAT). Detail: `audit-2026-10-10.md` section 9 in the nexus folder.
- 2026-10-10 readiness audit for hosting Ciel and Vaultwarden: resources are enough (31 GB RAM, 1.8 TB data). Needed first: HTTPS proxy, sshd and firewall, auth on admin UIs, backups per stack with secrets kept off the box, the reboot. Single-host data loss risk: restic repos on the same machine, off-site only the latest game snapshot on one 15 GB Drive.
- 2026-10-10 decision: roberta stays shared only temporarily; personal and work will get separate servers, and every fact is keyed by host and owner from now on. A clean split later needs a history-free repo, so personal detail stays out of overseer.
- 2026-10-10: backups get a 25 GB budget (`backup-budget.timer`), Minecraft retention 7 daily, 4
  weekly, 3 monthly; worlds-archive backed up weekly; Prometheus kept as the metrics store Ciel reads,
  capped at 5 GB, its history not backed up; Grafana and Dockge data disposable (Ciel replaces them).
- 2026-10-10: HTTPS proxy (Caddy, *.wuling.dev via Cloudflare DNS-01) and Vaultwarden 1.37.3 at
  vault.wuling.dev (1.33.2 was too old for current clients); sign-ups closed after rom's account.
- 2026-10-10 (home-server accbc67, f3ee7bd): the proxy mounts the `caddy/` folder (a file mount kept the
  old Caddyfile after a push); start page served at wuling.dev, home.wuling.dev only redirects to it
  (decision: one entry point, the apex).
- 2026-10-10 23:45: the backup SSD nvme0 (`/backup`) dropped off the bus (NVMe power-saving
  fault, first time in this boot's journal). 2026-10-11 00:10: rebooted with NVMe power saving off
  (home-server 87bf15d); journal replayed, fsck clean, restic check clean (vaultwarden, archive,
  terraria, minecraft). If it drops again with the option on, suspect the drive itself.
- 2026-10-11 (home-server, "Web apps only through their https names"): Dockge, Grafana, BlueMap
  and the start page publish no port, Caddy reaches them over the `proxy` network; nftables
  `host-firewall.service` drops everything from outside to host processes except ssh and ping
  (node-exporter :9100 closed). Checked from the laptop: all names answer, 80/3000/5001/8100/9100
  closed, 25565/8081/8090 open, Prometheus targets up.
- 2026-10-17 outage, `/var` full: `/var` (37 GB) hit 100% because Docker uses the containerd image
  store, which lives in `/var/lib/containerd` whatever Docker's `data-root` says; untagged images
  and build cache from deploys piled up there (the apps' `bin/deploy` prune runs on the laptop, not
  on roberta). aether's deploy crash-looped 18 times on ENOSPC and every Claude session got 502
  through aether; nothing alerted on the disk. rom pruned 19.9 GB by hand. Fixes 2026-10-18: Ciel
  `disk` check on every device, warn 85%, fail 95% (ciel 23e3738, deployed, all three devices ok);
  `docker-prune.timer` daily 04:40 on roberta (home-server 8747d9e, deployed); `setup.sh` puts the
  store in `/srv/containerd` on a fresh install and `scripts/containerd-move.sh` moves the current
  one (waits for rom's go, Open).
- 2026-10-21 outage: roberta last seen on the tailnet at 11:52 CEST (09:52 UTC); at 12:58 no answer
  over Tailscale (offline, relay par) and its LAN address is not reachable from mikaela (mikaela was
  on another network). No session or deploy touched roberta that day (transcripts, home-server git
  log), and nothing is scheduled near 11:52 (mc-restart is 10:00). Cause not known yet; candidates
  from the facts: a power cut (dead battery, no UPS, so it goes off at once), the Flybox losing 4G
  or its data allowance, roberta's Wi-Fi dropping, or a freeze. Nothing alerted, since Ciel runs on
  roberta. Next steps and the missing alert: Open.
