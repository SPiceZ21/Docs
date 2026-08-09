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
rivals · `009` duels.

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

## Commands

`/spz` · `/status` · `/fix` · `/tpm` · `/time` · `/weather` · `/synctime` · `/syncweather`

## Dependencies

`oxmysql` · `ox_lib`
