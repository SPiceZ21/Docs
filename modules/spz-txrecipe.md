# spz-txrecipe

> txAdmin recipe: a full SPiceZ-Core server in one deploy.

![SPiceZ-Core](https://img.shields.io/badge/SPiceZ--Core-beta-18181b)

## Overview

A txAdmin recipe that installs every dependency and SPZ module, and ships a ready `server.cfg` and `permissions.cfg`. The schema builds itself on first boot.

## Features

- One-click txAdmin deploy
- Latest release of every module
- Ready server.cfg

## Quick start

1. txAdmin → **Server Setup** → **Remote URL Template**.
2. Paste:
   ```
   https://raw.githubusercontent.com/SPiceZ21/spz-txrecipe/main/spz-recipe.yaml
   ```
3. Enter the server name, licence key and database connection.
4. Let it run, then start the server. The database schema builds itself on first boot.

## Server requirements

| Requirement | Notes |
|---|---|
| FXServer artifact | Build `27926` or newer |
| Game build | `sv_enforceGameBuild 3407` (set in the generated `server.cfg`) |
| MySQL / MariaDB | A running instance |
| FiveM licence key | [keymaster.fivem.net](https://keymaster.fivem.net/) |

## What it installs

- **Dependencies:** `oxmysql`, `ox_lib`, `fivem-appearance` (+ thumbnail pack), `pma-voice`, `screenshot-basic`, `screencapture`, `bob74_ipl`, `vMenu`.
- **SPiceZ modules:** every module of the beta, into `resources/[spz]/`, in dependency order.
- **Configs:** `server.cfg` (load order, `sv_stateBagStrictMode`, ACE defaults) and `permissions.cfg`.

## Contents

| File | Purpose |
|---|---|
| `spz-recipe.yaml` | The txAdmin recipe |
| `server.cfg` | Server config with the full load order |
| `permissions.cfg` | ACE permission defaults |

---

Source: [https://github.com/SPiceZ21/spz-txrecipe](https://github.com/SPiceZ21/spz-txrecipe)
