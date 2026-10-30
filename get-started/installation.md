# Installation

## Requirements

| Requirement | Notes |
|---|---|
| FXServer artifact | Build `27926` or newer |
| Game build | `3407` (`sv_enforceGameBuild 3407`) |
| MySQL / MariaDB | Any running instance |
| FiveM licence key | [keymaster.fivem.net](https://keymaster.fivem.net/) |

## Option A — txAdmin recipe (recommended)

1. txAdmin → **Server Setup** → **Remote URL Template**.
2. Paste:
   ```
   https://raw.githubusercontent.com/SPiceZ21/spz-txrecipe/main/spz-recipe.yaml
   ```
3. Enter the server name, licence key and database connection.
4. Let it finish, then start the server.

The recipe downloads every dependency and module and writes `server.cfg` in the right
order. Nothing else is needed — the database schema builds itself on first boot.

## Option B — manual

Download each module from [github.com/SPiceZ21](https://github.com/SPiceZ21) into
`resources/[spz]/`. Load order matters: a module started before its dependencies will not
find their exports.

```cfg
# Dependencies
ensure oxmysql
ensure ox_lib
ensure fivem-appearance
ensure pma-voice
ensure screenshot-basic
ensure screencapture
ensure bob74_ipl

# Core
ensure spz-rpc
ensure spz-loading
ensure spz-core
ensure spz-identity
ensure spz-chat
ensure spz-pausemenu
ensure spz-appearance
ensure spz-emotes
ensure spz-spawn
ensure spz-admin

# Racing
ensure spz-speedcam
ensure spz-vehicles
ensure spz-races
ensure spz-progression
ensure spz-nametag
ensure spz-poll
ensure spz-raceUI
ensure spz-leaderboard
ensure spz-carspawner
ensure spz-fpscap
ensure spz-raceline
ensure spz-speedometer
ensure spz-vehfunc
ensure spz-tunners
ensure spz-spectate
ensure spz-replay
ensure spz-analytics
ensure spz-crew
ensure spz-discord

# Minigames (hub first)
ensure spz-minigames
ensure spz-colorrush
ensure spz-cyclone
# ensure iak_demolition   # third-party arena map, needed by the two below
ensure spz-bomb
ensure spz-demolition
ensure spz-pursuit
```

`fivem-appearance` needs its thumbnail pack (`images/`) next to its `fxmanifest.lua`; the
recipe adds it automatically.

## Database

Point `oxmysql` at your database (`set mysql_connection_string ...`). That is the only
database step — there are no `.sql` files to import. `spz-core` applies every file in
`spz-core/migrations/` on first boot and records it in `spz_migrations`.

## Verify

Start the server and check the console for:

- migrations applied, then `spz-core` reporting ready,
- no `Failed to load resource` lines for `spz-*`,
- a player connecting reaching character creation, then the play menu.
