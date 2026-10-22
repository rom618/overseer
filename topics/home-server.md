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

- [x] (2026-10-21, home-server c4c4c76, takes effect on the next `hs push`) Bug (2026-10-15): `hs push` silently reverted any secret set on the box since the last `hs secrets-pull` (`host/secrets.py apply` wrote `secrets/roberta.yaml` over the `.env`); it undid rom's Mephisto password reset (9s mephisto). Fix: `apply` keeps salted hashes of the values it last applied on the box; changed on the box only: kept and named (pull it); in the file only: written; in both, or different with no record yet: nothing written, the deploy stops naming the keys (no values). So the first push with it stops if the box drifted: `hs secrets-pull`, commit, push. The secret-writing `hs` commands still do not pull by themselves (no longer needed for safety). How it works: home-server README, Secrets
- [ ] rom's list: stacks to add: Forgejo (GitHub replica, TODO), Pi-hole (network topic) and two of rom's own apps (9s); rom's own apps get French names (docs topic, naming) (since 2026-10-13)
- [ ] rom's list: auto-deploy CI once the local forge (Forgejo, GitLab or GitHub runners) is up: a push to `main` of home-server, Ciel and aether deploys it on roberta (today by hand: `bin/hs push`, `bin/deploy`), so the runner is never behind its repo (2026-10-17: home-server f236230, the burn runner's stop check, waited for a manual `bin/hs push`) (since 2026-10-17)
- [ ] Split into personal and work servers (roberta shared only for now): own restic password, repo, Drive account and proxy; parameterize `offsite-backup.sh` (since 2026-10-10)
- [ ] Hardening left on purpose: sudo NOPASSWD until the scoped agent user exists (the docker group is
  root-equivalent anyway, decision 2026-10-11); cAdvisor privileged (no port, Ciel replaces it) (since 2026-10-11)
- [x] (2026-10-11, camille session) Camille 8081 closed, AirPlay over https
- [ ] Docker repo in unattended-upgrades or a monthly upgrade; pin image versions (2026-10-13: monitoring stack pinned, newest releases at least two weeks old, no daily pulls; left: bluemap (pinning means a downgrade from 5.29), mc-backup in the minecraft stack and the pot provider in camille's, not touched at night) and prune old ones (since 2026-10-10)
- [ ] Disk clean-up (2026-10-13: 15.8 GB freed, untagged images left by deploys, two old tModLoader tags, Vaultwarden 1.33.2, build cache over 24 h; the `/var` part is done, see Log 2026-10-17): left: `/srv/staging`, leftover networks, logs, a disk budget per data dir (since 2026-10-10)
- [x] (2026-10-21, rom's go, agent ran it at 19:2x CEST with 0 players: `/etc/containerd/config.toml` root = `/srv/containerd`, `/var` 56% to 4%, 30 of 30 containers back, aether health ok) rom: go for the one-time move of containerd's image store to `/srv` (`scripts/containerd-move.sh`, run detached as its header says): every container stops for a few minutes, aether too (running agent sessions wait and retry), Minecraft is saved and stopped first; best with no players on. Until then `/var` is protected by the daily prune and Ciel's disk alert only. After it: check `/var` use and the Hub. `docker-prune.timer` runs (checked 2026-10-20: last run 2026-10-20 04:40, Result=success; `/var` at 56%) (since 2026-10-18)
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
- [x] (2026-10-22, cause found, see Log) rom: roberta offline since 2026-10-21 11:52 CEST: the Flybox lost its internet; roberta stayed on
- [x] (2026-10-22, rom restarted it with the power button at 21:49, cause found, see Log) roberta offline 2026-10-22 20:56 to 21:51 CEST: its Wi-Fi card dropped and never reconnected
- [ ] Wi-Fi self-heal on roberta: when `wlp3s0` has had no connection for about 5 minutes, reload `ath10k_pci` and reconnect, and reboot as a last resort; the 2026-10-22 outage needed someone at the laptop (proposed 2026-10-22, waits for rom's yes; Ethernet stays the real fix, item above) (since 2026-10-22)
- [ ] rom: unlock Camille after the 2026-10-22 reboot (Ciel, Stacks, camille, Unlock, or `hs camille-unlock`); Camille, Mephisto, qbittorrent and prowlarr wait on it (since 2026-10-22)
- [ ] rom: write down where the console password of `server` on roberta lives (rom did not remember it on 2026-10-22, roberta off the network); it must be reachable while roberta is down, so not only in Vaultwarden on roberta itself (since 2026-10-22)
- [ ] An alert when roberta itself goes down or loses internet: Ciel runs on roberta, so nothing alerted on 2026-10-21 or 2026-10-22 (the house's internet was down, so Ciel could not notify either); needs a watcher outside the house (mikaela, a Pi, or a free uptime service on a playit or tailnet probe) (since 2026-10-21)

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
  log), and nothing is scheduled near 11:52 (mc-restart is 10:00). Cause (found 2026-10-22): the
  Flybox lost its internet (4G or data allowance) from 11:52 to about 17:21 CEST, 5.5 hours.
  roberta stayed on (uptime 11 days, same boot) and its Wi-Fi stayed associated, with LAN traffic
  all along (Prometheus `node_network_carrier` 1 on wlp3s0); playit timed out from 09:52 to 15:21
  UTC, and aether failed every token refresh on DNS timeouts (Ciel issue 31, resolved). Fix: none
  needed on roberta. Nothing alerted, since Ciel runs on roberta and could not reach out; the
  outside watcher is still Open.
- 2026-10-22 outage: roberta last seen on the tailnet at 20:56 CEST (18:56 UTC); at 21:45 no
  answer on ssh, 443 or 25565, and from mikaela on the same LAN its ARP entry stays INCOMPLETE, so
  it is off the network entirely (unlike 2026-10-21: the Flybox and the internet were up, gateway
  and google.com answered). No session, deploy or home-server commit touched it after 18:32 CEST.
  Cause (previous boot's journal): at 20:56:20 the Wi-Fi card (Qualcomm QCA6174, driver
  `ath10k_pci`) lost the Flybox's beacons, disconnected, and from then on every reconnect timed
  out ("association took too long") while mikaela stayed connected to the same Flybox: the card or
  driver got stuck, not the Flybox. The system itself kept running (Docker, ciel-agent logging)
  until rom pressed the power button at 21:49, which shut it down cleanly; it came back at 21:51
  with Wi-Fi up and every stack running, except the four that need the Camille box unlocked
  (camille, mephisto, camille-qbittorrent, camille-prowlarr). Down 55 minutes. Fix: the reboot;
  nothing on the box reconnects a stuck card by itself yet (self-heal and Ethernet in Open).
  Nothing alerted (outside watcher still Open).
