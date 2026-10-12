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
- Repo: rom618/aether (private), clone `~/Playground/aether`, `main` = the public version plus the
  imported base and picked fixes, remote `upstream` = snipeship/ccflare. Local branches `import`
  and `trial` hold the company's original commits and are never pushed.

## Open

- [x] (2026-10-12) Public upstream cloned to `~/Playground/aether` and compared with the company fork (Facts)
- [x] (2026-10-12) Repo rom618/aether created by rom, public version pushed to `main`
- [x] (2026-10-12, rom) Access: behind the proxy stack, reachable at home and through Tailscale only; usage per device wanted (comes with the base: keys per machine)
- [x] (2026-10-12, aether eeca4e5a and 10 picks) Base accepted by rom and landed on `main` as one commit (the flagged example key replaced, rom's choice), plus 10 later fixes applied as they are (7 general, 3 for Codex and GPT accounts). Typecheck passes, 575 tests pass, 1 fails for lack of `curl` in the test image; gitleaks clean; pushed
- [ ] Port by hand, one at a time, the ledger's "by hand" list, and the Codex and GPT fixes rom wants kept that did not apply as they are (`fd8a4609`, `f260eefc`, `8c1d7770`, `8d650348`, `122a2178`, `653cdcac`, `1eeae924` with its test `99ec6a56`, `03296fc6`, `80c09bf2`, `5f96c598`); they need the fork's history, which is in `~/Downloads/ccflare.zip` (remote `keepers` of the clone points at a scratch copy) (since 2026-10-12)
- [ ] Clean the company's names out of docs and deploy files, rename to aether, point error reports at Ciel, write the README (how to build, run, deploy); add the repo to nier `watched-repos.txt` (since 2026-10-12)
- [x] (2026-10-12, home-server 069b880, aether c93b8a51) Runs on roberta as `stacks/aether`, https://aether.wuling.dev behind the proxy stack, data `/srv/data/aether` (0700), access control on; health answers, the API refuses without a key (checked)
- [ ] Back up `/srv/data/aether` or not (clear-text account tokens; losing it only costs the two logins and the usage history) (since 2026-10-12)
- [x] (2026-10-12) Reachable at home and on the tailnet only: the name points at the LAN address and the tailnet rules cover port 443
- [x] (2026-10-12, rom) Admin key made, both accounts linked from the dashboard (health: 2 accounts, 2 usable)
- [x] (2026-10-12, home-server 4ed3064) First device key `mikaela-rom`, saved by `bin/hs aether-key-save` to rom's `~/.config/aether/key` (never displayed); one real agent request answered through aether and counted on that key. The agent's address is `https://aether.wuling.dev/v1/claude-code` (the bare host answers "model does not exist")
- [x] (2026-10-12, dotfiles `conf.d/aether.fish`) rom accepted losing the account's connectors; rom's agents on mikaela go through aether from the next shell (falls back to the account when aether does not answer)
- [ ] After rom restarts a session through aether: check the statusline limits and cache-keepalive still work, pick `session` or `conversation` routing, work's key once the work account is reset (since 2026-10-12)
- [ ] Name (decided 2026-10-12): visible name to aether now (dashboard, terminal texts, README, docs); internals (package names, setting names, key prefix accepting both) after the hand ports (since 2026-10-12)
- Company fork last commit: 2026-10-12 00:19 (still about 50 commits a day); our base is its state of 2026-09-11.
- [ ] rom and work see each other's usage through one proxy: accept, or hide per owner as Ciel does (since 2026-10-12)

## Log

- 2026-10-12: decided by rom: ccflare becomes aether; built from a new fork of the public
  upstream, keeping from the company fork only what is relevant here (the company agrees to the
  reuse). Pooling the two accounts is accepted by rom as the only user.
