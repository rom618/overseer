---
name: aether
summary: Aether, the own proxy in front of the AI provider accounts (fork of ccflare); decided, not built yet
status: active
owner: shared
host: roberta
repos: none yet
folder: ~/claude/issues/ciel
verified: 2026-10-12
source: review of the public ccflare and of the company fork, 2026-10-12 (rom's ~/claude/issues/ciel/out/repo-review-2026-10-12.md)
---

# Aether

A proxy between the agents on every device and the provider: the agents send their requests to
it, it picks one of the linked accounts (rom's and work's), puts that account's credential in
and forwards the request unchanged. It gives one view of usage and limit windows, and lets work
continue on the other account when one is limited.

## Facts

- Base: ccflare, a Bun and TypeScript monorepo (one server process, SQLite, web dashboard,
  terminal UI). Public upstream `snipeship/ccflare`, MIT. The company runs a private fork with much
  more on top (fleet management, messages between sessions, a rules engine, French UI).
- Linking an account: `bun run ccflare --add-account <name> --provider claude-code` opens a
  browser login, once per account. Per machine: a client key (`--add-client-key <machine>`),
  the server started with `CCFLARE_REQUIRE_AUTH=true`, and `ANTHROPIC_BASE_URL` plus
  `ANTHROPIC_AUTH_TOKEN` in the agent settings env.
- Strategy `session` (stay on one account for its 5 h window, move to the other when limited)
  fits two accounts and one person.
- The account tokens are stored in clear text in the SQLite file (ccflare's own security doc).
- Comparison done 2026-10-12 from the fork's own history (1338 commits after the public `95c4c6a`,
  +302,000 lines): the public version has no access control at all and ignores `PORT`; the
  account sign-in code is identical. The fork's first commit adds keys per machine and
  conversation routing; team-only features start on 2026-09-11. Proposed base: the fork at
  `961c1ffa` (+36,000 lines over public, typecheck and 526 tests pass in the `oven/bun` container),
  7 later fixes apply as they are, about 30 more are small ports. Full ledger:
  rom's `~/claude/issues/ciel/out/aether-fork-diff-2026-10-12.md`. (An earlier note here said the
  keys could not be lifted out; that was wrong.)
- Repo: rom618/aether (private), clone `~/Playground/aether`, `main` = the public version, remote
  `upstream` = snipeship/ccflare. Local branches `import` (the base) and `trial` (base plus the 7
  fixes) are not pushed.

## Open

- [x] (2026-10-12) Public upstream cloned to `~/Playground/aether` and compared with the company fork (Facts)
- [x] (2026-10-12) Repo rom618/aether created by rom, public version pushed to `main`
- [x] (2026-10-12, rom) Access: behind the proxy stack, reachable at home and through Tailscale only; usage per device wanted (comes with the base: keys per machine)
- [ ] rom: accept the base `961c1ffa` plus the ledger's lists, and settle the flagged `ccf_` example key in the base history (ask the company, or import as one commit with the line replaced); nothing is pushed before (since 2026-10-12)
- [ ] After the base: port the ledger's "by hand" list one item at a time, clean the company's names out of docs and deploy files, point error reports at Ciel (since 2026-10-12)
- [ ] Decide where it runs (a `stacks/aether` in home-server on roberta is the default), data folder root-only, whether it is backed up given the clear-text tokens (since 2026-10-12)
- [ ] Reachable at home and on the tailnet only (aether.wuling.dev through the proxy stack), never public; depends on the own tailnet (`topics/network.md`) (since 2026-10-12)
- [ ] Link the two accounts, one client key per device and user, then point the agents at it (nier `settings.json` env); check the statusline and cache-keepalive still read their limits (since 2026-10-12)
- [ ] rom and work see each other's usage through one proxy: accept, or hide per owner as Ciel does (since 2026-10-12)

## Log

- 2026-10-12: decided by rom: ccflare becomes aether; built from a new fork of the public
  upstream, keeping from the company fork only what is relevant here (the company agrees to the
  reuse). Pooling the two accounts is accepted by rom as the only user.
