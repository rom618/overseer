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

## Open

- [ ] Create the repo as a new fork of the public upstream; diff the company fork against it and list what is worth carrying over (first candidates: client keys per machine if not upstream, the abnormal-exit report, the memory guard, the spend alert on the slope); leave out fleet management, the message bus and the rules engine (since 2026-10-12)
- [ ] Decide where it runs (a `stacks/aether` in home-server on roberta is the default), data folder root-only, whether it is backed up given the clear-text tokens (since 2026-10-12)
- [ ] Reachable at home and on the tailnet only (aether.wuling.dev through the proxy stack), never public; depends on the own tailnet (`topics/network.md`) (since 2026-10-12)
- [ ] Link the two accounts, one client key per device and user, then point the agents at it (nier `settings.json` env); check the statusline and cache-keepalive still read their limits (since 2026-10-12)
- [ ] rom and work see each other's usage through one proxy: accept, or hide per owner as Ciel does (since 2026-10-12)

## Log

- 2026-10-12: decided by rom: ccflare becomes aether; built from a new fork of the public
  upstream, keeping from the company fork only what is relevant here (the company agrees to the
  reuse). Pooling the two accounts is accepted by rom as the only user.
