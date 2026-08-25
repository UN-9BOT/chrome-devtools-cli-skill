# Navigation & Tab Management

`list_pages`, `new_page`, `navigate_page`, `select_page`, `close_page`
— all commands that change which page is active or what URL is loaded.

## Tab state basics

- Only the **selected** page accepts commands.
- `<pageId>` is the 1-based index from `list_pages` output (matches the
  number prefixed in `uid` strings like `1_3`).
- `close_page` refuses to close the **last** remaining tab. Open a new
  one first if you need to clear state.

## list_pages / select_page

```bash
# What's open?
cdt list_pages
# ## Pages
# 1: about:blank [selected]
# 2: https://example.com

# Switch to a different tab (positional pageId)
cdt select_page 2
cdt select_page 2 --bringToFront   # also focus the tab
```

`--bringToFront` is needed when a workflow expects the browser window
to be visible/focused (e.g., screenshot evidence).

## new_page

`new_page <url> [--background] [--isolatedContext] [--timeout]`. URL
is positional; toggles are flags.

```bash
# Open a new tab to a URL
cdt new_page https://example.com

# Open in background (don't switch focus)
cdt new_page https://example.com --background

# Open in an isolated browser context (separate cookie jar, storage)
cdt new_page https://example.com --isolatedContext test-run-1

# Set a load timeout (ms)
cdt new_page https://slow-site.example --timeout 10000
```

After `new_page`, the new tab becomes the selected page automatically.

## navigate_page

`navigate_page [--type] [--url] [--ignoreCache] [--handleBeforeUnload]
[--initScript] [--timeout]`. Four navigation modes via `--type`:

```bash
# Default: navigate to a URL
cdt navigate_page --type url --url https://example.com

# Reload current page
cdt navigate_page --type reload
cdt navigate_page --type reload --ignoreCache   # hard reload

# Back / forward
cdt navigate_page --type back
cdt navigate_page --type forward
```

Additional flags:

| Flag | Use |
| --- | --- |
| `--timeout <ms>` | Max wait for navigation to complete |
| `--ignoreCache` | Force re-fetch (hard reload semantics) |
| `--handleBeforeUnload accept\|dismiss` | Auto-respond to "Leave site?" prompts |
| `--initScript "js code"` | Run JS in every new document before page scripts (good for stubbing APIs) |

## close_page

```bash
cdt close_page 1   # positional pageId
```

Will fail with an error if it's the last open page.

## When to read other references

- Element interaction after navigating: `01-elements-and-interaction.md`
- File-writing operations after capturing evidence: `06-screenshots-and-files.md`