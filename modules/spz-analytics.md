# Analytics

> Analytics collector for Grafana.

> Part of [spz-core](spz-core.md): this was the separate resource `spz-analytics` until it was merged in. Nothing to install on its own.

## Overview

Writes sessions, activity per mode, the race funnel, laps, vehicle and feature usage, daily stats and per-minute server snapshots to the database in batches, ready for Grafana dashboards.

## Features

- Batched writes
- Per-minute server snapshots
- Race funnel and feature usage

## Configuration

- `spz-core/config_analytics.lua (`Config.Analytics`)`

See the file for every option.

## Exports

| Export | Side |
|---|---|
| `AdminAction` | server |
| `Event` | server |
| `Reject` | server |
| `Track` | server |

```lua
exports["spz-core"]:AdminAction(...)
```

---

Source: [https://github.com/SPiceZ21/spz-core](https://github.com/SPiceZ21/spz-core)
