---
name: quickshell-bar
summary: Local customizations of the Quickshell (illogical-impulse) bar and popups, and which upstream files were edited
status: active
owner: shared
host: mikaela
repos: /opt/dotfiles
folder: none
verified: 2026-10-11
source: config/quickshell/ii/ (modules/ii/bar, services), /opt/nier/tools/qs-known-warnings.txt, history.md 2026-09-12 to 2026-10-10
---

# Quickshell bar

The bar is end-4's ii shell in `/opt/dotfiles/config/quickshell/ii/`, shared by both accounts. Quickshell
has no `custom/` override layer, so every change below edits upstream files directly and can be
dropped by an `ii-update.sh` merge (`overseer/runbooks/ii-update.md`, `bootstrap/ii-diff.sh`).
The AI sidebar is in `topics/ai-sidebar.md`. Restart with `/opt/nier/tools/qs-restart.sh`
(reports only warnings outside the baseline).

## Facts

- Docker bubble: `bar/DockerIndicator.qml`, `DockerPopup.qml`, service `services/Docker.qml`. Hides when the
  `docker` CLI is absent or `docker info` fails; a user not in the `docker` group sees nothing (root
  cause of the missing icon for rom). It sits right of the weather bubble. The bar's right section is
  RTL: declared before X means visually right of X.
- Battery bubble: `BatteryIndicator` moved out of the crowded center-right group into its own `BarGroup`
  in `bar/BarContent.qml`, just left of the system tray (declared after `SysTray`). Hidden in the
  shortest bar form or without a battery.
- Ciel: `bar/CielIndicator.qml` and `services/Ciel.qml` (new files) plus one block in
  `bar/BarContent.qml` after the Claude indicator. Reads `$XDG_RUNTIME_DIR/ciel/state.json`, written by
  the user unit `ciel-notify.service` (dotfiles `config/hypr/custom/systemd/`); hidden when the file is
  missing or all is fine, a cloud with the count when something needs attention (red when failing), a
  crossed cloud when the hub is unreachable; click opens Ciel in Firefox. Details: ciel repo README.
- Claude usage: `ClaudeIndicator.qml`, `ClaudePopup.qml`, `services/ClaudeUsage.qml`, hides when the
  tools are absent. Data from `/opt/nier/tools/claude-usage.py` summing `input + output + cache_creation + cache_read`
  tokens from every assistant `usage` entry in `~/.claude/projects/*/*.jsonl` (subagent transcripts included, streamed
  duplicates counted once). Cost uses the script's per-model `PRICING` table with cache read/write multipliers;
  an unknown model prices at 0, so add new models there. Shown per type, today and all time.
- Limits (5h/7d percent and reset time) come from `claude-statusline.py` writing
  `~/.claude/claude-rate-limits.json` while a session renders; push-only, flagged stale after 6 h.
  No use of the undocumented OAuth usage endpoint.
- Energy and carbon rows are a rough estimate, not a measurement: the mean of three published
  output-token figures (about 0.0003, 0.00105, 0.0001083 Wh/token; sources in `claude-usage.py`),
  input and cache tokens scaled by the price ratios (5x output:input, 0.1x cache read, 1.25x cache
  write), 480 gCO2/kWh. The "(est.)" label was removed by the user's choice; do not present it as exact.
- Resources: one CPU icon with the popup (RAM, swap, disk), inside the clock's group, left of the clock.
  Sizes say GiB (binary). Disk is one combined section (`/` and `/home` are separate partitions).
- GPU usage: `config/hypr/custom/scripts/gpu-usage.py` diffs DRM fdinfo cycle counters
  (`/proc/*/fdinfo/*`, `drm-cycles-rcs` / `drm-total-cycles-rcs`). `intel_gpu_top` refuses Xe devices and
  `gputop` has no machine output. Low engine-busy during LLM decoding is normal (memory-bound).
- Media bubble: own width `Appearance.sizes.barMediaWidth` (230 verbose / 140), so the active-window
  title gets more room. Clicking the active window toggles the overview.
- Cheatsheet (SUPER+/): tabs Keybinds and Aliases (periodic table removed). `services/FishAliases.qml`
  parses `alias` lines and `function ... --description` blocks from `config.fish` and `conf.d/*.fish`.
  `WlrKeyboardFocus.OnDemand` is on (needed for Escape, slower to open); SwipeView is non-interactive.
- Brightness: fixed 0.05 step; the bar scroll-to-change area was removed. Weather retries after a failed
  fetch (the login-time DNS race with Surfshark made it blank for 10 minutes).
- Notifications: a left click on a popup or sidebar item (or the newest item of a group) calls
  `Notifications.invokeDefaultAction`; the default action is no longer drawn as a blank button. kitty attaches a
  `default` action to every OSC 99 notification and focuses the sending tab. Works only while the
  server still tracks the notification (not after a qs restart).
- SysTray separator dot shows only when both groups are non-empty.
- Persistent state fields (`Persistent.states.*`) must be declared on the JsonObject schema in
  `Persistent.qml`, or a write lands in the file but bindings never react.
- Known warnings: baseline list `/opt/nier/tools/qs-known-warnings.txt` (AiChat/Anime `commandPrefix` of
  null on sidebar teardown, `Ai.qml` undefined value, `NotificationPopup` enable, `BarContent` undefined to
  double, translation file missing). Update it only for cosmetic ones.
- Upstream files edited directly (from `ii-diff.sh`, re-apply after a merge): common `Appearance`, `Config`,
  `Persistent`, `NotificationGroup`, `NotificationItem`; bar `ActiveWindow`, `BarContent`, `Resources`,
  `ResourcesPopup`, `SysTray`; `Cheatsheet`; sidebarLeft `AiChat`, `AiMessage`, `SidebarLeftContent`;
  services `Ai`, `ai/ApiStrategy`, `ai/OpenAiApiStrategy`, `Brightness`, `Hyprsunset`, `Notifications`,
  `ResourceUsage`, `Weather`. New local files: Docker/Claude bar files, `Docker`, `ClaudeUsage`,
  `FishAliases`, `CheatsheetAliases`.

## Open

- [ ] Confirm the notification left click focuses the sender (reloaded cleanly, click not yet confirmed) (since 2026-10-10)
- [ ] Kitty tab rename alias needs `allow_remote_control`; works only in kitty processes started after it was added (since 2026-09-13)

## Log

- 2026-09-12: Docker bubble and Disk/Claude sections added to the Resources popup; periodic-table tab removed.
- 2026-09-13: bar cleanup rounds: combined disk, one CPU icon placed after an overflow regression
  (an anchored Row lets children spill into neighbours), scroll-brightness removed after the step
  tweaks failed, Aliases tab, cheatsheet Escape and swipe fixes, GiB labels, media width, Docker bubble
  moved next to weather. Claude popup split from Resources.
- 2026-09-13: GPU usage via fdinfo after `intel_gpu_top` failed on Xe (dotfiles e76cc1f); energy estimate
  added (e7108e8), "(est.)" label dropped (d35eb23).
- 2026-09-26: `claude-usage.py` fixed: Opus priced 0, duplicate stream lines counted twice, subagents skipped.
- 2026-10-09: Claude limits rows shown per window, reset day shown over 20 h away (c1c0370).
- 2026-10-10: notification click-to-focus (975b42b).
- 2026-10-11: battery given its own bubble next to the tray (center group was crowded).
