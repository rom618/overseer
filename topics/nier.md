---
name: nier
summary: Agent config repo /opt/nier (was claude-config): AGENTS.md rules, hooks, skills; going agent-agnostic
status: active
owner: shared
host: mikaela
repos: /opt/nier
folder: ~/claude/issues/ciel
verified: 2026-10-11
source: /opt/nier (AGENTS.md, README.md, tools/cache-keepalive/README.md, skills/README.md), notes/history.md 2026-09-24 to 2026-10-10, nexus README and audit-2026-10-10.md, git log of /opt/nier
---

# nier (agent configuration)

Configuration for every AI agent on the laptop, shared by rom and work: rules, hooks, permission
policy, skills, helper tools. Repo `/opt/nier` (GitHub rom618/nier), linked into each home by
`bootstrap/link.sh`. Renamed from `claude-config` on 2026-10-10. Currently Claude Code only; the
plan is to make it agent-agnostic (Open). Machine facts: `machines/mikaela.md`.

## Facts

- Layout and install commands: `/opt/nier/README.md`. The dotfiles installer asks whether to set
  nier up. Pre-commit hook runs gitleaks. No secrets in the repo.
- Old paths `/opt/claude-config` and `/opt/dotfiles/claude` are links to `/opt/nier` until work
  relinks; `settings.json` carries both old and new path rules meanwhile.
- `AGENTS.md` is the rule file for every agent. `CLAUDE.md` is a link to it and
  `/etc/claude-code/CLAUDE.md` points to it. `managed-settings.json` (deny list) is a root-owned
  copy in `/etc/claude-code`: after changing it, `sudo /opt/nier/bootstrap/setup-machine.sh`.
- `bin/nier context` prints the session context in layers: AGENTS.md, device found by
  `/etc/machine-id`, `os/<family>.md`, `roles/<role>.md`, `overseer/machines/<host>.md`, the docs
  indexes, TODO. Unknown host: banner UNREGISTERED or NAME COLLISION, agent must stop and ask.
  Claude gets it from the SessionStart hook; other agents run it first.
- Hooks (`settings.json`, scripts in `tools/`, linked as `~/claude/tools`):
  - `guard.sh` (PreToolUse): hard-blocks the never-run commands and private paths, judged on the
    target. It matches path text and does not resolve symlinks, so no symlinks into vaults.
    Tests in `tools/tests/`.
  - `session-start.sh` (SessionStart): titles sessions started under `~/claude/issues/<name>`
    (via `hookSpecificOutput.sessionTitle`, not on resume), reports other sessions' uncommitted
    work, injects `nier context` and the folder's `handoffs/latest.md`, logs each injection to
    `~/.claude/hooks/cache-keepalive/consumed.log`.
  - `claude-statusline.py` (statusLine, every 10 s): writes `~/.claude/claude-rate-limits.json`
    (5h/7d windows) read by the bar's Claude icon and the keepalive.
  - `cache-keepalive/` (Stop with asyncRewake, SessionEnd): see next block.
  - `stop-docs-check.sh` (Stop): files edited this turn (Edit, Write, NotebookEdit since the last
    user message) that are uncommitted or unpushed in a git repo send the session back once
    (`stop_hook_active` stops a loop). Edits made through Bash are not seen.
  - `claude-state.sh` (UserPromptSubmit, PostToolUse(Failure): running; PermissionRequest,
    Notification permission/elicitation, PreToolUse AskUserQuestion/ExitPlanMode: waiting; Stop:
    done; SessionEnd: removed): writes `$XDG_RUNTIME_DIR/claude-state/<KITTY_PID>-<KITTY_WINDOW_ID>`
    for the bar's workspace icons (`quickshell-bar` topic); does nothing outside kitty (since 2026-10-13).
- Git hooks `bootstrap/githooks/`: pre-commit (gitleaks; docs repos: `gen-index.sh --check`),
  commit-msg (a `Docs:` line is required). Applied by `link.sh` to the repos in
  `bootstrap/watched-repos.txt` that have no hooks of their own (since 2026-10-11).
- cache-keepalive: upstream C port (`upstream/`, unmodified, received 2026-09-26) plus 12 local
  changes listed in `tools/cache-keepalive/README.md`. Behavior: nudges an idle interactive
  session (max 2, none under 20k context tokens, none at 5h or 7d usage >= 90%, nothing at all at
  100%) to keep the prompt cache warm, then asks for a handoff and closes it; also on exit and at
  80% context. Never arms for `claude -p`. Never hands off or closes while the session works
  (registry status not idle, a Bash-tool process alive, or a subagent transcript written in the
  last 2 min). Config copied per user to `~/.claude/cache-keepalive.json` (later runs only add
  missing keys, so a changed default needs a manual edit). CLI: `keepalive --status|--disable|--enable|--exclude`.
