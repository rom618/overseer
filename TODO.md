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
- [x] (2026-10-10) overseer moved to /opt/overseer (shared with dots); seeded with topic pages
- [ ] Seed overseer and 9s; triage history.md into topic
      pages; shrink CLAUDE.md to rules; fix stale memories (since 2026-10-10)
- [x] (2026-10-10) rom ssh keys: id_ed25519_github, id_ed25519_roberta, id_ed25519_epita_forge, random
      passphrases in the login keyring, loaded at login (dotfiles 4e6d64d), `IdentityAgent` in
      ~/.ssh/config; roberta verified, old id_ed25519_server retired everywhere. (A first GitHub check was
      wrong: the old key answered; ssh-retire-old.sh now tests with the config ignored.)
- [x] (2026-10-10) New keys registered on GitHub and EPITA, verified with the config ignored; old
      id_ed25519 and id_ed25519_epita deleted. Keys left: id_ed25519_github, _roberta, _epita_forge
- [ ] WORK SESSION: vesscam-dev (user `associe`) and forgejo (`mirador`, port 2222, user `git`) are
      work's now and were removed from rom's ssh config on 2026-10-10. In the work account: create
      `~/.ssh/id_ed25519_vesscam` the same way (random passphrase in work's login keyring:
      `secret-tool store --label=... nier ssh-key path <file>`, then `ssh-keygen -N`), enable
      `ssh-agent.socket` for work, add Host blocks for both with that key, have the key registered
      on both (Forgejo profile > SSH keys; vesscam-dev through its admin), and make work the
      operator of the vesscam tailnet (`sudo tailscale set --operator=work`) (since 2026-10-10)
- [x] (2026-10-10) rom618/config: the leaking commits are on no branch or tag (only main, fresh since
      2026-09-12); no recreation (user's decision). The leaked keys get replaced; the passwords Firefox
      held on 2026-09-12 get changed during the Vaultwarden pass
- [ ] Then: code Ciel (SPEC.md milestones)

## Agents and access

- [ ] Use the Claude plan's spare quota: a user timer reads `~/.claude/claude-rate-limits.json` and,
      when a lot of the 5h/7d quota is left, runs queued agent tasks (`claude -p`, one at a time,
      logged, never pushing without review) from 00:00 to 05:00 every day and 08:00 to 19:00 on
      weekdays (since 2026-10-10). Task ideas, none queued yet:
      docs audit (topic pages and machine pages against reality, stale `verified` dates); tests and
      dependency updates of Camille and Ciel (cargo, npm) as review branches; security sweep (gitleaks
      over every repo, open ports, server drift vs its repo); review of the day's commits; research
      items from this TODO (Vaultwarden plan, iPhone recovery, GitHub repos inventory); Ciel
      milestones in a worktree once coding starts; mod update changelogs for the Minecraft pack;
      monthly restore test of the backups; refresh of the apartment and plot searches
- [ ] 2026-10-24: review cache-keepalive with its counters (`consumed.log`, nudge returns in
      the logs): keep or drop the exit background resume, the 80% rotation, `relaunch_cmd`
      (since 2026-10-10)
- [ ] nier multi-device: `nier register` (machine page from detection), install profiles
      (minimal, server, desktop-arch), `nier doctor`, Linux bootstrap script (since 2026-10-10)
- [ ] Session registry across agents and devices (JSONL per device, later synced to Ciel)
- [ ] Agents on servers run as a scoped `agent` user, not the NOPASSWD `server` user
- [ ] Try pi on a throwaway repo (skills, extension blocking, sessions) before relying on it
- [ ] Media privacy enforced at the OS level for every agent, not only by Claude's deny rules
- [ ] Phone access to agents: Tailscale + tmux on an always-on host + Termius; Claude Remote
      Control for laptop sessions
- [ ] Conversations hub: one archive of every agent's sessions and claude.ai exports, rom/work
      split, kept out of the docs repos

## Network

- [ ] Own Tailscale tailnet for remote access to the self-hosted apps (Headscale later needs a VPS)
- [ ] vesscam Headscale tailnet (Forgejo): connected again (control server was unreachable on the
      morning of 2026-10-10); make it work-owned (`tailscale set --operator=work`)
- [ ] Pi-hole
- [ ] Replace `camille.local` (avahi mDNS, kept on mikaela for the app) with a Tailscale MagicDNS
      name; first agree on one naming rule for devices and apps (since 2026-10-10)
- [ ] Laptop: Caddy 80/443 listens on every interface, also on untrusted Wi-Fi

## Servers

- [ ] Separate personal and work servers (roberta is shared only for now): own restic password,
      repo, Drive account, proxy; parameterize `offsite-backup.sh`
- [ ] Dedicated Ciel box (Raspberry Pi 5 + SSD)
- [x] (2026-10-10) roberta sshd: keys only, no root, AllowUsers server (setup.sh, verified);
      terraria-backup timer off while its stack is stopped
- [ ] roberta before Ciel and Vaultwarden: HTTPS reverse proxy; firewall or loopback binds;
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

- [ ] Decide whether ssh keys go into Vaultwarden (Bitwarden's ssh agent) and how keys are handled:
      one per device and purpose, passphrases, rotation, revocation when a device is lost (since 2026-10-10)
- [ ] Vaultwarden on the server: VPN only, HTTPS, its own backup, 2FA, master password and a
      recovery copy offline
- [ ] Pass over every password into it (inventory: security audit, 2026-10-10); machine secrets
      read by timers stay 0600 files, with a recovery copy in the vault

## GitHub

- [ ] Sort out the GitHub repos: list them, name and describe them consistently, archive or delete the
      dead ones, check visibility (private/public), topics (since 2026-10-10)

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

- [ ] Clean up the issue folders: done 2026-10-10 for READMEs and topic links (all folders) and
      nexus; left: trim location and perfect_plot READMEs to method + rerun (from their own
      sessions), file `nexus/scratch-rescue/` into topic pages, prune location's 1.3 GB of raw data
      (since 2026-10-10)
- [ ] Clean up the laptop's file system: home layout (Desktop, Playground, Documents, Downloads),
      leftovers (old repo copies, ISOs, stray files like ~/.bash_profile.bak), a documented place for
      each kind of thing (since 2026-10-10)
- [x] (2026-10-10, dotfiles 4163eb5) Untracked /etc changes now in `system/` and setup-machine.sh:
      avahi, nsswitch, fuse, Camille CA, Ollama override
- [x] (2026-10-10) Crafty-era wg0.conf and the mcvpn sudo rule removed by the user
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
