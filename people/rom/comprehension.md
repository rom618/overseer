# Comprehension log (rom)

Append-only. One line per observation, newest last. Current level = last line of a concept.
Levels: 0 unknown, 1 heard of it, 2 can decide, 3 solid. Concepts only: no secrets, no project
names, nothing from private media. Cap 200 lines; compact past 150 (last line per concept).

date | concept | level | evidence | what worked
2026-10-10 | tailscale-vs-headscale | 0 | said "i dont know which does what" | coordination server vs client
2026-10-10 | backup-snapshots-deduplication | 0 | asked why there are 36+ snapshots and why they are so small | each snapshot is a list over shared stored pieces; retention tiers
2026-10-10 | scripts-in-a-config-repo | 1 | asked how a repo command runs on a device that is not registered, thought it a binary | it is a script in the cloned repo; registration only changes what it finds
2026-10-10 | device-enrollment | 1 | asked how the monitoring app would register devices | one-time code, agent reports, user approves
2026-10-10 | magicdns | 2 | used the term correctly when asking for the rename | -
2026-10-11 | vpn-is-machine-wide-not-per-user | 0 | surprised the tailnet stayed up across reboots and users, linked it to ssh keys | cable between machines vs key to an account
2026-10-11 | ssh-public-key-access | 1 | asked what giving a public key grants | only the account it is added to, nothing else
2026-10-13 | sops-age-secret-files | 0 | named it as the way to keep secrets, then said "jsp" | -
2026-10-13 | tailscale-traffic-path | 1 | asked whether the tailnet goes through someone else's server | coordination server vs direct path vs relay
2026-10-13 | sops-age-secret-files | 0 | answered "idk" after the first explanation | second angle: two kinds of secrets, a locked box and its key
2026-10-13 | sops-age-secret-files | 2 | picked one key per machine and gave the reason (each machine is recorded on its own) | concrete rebuild walkthrough, numbered steps
2026-10-13 | user-key-vs-machine-key | 1 | asked whether the laptop already had an age key (it had the user key, not a machine key) | a key that follows the person vs a key that belongs to the computer
2026-10-13 | user-key-vs-machine-key | 1 | asked again why there are two keys after the first explanation | master key of a person vs the lock of one computer; said plainly the laptop one is unused today
2026-10-13 | llm-subscription-vs-api-key | 2 | asked why the billing question was needed, then picked the claude -p option knowing it shares quota | plain two-ways-to-reach-Claude framing, then the trade-off
2026-10-14 | tailscale-traffic-path | 1 | asked again whether the tailnet goes through someone's server, after the 2026-10-13 explanation | phone book vs direct call, relay as a sealed envelope
2026-10-14 | proxy-client-keys | 0 | said they did not understand how signing in to the AI proxy works | -
