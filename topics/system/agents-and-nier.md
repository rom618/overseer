# Agents and nier

Owners: rules `/opt/nier/AGENTS.md`; repo layout `/opt/nier/README.md`; state and gotchas
`/opt/overseer/topics/nier.md`; hooks and links on the laptop `machines/mikaela.md:47-80`.

## What an agent session is made of

```
/etc/claude-code/CLAUDE.md -> /opt/nier/CLAUDE.md -> AGENTS.md     rules (sections 1-8)
/etc/claude-code/managed-settings.json   root copy of nier/managed-settings.json (deny list)
~/.claude/settings.json -> /opt/nier/settings.json    hooks, allow/ask/deny, env, apiKeyHelper
~/.claude/skills       -> /opt/nier/skills            config, comprehension, burn, chat-review
~/claude/tools         -> /opt/nier/tools             hook scripts, helpers
~/claude/issues/<name>/                               the unit of work (per user, real dir)
agent memory (per start directory)                    user preferences only
```

Source: `/opt/nier/README.md:7-16`, `machines/mikaela.md:58-70`, `AGENTS.md:119-133`.
Both laptop users share all of it except the issue folders and memory.

## Session start: where context comes from

| Step | What | Source |
|---|---|---|
| 1 | `session-start.sh` (SessionStart hook) titles the session after its issue folder | `nier/settings.json:105-115`, `tools/session-start.sh:2-5` |
| 2 | It runs `nier context`: device found by `/etc/machine-id`, `os/<family>.md`, `roles/<role>.md`, the machine page, both docs indexes (work: overseer only), TODO heads | `topics/nier.md:29-32`, `bin/nier:100-103` |
| 3 | `nier context` appends a live Ciel brief (`/api/brief?device=`) using rom's read-only token `~/.config/ciel/token`, 4 s timeout, silent if Ciel is unreachable | `bin/nier:104-111` |
| 4 | It reports other sessions' uncommitted work in the config repos | `tools/session-start.sh:25` |
| 5 | It injects the folder's `handoffs/latest.md` and the first open items of the topic pages the issue README links | `tools/session-start.sh:45-68`, `topics/docs.md:75` |

Consequence: every session's picture of the system is the docs plus Ciel. A stale machine page
or topic summary is read by every session (see `findings.md`).

## Hooks during the session

| Hook | Script | Does | Source |
|---|---|---|---|
| PreToolUse (Bash, Read, Edit, Write...) | `tools/guard.sh` | hard-blocks private media paths, sddm/logind restarts, dots-hyprland `./setup`, recursive chown/chmod over /opt or homes | `settings.json:93-104`, `guard.sh:25-64` |
| statusLine (10 s) | `tools/claude-statusline.py` | writes `~/.claude/claude-rate-limits.json`; through aether, limits come from aether's `/api/statusline` | `claude-statusline.py:7-8, 95-102` |
| Stop (async) | `cache-keepalive/keepalive --hook` | nudges an idle session at most twice to keep the prompt cache warm, then writes the handoff and closes it | `topics/nier.md:50-57` |
| Stop | `tools/stop-docs-check.sh` | sends the session back once if files it edited this turn are uncommitted or unpushed | `topics/nier.md:44-46` |
| SessionEnd | `keepalive --session-end` | handoff on exit | `settings.json:134-144` |

Git hooks (not Claude hooks, so every committer gets them) are on `change-flow.md`.

## Model traffic

- `settings.json:156-160`: `ANTHROPIC_BASE_URL=https://aether.wuling.dev/v1/claude-code`,
  `CLAUDE_CODE_RETRY_WATCHDOG=1`, `apiKeyHelper=/opt/nier/tools/aether-key.sh`.
- `aether-key.sh` prints `~/.config/aether/key` or fails with a hint (`tools/aether-key.sh`).
  `bootstrap/link.sh:92-93` warns when a user has no key.
- The fish function `claude` (dotfiles `config/fish/conf.d/claude-aether.fish:1-10`) checks
  aether's `/health` (2 s); if down, or no key, it starts the session with
  `ANTHROPIC_BASE_URL=https://api.anthropic.com` and no key helper (account login). Session
  restore launches `claude --resume` through `fish -C`, so it gets the fallback too
  (`config/hypr/custom/scripts/session-restore.py:74-83`). A session started from bash, an IDE
  or a timer does not.
- Retry watchdog: when both accounts are limited, aether answers 429 with the reset time and
  sessions wait up to 6 h instead of ending (`topics/aether.md` Log 2026-10-13).
- Details: `aether.md`.

## Protection layers for private data and secrets

| Layer | Where | Covers | Gap |
|---|---|---|---|
| Rules | `AGENTS.md:57-91`, device context | everything, every agent | relies on the agent |
| Guard hook | `tools/guard.sh:25-47` | media paths on both machines, by path text | does not resolve symlinks (`topics/nier.md:34-35`) |
| Managed deny | `nier/managed-settings.json:3-42` | ssh, gnupg, keyrings, restic, rclone, `.env`, keys, media | no rule for `~/.config/aether/key`, `~/.config/ciel/token` |
| User deny | `settings.json:47-62` | older media paths | duplicates, old `media-library` names |
| Ciel agent | `CIEL_REDACT` in `/etc/ciel/agent.env` | log lines naming media paths never leave the device | path list hard-coded (`dotfiles/bootstrap/setup-machine.sh:100`) |

Claude Code only: other agents (pi) would get the rules but no enforcement
(`topics/nier.md:89-91`).

## Permissions that matter for the system

- `defaultMode: auto`; `sudo`, package installs, `systemctl restart/stop/enable/disable`, force
  push and edits under `/etc` ask first (`settings.json:63-80`).
- `Edit` is allowed on `/opt/dotfiles/**`, `/opt/nier/**`, `/opt/overseer/**`; git commit and
  push are allowed (`settings.json:9-45`). An agent can change the shared desktop and agent
  config of both users and push it without a prompt.
- sudo cannot take a password from an agent shell: scripts go to the issue folder's `pending/`
  and rom runs them (`AGENTS.md:57-80`). 14 such scripts in three days (`topics/setup-steps.md:35-36`).

## Skills

`config`, `comprehension`, `burn`, `chat-review` are tracked (`git ls-files skills`);
`skills/agent-review` and `skills/skill-review` exist on disk with only a `scripts/` folder,
untracked, retired on 2026-10-10 (`topics/nier.md:64-65`).

## What nier depends on

| Needs | For | If missing |
|---|---|---|
| aether (roberta) | every model call | fish fallback; otherwise sessions wait or fail |
| Ciel (roberta) | the live brief | brief silently absent |
| overseer and 9s checkouts | indexes, TODO | context without docs |
| `/etc/machine-id` matching a machine page | device identity | UNREGISTERED banner, agent must stop |
| sudo by rom | `/etc/claude-code` copies | managed deny list goes stale |
