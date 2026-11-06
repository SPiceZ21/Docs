# Appearance

> Freemode ped, personal outfits and crew outfits on top of fivem-appearance.

> Part of [spz-core](spz-core.md): this was the separate resource `spz-appearance` until it was merged in. Nothing to install on its own.

## Overview

Applies the right look to every player: their saved personal appearance, or their crew's outfit (clothing only) over their own face and hair. Opens the fivem-appearance editor for new characters and on demand.

## Features

- Personal appearance saved per player
- Crew owner's outfit pushed to every member (clothes only)
- Re-applied on spawn, crew change and resource restart
- Customiser lock so nothing overwrites a look being edited

## Commands

| Command | Who | Description |
|---|---|---|
| `/appearance` | player | Open the character editor |
| `/resetoutfit` | player | Drop your saved outfit (falls back to crew/default) |
| `/saveoutfit` | player | Save your current outfit |

## Exports

| Export | Side |
|---|---|
| `IsCustomizing` | client |
| `ReapplyMyOutfit` | client |

```lua
exports["spz-core"]:IsCustomizing(...)
```

---

Source: [https://github.com/SPiceZ21/spz-core](https://github.com/SPiceZ21/spz-core)
