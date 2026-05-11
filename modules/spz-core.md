# spz-core — Framework Bootstrap

> **Version**: `v2.0.0`  
> **SPiceZ-Core** | Main Orchestrator  
> Manages sessions, routing buckets, and the global state machine.

## 1. Overview
`spz-core` is the heartbeat of the framework. It handles the initial `playerConnecting` event and manages the "Session" objects that other modules use.

## 2. Key Features
- **Session Management**: Tracks every connected player with unique session data.
- **Routing Buckets**: Isolates race worlds from the freeroam world automatically.
- **State Machine**: Tracks if a player is in the `MENU`, `FREEROAM`, or `RACING` state.

## 3. Exports Reference
### `GetPlayerSession(source)`
Returns the session table for a player.
```lua
local session = exports["spz-core"]:GetPlayerSession(source)
```

### `AssignPlayerToBucket(source, bucketId)`
Safely moves a player to a routing bucket and handles cleanup of nearby entities.

## 4. Global Events
- `SPZ:coreReady`: Fired when the core has finished initializing.
- `SPZ:playerConnected`: Fired when a player is deferral-cleared and a session is created.
- `SPZ:playerDisconnected`: Cleanup event for sessions.
