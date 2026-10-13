# Home server (roberta) and its apps

Owners: machine facts `/opt/overseer/machines/roberta.md`; how to run, deploy, restore
`~/Desktop/perso/projects/home-server/README.md`; state and decisions
`/opt/overseer/topics/home-server.md`. Personal apps on it (Camille, Minecraft) are described
only by footprint here; the rest is in rom's own docs.

## The repo is the server

- `home-server` (rom618/home-server) defines the host and every stack; nothing is configured by
  hand (`machines/roberta.md:21-22`). Checkout on the server: `/srv/home-server`.
- `bin/hs push`: git push into the server's checked-out branch (`receive.denyCurrentBranch
  updateInstead`), then `sudo host/setup.sh` (idempotent: timezone, swap, sleep, Docker, updates,
  folders, generated secrets, restic repos, timers, firewall, Tailscale, Ciel agent)
  (README:37-52, `bin/hs:45-47`). It refuses if files were edited on the server.
- `setup.sh` reports each applied commit to Ciel's journal (`host/setup.sh:239`).
- Audit 2026-10-13: the server checkout is at `67b2460`, the same as the laptop repo head.

## Stacks

| Stack | Containers (live 2026-10-13) | Reached as | Data | Backup | Owner |
|---|---|---|---|---|---|
| proxy | caddy | :443 on LAN and tailnet, `*.wuling.dev` | `/srv/data/proxy` (cert keys, private) | none needed (certs re-issue) | shared |
| portal | portal (nginx) | https://wuling.dev, home redirects | repo | repo | shared |
| ciel | ciel | https://ciel.wuling.dev | `/srv/data/ciel` | daily + off-site | shared |
| aether | aether | https://aether.wuling.dev | `/srv/data/aether` | none (decided) | shared |
| vaultwarden | vaultwarden 1.37.3 | https://vault.wuling.dev | `/srv/data/vaultwarden` 0700 | daily 03:00 + off-site | shared |
| monitoring | prometheus, node-exporter, cadvisor, mc-monitor | no port; Ciel reads Prometheus | Prometheus 365 d | none (disposable) | shared |
| registry | registry 3.0.0 | 127.0.0.1:5000 only, via ssh tunnel | images | none | shared |
| bluemap | bluemap | https://map.wuling.dev | world renders | none | rom (game) |
| minecraft | minecraft, minecraft-backup | :25565 tcp/udp, 24454/udp, playit tunnel | `/srv/data/minecraft` | restic 30 min + latest off-site | rom (game) |
| playit | playit (host network) | outbound tunnels | key in `.env` | none | rom (game) |
| camille | camille, camille-vpn, camille-qbittorrent, camille-pot-provider | https://camille.wuling.dev, qBittorrent :8090 and qbit.wuling.dev | `/srv/data/camille` (private LUKS box) | never (rom's rule) | rom |
| terraria | (stopped) | playit tunnel when running | `/srv/data/terraria` | repo synced off-site | rom (game) |

Sources: `docker ps` on roberta 2026-10-13; README:6-19, 135-160, 186-253;
`stacks/proxy/hosts.txt`; `stacks/proxy/caddy/Caddyfile:8-48`; `scripts/offsite-backup.sh`.

## Ports and the network inside the box

- Open from the LAN and tailnet: 22 (ssh), 443 (Caddy), 25565 tcp+udp and 24454/udp
  (Minecraft), 8090 (qBittorrent through the VPN container). Docker-published ports bypass the
  host firewall; `host-firewall.service` (`host/firewall.nft`) lets only ssh and ping reach host
  processes (README:150-154).
- Web apps publish no port. They join the external docker network `proxy` and Caddy reaches
  them by container name (README:145-147). The same network carries ciel, aether, camille,
  vaultwarden, bluemap, portal and Prometheus, so these containers can reach each other by name.
- Internal links on `proxy`: Ciel to `prometheus:9090` and `aether:8080`; aether's Ciel DSN is
  rewritten to Ciel's container name (README:198-205); Caddy to `192.168.1.53:8090` for qbit.
- Leftover docker networks seen 2026-10-13: `dockge_default`, `pregen_default` (Open disk
  clean-up, `topics/home-server.md:58`).

## Host units (`host/systemd/`)

| Unit | When | Does |
|---|---|---|
| `vaultwarden-backup.timer` | 03:00 daily | SQLite copy to restic `/backup/vaultwarden` |
| `ciel-backup.timer` | 03:15 daily | SQLite copy to restic `/backup/ciel` |
| `archive-backup.timer` | Sun 03:30 | worlds archive to `/backup/archive` |
| `offsite-backup.timer` | 04:30 daily | empties Drive trash; copies latest Minecraft snapshot, Vaultwarden, Ciel, archive, Terraria repos to `gdrive:backups/home-server` |
| `backup-budget.timer` | Sun 05:30 | 25 GB budget on `/backup` |
| `mc-restart.timer` | 09:55, restart 10:00 | daily Minecraft restart with warnings |
| `terraria-backup.timer` | disabled | stack stopped |
| `cpu-cap`, `gpu-metrics`, `host-firewall` | services | 85% CPU cap, GPU textfile metrics, nftables |
| `ciel-agent.service` | always | Ciel agent (root), binary copied out of the `ciel` container |
| `tailscaled` | always | subnet router for 192.168.1.0/24 |

Source: `systemctl list-timers` on roberta 2026-10-13; README:12-13, 75-83, 155-158, 210-216.

## Where roberta's secrets live (names only)

| Secret | Location | Made by |
|---|---|---|
| per-stack settings and keys | `stacks/*/.env` on the server, 0600, not in git | `setup.sh` or `bin/hs <x>-key` |
| restic password (all server repos) | `/etc/home-server/restic.pw` | `setup.sh`; copy on the laptop `~/.config/home-server/` |
| rclone config for Drive | `/etc/home-server/rclone.conf` | by hand |
| Cloudflare DNS token | `/etc/home-server/cloudflare-api-token`, root only | by hand |
| Ciel agent token | `/etc/ciel/token`, 0600 root | `ciel-hub device add roberta shared` |
| Camille box passphrase | only with rom | never stored |

Source: README:19, 54-58, 137-139, 212-216; `scripts/offsite-backup.sh:3, 13`;
`topics/home-server.md:32-34`. More in `secrets-and-keys.md`.

## Resources and limits

- 31 GB RAM, about 21 GB used (Minecraft heap 16 GB, cap 20 GB); `/srv` 1.8 TB, 64% used;
  `/backup` 117 GB, 12% used (`free`, `df` on 2026-10-13; README:62-63).
- Wi-Fi only, dead battery, no UPS (`machines/roberta.md:15-16`). A power cut stops everything,
  and Camille comes back locked.
- Reboot gotcha: `docker.service` stops after 90 s but Minecraft needs 3 min: save and stop it
  by hand first, then `bin/hs up minecraft` (`machines/roberta.md:62-65`).
- The backup SSD dropped off the bus once with NVMe power saving on; now off
  (`topics/home-server.md:81-84`).

## How the laptop operates the server

| Tool | Path | Uses |
|---|---|---|
| `bin/hs` | home-server repo, runs on the laptop | ssh alias `roberta`, key `id_ed25519_roberta` (`bin/hs:4`) |
| `bin/deploy` | ciel, aether, camille repos | ssh tunnel to `127.0.0.1:5000`, then `docker compose up -d` |
| `scripts/deploy.sh` | mc-modpack | rsync to `/srv/staging/minecraft`, then `scripts/mc-update.sh` |
| Ciel UI | browser | start/stop/restart, logs, terminal, Camille unlock, through roberta's agent |

All of these need the laptop's ssh key except the Ciel UI. Agents on roberta itself are not set
up: no docs checkout, no scoped `agent` user (`topics/home-server.md:61`, `topics/docs.md:73`).
