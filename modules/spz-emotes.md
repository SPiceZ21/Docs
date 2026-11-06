# Emotes

> Emote menu: dances, gestures, poses, shared emotes, walk styles and moods.

> Part of [spz-hud](spz-hud.md): this was the separate resource `spz-emotes` until it was merged in. Nothing to install on its own.

> Players' saved favourites, walk style and mood are carried over from spz-emotes automatically the first time they join after the update.

## Overview

`O` opens an emote menu with search, categories, favourites and animated previews. `/e <name>` plays one directly, `X` stops it. Shared emotes (handshake, hug, high five) ask the nearest player to accept.

![Emote menu with animated previews](https://raw.githubusercontent.com/SPiceZ21/Docs/main/images/emote-menu.jpg)

## Features

- ~100 base-game emotes
- Animated previews
- Favourites
- Shared two-player emotes
- Walk styles and moods that persist

## Configuration

- `spz-hud/config_emotes.lua (`Config.Emotes`)`

See the file for every option.

## Commands

| Command | Who | Description |
|---|---|---|
| `/e` | player | `/e <name>` play an emote, `/e` stop |
| `/emotes` | player | Open the menu |

## Key binds

Rebindable in **Settings → Key Bindings → FiveM**.

| Action | Default |
|---|---|
| Emotes: open menu | `O` |
| Emotes: stop emote | `X` |

## Exports

| Export | Side |
|---|---|
| `OpenEmoteMenu` | client |
| `PlayEmote` | client |
| `StopEmote` | client |

```lua
exports["spz-hud"]:PlayEmote("dance")
```

---

Source: [https://github.com/SPiceZ21/spz-hud](https://github.com/SPiceZ21/spz-hud)
