---
name: ai-sidebar
summary: Quickshell left-sidebar AI chat on local Ollama (iGPU via Vulkan), models, fixes, aiconfig helper
status: active
owner: shared
host: mikaela
repos: /opt/dotfiles
folder: none
verified: 2026-10-10
source: config/quickshell/ii/services/Ai.qml, services/ai/, custom/scripts/ai-config.py, /etc/systemd/system/ollama.service.d/override.conf, ollama list, history.md 2026-09-12 to 2026-09-14
---

# AI sidebar

The ii left sidebar (the star button at the bar's top left, an upstream toggle) used as a chat
front end for a local Ollama, no API key, no cloud. Code in `/opt/dotfiles/config/quickshell/ii/`
(`services/Ai.qml`, `services/ai/`, `modules/ii/sidebarLeft/`). The persona is rom's: it lives in rom's
`~/.config/illogical-impulse/config.json` and its page is in the personal repo,
`9s/topics/ai-persona.md`. Nothing about it belongs in overseer.

## Facts

- Ollama: system service, Arch `extra/ollama` plus `extra/ollama-vulkan` (official packages; the base
  package has no Intel backend). Recent Ollama drops integrated GPUs unless
  `OLLAMA_IGPU_ENABLE=1`, set in `/etc/systemd/system/ollama.service.d/override.conf` (not yet tracked
  in `/opt/dotfiles/system/`). Confirmed `100% GPU` in `ollama ps` since 2026-09-13.
- Models (`ollama list`): `qwen3:14b`, `qwen3:4b`, and `qwen3:14b-gpu`, a derived model made with a Modelfile
  holding only `PARAMETER num_ctx 16384` on top of `FROM qwen3:14b`. 16384 fits fully on the iGPU
  (about 11 GB); 24576 already spills to CPU, and the default 32768 offloads 26 of 41 layers only.
  Default model in `~/.local/state/quickshell/states.json` (`ai.model`, `:` becomes `_`).
- `num_ctx` and `keep_alive` are honoured only by Ollama's native `/api/chat`, not by the
  `/v1/chat/completions` endpoint this app uses. Hence the derived model, and a client-side idle
  timer in `Ai.qml` (2 minutes after the last reply) that runs `ollama stop <model>`.
  `/clear` and the stop button run `ollama stop` too: Ollama keeps generating after the client leaves.
- Models are discovered by `ollama list` once at quickshell start; a newly pulled model needs a
  quickshell restart (`/opt/nier/tools/qs-restart.sh`). Discovered models need `"api_format": "openai"`
  or any request throws.
- Policy: `policies.ai` in the user's `config.json` must be 2 (Local: no cloud models, non-localhost
  blocked). 0 hides the whole Intelligence tab and shows an empty sidebar.
- Tool modes (`ai.tool`): `none`, `commands` (only `run_shell_command`), `functions` (adds
  `get_shell_config`/`set_shell_config`). Switch with `/tool`.
- Tool calls stream as one `delta.tool_calls` chunk with `finish_reason: "tool_calls"`;
  `OpenAiApiStrategy` originally ignored it, so a tool call vanished silently. Now accumulated and
  emitted as `functionCall`. An empty answer with no error now shows a fallback message.
- Thinking toggle (`/think on|off`, status-row icon, `Persistent.states.ai.thinking`). Ollama's compat endpoint
  still computes the reasoning trace with `think:false`; the app hides it client-side.
- History: the last conversation (`lastSession`) is loaded at startup. The chat scrolls to the end
  on open (debounced on `heightChanged`, the list has height 0 at `onCompleted`).
- Settings `ai.characterName` and `ai.avatarIcon` (Material Symbol name, or a file path rendered
  as a round image) replace the model name and avatar in message headers.
- `aiconfig` (fish alias for `config/hypr/custom/scripts/ai-config.py`): `show` prints all `ai.*`
  fields, `set FIELD VALUE` sets `tool`, `characterName`, `avatarIcon`, `model [name]` lists or sets
  the model, bare `aiconfig` edits the system prompt in `$EDITOR`. Use it instead of rewriting the JSON.
- Chat background is a translucent layer derived from the wallpaper and `appearance.transparency.*`.
- UX changes: hover-revealed message buttons, status icons moved to the bottom row, tab bar hidden
  with one tab, generic assistant avatar. The unmapped hardware AI key listener
  (`ai-key-listener.py` and unit) is retired and unwired.
- Japanese text needs `noto-fonts-cjk` (in `packages.txt`).
- No web search for local models (the user declined it).

## Open

- [x] (2026-10-10, dotfiles 4163eb5) Ollama drop-in tracked in `system/`
- [ ] "UI broke after /key, fixed by reopening" never reproduced; ask for a description if it recurs (since 2026-09-13)

## Log

- 2026-09-13: empty sidebar was `policies.ai: 0`, not a missing model; set to 2; `qwen3:14b` pulled.
- 2026-09-13: local models slow (CPU only): installed `ollama-vulkan`; GPU still dropped until
  `OLLAMA_IGPU_ENABLE=1` (drop-in, user ran the script, rebooted). `qwen3:4b` added as the fast fallback.
- 2026-09-13: derived `qwen3:14b-gpu` (num_ctx 16384) for full offload; verified through the real sidebar.
- 2026-09-13: thinking toggle bug was a missing field on the `Persistent.qml` schema; stop button and
  `ollama stop` on clear added; `commands` tool mode added.
- 2026-09-13: streaming tool-call fix, history autoload (dotfiles ff61119, 15d68a7).
- 2026-09-14: scroll-to-bottom fix and auto-follow (86a8807); `aiconfig set` added.
