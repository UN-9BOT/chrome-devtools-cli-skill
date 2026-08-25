# Advanced Features

Some capabilities are gated behind **server-start flags** — the daemon
must be running with them enabled. With `cdt`, set them via the
`CDT_START_FLAGS` env var; if the running daemon doesn't have them,
`cdt` restarts with the new flags added.

## When do you need this?

| Symptom | You need |
| --- | --- |
| `cdt get_heapsnapshot_summary /tmp/snap.heapsnapshot` returns "requires experimental feature --memoryDebugging" | `--memoryDebugging=true` |
| `cdt install_extension /path/to/ext` returns "invalid choice" | `--categoryExtensions=true` (already on by default — should just work) |
| `cdt launch_pwa https://example.com` returns "invalid choice" | `--categoryPwa=true` |
| `cdt screencast_start /tmp/v.mp4` returns "invalid choice" | `--experimentalScreencast=true` (+ ffmpeg on PATH) |
| `cdt click_at 100 200` returns "invalid choice" | `--experimentalVision=true` |
| `cdt list_webmcp_tools` returns "invalid choice" | `--categoryExperimentalWebmcp=true` |
| `cdt list_3p_developer_tools` returns "invalid choice" | `--categoryExperimentalThirdParty=true` |

## How to use them with `cdt`

```bash
# Memory debugging: investigate a leak
CDT_START_FLAGS="--memoryDebugging=true" \
  cdt take_heapsnapshot /tmp/before.heapsnapshot
# ... trigger the leak in the browser ...
CDT_START_FLAGS="--memoryDebugging=true" \
  cdt take_heapsnapshot /tmp/after.heapsnapshot
CDT_START_FLAGS="--memoryDebugging=true" \
  cdt get_heapsnapshot_summary /tmp/before.heapsnapshot | head -20
CDT_START_FLAGS="--memoryDebugging=true" \
  cdt compare_heapsnapshots /tmp/before.heapsnapshot /tmp/after.heapsnapshot

# PWA: install + launch
CDT_START_FLAGS="--categoryPwa=true" \
  cdt install_pwa https://example.com
CDT_START_FLAGS="--categoryPwa=true" \
  cdt launch_pwa https://example.com

# Multiple categories at once
CDT_START_FLAGS="--memoryDebugging=true --categoryPwa=true --experimentalScreencast=true" \
  cdt screencast_start /tmp/recording.mp4
```

`cdt` only restarts the daemon if the requested flag isn't already
active. Once running, the daemon keeps that flag across subsequent
calls (until you `cdt stop`).

## Full flag → tools mapping

| Flag | Tools added when enabled |
| --- | --- |
| `--memoryDebugging=true` | `take_heapsnapshot` (works without flag actually), `get_heapsnapshot_summary`, `get_heapsnapshot_class_nodes`, `get_heapsnapshot_details`, `get_heapsnapshot_dominators`, `get_heapsnapshot_retainers`, `get_heapsnapshot_retaining_paths`, `get_heapsnapshot_edges`, `get_heapsnapshot_object_details`, `get_heapsnapshot_duplicate_strings`, `compare_heapsnapshots`, `close_heapsnapshot` |
| `--categoryExtensions=true` (default) | `list_extensions`, `install_extension`, `uninstall_extension`, `reload_extension`, `trigger_extension_action` |
| `--categoryPwa=true` | `install_pwa`, `launch_pwa`, `get_os_app_state`, `uninstall_pwa` |
| `--experimentalVision=true` | `click_at <x> <y>` |
| `--experimentalScreencast=true` (+ ffmpeg) | `screencast_start`, `screencast_stop` |
| `--categoryExperimentalWebmcp=true` | `list_webmcp_tools`, `execute_webmcp_tool` |
| `--categoryExperimentalThirdParty=true` | `list_3p_developer_tools`, `execute_3p_developer_tool` |

## Manual control (if you want)

Skip `cdt` and start the daemon yourself:

```bash
chrome-devtools stop
chrome-devtools start \
  --memoryDebugging=true \
  --categoryPwa=true \
  --no-usage-statistics \
  --no-performance-crux
chrome-devtools status   # verify args
chrome-devtools take_heapsnapshot /tmp/snap.heapsnapshot
chrome-devtools stop
```

`cdt` and the manual daemon share the same UDS socket (`/run/user/$UID/chrome-devtools-mcp/server.sock`), so don't start both at once.

## Don't forget

- `--no-usage-statistics` and `--no-performance-crux` are always passed
  by `cdt` — you don't need to add them.
- `--allowUnrestrictedPaths` is **never** passed by `cdt` — don't try
  to add it via `CDT_START_FLAGS`. The wrapper will refuse.
- Heapsnapshots and screenshots are still sandboxed to `/tmp` even with
  advanced flags. Workaround: write to `/tmp` then `mv` to final
  location.

## When to read other references

- Standard 30-ish tools workflow: `SKILL.md` + `references/01–07.md`, `09-navigation.md`
- Why the daemon picks `:9222` vs `--isolated`: `11-browsers-and-ports.md`