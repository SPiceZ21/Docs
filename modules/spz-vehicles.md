# spz-vehicles

> Vehicle registry, classes, spawning, upgrades, customization · `v2.0.0`

## Overview

`spz-vehicles` decides what a player may drive. It owns the master vehicle table and class
definitions, spawns cars for free roam and race grids, validates and persists upgrades,
stores cosmetic setups, and builds the vehicle pool that `spz-poll` votes on.

## Data

| File | Contents |
|---|---|
| `data/vehicles.lua` | Master vehicle table — model, class, availability |
| `shared/classes.lua` | Class definitions and metadata |
| `shared/upgrades.lua` | Upgrade tiers and slots |

Class balancing can also use PP values from
[spz-physics](spz-physics.md) (`GetModelPP`).

## Exports

| Group | Exports |
|---|---|
| Registry | `GetVehicleRegistry` · `GetVehicleData` · `IsRegistered` · `GetClassMeta` · `GetClassVehicles` · `GetRaceClasses` |
| Spawning | `SpawnVehicle` · `FreeroamSpawn` · `SpawnRaceVehicle` · `DespawnVehicle` · `GetPlayerVehicle` · `GetFreeroamVehicles` |
| Poll pool | `GetPollPool` · `GetAllPollOptions` |
| Customization | `LoadCustomization` · `ResetCustomization` |
| Unlocks | `UnlockRaceVehicle` |

```lua
local data = exports['spz-vehicles']:GetVehicleData('banshee')
exports['spz-vehicles']:SpawnRaceVehicle(source, model, gridSlot)
```

## Spawning

Free-roam and race spawns are separate paths. Race spawns are grid-aware and are only
issued by `spz-races`; both are server-authoritative, so a client cannot conjure a vehicle
it has not unlocked.

## Commands

`/savecustom` · `/resetcustom`

## Dependencies

`ox_lib` · `spz-core` · `spz-identity` · `oxmysql`
