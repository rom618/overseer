---
name: docs
summary: How the docs repos overseer (shared, work) and 9s (rom only) work: levels, ownership, TODO, split
status: active
owner: shared
host: mikaela
repos: ~/Playground/overseer, ~/Playground/9s, /opt/nier
folder: ~/claude/issues/ciel
verified: 2026-10-11
source: /opt/nier/AGENTS.md section 2, /opt/nier/tools/gen-index.sh, ciel SPEC.md section 17, nexus README and audit-2026-10-10.md sections 1 and 7, overseer TODO.md
---

# Docs repos (overseer and 9s)

Plain Markdown in git that records how everything is set up, read by the user and by every agent
on every device, so a change is written down where the next session will look. Two repos:
`overseer` (GitHub rom618/overseer) holds everything both accounts share or work owns; `9s`
(rom618/9s) holds rom-only topics (personal apps and projects). The rules agents follow are in
`/opt/nier/AGENTS.md` section 2; this page is the state of the system.

## Facts

- Three levels, like a book: `INDEX.md` (table of contents, generated) then `topics/<name>.md`
  (state, facts, Open, Log) then detail where the work lives (code repo README, issue folder, git
  log), linked and never copied. A page over about 150 lines becomes `topics/<name>/index.md`.
- Layout of overseer: `INDEX.md`, `TODO.md`, `topics/`, `machines/<host>.md`, `runbooks/`,
  `people/rom/` (the comprehension profile). Ciel writes `inventory/index.md` and
  `journal/YYYY/MM/YYYY-MM-DD.md` (generated, never edited by hand) in both repos, routed by owner
  (rom's items to 9s, shared and work to overseer), daily at 00:30 through rom's
  `ciel-docs-sync.timer` (`/opt/nier/tools/ciel-docs-sync.sh`), since 2026-10-11.
- Topic front matter: name, summary (one line, under 120 chars), status (active, dormant, done),
  owner (rom, work, shared), host, repos, folder, verified (date), source. Template:
  `~/claude/issues/ciel/drafts/topic-page-template.md`. Machine pages have their own front matter.
- `/opt/nier/tools/gen-index.sh <repo> [--check]` builds `INDEX.md` from the front matter, warns
  about active topics not verified for 30 days and pages over 150 lines, and reports front matter
  errors (missing keys, bad status or owner, summary over 120 characters). `--check` fails on a
  stale index or an error; the pre-commit hook runs it in overseer and 9s. Never edit `INDEX.md` by hand.
- One owner per fact; elsewhere link: rules for agents in AGENTS.md; machine facts in
  `overseer/machines/<host>.md`; how to build, run and deploy a project in its repo README;
  decisions, open items and current state in the topic page; change history in git log (later the
  Ciel journal); user working preferences in agent memory only.
- Page blocks: Facts (only what is not in the code README), Open (`- [ ] text (since date)`,
  closed with `[x]`, date and commit), Log (condensed, dated, newest last, decisions with reasons,
  superseded facts dropped, pointing at a commit or file).
- TODO rule: anything the user must do later goes to `overseer/TODO.md` (rom-only items to
  `9s/TODO.md`) in the same turn: a handed-over step, a script's follow-up message, an unverified
  check. Reason: the plaintext copy that a script told the user to delete "yourself" lived only in
  that script's output and was forgotten for three weeks.
- TODO.md is hybrid (since 2026-10-11): the hand-written part above the BEGIN marker holds only items
  that belong to no topic, plus recently closed ones under Done; below it `gen-index.sh` writes every
  active and dormant topic's open `- [ ]` items (with their indented continuation lines). An item
  that belongs to a topic lives only in that topic's Open block, never in both.
- Commits in watched repos (`/opt/nier/bootstrap/watched-repos.txt`) carry a `Docs:` line naming
  the page, or `Docs: none (reason)`; the commit-msg hook refuses others. Claude's Stop hook
  `stop-docs-check.sh` catches files left uncommitted or unpushed at the end of a turn.
- rom and work: overseer is for both; sessions of work read it by absolute path; overseer never
  links into 9s. A personal project on shared infrastructure has its topic in 9s and only its
  footprint (stack, ports, jobs, checks) in overseer.
- Never in any repo: secrets (write where one lives, never the value; gitleaks in pre-commit),
  anything from private media (no paths, file names, titles, counts), and in overseer personal
  detail. Infrastructure detail (IPs, key names) in overseer is visible to both accounts, accepted
  explicitly. Reason for the personal-detail rule: overseer history will be readable by work, and a
  clean server split later needs a history-free repo.
- Agents see the docs through `nier context`, which injects the indexes (rom: both; work: overseer
  only), capped (about 500 tokens for overseer, 400 for 9s).

## Open

- [x] (2026-10-11, nier) Docs rules in force: front matter check, `Docs:` commit-msg hook, Stop hook for Claude; the owning-page rule was already in AGENTS.md section 2
- [x] (2026-10-11, nier) TODO.md hybrid: hand-written part for items with no topic, generated part with every topic's Open items (decision by rom)
- [x] (2026-10-11) Seed overseer and 9s: history.md triaged and frozen, stale memories fixed
- [x] (2026-10-10, nier c5bf44e) overseer in `/opt/overseer`
- [ ] The server gets overseer and 9s read-only (since 2026-10-10)
- [x] (2026-10-11) Issue folders: READMEs cut, nexus split and its scratch-rescue triaged
- [x] (2026-10-13, nier) Session start shows the first 5 open items of each topic page the issue README links (the top of TODO comes with `nier context`)
- [ ] rom's list (naming): French names for apps, English names for devices (as roberta); the laptop/app name clash is fixed (mikaela, Camille, 2026-10-10) (since 2026-10-12)

## Log

- 2026-10-10: decision to keep two repos: overseer and 9s, split on the ownership rule of Ciel (rom sees all, work sees work and shared). Convenience boundary, not security: both accounts have sudo.
- 2026-10-10: audit of docs design (5 agents) found staleness the main risk, so every page carries `verified` and `source`; two writers on one file (Ciel and hand edits) is avoided by generating INDEX.md and keeping Ciel's output in its own directories.
- 2026-10-10: rules 1 to 3 adopted by the user: front matter plus generated INDEX, one owner per fact, no personal detail in overseer. Agents always update the owning page when they change something.
- 2026-10-10: handoffs and issue-folder READMEs may not hold decisions or facts (second source of truth); decision goes to the topic page at decision time. Nier 61b7a33, 46c7ddd.
- 2026-10-10: overseer seeded with `machines/mikaela.md`, `roberta.md`, three runbooks and the comprehension profile (commit 126e411); this batch adds the first topic pages. CLAUDE.md "Current state" and "User differences" move to topics, machine facts to machine pages.
