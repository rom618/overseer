---
host: kuro
machine_id: 38b9c8fb6d844396b255ed0c6eddba47
os: linuxmint
role: desktop
owner: rom
users: romai (1000)
verified: 2026-10-17
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
- Gaming: Steam libraries `~/.steam/steam/steamapps` and `/mnt/GAMES/SteamLibrary`; extra Proton
  builds in `~/.steam/steam/compatibilitytools.d`; monitors DP-0 Samsung Odyssey G5 2560x1440 at
  165 Hz (primary) and HDMI-0 LG UltraGear (Korean model, menus in Korean) 1920x1080 at 144 Hz on
  X11. Wallpapers: one per monitor through HydraPaper (flatpak), files in `~/Pictures/wallpapers`. Cinnamon `org.cinnamon.muffin
  unredirect-fullscreen-windows` set to true (2026-10-15, against in-game micro-stutter).
  Flatpak apps need the `org.freedesktop.Platform.GL.nvidia-<driver>` runtime matching the
  NVIDIA driver (`flatpak update` installs it); without it they fail with an X BadDrawable error.
  Game mod setup: 9s `topics/gacha-skins.md`.
- Pull tracker watcher: user service `pulls-watch.service` (`~/.local/bin/pulls-watch`, state in
  `~/.local/state/pulls-watch/`, since 2026-10-17) reads the WuWa and Endfield history links from
  the game files and posts them to https://pulls.wuling.dev; how it works: the pulls repo README.
- Joined 2026-10-15 with nier `bootstrap/join.sh` (overseer `topics/setup-steps.md`, kuro plan):
  `/opt/nier` and `/opt/overseer`, machine age key `/etc/age/machine.key`, Ciel agent
  (`ciel-agent.service`, device `kuro`, owner rom).
- Access: the laptop's `ssh kuro` (key `id_ed25519_kuro`, sshd keys only, `/etc/ssh/sshd_config.d/10-nier.conf`);
  ufw is on and opens 22 on `tailscale0` only. rom decided 2026-10-17 to close it (privacy): the
  tailnet policy of record no longer lets mikaela reach kuro:22 (home-server
  `host/tailnet-policy.hujson`, not yet pasted in the console), the kuro side waits for rom (Open).
  Note: sshd listens on 0.0.0.0:22 through `ssh.socket` (Ubuntu 24.04 base), ufw is what limits it.
  Tailscale address 100.73.251.98, no accept-routes, shields down, Tailscale SSH off.
- Exposure seen 2026-10-17 (read-only check): avahi (mDNS 5353), cups and cups-browsed (no
  printer configured), bluetooth (powered, pairable, not discoverable), libvirt (`virbr0` DNS/DHCP
  on 192.168.122.1, one VM shut off; libvirt-guests enabled). Disk: `/` and swap on LUKS
  (`nvme0n1p4_crypt`); `/mnt/GAMES` and two unmounted ext4 disks (`sda1`, `sdb1`) are not
  encrypted. LightDM autologin to `romai` (the LUKS passphrase is the only boot prompt);
  Cinnamon locks on screensaver, idle after 15 min, lock delay 0. Firefox: no enterprise policy, no
  telemetry prefs changed (defaults on). Ciel agent runs as root with `/home` and `/root`
  inaccessible; it reports failed units, timer runs, journal lines on request, host metrics,
  network and apt history; an admin on Ciel can start, stop or restart system and user units
  (not ssh.service, ciel-agent, dbus, systemd-*, getty), no custom actions
  (`/etc/ciel/actions.json` absent), no terminal (Docker only, none here); it updates its own
  binary from the hub.
- Keys: aether key `kuro-rom` in `~/.config/aether/key`, Ciel read token `kuro-agents` in
  `~/.config/ciel/token`; no recovery copy (a lost kuro gets new ones).
- Music: Feishin (player for Mephisto, 9s mephisto topic) as the AppImage
  `~/.local/opt/feishin/Feishin.AppImage` (installed 2026-10-15 at 1.17.0, updates itself),
  launcher `~/.local/share/applications/feishin.desktop`, icon in `~/.local/share/icons`, starts at login
  (`~/.config/autostart/feishin.desktop`). Media
  keys reach it through MPRIS (`org.mpris.MediaPlayer2.Feishin`), Cinnamon's default bindings.
- Dev tools: Rust 1.98.1 through rustup in `~/.cargo` (minimal profile, not on PATH: call
  `~/.cargo/bin/cargo`; installed 2026-10-17 for the pulls tests); Node is Mint's 18 (too old for
  Vite builds: `npx tsc --noEmit` works). No Docker and no ssh to roberta, so deploys run from
  mikaela; `courier` (nier skill) not installed yet.

