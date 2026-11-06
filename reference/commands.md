# Commands

Every chat command in the beta. **Who:** `player` — anyone; `admin` — needs the module's ACE / admin; `dev` — diagnostics.

## Player

| Command | Module | Description |
|---|---|---|
| `/appearance` | [spz-core](../modules/spz-appearance.md) | Open the character editor |
| `/car` | [spz-vehicles](../modules/spz-vehicles.md) | Open the spawner, or `/car <model>` |
| `/crew` | [spz-crew](../modules/spz-crew.md) | Open the crew dashboard |
| `/crewradio` | [spz-crew](../modules/spz-crew.md) | Toggle crew radio |
| `/hudedit` | [spz-hud](../modules/spz-hud.md) | Move, resize and hide HUD elements; HUD settings (speedometer style) |
| `/customs` | [spz-vehicles](../modules/spz-vehicles.md) | Open customs |
| `/discardclip` | [spz-core](../modules/spz-core.md) | Stop recording without saving |
| `/duel` | [spz-races](../modules/spz-races.md) | `/duel <player> <track> <stake>` — challenge a player |
| `/dv` | [spz-vehicles](../modules/spz-vehicles.md) | Delete your vehicle |
| `/e` | [spz-hud](../modules/spz-emotes.md) | `/e <name>` play an emote, `/e` stop |
| `/editor` | [spz-core](../modules/spz-core.md) | Open the Rockstar Editor |
| `/emotes` | [spz-hud](../modules/spz-emotes.md) | Open the menu |
| `/fix` | [spz-core](../modules/spz-core.md) | Repair the vehicle you are in |
| `/help` | [spz-hud](../modules/spz-chat.md) | List every server feature and its key |
| `/idlecam` | [spz-vehicles](../modules/spz-vehicles.md) | Toggle the idle camera |
| `/joinrace` | [spz-races](../modules/spz-races.md) | Join the race queue (also `E` in freeroam) |
| `/leaderboard` | [spz-races](../modules/spz-races.md) | Open the leaderboard |
| `/leaverace` | [spz-races](../modules/spz-races.md) | Leave the queue / race |
| `/menu` | [spz-core](../modules/spz-core.md) | Open the radial menu |
| `/plate` | [spz-core](../modules/spz-identity.md) | Set your personal plate |
| `/pursuit` | [spz-pursuit](../modules/spz-pursuit.md) | Open the Hot Pursuit room menu |
| `/quittt` | [spz-races](../modules/spz-races.md) | Quit the time trial |
| `/raceboard` | [spz-races](../modules/spz-spectate.md) | Cycle the race board |
| `/raceboard_hide` | [spz-races](../modules/spz-spectate.md) | Hide / restore the race board |
| `/raceline` | [spz-raceline](../modules/spz-raceline.md) | Raceline options |
| `/racelinepanel` | [spz-raceline](../modules/spz-raceline.md) | Open the raceline panel (`F3`) |
| `/racelinetoggle` | [spz-raceline](../modules/spz-raceline.md) | Show / hide the line |
| `/raceresults` | [spz-races](../modules/spz-races.md) | Open the latest race results |
| `/rank` | [spz-races](../modules/spz-progression.md) | Show your rank |
| `/record` | [spz-core](../modules/spz-core.md) | Start recording a Rockstar Editor clip (again: stop and save) |
| `/refreshdiscord` | [spz-core](../modules/spz-identity.md) | Re-sync your Discord name/avatar (admins: for a player id) |
| `/resetcustom` | [spz-vehicles](../modules/spz-vehicles.md) | Reset it |
| `/resetoutfit` | [spz-core](../modules/spz-appearance.md) | Drop your saved outfit (falls back to crew/default) |
| `/rival` | [spz-races](../modules/spz-progression.md) | Show your rival and the gap on your last track |
| `/save_customs` | [spz-vehicles](../modules/spz-vehicles.md) | Save the current setup |
| `/savecustom` | [spz-vehicles](../modules/spz-vehicles.md) | Save the current car's customisation |
| `/saveoutfit` | [spz-core](../modules/spz-appearance.md) | Save your current outfit |
| `/spectate` | [spz-races](../modules/spz-spectate.md) | Toggle spectator mode |
| `/srace` | [spz-races](../modules/spz-races.md) | Start a quick solo race |
| `/stoprecord` | [spz-core](../modules/spz-core.md) | Stop and save the clip |
| `/time` | [spz-core](../modules/spz-core.md) | Local time preview |
| `/timetrail` | [spz-races](../modules/spz-races.md) | Start a time trial |
| `/tune` | [spz-vehicles](../modules/spz-vehicles.md) | Open the tuner |
| `/weather` | [spz-core](../modules/spz-core.md) | Local weather preview |

