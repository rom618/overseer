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

- [ ] rom: deploy aether `22d830aa` (`~/Playground/aether/bin/deploy`; an agent's deploy was refused by auto mode): an account at its own limit no longer files an error in Ciel; then resolve Ciel issue 14 (`RateLimitError`, the agents' token is read-only) (since 2026-10-13)
- [x] (2026-10-12) Public upstream cloned to `~/Playground/aether` and compared with the company fork (Facts)
- [x] (2026-10-12) Repo rom618/aether created by rom, public version pushed to `main`
- [x] (2026-10-12, rom) Access: behind the proxy stack, reachable at home and through Tailscale only; usage per device wanted (comes with the base: keys per machine)
- [x] (2026-10-12, aether eeca4e5a and 10 picks) Base accepted by rom and landed on `main` as one commit (the flagged example key replaced, rom's choice), plus 10 later fixes applied as they are (7 general, 3 for Codex and GPT accounts). Typecheck passes, 575 tests pass, 1 fails for lack of `curl` in the test image; gitleaks clean; pushed
- [ ] Port by hand, one at a time, the ledger's "by hand" list, and the Codex and GPT fixes rom wants kept that did not apply as they are (`fd8a4609`, `f260eefc`, `8c1d7770`, `8d650348`, `122a2178`, `653cdcac`, `1eeae924` with its test `99ec6a56`, `03296fc6`, `80c09bf2`, `5f96c598`); they need the fork's history, which is in `~/Downloads/ccflare.zip` (remote `keepers` of the clone points at a scratch copy) (since 2026-10-12)
- [x] (2026-10-13) Company names cleaned: their CI file, systemd deploy files and deploy scripts removed, people's names in tests and docs replaced; README written; error reports go to Ciel. Added to nier `watched-repos.txt` (shared): commits need a `Docs:` line
- [x] (2026-10-12, home-server 069b880, aether c93b8a51) Runs on roberta as `stacks/aether`, https://aether.wuling.dev behind the proxy stack, data `/srv/data/aether` (0700), access control on; health answers, the API refuses without a key (checked)
- [x] (2026-10-12, rom) `/srv/data/aether` is not backed up: losing it costs two logins and the usage history
- [x] (2026-10-12) Reachable at home and on the tailnet only: the name points at the LAN address and the tailnet rules cover port 443
- [x] (2026-10-12, rom) Admin key made, both accounts linked from the dashboard (health: 2 accounts, 2 usable)
- [x] (2026-10-12, home-server 4ed3064) First device key `mikaela-rom`, saved by `bin/hs aether-key-save` to rom's `~/.config/aether/key` (never displayed); one real agent request answered through aether and counted on that key. The agent's address is `https://aether.wuling.dev/v1/claude-code` (the bare host answers "model does not exist")
- [x] (2026-10-12, nier 7ebbaf1) rom accepted losing the account's connectors; on rom's request every session goes through aether through the shared nier `settings.json` (`ANTHROPIC_BASE_URL` + `apiKeyHelper` reading `~/.config/aether/key`); the fish snippet tried first was removed (dotfiles df7a3e9). Tested with one real request. No fallback when aether is down
- [ ] work: an aether key at `~/.config/aether/key` (rom runs `bin/hs aether-key-save mikaela-work /tmp/k`, work moves it in); until then work's sessions started from fish use the account directly (2026-10-13 fallback), so nothing breaks (since 2026-10-12)
- [x] (2026-10-13, dotfiles `config/fish/conf.d/claude-aether.fish`) Fallback built: `claude` from a fish shell checks aether's health (2 s); when aether is down, or the user has no key (work today), the session starts with `--settings` pointing at api.anthropic.com and no key helper, so it uses the account login. Tested: a direct session answered through the account. A session already running when aether dies waits (retry watchdog)
- [x] (2026-10-12, aether `/api/statusline`, nier 94547c0, dotfiles) Both bars show aether's view: the Claude Code status line (5h and 7d over the accounts in rotation, accounts serving, this session's cost) and the desktop bar icon and popup (cost today, usage left, per account). Through aether Claude Code gets no limits of its own, so they come from aether; the cache-keepalive guard now reads the pool average too
- [ ] Later (rom, 2026-10-12: aether works, other priorities first): port the company fork's latest usage and cost work, rewritten on our base since it builds on the left-out rules code: direct vs proxy usage (`898225eb` and 3 fixes), weekly per-model cap kept per request (`30747384`), token counts without cache reads (`992ca11b`), the meter model (`244feb7b`, `172055b4`, `c88c1be6`, `76e9986d`, `8fa68515`, `dc56439b`, `99dcb981`, `99e47c6d`, `498721a3`, head merge `071e0ed8`), Codex token counts (`f260eefc`); then the bars show the meter model's dollars (since 2026-10-12)
- [x] (2026-10-12, rom) Routing stays `session`
- [ ] Name (decided 2026-10-12): visible name to aether now (dashboard, terminal texts, README, docs); internals (package names, setting names, key prefix accepting both) after the hand ports (since 2026-10-12)
- Company fork last commit: 2026-10-12 00:19 (still about 50 commits a day); our base is its state of 2026-09-11.
- [ ] rom and work see each other's usage through one proxy: accept, or hide per owner as Ciel does (since 2026-10-12)

## Log

- 2026-10-13: the fork's later dashboard work (provider logos, account order and "closed", time
  zone) is built on its i18n layer, absent from our base: every string would need rewriting, so it
  is left until rom wants those pages; not urgent for one user.

- 2026-10-13 01:10 to 04:50: verified live: this session hit the 5 h limit of the serving account,
  waited (retry watchdog on aether's 429 with reset time) and continued by itself after the reset,
  without ending its turn.

- 2026-10-13 night (aether, home-server): ported from the fork: client gone aborts the provider
  call, the pin wait and the stream (proxy part of efae4abe); error reports in Sentry format sent
  to Ciel (app `aether`, `bin/hs aether-dsn`; a test event arrived). Skipped as not worth it for
  one user: the large-body chain (f90ed0b2, 2eaf4f62, f51ba92d, 87938595, 210e8ed7, then the
  cache-ttl splice fixes of 10-02: memory protection under team load), 1eb4e081 (type hygiene),
  7d798edd (529 wording). Still open: direct vs proxy usage, statusline totals, reset notice,
  per-model weekly cap (usage and cost work, deferred by rom).

- 2026-10-13 (aether, nier): a session hit the limit of both accounts and stopped for good:
  aether answered a 503, which Claude Code reads as a server failure. Now aether answers 429 with
  `anthropic-ratelimit-unified-reset` (earliest reset) and nier sets `CLAUDE_CODE_RETRY_WATCHDOG=1`,
  Claude Code's persistent retry mode, which waits until that reset (at most 6 h) and continues.
  Checked against a fake server: the session keeps retrying instead of ending its turn.

- 2026-10-12: decided by rom: ccflare becomes aether; built from a new fork of the public
  upstream, keeping from the company fork only what is relevant here (the company agrees to the
  reuse). Pooling the two accounts is accepted by rom as the only user.
