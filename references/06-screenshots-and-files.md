# Screenshots & Files

File-writing commands: `take_screenshot`, `take_heapsnapshot`,
`performance_stop_trace`, `lighthouse_audit`, `upload_file`. All are
sandboxed to `/tmp`.

## Why `/tmp` only?

By default the server only allows writes under `/tmp`. The escape hatch
is the `--allowUnrestrictedPaths` server flag — **do not pass it to
`chrome-devtools start`**. The `cdt` wrapper never enables it.

```bash
# Write to /tmp, then move to final location
cdt take_screenshot --filePath /tmp/page.png
mv /tmp/page.png ./screenshots/page.png
```

## CLI shape cheat sheet

| Command | filePath style |
| --- | --- |
| `take_screenshot` | `--filePath /tmp/p.png` (flag) |
| `take_snapshot` | `--filePath /tmp/snap.txt` (flag) |
| `take_heapsnapshot` | `take_heapsnapshot /tmp/snap.heapsnapshot` (positional) |
| `performance_start_trace` / `performance_stop_trace` | `--filePath /tmp/trace.json.gz` (flag) |
| `lighthouse_audit` | `--outputDirPath /tmp/lh` (flag) |
| `upload_file` | `upload_file <uid> /tmp/file.txt` (positional) |

## Screenshots

`take_screenshot [--format] [--quality] [--uid] [--fullPage]
[--filePath]`. Omit `--filePath` to get base64 inline (useful when the
agent needs to view it).

```bash
# Default: PNG
cdt take_screenshot --filePath /tmp/page.png

# JPEG with quality (smaller, good for HTML reports)
cdt take_screenshot --filePath /tmp/page.jpg --format jpeg --quality 80

# WebP
cdt take_screenshot --filePath /tmp/page.webp --format webp

# Full page (not just viewport)
cdt take_screenshot --filePath /tmp/full.png --fullPage

# Single element capture (uid is a flag, not positional, here)
cdt take_screenshot --filePath /tmp/btn.png --uid 1_3
```

## Heapsnapshot

```bash
cdt take_heapsnapshot /tmp/heap.heapsnapshot   # filePath is positional
```

Output is the binary Chrome DevTools `.heapsnapshot` format. Inspect in
DevTools Memory panel or `chrome://inspect` — not as text. Multiple
snapshots can be loaded and queried via `get_heapsnapshot_*` tools
(see `08-advanced-features.md`); note those use positional filePath too.

## Traces and Lighthouse reports

`performance_stop_trace --filePath /tmp/trace.json.gz` saves gzipped
JSON. Lighthouse writes JSON+HTML reports into
`--outputDirPath /tmp/lh-reports`. See `04-performance-and-audit.md`
for the full workflow.