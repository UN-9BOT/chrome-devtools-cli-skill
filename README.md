# chrome-devtools-cli skill

A thin CLI wrapper and agent-friendly skill for Google's
[`chrome-devtools-mcp`](https://github.com/ChromeDevTools/chrome-devtools-mcp) —
browser automation, inspection, and debugging via the DevTools Protocol.

The skill exposes a single binary, `cdt`, that delegates to the official
`chrome-devtools` daemon and manages its lifecycle (auto-detect, restart,
privacy defaults). It is designed to be invoked by AI agents from the shell.

## Installation

```bash
npx skills add UN-9BOT/chrome-devtools-cli-skill --skill chrome-devtools-cli
```

This installs the skill into your agent's skills directory and sets up the
`cdt` wrapper on `$PATH`.

### Manual install

If you don't use the `skills` CLI:

```bash
# 1. Install the official chrome-devtools CLI (one-time)
npm i -g chrome-devtools-mcp

# 2. Install this skill into your agent's skills directory
#    For opencode:
mkdir -p ~/.agents/skills
git clone https://github.com/UN-9BOT/chrome-devtools-cli-skill.git \
    ~/.agents/skills/chrome-devtools-cli
# Or, if you have the repo locally:
# cp -r chrome-devtools-cli ~/.agents/skills/

# 3. Symlink the wrapper into ~/.local/bin
mkdir -p ~/.local/bin
ln -sf ~/.agents/skills/chrome-devtools-cli/scripts/cdt ~/.local/bin/cdt
ln -sf ~/.agents/skills/chrome-devtools-cli/scripts/cdt ~/.local/bin/cdt-iso    # always isolated
ln -sf ~/.agents/skills/chrome-devtools-cli/scripts/cdt ~/.local/bin/cdt-live   # always :9222

# 4. Verify
which cdt && cdt --doctor
```

## What it does

- **Navigate pages**, click/fill forms, capture screenshots and a11y snapshots
- **Inspect** console messages and network requests
- **Profile memory** — heap snapshots, dominators, retainers, leaks
- **Record performance traces**, run Lighthouse audits
- **Manage Chrome extensions**, PWAs, and experimental features
- **Emulate** viewports, network conditions, CPU throttling, color schemes

Every `cdt` command maps to a DevTools tool. See `SKILL.md` for the
agent-facing reference.

## Requirements

- **Linux** (tested) or **macOS** (best-effort; uses `ss`/`lsof`/`netstat`)
- **Node.js** ≥ 18 (for `npm i -g chrome-devtools-mcp`)
- **Common Unix tools**: `sh`, `grep`, `sed`, `awk`, `pgrep`, `jq`, `curl`
  - On macOS install via `brew install jq`
- A **Chrome**, Chromium, or Brave binary on `$PATH` (or `cdt` will spawn
  its own ephemeral Chrome)

## Quick start

```bash
# Show the full browser-automation environment
cdt --doctor

# List open tabs (auto-detects :9222 or launches ephemeral Chrome)
cdt list_pages

# Navigate, snapshot, click
cdt new_page https://example.com
cdt take_snapshot
cdt click 1_3                             # positional uid

# Capture evidence
cdt take_screenshot --filePath /tmp/page.png
cdt list_console_messages --types '["error"]'

# Clean up
cdt close_page 1                          # positional pageId
```

## Browser setup for auto-detect

`cdt` auto-detects a Chrome/Chromium/Brave browser that is already
listening on `127.0.0.1:9222` (the standard DevTools Protocol port) and
automates your real tabs. To enable this, launch your browser with the
`--remote-debugging-port` flag:

```bash
# Brave
brave --remote-debugging-port=9222 &

# Google Chrome
google-chrome --remote-debugging-port=9222 &

# Chromium
chromium --remote-debugging-port=9222 &

# macOS (different binary path)
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" \
    --remote-debugging-port=9222 &
```

If nothing is listening on `:9222`, `cdt` falls back to launching its
own ephemeral headless Chrome (`--isolated`) — useful for CI or
sandboxed environments. See `references/11-browsers-and-ports.md` for
trade-offs.

**Security note:** `--remote-debugging-port=9222` exposes **full** read
and write access to every open tab to **any** process on your machine
that can connect to `127.0.0.1`. Only enable this on a trusted machine
where you control which processes run. Do not bind to a public
interface.

## Privacy and security

The wrapper never enables Google telemetry or CrUX performance data:

- `--no-usage-statistics` (Google data collection off)
- `--no-performance-crux` (field performance data off)
- `--allow-unrestricted-paths` is **never** passed — file writes are
  sandboxed to `/tmp` by default. Workaround: write to `/tmp` then `mv` to
  destination.

## Customization

### Per-invocation env vars

| Env var | Effect |
| --- | --- |
| `CDT_BROWSER_URL=isolated` | Force ephemeral Chrome (no live browser) |
| `CDT_BROWSER_URL=http://host:port` | Connect to a specific browser |
| `CDT_START_FLAGS="--memoryDebugging=true --categoryPwa=true"` | Add server flags (memory, PWA, vision, screencast, etc.); daemon restarts with extras |

### Persistent aliases (symlinks)

The wrapper is busybox-style — symlink it under different names to get
persistent modes without per-call env vars:

```bash
ln -sf ~/.agents/skills/chrome-devtools-cli/scripts/cdt ~/.local/bin/cdt
ln -sf ~/.agents/skills/chrome-devtools-cli/scripts/cdt ~/.local/bin/cdt-iso    # always isolated
ln -sf ~/.agents/skills/chrome-devtools-cli/scripts/cdt ~/.local/bin/cdt-live   # always :9222
```

| Binary | Effect |
| --- | --- |
| `cdt` | auto-detect `:9222`, fall back to isolated |
| `cdt-iso` | always isolated ephemeral Chrome |
| `cdt-live` | always connect to `127.0.0.1:9222` |

Use `cdt stop` to reset the daemon between modes.

## Project layout

```
SKILL.md                   Agent-facing reference (~1700 tokens)
references/
  01-elements-and-interaction.md
  02-scripting.md
  03-emulation.md
  04-performance-and-audit.md
  05-inspection.md
  06-screenshots-and-files.md
  07-output-handling.md
  08-advanced-features.md
  09-navigation.md
  11-browsers-and-ports.md
scripts/
  cdt                      CLI wrapper (delegates to chrome-devtools)
  cdt-doctor               Diagnostic report
evals/
  cases.yaml               Activation + behavior cases for AI-judge eval
```

## Contributing

Issues and PRs welcome. Test your changes with:

```bash
cdt --help
cdt --doctor
cdt list_pages
cdt stop
```

Run the
[meta-skill-zettelkasten](https://github.com/UN-9BOT/meta-skill-zettelkasten)
validators (optional — install the skill first if you don't have it):

```bash
npx skills add UN-9BOT/meta-skill-zettelkasten --skill meta-skill-zettelkasten -y -g
python3 ~/.agents/skills/meta-skill-zettelkasten/scripts/validate_skill.py .
python3 ~/.agents/skills/meta-skill-zettelkasten/scripts/analyze_skill.py .
```

## License

MIT — see `LICENSE`. Wraps Google `chrome-devtools-mcp` (Apache 2.0);
that package is installed separately, not redistributed.