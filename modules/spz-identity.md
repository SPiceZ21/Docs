# spz-identity

> Player profiles, citizen IDs, licenses, ranks, crews · `v1.5.0`

## Overview

The player data layer. It resolves a connecting player to a profile, runs first-time
character creation, and owns usernames, citizen IDs, license tiers, ranks, ratings and
crews. Other modules read player data through its exports, never from the `players` table
directly.

## Connection and creation flow

1. `spz-core` fires `SPZ:playerConnected`.
2. Identity looks the player up by license.
   - **Existing profile** with `first_time = 0` → cache warmed, `SPZ:playerReady` fires.
   - **No profile** → `CreateProfile` with `first_time = 1`.
3. When `first_time == 1` the server fires `SPZ:openCharacterCreation` on the client and
   **withholds** `SPZ:playerReady`.
4. The player picks a gender, a username, a nation flag and a race number (1–999). Username
   and race number are both validated for uniqueness server-side.
5. `fivem-appearance` then opens for face, hair and clothing. Only once it closes does the
   client ask for its spawn route — see [spz-spawn](spz-spawn.md).
5. The server stores them, clears `first_time`, then fires `SPZ:characterReady` and
   `SPZ:playerReady`.

Flag and race number appear on nametags and in the standings tower; flags ship as local
assets rather than being fetched from a CDN.

Modules that need player data must wait for `SPZ:playerReady`, not `SPZ:playerConnected`.

## Exports

| Group | Exports |
|---|---|
| Profile | `GetProfile` · `CreateProfile` · `UpdateProfile` · `SaveProfile` · `GetClientProfile` · `GetSyncSubset` · `SetPlayerState` |
| Identity | `GetCitizenId` · `GetByCitizenId` · `GetUsername` · `GetPlatformName` · `GetPlaytime` |
| Licenses | `HasLicense` · `GetLicenseTier` · `UnlockLicense` · `GetLicenseHistory` |
| Ranks | `GetRankName` |
| Crews | `CreateCrew` · `JoinCrew` · `LeaveCrew` · `GetCrew` · `GetCrewTag` · `GetOnlineCrewMembers` · `GetCrewCooldownSeconds` |
| Admin | `BanPlayer` |

```lua
local profile = exports['spz-identity']:GetProfile(source)
local tier    = exports['spz-identity']:GetLicenseTier(source)
```

## Events

| Event | Side | Meaning |
|---|---|---|
| `SPZ:openCharacterCreation` | Client | New player — open the creation form |
| `SPZ:characterCreated` | Server | NUI submitted gender and username |
| `SPZ:characterReady` | Server | Profile initialised |
| `SPZ:playerReady` | Server | Player data is safe to read |

## Schema

Owned by `spz-core/migrations/`. Key `players` columns:

| Column | Type | Default | Meaning |
|---|---|---|---|
| `id` | INT | auto | Primary key |
| `username` | VARCHAR | NULL | Unique racer name |
| `gender` | INT | NULL | 0 = male, 1 = female |
| `first_time` | INT | 1 | Character creation pending |
| `license_tier` | INT | 0 | 0 = C, 1 = B, 2 = A, 3 = S |

## Dependencies

`ox_lib` · `spz-core` · `oxmysql`
