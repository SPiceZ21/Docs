# Keybinds

Every default key binding on the server, in one table. **Check this file before
claiming a key**, and update it in the same change that claims one.

This exists because nothing listed what was taken, and four collisions grew out
of that: two commands on `F5`, two on `F7`, the race board on FiveM's console
key, the standings toggle on ox_lib's radial key, the flip-car key on ox_lib's
progress-cancel key, and hazards on GTA's own headlights key. Every one of them
fired both actions at once, which reads as "the key does something weird"
rather than as a conflict.

All of these are **defaults**. Players rebind them in *Settings → Key Bindings*.
Anything that prints a key reads the live binding through spz-core:
`exports['spz-core']:KeyFor(command, default, padButton)` returns the player's
current key (or the controller button while they use a pad), with the default
as the fallback. The [key strip](../modules/spz-hud.md#key-strip) and every
in-race hint use it, so a rebind shows within about a second.

## Claimed

| Key | Action | Resource | Registered in |
| --- | --- | --- | --- |
| `F1` | Admin menu (ace `spz.admin`) | spz-admin | `config.lua` → `Config.MenuKey`, used by `client/menu.lua` |
| `F2` | Speed camera records | spz-races | `config_speedcam.lua` → `Config.Speedcam.RecordsKey` |
| `F3` | Raceline: control panel | spz-raceline | `client/panel.lua` |
| `F4` | Race: back to last checkpoint | spz-races | `config.lua` → `Config.RecoverKey`, used by `client/recover.lua` |
| `F5` | Open car spawner | spz-vehicles | `client/garage.lua` |
| `F6` | Race results / leaderboard | spz-races | `client/ui/leaderboard.lua` |
| `F7` | Open the minigame hub (mapping command `minigames_hub`) | spz-minigames | `config.lua` → `Config.Key` (speed camera records moved to F2) |
| `F9` | Race board: cycle full / mini / hidden | spz-races | `config_spectate.lua` → `Config.Spectate.Board.keys.cycle` |
| `F10` | Toggle player nametags | spz-hud | `config_nametag.lua` → `Config.Nametag.Keybind` |
| `F11` | Race board: hide or restore | spz-races | `config_spectate.lua` → `Config.Spectate.Board.keys.hide` |
| `TAB` | Open the SPiceZ hub (profile, standings, crew, achievements…) — takes GTA's weapon-wheel key | spz-hud | `client/hub.lua` |
| `T` | Open chat | spz-hud | `config_chat.lua` → `Config.Chat.Keybind` |
| `B` | Race: rewind time (hold) | spz-races | `config.lua` → `Config.Rewind.key` |
| `K` | Race: flip car upright | spz-races | `client/recover.lua` |
| `Y` | Race: tune car during warmup (only while in a race warmup) | spz-races | `client/nui_bridge.lua` |
| `O` | Open the emote menu | spz-hud | `config_emotes.lua` → `Config.Emotes.MenuKey` |
| `N` | Race: toggle standings list | spz-races | `client/ui/raceui.lua` |
| `J` | Hazard lights | spz-vehicles | `client/vehfunc/main.lua` |
| `L` | Flash headlights (hold) | spz-vehicles | `client/vehfunc/main.lua` |
| `G` | Horn taunt | spz-vehicles | `client/vehfunc/taunts.lua` |
| `←` / `→` | Left / right indicator | spz-vehicles | `client/vehfunc/main.lua` |
| `BACKSPACE` | Time trial: restart to start | spz-races | `client/timetrail.lua` |

## Claimed by ox_lib

Not ours, and not to be taken. Rebinding them means editing a vendored library,
which is undone by the next ox_lib update.

| Key | Action |
| --- | --- |
| `Z` | Radial menu (carries the raceline / ghost controls); shown on the key strip in every state |
| `X` | Cancel the progress bar — also stops an emote (spz-hud `Config.Emotes.CancelKey`); the two never apply at once |

## Reserved by FiveM / GTA

| Key | Owner |
| --- | --- |
| `F8` | FiveM console |
| `H` | GTA: headlights |
| `E` | GTA: horn |
| `ESC` | Pause menu |

## Registered with no default

Deliberately unbound — the command works, and a player who wants the key binds
it themselves. Reach for this list before inventing a new default.

| Command | Action | Resource |
| --- | --- | --- |
| `crew` | Open crew dashboard | spz-crew |
| `crewradio` | Toggle crew radio | spz-crew |
| `pursuitmenu` | Hot Pursuit queue menu | spz-pursuit |
| `spectate` | Toggle spectator mode | spz-races |
| `srace` | Admin: start a race with a chosen track + car (ox_lib menu, skips the poll) | spz-races |
| `replays` | Race replay browser (while watching: SPACE play, arrows seek/speed, Q/E driver, C camera, H hide UI, B Rockstar Editor rec/stop, G open Rockstar Editor, TAB mouse, BACKSPACE exit) | spz-races |
| `racelinetoggle` | Raceline: toggle display | spz-raceline |
| `noclip` | Admin: toggle noclip | spz-admin |

## Free

`M`, `F12`, `C`, `,`, `.`

## Mode-only keys

Only active inside one tool or mode, so they may reuse a key that is taken
elsewhere — but never one that would fire the other action at the same time.

| Key | Action | Resource |
| --- | --- | --- |
| `ENTER` / `SHIFT+ENTER` / `DEL` / `↑↓←→` / `PGUP` `PGDN` / `F9` / `F10` / `ESC` | Track Studio: drop gate / save / remove last / shape gate / laps / circuit-sprint / controls / discard | spz-races `client/trackstudio.lua` |
| `SPACE` `←` `→` `↑` `↓` `Q` `E` `C` `H` `B` `G` `TAB` `BACKSPACE` | Replay while watching (all game input disabled) | spz-replay `client/main.lua` |
| `E` | Color Rush / Bomb Tag: car jump (once per wave) | spz-colorrush, spz-bomb |
| `LSHIFT` | Bomb Tag: boost (once per wave) | spz-bomb |

Controller defaults (`PAD_DIGITALBUTTON`) are registered next to most of these;
see the resource for the button.

## Not in this registry

Roughly forty call sites poll controls directly with
`IsControlJustPressed` / `IsDisabledControlJustPressed` (menu navigation,
editor and creator hotkeys, spectator camera). Those bypass
`RegisterKeyMapping` entirely: they cannot be rebound, they do not appear in
FiveM's key settings, and they are checked against GTA control IDs rather than
key names. They are not tracked here yet.