## Admin

| Command | Module | Description |
|---|---|---|
| `/admin` | [spz-admin](../modules/spz-admin.md) | Open the admin menu |
| `/clearstart` | [spz-races](../modules/spz-races.md) | Remove a surveyed start line |
| `/dc_leaderboard` | [spz-discord](../modules/spz-discord.md) | Post the leaderboard to Discord |
| `/freezetime` | [spz-core](../modules/spz-core.md) | Freeze / unfreeze server time |
| `/noclip` | [spz-admin](../modules/spz-admin.md) | Toggle noclip |
| `/racehud` | [spz-races](../modules/spz-races.md) | Re-read race HUD convars |
| `/reclassify` | [spz-vehicles](../modules/spz-vehicles.md) | Recompute car classes |
| `/revive` | [spz-admin](../modules/spz-admin.md) | `/revive [id]` |
| `/ride` | [spz-admin](../modules/spz-admin.md) | `/ride <id>` ride along |
| `/setstart` | [spz-races](../modules/spz-races.md) | Survey a start line for a track |
| `/spec` | [spz-admin](../modules/spz-admin.md) | `/spec <id>` spectate a player |
| `/spz` | [spz-core](../modules/spz-core.md) | `/spz <sub>` — framework diagnostics and hot reload |
| `/spzaddons` | [spz-vehicles](../modules/spz-vehicles.md) | List add-on cars |
| `/spzaddonscan` | [spz-vehicles](../modules/spz-vehicles.md) | Scan for add-on cars |
| `/spzbuckets` | [spz-core](../modules/spz-core.md) | List routing buckets and who is in them |
| `/startlines` | [spz-races](../modules/spz-races.md) | List surveyed start lines |
| `/synctime` | [spz-core](../modules/spz-core.md) | Set server time |
| `/syncweather` | [spz-core](../modules/spz-core.md) | Set server weather |
| `/tpm` | [spz-core](../modules/spz-core.md) | Teleport to your waypoint, or to a player id |
| `/vehcheck` | [spz-vehicles](../modules/spz-vehicles.md) | Inspect the car you are in |

## Dev

| Command | Module | Description |
|---|---|---|
| `/carcode` | [spz-admin](../modules/spz-admin.md) | Copy your car's spawn code |
| `/checkgateprops` | [spz-races](../modules/spz-races.md) | Check gate prop models resolve |
| `/copdebug` | [spz-races](../modules/spz-races.md) | Cop-chase debug overlay |
| `/cpdebug` | [spz-races](../modules/spz-races.md) | Draw checkpoint gates in 3D |
| `/dmwhere` | [spz-demolition](../modules/spz-demolition.md) | Print arena position info |
| `/fixheadings` | [spz-races](../modules/spz-races.md) | Recompute gate headings |
| `/flagdrop` | [spz-races](../modules/spz-races.md) | Preview the flag girl animation |
| `/ghostinfo` | [spz-raceline](../modules/spz-raceline.md) | Ghost replay diagnostics |
| `/heading` | [spz-admin](../modules/spz-admin.md) | Copy your heading |
| `/keydebug` | [spz-core](../modules/spz-core.md) | Show what GTA reports for a key binding |
| `/mgkey` | [spz-minigames](../modules/spz-minigames.md) | Show the hub key binding |
| `/phasing` | [spz-core](../modules/spz-core.md) | Print collision/phasing state |
| `/racesoundtest` | [spz-races](../modules/spz-races.md) | Play the grid sounds |
| `/rankuptest` | [spz-races](../modules/spz-races.md) | Preview the rank-up animation |
| `/rlstatus` | [spz-raceline](../modules/spz-raceline.md) | Recording / storage status |
| `/syncnametags` | [spz-hud](../modules/spz-nametag.md) | Force a nametag resync |
| `/vec3` | [spz-admin](../modules/spz-admin.md) | Copy your coords |
| `/vec4` | [spz-admin](../modules/spz-admin.md) | Copy your coords + heading |
| `/wantedtest` | [spz-races](../modules/spz-races.md) | Preview the wanted HUD |
