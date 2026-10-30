# Database

`spz-core` owns the schema. Every file in `spz-core/migrations/` runs once on first boot and
is recorded in `spz_migrations`; restarts and upgrades are no-ops. To change the schema, add
a new numbered file — never edit one that has shipped.

## Players and progression

| Table | Holds |
|---|---|
| `players` | Profiles, stats, licence tier, plate, Discord link |
| `driver_licenses` | Licence unlock history |
| `player_outfits` | Saved appearance per player |
| `player_badges` | Awards |
| `rank_awards` | Rank point awards |
| `player_rating_history` | iRating / SR history |
| `economy_transactions` | Credits ledger |
| `vehicle_customizations` | Saved car setups per player and model |

## Crews and rivals

| Table | Holds |
|---|---|
| `crews` | Crews, owner, settings, crew outfit |
| `crew_invites` · `crew_rivals` | Invites and crew rivalries |
| `rivals` · `rival_events` · `duels` | Player rivals and duels |

## Racing

| Table | Holds |
|---|---|
| `race_sessions` · `race_results` · `race_entries` | Races and per-player results |
| `race_laps` · `race_incidents` | Laps and incidents |
| `track_records` · `track_sectors` | Personal bests per track, class and sector |
| `racelines` | Stored best-lap lines |
| `race_replays` | Compressed replays |
| `race_poll_runs` · `race_poll_options` | Vote history |
| `speedcam_bests` | Speed camera records |

## Analytics

`player_sessions`, `player_activity`, `daily_player_stats`, `daily_credit_balances`,
`vehicle_usage`, `feature_usage`, `race_queue_events`, `race_engine_events`,
`connection_attempts`, `server_events`, `server_snapshots`, `admin_actions` — written by
`spz-analytics` for Grafana.

## Tracks are not in the database

Custom tracks from Track Studio are saved to `spz-races/data/custom_tracks.json`; the shipped
tracks live in `spz-races/data/tracks.lua`.
