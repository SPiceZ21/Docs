# spz-progression

> XP, ranks, Safety Rating, iRating, license promotion, seasons · `v2.1.0`

## Overview

`spz-progression` listens for race results and converts them into progress: XP and levels,
championship points, Safety Rating, an Elo-style iRating, rank movement and license
unlocks. It also owns seasons and rivals.

It never writes identity data directly — promotions and rating changes go through
`spz-identity` exports.

Scoring hangs off `SPZ:raceEnd`, which `spz-races` fires exactly once per race with the
complete field — iRating in particular is meaningless against a partial one. Do not add a
handler for `SPZ:racerFinished` here: that fires per finisher and would score a racer twice.

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

## Configuration

`config.lua` holds the multipliers and thresholds; the shared tables
(`shared/points.lua`, `shared/ranks.lua`, `shared/licenses.lua`) hold the curves both sides
read.

## Commands

`/spz` · `/rival`

## Dependencies

`ox_lib` · `spz-core` · `spz-identity` · `spz-races`
