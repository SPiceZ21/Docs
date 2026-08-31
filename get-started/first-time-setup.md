# First-time setup

What to configure once the server boots cleanly.

## 1. Core

`spz-core/config.lua` — intermission length, allowed classes, admin identifiers,
default time and weather. ACE permissions for `group.admin` live in `server.cfg` /
`permissions.cfg`.

## 2. Spawn points

`spz-spawn/config.lua`:

```lua
Config.Spawns = {
    [1] = { label = 'Safe Zone', coords = vector4(x, y, z, w) },
}
```

## 3. Races

`spz-races/config.lua` — join window, poll duration, warmup length, finish window,
intermission and spawn mode. Tracks live in `spz-races/data/tracks.lua` (101 shipped: 76
circuit, 25 sprint); build new ones in-game with `/trackcreator`.

Two knobs worth understanding before you touch them:

| Setting | Why it matters |
|---|---|
| `SpawnStaggerMs` (800) + `GridTpSettleMs` (400) | Per-racer cost of building the grid. The whole grid must spawn inside `SpawnTimeout` (30 s) or the session stages while cars are still being created. At 1.2 s per racer a full 16-car grid takes ~19 s. |
| `Rewind.maxCreditPerLapMs` (15 s) | Clock a rewind can win back, **per lap** — a 3-lap circuit allows three times this. Only governs how forgiving the mid-race feel is: any rewound run is already barred from records and personal bests. |

## 4. Vehicles

`spz-vehicles/data/vehicles.lua` is the master table — what exists, which class it belongs
to and what may be unlocked. Class definitions are in `shared/classes.lua`.

## 5. Discord integration

| Resource | Set |
|---|---|
| `spz-log/config.lua` | Webhook URL per category — they ship as placeholders |
| `spz-rpc/config.lua` | `Config.AppId` and the art asset keys from your Discord app |

## 6. Optional modules

| Resource | Configure |
|---|---|
| `spz-spectate/config.lua` | Live race board — rows per view, keys, post-race hold |
| `spz-speedcam/config.lua` | Detection radius, minimum speed, units, blips |
| `spz-physics/config.lua` | Powertrain tuning and PP class bands |
| `spz-fpscap/config.lua` | Frame cap and whether it enforces or only warns |

## 7. Smoke test

1. Connect with a fresh identifier — character creation should appear, then the play menu.
2. Spawn a car (`/car`), check the speedometer and nametags.
3. `/joinrace`, let the poll and countdown run, complete a lap.
4. Confirm results, XP and the leaderboard update afterwards.
