---
host: roberta
machine_id: a808ebffaf5e4039b4bc78bee0501167
os: debian
role: server
owner: shared
users: server (sudo NOPASSWD, docker)
verified: 2026-10-10
source: ~/Desktop/perso/projects/home-server (README, stacks/, host/), read-only ssh audit 2026-10-10
---

# roberta (home server)

MSI GT72S laptop running headless Debian 13, `server@192.168.1.53` (key `~/.ssh/id_ed25519_roberta` (ssh alias `roberta`; passphrase from the login keyring through the agent)
on mikaela), i7-6820HK, 31 GB RAM (about 22 GB used, mostly the Minecraft heap), Wi-Fi only
(Ethernet down), dead battery (no UPS). Shared by personal and work use only for now; separate
personal and work servers will replace it.

## Hard rules on this device

- Defined entirely by the repo `~/Desktop/perso/projects/home-server` (rom618/home-server):
  edit there, deploy with `bin/hs push`; never hand-edit the server.
- Never wipe the live Minecraft world (`/srv/data/minecraft/world`).
- The Minecraft restic repo is used only through `docker exec minecraft-backup restic`.
- No `/locate` on the live Minecraft server with players online (10 s freeze).
- `server` has passwordless sudo and docker: anything run there is root. Ask before any change.
- ssh: keys only, no root login, only `server` (`host/setup.sh`, since 2026-10-10). A change to
  sshd settings goes through setup.sh, which reverts it after 5 minutes unless a new login confirms.
- Private, never read, listed, probed or backed up, by any route (ssh, `docker exec`, the app's API):
  everything under `/srv/data/camille` (Camille's 1 TB LUKS container `box.luks`, its mount point
  `box` with media and app data, and Camille's local CA in `caddy`), plus the proxy's certificate
  keys under `/srv/data/proxy`. The guard blocks commands that name the Camille folder.
- HTTPS front door: the `proxy` stack (Caddy, port 443, Let's Encrypt wildcard `*.wuling.dev` by
  DNS challenge; token root-only in `/etc/home-server/cloudflare-api-token`). Names and their DNS
  records: `stacks/proxy/hosts.txt` + `caddy/Caddyfile`, records kept by `scripts/dns-records.sh`
  (DNS-only, LAN address 192.168.1.53). Today: wuling.dev (start page), map, grafana, dockge,
  vault, camille; home redirects to wuling.dev.

## Layout

Stacks under `stacks/` (Docker Compose): minecraft (+ minecraft-backup), playit (public tunnels),
bluemap, portal, dockge, monitoring (prometheus, grafana, node-exporter, cadvisor, mc-monitor),
vaultwarden, camille, proxy, registry, terraria (stopped). Open to the LAN: 22, 443, Minecraft
25565 and 24454/udp, qBittorrent 8090; everything else is closed (no published
port, plus `host-firewall.service` for host processes). Data `/srv/data/<stack>` (1.8 TB),
local restic repos `/backup/<stack>` (117 GB), deployed checkout `/srv/home-server`. Host units
from `host/systemd/`: cpu-cap, gpu-metrics, mc-restart (daily 10:00), offsite-backup (04:30,
latest Minecraft snapshot + Terraria to `gdrive:backups/home-server`), terraria-backup (disabled by
setup.sh while the Terraria stack is stopped), vaultwarden-backup, ciel-backup, backup-budget.
`ciel-agent.service` (root, since 2026-10-11) reports failed services, timer runs and changes to Ciel,
and runs what Ciel asks within its allowlist (stack start, stop, restart and logs; Camille's unlock from
`/etc/ciel/actions.json`); its binary and unit come out of the `ciel` container (home-server README, Ciel section).
Tailscale (since 2026-10-12, `setup.sh`): `tailscaled` announces the home network 192.168.1.0/24 to the
own tailnet (subnet router), takes no DNS and no routes from it, Tailscale ssh off; how to join and
check: home-server README "Remote access". Joined 2026-10-12 as `roberta`, 100.75.27.34, route approved,
key expiry off (`overseer/topics/network.md`).
Editor: vim is installed and is the default `editor` (setup.sh, since 2026-10-11); nano is still there.

Gotcha: the Flybox resolver answers names with a NAT64 address (`64:ff9b::...`) first, which
cannot reach the LAN; clients must try IPv4 first (curl does, the Ciel agent does since 2026-10-11).

Gotcha: `docker.service` stops with `TimeoutStopSec` 90 s, but the minecraft container's
`stop_grace_period` is 3 min. Before a reboot or a Docker upgrade, `save-all flush` over RCON and stop
the minecraft stack by hand; afterwards `bin/hs up minecraft` (`unless-stopped` does not restart a
container stopped by hand).

## Open

Hardening and backups before hosting Ciel or Vaultwarden: see `overseer/TODO.md` (Servers).
