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

### Three separate systems

GTA resolves entities against each other in three ways, and each needs its own handling.
Conflating them is the source of every collision bug this project has had:

| System | What it does | Controlled by |
|---|---|---|
| Contact response | Two entities push each other | `SetEntityNoCollisionEntity` |
| Camera sweep | The chase cam collides with bodywork | `DisableCamCollisionForObject` |
| **Placement resolver** | Ejects entities created or teleported inside each other | **Nothing — it cannot be switched off** |

The third is the one that bites. It runs on vehicle creation and on teleport, and no
collision exclusion touches it. The grid used to spawn every car on **identical
coordinates** (`SpawnMode = "point"`, radius 0) on the theory that ghosting made overlap
safe; it does not, and that is what launched cars across the map on race start. Two cars at
one coordinate are overlapping for the entire race, and every physics re-evaluation is
another chance to be thrown.

The fix is geometry, not flags: cars spawn on separated slots, and point mode floors its ring
radius rather than honouring a value that cannot work. The ped is also **frozen** on its slot
until its car exists, because the car being created around a standing ped ejects the ped —
released as soon as the vehicle appears rather than once the player is seated, so a failed
warp can never weld someone to the tarmac.

### Warmup grid vs. race start

The two phases want opposite shapes, so they are configured separately:

| Phase | Mode | Why |
|---|---|---|
| Warmup | `Config.WarmupSpawnMode` = `grid` | Nothing is being won yet, and spread-out slots are the safe way to put a full field on one road |
| Race start | `Config.RaceStartMode` = `point` | A staggered grid decides places before the lights |

Starting a race from the warmup grid hands row 1 roughly **56 m** over row 8 on a full field.
The race re-stage collapses the field onto **one coordinate** (`PointSpawnRadius = 0`), which
is the only genuinely equal start there is.

That works here and **not** at vehicle creation, and the difference is the whole point. At the
re-stage the cars already exist, they are frozen either side of the teleport, and they are
ghosted against each other — nothing simulates contact while they sit stacked on the line, and
on GO they drive through one another. At creation there is no such window: the resolver runs as
the vehicle comes into existence, which is why warmup still uses real slots.

One line of code decides whether this works. `SetEntityCoords`' final argument is `clearArea`,
which asks the engine to **shove whatever is already at the destination out of the way**. With
every car re-staged onto the same coordinate, each arrival was booting the cars that got there
before it, one after another — and no collision flag touches that path, because it is not a
collision. It is the engine doing exactly what it was asked. It is now `false`.

A non-zero `PointSpawnRadius` turns the start into a ring instead — still equal-distance, but
spread out. Ring spacing is driven by car **length**, not width (every car faces down the track,
so the two at the front and back of the circle sit nose-to-tail); six metres of arc per car keeps
a minimum of 5.4 m between any pair. Past `Config.PointSpawnMaxRadius` a ring is wider than the
road, so it falls back to a grid and logs why, and `Config.GridCarsPerRow` controls how flat that
fallback is — sixteen cars at 2 abreast is eight rows and 56 m of stagger, at 4 abreast four rows
and 24 m.

### Ghosting cadence

The no-collision flag is not as permanent as it looks. It is dropped when an entity is
re-created (stream out and back in), when network ownership migrates, and whenever anything
calls `SetEntityCollision` on either half of the pair — which `spz-spawn` does after every
spawn and `spz-appearance` after every outfit change. None of those change the entity
*handle*, so a signature check on handles cannot see them.

`spz-core/client/ghost.lua` therefore splits the work by who needs it:

- **Near pass, every frame** — your ped and your car against every player within 70 m. This
  is the only set whose physics you simulate, so it is the only set where a missing flag can
  throw *you*. A 200 ms re-assert used to leave an ~11 m hole at racing speed, which is one
  real hit and reads as "ghosting randomly failed".
- **Full pass, throttled** — every remaining pair, including remote-vs-remote. Worth doing
  for entities you own, but not per frame: you do not simulate contact between two cars
  someone else owns, you replay the positions their owner sends.

That last point bounds what any of this can fix. **Collision is resolved by the network
owner**, so ghosting is only ever as good as the worst-behaved client in the lobby — if
another player's client has not ghosted yet, you will be shown the bounce their client
simulated no matter what flags you set locally.

The camera flag lasts exactly one frame, so it is re-asserted every frame, in two passes
with the same split:

- **Players, every frame.** Handles resolved fresh, so a car that streams in alongside you
  is covered on the frame it appears. This is the pass that matters in a race, where cars
  close on each other fast enough that a cached list is already stale.
- **A throttled pool sweep** for what player enumeration cannot see: a ghosted car with
  nobody in it, a remote ped whose seat has not synced, duel and raceline ghosts,
  checkpoint gate props. None of those close on you in a tenth of a second.

NPC traffic is deliberately excluded — those cars really do collide, so the camera should
collide with them too.

## Checkpoints and sectors

Checkpoints use custom gate props that swap `_a` → `_b` as you cross, so the hit is
visually confirmed. Every track is split into three sectors by checkpoint count; sector
times are coloured purple / green / yellow and stored per player, track and class. The HUD
shows a live +/- split against your best at each checkpoint.

Lap count is derived from measured track length: long circuits run 2 laps, short ones 3.

**The turn guide** floats over the road ahead of your car, anchored in world space so it
travels with you rather than sitting in a screen corner. It calls the turn you make **at**
the next gate — not the direction of the gate itself, which the blips and props already
show — along with the distance to it and your speed.

The angle comes from the two legs of the route: the one you are driving (car → next gate)
against the one after it (next gate → the gate beyond). Bands: under 12° is straight, under
40° slight, under 100° a plain left/right, under 150° hard, beyond that a U-turn. The arrow
is rotated by the **real** angle rather than picked from a few fixed glyphs, so a kink and a
hairpin do not draw the same picture.

