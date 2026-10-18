---
host: mikaela
machine_id: 317c58351eca457c9cf0d3f04d9b73cd
os: arch
role: desktop
owner: shared
users: rom (1000, personal), work (1001, work); both in wheel and dots
verified: 2026-10-10
source: /opt/nier/CLAUDE.md before the 2026-10-10 split, notes/history.md
---

# mikaela (the laptop)

Arch Linux laptop shared by two accounts, `rom` and `work`. Both can sudo (work joined `wheel` on 2026-10-11), so either can read the
other's home through root: a convenience setup, not an isolation boundary. Hostname was `camille`
until 2026-10-10. The app Camille moved to roberta on 2026-10-10 (camille.wuling.dev); camille.local
is retired (avahi stops announcing it once `setup-machine.sh` has run).

## Hard rules on this device

- Private, never read, listed, probed, thumbnailed or streamed (directly, through `docker exec`,
  the app's API or a script's output; file names count): `~/Media`, `~/Media.vault`,
  `~/Playground/media-library/app-data*`, `~/srv/data/camille`, `~/srv/camille/caddy`. The guard
  hook and deny rules back this up; the rule stands where a pattern does not match.
- Never run unprompted:
  - `./setup install`, `./setup exp-update` or any other subcommand of the dots-hyprland repo: they
    are not symlink aware and silently rewrite the shared copy under `/opt/dotfiles` (or replace
    links with real files). Updates go through `/opt/dotfiles/bootstrap/ii-update.sh`
    (`overseer/runbooks/ii-update.md`).
  - `systemctl restart sddm` or `systemctl restart systemd-logind`: both kill the live graphical
    session (logind holds the seat; a restart on 2026-09-11 left a black screen). A logind
    drop-in waits for the next reboot.
- Synthetic input (`ydotool`) lands in whatever window the user, who may be typing at the same
  time, has focused. Prefer non-interactive checks (`hyprctl layers -j`, `binds -j`, logs, state
  files); if input is unavoidable, check `hyprctl activewindow -j` right before it, and stop and
  report if anything looks wrong.

## Stack

Hyprland (Lua config since 0.55, see `overseer/runbooks/hyprland-lua-dispatch.md`), Quickshell
(end-4's illogical-impulse, "ii": https://github.com/end-4/dots-hyprland, docs
https://ii.clsty.link; upstream ships configuration only), kitty, fish, neovim, SDDM (X11
greeter, SilentSDDM theme), yay. Markdown files open in Apostrophe (installed). Firefox is the browser for both users; Chrome removed 2026-10-11
(was installed for `work`, hidden for rom). Intel Lunar Lake, Arc 130V iGPU. `/` 98 GB and
`/home` 823 GB on separate partitions. Wi-Fi only (`wlo1`).

## Shared configuration

Two repos owned `rom:dots`, group writable with default ACLs, so neither user needs sudo:
`/opt/dotfiles` (rom618/mikaela, renamed from rom618/config on 2026-10-13; the desktop) and `/opt/nier` (rom618/nier, agent config). Each
has `bootstrap/links.manifest` (what is linked where) and `bootstrap/link.sh` (applies it for the
calling user). Changes: commit and push; both pre-commit hooks run gitleaks.

| Path in home | Real location | Scope |
|---|---|---|
| `~/.config/fish/config.fish`, `conf.d/`, `auto-Hypr.fish` | `/opt/dotfiles/config/fish/...` | shared |
| `~/.config/fish/fish_variables` | real file | per user |
| `~/.config/kitty`, `foot`, `nvim`, `quickshell` | `/opt/dotfiles/config/<name>` | shared |
| `~/.config/hypr/custom` | `/opt/dotfiles/config/hypr/custom` | shared storage, per-user behavior via `$USER` |
| `~/.config/hypr/hyprlock.conf`, `hypridle.conf`, `hyprland`, `hyprland.lua` | `/opt/dotfiles/config/hypr/...` | shared; `hyprland/colors.lua` is matugen output, last wallpaper change wins |
| `~/.config/hypr/hyprlock/colors.conf` | real file | per user |
| `~/.config/matugen`, `fuzzel`, `starship.toml` | `/opt/dotfiles/config/...` | shared |
| `~/.config/illogical-impulse/config.json` | real file | per user, written by the shell UI (`aiconfig show/set` for `ai.*`) |
| `~/.vimrc`, `~/.ideavimrc` | `/opt/dotfiles/config/...` | shared |
| `~/.claude/settings.json`, `~/.claude/skills`, `~/claude/tools` | `/opt/nier/...` | shared |
| `~/claude/issues` | real dir | per user |
| `~/.config/systemd/user/restic-*`, `backup-failed@.service` | `/opt/dotfiles/backup/...` | shared units, per-user timers |
| `/etc/claude-code/CLAUDE.md` | `/opt/nier/CLAUDE.md` | machine |
| `/etc/claude-code/managed-settings.json` | root copy of `/opt/nier/managed-settings.json` (`sudo /opt/nier/bootstrap/setup-machine.sh`) | machine |
| anything else in `~/.config` | real file | per user |

