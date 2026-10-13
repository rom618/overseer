# Findings of the system audit (2026-10-13)

Severity: HIGH = an outage or loss would go unnoticed or be unrecoverable; MEDIUM = a wrong fact
that misleads; LOW = tidiness. Paths: `ov/` = `/opt/overseer/`, `hs/` =
`~/Desktop/perso/projects/home-server/`.

## Single points of failure and risks

1. HIGH: if roberta goes down, nobody is alerted. The hub, roberta's agent and every web check run
   on roberta. The laptop agent pages ntfy only while it is on the hub's /24
   (`ciel/README.md:153-156`). (Correction 2026-10-13: an ntfy topic was already set up; the
   Open line saying otherwise was stale.) roberta has
   no UPS and is on Wi-Fi only (`ov/machines/roberta.md:15-16`). The only signal is the laptop
   bar's crossed cloud (gap acknowledged in `ciel/SPEC.md:150-151`).
2. HIGH: all AI work depends on roberta and its 4G uplink (`nier/settings.json:156-160`). If the
   home internet is down the fallback fails too (the laptop uses the same Flybox at home).
   Sessions started outside fish (bash, timers, the planned `claude -p` runners,
   `ov/topics/nier.md:94`) have no fallback.
3. HIGH: the recovery chain rests on one password rom remembers: laptop restic password, then the
   laptop backup with the server secrets (`backup/includes.txt:15`), then the server off-site
   copies, then Vaultwarden. The server restore was never rehearsed (`ov/topics/backups.md:71`);
   the secrets-restore step is still Open (`ov/topics/setup-steps.md:41`). No page says where the
   restic password's recovery copy is.
4. HIGH: the laptop copy of roberta's secrets (`~/.config/home-server/roberta-secrets.tgz`, file
   time 2026-10-10 19:35) predates the Ciel agent token, the Ciel and aether DSNs, the Ciel admin
   key and the Tailscale state. No page says what it holds or how to refresh it
   (`ov/topics/backups.md:72`). Not opened.
5. MEDIUM: one personal 15 GB Drive holds every off-site copy and the modpack zips
   (`ov/topics/backups.md:51-54`); it filled once (`:78`).
6. MEDIUM: the Ciel hub has root on both machines through agent self-update and the container
   terminal (`ov/topics/ciel.md:73`, `ciel/README.md:107-111`). Accepted by rom; it makes the
   image, registry and build path the most sensitive path in the system.
7. MEDIUM: one flat docker network `proxy` joins aether (provider tokens in clear text,
   `ov/topics/aether.md:31`), an unauthenticated Prometheus, vaultwarden, camille, ciel, bluemap
   and portal; all reach each other by name (`hs/README.md:145-147, 203-205`).
8. MEDIUM: names need Cloudflare DNS even at home (`hs/stacks/proxy/hosts.txt:7-14`); no local
   resolver (Pi-hole is Open, `ov/topics/network.md:70, 76`). Not tested with the uplink down.
9. MEDIUM: `~/.config/aether/key` and `~/.config/ciel/token` are covered by no rule in
   `nier/managed-settings.json:3-42` nor `tools/guard.sh:25-26`; only the written rule protects
   them.
10. MEDIUM: ciel and aether `bin/deploy` (lines 8-9) build the working tree and tag `latest`: no
    revision recorded (camille does, `camille/bin/deploy:16-17`), no rollback tag, and a deploy
    shows in the journal only as a docker "recreated" event (unlike `hs/host/setup.sh:239`).
11. LOW: the aether fork history lives only in the local aether clone (remote `keepers`); not
    pushed anywhere and `~/Downloads` is not backed up (`ov/topics/backups.md:61-62`).

## Docs that contradict reality or each other

12. MEDIUM: `aether/README.md:24` "No fallback when aether is down": wrong since dotfiles dd14ffa.
13. MEDIUM: `ov/topics/aether.md:3` ("decided, not built yet") and `:7` (`repos: none yet`), shown
    in `ov/INDEX.md:8`; aether is live and has a repo.
