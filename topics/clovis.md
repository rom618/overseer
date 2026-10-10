---
name: clovis
summary: Work startup project (working name Clovis): three networks, no repo yet, planning only
status: dormant
owner: work
host: mikaela
repos: none
folder: none
verified: 2026-10-10
source: ~/Desktop/work/clovis/note.txt, memory clovis-three-network-setup (2026-09-26 session in rom's account)
---

# Clovis (work startup project)

A start-up project of the work account. The only session so far (2026-09-26) produced a plan and a
list of questions; nothing was built, there is no repo and no infrastructure. The project's real
name is not Clovis: the user asked to be asked for proper repo names before anything is created.
Project files sit unsorted in one folder on another machine, not on the laptop.

## Facts

- Three networks: N1 deployment site (runs the product, probably not controlled by the team), N2
  wherever the user works from (this laptop, client only), N3 office or lab (git server, a Jetson for
  development and tests, possibly a jump host).
- Plan proposed, not decided:
  - Connect networks with an overlay VPN (Tailscale; Headscale if everything must be self-hosted)
    instead of port forwarding, ACL tags per role. N1 must keep working when the tunnel is down.
  - Self-hosted Forgejo (or Gitea) in N3 with CI, registry and LFS or DVC for models and data,
    plus a push mirror to a private company repo as off-site copy; protected main, releases as tags.
  - Repo layout with `src/`, `config/`, `deploy/`, `docker/`, `scripts/`, `tests/`, `docs/` (adr,
    runbooks, network map). Take an untouched tar snapshot of the current folder first, inventory it,
    then make a clean first commit.
  - In-repo agent configuration modeled on `/opt/nier` (AGENTS.md, permission rules denying `.env`,
    keys and customer data, skills for Jetson tests, release and deploy, a gitleaks hook). Agents get
    no unattended access to N1; deploys go through CI. Use a company plan for the agent, not the
    personal subscription.
  - Early items: pin JetPack/L4T, arm64 build on a Jetson runner, time sync (no RTC battery),
    read-only root or watchdog, monitoring of the deployed device, GDPR decision on what leaves N1.
- Questions waiting for the user: language and framework and the deploy device's architecture; who
  controls N1 and whether a company VPN exists; hosted versus self-hosted coordination and forge;
  team size now and in six months and which accounts (domain, GitHub org, Claude plan); size and
  location of the existing folder and whether it holds credentials; whether to work from the work
  account.
- Where the notes are: `~/Desktop/work/clovis/note.txt` in rom's home, and the session memory under
  rom's `~/.claude/projects/`. Both are in rom's personal restic backup (see `topics/backups.md`),
  so company information there leaves the work boundary.

## Open

- [ ] Move the Clovis notes from rom's `~/Desktop/work` (personal backup) into the work account; keep company code and notes out of rom's backup (since 2026-10-10)
- [ ] Ask the user for the project's proper name and the repo names before creating anything (since 2026-10-10)
- [ ] Get the answers to the questions above before building the repo skeleton and the ssh config template (since 2026-09-26)

## Log

- 2026-09-26: first session in rom's account: plan for networks, repo and agent configuration, with the list of questions; nothing built. Flagged that rom's `~/Desktop` and `~/.claude` go to a personal Google Drive backup while the work account exists for this.
- 2026-10-10: topic page created from the notes; status dormant until the user answers.
