---
name: system
summary: Wiki of how everything fits together (devices, agents, docs, Ciel, aether, server, backups, failures)
status: active
owner: shared
host: mikaela, roberta
repos: none (map over every repo)
folder: ~/claude/issues/ciel
verified: 2026-10-13
source: read-only audit of every repo, both machines and the docs, 2026-10-13
---

# System wiki: how the infrastructure fits together

Written 2026-10-13 by a read-only audit session. It explains the connections between
the parts. It is not an owner of any fact: every line names the page or file that owns it, so
check there and fix there. rom's personal apps appear here only by footprint; how they plug in is
in rom's own docs.

## The map

```
                         internet (Anthropic, GitHub, Cloudflare DNS, Google Drive, ntfy.sh, playit.gg)
                                   ^                ^                       ^
                                   | 4G (CGNAT)     |                       | playit tunnels
   +-------------------------------+----------------+-----------------------+-----------+
   | home LAN 192.168.1.0/24, Flybox 4G box 192.168.1.1                                 |
   |                                                                                    |
   |  mikaela (laptop, Arch, rom + work)          roberta (server, Debian, 192.168.1.53) |
   |  +------------------------------+   443    +------------------------------------+ |
   |  | Claude Code sessions --------+--------->| proxy (Caddy, *.wuling.dev)        | |
   |  |   nier hooks, rules          |          |   -> aether  (AI accounts proxy) --+-+--> Anthropic
   |  | ciel-agent (root) ===========+== wss ==>|   -> ciel    (hub, SQLite)         | |
   |  | ciel-notify -> bar, desktop  |          |   -> vault, camille, map, portal   | |
   |  | restic (per user) -----------+---------------------------------------------+-+--> Google Drive
   |  | bin/deploy, bin/hs push -----+-- ssh -->| registry 127.0.0.1:5000, git checkout| |
   |  +------------------------------+          | ciel-agent (root), prometheus      | |
   |        |                                   | minecraft, playit, restic -> /backup+-+--> Drive (04:30)
   |        | away from home:                   | tailscaled = subnet router         | |
   |        +--- tailnet (hosted Tailscale) --->+------------------------------------+ |
   +------------------------------------------------------------------------------------+
   iPhone: tailnet node, reaches the same names; ntfy app for Ciel alerts (not set up yet)
```

Read it as three loops:

1. **Work loop**: an agent session on mikaela loads rules from `/opt/nier`, docs from overseer
   and 9s, talks to the model through aether on roberta, and edits repos.
2. **Change loop**: a commit goes to GitHub, is recorded in the Ciel journal by a git hook, and
   reaches roberta by `bin/hs push` (infrastructure) or `bin/deploy` (app images).
3. **Watch loop**: Ciel agents on both machines report failures, timer runs and changes to the
   Ciel hub; the hub alerts on the laptop bar and (once set up) the phone; every night the
   journal flows back into the docs repos.

## Pages

| Page | What it answers |
|---|---|
| [devices-and-network](devices-and-network.md) | Which machines exist, how they reach each other at home and away, names and DNS |
| [agents-and-nier](agents-and-nier.md) | How an agent session is configured, guarded and fed context |
| [docs-system](docs-system.md) | overseer vs 9s, where each kind of fact lives, how docs stay current |
| [ciel](ciel.md) | The sysadmin hub and its agents: what flows in and out, what it trusts |
| [aether](aether.md) | The AI accounts proxy: request path, keys, fallback, its ties to Ciel |
| [home-server-and-apps](home-server-and-apps.md) | roberta's stacks, ports, host units, secrets, deploy |
| [backups-and-recovery](backups-and-recovery.md) | What is backed up where, what is not, recovery order |
| [secrets-and-keys](secrets-and-keys.md) | Where every secret lives (never its value) and what depends on it |
| [change-flow](change-flow.md) | Edit, commit, push, deploy: which hook and which script does what |
| [failure-modes](failure-modes.md) | What breaks when aether, roberta, the tailnet, Ciel or the laptop is gone |
| [findings](findings.md) | Contradictions, stale facts, single points of failure, missing docs |

