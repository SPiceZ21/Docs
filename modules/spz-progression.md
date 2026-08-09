# spz-progression

> XP, ranks, Safety Rating, iRating, license promotion, seasons · `v2.1.0`

## Overview

`spz-progression` listens for race results and converts them into progress: XP and levels,
championship points, Safety Rating, an Elo-style iRating, rank movement and license
unlocks. It also owns seasons, rivals and series.

It never writes identity data directly — promotions and rating changes go through
`spz-identity` exports.

## Systems

| System | What it measures |
|---|---|
| **XP / level** | Total participation and performance over time |
| **Championship points** | Finishing position in a race, per the shared points table |
| **Safety Rating (SR)** | Clean driving — incidents and contact push it down |
| **iRating** | Relative pace against the field, Elo-style |
| **Rank** | Derived display tier from the values above |
| **License** | Unlock gate for higher classes — C → B → A → S |

## Exports

| Group | Exports |
|---|---|
| XP | `CalculateXP` · `LevelFromXP` · `XPRequired` · `GrantBonus` |
| Points | `CalculatePoints` |
| Ratings | `CalculateSRDelta` · `ApplySR` · `CalculateIRatingDeltas` |
| Ranks | `ComputeRank` · `CheckRankPromotion` |
| Licenses | `CheckLicenseUnlock` |
| Series | `GetSeries` |

## Configuration

`config.lua` holds the multipliers and thresholds; the shared tables
(`shared/points.lua`, `shared/ranks.lua`, `shared/licenses.lua`) hold the curves both sides
read.

## Commands

`/spz` · `/rival` · `/series`

## Dependencies

`ox_lib` · `spz-core` · `spz-identity` · `spz-races`