## Open

- [x] (2026-10-15, rom) Tailscale key expiry off for kuro; kuro's Firefox signed in with Ciel
- [x] (2026-10-15) Claude Code 2.1.3 in `~/.local/bin/claude` (official installer), a test request answered through aether; Ciel shows kuro online with its age key. `~/.bashrc` adds `~/.local/bin` to PATH (Mint's `~/.profile` only does it when the folder exists at login, so a session started before the install did not find `claude`)
- [x] (2026-10-15) Look-only check (rom's `~/claude/issues/overseer/kuro-survey.sh`, run over ssh): nothing suspicious. Package sources: Mint, Ubuntu, Spotify, the OBS PPA, Prism Launcher, Tailscale; hand-installed apps known (OBS, Wine, QEMU/libvirt, CurseForge in `/opt`, flatpaks such as Discord, Surfshark, Tor Browser, ProtonPlus); autostart: Surfshark, Emote, Firefox; no custom timers or cron; Firefox extensions uBlock Origin and Bitwarden; no Chrome; 0 updates pending; ufw on; Surfshark's WireGuard runs beside Tailscale
- [x] (2026-10-15, rom) The port on 127.0.0.1:8743 is the Ciel agent's own (local only)
- [ ] Optional virus scan, mainly for Windows files run through Wine (and the gacha skin tools later): rom does it with Claude on kuro (`sudo apt install clamav`, then `clamscan -r` on the home folder) (since 2026-10-15)
- [x] (2026-10-15, rom) LG UltraGear (HDMI-0) turning off: cause was LG's own Automatic Standby (off after about 4 h without a button press; Xorg never saw a signal loss, Mint's timeouts act on both screens). Menu has no English: language set to 日本語 (일반 > 언어), then 一般 > 自動スタンバイ > オフ; works
- [ ] rom: `sudo ~/claude/issues/mephisto-taste/pending/remove-spotify.sh` in a real terminal (purges `spotify-client`, its apt source and keys; Mephisto + Feishin replace it, as on mikaela); the agent already deleted `~/.cache/spotify` and `~/.config/spotify`; then the agent drops Spotify from the package sources above (since 2026-10-15)
- [ ] rom: Feishin login to Mephisto failed on kuro and mikaela (Amperfy works); reset in progress (9s mephisto topic Open); then log in (type Subsonic, user `rom`) and press the media keys while music plays so the agent checks they reach Feishin (since 2026-10-15)
- [ ] Later: gpu-queue (9s), gacha skin tools for Wuthering Waves and Endfield (9s TODO) (since 2026-10-12)
- [ ] Close ssh into kuro (rom's privacy brief, 2026-10-17): (1) before ssh closes, the agent copies to kuro on rom's ok whatever still needs it (`~/claude/issues/aether/pending/kuro-close-ssh.sh`, and the Courier binary and `bin/courier-install` if Courier goes on kuro, aether topic); (2) rom on kuro's own screen: `sudo bash kuro-close-ssh.sh` (add `--shields-up` to refuse every incoming tailnet connection): removes `10-nier.conf`, deletes ufw's 22/tcp on tailscale0, stops, disables and masks `ssh.socket` and `ssh.service`, removes the laptop key from `~/.ssh/authorized_keys` last; (3) rom pastes home-server `host/tailnet-policy.hujson` into https://login.tailscale.com/admin/acls/file; (4) agent on mikaela removes the `Host kuro` ssh entry and `~/.ssh/id_ed25519_kuro*`, and nier `bootstrap/ssh/kuro.pub` (so a rerun of `join.sh` does not put it back), then updates Access above. The gpu-queue plan (9s) relied on this ssh and needs another path (since 2026-10-17)
- [ ] Privacy, rom to pick (proposed 2026-10-17, facts under "Exposure"): disable avahi-daemon, cups and cups-browsed (no printer), bluetooth if unused, libvirtd/libvirt-guests until the VM is needed; turn LightDM autologin off or keep it (LUKS prompt already guards boot); Firefox telemetry off via `/etc/firefox/policies/policies.json` (`DisableTelemetry`, `DisableFirefoxStudies`); Ciel agent: keep reporting, consider denying unit actions for kuro; move personal detail of this page (games, mods, music, apps) from overseer, which work can read, to 9s, leaving machine facts here (since 2026-10-17)

## Detected at registration

- Enabled services: accounts-daemon anacron apparmor avahi-daemon blk-availability blueman-mechanism bluetooth casper-md5check casper ciel-agent console-setup cron cups-browsed cups dmesg e2scrub_reap finalrd getty@ gpu-manager grub-common grub-initrd-fallback irqbalance kerneloops keyboard-setup libvirt-guests libvirt
- Tailscale installed
