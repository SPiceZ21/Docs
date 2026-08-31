# spz-core

> Framework bootstrap, sessions, routing buckets, config and schema · `v2.0.0`

## Overview

`spz-core` boots the framework and owns what every other module shares: sessions, routing
buckets, config, permissions and the database schema. It is the first `spz-*` resource in
the load order and the only one that ships SQL.

## Migrations

Every `.sql` in `spz-core/migrations/` is applied once on boot and recorded in
`spz_migrations`, so restarts and upgrades are no-ops. `SPZ:coreReady` waits for this, so
nothing ever queries a half-built database.

Current set: `001` core schema · `002` race columns · `003` module tables · `004` identity
columns · `005` track sectors · `006` racelines · `007` nation and race number · `008`
rivals · `009` duels · `010` leaderboard indexes · `011`–`014` crew image, settings,
invites and rivals · `015` rival events · `016` race number widened to 1–999.

Add changes as a new numbered file. Never edit one that has shipped.

## Sessions

A session is created when a player is deferral-cleared and torn down on disconnect. It is
the handle other modules use for per-player state.

```lua
local session = exports['spz-core']:GetPlayerSession(source)
```

| Export | Purpose |
|---|---|
| `CreateSession` · `GetPlayerSession` · `GetAllSessions` | Session lifecycle |
| `CleanupPlayer` | Force teardown |
| `GetPlayerContext` · `SetPlayerMode` · `IsPlayerInMode` | Per-player context and mode |

## Routing buckets

Races run in isolated buckets so a race world never sees free-roam traffic.

```lua
exports['spz-core']:AssignPlayerToBucket(source, bucketId)
```

| Export | Purpose |
|---|---|
| `CreateBucket` · `DeleteBucket` | Bucket lifecycle |
| `AssignPlayerToBucket` · `RemovePlayerFromBucket` | Membership, with entity cleanup |
| `GetPlayerBucket` · `GetBucketPlayers` · `GetBucketRegistry` | Inspection |
| `SetContextBucket` | Bind a context to a bucket |

## Other exports

| Group | Exports |
|---|---|
| Modules | `RegisterModule` · `RequireModule` · `GetRegisteredModules` · `IsCoreReady` · `WaitForMigrations` · `GetVersion` |
| Config / cache | `GetConfig` · `GetCache` · `SetCache` |
| Permissions | `HasPermission` · `IsAdmin` |
| Environment | `SetSyncedTime` · `SetSyncedWeather` |
| Fade (client) | `FadeIn` · `FadeOut` · `FadeHold` · `FadeTransition` |
| Misc | `RegisterSPZCommand` · `RelayError` |

## Events

| Event | Meaning |
|---|---|
| `SPZ:coreReady` | Core initialised and migrations applied |
| `SPZ:playerConnected` | Session created |
| `SPZ:playerDisconnected` | Session teardown |

## Connection gate

`playerConnecting` is the only moment a player can be held before they exist in the
session, so it is where the profile is loaded. `spz-core` defers the connection and calls
`spz-identity:PrepareProfile(identifier)`; a failure there ends the connection with a real
reason on screen rather than letting the player in to discover there is no profile behind
them.

This previously called `deferrals.done()` immediately, with the DB lookup left to
`spz-identity` — which took a `deferrals` argument the event never passed, so all of its
deferral handling was dead code and nothing gated the connection at all.

If `spz-identity` is not running, the connection is allowed through: a server without
identity is degraded, not closed.

## Loading screen

The loading screen is `loadscreen_manual_shutdown`, so something must take it down. That
owner is `spz-loading`, and it is the only thing that calls `ShutdownLoadingScreen`:

| Export | Purpose |
|---|---|
| `Stage(key, label?)` | Report boot progress — `booting`, `connect`, `profile`, `world`, `ready` |
| `Finish()` | Take the screen down; idempotent |
| `IsFinished()` | Whether it is already down |

The player sees two phases on one bar. The engine drives the first (resource streaming,
0→60%); `Stage` drives the second (60→100%), covering the server handshake and world
streaming. That second phase previously had no signal at all, which is why a stall there
was indistinguishable from a crash — the bar simply sat at 100%.

A watchdog reports which stage stalled and force-closes the screen after 45s. The old
failsafe was a blind timer that killed the screen silently and dropped the player into an
unstreamed world with no explanation.

## Quick-access radial menu

`client/radial.lua` builds a context-aware radial menu on ox_lib (default key
**Z**, remappable in FiveM's keybind settings; `/menu` opens the same ring).

It is a surface over the existing commands — it drives `/joinrace`, `/timetrail`,
`/car`, `/customs` and the rest rather than reaching into each resource, so the
resource that owns a behaviour stays the only place that implements it.

| Ring | Contains |
|---|---|
| Racing | Join / leave race or queue, time trial start / restart / quit, last checkpoint, flip car, standings, spectate, race board |
| Minigames | Hide & Seek, Pursuit, Duel (prompts for a player ID), racing line |
| Vehicle | Spawn, delete, repair, tune, customs, save build, idle cam |
| Appearance | Character editor, save outfit, reset outfit |
| Leaderboard | Full board, last race results |
| Crew | Crew dashboard, crew radio |

Two rules govern what appears:

1. **Resource must be running.** Each item is gated on `GetResourceState`; a
   category with nothing left in it is dropped from the root ring entirely.
2. **Option must be valid now.** Join and Leave share one slot. Recovery and
   standings appear only while a route is running; vehicle spawn, delete, tune
   and customs disappear while one is.

Rule 2 means the menu is rebuilt on `inRace` / `inQueue` / `pendingRace` statebag
changes and on `SPZ:tt:Begin` / `SPZ:tt:End` — no polling. Rebuilding while the
menu is open refreshes it in place, so Join flips to Leave under the cursor the
moment the race starts.

Exported as `RefreshRadial` for resources that change what should be offered.

## Commands

`/spz` · `/status` · `/fix` · `/tpm` · `/menu` · `/time` · `/weather` · `/synctime` · `/syncweather`

## Dependencies

`oxmysql` · `ox_lib`
