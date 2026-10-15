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
- Memory (2026-10-10 night): 22 of 31 GB used, of which Minecraft 18.3 GB (16 GB heap, 20 GB
  limit) and BlueMap 2.3 GB; the Camille stack about 0.4 GB.

## Open

- [ ] rom's list: stacks to add: Forgejo (GitHub replica, TODO), Pi-hole (network topic) and two of rom's own apps (9s); rom's own apps get French names (docs topic, naming) (since 2026-10-13)
- [ ] Split into personal and work servers (roberta shared only for now): own restic password, repo, Drive account and proxy; parameterize `offsite-backup.sh` (since 2026-10-10)
- [ ] Hardening left on purpose: sudo NOPASSWD until the scoped agent user exists (the docker group is
  root-equivalent anyway, decision 2026-10-11); cAdvisor privileged (no port, Ciel replaces it) (since 2026-10-11)
- [x] (2026-10-11, camille session) Camille 8081 closed, AirPlay over https
- [ ] Docker repo in unattended-upgrades or a monthly upgrade; pin image versions (2026-10-13: monitoring stack pinned, newest releases at least two weeks old, no daily pulls; left: bluemap (pinning means a downgrade from 5.29), mc-backup in the minecraft stack and the pot provider in camille's, not touched at night) and prune old ones (since 2026-10-10)
- [ ] Disk clean-up (2026-10-13: 15.8 GB freed, untagged images left by deploys, two old tModLoader tags, Vaultwarden 1.33.2, build cache over 24 h; deploy scripts now prune by themselves; left: /srv/staging, networks, logs, a disk budget): old Docker images (about 5 GB of old tModLoader tags), `/srv/staging`, leftover networks, logs; a disk budget per data dir watched by Ciel (since 2026-10-10)
- [x] (2026-10-10) Terraria backup timer disabled by setup.sh while the stack is stopped
- [ ] Ethernet instead of Wi-Fi if possible (many retries, no UPS, dead battery) (README memory figures fixed 2026-10-13) (since 2026-10-10)
- [ ] Agent access: agents should run as a scoped `agent` user, not the NOPASSWD `server` (since 2026-10-10)
- [ ] rom's list: see what the Raspberry Pis can run (Ciel hub, Vaultwarden, Pi-hole are candidates) (since 2026-10-12)
- [ ] rom's list: optimize performance on every server and PC (measure first: Ciel charts, milestone 7) (since 2026-10-12)
- [x] (2026-10-14, home-server b4a5f4b, deployed) Hub: status dots through same-origin `/probe/<name>` (a 502 shows down), page renamed Hub; every probe answered live (200, or 403 from Camille and clipboard: up). Red on a stopped app not seen live yet
- [ ] rom's list (2026-10-14): rename the home-server repo (not the machine roberta); rom has no name yet. When picked: GitHub repo, local clone path, `watched-repos.txt`, topic front matter `repos:` and every doc link (since 2026-10-14)
- [ ] rom's list (2026-10-14): an icon per site (Ciel, Camille, Mephisto, aether, clipboard, Emile) so Add to Home Screen shows it: a web manifest and an apple-touch-icon in each app; Emile has one; aether has one (aether 02301899, deployed 2026-10-14); Camille has one (camille 9722702, deployed 2026-10-14); Ciel has one (ciel e23f8d7, deployed 2026-10-14); clipboard already ships one (icon-180, manifest); Hub has one (home-server 99f1e08); left: Mephisto (sent to its session 2026-10-15) (since 2026-10-14)
- [ ] rom (2026-10-15): pull down at the top to reload, in every Home Screen app (no address bar there): Hub done (home-server 9145634, standalone mode only; rom to try it on the iPhone); Ciel live (ciel 1629249); Camille built (camille 5818868), deploy waiting for rom's "deploy" in its session; sent to the clipboard and Mephisto sessions (since 2026-10-15)

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
