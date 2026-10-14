# Change flow: from an edit to production

Owners: rules `AGENTS.md:92-104` (commit and push, stage own paths); hooks
`/opt/nier/README.md:23-27`, `overseer/topics/nier.md:33-49`; deploy commands in each repo
README.

## The common path

```
 1 edit (Edit tool; guard.sh checks the target)
 2 git add <own paths>        (commit-precise.sh; never git add -A: other sessions' work)
 3 git commit
     pre-commit : gitleaks; docs repos: gen-index.sh --check
     commit-msg : needs "Docs: <page>" or "Docs: none (reason)"
     post-commit: ciel-commit.sh in the background -> ciel-agent change -> Ciel journal
 4 git push (GitHub)            allowed without prompt; force push asks
 5 Stop hook stop-docs-check.sh: edited files still uncommitted or unpushed -> session sent back once
 6 deploy (depends on the repo, below)
 7 nightly: ciel-docs-sync.sh writes the journal into overseer / 9s
```

Sources: `nier/bootstrap/githooks/post-commit:1-3`, `nier/tools/ciel-commit.sh:1-31`,
`nier/bootstrap/githooks/commit-msg:8-9`, `nier/settings.json:20-27, 76-77, 117-133`.

Hooks are wired by `link.sh` through `core.hooksPath`. Checked 2026-10-13: overseer, 9s, ciel,
aether, home-server, camille point at `/opt/nier/bootstrap/githooks`; nier and dotfiles at their
own `bootstrap/githooks` (dotfiles has its own copy with the same post-commit). mc-modpack has
no hooks.

## Per repo: where a change goes live

| Repo | Live when | Deploy command | Recorded in Ciel journal as | Docs page |
|---|---|---|---|---|
| `/opt/nier` | immediately for both users (symlinks); `/etc/claude-code` copies need sudo | none; `sudo bootstrap/setup-machine.sh` for managed settings | git | `overseer/topics/nier.md` |
| `/opt/dotfiles` | immediately (symlinks); `hyprctl reload` or relogin; `system/` needs sudo | `sudo bootstrap/setup-machine.sh` for `system/` and the Ciel agent unit | git, then systemd/packages entries | `topics/dotfiles.md`, `machines/mikaela.md` |
| `/opt/overseer`, `9s` | on push (agents read the local checkout) | none | git | `topics/docs.md` |
| home-server | `bin/hs push` | git push into `/srv/home-server`, then `sudo host/setup.sh` | git, then `deploy` from `setup.sh:239`, then docker events | home-server README, `machines/roberta.md` |
| ciel | `bin/deploy` | build on laptop, push via ssh tunnel to registry, `docker compose up -d hub`; then `bin/hs push` for roberta's agent binary | git; docker event "ciel recreated" | Ciel README, `topics/ciel.md` |
| aether | `bin/deploy` | same pattern, `docker compose up -d aether` | git; docker event | aether README, `topics/aether.md` |
| camille | `bin/deploy --roberta` | same pattern, revision baked into the image | git; docker event | camille README |
| mc-modpack | `scripts/deploy.sh --force` (rom only, on his go) | rsync to `/srv/staging/minecraft`, `mc-update.sh` | nothing (not watched) | mc-modpack README |

Sources: home-server `README.md:37-52`, `bin/hs:45-47`; `ciel/bin/deploy`; `aether/bin/deploy`;
`camille/bin/deploy:11-27`; `mc-modpack/scripts/deploy.sh:1-34`.

## Two kinds of server change, kept apart

- **Definition** (compose files, host units, firewall, Caddyfile, names): only through the
  home-server repo and `bin/hs push`. The push refuses when files were edited on the server
  (README:44-47). A Caddyfile change also needs `caddy reload` (README:142-144); a new name needs
  a line in `hosts.txt` (DNS record) and a block in the Caddyfile.
- **App image** (ciel, aether, camille): `bin/deploy` from the app repo. It changes no file on
  the server, only the image behind `127.0.0.1:5000/<app>:latest`.
- **Not a deploy**: Ciel's Stacks page starts, stops and restarts but never pulls or recreates
  (Ciel README:166-171). `.env` edits on the server need `docker compose up -d` in the stack
  folder, not `restart` (README:54-58).

## What a deploy does not guarantee

- Ciel and aether `bin/deploy` build from the working tree and tag only `latest`: an uncommitted
  change can go live, and nothing records which commit runs (Camille marks `-dirty`, the other
  two do not). Compare `camille/bin/deploy:16-17` with `ciel/bin/deploy:8-9`.
- No deploy step runs tests: `bin/check` (aether), `cargo test` (ciel) are run by hand
  (aether README:12-13, 27-29; Ciel README:294-298).
- No rollback command: each deploy overwrites the one tag `latest` in roberta's registry, and
  no older tag is kept. Rolling back means checking out the old commit and deploying again.
- After a Ciel image, roberta's agent binary changes only at the next `bin/hs push`; the
  self-update covers the binary, not the unit (Ciel README:107-111).

## The machine-wide steps that need rom

Some changes cannot finish from an agent: anything with sudo. The agent writes a script into the
issue folder's `pending/` and a TODO item; rom runs it in a real terminal (`AGENTS.md:63-80`).
Examples (both run by rom on 2026-10-13): `sudo setup-machine.sh` for the laptop agent's unit,
the Wi-Fi secrets re-run. Tailnet policy
changes are pasted by hand into the Tailscale console (home-server README:181-184).

## A change in one repo that needs another

| If you change | Also change |
|---|---|
| a new web app | home-server `stacks/<app>`, `stacks/proxy/hosts.txt`, `Caddyfile`, portal card (`README:9`), a Ciel web check |
| an app's error reporting | a Ciel app (`ciel-hub app add`), a DSN in the stack's `.env` (`hs <app>-dsn`) |
| a new device or user | nier machine page (`nier register`), Ciel device token, aether key, tailnet node + policy, restic setup (`topics/setup-steps.md`) |
| a private path | `guard.sh`, `managed-settings.json` (then sudo copy), `CIEL_REDACT` (dotfiles `setup-machine.sh:100`, home-server `setup.sh:207`), `backup/never.txt` |
| a watched repo | `/opt/nier/bootstrap/watched-repos.txt` and `link.sh` |
| the Ciel agent protocol | ciel image deploy, `bin/hs push` on roberta, `sudo setup-machine.sh` on the laptop for the unit |
