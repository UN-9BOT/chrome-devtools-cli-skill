# Elements & Interaction

UIDs and interaction semantics for `click`, `fill`, `fill_form`, `drag`,
`hover`, `type_text`, `upload_file`, `press_key`, `wait_for`.

## UID format

`<pageNum>_<elementNum>` — e.g., `1_3`, `2_45`.

- `pageNum` = the page's index from `list_pages` (1-based).
- `elementNum` = the element's index within that snapshot (1-based).
- After `new_page` / `close_page`, the prefix can shift. **Always re-snapshot.**

## Required pre-action step

```bash
cdt take_snapshot
```

UIDs are page-scoped, tied to the snapshot that produced them, and
expire on any navigation or DOM mutation. Stale UIDs return "element
not found".

Options:

- `--verbose` — include more a11y attributes (useful when refs are ambiguous).
- `--filePath /tmp/snap.txt` — save the snapshot to a file instead of
  inline (dense pages can produce hundreds of lines).

## `type_text` submit key

`type_text` can press a key after typing — useful for forms submitted by
Enter:

```bash
cdt type_text "search query" --submitKey Enter
cdt type_text "user@host" --submitKey Tab
```

## Interaction commands

CLI shape: identifiers are **positional**; toggle flags are camelCase.

| Goal | Command | Notes |
| --- | --- | --- |
| Click an element | `click <uid>` | Add `--includeSnapshot` to get the post-click snapshot inline (saves a round-trip). |
| Hover an element | `hover <uid>` | Useful for triggering CSS-driven dropdowns. |
| Type into an input | `fill <uid> <value>` | Replaces the field's value. |
| Fill many fields at once | `fill_form '[{"uid":"1_2","value":"a"},{"uid":"1_3","value":"b"}]'` | One JSON array; much faster than per-field `fill`. |
| Drag-and-drop | `drag <from_uid> <to_uid>` | Both UIDs from the **same** snapshot. Add `--includeSnapshot` to get the post-drag snapshot. |
| Type text after focusing | `type_text <text>` | Fires `input`/`keydown`/`keyup` events. Add `--submitKey Enter` to press a key when done. |
| Press a key combination | `press_key <key>` | Triggers real browser events. Modifiers: `Control`, `Shift`, `Alt`, `Meta`. |
| Upload a file | `upload_file <uid> <filePath>` | `uid` is typically an `<input type="file">`. |
| Wait for content | `wait_for '["loaded","success"]'` | Returns when **any** string appears. JSON array. |

## When to read other references

- `evaluate_script` for dialog handling or returning DOM data: see
  `02-scripting.md`.
- Saving screenshots/element captures: see `06-screenshots-and-files.md`.
- Memory/extension/PWA/experimental tools (gated by server flags):
  `08-advanced-features.md`.