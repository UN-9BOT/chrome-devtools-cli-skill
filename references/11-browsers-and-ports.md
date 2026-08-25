# Browsers & Ports

How `cdt` connects to a browser, where state persists, and how to share
state with the rest of your environment.

## How `cdt` connects

```
cdt list_pages                # foreground command (exits after request)
  └─ chrome-devtools list_pages   # Node CLI thin wrapper (also exits)
       └─ daemon (daemon.js, persistent — started by cdt on first call)
            └─ UDS socket: /run/user/$UID/chrome-devtools-mcp/server.sock
                 └─ chrome-devtools-mcp server
                      └─ Chrome browser (--browser-url=:9222 OR --isolated)
```

The daemon persists across `cdt` calls — its tab list, console buffer,
and network log all accumulate. Restart with `cdt stop` to reset.

## Auto-detect: :9222 vs --isolated

`cdt` decides which browser to connect to on every call (it may restart
the daemon if the decision changes):

```bash
# What does cdt see?
cdt --doctor
# DevTools Protocol endpoints ==
#   ✓ 127.0.0.1:9222  pid=...  (brave)       ← your live browser
#   ...
# cdt decision (what the next call will do) ==
#   ✓ auto-detect → --browser-url=http://127.0.0.1:9222  (your live browser)
```

| `:9222` listening? | target_mode | start args |
| --- | --- | --- |
| yes | `auto` | `--browserUrl=http://127.0.0.1:9222` |
| no | `auto` | `--isolated` (ephemeral Chrome) |
| override `CDT_BROWSER_URL=isolated` | `isolated` | `--isolated` |
| override `CDT_BROWSER_URL=http://h:9333` | `browserurl` | `--browserUrl=http://h:9333` |

Override per-call with `CDT_BROWSER_URL`. To switch modes permanently:
`cdt stop`, then call `cdt` again with the desired env var.

## What state does the daemon keep?

| Persists across `cdt` calls | Resets on `cdt stop` |
| --- | --- |
| Open tabs (selected page, order) | Loaded heapsnapshots (`get_heapsnapshot_*` results) |
| Console messages buffer | Server-level emulation settings (viewport, network, etc.) |
| Network log | |
| Navigation history per tab | |
| Cookies/localStorage in the connected profile | |
| Heapsnapshots (until `close_heapsnapshot` or restart) | |

## What else is on your machine

If you use multiple agents (opencode, Claude Code, Cursor, etc.) or
other browser-automation tools, you may have **additional
chrome-devtools-mcp processes** that `cdt` does NOT own:

```bash
# Linux
ss -tlnp | grep 9222
# 127.0.0.1:9222  ...  users:(("brave",pid=...))   ← user-launched browser

# macOS
lsof -nP -iTCP:9222 -sTCP:LISTEN

ps -ef | grep chrome-devtools-mcp | grep -v grep
# /path/to/your-agent/.../mcp_stdio_watchdog.py -- ... --browser-url=...
# chrome-devtools-mcp                                  ← other agent's MCP client
```

These are independent processes that connect to whatever browser the
user (or another tool) is running. They do NOT share state with the
daemon `cdt` manages — `cdt` only owns the daemon it itself starts.

## Trade-offs

| Mode | Pros | Cons |
| --- | --- | --- |
| Auto-detect with live browser (`:9222` up) | Same tabs as your other tools. One fewer Chrome to launch. | Automation touches your real browser — a misclick could overwrite a real login form. If `:9222` goes down, the daemon fails too. |
| `--isolated` (own ephemeral Chrome) | No risk of touching your real browser. Works offline. | Tab state doesn't share with other tools. Ephemeral — clean profile per restart (no cookies, no logins). |

## Sharing state explicitly

If you want the daemon to talk to a SPECIFIC browser, not auto-detect:

```bash
# A specific browser on a different port
CDT_BROWSER_URL=http://localhost:9333 cdt list_pages

# Force isolated (ephemeral)
CDT_BROWSER_URL=isolated cdt list_pages
```

## Diagnosing "why doesn't cdt see my page"

```bash
# 1. One-shot diagnosis
cdt --doctor

# 2. Confirm which mode cdt is in
cdt list_pages
# If it shows tabs you didn't open via cdt, you're connected to a shared browser.
# If it shows only about:blank, you're in --isolated mode.

# 3. Switch modes
cdt stop
cdt list_pages               # default: re-runs auto-detect

# 4. Reset the daemon entirely (kills the daemon process; next cdt recreates)
cdt stop
rm /run/user/$UID/chrome-devtools-mcp/server.sock   # only if daemon is wedged
```

## When to read other references

- Service model and daemon lifecycle: `SKILL.md`
- Memory / extensions / PWA / experimental (gated by start flags):
  `08-advanced-features.md`