The cluster mirrors around the arrow — right turns put the arrow right and the speed left,
left turns flip both — so the layout points the way before a word of it is read. Inside 40 m
it goes accent-coloured: the call has stopped being advice and become an instruction.

**The CP distance pill** is the other half of the pair: pinned to the checkpoint with a stem
down to the gate point, it answers *where is the gate* rather than *what does the road do*.
Still useful on an unfamiliar track or when a gate sits behind geometry, but its distance
duplicates the guide's when both run, so it is **off by default**.

### Turning them on and off

Both are server-controlled, independently. Defaults live in `spz-races/config.lua`:

```lua
Config.Hud = {
  TurnGuide      = true,
  CpDistancePill = false,
}
```

Either can be overridden live from `server.cfg` without editing the file or restarting the
resource — the convar wins when set:

```
setr spz_hud_turn_guide 1
setr spz_hud_cp_pill 0
```

`/racehud` (admin, or the server console) re-reads the convars and republishes. The flags
travel in `GlobalState`, so a change reaches every connected client immediately; nobody has
to reconnect.

An unset convar is told apart from one explicitly set to `0` by reading it as a string with
a sentinel default — `GetConvarInt` cannot express that difference, since it returns `0` for
both "off" and "absent", which would silently override the config file with a value nobody
set.

The client sends only the half the server has enabled, so a disabled readout costs no
projection work and its absence from the payload *is* the off switch — there is no separate
visibility flag that could drift out of step.

**The map shows three gates ahead**, not one: the gate you are driving at, plus the two
after it. They stay numbered and fade with distance ahead so the order reads at a glance,
and they are long-range blips so they sit on the minimap edge before you can see them.

Only the active gate carries a **GPS route line**. The game draws one line per routed blip
and three overlap into an unreadable tangle, so the lookahead is carried by the gates
themselves being visible rather than by more route lines competing with the one you are
actually following. The lookahead wraps on circuits — two gates past the last one is gate
two of the next lap.

Every crossing is banked against a progress index (gates cleared since GO), which is what
lets the tower state gaps in **seconds** rather than "+2 CP": the gap is how long ago the
car ahead was standing where you are now. It stays a real time across a lap boundary — a
lapped car reads as the time it is actually down, with a `1L` suffix. `interval` (gap to
the car directly ahead) is published alongside `gap` (gap to the leader).

A gate scores when you cross its plane between the posts, in **either direction**.

Direction used to be enforced — only a "forward" crossing counted — and it was removed on
request. Players who spun, reversed through a gate, or arrived at it from the far side found
the checkpoint silently refusing to count, and being stranded on track is worse than the
shortcut this allows. **Order and proximity are still enforced server-side**: it has to be
the gate you were due to cross and you have to be near it. Only the direction is free.

Two things the crossing test depends on:

- The gate plane is **infinite**, so which side a player is on only means anything *near
  the gate*. Side is tracked inside a corridor (50 m deep, one gate-width of lateral slack
  each way) and forgotten outside it, because inside that corridor the only way to get from
  one side to the other is to actually pass the gate. Two earlier versions got this wrong in
  opposite directions: assuming "before" everywhere scored the gate the instant it armed if
  you happened to be past the plane, and reading the raw side everywhere seeded you as
  "past" while you were still far out on the flank, so driving through correctly registered
  nothing and you had to double back to score.

  The corridor is coupled to the detector's poll rate — the whole corridor must be sampled
  at 20 ms or better, or a fast car can enter it and cross the plane inside one sample.
  Change one and change the other.
- A **hysteresis band** of 0.4 m around the plane. A car sitting on the line is never
  perfectly still — suspension settle, collision jitter and network correction all wobble it
  by centimetres, and each wobble across the plane is a side change. This matters much more
  now that a crossing counts either way: one-directional scoring only ever saw half of that
  jitter.

`heading` still orients the plane, and still has to survive persistence (every conversion in
`server/creator.lua` carries it) — it is what makes the start line seed as "before" so the
first checkpoint of a race can be scored at all.

**Missing a gate is told to you, not left to be discovered.** The crossing test already
knows the difference between going through the gate and going past it — same plane flip,
but wide of the posts — so a miss raises a prompt offering both recoveries by key: `F5` to
rewind, `F4` to teleport to the last checkpoint you crossed. Without it the first sign of
trouble is the next gate never arming, and eventually the idle-kick DNF.

A miss also requires being **near** the gate: within one gate-width of a post. The plane
has no width, so a flip can happen hundreds of metres to the side, where nothing has been
missed — the car is simply elsewhere on the track.

## What you cannot do mid-race

Spawning or deleting a car during a race, a queue or a time trial is refused **server-side**
(`spz-carspawner`). The radial menu also hides those options while a route is running, but
that is presentation: `/car` and `/dv` are still typeable and the net event behind the menu
is still callable from a client, so the decision belongs on the server. A car the race
engine never issued desyncs the field, orphans the old vehicle in the race bucket, and
carries none of the race's handling or damage state.

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
replayed as the time-trial ghost and used as duel targets, so a refunded lap would seed a
ghost nobody can beat. Rewinding inside a **duel** earns no clock credit at all: duels pay real
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
the track, the leader's lap, and the running order with real time gaps, and held
reconnect slots marked `DC`. It needs no command and takes no input — it
appears when a race goes live and holds the final classification for a few seconds after
the flag.

`F7` cycles it between the full board, a three-row mini view, and hidden; `F8` hides or
restores it outright. Both keys are shown on the board itself and are rebindable, and
`/raceboard` works from chat. The board lives in [spz-spectate](../modules/README.md) —
same audience, so it shares that resource rather than adding another.
