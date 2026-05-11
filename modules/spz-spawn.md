# spz-spawn — Spawning Manager

> **Version**: `v2.0.0`  
> **SPiceZ-Core** | Spawning & World Entry  
> Handles the transition from the loading screen to the game world.

## 1. Overview
`spz-spawn` manages how players enter the server. It provides a "Play Menu" (welcome screen) and handles the physical teleportation to spawn points.

## 2. Spawning Lifecycle
The module is designed to wait for `spz-identity` to signal that the player is ready.

### 2.1 The Play Menu
For existing players, `spz-spawn` automatically opens a cinematic menu showing:
- Player Stats (Rank, Tier, Playtime)
- Spawn Point Selection
- Character Preview

### 2.2 New Player Handling
To prevent UI conflicts, `spz-spawn` is "aware" of first-time players:
- **Server-side**: It ignores requests for the Play Menu if `profile.first_time == 1`.
- **Client-side**: It detects the `firstTime` state and:
    1.  Shuts down the loading screen.
    2.  Ensures the world fades in.
    3.  Waits for character creation to complete before requesting the spawn menu.

## 3. Client Events
### `SPZ:showPlayMenu`
Opens the NUI spawn menu with player metadata.
```lua
TriggerEvent("SPZ:showPlayMenu", {
    name = "RacerX",
    rank = "C-5",
    tier = 0,
    gender = 0
})
```

### `SPZ:spawnPlayerTarget`
Performs the physical spawn (model setting, teleport, resurrect).

## 4. Configuration
Spawns are configured in `config.lua`:
```lua
Config.Spawns = {
    [1] = { label = "Safe Zone", coords = vector4(X, Y, Z, W) },
    ...
}
```
