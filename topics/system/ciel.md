# Ciel (the sysadmin hub)

Owners: product `~/Playground/ciel/SPEC.md`; how it works and the API
`~/Playground/ciel/README.md`; state and decisions `/opt/overseer/topics/ciel.md`; how it runs
on roberta home-server `README.md:201-222`.

## Parts

| Part | Runs where | As | Source |
|---|---|---|---|
| `ciel-hub` | roberta, container `ciel`, stack `stacks/ciel`, data `/srv/data/ciel` (SQLite) | Rust axum + React UI, behind Caddy at https://ciel.wuling.dev | README:10-21, `stacks/ciel/compose.yaml` |
| `ciel-agent run` | every device, root systemd unit `ciel-agent.service`, spool `/var/lib/ciel-agent` | outbound WebSocket to the hub, token `/etc/ciel/token` | README:83-105 |
| `ciel-agent notify` | per user on the laptop, user unit `ciel-notify.service` (dotfiles) | desktop notifications, `$XDG_RUNTIME_DIR/ciel/state.json` for the bar | README:142-156 |
| bar indicator | Quickshell `services/Ciel.qml`, `bar/CielIndicator.qml` (dotfiles) | dot with a count, crossed cloud when the hub is unreachable | README:150-152 |

The hub holds no host privileges: everything that touches a machine goes through that
machine's agent, including roberta (SPEC 4.1). The hub is meant to move to a Raspberry Pi later
(`topics/ciel.md:29-30`).

## What flows into Ciel

| From | How | Becomes | Source |
|---|---|---|---|
| systemd on each device (system, rom, work) | agent follows the journal | issue `<device>-units` or `<device>-<user>-units` with 50 log lines | README:88-92 |
| timer jobs | agent sees "Finished" | job runs, then heartbeat checks | README:93-95, 120-123 |
| Camille (backend and browser) | Sentry DSN, private app | error type and code location only | README:75-81, home-server README:250-251 |
| aether | Sentry format, app `aether`, DSN by container name | issues (e.g. issue 14, RateLimitError) | home-server README:198-199, `topics/aether.md` Open |
| scripts | `ciel-agent report` or plain JSON `/api/v1/events` | issues | README:57-73, 96-98 |
| git commits in watched repos | nier `post-commit` -> `ciel-agent change` | journal entries | README:231-233 |
| `bin/hs push` | `host/setup.sh:239` -> `ciel-agent change --source deploy` | journal entry "home-server deployed" | home-server `host/setup.sh:239` |
| pacman / apt, docker events, unit states | agent | journal entries | README:234-242 |
| host metrics | agent, one sample a minute, kept 31 days | Charts, Devices | README:213-215 |
| Prometheus (roberta) | hub queries `http://prometheus:9090` over the `proxy` network | Charts: server, containers, Minecraft | README:205-209, home-server README:203-205 |
| aether usage | hub reads `http://aether:8080/api/usage/series`, admin key `ciel` in `stacks/ciel/.env` | Charts: AI usage, dev vs `runner-*` | README:210-212 |
| network state | agent reads Wi-Fi, routes, DNS, WireGuard, `tailscale status` | Network card, `network` check | README:218-226, 124-128 |
| web and port checks | hub probes every `*.wuling.dev`, Minecraft port, aether, qBittorrent 8090 | incidents | `topics/ciel.md` Log 2026-10-11, 2026-10-13 |

## What flows out of Ciel

| To | How | Source |
|---|---|---|
| laptop desktop and bar | over the agent link to `ciel-agent notify` | README:144-152 |
| phone | ntfy.sh topic per user; push names only what and where, never the error text | README:163-165; set up (topic saved in Ciel Settings, iPhone app subscribed; `topics/ciel.md` Open, phone alerts) |
| docs repos | `ciel-docs-sync.sh` pulls `/api/journal/day`, `/api/inventory` nightly | README:248-252 |
| agent sessions | `nier context` pulls `/api/brief` | README:253-254 |
| devices | requests over the agent link: stack start/stop/restart, unit run/stop, logs, custom actions, container terminal, rename/remove | README:158-201 |
| devices (binary) | self-update: hub sends its agent checksum; agents download `/download/ciel-agent` and replace themselves | README:107-111 |

## What Ciel trusts and is trusted with

- The hub is as trusted as root on every device: agent self-update was accepted by rom on
  2026-10-11 (`topics/ciel.md:73`). Whoever controls the `ciel` image or the hub controls both
  machines.
- The agent refuses itself, ssh, docker, NetworkManager, the display manager, `systemd-*`,
  `dbus*`, `user@*` and gettys (README:172-175). Ciel's own stack and `proxy` cannot be
  stopped; `minecraft` needs its name typed (`stacks/ciel/compose.yaml`).
- Camille unlock passphrase: typed in the UI, piped by roberta's agent into
  `scripts/camille-box.sh unlock` through `systemd-run`, never stored (README:180-185).
- Terminal into containers: admins only, keystrokes never stored (README:186-191). Decision
  2026-10-12: tailnet plus login are enough (`topics/ciel.md` Log).
- Users: `rom` admin, `work` member; work sees only work and shared (README:49-55, 290-292).
  work has no account yet (`topics/ciel.md:68`).

## Who watches the watcher

- Each agent that cannot reach the hub for 5 minutes, while its own address is in the hub's
  /24, pushes "Ciel hub unreachable" to the ntfy topics it cached (README:153-156).
- So: the laptop away from home (tailnet address, not 192.168.1.x) does not page; roberta's
  own agent cannot page if roberta itself is down. The ntfy topic is set up (Ciel Settings).
  See `failure-modes.md`.
- The laptop's bar shows a crossed cloud when the hub is unreachable (README:152): this is
  today's only signal that roberta is gone.

## Dependencies of Ciel

| Needs | Why |
|---|---|
| roberta, Docker, the `proxy` stack (Caddy) | it runs there, reached only through Caddy |
| registry on roberta | image comes from `127.0.0.1:5000/ciel:latest` |
| Prometheus on the `proxy` network | charts |
| aether | AI usage chart only |
| ntfy.sh | phone alerts |
| Cloudflare DNS + tailnet | agents and browsers find `ciel.wuling.dev` |

## Backup and deploy

- Backup: `ciel-backup.timer` 03:15 daily, consistent SQLite copy, restic `/backup/ciel`, copied
  off-site at 04:30 (home-server README:210-211, `scripts/offsite-backup.sh:29-32`).
- Deploy: `bin/deploy` builds the image on the laptop from the working tree (`--network host`),
  pushes it through an ssh tunnel to roberta's registry, then `docker compose up -d hub`
  (`~/Playground/ciel/bin/deploy`). After a new image, `bin/hs push` copies the agent binary and
  unit to roberta's host (home-server README:212-216); the laptop's agent updates itself, its
  unit file through `sudo setup-machine.sh` (`dotfiles/bootstrap/setup-machine.sh:94-111`).

## Milestones (SPEC 15) and state

1-6 done 2026-10-11; 7 (charts) live 2026-10-13, Grafana retired; 8 read part and network checks
live 2026-10-13; 9 (network act) and 10 (more sources) open (`topics/ciel.md:59-62` and Log).
The README and the topic summary still say "1 to 6" (`findings.md`).
