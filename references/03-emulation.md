# Emulation

All emulated features go through the single `emulate` command. Settings
persist for the session until cleared.

## Viewport syntax

`--viewport "<width>x<height>x<devicePixelRatio>[,mobile][,touch][,landscape]"`

```bash
# Mobile iPhone-ish viewport with touch
cdt emulate --viewport "375x812x3,mobile,touch"

# Desktop landscape, high DPR
cdt emulate --viewport "1920x1080x2,landscape"

# Bare dimensions resize without changing device class
cdt emulate --viewport "1024x768"
```

All four segments after the `x` are optional.

## Other emulation features

| Feature | Flag | Values |
| --- | --- | --- |
| Network throttling | `--networkConditions` | `Offline`, `Slow 3G`, `Fast 3G`, `Slow 4G`, `Fast 4G` |
| CPU slowdown | `--cpuThrottlingRate` | integer, 1–20; `1` disables |
| Geolocation | `--geolocation` | `"<latitude>,<longitude>"` (omit to clear) |
| HTTP headers | `--extraHttpHeaders` | JSON object string, e.g. `'{"X-Foo":"bar"}'`; `""` clears |
| User agent | `--userAgent` | string; `""` clears |
| Color scheme | `--colorScheme` | `dark`, `light`, `auto` |

## Clearing settings

Each flag resets when set to an empty string or omitted (per flag). To
reset *all* emulation, restart the daemon (`cdt stop` then the next
`cdt` call).