Sharing a config directory does not share the state beside it: `~/.local/share` and `~/.cache`
stay per user (nvim plugins, fish history). `~/.config/hypr` is linked file by file.

| Intent | Location |
|---|---|
| Desktop for both users | `/opt/dotfiles/config/...` |
| Hyprland for one user | `/opt/dotfiles/config/hypr/custom/`, behind a `$USER` branch (`overseer/runbooks/per-user-dispatch.md`); input tweaks in `custom/general.lua` |
| Agent behavior for both users | `/opt/nier` |
| Personal, throwaway, machine-local | that user's home, outside `~/.config` |
| Secrets | never a repo: a gitignored file under `~/.config/<tool>/`, covered by the backup |

## Disk

One pool per partition for both users, no quotas: `/home` (both homes, 823 GB) and `/` (system,
`/opt`, Docker, 98 GB); `/tmp` is RAM. A user filling `/home` breaks the other user too. Decision
2026-10-18 (user): no quota, Ciel's `disk` check (warn 85%, fail 95%) is the guard. Large task
output goes in a folder the backup skips (`backup/excludes.txt`), or it is uploaded every 3 hours.

## Backups

Per user, restic to Google Drive (rclone remote `gdrive`), repo pinned in
`~/.config/backup/restic.env` (rom: `gdrive:backups/camille-rom`, host `mikaela`), every 3 hours
(`restic-backup.timer`), weekly forget/prune/check (`restic-maintain.timer`), desktop
notification on failure. Git repos with a remote are replaced by a small state snapshot
(`backup/git-snapshot.sh`, `git-restore.sh`). `backup/never.txt`: never backed up in any form
(Camille, the media vault). Wi-Fi secrets: `sudo backup/system-secrets.sh` (age, to the sops admin key). The user's own secret files: `sops/mikaela-<user>.yaml` in dotfiles (`bootstrap/user-sops.py`). The restic
password (`~/.config/restic/password`) is never readable by agents. Details:
`/opt/dotfiles/backup/README.md`.

## Apps and services here

