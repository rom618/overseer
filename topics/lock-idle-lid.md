---
name: lock-idle-lid
summary: hyprlock, hypridle, idle suspend, lid policy, Keep-awake lid alert, night light following the sun
status: active
owner: shared
host: mikaela
repos: /opt/dotfiles
folder: none
verified: 2026-10-22
source: config/hypr/hypridle.conf, custom/keybinds.lua, custom/scripts/, /etc/systemd/logind.conf.d/lid.conf, history.md 2026-09-13/14/20, 2026-10-09
---

# Lock, idle, lid

Screen locking, idle steps, suspend and lid behaviour on mikaela. Files: `config/hypr/hypridle.conf`,
`config/hypr/hyprlock.conf`, `config/hypr/custom/keybinds.lua`, `custom/scripts/` (all in
`/opt/dotfiles`, shared by both accounts). Per-user differences are in `machines/mikaela.md`.

## Facts

- Lock command: bare `pidof hyprlock || hyprlock`, used by SUPER+L and by hypridle's `lock_cmd`
  and `before_sleep_cmd`. The `pidof` guard must be on every call site, or locks stack.
- hypridle steps: 5 min lock, 10 min dpms off, 15 min `idle-suspend.sh`. hypridle is started by
  `hl.exec_cmd`, not systemd: a config change needs a hypridle restart to take effect.
- Wake key: Hyprland (`key_press_enables_dpms`) turns the screen on and still passes the key to
  hyprlock, which has no option to drop it. The dpms listener's on-resume runs
  `custom/scripts/idle-wake.sh`: dpms on, then, if hyprlock runs, `wtype -k Escape` after 0.1 s,
  which clears hyprlock's field. Keys typed within that 0.1 s are cleared too.
- Idle suspend: `idle-suspend.sh` branches on `$USER`. rom never suspends unless on battery
  below 20%; work stays awake on AC and suspends on battery.
- Lid: `/etc/systemd/logind.conf.d/lid.conf` (source `system/etc/systemd/logind.conf.d/`, installed
  by `setup-machine.sh`) sets `HandleLidSwitch`, `...ExternalPower` and `...Docked` to ignore, for
  both users. `HandleLidSwitchExternalPower` alone was ignore before 2026-09-20, which is why
  closing the lid on battery suspended the laptop and took the media app down.
  Never `systemctl restart systemd-logind`; a live change is `systemctl reload systemd-logind`.
- Keep awake: quickshell toggle (`services/Idle.qml`, persisted as `idle.inhibit` in
  `~/.local/state/quickshell/states.json`). With it on and the lid closed, the screen stays on.
- Lid alert: `custom/scripts/lid-keepawake-alert.sh`, bound to `switch:on:Lid Switch` (locked) in
  `custom/keybinds.lua`, both users. If Keep awake is on it fades all sink-inputs to 0, plays
  dialog-warning.oga 3 times at 80% unmuted, restores sink volume and mute, fades streams back.
  It parses `pactl list sink-inputs` text, because `pactl -f json` fails on non-ASCII properties.
- Night light: `light.night.followSun` (default false, `Config.qml`) makes `Hyprsunset.qml` use
  `Weather.data.sunset`/`sunrise` (wttr.in, "08:02 AM" format), falling back to the fixed
  from/to until weather data arrives. Enabled in rom's `config.json` only. Both are edits to
  upstream ii files: re-apply after an `ii-update.sh` merge that drops them.
- Crashed lock: if hyprlock dies, Hyprland shows "lockscreen app died". Recover from another tty
  or session with `hyprctl --instance 0 eval 'hl.clear_crashed_lockscreen()'`. Cause of the one
  crash seen (2026-09-14) was never found; check `journalctl` if it recurs.

## Open

- [ ] hyprlock crash of 2026-09-14: root cause unknown, investigate only if it recurs (since 2026-09-14)
- [x] (2026-10-23, rom: works) Check the wake-key fix live: lock, wait 10 min for the screen to go off, press a key, the field should be empty (since 2026-10-22)
- [x] (2026-10-11) `system/README.md` lid.conf text found already corrected

## Log

- 2026-09-11: a logind restart killed the Wayland session (black screen); rule: reload only.
- 2026-09-13: six stacked hyprlock processes: idle listener and before_sleep called bare
  `hyprlock`, bypassing the `pidof` guard. Fixed in `hypridle.conf` and hypridle relaunched; the stacked processes were left (killing them would bypass the lock).
- 2026-09-14: SUPER+L launched hyprlock while idle used quickshell's native lock. After testing
  both, the user chose bare hyprlock for both (1a25286, 7cfef22). Quickshell's lock is not used.
- 2026-09-19: rom idle-suspend: stay awake on AC (8becb95), then on battery too above 20%
  (14573c7), so the media app stays reachable.
- 2026-09-20: lid close on battery was the real cause of "not served when closed"; lid now ignored
  on battery (live `reload`, not restart).
- 2026-10-19: work idle-suspend: stay awake on AC, so its message bus and remote task delivery
  stay reachable while the user is away (an idle suspend took them offline for an hour).
- 2026-10-09: Keep awake left the screen on overnight; lid alert script added (dfe5e01, c32af0a);
  hypridle had run with stale 15/20/25 min timings after an interrupted restart, restarted.
- 2026-10-09: night light follows the sun (ba0bb5e); lid.conf tracked in `system/` (d2c5a59).
- 2026-10-22: the key that woke the screen was typed into the password field; `idle-wake.sh`
  clears it with an Escape on dpms resume. hypridle restarted.
