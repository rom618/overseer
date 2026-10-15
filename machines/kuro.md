---
host: kuro
machine_id: 38b9c8fb6d844396b255ed0c6eddba47
os: linuxmint
role: desktop
owner: rom
users: romai (1000)
verified: 2026-10-15
source: nier register (detected); add what detection cannot know
---

# kuro

rom's desktop and gaming PC. Linux Mint 22.3 (Cinnamon), AMD Ryzen 7 5800X, 63 GB RAM, NVIDIA
RTX 3080 (10 GB). Disks: / 912G, /boot 1,7G, /mnt/GAMES 916G, /boot/efi 512M. User `romai` (rom).

## Hard rules on this device

- rom plays games here: never stop a game, never run GPU-heavy work while one runs (the gpu-queue
  decides when, 9s gpu-queue topic).
- Packages: apt (`nier/os/debian.md`); installs and upgrades are rom's.
- Private paths: none declared yet (ask rom before reading personal folders).

## Layout

- Kept as installed (rom, 2026-10-15): no reinstall, Cinnamon stays, no dotfiles (dotfiles is
  mikaela only), no backup for now.
- Joined 2026-10-15 with nier `bootstrap/join.sh` (overseer `topics/setup-steps.md`, kuro plan):
  `/opt/nier` and `/opt/overseer`, machine age key `/etc/age/machine.key`, Ciel agent
  (`ciel-agent.service`, device `kuro`, owner rom).
- Access: the laptop's `ssh kuro` (key `id_ed25519_kuro`, sshd keys only, `/etc/ssh/sshd_config.d/10-nier.conf`);
  ufw is on and opens 22 on `tailscale0` only; the tailnet policy lets only mikaela reach kuro:22
  (home-server `host/tailnet-policy.hujson`). Tailscale address 100.73.251.98, no accept-routes.
- Keys: aether key `kuro-rom` in `~/.config/aether/key`, Ciel read token `kuro-agents` in
  `~/.config/ciel/token`; no recovery copy (a lost kuro gets new ones).
- Music: Feishin (player for Mephisto, 9s mephisto topic) as the AppImage
  `~/.local/opt/feishin/Feishin.AppImage` (installed 2026-10-15 at 1.17.0, updates itself),
  launcher `~/.local/share/applications/feishin.desktop`, icon in `~/.local/share/icons`. Media
  keys reach it through MPRIS (`org.mpris.MediaPlayer2.Feishin`), Cinnamon's default bindings.

## Open

- [x] (2026-10-15, rom) Tailscale key expiry off for kuro; kuro's Firefox signed in with Ciel
- [x] (2026-10-15) Claude Code 2.1.3 in `~/.local/bin/claude` (official installer), a test request answered through aether; Ciel shows kuro online with its age key. `~/.bashrc` adds `~/.local/bin` to PATH (Mint's `~/.profile` only does it when the folder exists at login, so a session started before the install did not find `claude`)
- [x] (2026-10-15) Look-only check (rom's `~/claude/issues/overseer/kuro-survey.sh`, run over ssh): nothing suspicious. Package sources: Mint, Ubuntu, Spotify, the OBS PPA, Prism Launcher, Tailscale; hand-installed apps known (OBS, Wine, QEMU/libvirt, CurseForge in `/opt`, flatpaks such as Discord, Surfshark, Tor Browser, ProtonPlus); autostart: Surfshark, Emote, Firefox; no custom timers or cron; Firefox extensions uBlock Origin and Bitwarden; no Chrome; 0 updates pending; ufw on; Surfshark's WireGuard runs beside Tailscale
- [x] (2026-10-15, rom) The port on 127.0.0.1:8743 is the Ciel agent's own (local only)
- [ ] Optional virus scan, mainly for Windows files run through Wine (and the gacha skin tools later): rom does it with Claude on kuro (`sudo apt install clamav`, then `clamscan -r` on the home folder) (since 2026-10-15)
- [ ] Second monitor: menus in Korean (the monitor's own menu: rom, with its buttons; Mint's: the agent) and it often turns off (Mint power settings: the agent; the monitor's eco or auto-off: rom); rom does it with Claude on kuro (since 2026-10-12)
- [ ] rom: `sudo ~/claude/issues/mephisto-taste/pending/remove-spotify.sh` in a real terminal (purges `spotify-client`, its apt source and keys; Mephisto + Feishin replace it, as on mikaela); the agent already deleted `~/.cache/spotify` and `~/.config/spotify`; then the agent drops Spotify from the package sources above (since 2026-10-15)
- [ ] Later: gpu-queue (9s), gacha skin tools for Wuthering Waves and Endfield (9s TODO) (since 2026-10-12)

## Detected at registration

- Enabled services: accounts-daemon anacron apparmor avahi-daemon blk-availability blueman-mechanism bluetooth casper-md5check casper ciel-agent console-setup cron cups-browsed cups dmesg e2scrub_reap finalrd getty@ gpu-manager grub-common grub-initrd-fallback irqbalance kerneloops keyboard-setup libvirt-guests libvirt
- Tailscale installed