- Handoffs are session state only: English, under 1.5 KB, `~/claude/issues/<name>/handoffs/latest.md`
  (else `~/claude/issues/_unfiled/handoffs/<start-dir-slug>.md`), overwritten. Decisions go to topic pages.
- Skills (shared via `~/.claude/skills`, authoring rules in `skills/README.md`): `config`
  (device config changes, thin pointer plus scripts), `comprehension` (profile in
  `people/rom/comprehension.md`), `chat-review` (Claude-only, reads `~/.claude` transcripts), `burn` (unattended work while rom is away:
  one round of questions, a budget on weekly usage, a morning report; since 2026-10-12).
  Retired 2026-10-10 as unused: agent-review, cold-review, skill-review, inbox; their scripts live
  on as `tools/collect-changes.sh` and `tools/skill-lint.sh`. `skills/synced/` (Anthropic
  skills) stays out of the shared set. Do not name a skill like a bundled command (`code-review`,
  `simplify`, `init`): it would replace the bundled one.
- Working directories per user: `~/claude/issues/<name>/` is the unit of work (`README.md` goal and
  rerun command, `in/`, `out/`, `pending/` for sudo scripts, `handoffs/latest.md`); sessions outside
  one use `issues/_unfiled/`; `~/claude/scratch/tmp/` is throwaway, cleaned after 30 days by a
  user timer. The inbox hook, inbox/outbox folders and fish helpers are retired. Backup keeps only
  small documents of `issues/` (`topics/backups.md`).
- Memory is per starting directory and holds user preferences only.
- Gotchas with root cause:
  - `Write(path)` permission rules are never matched by Claude Code, only `Edit(path)` covers writes.
  - Permission prompts match the whole Bash text, heredoc bodies included: write long files with the Write tool.
  - The hook payload `cwd` follows every `cd` into an allowed dir, so handoffs landed under `/opt/...`:
    the keepalive keys on the transcript's first `cwd`.
  - Registry status written at the end of a turn still says busy: read `~/.claude/sessions/<pid>.json`
    fresh at the deadline (else it nudged forever, 2026-09-27).
  - `claude -p` registers as `kind: interactive` (only `entrypoint` is `sdk-cli`) and waits for async Stop hooks.
  - `pkill -f` kills the agent's own shell; use `pkill -x` or a pid file.
  - Upstream `read_file` sized reads by fstat, which is 0 for `/proc`: the recycled-pid check never worked.

## Open

