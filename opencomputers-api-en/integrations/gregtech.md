# gregtech

## Summary

This page documents the GregTech and TecTech integrations exposed through OpenComputers and Computronics. The available components cover general machine state, battery buffers, digital chests, circuit-configurable machines, device sensor information, and TecTech multiblock parameters.

## Availability

- Dependency: `gregtech`
- Label: `integration-required`

## Component Names

- `gt_machine`
- `gt_batterybuffer`
- `digital_chest`
- `tt_machine`
- `bec_diode`
- `bec_io_node`
- `bec_storage`

## What This Integration Is Good For

- Reading stored EU and steam from GregTech machine tiles.
- Monitoring work progress, active state, and owner data.
- Querying or changing integrated-circuit settings on supported machines.
- Inspecting batteries in battery buffers.
- Reading digital chest contents and TecTech parameter groups.

## Components

### gt_machine

`gt_machine` is a shared component name used by several different GregTech drivers. A particular machine may expose only part of the API below, depending on which driver matched it.

#### BaseMetaTileEntity-backed methods

These callbacks come from the `BaseMetaTileEntity` driver and are available when the target is a GregTech base meta tile.

##### getEUStored()

- Syntax: `getEUStored(): number`
- Purpose: Return the EU currently stored in the block.

##### getSteamStored()

- Syntax: `getSteamStored(): number`
- Purpose: Return the steam currently stored in the block.

##### getEUMaxStored()

- Syntax: `getEUMaxStored(): number`
- Purpose: Return the maximum EU capacity of the block.

##### getSteamMaxStored()

- Syntax: `getSteamMaxStored(): number`
- Purpose: Return the maximum steam capacity of the block.

##### getEUInputAverage()

- Syntax: `getEUInputAverage(): number`
- Purpose: Return the block's average EU input.

##### getEUOutputAverage()

- Syntax: `getEUOutputAverage(): number`
- Purpose: Return the block's average EU output.

##### getOwnerName()

- Syntax: `getOwnerName(): string`
- Purpose: Return the owner name recorded by the block.

#### IMachineProgress-backed methods

These callbacks come from the `IMachineProgress` driver and are available on GregTech machines exposing work-progress data.

##### hasWork()

- Syntax: `hasWork(): boolean`
- Purpose: Return whether the machine currently has work to do.

##### getWorkProgress()

- Syntax: `getWorkProgress(): number`
- Purpose: Return the machine's current progress value.

##### getWorkMaxProgress()

- Syntax: `getWorkMaxProgress(): number`
- Purpose: Return the maximum progress value for the current work cycle.

##### isWorkAllowed()

- Syntax: `isWorkAllowed(): boolean`
- Purpose: Return whether the machine is currently allowed to work.

##### setWorkAllowed(work)

- Syntax: `setWorkAllowed(work: boolean)`
- Purpose: Enable or disable machine work permission.
- Parameters:
  - `work`: `true` to enable working, `false` to disable it.
- Returns:
  - No useful return value on success.
- Usage notes:
  - The driver only changes state when exactly one boolean argument is supplied.

##### isMachineActive()

- Syntax: `isMachineActive(): boolean`
- Purpose: Return whether the machine is currently active.

##### getCoordinates()

- Syntax: `getCoordinates(): number, number, number`
- Purpose: Return the machine coordinates.
- Returns:
  - `x, y, z` when the Computronics GregTech coordinate option is enabled.
  - An empty return list when that option is disabled.

##### getName()

- Syntax: `getName(): string`
- Purpose: Return the GregTech meta-tile name.

#### Device information method

This callback comes from the `IGregTechDeviceInformation` driver and is only exposed when the tile reports that it is currently giving information.

##### getSensorInformation()

- Syntax: `getSensorInformation(): table`
- Purpose: Return the GregTech info text lines for the machine.
- Returns:
  - The array returned by `getInfoData()`.

#### Circuit-configuration methods

These callbacks are exposed when the machine implements `IConfigurationCircuitSupport`.

##### getCircuitConfiguration()

- Syntax: `getCircuitConfiguration(): number | nil, string?`
- Purpose: Return the installed integrated-circuit configuration.
- Returns:
  - A configuration number when a valid integrated circuit is installed.
  - `-1` when the circuit slot is valid but currently empty or contains a non-circuit item.
  - `nil, "Machine does not support circuit configuration."` when the machine does not support this feature.
  - `nil, "Invalid circuit slot index."` when the reported circuit slot is outside the inventory range.

##### setCircuitConfiguration(config)

- Syntax: `setCircuitConfiguration(config: number): boolean | nil, string`
- Purpose: Install, replace, or remove an integrated circuit.
- Parameters:
  - `config`: `1` through `24` to install a circuit with that setting, or `-1` to remove the installed circuit.
- Returns:
  - `true` on success.
  - `nil, "Machine does not support circuit configuration."` when unsupported.
  - `nil, "Invalid circuit slot index."` when the reported slot is invalid.
  - `nil, "Invalid circuit configuration value: ..."` when the value is outside `1..24` and not `-1`.

### gt_batterybuffer

`gt_batterybuffer` is exposed on GregTech basic battery buffers.

#### getBatteryCharge(slot)

- Syntax: `getBatteryCharge(slot: number): number | nil, string?`
- Purpose: Return the EU currently stored in the battery located in the specified slot.
- Parameters:
  - `slot`: one-based slot index.
