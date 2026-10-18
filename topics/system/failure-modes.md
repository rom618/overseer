# Failure modes: what breaks when X is down

Derived from the connections on the other pages. Each row names the mechanism and its source.
"Not tested" marks reasoning that no page or check has confirmed.

## aether down (container stopped, bad deploy, crash)

| Effect | Why | Source |
|---|---|---|
| New sessions from fish go straight to the account; others fail to authenticate | fish `claude` checks `/health` (2 s) | dotfiles `conf.d/claude-aether.fish:2-9` |
| Running sessions wait and retry | `CLAUDE_CODE_RETRY_WATCHDOG=1` | `nier/settings.json:158` |
| `claude -p` runners, sessions from bash or a timer fail | no fallback outside fish | `claude-aether.fish` is a fish function |
| Status line and bar show no pooled limits; keepalive guard reads stale data | they read `/api/statusline` | `nier/tools/claude-statusline.py:7-8` |
| Ciel AI usage chart empty | `CIEL_AETHER` | Ciel README:210-212 |
| Ciel opens an incident | web check on aether.wuling.dev | `topics/ciel.md` Log 2026-10-13 |
| Nothing else | aether is a leaf for everything but AI | |

## roberta down (power cut, Wi-Fi drop, crash, reboot)

| Effect | Why | Source |
|---|---|---|
| All `*.wuling.dev` apps gone: Ciel, aether, Vaultwarden, Camille, map, start page | one host, one Caddy | home-server README:135-147 |
| Agent sessions: as "aether down" above | aether runs there | |
| No alerts at all, except the laptop bar's crossed cloud | the hub is down; roberta's own agent is down; the laptop agent pages ntfy only on the hub's /24 | Ciel README:152-156 |
| Laptop agents spool; nothing is lost | local SQLite spool | Ciel README:99-101 |
| Laptop backups keep working | they go to Google Drive directly | `backup/README.md:1-4` |
| Vaultwarden clients work offline with their cache; no sync, no new device login | server down | Bitwarden client behavior, not tested here |
| After it comes back: Camille stays locked and down until rom unlocks it | LUKS box | rom's Camille docs |
| After a dirty stop, Minecraft may lose the last minutes | `docker.service` timeout 90 s vs 3 min | `machines/roberta.md:62-65` |
| ciel-docs-sync exits 75 and retries every 30 min; missed days caught up | | `nier/tools/ciel-docs-sync.sh:6-7` |
| Session context has no Ciel brief | 4 s timeout, silent | `nier/bin/nier:104-111` |

No UPS and a dead battery (`machines/roberta.md:15-16`): a short power cut is a full outage and a
Camille unlock.

## Home internet (Flybox 4G) down, LAN up

| Effect | Why | Source |
|---|---|---|
| aether cannot reach the provider: all AI work stops, fallback too | both paths need the internet | |
| Name lookups for `*.wuling.dev` may fail at home once caches expire | names are public Cloudflare records, no local DNS | `stacks/proxy/hosts.txt`, `topics/network.md:70, 76`; not tested |
| Apps by IP still work (qBittorrent 8090) | LAN | |
| Ciel and Ciel agents keep working if names are cached | LAN | not tested |
| Off-site backups and laptop backups fail; Ciel turns them into issues | Drive unreachable | |
| Players cannot join | playit tunnels | home-server README:119-126 |

## Tailnet down (Tailscale coordination, or the laptop logged out)

| Effect | Why | Source |
|---|---|---|
| At home: nothing, the laptop uses the LAN; the home route is off on the home Wi-Fi | NM hook | dotfiles `system/README.md:24-29` |
| Away: no app, no aether (fish fallback to the account works), no Ciel; agent spools | names point at a LAN address | home-server README:164-169 |
| Ciel `network` check fails when a node needs a login | | Ciel README:124-128 |
| Existing direct connections often survive a short coordination outage | Tailscale design | not tested here |
| work's job tailnets unaffected | separate daemons | `machines/mikaela.md:109` |

## Ciel down (hub container), roberta up

| Effect | Why | Source |
|---|---|---|
| No issues, alerts, journal, charts, stack controls, Camille unlock from the UI | | Ciel README |
| Agents spool events and changes; replayed in order later | | Ciel README:99-101 |
| Agents on the hub's LAN push "Ciel hub unreachable" after 5 min, if an ntfy topic exists | | Ciel README:153-156 |
| Bar shows the crossed cloud | | Ciel README:152 |
| Docs feed and session brief pause | | `ciel-docs-sync.sh:31-34`, `bin/nier:104-111` |
| Camille can still be unlocked with `bin/hs camille-unlock` | ssh path | home-server README:237-240 |
| Everything else runs | Ciel controls nothing by itself | SPEC 4.1 |

## Laptop away (travel, sleeping, off)

| Effect | Why | Source |
|---|---|---|
| Away but online: works through the tailnet; Surfshark and Tailscale coexist (direct connection seen) | | `machines/mikaela.md:109` |
| Laptop off: Ciel marks the device and its heartbeats "unknown", not failing | `device away` | Ciel README:121-123 |
| Laptop off: nobody can deploy, run `bin/hs`, or unlock Camille except through the Ciel UI from the phone | all tools and the ssh key live on the laptop | `home-server-and-apps.md` |
| Laptop off: rom's docs sync and laptop backups wait | user timers | |
| roberta failure while away: no alert reaches rom | see roberta down | |
| Laptop lost: restore from Drive needs the restic password from memory | | `backups-and-recovery.md` |

## Other single points

| If this fails | Breaks | Source |
|---|---|---|
| Google Drive account (full, locked out) | every off-site copy and laptop backup; modpack releases | `topics/backups.md:51-54, 78` |
| Cloudflare token expired or revoked | certificate renewal, then every https name | home-server README:137-139 |
| GitHub | pushes fail; ciel-docs-sync commits wait; nothing live breaks | |
| roberta's backup SSD | all local server backups (it dropped once) | `topics/home-server.md:81-84` |
| `server` user's ssh key on the laptop | every deploy and `bin/hs` | `bin/hs:4` |
| The `proxy` stack | every web app; Ciel cannot stop it on purpose | `stacks/ciel/compose.yaml` |
| roberta's `/var` full (containerd image store, until moved to `/srv`) | image pulls and container starts: aether crash-looped, every agent session got 502 (2026-10-17); Ciel's `disk` check warns at 85% | `topics/home-server.md` Log 2026-10-17 |
| The registry stack | no new deploys of ciel, aether, camille; running containers unaffected | `ciel/bin/deploy` |
| Surfshark on the laptop with Docker builds | builds stall unless `--network host` | `topics/network.md:37-40` |
| A kernel update without reboot (laptop) | Surfshark and Docker | `topics/network.md:29-33` |
