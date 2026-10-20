---
name: session-restore
summary: Per-minute window/terminal/Claude session save and login restore for both users, Firefox own tab restore
status: active
owner: shared
host: mikaela
repos: /opt/dotfiles
folder: none
verified: 2026-10-10
source: config/hypr/custom/scripts/session-{save,restore}.py, custom/execs.lua, links.manifest, system/etc/firefox/policies, history.md 2026-09-12, 2026-09-14, 2026-10-09
---

# Session restore

At login each account gets its windows, kitty terminals (with tabs and cwd) and Claude Code sessions
back on their workspaces. Scripts in `/opt/dotfiles/config/hypr/custom/scripts/`, units in
`config/hypr/custom/systemd/`. Current design dates from 2026-10-09; it replaced an earlier one.

## Facts

- Save: `hypr-session-save.timer` runs `session-save.py` every minute into
  `~/.local/state/hypr-session/session.json`. It records every window's workspace and geometry. It
  skips while a `restoring` flag exists or `systemctl --user is-system-running` says stopping.
  It runs every minute because a save at shutdown races windows closing.
- kitty: one `kitty -1` process owns every OS window, so `/proc` cannot tell windows apart.
  The saver reads `kitty @ ls`, matches OS windows to Hyprland clients by title, and records each
  tab's cwd plus its Claude session id (from `~/.claude/sessions/<pid>.json`) or an allowlisted program.
- Restore: `execs.lua` runs `session-restore.py` at Hyprland start. It starts one
  `kitty --session` process per saved OS window with an `exec_cmd` workspace rule (the rule matches the
  launched pid, so a shared process fails, see Log), `fish -C 'claude --resume <id>'` for Claude tabs
  (preceded by an OSC 2 title "star + session name", saved from `~/.claude/sessions/<pid>.json` `name`:
  a resumed Claude sets no terminal title until its next message, so the window and the bar's
  top-left read "fish" otherwise),
  other apps once per process, then moves late or extra windows (Firefox) to their saved
  workspace by address for 90 s.
- Per user: rom calls `session-restore.py --ensure feishin --ensure surfshark --ensure firefox`
  (an app already restored is not opened twice); work calls it plain. Details in `machines/mikaela.md`.
- Firefox restores its own tabs: `system/etc/firefox/policies/policies.json` sets
  `Homepage.StartPage = previous-session` (not locked). Installed by `sudo bootstrap/setup-machine.sh`.
  Firefox is not signed in to Sync; logins and bookmarks are staged for the backup instead.
- Claude sessions: resume by id, so the transcript continues. Handoff and keepalive behaviour
  is separate (`/opt/nier`). `claude -p` windows are not Claude sessions for this purpose.
- `link.sh` links the units and enables the timer by symlink (works for `sudo -u work`).
- Limit: only window placement and the listed app state come back. Editor buffers and shell history
  are each app's job.

## Open

- [x] (2026-10-20, rom: "yes done") Verify at a real login (not only the synthetic test) that every window lands right (since 2026-10-10)
- [ ] work: `systemctl --user disable --now hypr-session-save.service` if the old service was ever
      enabled there; the unit is now a timer (since 2026-10-10)
- [x] (2026-10-20, rom: "yes done") At the next reboot: restored Claude windows are titled with their session name and the bar
      shows them done, not spinning (since 2026-10-16)

## Log

- 2026-09-12: first version: one argv/cwd per pid, saved at shutdown and on SUPER+SHIFT+Backspace,
  replayed with `hyprctl dispatch`. Failed for terminals: all kitty windows share one process, and
  `exec_cmd` rules match the launched pid, which hands off to the existing instance and exits,
  so every terminal opened on one workspace.
- 2026-09-14: the user only wanted Spotify/Surfshark/Firefox back, not terminals; feature disabled
  (7cfef22). Reversed 2026-10-09.
- 2026-10-09: user asked for windows, terminals and Claude sessions at every login. Rebuilt as above
  (dotfiles ffa7ca5), verified against 4 live kitty windows and a test session on a special workspace.
- 2026-10-09: containers need no change: the media app is `on-failure` on purpose (a PIN lockout exit
  stays down), others are `unless-stopped`.
- 2026-10-16: after a battery-out shutdown, restored Claude windows read "fish" and the bar showed them
  running; restore now titles them with the session name, the bar reads Claude's idle/busy status.
