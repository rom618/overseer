# Devices and network

Owners: `/opt/overseer/machines/mikaela.md`, `/opt/overseer/machines/roberta.md`,
`/opt/overseer/topics/network.md`, home-server `README.md` "Remote access". This page links
them; it owns nothing.

## Devices

| Device | What | Role | Network | Source |
|---|---|---|---|---|
| mikaela | Arch laptop, users rom (1000) and work (1001) | desktop, owner shared | Wi-Fi only (`wlo1`); Surfshark WireGuard for rom; tailnet node 100.67.32.103 | `machines/mikaela.md:1-8, 38-46, 107-109` |
| roberta | MSI laptop, headless Debian 13, user `server` | server, owner shared | Wi-Fi only (Ethernet down), 192.168.1.53, tailnet 100.75.27.34, subnet router | `machines/roberta.md:14-17, 53-56` |
| iPhone | `iphone-14` | phone, not managed by Ciel | tailnet 100.73.199.29; one VPN at a time | `topics/network.md:58, 67` |
| kuro | rom's desktop (Linux Mint) | not registered yet | not on the tailnet | rom's TODO |
| Flybox | Orange 4G box, 192.168.1.1 | the only internet uplink | CGNAT: no port forwarding ever | `topics/network.md:25-28` |

Both computers are laptops without a working battery on the server side (dead battery, no UPS:
`machines/roberta.md:16`). roberta is the only always-on machine.

## How traffic flows

```
 at home                                     away from home
 mikaela --Wi-Fi--> 192.168.1.53:443         mikaela --tailnet--> roberta (subnet router)
   (names resolve via public DNS               --> 192.168.1.53:443 (same names, same IP)
    to the LAN address)                      iPhone (mobile data) --tailnet--> same path
 roberta --4G--> internet                    players --> playit.gg tunnel --> roberta:25565
```

- Names: each line of `home-server/stacks/proxy/hosts.txt` is a DNS-only A record at Cloudflare,
  kept by `scripts/dns-records.sh`; all point at 192.168.1.53 (hosts.txt:6-14). Today:
  wuling.dev (start page), home (redirect), map, vault, camille, ciel, aether, qbit.
- Certificates: Caddy gets a Let's Encrypt wildcard `*.wuling.dev` by DNS challenge with the
  Cloudflare token in `/etc/home-server/cloudflare-api-token` (home-server README:137-139). No
  port is open to the internet; the CGNAT would block it anyway.
- Tailnet: hosted Tailscale (not Headscale, because of the CGNAT: Ciel SPEC section 11). roberta
  announces 192.168.1.0/24, takes no DNS and no routes, Tailscale ssh off (README:164-169).
- Access rules: `home-server/host/tailnet-policy.hujson` is the file of record, pasted by hand into
  the Tailscale console (README:178-184). Phone and laptop reach roberta on 443 and 8090, the
  laptop also on 22; roberta starts nothing; the rest of the LAN is closed.
- Home route on the laptop: the NetworkManager hook `50-tailnet-home-route` refuses the tailnet
  route on the home Wi-Fi and accepts it elsewhere (`/opt/dotfiles/system/README.md:24-29`,
  installed by `bootstrap/setup-machine.sh:66-68`).
- Key expiry is off on every node (`topics/network.md:58`), so nodes do not drop out after
  180 days. Ciel's `network` check fails when a node needs a login (Ciel README:124-128).
- Public game access: playit.gg agent on roberta (host network) tunnels Minecraft and Terraria
  (home-server README:119-126). Web pages are not public at all.

## Two tailnets on one laptop

| Tailnet | Daemon | Who | Purpose | Source |
|---|---|---|---|---|
| own (hosted Tailscale) | system `tailscaled`, operator rom | rom, machine-wide | reach roberta and the apps away from home | `machines/mikaela.md:109` |
| a work job's tailnet (someone else's coordination server) | a user unit from the job folder, userspace, own SOCKS5 port | work only | the job's servers | `machines/mikaela.md` (Job folders), the job's docs |

Rule: never run both daemons on the same node state (`machines/mikaela.md:109`).

## Surfshark and Docker

- Surfshark autostarts for rom and routes all laptop traffic through `surfshark_wg`
  (`topics/network.md:16`). Ciel's network check expects it up on mikaela
  (`topics/ciel.md:59`).
- `surfshark_wg` has MTU 1280; Docker bridges default to 1500, so full-size packets drop. Every
  image build on the laptop uses `--network host` (Ciel, aether, Camille `bin/deploy`), and
  laptop compose stacks set MTU 1280 (`topics/network.md:37-40`).
- roberta's qBittorrent has its own Surfshark tunnel inside the Camille stack (gluetun), not
  related to the laptop's (home-server README:245-246).
- A kernel update without reboot breaks Surfshark and Docker (modules gone):
  reboot (`topics/network.md:29-33`).

## Gotchas that cross machines

- The Flybox resolver answers with a NAT64 address first; clients must try IPv4 first. The Ciel
  agent does (`machines/roberta.md:59-60`).
- A new `<name>.wuling.dev` queried before its record exists is cached as "not found" for up
  to 30 minutes (`topics/network.md:93-95`).
- On a foreign Wi-Fi that also uses 192.168.1.x, the phone prefers that network and the names
  do not answer (home-server README:174-177).
- Name resolution always needs Cloudflare's public DNS, even at home: there is no local resolver
  for wuling.dev yet (Pi-hole is Open, `topics/network.md:70, 76`). Inferred from hosts.txt and
  the absence of local DNS; not tested with the uplink down.

## ssh

- Laptop to server: alias `roberta` = `server@192.168.1.53`, key `~/.ssh/id_ed25519_roberta`,
  passphrase in the login keyring, loaded at login (`machines/mikaela.md:108`). Everything
  that changes roberta (`bin/hs`, every `bin/deploy`) rides on this one key and one user.
- On roberta: keys only, no root login, only `server`, passwordless sudo
  (`machines/roberta.md:26-28`). Anything run as `server` is root.