## Owners of the big facts (start here when something changed)

| Topic | Owning page |
|---|---|
| Agent rules | `/opt/nier/AGENTS.md` (also `/etc/claude-code/CLAUDE.md`) |
| Laptop facts | `/opt/overseer/machines/mikaela.md` |
| Server facts | `/opt/overseer/machines/roberta.md` |
| Server how-to | `~/Desktop/perso/projects/home-server/README.md` |
| Ciel how-to / product | `~/Playground/ciel/README.md`, `SPEC.md`; state: `overseer/topics/ciel.md` |
| aether how-to / state | `~/Playground/aether/README.md`; state: `overseer/topics/aether.md` |
| Backups how-to / state | `/opt/dotfiles/backup/README.md`; state: `overseer/topics/backups.md` |
| Network state | `overseer/topics/network.md` |
| Docs rules / state | `/opt/nier/AGENTS.md` section 2; `overseer/topics/docs.md` |
| rom's apps | rom's own docs (not linked from here) |

## Ten connections a newcomer must understand

1. Every `*.wuling.dev` name is a public Cloudflare DNS record pointing at the LAN address
   192.168.1.53 (`home-server/stacks/proxy/hosts.txt`). At home it is reached directly; away,
   only through the tailnet, where roberta routes 192.168.1.0/24. One Caddy is the only door.
2. Every agent session goes through aether on roberta (`/opt/nier/settings.json:156-160`).
   AI work therefore depends on roberta, its 4G uplink, and the tailnet when away. The fish
   `claude` wrapper falls back to the account when aether is down (dotfiles
   `config/fish/conf.d/claude-aether.fish`).
3. Ciel agents run as root on each device, dial out to the hub, report everything, run what
   the hub asks within an allowlist, and update themselves from the hub (Ciel README 83-114).
   The hub is as trusted as root on both machines.
4. Two deploy paths: `bin/hs push` (git into `/srv/home-server`, then `host/setup.sh`) for
   the server's definition; `bin/deploy` in each app repo (Ciel, aether, Camille) for app
   images, built on the laptop and pushed through an ssh tunnel to roberta's registry.
5. Commits in watched repos feed the Ciel journal (nier `post-commit` hook), and rom's nightly
   `ciel-docs-sync.timer` writes the journal and inventory back into overseer and 9s.
6. `nier context` at every session start injects the machine page, docs indexes, TODO heads and
   a live Ciel brief (`/opt/nier/bin/nier:100-112`): agents start from the docs, so stale docs
   mislead every session.
7. The owner model rom / work / shared decides the docs repo (overseer or 9s), Ciel visibility,
   journal routing (`/opt/nier/bootstrap/watched-repos.txt` column 2) and stack ownership.
8. `/opt/dotfiles` and `/opt/nier` are shared by both laptop users through symlinks: one edit
   changes both; root copies (`/etc/claude-code`, `/etc/...`) need `sudo setup-machine.sh`.
9. Private media is protected in five layers: the guard hook and deny rules, `CIEL_REDACT` at
   the agents, Ciel's private-app flag, `backup/never.txt`, and the docs rules.
10. Backups: laptop restic per user and the server's nightly off-site copy share one 15 GB
    Google Drive. Camille and aether are never backed up. The laptop restic password exists only
    with rom and opens the copy of the server's secrets.

## Open

Decisions on the audit's four HIGH findings (rom, 2026-10-13), each kept on its owner's page:
- [ ] 1, roberta down goes unnoticed: Ciel moves to its own box later and watches roberta
  (`topics/ciel.md`, "Dedicated Ciel box") (since 2026-10-13)
- [x] (2026-10-13, rom) 2, AI work depends on roberta and the uplink: accepted
- [ ] 3, one remembered password: restore rehearsal later (`topics/backups.md`) (since 2026-10-13)
- [ ] 4, stale copy of roberta's secrets: later, with one encrypted store for every secret and
  ssh key (`topics/setup-steps.md`, "secrets restore") (since 2026-10-13)
