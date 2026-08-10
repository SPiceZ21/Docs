# spz-physics

> Powertrain simulation, tire compounds, PP rating, drift scoring · `v0.4.0`

## Overview

`spz-physics` replaces GTA's flat drive force with a simulated powertrain. Engine RPM is
derived from wheel speed × gearing, torque comes from a per-car dyno curve, and power
delivery is reshaped every frame via `SetVehicleCheatPowerIncrease`. GTA still handles
suspension, collision, steering, braking and top speed.

Standalone — it has no framework dependencies, and other resources pull state from it
rather than being pushed to.

## What it simulates

- Torque-curve power band from per-model profiles
- RPM driven by the driven wheels, so wheelspin revs the engine, with crank inertia
  smoothing the response
- Simulated RPM driving engine audio
- Auto, sequential and manual + clutch transmissions, with shift cut, clutchless
  grinding penalty, a modelled reverse gear and an auto box that won't hunt
- Rev limiter that bounces, mass-normalised engine braking, free rev in neutral
- Turbo (spool and blow-off) and supercharger boost
- Launch control, TCS, and an LSD that brake-vectors the spinning drive wheel
- Tire compounds as live grip multipliers: `street · sport · semislick · drift · offroad`

## Handling safety

Compounds are the only writes this resource makes to `CHandlingData`, limited to three grip
fields and funnelled through `client/handling.lua`.

Because compounds multiply stock grip, stock has to be exact. The baseline is captured once
per model *before* anything is written and persisted to KVP, so a restart mid-drive can
never record already-modified values as stock and ratchet grip upward. Applies and restores
compute from that baseline, making them idempotent. Implausible baselines disable compounds
for the model instead of corrupting it, a stored baseline that disagrees with a clean
vehicle is treated as a changed `handling.meta` and adopted, and the same disagreement after
a write is reported as cross-vehicle leakage.

Diagnostics: `/physhandling`, `/physrestore`, `/physforgetbaseline`.

## PP rating

A 0–1000 performance number from peak power (torque × boost), weight and grip. Class
bands (`PPClassBands`):

| Class | PP |
|---|---|
| C | ≤ 450 |
| B | ≤ 600 |
| A | ≤ 750 |
| S | ≤ 1000 |

Useful for class balancing — the server can rate a model without anyone driving it.

## Drift scoring

Slip-angle based: hold 12–65° to score, combo grows every 3 s up to ×5, straighten to
**bank**, spin past 85° or crash to **bust**.

## Profiles

`data/profiles.lua`, keyed by lowercase model display name
(`GetDisplayNameFromVehicleModel`); unlisted models use `Default`. Examples included:
`sultan` (AWD), `banshee` (RWD V8), `elegy` (tuner), `bati` (sport bike).

## Integration

```lua
-- client
local st       = exports['spz-physics']:GetPhysicsState()
local pp, cls  = exports['spz-physics']:GetVehiclePP()
local drift    = exports['spz-physics']:GetDriftSession()

exports['spz-physics']:SetTransmissionMode('sequential')
exports['spz-physics']:SetTCS(false)
exports['spz-physics']:SetTireProfile('semislick')

AddEventHandler('spz-physics:driftBanked', function(points) end)

-- server (class balancing)
local pp, class = exports['spz-physics']:GetModelPP('banshee', 'sport')
local class     = exports['spz-physics']:GetClassForPP(612)

-- any resource: gear and RPM are on the vehicle statebag
local gear = Entity(veh).state.physGear
```

## Commands

`/transmode` · `/tcs` · `/tires <compound>` · `/pp` · `/driftreset` · `/physhud`
· `/physhandling` · `/physrestore` · `/physforgetbaseline`

Shift up, shift down and clutch are rebindable in Settings → Key Bindings
(`LEFT SHIFT`, `LEFT CTRL`, `LEFT ALT` by default).

## Dependencies

None.