| What | Where | Owner |
|---|---|---|
| Camille (media app) | moved to roberta 2026-10-10; laptop containers and images removed 2026-10-11; the laptop files and vaults were deleted on 2026-10-14 (rom) | rom (9s: camille) |
| location tools | containers `location-chromium`, `location-cdp-proxy` (Chromium in a closed container, allowed) | rom (9s: location) |
| Session restore | `hypr-session-save.timer` + `session-restore.py` at login, both users | shared |
| Agent hooks | guard, session-start, cache-keepalive, statusline (`/opt/nier/CLAUDE.md`) | shared |
| Ollama | system service, iGPU via `ollama-vulkan` | shared |
| Ciel agent | `ciel-agent.service` (root): failed services and timer runs of the system, rom and work to https://ciel.wuling.dev, spooled while away; installed and updated by `setup-machine.sh` once `/etc/ciel/token` exists; log lines mentioning the private media paths are removed before sending; it updates itself from the hub | shared |
| Ciel notifier | user unit `ciel-notify.service` (dotfiles, linked by link.sh): desktop notifications that open Ciel in Firefox, `$XDG_RUNTIME_DIR/ciel/state.json` for the bar's Ciel indicator | per user |
| Ciel docs feed | post-commit hook in every watched repo (commits into the Ciel journal through the local agent); rom's user timer `ciel-docs-sync.timer` (journal and inventory into overseer and 9s, daily 00:30); rom's read-only Ciel API token in `~/.config/ciel/token`, also read by `nier context` for the live brief | rom |
| Aether client | every agent session goes through https://aether.wuling.dev (nier `settings.json` `env` + `apiKeyHelper` `tools/aether-key.sh`, since 2026-10-12); `claude` started from fish falls back to the account login when aether is down or the user has no `~/.config/aether/key` (`conf.d/claude-aether.fish`, 2026-10-13). Per user the default key is `~/.config/aether/key` (optional `url`); inside a job folder the job's `.aether/{key,url}` (Job folders below, aether topic Facts) | per user, per job |
| Surfshark | autostart for rom; Docker bridges need MTU 1280 while it is up | rom |
| ssh | one key per purpose (`id_ed25519_github`, `_roberta`, `_epita_forge`), random passphrases in the login keyring, loaded at login into `ssh-agent.socket` by `custom/scripts/ssh-keys-load.sh`; `~/.ssh/config` aliases `github.com`, `roberta`, `git.forge.epita.fr`. New keys: `/opt/nier/tools/ssh-key-new.sh NAME [ALIAS HOSTNAME [USER]]` (needs a `login` keyring collection; work had none on 2026-10-11, autologin creates none). A job's keys, Host blocks and known_hosts live in that job's `.ssh` (Job folders below) | rom; work: `_github`, job keys in the job folders |
| Tailscale | System `tailscaled`: rom's own tailnet since 2026-10-12 (node `mikaela`, 100.67.32.103, operator rom, key expiry off, no tailnet DNS, incoming blocked; the home route 192.168.1.0/24 through roberta is refused on the home Wi-Fi and accepted elsewhere by the NetworkManager hook `50-tailnet-home-route`, dotfiles `system/README.md`); direct connection to roberta with Surfshark on (2026-10-12); an old work tailnet node state was removed from it 2026-10-12. A job's tailnet never uses the system daemon: it runs as a userspace user unit from the job folder (state, PAC file and CLI wrapper in the job, own SOCKS5 port, ssh through `ProxyCommand <wrapper> nc %h %p`; details in that job's docs). Never run two daemons on the same node state | rom (system daemon); work (job units) |

## Job folders (work)

work holds separate jobs (user, 2026-10-18). A job is a folder `~/<job>/` with a `.job` file and
holds everything of that job, so removing it leaves nothing behind; its facts live in
`<job>/docs/`, never in overseer (rules: nier `AGENTS.md` section 7).

- `.claude`: Claude Code data. The fish `claude` function (dotfiles `conf.d/claude-aether.fish`)
  finds the job of the current folder, sets `CLAUDE_CONFIG_DIR=<job>/.claude`, takes the proxy
  from `<job>/.aether/{key,url}` (no `url`: aether) and exports `AETHER_KEY_FILE` for nier's key
  helper `tools/aether-key.sh`. Issue folders `<job>/claude/issues`, scratch `<job>/claude/scratch`.
- `docs/`: the job's pages.
- `.ssh`: keys, Host blocks, known_hosts; pulled in by `Include ~/*/.ssh/config` at the top of
  `~/.ssh/config`, keys loaded at login by `ssh-keys-load.sh`; new key:
  `SSH_KEY_DIR=<job>/.ssh /opt/nier/tools/ssh-key-new.sh NAME ...`.
- `.gitconfig`: the job's git identity, included for repos under the job (`includeIf` in `~/.gitconfig`).
- `.firefox`: own Firefox profile, launcher `firefox-<job>` ("Firefox (<job>)").
- `.config/systemd`: the job's user units, linked into `~/.config/systemd/user/`.
- fish history `job_<job>_history` inside a job (`conf.d/job-history.fish`), excluded from the backup.
- Never backed up.
- `job list|new|remove` (dotfiles `bootstrap/job.py`, `~/.local/bin/job`). `remove` (real
  terminal, asks for the name) also clears the job's keyring entries, agent keys, units, links,
  the git include, fish history, Firefox entries, recent files, thumbnails, clipboard history and
  Claude traces in `~/.claude` and `/tmp`; traces outside the machine are listed, not deleted.

## User differences

| Area | rom | work | Where |
|---|---|---|---|
| Job folders | none | one per job (Job folders above) | dotfiles `bootstrap/job.py` |
| Autostart | session restore + Feishin (music player for Mephisto; replaced Spotify 2026-10-14; the window rule sends a new Feishin window to the special workspace), Surfshark, Firefox (`session-restore.py --ensure`) | session restore only | `config/hypr/custom/execs.lua` |
| Idle suspend | never, unless on battery below 20% (`custom/scripts/idle-suspend.sh`); lid ignored machine-wide (`system/etc/systemd/logind.conf.d/lid.conf`); lid close with Keep awake on plays a warning | idle suspends as before; lid ignored | `config/hypr/hypridle.conf` |
| Session audio | normal | boots muted | `config/hypr/custom/execs.lua` |
| Docker | in group `docker` (docker without sudo, root-equivalent) | not in the group | `usermod -aG docker`, set by hand in 2026-09, no script |
| Clipboard app | Super+Shift+V sends what is copied (`~/.local/bin/clip`); user unit `clipboard-listen` shows arrivals from other devices; key in `~/.config/clipboard/key` | none | `config/hypr/custom/keybinds.lua`; `clients/linux/install.sh` in the clipboard repo (9s: clipboard) |
| Clipboard history (Super+V) | wiped every day at 08:00 by `clipboard-wipe.timer` (deletes the cliphist db so the `#N` item numbers restart at 1; `cliphist wipe` keeps the counter; `Persistent=true` runs it at the next login or wake when 08:00 was missed). The `;` in the search box on Super+V is ii's clipboard search prefix (`search.prefix.clipboard`), needed: without it the overview searches apps | history kept (cliphist keeps the last 750) | `config/hypr/custom/systemd/clipboard-wipe.*`, `custom/scripts/clipboard-wipe.sh`, enabled by `bootstrap/link.sh` for rom only |
| Claude Code desktop notifications (banner when a session finishes or needs input) | off since 2026-10-17 (`"preferredNotifChannel": "notifications_disabled"`; the bar's workspace badges show the state instead) | on (default `auto`, kitty notifications) | `~/.claude.json`, per user, no repo; the Ciel notifications are separate (`ciel-notify.service`) |

## Useful checks

`readlink -f ~/.config/hypr/custom`, `git -C /opt/dotfiles status`, `git -C /opt/nier status`,
`/opt/dotfiles/bootstrap/link.sh --dry-run`, `/opt/nier/bootstrap/link.sh --dry-run`, `id`,
`hyprctl reload`, `systemctl --user list-timers`, `xdg-settings get default-web-browser`.
