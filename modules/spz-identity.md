# spz-identity — Player Profiles & Licensing

> **Version**: `v1.4.0`  
> **SPiceZ-Core** | Identity module  
> Manages player profiles, driver license tiers, crew membership, and character initialization.

## 1. Overview
`spz-identity` is the player data layer for SPiceZ-Core. It manages the lifecycle of a player's profile from the first connection to character creation and persistent storage.

## 2. The Connection & Creation Flow
This is the most critical part of the identity module. It differentiates between returning players and new racers.

### 2.1 Connection Flow
1. **Connect**: `spz-core` fires `SPZ:playerConnected`.
2. **Lookup**: `spz-identity` checks the database for a matching Rockstar License.
3. **Branching Logic**:
    *   **Existing Player**: If a profile is found and `first_time = 0`, it warms the cache and fires `SPZ:playerReady`.
    *   **New Player**: If no profile exists, `CreateProfile` is called. The profile is initialized with `first_time = 1`.
4. **First-Time Detection**: If `first_time == 1`, the server:
    *   Fires `SPZ:openCharacterCreation` on the client.
    *   **Wait**: It does NOT fire `SPZ:playerReady` yet.

### 2.2 Character Creation
The player is presented with a UI to choose their:
- **Gender** (Male/Female)
- **Username** (Validated for uniqueness)

Once submitted, the server updates the profile:
- Sets `username` and `gender`.
- Sets `first_time = 0`.
- Fires `SPZ:characterReady` and `SPZ:playerReady`.

## 3. Database Schema
### `players` table
| Column | Type | Default | Description |
|---|---|---|---|
| `id` | INT | Auto Inc | Primary Key |
| `username` | VARCHAR | NULL | Unique Racer Name |
| `gender` | INT | NULL | 0=Male, 1=Female |
| `first_time` | INT | 1 | Flag for new players |
| `license_tier`| INT | 0 | 0=C, 1=B, 2=A, 3=S |

## 4. Events Reference
| Event | Side | Description |
|---|---|---|
| `SPZ:openCharacterCreation` | Client | Triggered for new players to open the NUI form. |
| `SPZ:characterCreated` | Server | Callback from NUI with gender and username. |
| `SPZ:characterReady` | Server | Fired after profile initialization is complete. |
| `SPZ:playerReady` | Server | Global signal that player data is safe to use. |
