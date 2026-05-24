# draconicevolution

## Summary

This page documents the OpenComputers integration implemented directly inside Draconic Evolution. The mod exposes several different block types as components instead of a single shared adapter, so the available methods depend on which Draconic block you connect to.

## Availability

- Dependency: `draconicevolution`
- Label: `integration-required`

## Component Names

- `particle_generator`
- `draconic_rf_storage`
- `flux_gate`
- `fluid_gate`
- `draconic_reactor`

## What This Integration Is Good For

- Reading Draconic Evolution energy storage values through pylons.
- Tuning particle and beam effects on particle generators from Lua.
- Driving Flux Gates and Fluid Gates from software instead of only redstone.
- Monitoring and controlling a Draconic Reactor from a stabilizer or injector.

## Components

### particle_generator

`particle_generator` is exposed by the Draconic Evolution particle generator block.

#### `setGeneratorProperty(property, value)`

- Syntax: `setGeneratorProperty(property: string, value: boolean|number): boolean`
- Purpose: Change one generator setting by name.
- Parameters:
  - `property: string`
    Internal property key to modify.
  - `value: boolean|number`
    New value for the selected property. The required type depends on the property.
- Returns:
  - `true` if the property name and value type were accepted.
  - `false` if the property name is unknown, the type is wrong, or the call did not pass exactly two arguments.
- Usage notes:
  - Numeric arguments are handled as Lua numbers and converted to integers or floats as needed.
  - Every accepted change is applied immediately and synced to the block.
  - Invalid names do not partially update state.

Supported particle properties:

- `particles_enabled: boolean`
  Enable or disable particle spawning entirely.
- `red`, `green`, `blue: number`
  Base RGB color channels, clamped to `0..255`.
- `random_red`, `random_green`, `random_blue: number`
  Random RGB additions, clamped to `0..255`.
- `motion_x`, `motion_y`, `motion_z: number`
  Base motion vector, clamped to `-5..5`.
- `random_motion_x`, `random_motion_y`, `random_motion_z: number`
  Random motion offsets, clamped to `-5..5`.
- `scale: number`
  Base particle size, clamped to `0.01..50`.
- `random_scale: number`
  Random extra size, clamped to `0.01..50`.
- `life: number`
  Base lifetime in ticks, clamped to `0..1000`.
- `random_life: number`
  Random extra lifetime in ticks, clamped to `0..1000`.
- `spawn_x`, `spawn_y`, `spawn_z: number`
  Base spawn offset from the block center, clamped to `-50..50`.
- `random_spawn_x`, `random_spawn_y`, `random_spawn_z: number`
  Random spawn offsets, clamped to `-50..50`.
- `fade: number`
  Fade time, clamped to `0..100`.
- `spawn_rate: number`
  Delay between spawn attempts, clamped to `1..200`.
- `collide: boolean`
  Enable or disable particle collision.
- `selected_particle: number`
  Particle preset index, clamped to `1..3`.
- `gravity: number`
  Gravity applied to particles, clamped to `-5..5`.

Supported beam properties:

- `beam_enabled: boolean`
  Enable or disable the beam renderer.
- `render_core: boolean`
  Enable or disable the beam core rendering.
- `beam_red`, `beam_green`, `beam_blue: number`
  Beam RGB channels, clamped to `0..255`.
- `beam_scale: number`
  Beam width multiplier, clamped to `0..5`.
- `beam_pitch: number`
  Vertical beam angle, clamped to `-180..180`.
- `beam_yaw: number`
  Horizontal beam angle, clamped to `-180..180`.
- `beam_length: number`
  Beam length, clamped to `0..320`.
- `beam_rotation: number`
  Beam texture rotation speed, clamped to `-1..1`.

#### `getGeneratorState()`

- Syntax: `getGeneratorState(): table`
- Purpose: Return the full current generator configuration as a key-value table.
- Returns:
  - A table containing every property accepted by `setGeneratorProperty`.
- Usage notes:
  - The returned keys are exactly the internal keys listed above.
  - This is the easiest way to snapshot the current generator setup before changing it.

#### `resetGeneratorState()`

- Syntax: `resetGeneratorState(): boolean`
- Purpose: Restore all generator settings to their default values.
- Returns:
  - `true` after the reset is applied.
- Usage notes:
  - This resets both particle and beam settings.
  - After a reset the most important defaults are:
    `particles_enabled=true`, `scale=1`, `life=100`, `spawn_rate=1`, `selected_particle=1`, `beam_enabled=false`, `beam_scale=1`.

#### Example

