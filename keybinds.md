# Keybind registry

Every default key binding on the server, in one table. **Check this file before
claiming a key**, and update it in the same change that claims one.

This exists because nothing listed what was taken, and four collisions grew out
of that: two commands on `F5`, two on `F7`, the race board on FiveM's console
key, the standings toggle on ox_lib's radial key, the flip-car key on ox_lib's
progress-cancel key, and hazards on GTA's own headlights key. Every one of them
fired both actions at once, which reads as "the key does something weird"
rather than as a conflict.

All of these are **defaults**. Players rebind them in *Settings → Key Bindings*,
and FiveM offers no way to read a live binding back — which is why any HUD that
prints a key reads the same constant the mapping was registered with, rather
than guessing.

## Claimed

| Key | Action | Resource | Registered in |
| --- | --- | --- | --- |
| `F3` | Raceline: control panel | spz-raceline | `client/panel.lua` |
| `F4` | Race: back to last checkpoint | spz-races | `config.lua` → `Config.RecoverKey`, used by `client/recover.lua` |
| `F5` | Open car spawner | spz-carspawner | `client/main.lua` |
| `F6` | Race results / leaderboard | spz-leaderboard | `client/main.lua` |
| `F7` | Speed camera records | spz-speedcam | `client/main.lua` |
| `F9` | Race board: cycle full / mini / hidden | spz-spectate | `config.lua` → `Config.Board.keys.cycle` |
| `F10` | Toggle player nametags | spz-nametag | `config.lua` → `Config.Keybind` |
| `F11` | Race board: hide or restore | spz-spectate | `config.lua` → `Config.Board.keys.hide` |
| `T` | Open chat | spz-chat | `config.lua` → `Config.Keybind` |
| `B` | Race: rewind time (hold) | spz-races | `config.lua` → `Config.Rewind.key` |
| `K` | Race: flip car upright | spz-races | `client/recover.lua` |
| `N` | Race: toggle standings list | spz-raceUI | `client/main.lua` |
| `J` | Hazard lights | spz-vehfunc | `client/main.lua` |
| `L` | Flash headlights (hold) | spz-vehfunc | `client/main.lua` |
| `G` | Horn taunt | spz-vehfunc | `client/taunts.lua` |
| `←` / `→` | Left / right indicator | spz-vehfunc | `client/main.lua` |
| `BACKSPACE` | Time trial: restart to start | spz-races | `client/timetrail.lua` |
| `LSHIFT` | Physics: shift up | spz-physics | `client/main.lua` |
| `LCONTROL` | Physics: shift down | spz-physics | `client/main.lua` |
| `LALT` | Physics: clutch (hold) | spz-physics | `client/main.lua` |

## Claimed by ox_lib

Not ours, and not to be taken. Rebinding them means editing a vendored library,
which is undone by the next ox_lib update.

| Key | Action |
| --- | --- |
| `Z` | Radial menu (carries the raceline / ghost controls) |
| `X` | Cancel the progress bar |

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
| `hideseekmenu` | Hide & Seek queue menu | spz-hideseek |
| `pursuitmenu` | Hot Pursuit queue menu | spz-pursuit |
| `spectate` | Toggle spectator mode | spz-spectate |
| `racelinetoggle` | Raceline: toggle display | spz-raceline |

## Free

`F1`, `F2`, `F12`, `C`, `M`, `P`, `U`, `Y`, `,`, `.`

## Not in this registry

Roughly forty call sites poll controls directly with
`IsControlJustPressed` / `IsDisabledControlJustPressed` (menu navigation,
editor and creator hotkeys, spectator camera). Those bypass
`RegisterKeyMapping` entirely: they cannot be rebound, they do not appear in
FiveM's key settings, and they are checked against GTA control IDs rather than
key names. They are not tracked here yet.
