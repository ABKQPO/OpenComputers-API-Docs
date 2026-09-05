# openmodularturrets

## Summary

This page documents the OpenComputers integration of Open Modular Turrets. Each turret base tier exposes the same control API under a tier-specific component name.

## Availability

- Dependency: `openmodularturrets` and `OpenComputers`
- Additional requirement: the turret base must contain a Serial Port Addon.
- Label: `integration-required`

## Component Names

- `tierOneTurretBase`
- `tierTwoTurretBase`
- `tierThreeTurretBase`
- `tierFourTurretBase`
- `tierFiveTurretBase`

All five components expose the callbacks below. The tier affects the base's inventory, energy capacity, and turret capabilities, but not the Lua method names.

## Access Gate

The callbacks are enabled only when OpenComputers is loaded and the base has a Serial Port Addon. When computer access is disabled, the driver returns the string `Computer access deactivated!` as the first result instead of performing the requested operation.

## Callbacks

### `getOwner()`

- Syntax: `getOwner(): string`
- Purpose: Return the name of the player recorded as the turret base owner.

### `isAttacksMobs()`

- Syntax: `isAttacksMobs(): boolean`
- Purpose: Return whether the base is configured to attack hostile mobs.

### `setAttacksMobs(value)`

- Syntax: `setAttacksMobs(value: boolean)`
- Purpose: Enable or disable attacks against hostile mobs.
- Returns: No useful value on success.

### `isAttacksNeutrals()`

- Syntax: `isAttacksNeutrals(): boolean`
- Purpose: Return whether the base is configured to attack neutral mobs.

### `setAttacksNeutrals(value)`

- Syntax: `setAttacksNeutrals(value: boolean)`
- Purpose: Enable or disable attacks against neutral mobs.
- Returns: No useful value on success.

### `isAttacksPlayers()`

- Syntax: `isAttacksPlayers(): boolean`
- Purpose: Return whether the base is configured to attack players.

### `setAttacksPlayers(value)`

- Syntax: `setAttacksPlayers(value: boolean)`
- Purpose: Enable or disable attacks against players.
- Returns: No useful value on success.

### `getTrustedPlayers()`

- Syntax: `getTrustedPlayers(): table`
- Purpose: Return the trusted-player list.
- Returns:
  - A table containing trusted-player records.
  - Records expose the public fields `name`, `canOpenGUI`, `canChangeTargeting`, `admin`, and `uuid` when the runtime converts them to Lua.

### `addTrustedPlayer(name[, canOpenGUI[, canChangeTargeting[, admin]]])`

- Syntax: `addTrustedPlayer(name: string[, canOpenGUI: boolean[, canChangeTargeting: boolean[, admin: boolean]]])`
- Purpose: Add a player to the trusted-player list and optionally set permissions.
- Parameters:
  - `name`: Player name. The player must be valid according to the server's online/offline-mode rules.
  - `canOpenGUI`: Whether the player may open the base GUI. Defaults to `false`.
  - `canChangeTargeting`: Whether the player may change targeting settings. Defaults to `false`.
  - `admin`: Whether the player is marked as an administrator. Defaults to `false`.
- Returns:
  - No useful value on success.
  - `"Name not valid!"` when the player cannot be added.
- Implementation note: In the current OpenComputers callback implementation, all three optional permission fields are read from argument 2. Passing separate values for `canChangeTargeting` or `admin` does not currently change those fields.

### `removeTrustedPlayer(name)`

- Syntax: `removeTrustedPlayer(name: string)`
- Purpose: Remove a player from the trusted-player list.
- Parameters:
  - `name`: Exact player name to remove.
- Returns: No useful value, whether or not a matching entry existed.

### `getMaxEnergyStorage()`

- Syntax: `getMaxEnergyStorage(): number`
- Purpose: Return the maximum energy storage of the turret base, including configured power expanders.

### `getCurrentEnergyStorage()`

- Syntax: `getCurrentEnergyStorage(): number`
- Purpose: Return the energy currently stored in the turret base.

### `getActive()`

- Syntax: `getActive(): boolean`
- Purpose: Return whether the turret base is active.

### `setInverted(value)`

- Syntax: `setInverted(value: boolean)`
- Purpose: Set whether the base's redstone control is inverted.
- Returns: No useful value on success.

### `getInverted()`

- Syntax: `getInverted(): boolean`
- Purpose: Return whether redstone control is inverted.

### `getRedstone()`

- Syntax: `getRedstone(): boolean`
- Purpose: Return the current redstone state observed by the base.

## Example

```lua
local component = require("component")

local name = "tierOneTurretBase"
if component.isAvailable(name) then
  local base = component[name]
  print("Owner:", base.getOwner())
  print("Energy:", base.getCurrentEnergyStorage(), "/", base.getMaxEnergyStorage())
  base.setAttacksMobs(true)
  base.setInverted(false)
end
```

## Related

- `opencomputers`
- `gregtech`