14. MEDIUM: Ciel milestone status stale: `ov/topics/ciel.md:3, 19-20` and `ciel/README.md:7-8`
    ("1 to 6 live, next 7"); `ciel/README.md:216` "Not yet: Grafana's retirement";
    `ciel/README.md:226` "Not yet: checks on top".
15. MEDIUM: Minecraft retention: `hs/README.md:70-71` (14/8/6) vs `hs/stacks/minecraft/
    compose.yaml:81` (2d/7/4/3, also `ov/topics/home-server.md:73-74`, `ov/TODO.md:90-91`).
16. MEDIUM: off-site list incomplete in `ov/topics/backups.md:55-57` and
    `ov/machines/roberta.md:48-49` (only Minecraft and Terraria); `hs/scripts/offsite-backup.sh:
    24-42` also copies Vaultwarden, Ciel and the worlds archive; roberta.md omits archive-backup.
17. MEDIUM: `hs/README.md:190-191` says aether backup "decision open"; decided 2026-10-12 (no).
18. MEDIUM: `ov/topics/home-server.md`: `:25-26` Camille 8081 open (closed per `:56`); `:43-44`
    terraria-backup hourly (disabled); `:46` lists Dockge.
19. MEDIUM: `ov/topics/network.md:41-44` (avahi camille.local), `:45-48` (vesscam on the system
    daemon), `:51` (laptop Caddy) superseded.
20. MEDIUM: `ov/machines/mikaela.md:109` Tailscale row opens with the old vesscam state.
21. MEDIUM: `mikaela.md:7` work is in wheel, `ov/TODO.md:17-20, 27` plans a reset "without sudo":
    target undecided.
22. LOW: `hs/README.md:3` "Game servers"; `:132-133` stale Terraria power line; `:141` and
    `hosts.txt:3` "tailnet once it exists"; `:18` 119 GB vs roberta.md 117 GB for `/backup`.
23. LOW: `ov/machines/roberta.md:69` points to a TODO "Servers" section that does not exist; the
    hardening it waits for is done.
24. LOW: `ciel/SPEC.md:103` hub on the monitoring network and `:131-138` remote write (built: the
    `proxy` network and hub-kept samples); `:3-5` decisions in the issue README, against
    `AGENTS.md:29`.
25. LOW: `ov/topics/docs.md:7` gives `~/Playground/overseer`; the repo is `/opt/overseer`.
26. LOW: `nier/README.md:15` mentions the retired inbox and outbox folders.
27. LOW (rom's docs): the Camille hosting page describes the laptop-era compose.

## Code and config against the rules

28. MEDIUM: `mc-modpack/scripts/deploy.sh:4-5` defaults to the raw IP and `id_ed25519_server`,
    replaced on 2026-10-10 (`ov/TODO.md:94-95`); the pending 0.1.11 deploy will hit it. The repo
    is not in `watched-repos.txt`.
29. LOW: `dotfiles/bootstrap/setup-machine.sh:100` hard-codes `/home/rom` in `CIEL_REDACT`.
30. LOW: `nier/settings.json:174-194` generated autoMode text with escaped em dashes and stale
    paths; old `/opt/claude-config` rules (`:12-18, 39, 83`) wait for work's relink.
31. LOW: `nier/skills/agent-review` and `skill-review` are untracked leftovers.
32. LOW: roberta still has the docker networks `dockge_default` and `pregen_default`.

## Missing docs

33. No page owns the system-level picture (this wiki). Candidate: overseer topic `system`, with
    the parts about rom's own apps in rom's docs (done 2026-10-13).
34. No page says how to refresh the laptop copy of roberta's secrets, or where the restic
    password's recovery copy is.
35. The Google, Tailscale, Cloudflare and GitHub accounts are not documented as dependencies with
    a recovery path.
36. Stale `verified` dates on both machine pages and the ciel, docs, nier, home-server, backups,
    network and dotfiles topics; `ov/topics/ciel.md` is 156 lines (should split).
