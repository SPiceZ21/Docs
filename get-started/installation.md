# Installation Guide

## 1. Prerequisites
- FiveM Server (latest recommended)
- `oxmysql`
- `ox_lib`
- `fivem-appearance`

## 2. Load Order
SPiceZ-Core is highly modular, so load order is critical. Add the following to your `server.cfg`:

```cfg
# Dependencies
ensure oxmysql
ensure ox_lib
ensure fivem-appearance

# Core
ensure spz-core
ensure spz-identity
ensure spz-appearance
ensure spz-spawn

# Modules
ensure spz-vehicles
ensure spz-races
...
```

The [txAdmin recipe](https://github.com/SPiceZ21/spz-txrecipe) writes this
`server.cfg` for you, in full and in the right order.

## 3. Database Setup
Point `oxmysql` at your database before starting the modules — that is the only
setup step. There are no `.sql` files to import: `spz-core` owns the schema and
applies everything in its `migrations/` directory on first boot, recording each
file in the `spz_migrations` table so restarts and upgrades are no-ops.
