# spz-spawn

> Play menu, spawn points, world entry · `v2.2.0`

## Overview

`spz-spawn` moves the player from the loading screen into the world. Returning players see
a cinematic play menu with their stats and a spawn-point picker; first-time players go
through character creation first.

## Boot flow

The flow is **pull-based**: the client asks, the server answers. This matters more than it
sounds. At `playerConnecting` and `playerJoining` the client is still on the loading screen
with none of its scripts running, so anything the server sends it at that point is
discarded — which is exactly what the previous push-based version did.

```
playerConnecting        spz-core defers the connection
                        └─ spz-identity:PrepareProfile(identifier)
                           DB only: look up or create, check the ban.
                           No statebags (no Player yet), no client events (no client yet).
                           Failure here shows a real reason on the connection screen.

playerJoining           spz-core creates the session, fires SPZ:playerConnected
                        (server-side listeners only)

client scripts start    spz-spawn sends SPZ:spawn:hello
                        └─ retried with backoff, 8 attempts over ~60s

server                  spz-identity:AttachProfile(source)
                        binds the prepared profile, writes statebags, syncs,
                        fires SPZ:playerReady
                        └─ replies SPZ:spawn:route { mode = "create" | "menu" | "error" }

client                  acts on the route, streams the world behind a cover,
                        then hands off from the loading screen to the menu
```

One request, one reply. No polling, and a failure surfaces as a message rather than an
indefinite wait.

### First-time players

`SPZ:playerReady` means *this player has a usable identity*, and seven resources act on it.
A first-time player has no username until character creation finishes, so the handshake
deliberately withholds the event for them — `spz-identity` fires it once, from the
character-creation completion path, instead of firing a half-built profile at everyone and
then firing again.

After the appearance editor closes the client calls `SPZ:spawn:requestMenu`, and the same
routing decision now answers `menu` because `first_time` is `0`.

## Loading screen handoff

`spz-loading` owns `ShutdownLoadingScreen` (see [spz-core → loading](spz-core.md)). This
module never calls it directly; it reports stages and says when the handoff is safe:

```lua
exports['spz-loading']:Stage('world')
exports['spz-loading']:Finish()
```

The screen comes down only once the branded cover has painted **and** collision has
streamed at the preview location — so the player goes from one full-screen cover to
another and never sees raw world streaming.

## Events

| Event | Direction | Purpose |
|---|---|---|
| `SPZ:spawn:hello` | client → server | Client is running; asking for a route |
| `SPZ:spawn:route` | server → client | The decision: `create`, `menu`, or `error` |
| `SPZ:spawn:requestMenu` | client → server | Re-ask after character creation |
| `SPZ:requestSpawn` | client → server | Player picked a spawn point |
| `SPZ:showPlayMenu` | local | Open the menu with player metadata |
| `SPZ:spawnPlayerTarget` | server → client | Perform the physical spawn — model, teleport, resurrect |

`SPZ:requestPlayMenu` is kept as an alias for `SPZ:spawn:requestMenu` so older callers
keep working; both route through the same decision.

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
