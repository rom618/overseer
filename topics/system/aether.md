# aether (the AI accounts proxy)

Owners: how it works `~/Playground/aether/README.md:1-29`; state, decisions, ports from the
company fork `/opt/overseer/topics/aether.md`; how it runs on roberta home-server
`README.md:186-199`.

## What it is

A fork of ccflare (Bun + TypeScript, one server process, SQLite, dashboard). It sits between
every agent and the provider: it picks one of the linked accounts (rom's and work's), inserts
that account's credential and forwards the request unchanged. Routing strategy `session`: stay
on one account for its 5 h window, move when limited (`topics/aether.md:15-31`, Open
"Routing stays `session`").

## Request path

```
claude (mikaela)                         roberta                              provider
  settings.json env                      Caddy :443                          
  ANTHROPIC_BASE_URL =  --- https --->   aether.wuling.dev -> aether:8080 --> api.anthropic.com
  https://aether.wuling.dev/v1/claude-code     (proxy network)   picks account,     (over 4G)
  apiKeyHelper -> ~/.config/aether/key                           adds its token
```

- Client key per device and user (`mikaela-rom` today), made by `bin/hs aether-key-save NAME` in
  the home-server repo, written to `~/.config/aether/key` (0600) without being shown (home-server
  README:192-196).
- `CCFLARE_REQUIRE_AUTH=true` in `stacks/aether/compose.yaml`: nothing answers without a key.
- Reachable at home and on the tailnet only: the name points at the LAN address
  (`topics/aether.md` Open, 2026-10-12).
- The bare host answers "model does not exist"; the `/v1/claude-code` prefix is required
  (`topics/aether.md` Open, first device key).

## Data and secrets

| What | Where | Note |
|---|---|---|
| Provider account tokens (OAuth) | `/srv/data/aether` SQLite, 0700 | clear text in the file (`topics/aether.md:31`) |
| Client keys | same database; each user's copy in `~/.config/aether/key` | rom has one, work does not (`topics/aether.md` Open) |
| Admin key `ciel` | `stacks/ciel/.env` on roberta | Ciel reads usage (home-server README:203-205) |
| Ciel DSN | `stacks/aether/.env` on roberta | made by `bin/hs aether-dsn` |
| Backup | none | decided by rom 2026-10-12: losing it costs two logins and the usage history (`topics/aether.md` Open) |

## Who uses aether

| Consumer | What | Source |
|---|---|---|
| every Claude Code session (shared nier settings) | model calls | `/opt/nier/settings.json:156-160` |
| status line and cache-keepalive guard | `/api/statusline`: 5h and 7d over the pooled accounts | `nier/tools/claude-statusline.py:7-8`, `tools/aether_status.py` |
| desktop bar Claude icon and popup | cost today, usage left per account | dotfiles `quickshell/ii/services/ClaudeUsage.qml`, `bar/ClaudeIndicator.qml` |
| Ciel Charts | `/api/usage/series`, split development vs `runner-*` keys | Ciel README:210-212 |
| future runners (spare quota `claude -p`) | one key per runner | `topics/nier.md:94` |

## What aether depends on

| Needs | If missing |
|---|---|
| roberta up, Docker, the `proxy` stack | no model calls through aether |
| roberta's 4G uplink | provider unreachable; roberta is the internet path for all AI work |
| Cloudflare DNS for the name, tailnet when away | laptop cannot find or reach it |
| registry on roberta | image `127.0.0.1:5000/aether:latest` |
| both provider accounts linked | health shows "2 accounts, 2 usable" (`topics/aether.md` Open) |

## When aether is down

- Sessions started through the fish `claude` function check `https://aether.wuling.dev/health`
  (2 s) and, if it fails, go straight to the account (dotfiles
  `config/fish/conf.d/claude-aether.fish:2-9`). Session restore uses fish
  (`session-restore.py:74-83`).
- A session already running when aether dies waits under the retry watchdog
  (`topics/aether.md` Open, fallback item).
- Sessions started any other way (bash, a systemd timer, `claude -p` runners) have no fallback.
- The aether README still says "No fallback when aether is down" (README:24): stale.
- Deploying aether restarts the container that serves the deploying session itself: the
  in-flight request fails and the watchdog retries. If a broken image is deployed, a fresh fish
  session falls back to the account, so the fix can still be made.

## Deploy

`bin/deploy` builds the image on the laptop from the working tree (`--network host`), tags
`127.0.0.1:<free port>/aether:latest`, tunnels to roberta's registry, pushes, then
`docker compose up -d aether` on roberta (`~/Playground/aether/bin/deploy`). The stack
definition (`stacks/aether`) lives in home-server and changes through `bin/hs push`.
`bin/check` runs typecheck, tests and lint in the image's Bun (README:12-13).

## Links to the company fork

The fork's history is kept outside the repo, in `~/Downloads/ccflare.zip` and a scratch remote
`keepers` (`topics/aether.md` Open, port list). Local branches `import` and `trial` are never
pushed (`topics/aether.md:40-42`). Losing `~/Downloads` (not backed up,
`topics/backups.md:61-62`) loses the source for the remaining ports.

## Open points that touch other parts

- work's key: until it exists, work's fish sessions use the account directly.
- rom and work see each other's usage in one proxy: accept or hide per owner (Open).
- New device or user: the nier installer should make the key or warn (`topics/nier.md:102`;
  `link.sh` warns since d1dcb6e).
