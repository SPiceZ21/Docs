# Player profiles

> Player profiles, licences, crews and vanity plates.

> Part of [spz-core](spz-core.md): this was the separate resource `spz-identity` until it was merged in. Nothing to install on its own.

## Overview

Creates and caches each player's profile on join and exposes it to every other module through exports and statebags. Also owns driver licences (the C–S class letter), the crew backend and personal number plates.

## Features

- Profile created on first join, cached for the session
- Licence class letter (C to S) derived from rank points
- Crew create / join / leave backend (UI is spz-crew)
- Personal vanity plates
- Optional Discord linking

## Configuration

- `spz-core/config_identity.lua (`Config.Identity`, server only)`

See the file for every option.

## Commands

| Command | Who | Description |
|---|---|---|
| `/plate` | player | Set your personal plate |
| `/refreshdiscord` | player | Re-sync your Discord name/avatar (admins: for a player id) |

## Exports

| Export | Side |
|---|---|
| `AttachProfile` | server |
| `CreateCrew` | server |
| `GetClientProfile` | client |
| `GetCrew` | server |
| `GetCrewCooldownSeconds` | server |
| `GetOnlineCrewMembers` | server |
| `GetPlate` | server |
| `GetPlatformName` | server |
| `GetPlaytime` | server |
| `GetProfile` | server |
| `GetRankName` | server |
| `GetSyncSubset` | server |
| `JoinCrew` | server |
| `LeaveCrew` | server |
| `NormalisePlate` | server |
| `PrepareProfile` | server |
| `SetPlate` | server |
| `UnlockLicense` | server |
| `UpdateProfile` | server |

```lua
exports["spz-core"]:AttachProfile(...)
```

---

Source: [https://github.com/SPiceZ21/spz-core](https://github.com/SPiceZ21/spz-core)
