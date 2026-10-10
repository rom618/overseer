---
name: docs
summary: How the docs repos overseer (shared, work) and 9s (rom only) work: levels, ownership, TODO, split
status: active
owner: shared
host: mikaela
repos: ~/Playground/overseer, ~/Playground/9s, /opt/nier
folder: ~/claude/issues/ciel
verified: 2026-10-10
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
  `people/rom/` (the comprehension profile). Later Ciel writes `inventory/` and `journal/` (generated, never edited by hand).
- Topic front matter: name, summary (one line, under 120 chars), status (active, dormant, done),
  owner (rom, work, shared), host, repos, folder, verified (date), source. Template:
  `~/claude/issues/ciel/drafts/topic-page-template.md`. Machine pages have their own front matter.
- `/opt/nier/tools/gen-index.sh <repo> [--check]` builds `INDEX.md` from the front matter, warns
  about active topics not verified for 30 days and pages over 150 lines. `--check` fails when the
  index is stale. Never edit `INDEX.md` by hand.
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
  that script's output and was forgotten for three weeks. Until the generator exists TODO.md is hand-kept;
  a topic Open line already in TODO.md is written as a short line plus "(TODO)".
- Commits in watched repos carry a `Docs:` line naming the page, or `Docs: none (reason)`.
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

- [ ] Docs rules in force: front matter check and generated INDEX and TODO from Open blocks, AGENTS.md rule to update the owning page, pre-commit `Docs:` check, Stop hook for Claude (TODO)
- [ ] Seed overseer and 9s: topic pages from `notes/history.md` (a triage, not a copy), then freeze history.md with a pointer; fix stale memories (TODO)
- [ ] Move overseer to `/opt/overseer` (owner rom:dots, group writable; needs a step in nier `setup-machine.sh`, so a sudo handoff); 9s stays under rom's home; the server gets both read-only (TODO)
- [ ] Issue folders: README cut to goal plus topic link plus rerun command; split the `nexus` folder into topics ciel, docs, nier and cleanup (TODO)
- [ ] Session start shows the topic's Open block and the top of TODO (since 2026-10-10)

## Log

- 2026-10-10: decision to keep two repos: overseer and 9s, split on the ownership rule of Ciel (rom sees all, work sees work and shared). Convenience boundary, not security: both accounts have sudo.
- 2026-10-10: audit of docs design (5 agents) found staleness the main risk, so every page carries `verified` and `source`; two writers on one file (Ciel and hand edits) is avoided by generating INDEX.md and keeping Ciel's output in its own directories.
- 2026-10-10: rules 1 to 3 adopted by the user: front matter plus generated INDEX, one owner per fact, no personal detail in overseer. Agents always update the owning page when they change something.
- 2026-10-10: handoffs and issue-folder READMEs may not hold decisions or facts (second source of truth); decision goes to the topic page at decision time. Nier 61b7a33, 46c7ddd.
- 2026-10-10: overseer seeded with `machines/mikaela.md`, `roberta.md`, three runbooks and the comprehension profile (commit 126e411); this batch adds the first topic pages. CLAUDE.md "Current state" and "User differences" move to topics, machine facts to machine pages.
