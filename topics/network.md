---
name: network
summary: mikaela networking: Wi-Fi profiles, Flybox CGNAT, Surfshark issues, Docker MTU, avahi, Tailscale
status: active
owner: shared
host: mikaela
repos: /opt/dotfiles
folder: ~/claude/issues/ciel
verified: 2026-10-10
source: nmcli, /etc/avahi/avahi-daemon.conf, tailscale status, history.md 2026-09-13/14/17/26, 2026-10-10, overseer/TODO.md
---

# Network

Wi-Fi only (`wlo1`), NetworkManager. Hostname `mikaela` (machine facts: `overseer/machines/mikaela.md`).
Surfshark autostarts for rom and routes everything through `surfshark_wg`. Server-side networking
is on `machines/roberta.md`.

## Facts

- Wi-Fi profiles: Flybox-319A, Freebox-753BAF, iPhone (hotspot), TheKeepers. `ipv6.method` is `disabled`
  on all but Flybox-319A (`auto`). A new network that breaks Surfshark gets
  `nmcli connection modify "<name>" ipv6.method disabled`. Wi-Fi secrets are backed up by
  `sudo backup/system-secrets.sh` (refreshed 2026-10-13).
- Flybox-319A is Orange's 4G box (not Bouygues), behind CGNAT (hop 192.0.0.1, shared public IPv4):
  IPv4 port forwarding can never work there. It gives the laptop a public IPv6 /64 and outbound IPv6
  works; inbound is dropped by the box's IPv6 firewall until a rule is opened in its web UI
  (http://192.168.1.1).
- Surfshark 3.13+ adds a mandatory "IPv6 leak protection" step that creates a `surfshark_ipv6` dummy
  interface. It failed with "Operation not supported" when the running kernel's modules were gone from
  `/usr/lib/modules/` (updated kernel, no reboot): the `dummy` module could not load. The same stale
  kernel broke Docker. Fix is a reboot after kernel updates. The earlier IPv6 `disabled` setting is
  harmless but was not the root cause.
- DNS race at login: Wi-Fi connects and sets router DNS at about T+10 s, Surfshark activates 8-10 s
  later and takes over DNS, so lookups in that window can fail (one `ERR_NAME_NOT_RESOLVED` in
  Firefox, a blank weather widget before its retry fix). Not fixable on our side.
- Docker under Surfshark: `surfshark_wg` has MTU 1280, bridge networks default to 1500, so full-size packets
  drop (TLS handshakes, downloads time out while DNS and TCP connect work). Fix per project: compose
  `networks.default.driver_opts."com.docker.network.driver.mtu": "1280"` and `build.network: host`
  (or `docker build --network host`).
- avahi: `camille.local` is retired (apps are named under wuling.dev); avahi stops announcing it
  once `setup-machine.sh` has run (machines/mikaela.md).
- Tailscale: the system `tailscaled` is on rom's own tailnet; work's vesscam tailnet runs on work's
  own userspace `tailscaled-work` (since 2026-10-11). Current state: machines/mikaela.md.
- Old WireGuard profile `wg0` (a link into another network, peer never answered) was deleted on
  2026-10-10 with the leftovers of the old game-server setup. No other VPN profile besides `surfshark_wg`.
- The laptop no longer runs the media app or its Caddy (moved to roberta 2026-10-10, containers
  removed 2026-10-11).

## Open

- [ ] rom's list: on the iPhone only one VPN runs at a time, so Surfshark and the tailnet cannot both be on as on the laptop; options to compare: an exit node on the tailnet that goes out through Surfshark (roberta or a Pi), or Surfshark only on untrusted Wi-Fi (since 2026-10-13)
- [ ] rom's list: research how others give a home server behind CGNAT a public address under its own domain (Minecraft and similar), e.g. the Reddit threads on Cloudflare Tunnel for Minecraft, a cheap VPS relay, playit with a custom domain; rom has a thread in mind (link to ask) (since 2026-10-13)
- [ ] rom's list: a local network between rom's devices that keeps working with no internet (Flybox down): what still works today, and what it takes (local DNS, the GitHub replica, aether cannot) (since 2026-10-13)
- [x] (2026-10-12) Own tailnet for remote access to the self-hosted apps: hosted Tailscale, roberta subnet router, iPhone and mikaela joined; vesscam stays on work's own tailscaled
- [ ] vesscam Headscale tailnet (Forgejo): operator is work, shields-up on (2026-10-11). Moved to tailscaled-work 2026-10-11 (machines/mikaela.md). DNS: the server pushes no upstream resolvers (`tailscale dns status`, 2026-10-11), so 100.100.100.100 answers `*.labo.internal` and forwards the rest to the system resolver; vesscam sees no lookups.; re-check DNS if they add resolvers (since 2026-10-10)
- [x] (2026-10-10) `camille.local` retired; apps are named under wuling.dev
- [x] (2026-10-12, home-server 6161d05, 55d66f9) Own tailnet exists (hosted Tailscale, rom's personal Google account `romainteyssier1@`): roberta 100.75.27.34 joined, route 192.168.1.0/24 approved, key expiry off; iPhone `iphone-14` 100.73.199.29 joined. Verified from roberta (`tailscale status`). Key expiry off on every node (roberta, iPhone, mikaela; checked 2026-10-13), and on each new one when it joins (rom)
- [x] (2026-10-12, rom) From the iPhone on mobile data, https://camille.wuling.dev and qBittorrent at http://192.168.1.53:8090 both open
- [x] (2026-10-12, rom ran pending/mikaela-tailscale-join.sh; dotfiles b9c4cdb) mikaela on the own tailnet: 100.67.32.103, home route switched by the NetworkManager hook, direct connection to roberta with Surfshark on, DNS untouched (`machines/mikaela.md`)
- [x] (2026-10-12, rom) Laptop away test: on the iPhone hotspot with Surfshark on, the hook turned the home route on by itself (journal `tailnet-home-route`) and https://camille.wuling.dev opened.; back on the home Wi-Fi the hook turned it off again and roberta is reached directly over the LAN (checked 2026-10-12)
- [x] (2026-10-12, rom ran pending/mikaela-tailscale-old-state-remove.sh) Old vesscam node state removed from mikaela
- [ ] Other devices (the desktop, further servers) join the own tailnet later, registered from Ciel once Ciel and aether are done (rom, 2026-10-12); nothing to do before (since 2026-10-12)
- [x] (2026-10-12, home-server ba0132c, rom pasted it in the console) Tailnet access rules active: phone and laptop reach roberta on 443 and 8090, the laptop also on 22, roberta starts nothing, the rest of the home network is closed. Checked from mikaela over the tailnet address: 443, 8090, 22 open; 25565 and 9100 closed; roberta to mikaela 22 closed. File of record and how to change it: home-server README "Remote access". Mephisto and aether get subdomains later (covered by 443)
- [x] (2026-10-12, dropped by rom) Phone recheck under the new rules not done: the path through the home route (192.168.1.53 from the phone) is covered by the policy's own `tests` block only
- [x] (2026-10-12) mikaela on the own tailnet: node `mikaela` 100.67.32.103, old vesscam node state removed, direct to roberta with Surfshark on; home route only away from home (NetworkManager hook, machines/mikaela.md)
- [ ] Own tailnet, how the wuling.dev names are reached (decided 2026-10-12): roberta announces the home subnet to the tailnet (subnet router), the names keep their LAN address 192.168.1.53, so devices without Tailscale still work at home. Devices: iPhone, mikaela, the desktop, the servers. To test: Surfshark together with Tailscale on mikaela; on the iPhone only one VPN is active at a time (since 2026-10-12)
- [ ] Public access: Minecraft stays on playit (Cloudflare Tunnel carries web traffic only; a game client cannot use it without Cloudflare's client on every player's machine). Cloudflare Tunnel is a candidate for public web pages only (map, modpack portal); not decided (since 2026-10-12)
- [ ] Device registration and per-device access through Ciel (Tailscale API: one-time keys, device list, tags in the tailnet policy); review what each device type may reach; Ciel milestones 8 and 9 (since 2026-10-12)
- [ ] Pi-hole (since 2026-10-10)
- [x] (2026-10-11) Laptop Caddy gone with the laptop Camille stack
- [x] (2026-10-13, checked) Surfshark up and working (`surfshark_wg` active all day, direct path to roberta)
- [x] (2026-10-12) From rom's list: ssh to roberta from outside the home network works through the own tailnet (mikaela reaches roberta on 22 by the access rules)
- [ ] rom's list: what does a Cloudflare Tunnel give that the tailnet does not? Answer given 2026-10-12: public web pages for people outside the tailnet (no app to install); not games. Decide whether any page should be public (since 2026-10-12)
- [ ] rom's list: refuse cookies network-wide: not possible at the DNS or router level (banners are page scripts, Pi-hole can only block domains). Per browser: Firefox (laptop) can reject banners itself (`cookiebanners.service.mode = 1`) or with the Consent-O-Matic extension, both installable for both users through `dotfiles/system/etc/firefox/policies/policies.json`; iPhone Safari needs an extension. rom: pick one per device, later (2026-10-13) (since 2026-10-12)
- [ ] rom's list: Pi-hole becomes the home router's DNS (and possibly the own router later), with its own name under wuling.dev (since 2026-10-12)
- [ ] rom's list: link the email senders Sweego and Resend to the domain wuling.dev (the DNS records it asks for: SPF, DKIM, return path) (since 2026-10-12)
- [ ] rom's list: reduce captchas one way or another (Cloudflare checkboxes, DataDome on Leboncoin, Google): they come from IP reputation and browser signals; Surfshark exits trigger them, so options are a cleaner exit (no VPN at home, a residential exit), Privacy Pass tokens, or fewer anti-fingerprint settings (since 2026-10-12)

## Log

- 2026-09-13: Surfshark failures: first blamed IPv6 on the active profile (disabled); then the real cause,
  the 3.13 dummy-interface step plus stale kernel modules. Docker had the same stale-kernel cause.
- 2026-09-14: Docker "no internet" root cause is MTU 1280 vs 1500 under Surfshark, not an outage.
  `ipv6.method disabled` applied to all Wi-Fi profiles then in use.
- 2026-09-17: avahi restricted to `wlo1`; `nss-mdns` added so the laptop resolves `camille.local`.
- 2026-09-26: Flybox identified as Orange 4G behind CGNAT; its profile set back to `auto` at the
  user's request (if Surfshark breaks on it, that is why). `wg0` noted as no inbound path.
- 2026-10-10: laptop renamed `mikaela`, avahi keeps `camille.local`; `wg0` deleted.
- 2026-10-10: naming and HTTPS decided: an own domain (to buy), names like `<app>.<domain>` used
  inside the LAN and the tailnet only, real certificates from Let's Encrypt through a DNS challenge
  (no port opened, works behind the CGNAT), served by a reverse proxy on the server.
- 2026-10-10: gotcha: a new `<name>.wuling.dev` queried before its DNS record exists stays
  "not found" for up to 30 minutes (Cloudflare negative TTL 1800 s) in the laptop's resolver, which is
  Tailscale's 100.100.100.100 while the vesscam tailnet is up. Create the record before the first query.
