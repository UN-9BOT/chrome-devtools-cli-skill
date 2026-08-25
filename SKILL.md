---
name: chrome-devtools-cli
description: Control a real Chrome browser via Google's chrome-devtools-mcp daemon. Navigate pages, click/fill forms, capture screenshots and a11y snapshots, run Lighthouse audits, record performance traces, inspect console/network, profile memory, install extensions, test PWAs. Trigger when asked to browse the web, automate a browser flow, scrape a page, test a UI, or debug a webpage.
allowed-tools: Bash(bash *)
---

# Chrome DevTools MCP CLI

Browser automation, inspection, and debugging through Google's
`chrome-devtools-mcp` daemon via the `cdt` wrapper. Every `cdt` command
maps to a DevTools tool. The wrapper auto-detects an existing browser
on `:9222` and manages the daemon lifecycle so state persists across
calls.

## Install

The skill ships a wrapper at `scripts/cdt`. It must be reachable on
`$PATH` (and the `chrome-devtools` CLI must be installed separately):

```bash
# 1. Install the official chrome-devtools CLI (one-time)
npm i -g chrome-devtools-mcp

# 2. Symlink the wrapper into ~/.local/bin (idempotent)
mkdir -p ~/.local/bin
ln -sf ~/.agents/skills/chrome-devtools-cli/scripts/cdt ~/.local/bin/cdt

# 3. Verify
which cdt && cdt --doctor
```

The wrapper requires `chrome-devtools` on `PATH` (or in
`~/.npm-global/bin/chrome-devtools`). It calls `chrome-devtools status`,
`start`, `stop`, and forwards all other commands to the running daemon.

## Browser setup (for auto-detect)

`cdt` auto-detects a Chrome/Chromium/Brave already listening on
`127.0.0.1:9222` (the DevTools Protocol port) and automates your real
tabs. Enable this by launching your browser with
`--remote-debugging-port=9222`:

```bash
brave --remote-debugging-port=9222 &                    # Brave
google-chrome --remote-debugging-port=9222 &           # Google Chrome
chromium --remote-debugging-port=9222 &                # Chromium
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" \
    --remote-debugging-port=9222 &                      # macOS
```

If nothing is on `:9222`, `cdt` falls back to launching its own
ephemeral headless Chrome (`--isolated`). See
`references/11-browsers-and-ports.md` for the trade-offs.

**Security:** `--remote-debugging-port=9222` exposes **full** read/write
access to every open tab to any process that can reach `127.0.0.1`.
Only enable on a trusted machine. Never bind to a public interface.

## Service model

`cdt` is a thin shim over a **persistent daemon**:

```
cdt <cmd>           # foreground command (read/write only)
  └─ chrome-devtools <cmd>     # Node CLI (exits after the call)
       └─ daemon (daemon.js, long-running)
            └─ /run/user/$UID/chrome-devtools-mcp/server.sock (UDS)
                 └─ chrome-devtools-mcp server process
                      └─ Chrome (one profile: $HOME/.cache/chrome-devtools-mcp/chrome-profile)
```

The wrapper manages the daemon lifecycle:

| Trigger | Effect |
| --- | --- |
| `:9222` listening on `127.0.0.1` | Start daemon with `--browserUrl=http://127.0.0.1:9222` (talks to your existing browser) |
| `:9222` not listening | Start daemon with `--isolated` (launches ephemeral Chrome) |
| Daemon running with wrong args | Stop + restart |
| `cdt stop` | Stop daemon (frees memory, lets next call re-decide) |

Override per call with `CDT_BROWSER_URL=isolated|http://host:port`. The
override is enforced by restarting the daemon if needed (state is
persistent between calls until you `cdt stop`).

**Telemetry is always off** (`--no-usage-statistics --no-performance-crux`)
and **path sandbox is always on** (never `--allow-unrestricted-paths`).

## Quick start

```bash
# Show the full browser-automation environment
cdt --doctor

# List all open tabs (in your live browser if :9222 is up)
cdt list_pages

# Get help for any command (also reveals exact flag names)
cdt navigate_page --help

# Run a command (auto-starts daemon if needed)
cdt take_snapshot

# Pipe JSON-shaped output through jq
cdt evaluate_script '() => document.title' | jq .
```

