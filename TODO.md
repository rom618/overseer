# TODO (shared and work)

Hand-kept until `gen-index.sh` generates it from the topic pages' Open blocks. Add `- [ ] text
(since YYYY-MM-DD)`, close with `[x]`, the date and the commit. rom's personal list is `9s/TODO.md`.
Sources: the Ciel planning folder (`~/claude/issues/nexus`, README and `audit-2026-10-10.md`),
`/opt/claude-config/notes/history.md`, CLAUDE.md "Open, user action".

## Structure (before coding Ciel, in this order)

- [x] (2026-10-10, nier d001f85) Skills: cold-review, agent-review, skill-review retired, scripts kept in tools/
- [ ] Docs rules: topic front matter (status, owner, host, verified, source), `gen-index.sh`
      generating INDEX.md and this TODO, AGENTS.md rule "update the owning page when you change
      something", pre-commit `Docs:` check, Stop hook for Claude (since 2026-10-10)
- [ ] Issue folders: README cut to goal + topic link + rerun command; decisions and open items go
      to the topic page at decision time; handoffs become session state only (keepalive prompt
      change); split the `nexus` folder into topic pages (since 2026-10-10)
- [x] (2026-10-10, nier d4d14a5) `/opt/claude-config` -> `/opt/nier`, rom relinked, memory dir copied
- [ ] work: run `/opt/nier/bootstrap/link.sh` from a work session; then remove the links
      `/opt/claude-config` and `/opt/dotfiles/claude` and the old-path rules in settings.json (since 2026-10-10)
- [x] (2026-10-10) Hostname `camille` -> `mikaela`: script run, avahi keeps `camille.local`
      (resolves), 39 restic snapshots relabelled `mikaela`; relogin pending (user restarts when needed)
- [ ] nier agent-agnostic: AGENTS.md canonical (CLAUDE.md imports it), `policy/` generating each
      agent's deny rules, shared skills in spec format (no `${CLAUDE_SKILL_DIR}`, no tool names),
      Claude-only skills (chat-review, part of skill-review) under `agents/claude/`; the dotfiles
      installer keeps asking whether to set nier up, and for which agents (since 2026-10-10)
- [ ] Comprehension skill: draft audited, installed for every agent (since 2026-10-10)
- [ ] Seed overseer and 9s; move overseer to `/opt/overseer` (sudo); triage history.md into topic
      pages; shrink CLAUDE.md to rules; fix stale memories (since 2026-10-10)
- [ ] ssh config: one alias per host, keys named for what they open; unused EPITA key and
      `known_hosts.old`; passphrases; rotate the two keys leaked in the old dotfiles history
- [ ] Then: code Ciel (SPEC.md milestones)

## Agents and access

- [ ] 2026-10-24: review cache-keepalive with its counters (`consumed.log`, nudge returns in
      the logs): keep or drop the exit background resume, the 80% rotation, `relaunch_cmd`
      (since 2026-10-10)
- [ ] Try pi on a throwaway repo (skills, extension blocking, sessions) before relying on it
- [ ] Media privacy enforced at the OS level for every agent, not only by Claude's deny rules
- [ ] Phone access to agents: Tailscale + tmux on an always-on host + Termius; Claude Remote
      Control for laptop sessions
- [ ] Conversations hub: one archive of every agent's sessions and claude.ai exports, rom/work
      split, kept out of the docs repos

## Network

- [ ] Own Tailscale tailnet for remote access to the self-hosted apps (Headscale later needs a VPS)
- [ ] vesscam Headscale tailnet (Forgejo): currently logged out, control server unreachable;
      make it work-owned (`tailscale set --operator=work`)
- [ ] Pi-hole
- [ ] Replace `camille.local` (avahi mDNS, kept on mikaela for the app) with a Tailscale MagicDNS
      name; first agree on one naming rule for devices and apps (since 2026-10-10)
- [ ] Laptop: Caddy 80/443 listens on every interface, also on untrusted Wi-Fi

## Servers

- [ ] Separate personal and work servers (roberta is shared only for now): own restic password,
      repo, Drive account, proxy; parameterize `offsite-backup.sh`
- [ ] Dedicated Ciel box (Raspberry Pi 5 + SSD)
- [ ] roberta before Ciel and Vaultwarden: HTTPS reverse proxy; sshd `PasswordAuthentication no`,
      `PermitRootLogin no`, `AllowUsers server` in `host/setup.sh`; firewall or loopback binds;
      Dockge not open on the LAN without auth; cAdvisor not privileged; review sudo NOPASSWD
- [ ] roberta: reboot for the pending kernel; Docker repo in unattended-upgrades or a monthly
      upgrade; pin image versions; prune old images
- [ ] roberta backups: monitoring, grafana, dockge, worlds-archive have none; restic.pw, .env
      files and rclone.conf exist only on the box; rehearse a restore on another machine
- [ ] Terraria stack stopped but `terraria-backup.timer` runs hourly; check the playit tunnel
- [ ] Ethernet instead of Wi-Fi if possible (no UPS either); README memory figures are stale

## Ciel

- [ ] Code it (after the structure section); retire Dockge (milestone 6) and Grafana (7)
- [ ] Watch Vaultwarden and the other self-hosted apps; network gear (Flybox) later

## Passwords

- [ ] Vaultwarden on the server: VPN only, HTTPS, its own backup, 2FA, master password and a
      recovery copy offline
- [ ] Pass over every password into it (inventory: security audit, 2026-10-10); machine secrets
      read by timers stay 0600 files, with a recovery copy in the vault

## Backups

- [ ] Separate project: back up every GitHub repo and its commits
- [x] (2026-10-10, dotfiles 387c6a8) Never-backup list `backup/never.txt`, used by restic and git-snapshot
- [ ] `git-restore.sh` checks out the remote tip, not the recorded HEAD; drops patches silently
- [ ] Re-run `system-secrets.sh` (Wi-Fi profiles changed since 2026-09-12)
- [ ] `~/.config/mc-server/home-server-restic.pw` is not backed up; decide what in `~/.config`,
      `~/.ollama`, `~/Downloads` is disposable on purpose
- [ ] work: its own restic setup, `link.sh`, relogin for `dots`, docker group, delete
      `/home/work/.local/state/dotfiles-backup/*`

## Laptop

- [ ] Track `/etc/systemd/system/ollama.service.d/override.conf` in `system/`
- [ ] `packages-aur.txt` still lists `illogical-impulse-*` (next `ii-update.sh --deps`)
- [ ] Verify: notification left click focuses the sender; session restore at a real login;
      Surfshark after the stale-kernel reboot
- [ ] `append-current-state.sh` anchors on a heading that no longer exists
- [ ] Stale text: CLAUDE.md tells work to disable `hypr-session-save.service` (now a timer);
      `system/README.md` says lid.conf needs a reboot
- [ ] Uncommitted changes of other sessions: `bootstrap/packages.txt`, `config/fish/config.fish`
      (`alias cd-mcs` with a hard-coded /home/rom), `/opt/claude-config/settings.json`
- [ ] Triage `~/claude/scratch/*.sh` helpers and the `~/claude/scratch/claude-config` clone with a
      modified settings.json (deleted by the 30-day cleanup)
- [ ] Docker: `pregen_default` network, 4 stopped anonymous containers, ~10 GB of images
- [ ] Clovis work notes sit in rom's `~/Desktop/work` (in rom's personal backup): move to `work`
