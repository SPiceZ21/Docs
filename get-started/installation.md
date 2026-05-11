# Installation Guide

## 1. Prerequisites
- FiveM Server (latest recommended)
- `oxmysql`
- `ox_lib`

## 2. Load Order
SPiceZ-Core is highly modular, so load order is critical. Add the following to your `server.cfg`:

```cfg
# Dependencies
ensure oxmysql
ensure ox_lib

# Core
ensure spz-lib
ensure spz-core

# Modules
ensure spz-identity
ensure spz-spawn
ensure spz-vehicles
ensure spz-races
...
```

## 3. Database Setup
Each module typically comes with its own `.sql` file or automated migrations. Ensure `oxmysql` is configured before starting the modules.