**Naming convention** (chrome-devtools CLI, not mcp2cli):

- Commands: `snake_case` — `list_pages`, `take_snapshot`, `evaluate_script`
- Flags: `camelCase` — `--includeSnapshot`, `--browserUrl`, `--filePath`
- Positional args for identifiers — `cdt click 1_3` (not `--uid "1_3"`),
  `cdt close_page 1` (not `--page-id 1`)
- File paths are mostly positional — `cdt take_heapsnapshot /tmp/snap.heapsnapshot`,
  `cdt upload_file <uid> /tmp/file.txt`. Exceptions: `take_screenshot`,
  `take_snapshot`, `performance_*`, `lighthouse_audit` use `--filePath` /
  `--outputDirPath` flags.

`cdt <command> --help` always shows the exact shape — don't guess.

## Core workflow

A typical automation flow:

```bash
# 1. Open a fresh tab
cdt new_page https://example.com

# 2. Capture a11y snapshot to discover element UIDs
cdt take_snapshot

# 3. Act on elements (use UIDs from step 2; positional arg)
cdt click 1_3
cdt fill 2_5 "search query"

# 4. Capture evidence
cdt take_screenshot /tmp/page.png
cdt list_console_messages --types '["error"]'
cdt list_network_requests --pageSize 20

# 5. Clean up (positional pageId)
cdt close_page 1
```

## Before Querying — decision framework

Apply these checks before every `cdt` call:

- **Tab management**: only the **selected** page accepts commands. Use
  `list_pages` to see what's open (marked `[selected]`) and
  `select_page <pageId> [--bringToFront]` to switch. `<pageId>` is the
  1-based index from `list_pages`.
- **UID freshness**: UIDs from `take_snapshot` are page-scoped and expire
  on navigation or DOM mutation. After any navigate/click, re-snapshot —
  stale UIDs return "element not found".
- **Snapshot vs screenshot**: prefer `take_snapshot` (text, cheap,
  indexable) for finding elements or extracting data; use
  `take_screenshot` only for visual proof, OCR, or CSS layout debugging.
- **Last page cannot be closed**: `close_page` refuses the final tab. Open
  a new one first if you need to clear state.
- **File writing is sandboxed**: screenshots, traces, heapsnapshots can
  only write to `/tmp` by default. **Never pass `--allow-unrestricted-paths`**
  to `chrome-devtools start` — it grants write access to the whole
  filesystem. Workaround: write to `/tmp` then `mv`/`cp` to the
  destination.
- **Daemon restarts are expensive** (~3 s): avoid starting/stopping the
  daemon in tight loops. Once it's up with the right args, it stays up
  until you `cdt stop`.

## Reference index

Read a reference when the task matches its trigger condition.

| Trigger condition | Reference |
| --- | --- |
| Interacting with elements (click, fill, type, drag, hover, upload, key press) | `references/01-elements-and-interaction.md` |
| Calling `evaluate_script` (return values, async, args, dialog handling) | `references/02-scripting.md` |
| Calling `emulate` (viewport, network, CPU, headers, geo, color scheme) | `references/03-emulation.md` |
| Recording a performance trace or running Lighthouse | `references/04-performance-and-audit.md` |
| Reading console messages or network requests | `references/05-inspection.md` |
| Writing screenshot, heapsnapshot, or trace files | `references/06-screenshots-and-files.md` |
| Piping `cdt` output to jq/grep, debugging server banner noise | `references/07-output-handling.md` |
| Memory debugging, extensions, PWA, experimental features (need extra server flags) | `references/08-advanced-features.md` |
| Page lifecycle (new_page, navigate_page types, select_page, close_page, tabs) | `references/09-navigation.md` |
| Which browser `cdt` connects to, how the daemon picks :9222 vs --isolated, sharing state with other tools | `references/11-browsers-and-ports.md` |

## Resources

- Source: https://github.com/ChromeDevTools/chrome-devtools-mcp
- MCP spec: https://modelcontextprotocol.io/
- Official CLI: `chrome-devtools` (installed with `chrome-devtools-mcp`)