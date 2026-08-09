# Installation

## Prerequisites

| Requirement | Notes |
|---|---|
| FiveM server artifact | Build `27926` or newer |
| MySQL / MariaDB | Any running instance; XAMPP is fine locally |
| FiveM license key | [keymaster.fivem.net](https://keymaster.fivem.net/) |
| `oxmysql` · `ox_lib` · `fivem-appearance` · `pma-voice` | Required dependencies |
| `screenshot-basic` · `screencapture` | Optional — headshots and overtake clips |

## Option A — txAdmin recipe (recommended)

1. txAdmin → **Server Setup** → **Remote URL Template**.
2. Paste:
   ```
   https://raw.githubusercontent.com/SPiceZ21/spz-txrecipe/main/spz-recipe.yaml
   ```
3. Enter your server name, license key and database connection.
4. Let the wizard finish, then start the server.

The recipe downloads every dependency and module and writes `server.cfg` in the correct
order.

> The recipe does not currently download `spz-physics` or `spz-raceline`, although
> `server.cfg` ensures them. Install those two manually or comment out their `ensure`
> lines.

## Option B — manual

Load order matters. A module started before its dependencies will not find their exports.

```cfg
# ── Dependencies ─────────────────────────────
ensure oxmysql
ensure ox_lib
ensure fivem-appearance
ensure pma-voice
ensure screenshot-basic      # optional
ensure screencapture         # optional

# ── Core ─────────────────────────────────────
ensure spz-rpc
ensure spz-loading
ensure spz-core
ensure spz-identity
ensure spz-appearance
ensure spz-spawn

# ── Racing ───────────────────────────────────
ensure spz-speedcam
ensure spz-vehicles
ensure spz-races
ensure spz-progression
ensure spz-nametag
ensure spz-poll
ensure spz-raceUI
ensure spz-leaderboard
ensure spz-carspawner
ensure spz-physics
ensure spz-fpscap
ensure spz-raceline
ensure spz-speedometer
ensure spz-nos
ensure spz-vehfunc
ensure spz-tunners
ensure spz-spectate
ensure spz-betting

# ── Admin (last) ─────────────────────────────
ensure vMenu
```

## Database

Point `oxmysql` at your database. That is the only database step — there are no `.sql`
files to import.

`spz-core` owns the schema: every file in `spz-core/migrations/` is applied on first boot
and recorded in the `spz_migrations` table, so restarts and upgrades are no-ops.
`SPZ:coreReady` does not fire until migrations finish.

To change the schema, add a new numbered file to `spz-core/migrations/`. Never edit one
that has already shipped.

## Building the NUI resources

Resources with a `ui/` directory ship prebuilt `ui/dist/`. Rebuild after changing UI
source:

```bash
cd <resource>/ui && npm install && npm run build
```

## Verify

Start the server and check the console for:

- migrations applied, then `spz-core` reporting ready,
- no `Failed to load resource` lines for `spz-*`,
- a player connect producing a session and reaching the spawn menu.

`/spz` and `/status` in-game report core state.
