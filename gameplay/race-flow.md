# Race flow

One continuous cycle, driven by `spz-races` on the server. There is no fixed schedule and
no minimum player count — the first person to queue starts the clock.

## The cycle

| Phase | What happens |
|---|---|
| **Idle** | Nobody queued. The first `/joinrace` arms a dynamic join window. |
| **Poll** | Players vote on track and vehicle class ([spz-poll](../modules/README.md)). |
| **Warmup** (90 s) | World and bucket set up, grid spawned. Doubles as spawn grace for slow clients. `/leaverace` here tears everything down cleanly. |
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

Physics collision and **camera** collision are separate systems in GTA. Turning off the
first lets cars pass through each other; the chase camera still sweeps against the other
car's bounds and gets shoved into or under your own vehicle as you overlap. Both are
handled in `spz-core/client/ghost.lua`: pairwise no-collision for the bodies, and a
per-frame camera-collision guard. The camera flag lasts exactly one frame, so it is
re-asserted every frame, in two passes with different freshness needs:

- **Players, every frame.** Handles resolved fresh, so a car that streams in alongside you
  is covered on the frame it appears. This is the pass that matters in a race, where cars
  close on each other fast enough that a cached list is already stale.
- **A throttled pool sweep** for what player enumeration cannot see: a ghosted car with
  nobody in it, a remote ped whose seat has not synced, race bots, duel and raceline
  ghosts, checkpoint gate props. None of those close on you in a tenth of a second.

NPC traffic is deliberately excluded — those cars really do collide, so the camera should
collide with them too.

## Checkpoints and sectors

Checkpoints use custom gate props that swap `_a` → `_b` as you cross, so the hit is
visually confirmed. Every track is split into three sectors by checkpoint count; sector
times are coloured purple / green / yellow and stored per player, track and class. The HUD
shows a live +/- split against your best at each checkpoint.

Lap count is derived from measured track length: long circuits run 2 laps, short ones 3.

Every crossing is banked against a progress index (gates cleared since GO), which is what
lets the tower state gaps in **seconds** rather than "+2 CP": the gap is how long ago the
car ahead was standing where you are now. It stays a real time across a lap boundary — a
lapped car reads as the time it is actually down, with a `1L` suffix. `interval` (gap to
the car directly ahead) is published alongside `gap` (gap to the leader).

**Missing a gate is told to you, not left to be discovered.** The crossing test already
knows the difference between going through the gate and going past it — same plane flip,
but wide of the posts — so a miss raises a prompt offering both recoveries by key: `F5` to
rewind, `F4` to teleport to the last checkpoint you crossed. Without it the first sign of
trouble is the next gate never arming, and eventually the idle-kick DNF.

Your client decides *when* you crossed — it owns the frame the gate plane was passed, and
the server cannot see that. The server decides *whether you could have*: a claimed hit is
rejected unless the checkpoint is the one you were actually due to cross and your ped is
within range of it. The radius is deliberately loose (it has to tolerate a network tick of
lag), so it never affects real racing — it exists so a modified client cannot claim a lap
it never drove.

## Rewind

Hold `F5` to scrub the car backward along its recent path, Forza-style, then release to
resume from that point with the momentum it had. The race clock rewinds with the car, so a
rewind reads as *undo* rather than *teleport plus penalty*.

Bounded by the rolling history buffer (10 s) and by a server-side ceiling on how much clock
a single lap can win back (`Config.Rewind.maxCreditPerLapMs`, 15 s). Every gate you scrub
back past still has to be re-crossed for real.

**A rewound run is not a record.** Any run that credits even a millisecond back is barred
from track records and personal bests, and its driven line is not stored — those lines are
replayed as ghost-bots and used as duel targets, so a refunded lap would seed a ghost
nobody can beat. Rewinding inside a **duel** earns no clock credit at all: duels pay real
credits against a stored time, so the rewind still works but costs exactly what it costs.

## Reconnecting

A crash or timeout holds your grid slot for 60 seconds. Rejoin inside that window and your
lap, checkpoint and time are restored at your last checkpoint instead of an instant DNF.

## Overtakes

A completed pass is detected server-side from the live running order. It posts an auto-clip
to Discord, toasts the whole field, and the **overtaker's car spits nitrous flames** — a
networked particle, so everyone sees it on that car. Both share one per-pair cooldown
(`Config.OvertakeCooldown`), so a back-and-forth battle cannot spam either.

The flames are visual only. `Config.OvertakeNos.boost` can add a real Rocket-Voltic speed
boost on top, and is **off by default**: paying for a pass with pace makes overtaking
self-reinforcing, which is a racing decision rather than a cosmetic one.

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

**Leaving the car ends the run.** A time trial is the car — on foot there is no lap to
time, and the session would otherwise sit open holding a routing bucket and a spawned
vehicle. You get a few seconds to climb back in before it ends; getting knocked out of the
seat or ragdolling does not count while you are still getting back in.

## Duels

`/duel <player|id> <track> <stake>` — stake credits against an opponent's **stored best
line**, replayed as a translucent ghost. The opponent does not need to be online; you are
racing their recorded lap.

The stake is escrowed immediately and the server compares its own measured time to the
opponent's stored best — the client never reports the outcome. Win pays `stake × 2`. By
default (`Config.Duel.HouseFunded`) winnings are a house bounty, so an offline opponent is
never charged; flip it off for opponent-matched stakes. Quitting, disconnecting or a failed
start voids the duel and refunds the escrow, and a duel against a player with no stored
line on that track is rejected up front. Rewind earns no clock credit inside a duel. Every
duel lands in the `duels` ledger.

## Watching a race

Anyone not racing can `/spectate` — the viewer is moved into the target's bucket so
isolated racers stay visible.

Everyone outside the race also gets a passive **live race board** in the top-right corner:
the track, the leader's lap, and the running order with real time gaps, ghost-bots marked
`BOT` and held reconnect slots marked `DC`. It needs no command and takes no input — it
appears when a race goes live and holds the final classification for a few seconds after
the flag.

`F7` cycles it between the full board, a three-row mini view, and hidden; `F8` hides or
restores it outright. Both keys are shown on the board itself and are rebindable, and
`/raceboard` works from chat. The board lives in [spz-spectate](../modules/README.md) —
same audience, so it shares that resource rather than adding another.
