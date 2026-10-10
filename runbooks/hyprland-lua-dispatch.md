# Hyprland Lua dispatch (mikaela)

This is the Lua-config Hyprland (0.55+). `hyprctl dispatch <text>` no longer takes the classic
`<dispatcher> <args>` syntax: it evaluates `<text>` as Lua, literally `return hl.dispatch(<text>)`.
To run a dispatcher with arguments from a shell or script, for example `exec_cmd` with
window-placement rules:

```
hyprctl dispatch "hl.dsp.exec_cmd('shell command', {workspace = '3', float = true, move = '100 100', size = '640 480', fullscreen = true})"
```

Confirmed keys for `exec_cmd`'s second (table) argument: `workspace` (name string, including
`special:<name>`), `float`, `move` ("x y"), `size` ("w h"), `fullscreen` (bool). Values are Lua
syntax (quoted strings, `true`/`false`), not shell or JSON.

`hyprctl binds -j` does not show the command behind an `hl.bind` key (the dispatcher shows as an
opaque `__lua`): match on the `description` field instead.
