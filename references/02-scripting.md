# Scripting (`evaluate_script`)

Rules for the `evaluate_script` command. The MCP server runs your
function in the page context and JSON-serializes the return value.

## JSON-serializable returns only

Functions that return DOM nodes, `Map`, `Set`, `Promise`, functions, etc.
fail with serialization errors. Wrap in objects of primitive fields.

```bash
# BAD — returns a DOM element
cdt evaluate_script '() => document.querySelector("h1")'

# GOOD — returns an object of primitives
cdt evaluate_script '() => ({title: document.title, h1: document.querySelector("h1")?.innerText})'

# GOOD — awaits a promise
cdt evaluate_script 'async () => { const r = await fetch("/api"); return await r.json(); }'
```

## Async, --args, and platform limits

- Declare functions `async` to use `await`.
- Pass snapshot UIDs via `--args` (JSON array of strings) — the server
  resolves them to element handles inside the function scope.
- `evaluate_script` **cannot** access `chrome.*` extension APIs or other
  privileged Chrome internals — only standard web platform +
  DevTools-exposed surfaces.

## Dialogs block the script

`window.confirm()`, `window.prompt()`, `window.alert()` leave a
single-slot dialog handler. If you trigger one via `evaluate_script`
without arming the handler, the script hangs.

```bash
# Arm the dialog handler BEFORE triggering a confirm/prompt
cdt handle_dialog --dialogAction accept
cdt evaluate_script '() => confirm("ok?")'
```

For `--dialogAction`: `accept` or `dismiss`. For `prompt`, also pass
`--promptText "value"`.

## When to read other references

- Element interaction (passing UIDs via `--args`): `01-elements-and-interaction.md`.