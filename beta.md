# Beta notes

SPiceZ-Core is in **public beta**. The modules listed in [Modules](modules/README.md) are
the beta release: they are installed by the [txAdmin recipe](https://github.com/SPiceZ21/spz-txrecipe)
and covered by these docs.

## Status

| Area | State |
|---|---|
| Race loop, timing, results | Stable — runs unattended |
| Progression, records, leaderboard | Stable |
| Track Studio | Usable; new in this beta |
| Minigames | Playable; balance still moving |
| Replays, spectate | Usable |

## What to expect

- **Breaking changes are possible** between beta versions. Config keys and exports may be renamed; release notes call them out.
- **Schema changes only add.** New migrations are appended in `spz-core/migrations/`; existing ones are never edited, so upgrading never needs a manual SQL step.
- **Versions are per module.** Each repository releases on its own; the recipe always pulls the latest release.

## Known limitations

- **Game build 3407** is enforced by the recipe's `server.cfg`; other builds are untested.
- **Bomb Tag and Demolition** need the third-party `iak_demolition` arena map, which is not included.
- **Controller binds** print `Invalid I/O parameter pairing` warnings in the F8 console on some clients; keyboard binds are unaffected.
- **Custom tracks** are stored in `spz-races/data/custom_tracks.json`, not the database — back the file up.

## Not part of the beta

These live in the organisation but are not installed by the recipe or covered here:
`spz-physics`, `spz-carfx`, `spz-vegetation`, `spz-rocketleague`, `spz-tutorial`,
`spz-log`, `spz-grafnana`, `spz-checkpointprop`.

## Reporting issues

Open an issue on the module's GitHub repository with:

1. the module and version (from its `fxmanifest.lua`),
2. what you did and what happened,
3. the server console and F8 output around it.
