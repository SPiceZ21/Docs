# spz-spawn

> Play menu, spawn points, world entry · `v2.1.0`

## Overview

`spz-spawn` moves the player from the loading screen into the world. Returning players see
a cinematic play menu with their stats and a spawn-point picker; first-time players go
through character creation first.

## Lifecycle

The module waits for `spz-identity` to signal readiness before doing anything.

**Returning player** — the menu opens with rank, tier, playtime, a character preview and
the spawn-point list. Choosing a point performs the spawn.

**First-time player** — to stop two UIs fighting:

- *Server side*: play-menu requests are ignored while `profile.first_time == 1`.
- *Client side*: the loading screen is shut down, the world faded in, and the menu is only
  requested once character creation completes.

## Events

| Event | Purpose |
|---|---|
| `SPZ:showPlayMenu` | Open the menu with player metadata |
| `SPZ:spawnPlayerTarget` | Perform the physical spawn — model, teleport, resurrect |

```lua
TriggerEvent('SPZ:showPlayMenu', {
    name   = 'RacerX',
    rank   = 'C-5',
    tier   = 0,
    gender = 0,
})
```

## Configuration

`spz-spawn/config.lua`:

```lua
Config.Spawns = {
    [1] = { label = 'Safe Zone', coords = vector4(x, y, z, w) },
}
```

## Commands

`/testspawn` · `/testcreation` — development helpers.

## Dependencies

`spawnmanager` · `ox_lib` · `spz-core` · `spz-identity`
