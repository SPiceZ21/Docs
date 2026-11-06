# Progression

> XP, safety rating, iRating and rank points.

> Part of [spz-races](spz-races.md): this was the separate resource `spz-progression` until it was merged in. Nothing to install on its own.

## Overview

Turns race results into progression: XP and levels, a safety rating, an iRating-style skill number and rank points that decide your rank tier. Also tracks rivals.

## Features

- XP and levels
- Safety rating from incidents
- iRating-style skill rating
- Rank points and tiers
- Rivals

## Configuration

- `spz-races/config_progression.lua` (`Config.Progression`)

See the file for every option.

## Commands

| Command | Who | Description |
|---|---|---|
| `/rank` | player | Show your rank |
| `/rival` | player | Show your rival and the gap on your last track |

## Exports

| Export | Side |
|---|---|
| `AdminRankInfo` | server |
| `AdminRankStats` | server |
| `AdminRecentRaces` | server |
| `AdminSetRP` | server |
| `AdminVoidRace` | server |
| `CalculateXP` | server |
| `GrantBonus` | server |
| `GrantFlatRP` | server |
| `LevelFromXP` | server |
| `XPRequired` | server |

```lua
exports["spz-races"]:AdminRankInfo(...)
```

---

Source: [https://github.com/SPiceZ21/spz-races](https://github.com/SPiceZ21/spz-races)
