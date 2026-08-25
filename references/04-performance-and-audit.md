# Performance & Audit

`performance_start_trace`, `performance_stop_trace`,
`performance_analyze_insight`, and `lighthouse_audit`. All output
**large** artifacts — always set `--filePath`.

## Performance trace workflow

```bash
# 1. Navigate FIRST (--reload traces the current URL)
cdt navigate_page --type url --url "https://target.com"

# 2. Start trace; --reload captures the fresh navigation automatically
cdt performance_start_trace --reload --autoStop --filePath /tmp/trace.json.gz

# 3. After auto-stop, analyze an insight set
cdt performance_analyze_insight <insightSetId> LCPBreakdown
```

Note: `performance_analyze_insight` takes both `<insightSetId>` and the
insight name as **positional** args.

Trace flag reference:

| Flag | Effect |
| --- | --- |
| `--reload` | Reload the current page and trace the fresh navigation |
| `--autoStop` | Auto-stop the trace when the page is idle (recommended) |
| `--filePath /tmp/trace.json.gz` | Save trace directly to file (skip inline output) |

- `performance_start_trace --reload` only works if the **current page**
  is the one you want to trace. Navigate first.
- Trace files are `.json.gz` (gzipped JSON). Load into DevTools
  Performance tab directly.
- Common insight names: `DocumentLatency`, `LCPBreakdown`,
  `INPBreakdown`, `CLSBreakdown`. Run `--help` for the full list.

## Lighthouse audit

| Mode | Behavior | Use when |
| --- | --- | --- |
| `navigation` (default) | Reloads the page and audits the fresh load | Performance, LCP, FCP, SEO of public pages |
| `snapshot` | Audits current DOM state without reloading | SPAs / internal pages where reload would lose state |

```bash
cdt lighthouse_audit --mode navigation --outputDirPath ./lh-reports
```

Lighthouse audits always reload in `navigation` mode — expect fresh
page state. `--mode snapshot` excludes performance category (Lighthouse
limitation).

## Related

- File output and `/tmp` sandbox: see `06-screenshots-and-files.md`.
- Tracing on a specific tab (not the selected one): currently
  `performance_*` traces the selected page only. If you need to trace a
  background tab, `select_page` first.