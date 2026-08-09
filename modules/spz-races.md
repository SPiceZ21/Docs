# spz-races

> Race engine — queue, poll, countdown, checkpoints, sectors, results · `v1.9.0`

## Overview

`spz-races` runs the race: lifecycle state machine, join queue, track vote, grid and
countdown, checkpoint and sector validation, live positions, DNF and reconnect handling,
results, and the intermission that loops into the next race. It also hosts the leaderboard
back end (`server/leaderboard/`) that `spz-leaderboard` renders.

The server decides everything. Clients report checkpoint hits; they never decide an
outcome.

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
| State | `GetRaceState` · `SetRaceState` · `ResetToIdle` |
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

## Dependencies

`ox_lib` · `spz-core` · `spz-identity` · `spz-vehicles` · `oxmysql`
