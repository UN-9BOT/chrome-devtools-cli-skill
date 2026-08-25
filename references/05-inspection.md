# Inspection (Console & Network)

`list_console_messages`, `list_network_requests`, `get_network_request`.

## Console messages

```bash
# Just errors
cdt list_console_messages --types '["error"]'

# Errors + warnings across the last 3 navigations
cdt list_console_messages --types '["error","warn"]' --includePreservedMessages
```

Available types: `log`, `debug`, `info`, `error`, `warn`, `dir`, `table`,
`trace`, `assert`, `count`, `timeEnd`, `issue`, `verbose`. JSON array.

## Network requests

`list_network_requests` shows requests since the **last navigation**. On
noisy pages, paginate or filter.

```bash
# First 20 requests
cdt list_network_requests --pageSize 20

# Across the last 3 navigations
cdt list_network_requests --includePreservedRequests
```

To read request/response bodies, get the request id from the list and
fetch it (positional arg):

```bash
cdt get_network_request <reqid>
```

## When to use

Use both for debugging rendering issues, XHR/fetch failures, and CORS
errors. Console messages surface JS exceptions; network requests
surface 4xx/5xx and timing.