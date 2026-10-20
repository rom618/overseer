---
name: aether
summary: Aether, the own proxy in front of the AI accounts (fork of ccflare); live at aether.wuling.dev since 2026-10-12
status: active
owner: shared
host: roberta
repos: ~/Playground/aether (rom618/aether)
folder: ~/claude/issues/ciel
verified: 2026-10-17
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
- Courier (2026-10-15, aether f1e91fec): the fork's "La Poste" ported and renamed on rom's
  request: Claude Code sessions on different machines write to each other through aether. A
  device is a client key (`mikaela-rom`, `kuro-rom`); each machine runs the `courier` binary
  (zero-tool MCP companion, per-machine daemon, CLI) and the user unit `courier.service`; the
  nier skill `courier` tells sessions how to use it. rom's choice: between rom's own machines
  mail goes through without a popup (rules `*-rom -> *-rom` and `rom-admin -> *-rom` allow);
  anything else waits for a yes on the receiving machine or on the dashboard page Courier.
  kuro is receive-only (rom, 2026-10-17): anything may be sent to it, nothing reads from or runs
  on it. How it works and what was left out: aether `docs/courier.md`.
- Endpoint per user and per job (2026-10-18): the shared nier `settings.json` sets aether's URL
  and the key helper (`tools/aether-key.sh`; its key goes out as `x-api-key` and
  `Authorization: Bearer`). Per user the default is `~/.config/aether/key` (mode 600, in
  `user-sops.py`), with an optional `url` beside it for another proxy. Inside a job folder the
  job's `<job>/.aether/key` and optional `url` replace it (machines/mikaela.md, Job folders). The
  fish `claude` function picks them from the current folder, starts the session with
  `--settings` carrying the URL, exports `AETHER_KEY_FILE` for the key helper and checks that
  host's `/health`; no key means the account directly, never another key. Never put a URL or
  token for one user into the shared `settings.json`: it applies to every user. Only fish
  launches see this (every desktop launch and session restore go through fish); a `claude`
  started from another shell uses aether's URL and the default key.

## Open

