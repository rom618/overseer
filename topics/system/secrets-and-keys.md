# Secrets and keys: where each lives, what depends on it

Never a value here, only the location. Rule: `AGENTS.md:33-48` (secrets never in a repo; write
where a secret lives). Agents never read any of these files; helpers read some of them on the
agent's behalf (`aether-key.sh`, `nier context`, `ciel-docs-sync.sh`).

## On mikaela (per user unless noted)

| Secret | Location | Used by | Backed up | If lost |
|---|---|---|---|---|
| ssh keys (github, roberta, epita_forge) | `~/.ssh/id_ed25519_*`, passphrases in the login keyring | git push, `bin/hs`, every `bin/deploy` | yes (`~/.ssh`, `~/.local/share/keyrings` in `includes.txt`) | no deploys; roberta reachable only from its console |
| aether client key | `~/.config/aether/key` (0600) | `nier/tools/aether-key.sh`, statusline, bar | no | make a new one: `bin/hs aether-key-save NAME` |
| Ciel read-only API token (rom) | `~/.config/ciel/token` (0600) | `nier context` brief, `ciel-docs-sync.sh` | no | `ciel-hub token add rom <label>` on roberta |
| restic password | `~/.config/restic/password` | `restic-backup.sh` | no (it is the key) | the laptop backup is unreadable |
| rclone Drive config | `~/.config/rclone` | restic, modpack release | yes | `rclone config` again |
| restic repo pin | `~/.config/backup/restic.env` | `restic-backup.sh:13` | yes | wrong repo path after a hostname change |
| copies of roberta's secrets | `~/.config/home-server/` (restic password, a secrets archive of 2026-10-10) | disaster recovery of roberta | yes | see roberta below |
| Minecraft restic password copy | `~/.config/mc-server/` | mc-modpack tools | no (gap) | copy again from roberta |
| Claude account login | `~/.claude/.credentials.json` | fallback when aether is down | yes (`~/.claude`) | log in again |
| Ciel agent token (machine) | `/etc/ciel/token` (root) | `ciel-agent.service` | no | `ciel-hub device token mikaela` |
| Wi-Fi secrets (machine) | NetworkManager, root | Wi-Fi | `system-secrets.sh` age file, stale | re-enter |
| work's vesscam Tailscale state | `~work/.local/share/tailscale-work` | work's tailnet | work has no backup | rejoin |

Sources: `machines/mikaela.md:83-92, 103-109`; `backup/includes.txt`; `topics/backups.md:32-33,
69`; `topics/ciel.md:64`; dotfiles `bootstrap/setup-machine.sh:94-100`.

## On roberta

| Secret | Location | Used by | Backed up | If lost |
|---|---|---|---|---|
| stack settings and keys | `stacks/<name>/.env` (0600) | each stack | laptop copy (see note) | regenerate per stack |
| restic password (all repos) | `/etc/home-server/restic.pw` | backup timers, off-site copy | laptop copy | every local and off-site server backup unreadable |
| rclone config | `/etc/home-server/rclone.conf` | `offsite-backup.sh:3` | laptop copy | off-site copy skipped silently ("off-site copy skipped") |
| Cloudflare API token | `/etc/home-server/cloudflare-api-token` (root) | Caddy DNS challenge, `dns-records.sh` | laptop copy | certificates stop renewing after about 90 days |
| proxy certificate keys | `/srv/data/proxy` (private) | Caddy | no | re-issued |
| Ciel agent token | `/etc/ciel/token` (0600 root) | roberta's agent | no | `ciel-hub device token roberta` |
| aether provider tokens and client keys | `/srv/data/aether` SQLite, clear text | aether | no (decided) | relink accounts, new keys everywhere |
| Ciel DSNs for Camille and aether | `stacks/camille/.env`, `stacks/aether/.env` | error reports | laptop copy predates them | `hs camille-dsn`, `hs aether-dsn` |
| Ciel admin key for aether usage | `stacks/ciel/.env` | Ciel charts | no | `hs ciel-aether-key` |
| playit secret key | `stacks/playit/.env` | game tunnels | laptop copy | new key from playit.gg |
| Surfshark WireGuard key (qBittorrent) | `stacks/camille/.env` | gluetun | laptop copy | `hs camille-vpn-key` |
| Camille box passphrase | rom only | unlock after reboot | never | Camille's data is gone |
| Vaultwarden master password | rom only, written down offline | vault | never on the server | vault contents gone |
| Tailscale node key | tailscaled state | subnet router | no | `bin/hs tailscale-login`, approve route, key expiry off |

Note: "laptop copy" means the copy of the server secrets made on 2026-10-10 into
`~/.config/home-server/` (`topics/backups.md:72`). This audit did not open it, so which secrets
it holds is not verified.

Sources: home-server `README.md:19, 54-58, 137-139, 155-160, 186-199, 212-216, 245-251`;
`machines/roberta.md:29-35`; `topics/setup-steps.md:31-34`; `scripts/offsite-backup.sh:3-8`.

## Outside both machines

| Secret | Where | Note |
|---|---|---|
| Google account (Drive) | rom | holds every off-site copy; recovery tied to the phone (rom's TODO) |
| Tailscale account | rom's personal Google sign-in | admin console: routes, key expiry, the policy file |
| Cloudflare account | rom | DNS for wuling.dev |
| GitHub account | rom | every repo; the only off-machine copy of most code |
| Anthropic accounts (rom and work) | linked into aether by browser login | tokens live in aether's database |
| ntfy topic | Ciel settings, cached by agents | set up; the topic name acts as a password, never write it in a doc |

## Observations

- The laptop's copy of roberta's secrets is dated 2026-10-10 19:35 (file time). Secrets created
  later (Ciel token, Ciel and aether DSNs, the aether admin key, the Tailscale node) are not in it,
  and no page says how or when it is refreshed (`findings.md`).
- Nothing in the managed deny list or the guard covers `~/.config/aether/key` or
  `~/.config/ciel/token` (`nier/managed-settings.json:3-42`, `tools/guard.sh:25-26`); only the
  rule protects them.
- One restic password covers every server repo, personal and shared (`topics/home-server.md:47`),
  and it will need splitting with the server (Open, `topics/home-server.md:53`).
