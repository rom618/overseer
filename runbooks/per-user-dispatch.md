# Per-user behavior in shared config (mikaela)

`/opt/dotfiles/config/hypr/custom` is shared storage but the place for per-user behavior: it
branches at runtime on `$USER` instead of holding two copies.

Lua:

```lua
local user = os.getenv("USER")

if user == "rom" then
    require("custom.rom")
elseif user == "work" then
    require("custom.work")
end
```

Shell:

```bash
#!/usr/bin/env bash
case "$USER" in
    rom) exec "$HOME/.config/hypr/custom/scripts/rom-startup.sh" ;;
    work) exec "$HOME/.config/hypr/custom/scripts/work-startup.sh" ;;
esac
```

Record every intentional difference in the "User differences" table of
`overseer/machines/mikaela.md`.
