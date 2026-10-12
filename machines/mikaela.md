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
`/opt/dotfiles` (rom618/config, the desktop) and `/opt/nier` (rom618/nier, agent config). Each
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

## Backups

Per user, restic to Google Drive (rclone remote `gdrive`), repo pinned in
`~/.config/backup/restic.env` (rom: `gdrive:backups/camille-rom`, host `mikaela`), every 3 hours
(`restic-backup.timer`), weekly forget/prune/check (`restic-maintain.timer`), desktop
notification on failure. Git repos with a remote are replaced by a small state snapshot
(`backup/git-snapshot.sh`, `git-restore.sh`). `backup/never.txt`: never backed up in any form
(Camille, the media vault). Wi-Fi secrets: `sudo backup/system-secrets.sh` (age). The restic
password (`~/.config/restic/password`) is never readable by agents. Details:
`/opt/dotfiles/backup/README.md`.

## Apps and services here

| What | Where | Owner |
|---|---|---|
| Camille (media app) | moved to roberta 2026-10-10; laptop containers and images removed 2026-10-11; the files in `~/srv/camille` and the vaults in `~/srv/data/camille` remain as a fallback until rom removes them | rom (9s: camille) |
| location tools | containers `location-chromium`, `location-cdp-proxy` (Chromium in a closed container, allowed) | rom (9s: location) |
| Session restore | `hypr-session-save.timer` + `session-restore.py` at login, both users | shared |
| Agent hooks | guard, session-start, cache-keepalive, statusline (`/opt/nier/CLAUDE.md`) | shared |
| Ollama | system service, iGPU via `ollama-vulkan` | shared |
| Ciel agent | `ciel-agent.service` (root): failed services and timer runs of the system, rom and work to https://ciel.wuling.dev, spooled while away; installed and updated by `setup-machine.sh` once `/etc/ciel/token` exists; log lines mentioning the private media paths are removed before sending; it updates itself from the hub | shared |
| Ciel notifier | user unit `ciel-notify.service` (dotfiles, linked by link.sh): desktop notifications that open Ciel in Firefox, `$XDG_RUNTIME_DIR/ciel/state.json` for the bar's Ciel indicator | per user |
| Ciel docs feed | post-commit hook in every watched repo (commits into the Ciel journal through the local agent); rom's user timer `ciel-docs-sync.timer` (journal and inventory into overseer and 9s, daily 00:30); rom's read-only Ciel API token in `~/.config/ciel/token`, also read by `nier context` for the live brief | rom |
| Surfshark | autostart for rom; Docker bridges need MTU 1280 while it is up | rom |
| ssh | one key per purpose (`id_ed25519_github`, `_roberta`, `_epita_forge`), random passphrases in the login keyring, loaded at login into `ssh-agent.socket` by `custom/scripts/ssh-keys-load.sh`; `~/.ssh/config` aliases `github.com`, `roberta`, `git.forge.epita.fr`. New keys: `/opt/nier/tools/ssh-key-new.sh NAME [ALIAS HOSTNAME [USER]]` (needs a `login` keyring collection; work had none on 2026-10-11, autologin creates none) | rom; work: `_github`, `_mirador` (tailnet 100.64.0.1) |
| Tailscale | `tailscaled`; tailnet `hs.vesscam.com` (someone else's Headscale, for a Forgejo), connected, node `camille` (old hostname) under tailnet user `romain`, `OperatorUser` work; `mirador` (100.64.0.1) answers `tailscale ping` (2026-10-11); `~/.ssh/config` aliases for work: `mirador` (port 22 closed to us) and `mirador-dev` (100.64.0.5, user `admin`, key `_mirador`; work copy of the repo in `~/associe/inmarsee-vesscam` there, origin `forgejo:mirador/inmarsee-vesscam.git`, 2026-10-11). Since 2026-10-11 on work's user unit `tailscaled-work.service` (real file in work's `~/.config/systemd/user`, userspace networking, state `~/.local/share/tailscale-work`, SOCKS5 localhost:1055, CLI wrapper `~/.local/bin/tsw`, Firefox via PAC `~/.config/tailscale-work/proxy.pac` (tailnet names and 100.64.0.0/10 to the SOCKS5, rest direct), ssh through `ProxyCommand tsw nc %h %p`); the system `tailscaled` is on rom's own tailnet since 2026-10-12 (node `mikaela`, 100.67.32.103, operator rom, no tailnet DNS, incoming blocked; the home route 192.168.1.0/24 through roberta is refused on the home Wi-Fi and accepted elsewhere by the NetworkManager hook `50-tailnet-home-route`, dotfiles `system/README.md`); direct connection to roberta with Surfshark on (2026-10-12); the old vesscam node state was removed 2026-10-12. Never run both on the same node state | work |

## User differences

| Area | rom | work | Where |
|---|---|---|---|
| Autostart | session restore + Spotify, Surfshark, Firefox (`session-restore.py --ensure`); vault passphrase prompt (`scripts/media-vault-unlock.sh`) | session restore only | `config/hypr/custom/execs.lua` |
| Idle suspend | never, unless on battery below 20% (`custom/scripts/idle-suspend.sh`); lid ignored machine-wide (`system/etc/systemd/logind.conf.d/lid.conf`); lid close with Keep awake on plays a warning | idle suspends as before; lid ignored | `config/hypr/hypridle.conf` |
| Session audio | normal | boots muted | `config/hypr/custom/execs.lua` |
| Docker | in group `docker` (docker without sudo, root-equivalent) | not in the group | `usermod -aG docker`, set by hand in 2026-09, no script |

## Useful checks

`readlink -f ~/.config/hypr/custom`, `git -C /opt/dotfiles status`, `git -C /opt/nier status`,
`/opt/dotfiles/bootstrap/link.sh --dry-run`, `/opt/nier/bootstrap/link.sh --dry-run`, `id`,
`hyprctl reload`, `systemctl --user list-timers`, `xdg-settings get default-web-browser`.
