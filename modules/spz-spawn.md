# spz-spawn

> Play menu, spawn points, world entry · `v2.2.0`

## Overview

`spz-spawn` moves the player from the loading screen into the world. Returning players see
a cinematic play menu with their stats and a spawn-point picker; first-time players go
through character creation first.

## Boot flow

The flow is **pull-based**: the client asks, the server answers. This matters more than it
sounds. At `playerConnecting` and `playerJoining` the client is still on the loading screen
with none of its scripts running, so anything the server sends it at that point is
discarded — which is exactly what the previous push-based version did.

```
playerConnecting        spz-core defers the connection
                        └─ spz-identity:PrepareProfile(identifier)
                           DB only: look up or create, check the ban.
                           No statebags (no Player yet), no client events (no client yet).
                           Failure here shows a real reason on the connection screen.

playerJoining           spz-core creates the session, fires SPZ:playerConnected
                        (server-side listeners only)

client scripts start    spz-spawn sends SPZ:spawn:hello
                        └─ retried with backoff, 8 attempts over ~60s

server                  spz-identity:AttachProfile(source)
                        binds the prepared profile, writes statebags, syncs,
                        fires SPZ:playerReady
                        └─ replies SPZ:spawn:route { mode = "create" | "menu" | "error" }

client                  acts on the route, streams the world behind a cover,
                        then hands off from the loading screen to the menu
```

One request, one reply. No polling, and a failure surfaces as a message rather than an
indefinite wait.

## Character creation

Two acts, in this order:

1. **Build** — pick a base model, then open the full appearance editor.
2. **Name** — alias, nation, race number.

Building comes first on purpose. Naming something you cannot see is an abstract form;
naming a racer standing in front of you is a decision. The ped is **live and visible**
behind the panel throughout — it used to be hidden for the whole of creation, so players
chose a model, a name, a nation and a number for someone they never saw.

The panel is docked to the left rather than centred for the same reason: a centred modal
covers the only thing worth looking at. The scrim is a one-sided gradient — readable on
the left, fully clear by mid-screen.

| Callback | Effect |
|---|---|
| `previewGender` | Swaps the live ped to the chosen base model, re-places and re-poses it |
| `openAppearanceStep` | Hands the screen to `fivem-appearance`, returns on close |
| `submitCharacterCreation` | Writes the profile — appearance is already done by then |

Handing off to the editor sends `creationPause`, **not** `hide`: the UI holds the model
choice and which act it is in, and the editor can be open for minutes. Unmounting it would
silently reset both. `SPZ:appearanceCustomizationDone` rebuilds the preview scene and sends
`appearanceStepDone`, which advances the UI to naming — an event, not a timer.

Nation is a search combobox with flags rather than a native `<select>`, which with seventy
entries offered no search, no flag, clipped the chosen name inside the control, and drew an
OS dropdown in the middle of the game.

### First-time players

`SPZ:playerReady` means *this player has a usable identity*, and seven resources act on it.
A first-time player has no username until character creation finishes, so the handshake
deliberately withholds the event for them — `spz-identity` fires it once, from the
character-creation completion path, instead of firing a half-built profile at everyone and
then firing again.

After the appearance editor closes the client calls `SPZ:spawn:requestMenu`, and the same
routing decision now answers `menu` because `first_time` is `0`.

## Spawn menu

Composed as a shot rather than a page. The ped and the world behind it are the subject, so
the UI is furniture around the frame edges and the middle stays clear — three anchors, no
floating boxes: **who you are** (top-left), **where you are going** (bottom-left), **the
commitment** (bottom-right).

**Cinematic bars** slide in from off-frame and crop the shot. Height is in `vh`, so the crop
is proportional at any resolution instead of a fixed band that swallows 720p and vanishes on
1440p. No trim on the inner edge — a letterbox is a crop, not a component; an accent line
along it turns the bars into a UI frame.

The vignette is **legibility insurance, not mood**. The backdrop is whatever the world
happens to be behind the ped, and white 300-weight display type disappears completely into a
midday Los Santos sky. Layered gradients guarantee dark ground under every anchor, and the
display type carries its own text shadow as a second line of defence.

Destination uses segments rather than dots (dots stop scanning past about five) and **cuts**
between names rather than cross-fading — faster to read, and it matches the camera language.

## Cinematic camera

A slow orbit with a **handheld** feel: the operator is breathing and shifting weight, so the
frame wanders slightly. Not shake — shake reads as an explosion.

The wander is summed sine waves at deliberately incommensurate frequencies, not random
jitter. Random per-frame values buzz; sines whose periods never line up produce a path that
keeps wandering without visibly repeating, and it is smooth by construction so it can never
pop between frames. Translation is in centimetres; the rotational sway is larger, applied by
nudging the *look-at point* rather than setting camera rotation — same effect, no matrix
work, and it cannot fight `PointCamAtCoord`. FOV breathes under a degree, because a perfectly
locked focal length reads as CG.

Everything is driven by **wall time**, not per-frame increments. The previous orbit advanced
a fixed amount each frame, so it ran at whatever the client's framerate happened to be: a
144 Hz machine orbited nearly 2.4× faster than a 60 Hz one, and any frame hitch jerked the
camera.

## Loading screen handoff

`spz-loading` owns `ShutdownLoadingScreen` (see [spz-core → loading](spz-core.md)). This
module never calls it directly; it reports stages and says when the handoff is safe:

```lua
exports['spz-loading']:Stage('world')
exports['spz-loading']:Finish()
```

The screen comes down only once the branded cover has painted **and** collision has
streamed at the preview location — so the player goes from one full-screen cover to
another and never sees raw world streaming.

## Events

| Event | Direction | Purpose |
|---|---|---|
| `SPZ:spawn:hello` | client → server | Client is running; asking for a route |
| `SPZ:spawn:route` | server → client | The decision: `create`, `menu`, or `error` |
| `SPZ:spawn:requestMenu` | client → server | Re-ask after character creation |
| `SPZ:requestSpawn` | client → server | Player picked a spawn point |
| `SPZ:showPlayMenu` | local | Open the menu with player metadata |
| `SPZ:spawnPlayerTarget` | server → client | Perform the physical spawn — model, teleport, resurrect |

`SPZ:requestPlayMenu` is kept as an alias for `SPZ:spawn:requestMenu` so older callers
keep working; both route through the same decision.

## Configuration

`spz-spawn/config.lua`:

```lua
Config.Spawns = {
    [1] = { label = 'Safe Zone', coords = vector4(x, y, z, w) },
}
```

## Commands

`/testspawn` · `/testcreation` — development helpers.

## Dependencies

`spawnmanager` · `ox_lib` · `spz-core` · `spz-identity`
