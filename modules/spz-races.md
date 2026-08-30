# spz-races

> Race engine — queue, poll, countdown, checkpoints, sectors, results · `v1.9.0`

## Overview

`spz-races` runs the race: lifecycle state machine, join queue, track vote, grid and
countdown, checkpoint and sector validation, live positions, DNF and reconnect handling,
results, and the intermission that loops into the next race. It also hosts the leaderboard
back end (`server/leaderboard/`) that `spz-leaderboard` renders.

The server decides everything. A client may *propose* an event — "I crossed gate 7", "I
took a hit at 140 km/h", "I rewound 3.2 s" — and the server accepts it only if it can
check the claim against data it owns: the checkpoint you were due to cross and your
position relative to it, the state you are in, and hard per-race ceilings. Nothing a
client reports decides an outcome on its own.

## Modes

| Mode | Entry |
|---|---|
| Race | `/joinrace` — the standard cycle |
| Time trial | `/timetrail`, restart with `/tt_restart`, exit with `/quittt` |
| Duel | `/duel` — head-to-head challenge |

See [Race flow](../gameplay/race-flow.md) for the full cycle.

## Tracks

`data/tracks.lua` ships 101 tracks (76 circuit, 25 sprint), generated from JSON exports
with per-checkpoint headings and left/right gate posts. Lap count is derived at runtime
from measured track length — long circuits 2 laps, short ones 3.

Build tracks in-game with `/trackcreator` and `/trackeditor`; `/trackname` and
`/tracktype` set metadata, `/fixheadings` repairs checkpoint orientation.

## Exports

| Group | Exports |
|---|---|
| State | `GetRaceState` · `SetRaceState` · `ResetToIdle` · `ClearRaceState` |
| Queue | `JoinQueue` · `LeaveQueue` · `IsQueued` · `GetQueueCount` · `GetQueuePlayers` · `BroadcastQueueUpdate` · `FlushPendingToQueue` · `ClearPending` |
| Flow | `StartRacePoll` · `StartWarmupPhase` · `StartCountdownSequence` · `SetupRaceWorld` · `StartIntermission` · `RunRaceCleanup` |
| Checkpoints | `SetActiveCheckpoint` · `GetCurrentCP` · `HandleCheckpointAdvance` · `StartCheckpointVisuals` · `StopCheckpointVisuals` · `IsCheckpointVisualsActive` · `GetRespawnPoint` |
| Sectors | `RecordSectorHit` · `GetSessionBestSectors` |
| Positions | `CalculatePositions` · `UpdatePositions` |
| Results | `CheckAllFinished` · `ProcessRaceResults` · `MarkDNF` · `ProcessDNF` · `HandlePlayerDisconnect` |
| Time trial | `IsInTimeTrial` |
| Tracks | `SaveTrack` · `AddTrackCheckpoint` · `DeleteLastCheckpoint` · `CancelTrackCreator` |
| Spawn | `ConfirmRaceSpawn` |

## Timing

Every track is split into three sectors by checkpoint count. Sector times are coloured
purple / green / yellow and persisted per player, track and class, in both races and time
trials. `spz-raceUI` shows an F1-style live split against your best at each checkpoint.

A rewind shifts the racer's clock epochs forward rather than carrying a separate offset,
so race time, lap time, sector time and the idle watchdog all follow automatically. The
credited total rides through to the results payload as `rewind_ms`; anything non-zero
disqualifies the time from track records, personal bests and raceline storage.

## Events

Race event names live in `shared/events.lua`, merged into the `SPZ.Events` table that
`spz-core/shared/events.lua` declares. Another resource sees them by adding
`'@spz-core/shared/events.lua'` to its `shared_scripts` — each FiveM resource has its own
Lua state, so the table is not visible across resources without that import.

Two names are easy to confuse:

| Event | When | Use for |
|---|---|---|
| `SPZ:racerFinished` | Once **per finisher**, the moment they cross | Feeds, telemetry, anything reactive. The field is still incomplete. |
| `SPZ:raceEnd` | Once **per race**, from `ProcessRaceResults` | Scoring, persistence, payouts. This is the end-of-session contract. |

`SPZ:raceEnd` must be fired by nothing except `ProcessRaceResults`. Emitting it per
finisher ran every listener twice per race: double XP/SR/iRating, duplicate `race_results`
rows, a Discord post per finisher, and the betting pool settling on the first crossing.

## Dependencies

`ox_lib` · `spz-core` · `spz-identity` · `spz-vehicles` · `oxmysql`