- [x] (2026-10-14, rom; checked from roberta: "Hi rom618") The runner's GitHub key is on rom's account
- [ ] work: run `/opt/nier/bootstrap/link.sh` from a work session, then remove `/opt/claude-config`, `/opt/dotfiles/claude` and the old-path rules in settings.json (since 2026-10-10)
- [ ] 2026-10-24: review cache-keepalive counters (`consumed.log`, "repris la main" lines in per-session logs): keep or drop exit background resume, 80% rotation, `relaunch_cmd` (since 2026-10-10)
- [ ] Agent-agnostic, deferred until a second agent is adopted (decision 2026-10-11): `policy/` data generating each agent's deny rules (`adapters/claude`, `adapters/pi`), skills in spec format (no `${CLAUDE_SKILL_DIR}`, no tool names, own Hard rules block), Claude-only skills under `agents/claude/`, the comprehension skill for every agent, the dotfiles installer asking for which agents (since 2026-10-10)
- [ ] Try pi (pi-coding-agent) on a throwaway repo; it has no permission system or sandbox, so deny rules need an extension or OS isolation. rom 2026-10-14: dropped for now, Claude does not run on it; the runners stay on `claude -p` (since 2026-10-10)
- [ ] Media privacy at OS level for every agent, not only Claude deny rules (since 2026-10-10)
- [ ] Phone access: Tailscale + tmux on an always-on host + Termius; Claude Remote Control for laptop sessions; laptop sleeps, roberta is tight on RAM (since 2026-10-10)
- [ ] Conversations hub: per-agent adapters into SQLite FTS5 plus Markdown mirror, claude.ai exports, rom/work split, never in docs repos (since 2026-10-10)
- [ ] Runners (decided 2026-10-13, rom): three roles, each with its own aether key, context and rules: burn-runner (the spare-quota tasks below, only when quota would be lost, in a container on roberta, proposes branches, never `main`), help-runner (the agent reached from Ciel, on demand; rom 2026-10-15: access through Ciel only: it reads what Ciel knows, asks for actions (start, stop, restart, run) that run only after rom taps Approve in the chat, code fixes as branches; no shell on the server; live 2026-10-15: Ciel f701ab2 Help page, home-server 2e5a1f5 container `runner-help`, aether key `runner-help`; tested locally with a stand-in claude, first real conversation by rom pending), pantry-runner (Pantry-cook's own `claude -p` calls, in its stack, Haiku or Sonnet). aether regulates usage per key (allowed models, pause thresholds), Ciel regulates work (queue, hours, reports, approvals, journal). Order: per-key rules in aether (done 2026-10-13, aether 0a076ff9), then burn (Ciel side done 2026-10-13, ciel 267d3c2: `/runner` page, task queue, gate on idle time, night or usage about to be lost, reports; runner stack built 2026-10-14, home-server 4bb1f5b: Claude runs as its own user without the GitHub key or Ciel token, changes come back as a patch pushed to `runner/burn-<run>`; tested locally with a fake Ciel and a local repo, not yet with a real model. Live 2026-10-14 23:29 (rom ran `bin/hs push` and `bin/hs runner-keys`; key `runner-burn` policy Haiku/Sonnet/Opus paused above 50 % 5h or 90 % week): container waiting on Ciel's gate, a test call reached aether and was paused by the policy as designed. Left: rom adds tasks on Ciel's /runner page (admin login; the agent has only a read token), first one proposed: dependency updates for Ciel, rom618/ciel, Sonnet, every 7 days; then check its first real run and report), then help. rom's list (2026-10-15): explore the other shape too, one runner service outside containers taking jobs from a queue, against today's one container per runner role. Compared 2026-10-15 (ciel session): idle runner containers cost 1 to 5 MB; containers keep each role's keys apart and keep Claude off the host (rom's help decision); proposed: keep one container per role, merge the two loops into one program with roles as settings, keep Ciel as the one queue, and do host jobs (restore test, drift) as Ciel agent actions the runner requests. rom 2026-10-15: agreed (one program for every runner, roles as settings), and the engine stays swappable (`claude -p` today, pi or another later: one adapter per engine) (since 2026-10-13)
- [ ] Spare quota: a user timer reads `~/.claude/claude-rate-limits.json` and, when much of the 5h/7d quota is left, runs queued `claude -p` tasks one at a time, logged, never pushing without review (00:00 to 05:00 daily, 08:00 to 19:00 weekdays); one aether key per runner (`runner-*`), so their usage shows apart from development in Ciel's charts (rom's list, 2026-10-12). The 11 task ideas (nexus session, 2026-10-10): (1) docs audit, topic and machine pages against reality, stale `verified` dates; (2) tests and (3) dependency updates of Camille and Ciel (cargo, npm) as review branches; (4) security sweep: gitleaks over every repo, open ports, server drift vs its repo; (5) review of the day's commits; (6) research items from the TODO (Vaultwarden plan, iPhone recovery, GitHub repos inventory); (7) Ciel milestones in a worktree; (8) mod update changelogs for the Minecraft pack; (9) monthly restore test of the backups; (10) refresh of the apartment search and (11) of the plot search (since 2026-10-10)
- [ ] Multi-device: `nier register` (machine page from detection, on a device branch), `nier install --profile minimal|server|desktop-arch`, `$NIER_HOME` instead of hard-coded paths, laptop-only parts in a desktop module, host paths in `policy/paths.d/`, enrollment through Ciel, macOS and Windows undecided (since 2026-10-10)
- [ ] Session registry: append-only JSONL per device (agent, device, user, cwd/topic, status, resume command) fed by hooks, later synced to Ciel (since 2026-10-10)
- [x] (2026-10-11) CLAUDE.md is AGENTS.md (rules only, about 2.6k tokens); `notes/history.md` triaged and frozen
- [ ] rom's list: not depend on Claude: switch easily between AI providers (aether already routes Codex and OpenAI accounts; the agent-agnostic nier above) (since 2026-10-12)
- [ ] rom's list: use Claude Code from the phone (the tailnet reaches the laptop and roberta; options: Remote Control, a web session, ssh from the phone) (since 2026-10-12)
- [x] (2026-10-12) From rom's list: agent config separate from the desktop config repo: done since 2026-10-11 (`/opt/nier`, its own `bootstrap/link.sh`); the install prompt in the config repo is the open part
- [x] (2026-10-13, checked) The dotfiles installer already asks for the agent config (`bootstrap/install.sh` step 9 clones rom618/nier and runs its installer); nothing to do
- [ ] New device or user: the nier installer should also make the aether key (`hs aether-key-save NAME`, needs ssh to roberta) or warn that sessions fail without `~/.config/aether/key` (settings.json sends every session to aether since 2026-10-12) (since 2026-10-13)
- [ ] rom's list: one app holding every conversation, reachable from every computer, nothing lost (9s conversations topic has the archive of 2026-10-09) (since 2026-10-12)
- [ ] rom's list: a `claude -p` always running that rom talks to from an app, which runs commands on every app on request and queues jobs (Ciel's agents and actions are the safe channel) (since 2026-10-12)
- [x] (2026-10-14, nier, home-server 31f9b6d and 7c819f7, dotfiles 83b2798) Chat review of 2026-10-13/14 (rom's `~/claude/issues/overseer/out/chat-review-2026-10-14.md`), all 7 done: deploy commands allowed in `settings.json` (`bin/hs push`, `bin/deploy`, Caddy reload; first `hs push` after it ran unblocked); AGENTS.md: Read before Edit, both docs paths, diff a file before staging it, deploys chained with `&&`; `hs push` one at a time (local lock, server lock) and lists what it deploys; pre-commit judges only staged pages, stale-status lint on the system wiki (docs topic); stack env examples renamed `example.env` (aether's `.env.example` left to its session); `gh` installed and signed in by rom
- [ ] The deploy allow rules do not stop the auto mode classifier: on 2026-10-15 camille's `bin/deploy --roberta` was still refused as "Production Deploy" (`bin/hs push` passes). Until a fix is found, rom says "deploy" in the session that built the change; never deploy another session's refused build from here (since 2026-10-15)

