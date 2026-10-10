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

MSI GT72S laptop running headless Debian 13, `server@192.168.1.53` (key `~/.ssh/id_ed25519_server`
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

## Layout

Stacks under `stacks/` (Docker Compose): minecraft (+ minecraft-backup), playit (public tunnels),
bluemap :8100, portal :80, dockge :5001, monitoring (prometheus internal :9090, grafana :3000,
node-exporter :9100, cadvisor, mc-monitor), terraria (stopped). Data `/srv/data/<stack>` (1.8 TB),
local restic repos `/backup/<stack>` (117 GB), deployed checkout `/srv/home-server`. Host units
from `host/systemd/`: cpu-cap, gpu-metrics, mc-restart (daily 10:00), offsite-backup (04:30,
latest Minecraft snapshot + Terraria to `gdrive:backups/home-server`), terraria-backup.

## Open

Hardening and backups before hosting Ciel or Vaultwarden: see `overseer/TODO.md` (Servers).