```lua
local component = require("component")
local gen = component.particle_generator

assert(gen.resetGeneratorState())
assert(gen.setGeneratorProperty("particles_enabled", true))
assert(gen.setGeneratorProperty("red", 32))
assert(gen.setGeneratorProperty("green", 180))
assert(gen.setGeneratorProperty("blue", 255))
assert(gen.setGeneratorProperty("spawn_y", 0.75))
assert(gen.setGeneratorProperty("motion_y", 0.03))
assert(gen.setGeneratorProperty("life", 40))

local state = gen.getGeneratorState()
print("Particle preset:", state.selected_particle)
print("RGB:", state.red, state.green, state.blue)
```

### draconic_rf_storage

`draconic_rf_storage` is exposed by the Draconic Evolution energy pylon when it is linked to an Energy Storage Core. The component name comes from the tile implementation itself and is not `draconic_storage`.

#### `getEnergyStored()`

- Syntax: `getEnergyStored(): number`
- Purpose: Return the current amount of stored energy in the linked core.
- Returns:
  - Current stored energy as a whole number.
- Usage notes:
  - If the pylon is not linked to a storage core, this returns `0`.
  - The pylon forwards the value from the linked storage core, so capacity can exceed 32-bit RF values.

#### `getMaxEnergyStored()`

- Syntax: `getMaxEnergyStored(): number`
- Purpose: Return the total capacity of the linked storage core.
- Returns:
  - Maximum stored energy as a whole number.
- Usage notes:
  - If the pylon is not linked to a storage core, this returns `0`.

#### Example

```lua
local component = require("component")
local pylon = component.draconic_rf_storage

local stored = pylon.getEnergyStored()
local capacity = pylon.getMaxEnergyStored()

print(("Stored: %d / %d"):format(stored, capacity))
if capacity > 0 then
  print(("Fill: %.2f%%"):format(stored / capacity * 100))
end
```

### flux_gate

`flux_gate` is exposed by the Flux Gate block.

#### `getFlow()`

- Syntax: `getFlow(): number`
- Purpose: Return the flow value currently being used by the gate.
- Returns:
  - The active transfer limit in RF/t.
- Usage notes:
  - If override mode is enabled, this returns the override value.
  - Otherwise it linearly interpolates between the low-flow and high-flow settings from the current redstone strength `0..15`.

#### `setOverrideEnabled(enabled)`

- Syntax: `setOverrideEnabled(enabled: boolean)`
- Purpose: Enable or disable direct software override mode.
- Parameters:
  - `enabled: boolean`
    `true` to use the override flow value, `false` to use the redstone-derived flow.
- Returns:
  - No values.

#### `getOverrideEnabled()`

- Syntax: `getOverrideEnabled(): boolean`
- Purpose: Return whether override mode is active.
- Returns:
  - `true` if the gate currently uses the override flow value.
  - `false` if the gate currently uses redstone interpolation.

#### `setFlowOverride(flow)`

- Syntax: `setFlowOverride(flow: number)`
- Purpose: Set the direct software-controlled flow value.
- Parameters:
  - `flow: number`
    Desired RF/t limit used while override mode is enabled.
- Returns:
  - No values.
- Usage notes:
  - The implementation does not clamp this value.
  - Use non-negative values to avoid undefined transfer behavior.

#### `setSignalHighFlow(flow)`

- Syntax: `setSignalHighFlow(flow: number)`
- Purpose: Set the flow used when redstone input is `15`.
- Parameters:
  - `flow: number`
    Desired high-signal RF/t limit.
- Returns:
  - No values.

#### `getSignalHighFlow()`

- Syntax: `getSignalHighFlow(): number`
- Purpose: Return the configured high-signal flow.
- Returns:
  - Current high-signal RF/t value.

#### `setSignalLowFlow(flow)`

- Syntax: `setSignalLowFlow(flow: number)`
- Purpose: Set the flow used when redstone input is `0`.
- Parameters:
  - `flow: number`
    Desired low-signal RF/t limit.
- Returns:
  - No values.

#### `getSignalLowFlow()`

- Syntax: `getSignalLowFlow(): number`
- Purpose: Return the configured low-signal flow.
- Returns:
  - Current low-signal RF/t value.

#### Example

```lua
local component = require("component")
local gate = component.flux_gate

gate.setSignalLowFlow(0)
gate.setSignalHighFlow(50000)
gate.setOverrideEnabled(true)
gate.setFlowOverride(12000)

print("Override active:", gate.getOverrideEnabled())
print("Current flow:", gate.getFlow(), "RF/t")
```

### fluid_gate

`fluid_gate` is exposed by the Fluid Gate block.

#### `getFlow()`

- Syntax: `getFlow(): number`
- Purpose: Return the flow value currently being used by the gate.
- Returns:
  - The active transfer limit in mB/t.
- Usage notes:
  - The control logic is the same as `flux_gate`, but the unit is millibuckets per tick.

#### `setOverrideEnabled(enabled)`

- Syntax: `setOverrideEnabled(enabled: boolean)`
- Purpose: Enable or disable direct software override mode.
- Parameters:
  - `enabled: boolean`
    `true` to use the override flow value, `false` to use the redstone-derived flow.