## Log

- 2026-09-24: Claude config split out of dotfiles into its own repo (history kept by subtree split); dotfiles installer offers it. 587cb44.
- 2026-09-26: overhaul from a review of the replicant repo: CLAUDE.md cut from 83 KB to 21 KB (log moved to history.md), commit attribution off, guard/session-start/keepalive/statusline hooks, skills hardened. 1b1889b.
- 2026-09-26: keepalive switched to the upstream C port plus local patches (8c514ca); guards for subscription limits and "never while working" added (a4a5866); 100/100 upstream tests, tests-local.sh passes.
- 2026-09-27: keepalive stale-busy fix: status re-read at the deadline. 51ba8ba.
- 2026-10-08: sessions in `~/claude/issues/<name>` titled after the folder. 479308b.
- 2026-10-09: handoffs per issue folder keyed on the start dir; three memory-only rules (no Chrome, chmod after Write, ydotool) moved into CLAUDE.md because memory is per start dir. 5aefbed.
- 2026-10-10 decisions (audit of five read-only agents, `audit-2026-10-10.md`): AGENTS.md plus one policy so skills and rules do not depend on Claude; agent from the phone and one conversations hub wanted; the installer stays separate from dotfiles and keeps asking; CLAUDE.md was 86% of the fixed session load (about 6.4k tokens, 44% machine facts), target AGENTS.md under 3k tokens plus capped indexes.
- 2026-10-10: skills pruned after a month of transcripts (config 7 uses, inbox 14, chat-review 2, others 0; unused descriptions cost about 400 tokens per session). d001f85.
- 2026-10-10: rename to nier, old paths kept as links. d4d14a5. Guard protects Camille's new paths and every `app-data*` name. 78bdfdc.
- 2026-10-10: AGENTS.md universal, `nier context` layers injected at session start, comprehension skill. d8fb35c.
- 2026-10-10: issue folders with `in/ out/ pending/ handoffs/latest.md`, `_unfiled`; inbox hook, `commit.sh` (`git add -A`) and `append-current-state.sh` retired; scratch cleaner limited to `scratch/tmp`. 46c7ddd. Reason: inbox dead since 2026-09-17, scratch held a 793 MB ISO, and a Read consumed inbox files.
- 2026-10-10: handoffs reshaped (state only, one `latest.md`, no chains), consumed counter added. Reason: 34 handoffs in 14 days, about 1 in 5 read, mixed languages, copy-forward chains as a second source of truth. 61b7a33.
- 2026-10-10: keepalive kept (44 nudges, about half the idle episodes ended with the user back); heavy parts to be judged on 2026-10-24.
