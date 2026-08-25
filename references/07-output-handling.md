# Output Handling

`cdt` output is whatever the MCP daemon emits — markdown for some
commands (`list_pages`, `take_snapshot`), raw JSON for others
(`evaluate_script`, `list_network_requests`,
`list_console_messages`). There are no `--pretty` or `--json` flags;
pipe through `jq` instead.

## Piping JSON output through jq

```bash
# Pretty-print JSON-shaped output
cdt evaluate_script '() => document.title' | jq .

# Extract one field
cdt evaluate_script '() => document.title' | jq -r .result

# Combine with grep / awk
cdt list_network_requests | jq -r '.[].url' | grep -i api
```

Note: not every command produces JSON. Markdown-style output (e.g.,
`list_pages`) won't parse with jq.

## Server startup banner

Some commands print a Node warning before the actual output:

```
(node:111857) ExperimentalWarning: localStorage is not available because --localstorage-file was not provided.
(Use `node --trace-warnings ...` to show where the warning was created)
```

It's emitted by the chrome-devtools Node binary itself (a side effect of
it loading the daemon protocol module). It's informational and can be
silenced when piping:

```bash
cdt evaluate_script '() => document.title' 2>/dev/null | jq .
```

## Flag-name convention

Unlike the older `mcp2cli` wrapper, `cdt` does **not** rename flags.
The chrome-devtools CLI uses camelCase (`--includeSnapshot`,
`--browserUrl`, `--ignoreCache`) and positional args for identifiers
(`<uid>`, `<pageId>`, `<filePath>`). Run `cdt <command> --help` to see
the exact shape — do not guess.