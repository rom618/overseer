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
  `sudo backup/system-secrets.sh` (stale since 2026-09-12; see Open).
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
- avahi: `/etc/avahi/avahi-daemon.conf` has `host-name=camille` and `allow-interfaces=wlo1`, so
  `camille.local` still resolves for the app Camille after the 2026-10-10 rename, announced on Wi-Fi only
  (it had announced Docker's 172.18.0.1 first). `camille.local` on the laptop itself needs `nss-mdns`
  with `mdns_minimal` (in `packages.txt`).
- Tailscale: `tailscaled` runs. The machine page says the laptop is logged out of the Headscale
  tailnet `hs.vesscam.com` (Forgejo) and that it should become work-owned
  (`tailscale set --operator=work`). On 2026-10-10 `tailscale status` showed BackendState Running with
  peers on 100.64.0.0/10, which contradicts that; re-check before relying on either.
- Old WireGuard profile `wg0` (a link into another network, peer never answered) was deleted on
  2026-10-10 with the leftovers of the old game-server setup. No other VPN profile besides `surfshark_wg`.
- Caddy for the media app listens on 80/443 on every interface, also on untrusted Wi-Fi (see Open).

## Open

- [ ] Own Tailscale tailnet for remote access to the self-hosted apps (Headscale later needs a VPS). One tailscaled holds one tailnet at a time (`tailscale switch` only swaps), so on mikaela the vesscam tailnet then moves to a second, userspace tailscaled run by work; both users reach the own tailnet, only work reaches vesscam (decided 2026-10-11) (since 2026-10-10)
- [ ] vesscam Headscale tailnet (Forgejo): operator is work, shields-up on (2026-10-11). Moved to tailscaled-work 2026-10-11 (machines/mikaela.md). DNS: the server pushes no upstream resolvers (`tailscale dns status`, 2026-10-11), so 100.100.100.100 answers `*.labo.internal` and forwards the rest to the system resolver; vesscam sees no lookups.; re-check DNS if they add resolvers (since 2026-10-10)
- [x] (2026-10-10) `camille.local` retired; apps are named under wuling.dev
- [ ] Pi-hole (since 2026-10-10)
- [x] (2026-10-11) Laptop Caddy gone with the laptop Camille stack
- [ ] Verify Surfshark after the stale-kernel reboot (since 2026-09-13)

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
