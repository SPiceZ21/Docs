# Progression and licensing

Every finished race feeds [spz-progression](../modules/spz-progression.md), which updates
five separate things.

| System | Meaning |
|---|---|
| **XP / level** | Participation and performance over time |
| **Championship points** | Finishing position, from the shared points table |
| **Safety Rating (SR)** | Clean driving — incidents and contact push it down |
| **iRating** | Relative pace against the field, Elo-style |
| **License** | The unlock gate for higher classes: C → B → A → S |

Ghost-bots are excluded from all of it — you are only ever scored against humans.

## Licenses

License tier is stored on the profile as `license_tier` (`0 = C`, `1 = B`, `2 = A`,
`3 = S`). Promotion is evaluated after each race against the criteria in
`spz-progression/shared/licenses.lua`; unlocks are granted through
`spz-identity:UnlockLicense` and recorded in the license history.

Vehicle classes are gated by tier, so a license is what opens faster cars rather than a
purchase.

## Vehicle classes

Race classes only — Coupes, Muscle, Sports Classics, Sports, Super, Open Wheel. Civilian,
service, utility and prop vehicles are not spawnable.

Classes can also be balanced by PP, the 0–1000 performance number computed by
[spz-physics](../modules/spz-physics.md) from power, weight and grip:

| Class | PP |
|---|---|
| C | ≤ 450 |
| B | ≤ 600 |
| A | ≤ 750 |
| S | ≤ 1000 |

## Rolling championship series

Every finished race is a round scoring F1-style points. Standings carry across rounds, and
after a dynamic number of rounds a champion is crowned and rewarded before a fresh series
starts automatically. There is no schedule — it flows race to race. `/series` shows the
live table.

## Rivals

Each player is paired with the nearest player by iRating. Beating your rival's stored time
on a track pings both of you in game and on Discord. `/rival` shows yours.

## Bonuses

**Perfect lap** — purple in all three sectors of the same lap, in a race or a time trial,
grants flat XP and credits (`Config.PerfectLap`) with a notification, rank-up sting and
post-fx flourish. It rewards the flawless lap, not merely the fast one.

Other one-off rewards use the generic export:

```lua
exports['spz-progression']:GrantBonus(source, { xp = 250, credits = 500, reason = 'event' })
```

## Record crowns

Holding a track's fastest stored line earns a gold crown on your nametag, rendered by
`spz-nametag` from the `spz:records` statebag (with a count when you hold more than one).
Taking a record broadcasts to everyone — "X snatched the [track] record from Y" — and logs
to Discord. Crown counts refresh live for both the new holder and the dethroned one.

In-world record boards at configurable spots show a track's fastest-lap holders as floating
scoreboards you walk up to and read — no menu.