- Returns:
  - No values.

#### `getOverrideEnabled()`

- Syntax: `getOverrideEnabled(): boolean`
- Purpose: Return whether override mode is active.
- Returns:
  - `true` if the gate currently uses the override flow value.
  - `false` if the gate currently uses redstone interpolation.

#### `setFlowOverride(flow)`

- Syntax: `setFlowOverride(flow: number)`
- Purpose: Set the direct software-controlled flow value.
- Parameters:
  - `flow: number`
    Desired mB/t limit used while override mode is enabled.
- Returns:
  - No values.
- Usage notes:
  - Use non-negative values. The tile implementation does not validate them.

#### `setSignalHighFlow(flow)`

- Syntax: `setSignalHighFlow(flow: number)`
- Purpose: Set the flow used when redstone input is `15`.
- Parameters:
  - `flow: number`
    Desired high-signal mB/t limit.
- Returns:
  - No values.

#### `getSignalHighFlow()`

- Syntax: `getSignalHighFlow(): number`
- Purpose: Return the configured high-signal flow.
- Returns:
  - Current high-signal mB/t value.

#### `setSignalLowFlow(flow)`

- Syntax: `setSignalLowFlow(flow: number)`
- Purpose: Set the flow used when redstone input is `0`.
- Parameters:
  - `flow: number`
    Desired low-signal mB/t limit.
- Returns:
  - No values.

#### `getSignalLowFlow()`

- Syntax: `getSignalLowFlow(): number`
- Purpose: Return the configured low-signal flow.
- Returns:
  - Current low-signal mB/t value.

#### Example

```lua
local component = require("component")
local gate = component.fluid_gate

gate.setOverrideEnabled(false)
gate.setSignalLowFlow(0)
gate.setSignalHighFlow(4000)

print("Low signal flow:", gate.getSignalLowFlow(), "mB/t")
print("High signal flow:", gate.getSignalHighFlow(), "mB/t")
```

### draconic_reactor

`draconic_reactor` is exposed by reactor stabilizers and reactor energy injectors. Both block types forward the same four methods to the linked reactor core.

#### `getReactorInfo()`

- Syntax: `getReactorInfo(): table`
- Purpose: Return the reactor's current operating state as a table.
- Returns:
  - A table with the following keys:
    - `temperature`
      Reactor temperature rounded to two decimal places.
    - `fieldStrength`
      Current shield field value rounded to two decimal places.
    - `maxFieldStrength`
      Maximum shield field value rounded to two decimal places.
    - `energySaturation`
      Current internal energy saturation.
    - `maxEnergySaturation`
      Maximum internal energy saturation.
    - `fuelConversion`
      Converted fuel amount rounded to three decimal places.
    - `maxFuelConversion`
      Total fuel inventory the reactor can still convert in the current run.
    - `generationRate`
      Current RF/t generation.
    - `fieldDrainRate`
      Current RF/t shield drain.
    - `fuelConversionRate`
      Current fuel conversion rate multiplied by `1,000,000`.
    - `status`
      One of `offline`, `charging`, `charged`, `online`, `stopping`, or `invalid`.
- Usage notes:
  - If the stabilizer or injector is not linked to a reactor core, the component call returns `nil`.
  - `charging` and `charged` are both reported while the reactor is in the startup phase.

#### `chargeReactor()`

- Syntax: `chargeReactor(): boolean`
- Purpose: Put the reactor into its startup charging phase.
- Returns:
  - `true` if the reactor accepted the request.
  - `false` if the reactor is not in a state where charging can begin.
- Usage notes:
  - Charging is only allowed when the structure is valid, the reactor is not already online, and at least `144` fuel units are loaded.

#### `activateReactor()`

- Syntax: `activateReactor(): boolean`
- Purpose: Bring a charged reactor online.
- Returns:
  - `true` if the reactor entered the online state.
  - `false` if startup requirements are not yet met.
- Usage notes:
  - Activation requires a valid structure, at least half field charge, at least half energy saturation, temperature at or above `2000`, and at least `144` total fuel units.

#### `stopReactor()`

- Syntax: `stopReactor(): boolean`
- Purpose: Start the controlled reactor shutdown sequence.
- Returns:
  - `true` if the reactor accepted the stop request.
  - `false` if the reactor cannot be stopped right now.
- Usage notes:
  - This requests the safe stopping state. It is not an instant power-off.

#### Example

```lua
local component = require("component")
local reactor = component.draconic_reactor

local info = reactor.getReactorInfo()
if not info then
  error("No linked reactor core")
end

print("Status:", info.status)
print("Temperature:", info.temperature)
print("Shield:", info.fieldStrength, "/", info.maxFieldStrength)

if info.status == "charged" then
  assert(reactor.activateReactor())
end
```

## Related

- `mekanism`
- `cofh`
