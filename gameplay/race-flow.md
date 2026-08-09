# Race flow

One continuous cycle, driven by `spz-races` on the server. There is no fixed schedule and
no minimum player count — the first person to queue starts the clock.

## The cycle

| Phase | What happens |
|---|---|
| **Idle** | Nobody queued. The first `/joinrace` arms a dynamic join window. |
| **Poll** | Players vote on track and vehicle class ([spz-poll](../modules/README.md)). |
| **Warmup** (60 s) | World and bucket set up, grid spawned. Doubles as spawn grace for slow clients. `/leaverace` here tears everything down cleanly. |
| **Grid** | Cars settle on the start point. Ghosting is armed before vehicles spawn, so a single spawn point is safe. |
| **Countdown** | 3-2-1, then GO. |
| **Live** | Checkpoints, sectors, positions, overtakes and incidents. |
| **Finish** | The first finisher arms a straggler countdown — warnings at 60 / 30 / 10 s, then force-DNF. The podium never waits. |
| **Results + intermission** | Run in parallel; the next join window arms immediately. |

Post-race overlap cuts what used to be ~110 s of dead time to roughly 60 s.

## Isolation

Each race runs in its own routing bucket, so racers never see free-roam traffic. Players
and their cars never collide with each other anywhere — ghosting is global and
client-side.

## Checkpoints and sectors

Checkpoints use custom gate props that swap `_a` → `_b` as you cross, so the hit is
visually confirmed. Every track is split into three sectors by checkpoint count; sector
times are coloured purple / green / yellow and stored per player, track and class. The HUD
shows a live +/- split against your best at each checkpoint.

Lap count is derived from measured track length: long circuits run 2 laps, short ones 3.

## Reconnecting

A crash or timeout holds your grid slot for 60 seconds. Rejoin inside that window and your
lap, checkpoint and time are restored at your last checkpoint instead of an instant DNF.

## Ghost-bots

Thin grids are backfilled to `Config.Bots.targetField` (default 6) with real stored human
lines from [spz-raceline](../modules/README.md), replayed as solid, non-collidable cars at
a mixed pace spread. The server simulates their progress from recorded checkpoint splits;
clients replay them locally off the GO clock, so there is no per-frame network cost.

Bots appear in the standings tower and on the map as grey `[BOT]` blips and push your
visible position, but they are **never** in `results.finishers` and grant no rewards —
humans are scored only against humans. A track needs at least one stored line before bots
can run it.

## Time trial

`/timetrail` — pick a car, then a track. The car spawns at a head-start point before the
line, and hotlaps roll continuously: cross the line, the lap banks, the next one starts.
No stop-start countdown. On circuits the start/finish line is the last checkpoint of the
lap. Race-join prompts are suppressed inside a time trial.

## Duels

`/duel <player|id> <track> <stake>` — stake credits against an opponent's **stored best
line**, replayed as a translucent ghost. The opponent does not need to be online; you are
racing their recorded lap.

The stake is escrowed immediately and the server compares its own measured time to the
opponent's stored best — the client never reports the outcome. Win pays `stake × 2`. By
default (`Config.Duel.HouseFunded`) winnings are a house bounty, so an offline opponent is
never charged; flip it off for opponent-matched stakes. Quitting, disconnecting or a failed
start voids the duel and refunds the escrow, and a duel against a player with no stored
line on that track is rejected up front. Every duel lands in the `duels` ledger.

## Spectating and betting

Anyone not racing can `/spectate` — the viewer is moved into the target's bucket so
isolated racers stay visible. Non-racers can also `/bet` on a human racer while the
betting window is open; odds are pure pari-mutuel and shift as bets land and the order
changes. See [spz-betting](../modules/README.md).
