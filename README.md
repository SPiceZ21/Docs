# Introduction

**SPiceZ-Core** (`spz-*`) is an open-source, racing-only framework for FiveM. Every
feature is a standalone resource; modules talk to each other only through events and
exports, and the server owns every decision that affects a race result.

## Philosophy

1. **No bloat** — racing only. No jobs, no housing, no crime.
2. **Modular** — run the modules you want, drop the rest.
3. **Event-driven** — clean separation via exports and events.
4. **Server-authoritative** — the server owns race state, positions and results.
5. **One schema owner** — all SQL lives in `spz-core/migrations/`.

## Where to start

| You want to | Go to |
|---|---|
| Get a server running | [Installation](get-started/installation.md) |
| Configure it after first boot | [First-time setup](get-started/first-time-setup.md) |
| Understand how the pieces fit | [Architecture](architecture.md) |
| Look up a module's exports | [Modules](modules/README.md) |
| Know how a race actually runs | [Race flow](gameplay/race-flow.md) |

## Versions

Each module carries its own version in `fxmanifest.lua`; the module list in
[Modules](modules/README.md) tracks the current set. Cross-repo change history lives in
`CHANGELOG.md` at the root of the workspace.
