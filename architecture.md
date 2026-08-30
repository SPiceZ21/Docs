# Architecture

## Layers

```
oxmysql · ox_lib · fivem-appearance          third-party dependencies
        │
     spz-core                                sessions · buckets · migrations · config
        │
   ┌────┴─────┬──────────────┐
spz-identity  spz-appearance  spz-spawn      player layer
        │
   spz-vehicles                              vehicle registry
        │
    spz-races                                race engine (server-authoritative)
        │
   ┌────┴──────┬──────────────┬────────────┐
spz-progression spz-raceUI  spz-leaderboard  spz-poll ...
```

Load order follows this diagram. A module started before its dependencies will not find
their exports.

## Rules

**One schema owner.** All SQL lives in `spz-core/migrations/`. Each file applies once and
is recorded in `spz_migrations`, so restarts and upgrades are no-ops. A module never
creates or alters another module's tables — schema changes go in a new numbered migration.

**Exports for reads, events for signals.** Need a player's profile? Call
`exports['spz-identity']:GetProfile(source)`. Need to react to something happening? Listen
for the event.

**Event names are constants, not literals.** `SPZ.Events` (`spz-core/shared/events.lua`) is
the registry; `spz-races/shared/events.lua` merges the race names into the same table. A
resource that needs them imports `'@spz-core/shared/events.lua'` in its `shared_scripts` —
resources have separate Lua states, so the table does not cross a resource boundary on its
own. A mistyped literal is silent: the handler simply never fires.

**One emitter per event.** An event that drives scoring, persistence or payouts has exactly
one place it may be fired from, and that is documented where it is defined. Anything that
wants to react earlier gets its own name — `SPZ:racerFinished` exists precisely so nothing
needs to borrow `SPZ:raceEnd`.

**Server decides.** A client may *propose* an event; the server accepts it only if it can
falsify it from data the server owns. Checkpoint hits are checked against the gate the
racer was due to cross and their position relative to it; incident reports are speed-sanity
checked and capped per race; rewind claims are clamped per claim, per lap and against the
history buffer. The client owns timing precision — only it has the frame — but no client
input decides a race result.

**Config-driven.** Every tunable lives in the module's `config.lua`. Core config is synced
to clients by `spz-core/client/config_sync.lua`.

## Boot sequence

1. `spz-core` starts, applies `migrations/*.sql`, then fires `SPZ:coreReady`.
2. Modules register themselves with the core registry and wait for that event.
3. A player connects → `spz-core` creates a session → `SPZ:playerConnected`.
4. `spz-identity` resolves or creates the profile, then fires `SPZ:playerReady` (or
   `SPZ:openCharacterCreation` first, for new players).
5. `spz-spawn` shows the play menu and puts the player in the world.

## Routing buckets

Races run in isolated routing buckets so a race world never sees free-roam traffic.
`spz-core/server/buckets.lua` owns creation, assignment and cleanup; `spz-races` requests
a bucket per race, and `spz-spectate` moves a viewer into the target's bucket so isolated
racers stay visible.

A bucket is released on **both** exits: the normal one (`RunRaceCleanup`) and the abort
path (`ResetToIdle` — empty queue, dead poll, nobody spawned). Any code path that can end a
race owes the bucket back; leaking one strands its players and the registry entry lives for
the server's whole uptime.

## Key events

| Event | Fired by | Meaning |
|---|---|---|
| `SPZ:coreReady` | spz-core | Framework up, migrations applied |
| `SPZ:playerConnected` | spz-core | Session created |
| `SPZ:openCharacterCreation` | spz-identity | New player needs a character |
| `SPZ:characterReady` | spz-identity | Profile initialised |
| `SPZ:playerReady` | spz-identity | Player data is safe to read |
| `SPZ:playerDisconnected` | spz-core | Session teardown |
| `SPZ:raceStateChanged` | spz-races | Race lifecycle transition (mirrors `GlobalState.raceState`) |
| `SPZ:racerFinished` | spz-races | One racer crossed the line — reactive use only, field incomplete |
| `SPZ:raceEnd` | spz-races | Race over, results final. Scoring and persistence hang off this |
| `SPZ:standings` | spz-races | Live running order for out-of-race consumers (betting, spectator boards) |

`spz-races` also drives lifecycle through `GlobalState.raceState`, which every client reads
for free. Statebag handlers fire on every *set*, not only on a change, so the transition
handler is one-shot guarded — re-asserting the current value must never replay a phase.
