# SPiceZ-Core

**SPiceZ-Core** (`spz-*`) is an open-source, racing-only framework for FiveM. It turns a
FiveM server into a competitive racing server: a continuous race loop with voting,
server-side timing, ranks and records, plus freeroam, crews and minigames around it.

> **Public beta.** Everything here is playable and running on live servers, but expect
> rough edges. See [Beta notes](beta.md) for what is in scope and how to report issues.

## What you get

- **A full race loop** — queue, vote on track and class, warmup, grid start, race, results — running on its own.
- **Server-authoritative timing** — lap, sector and finish times are measured on the server; no client decides a result.
- **Progression** — XP, safety rating, iRating-style skill and rank tiers, records and rivals.
- **Racing tools** — Time Trials, duels, recorded best lines with ghost cars, replays with broadcast cameras, spectate.
- **Track Studio** — build new tracks in the world by driving the route.
- **Freeroam & social** — car spawner, tuner, emotes, crews with radio, chat, nametags.
- **Minigames** — Color Rush, Cyclone, Bomb Tag, Demolition and Hot Pursuit.
- **One-click install** — a txAdmin recipe; the database builds itself on first boot.

## Principles

1. **Racing only** — no jobs, housing or crime systems.
2. **Modular** — every feature is its own resource; run what you want.
3. **Event-driven** — modules talk only through exports and events.
4. **Server decides** — race state, positions and results live on the server.
5. **One schema owner** — all SQL lives in `spz-core/migrations/`.

## Where to start

| You want to | Go to |
|---|---|
| Get a server running | [Installation](get-started/installation.md) |
| Configure it after first boot | [First-time setup](get-started/first-time-setup.md) |
| See every module | [Modules](modules/README.md) |
| Understand how the pieces fit | [Architecture](architecture.md) |
| Know how a race runs | [Race flow](gameplay/race-flow.md) |
| Look up a key or command | [Keybinds](reference/keybinds.md) · [Commands](reference/commands.md) |

## Links

- GitHub: [github.com/SPiceZ21](https://github.com/SPiceZ21)
- txAdmin recipe: [spz-txrecipe](https://github.com/SPiceZ21/spz-txrecipe)