- [x] (2026-10-17, aether 62ebe2d0) Courier live: deployed (with receive-only machines), installed for rom on mikaela and on kuro (`--receive-only`, over ssh before the cut), test mikaela to kuro shown on kuro's screen, no conversation opened
- [ ] rom: Courier rules on the dashboard page Courier, Rules, Edit (the agent's direct database write was refused by the auto-mode classifier): `[{"label":"rom's machines","fromPattern":"*-rom","toPattern":"*-rom","decision":"allow"},{"label":"rom's dashboard","fromPattern":"rom-admin","toPattern":"*-rom","decision":"allow"}]`; until then every message asks the recipient (kuro shows a popup, mikaela has none and waits for the dashboard); then a test kuro to mikaela (since 2026-10-17)
- [x] (2026-10-17, rom: anything may be sent to kuro, nothing reads from or runs on it; aether 62ebe2d0) Courier guard for kuro: kuro is receive-only (`inbound: info` in its `~/.claude/courier/config.json`, set on kuro so no rule elsewhere undoes it): mail is shown, never opens a conversation, reaches a session as information only, and kuro publishes no session titles, folders or descriptions
- [ ] rom: if you started a Claude session on mikaela between 13:51 and 14:15 on 2026-10-18, it went to a company proxy (Log); to be safe, replace the `mikaela-rom` key (`bin/hs aether-key-save mikaela-rom`) (since 2026-10-18)
- [ ] Bug (2026-10-20, rom): the limit bars lie when no account can serve. `summarizePool` (`/api/statusline`) then averages every account, a spent account included: on 2026-10-17 18:15 to 21:55 UTC "Rom" was 5h 100% 7d 84% and "Rom Ciel" 7d 100%, and the bars showed 5h 50% 7d 92% while aether refused every request (its 429 is not logged in `requests`). The same average feeds the desktop bar and the cache-keepalive guard. Proposed (waits for rom's yes): with an account serving, show the account serving this session (its 5h and 7d, its name, live/total), never an average; with none, show "blocked until HH:MM" in red, the earliest time an account has all its windows under 100% (for one account, the later of its full windows' resets). The other state seen, 5h 0% 7d 100%, is Rom Ciel alone: most likely a session that started straight on the account (fish fallback when aether's health check passes 5 s), whose status line shows Claude Code's own numbers for its login account; not proven, no transcript kept (since 2026-10-20)
- [ ] Courier for work on mikaela: `bin/courier-install` as work once work has a default aether key (`~/.config/aether/key`; today work's only aether key is a job's) (since 2026-10-15)
- [x] (2026-10-20, aether courier fix, dotfiles Hyprland rule) Courier on mikaela: kuro's 2 messages got no popup (the systemd-started daemon had no display in its environment; it now reads it from the user manager), then failed after rom accepted (new conversations opened in `~`, which `claude --bg` refuses as untrusted; now `~/claude/issues/_unfiled`). Accepting now also opens a kitty window attached to the new session; the question floats centered and pinned. kdialog was installed all along (docs/courier.md)
- [ ] rom: ask kuro to resend its 2 messages of 2026-10-20 ~05:18 (both failed with "could not open a conversation"); check the popup is centered and a kitty window opens on accept (since 2026-10-20)
- [x] (2026-10-17, aether 62ebe2d0) `.env.example` renamed `example.env` (no references to fix)
- [x] (2026-10-13, aether 0a076ff9, home-server 4fbbcd0) Key policies: allowed model families and 5h/7d pause thresholds per client key (`bin/hs aether-key-policy`), the base for the runners
- [ ] Bug (2026-10-13): a plain Messages API call (own system prompt, no Claude Code identity) on the subscription route `/v1/claude-code/...` is refused by Anthropic and aether takes the refusal for a quota limit: it benched "Rom Ciel" until its 5h reset (rate_limited_until 22:50 UTC, status rejected) after one test. Fix: record the refusal's shape and do not bench for it (reproduce only on a spare account, never on the two in use); until then no app may call the subscription route (since 2026-10-13)
- [x] (2026-10-13, rom, through the pantry-cook session) rom's apps call the AI by running genuine Claude Code (`claude -p --model haiku|sonnet`) through aether with their own client key (`app-<name>`), on the subscription accounts; no pay-per-use account for now
- [x] (2026-10-13) Ciel issues 14 and 15 (aether noise) resolved; no event since the fixes
- [x] (2026-10-12) Public upstream cloned to `~/Playground/aether` and compared with the company fork (Facts)
- [x] (2026-10-12) Repo rom618/aether created by rom, public version pushed to `main`
- [x] (2026-10-12, rom) Access: behind the proxy stack, reachable at home and through Tailscale only; usage per device wanted (comes with the base: keys per machine)
- [x] (2026-10-12, aether eeca4e5a and 10 picks) Base accepted by rom and landed on `main` as one commit (the flagged example key replaced, rom's choice), plus 10 later fixes applied as they are (7 general, 3 for Codex and GPT accounts). Typecheck passes, 575 tests pass, 1 fails for lack of `curl` in the test image; gitleaks clean; pushed
- [x] (2026-10-13, aether 6b5cb046 to 0b9603a1, all deployed) Fork hand ports done: `4048a388` (client gone logged at info), `7d798edd` (stream usage mirror batched, the fork's OOM fix; clear 529), `97428b2a` (1h cache on every subscription request), `1eb4e081`, `3bfd5ed3` with `b4ffd023` and `f0600801` (413 so Claude Code compacts instead of stripping images), and the Codex/GPT list on rom's request (10 commits: published CLI version, also for the Anthropic usage poll, which now sends `cedar_ember` and the published claude-cli User-Agent; Codex token counting; stored resets listed and spendable from the Accounts page; one header contract, so x-forwarded-*, x-ccflare-* and `access_token` are no longer forwarded; ChatGPT-Account-Id; idle Codex windows polled; compat routes `/v1/ccflare/anthropic/messages` and `/v1/ccflare/openai/responses` removed, no client found). Not applicable to our base (no unparsed large-body path, refusals forwarded as they are): `210e8ed7`, `2b17921c`, `87938595`, `62c8ba58`, `cf70261e`; `21eaf186` was already in 1261306c. Left out: web-chat, user-role and French-UI parts. Codex parts untested live until a Codex account is linked. The fork's history is the remote-tracking ref `keepers/main` of `~/Playground/aether` (not pushed anywhere)
- [x] (2026-10-13) Company names cleaned: their CI file, systemd deploy files and deploy scripts removed, people's names in tests and docs replaced; README written; error reports go to Ciel. Added to nier `watched-repos.txt` (shared): commits need a `Docs:` line
- [x] (2026-10-12, home-server 069b880, aether c93b8a51) Runs on roberta as `stacks/aether`, https://aether.wuling.dev behind the proxy stack, data `/srv/data/aether` (0700), access control on; health answers, the API refuses without a key (checked)
- [x] (2026-10-12, rom) `/srv/data/aether` is not backed up: losing it costs two logins and the usage history
- [x] (2026-10-12) Reachable at home and on the tailnet only: the name points at the LAN address and the tailnet rules cover port 443
- [x] (2026-10-12, rom) Admin key made, both accounts linked from the dashboard (health: 2 accounts, 2 usable)
- [x] (2026-10-12, home-server 4ed3064) First device key `mikaela-rom`, saved by `bin/hs aether-key-save` to rom's `~/.config/aether/key` (never displayed); one real agent request answered through aether and counted on that key. The agent's address is `https://aether.wuling.dev/v1/claude-code` (the bare host answers "model does not exist")
- [x] (2026-10-12, nier 7ebbaf1) rom accepted losing the account's connectors; on rom's request every session goes through aether through the shared nier `settings.json` (`ANTHROPIC_BASE_URL` + `apiKeyHelper` reading `~/.config/aether/key`); the fish snippet tried first was removed (dotfiles df7a3e9). Tested with one real request. No fallback when aether is down
- [x] (2026-10-18, dotfiles 7d8d82f, nier) Per-user and per-job endpoints (Facts), tested with a real request from a job folder
- [x] (2026-10-18) An aether key for work exists, kept in the job folder that uses it (that job's docs); work has no default key, so work sessions outside a job use the account directly
- [x] (2026-10-13, dotfiles `config/fish/conf.d/claude-aether.fish`) Fallback built: `claude` from a fish shell checks aether's health (2 s, 5 s since 2026-10-18: aether answers in about 1.7 s, other proxies can take about 3 s); when aether is down, or the user has no key (work today), the session starts with `--settings` pointing at api.anthropic.com and no key helper, so it uses the account login. Tested: a direct session answered through the account. A session already running when aether dies waits (retry watchdog)
- [x] (2026-10-12, aether `/api/statusline`, nier 94547c0, dotfiles) Both bars show aether's view: the Claude Code status line (5h and 7d over the accounts in rotation, accounts serving, this session's cost) and the desktop bar icon and popup (cost today, usage left, per account). Through aether Claude Code gets no limits of its own, so they come from aether; the cache-keepalive guard now reads the pool average too
- [ ] Later (rom, 2026-10-12: aether works, other priorities first): port the company fork's latest usage and cost work, rewritten on our base since it builds on the left-out rules code: direct vs proxy usage (`898225eb` and 3 fixes), weekly per-model cap kept per request (`30747384`), token counts without cache reads (`992ca11b`), the meter model (`244feb7b`, `172055b4`, `c88c1be6`, `76e9986d`, `8fa68515`, `dc56439b`, `99dcb981`, `99e47c6d`, `498721a3`, head merge `071e0ed8`), Codex token counts (`f260eefc`); then the bars show the meter model's dollars (since 2026-10-12)
- [x] (2026-10-12, rom) Routing stays `session`; a new session starts on the account whose weekly window resets first (rom 2026-10-13, aether 8647d5bc), an active session is not moved (its prompt cache)
- [ ] Name (decided 2026-10-12): visible name to aether now (dashboard, terminal texts, README, docs); internals (package names, setting names, key prefix accepting both) after the hand ports (since 2026-10-12)
- Company fork last commit: 2026-10-12 00:19 (still about 50 commits a day); our base is its state of 2026-09-11.
- [ ] rom and work see each other's usage through one proxy: accept, or hide per owner as Ciel does (since 2026-10-12)
- [x] (2026-10-14, rom: "forget it") Dropped: keyless dashboard for the iPhone. Wanted tailnet only, not the home Wi-Fi; aether cannot tell them apart by source address (subnet route SNAT, Caddy in Docker), so it would need a tailnet-only entry (`tailscale serve` to a second aether port on roberta's 127.0.0.1). Nothing built or deployed; the dashboard still needs the admin key, which the browser keeps after the first paste

## Log

- 2026-10-18 (work session): a hand edit at 13:51 put a company proxy's URL and work's token for
  it into the shared nier `settings.json` (uncommitted), which sent every user's new session
  there, rom's included, until about 14:15 (with work's token, which takes precedence over the
  key helper; rom's aether key should not have been sent, see Open). Fix: per-user, then per-job
  endpoints (Facts), the token moved into the job folder that uses it, `settings.json` back to its
  committed state (the token never reached git). Details in that job's docs.

- 2026-10-17: Courier live (aether 62ebe2d0, receive-only machines); mikaela and kuro
  installed, kuro receive-only; first message mikaela to kuro shown there. Rules left to rom.
  Delivery into an open session fixed (aether 0a39818e, 180f44a0): a resumed or fresh session
  records no mode, the relay now takes the settings' `defaultMode` (`auto` through nier; a relay
  in another mode is refused), and a delivery gives up after 3 failures (each retry was a haiku
  call). The privacy list reached kuro's session `privacy-ea`.
- 2026-10-17 (brief from a kuro session, done from mikaela): Courier still not deployed; rom is
  closing ssh into kuro, which makes Courier the only way into kuro; guard options listed in Open.

- 2026-10-15: Courier ported from the fork's La Poste (aether f1e91fec, pushed, not yet
  deployed); 703 tests, typecheck and lint pass. Not live-tested: delivery through `claude -p`
  and `SendMessage` was verified by the company on Claude Code 2.1.269, here 2.1.301.

- 2026-10-14 (rom, aether 0613c582): a conversation stays on the account holding its warm
  cache (1 h) until that account is rate limited.

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