- Returns:
  - The current charge for an electric item.
  - `nil, "slot does not exist"` when the slot index is out of range.
  - `nil, "slot is empty"` when the slot is empty.
  - `nil, "item in slot is not electric"` when the item is not recognized as an electric item.

#### getMaxBatteryCharge(slot)

- Syntax: `getMaxBatteryCharge(slot: number): number | nil, string?`
- Purpose: Return the maximum EU capacity of the battery located in the specified slot.
- Parameters:
  - `slot`: one-based slot index.
- Returns:
  - The maximum charge for an electric item.
  - `nil, "slot does not exist"` when the slot index is out of range.
  - `nil, "slot is empty"` when the slot is empty.
  - `nil, "item in slot is not electric"` when the item is not recognized as an electric item.

### digital_chest

`digital_chest` is exposed on GregTech digital chests.

#### getContents()

- Syntax: `getContents(): table`
- Purpose: Return the digital chest's stored-item data table.
- Returns:
  - The raw structure returned by `getStoredItemData()`.
- Usage notes:
  - This is the chest's own storage descriptor, so its field layout follows GregTech's digital-chest data format.

### tt_machine

`tt_machine` is exposed on TecTech multiblocks derived from `TTMultiblockBase`.

#### getParameters(hatch, id)

- Syntax: `getParameters(hatch: number, id: number): number`
- Purpose: Return the current parameter value for one TecTech parameter input slot.
- Parameters:
  - `hatch`: parameter group index.
  - `id`: parameter index within that group.

#### setParameters(hatch, id, value)

- Syntax: `setParameters(hatch: number, id: number, value: number)`
- Purpose: Write a new parameter value into one TecTech parameter slot.
- Parameters:
  - `hatch`: parameter group index.
  - `id`: parameter index within that group.
  - `value`: new floating-point value.
- Returns:
  - No useful return value on success.

### `bec_diode`

`bec_diode` is exposed on TecTech Bose-Einstein condensate diode meta tiles.

#### `getCondensateFilter()`

- Syntax: `getCondensateFilter(): string | nil`
- Purpose: Return the fluid registry name used as the condensate filter, or `nil` when no filter is set.

#### `setCondensateFilter(fluidName)`

- Syntax: `setCondensateFilter(fluidName: string | nil)`
- Purpose: Set the condensate filter by fluid registry name. Passing `nil` clears the filter.
- Errors:
  - Unknown fluid names raise `Unknown fluid: ...`.

### `bec_io_node`

`bec_io_node` is exposed on TecTech BEC input/output nodes.

#### Read-only recipe state

- `getRequiredCondensate(): table | nil`: map fluid registry names to amounts required by the current recipe.
- `getConsumedCondensate(): table | nil`: map fluid registry names to amounts consumed so far.
- `getProvidedTier(): table | nil`: return `{name, tier}` for the nanite tier provided by this node.
- `getRequiredTier(): table | nil`: return `{name, tier}` for the current recipe's required tier.
- `getAvailableNanites(): number`: return available nanites.
- `getSlowdowns(): number`: return the number of applied slowdowns.
- `getParallelRecipesInProgress(): number`: return the number of parallel recipes in progress.
- `getMinParallel(): number`: return the configured minimum parallel recipe count.
- `getMaxParallel(): number`: return the configured maximum parallel recipe count.
- `getManualSlowdown(): number`: return the configured manual slowdown.
- `getRecipeSteps(): table | nil`: return recipe step tables with `nanite`, `start`, `end`, and `index` fields.
- `getState(): string`: return `idle`, `unpowered`, `assembler-offline`, `nanite-tier-too-low`, `paused-step`, `paused-immediate`, `crafting`, or `internal-error`.

#### `setMinParallel(min)`

- Syntax: `setMinParallel(min: number)`
- Purpose: Set the minimum parallel recipe count.

#### `setMaxParallel(max)`

- Syntax: `setMaxParallel(max: number)`
- Purpose: Set the maximum parallel recipe count.

#### `setSpeedDivisor(divisor)`

- Syntax: `setSpeedDivisor(divisor: number)`
- Purpose: Set the recipe speed divisor.

Condensate tables map fluid registry names to numeric amounts. Nanite tier tables contain `name` and numeric `tier`; recipe step tables contain the nanite tier plus `start`, `end`, and `index`.

### `bec_storage`

`bec_storage` is exposed on TecTech BEC storage nodes.

#### `getFieldStrength()`

- Syntax: `getFieldStrength(): number`
- Purpose: Return the storage node field strength.

#### `setFieldStrength(strength)`

- Syntax: `setFieldStrength(strength: number)`
- Purpose: Set the storage node field strength.

#### `getStoredCondensate()`

- Syntax: `getStoredCondensate(): table`
- Purpose: Return a table mapping fluid registry names to stored condensate amounts.

## Example

```lua
local component = require("component")

if component.isAvailable("gt_machine") then
  local machine = component.gt_machine
  print("Stored EU:", machine.getEUStored())
  print("Machine active:", machine.isMachineActive and machine.isMachineActive())
end

if component.isAvailable("gt_batterybuffer") then
  local buffer = component.gt_batterybuffer
  local charge, err = buffer.getBatteryCharge(1)
  print("Slot 1:", charge or err)
end
```

## Related

- `ic2`
- `gregtech`
