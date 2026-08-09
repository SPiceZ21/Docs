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

**Server decides.** Clients report checkpoint hits, positions and speed; the server
validates them and owns the outcome. No client input decides a race result.

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

## Key events

| Event | Fired by | Meaning |
|---|---|---|
| `SPZ:coreReady` | spz-core | Framework up, migrations applied |
| `SPZ:playerConnected` | spz-core | Session created |
| `SPZ:openCharacterCreation` | spz-identity | New player needs a character |
| `SPZ:characterReady` | spz-identity | Profile initialised |
| `SPZ:playerReady` | spz-identity | Player data is safe to read |
| `SPZ:playerDisconnected` | spz-core | Session teardown |
