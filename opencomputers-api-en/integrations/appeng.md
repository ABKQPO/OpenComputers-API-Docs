# appeng

## Summary

This page documents the OpenComputers API exposed by the Applied Energistics 2 integration.
It covers stationary ME network components, bus and interface configuration, interface terminal pattern transfer, crafting control, network monitoring, and the robot or drone AE upgrade.

## Availability

- Dependency: `appeng`
- Label: `integration-required`

## Component Names

- `me_controller`
- `me_exportbus`
- `me_importbus`
- `me_storagebus`
- `me_interface`
- `me_interface_terminal`
- `upgrade_me`

## Data Shapes Used By This Integration

### Item stack tables

Most item-oriented callbacks return or accept item tables based on the standard OpenComputers item converter.
Common fields include:

- `name`: registry name such as `minecraft:iron_ingot`
- `label`: localized display name
- `damage`: metadata / damage value
- `size`: stack size
- `maxSize`: maximum stack size
- `maxDamage`: maximum durability
- `hasTag`: whether the stack has NBT
- `id`: numeric item id when OC is configured to include ids
- `oreNames`: ore dictionary names when OC is configured to include them
- `enchantments`, `lore`, `tag` when present and allowed by OC settings

AE item tables also include:

- `isCraftable`: whether the entry represents a craftable target

### Fluid tables

Common fluid fields include:

- `name`: internal fluid name such as `water`
- `label`: localized fluid name
- `amount`: millibuckets
- `hasTag`: whether the fluid stack has NBT
- `id`: numeric fluid id when OC is configured to include ids

AE fluid tables also include:

- `size`: AE stack size
- `isCraftable`: whether the entry represents a craftable target

### Location tables

The interface terminal uses location tables with these fields:

- `x`
- `y`
- `z`
- `dimId` optional, defaults to the terminal dimension

### Pattern tables

`getInterfacePattern` returns the encoded pattern item stack.
Its converted data typically includes:

- `inputs`: array of input item stacks
- `outputs`: array of output item stacks
- `isCraftable`: whether the pattern is a crafting pattern

## Shared Network Control

These callbacks are available on `me_controller` and on `upgrade_me` while it is successfully linked to an AE network.

### `getCpus`

- Syntax: `me.getCpus()`
- Returns: array of CPU descriptor tables
- Purpose: lists all AE crafting CPUs visible to the network

Each descriptor usually contains:

- `name`
- `storage`
- `coprocessors`
- `busy`
- `cpu`: userdata handle for detailed CPU inspection

Example:

```lua
for _, info in ipairs(component.me_controller.getCpus()) do
  print(info.name, info.storage, info.busy)
end
```

### `getCraftables`

- Syntax: `me.getCraftables([filter])`
- Parameters:
  - `filter`: optional table matched against converted craftable output fields
- Returns: array of `Craftable` userdata values
- Purpose: lists recipes that the network knows how to craft

The filter is exact-match on converted fields after string or numeric normalization.
Useful keys are usually `name`, `label`, `damage`, and sometimes `id`.

Example:

```lua
local craftables = component.me_controller.getCraftables({label = "Printed Silicon"})
print(#craftables)
```

### `getCraftable`

- Syntax: `me.getCraftable(detail, type)`
- Parameters:
  - `detail`: descriptor table for one AE stack identity
  - `type`: registered AE stack type id
- Returns: `Craftable` userdata or `nil`
- Purpose: resolves one known craftable recipe target directly

Usage notes:

- The source parses `detail` through the registered AE stack parser for the supplied `type`.
- In normal AE2 item networks, `type` is typically the item stack type id.
- Additional integrations can register other AE stack types, so this callback is more general than a plain item-only lookup.

Example:

```lua
local craftable = component.me_controller.getCraftable(
  {name = "minecraft:glass", damage = 0, size = 1},
  "item"
)
```

### `getItemsInNetwork`

- Syntax: `me.getItemsInNetwork([filter])`
- Parameters:
  - `filter`: optional item filter table
- Returns: array of converted item tables
- Purpose: lists stored items currently present in the network

Example:

```lua
local items = component.me_controller.getItemsInNetwork({name = "minecraft:iron_ingot"})
for _, stack in ipairs(items) do
  print(stack.label, stack.size)
end
```

### `getItemInNetwork`

- Syntax: `me.getItemInNetwork(nameOrId[, damage[, nbtJson]])`
- Syntax: `me.getItemInNetwork(detail)`
- Parameters:
  - `nameOrId`: registry name string or numeric item id
  - `damage`: optional metadata, defaults to `0`
  - `nbtJson`: optional SNBT string, defaults to `{}`
  - `detail`: item descriptor table parsed through the registered AE item stack parser
- Returns: converted item table or `nil`
- Purpose: looks up one exact stored item variant

Usage notes:

- The string/id form builds a temporary `ItemStack` and parses the third argument with Minecraft's SNBT parser.
- The table form is the current source-backed path for callers that already have a converted descriptor table.
- Exact lookups therefore no longer need to be expressed only as name plus damage plus NBT text.

Example:

```lua
local stack = component.me_controller.getItemInNetwork("minecraft:wool", 14)
if stack then
  print(stack.label, stack.size)
end
```

```lua
local exact = component.me_controller.getItemInNetwork({
  name = "minecraft:wool",
  damage = 14,
  size = 1
})
```

### `getItemsInNetworkById`

- Syntax: `me.getItemsInNetworkById(filterIds)`
- Parameters:
  - `filterIds`: array of registry names or numeric item ids
- Returns: array of converted item tables
- Purpose: fetches all stored item entries whose base item matches one of the supplied ids

This filters by item type only, not by metadata or NBT.

Example:

```lua
local items = component.me_controller.getItemsInNetworkById({
  "minecraft:cobblestone",
  "minecraft:stone"
})
```

### `getFluidInNetwork`

- Syntax: `me.getFluidInNetwork([name])`
- Syntax: `me.getFluidInNetwork(detail)`
- Parameters:
  - `name`: optional fluid name such as `water`
  - `detail`: fluid descriptor table parsed through the registered AE fluid stack parser
- Returns: converted fluid table or `nil`
- Purpose: looks up one fluid type in the network

Usage notes:

- The descriptor-table form mirrors the item lookup path and is the current source-backed way to do exact fluid lookups from previously converted values.
- If the named fluid does not exist, the callback returns `nil`.

Example:

```lua
local water = component.me_controller.getFluidInNetwork("water")
print(water and water.amount or 0)
```

```lua
local fluid = component.me_controller.getFluidInNetwork({name = "water", amount = 1})
```

### `allItems`

- Syntax: `me.allItems()`
- Returns: iterator userdata
- Purpose: iterates through network items without building one huge Lua table up front

The iterator returns one converted item entry per call until it returns `nil`.
This is useful on very large AE networks.

Example:

```lua
local it = component.me_controller.allItems()
while true do
  local stack = it()
  if not stack then break end
  print(stack.label, stack.size)
end
```

### `store`

- Syntax: `me.store(filter, databaseAddress[, startSlot[, count]])`
- Parameters:
  - `filter`: item filter table
  - `databaseAddress`: target database component address
  - `startSlot`: optional one-based first database slot, defaults to `1`
  - `count`: optional maximum number of entries to copy
- Returns: `true`
- Purpose: copies matching item descriptors from the network into a database

The method skips occupied database slots until it finds empty ones.

Example:

```lua
local ok = component.me_controller.store(
  {label = "Iron Ingot"},
  database.address,
  1,
  8
)
```

### `getFluidsInNetwork`

- Syntax: `me.getFluidsInNetwork()`
- Returns: array of converted fluid tables
- Purpose: lists fluids visible to the AE network

Some hidden or virtual fluid entries are filtered out by compatibility helpers.

Example:

```lua
for _, fluid in ipairs(component.me_controller.getFluidsInNetwork()) do
  print(fluid.label, fluid.amount)
end
```

### `getAvgPowerInjection`

- Syntax: `me.getAvgPowerInjection()`
- Returns: number
- Purpose: reports average AE power input

### `getAvgPowerUsage`

- Syntax: `me.getAvgPowerUsage()`
- Returns: number
- Purpose: reports average AE power consumption

### `getIdlePowerUsage`

- Syntax: `me.getIdlePowerUsage()`
- Returns: number
- Purpose: reports idle AE power draw

### `getMaxStoredPower`

- Syntax: `me.getMaxStoredPower()`
- Returns: number
- Purpose: reports AE network energy capacity

### `getStoredPower`

- Syntax: `me.getStoredPower()`
- Returns: number
- Purpose: reports currently stored AE energy

Example:

```lua
local me = component.me_controller
print(me.getStoredPower(), "/", me.getMaxStoredPower())
```

### `setFluidEventSubscription`

- Syntax: `me.setFluidEventSubscription(enabled)`
- Parameters:
  - `enabled`: boolean
- Returns: nothing
- Purpose: subscribes or unsubscribes from `network_fluid_changed`

When enabled, the component sends a computer signal whose first argument is the event name and whose remaining arguments are converted fluid entries that changed.

### `isFluidEventSubscription`

- Syntax: `me.isFluidEventSubscription()`
- Returns: boolean
- Purpose: checks whether fluid change subscription is enabled

### `setItemEventSubscription`

- Syntax: `me.setItemEventSubscription(enabled)`
- Parameters:
  - `enabled`: boolean
- Returns: nothing
- Purpose: subscribes or unsubscribes from `network_item_changed`

### `isItemEventSubscription`

- Syntax: `me.isItemEventSubscription()`
- Returns: boolean
- Purpose: checks whether item change subscription is enabled

Example:

```lua
local event = require("event")
local me = component.me_controller

me.setItemEventSubscription(true)
local _, _, changed = event.pull("network_item_changed")
print(changed.label, changed.size)
```

## `me_controller`

`me_controller` exposes the shared network control callbacks listed above.
Use it when you want a stationary machine to inspect or automate an AE network directly.

## `me_exportbus`

`me_exportbus` configures an AE export bus on a multipart host.
All bus directions are regular Forge sides `0` through `5`.

### `getExportConfiguration`

- Syntax: `bus.getExportConfiguration(side[, slot])`
- Parameters:
  - `side`: side the export bus is installed on
  - `slot`: optional one-based configuration slot, defaults to `1`
- Returns: configured item descriptor table or `nil`
- Purpose: reads one export filter slot

### `setExportConfiguration`

- Syntax: `bus.setExportConfiguration(side[, slot][, databaseAddress, entry])`
- Syntax: `bus.setExportConfiguration(side[, slot][, detail])`
- Parameters:
  - `side`: bus side
  - `slot`: optional one-based configuration slot, defaults to `1`
  - `databaseAddress`, `entry`: copy an item descriptor from a database
  - `detail`: direct item descriptor table
- Returns: `true`
- Purpose: sets or clears one export filter slot

Omitting the descriptor clears the slot.

### `getExportSlotSize`

- Syntax: `bus.getExportSlotSize(side)`
- Returns: number
- Purpose: returns how many config slots are currently usable

The result depends on installed AE capacity upgrades.

### `getExportOreFilter`

- Syntax: `bus.getExportOreFilter(side)`
- Returns: string
- Purpose: reads the ore dictionary filter string

### `setExportOreFilter`

- Syntax: `bus.setExportOreFilter(side, filter)`
- Parameters:
  - `filter`: ore dictionary name or filter string accepted by AE2
- Returns: `true`
- Purpose: changes the export bus ore filter

### `exportIntoSlot`

- Syntax: `bus.exportIntoSlot(side, slot)`
- Parameters:
  - `side`: bus side
  - `slot`: one-based target inventory slot on the adjacent inventory
- Returns: `true` on success, `false` when nothing was exported, or `nil, "no inventory"` when no adjacent inventory exists
- Purpose: performs one immediate export attempt into a specific inventory slot

The call pauses for about `0.25` seconds after a successful transfer burst.

Example:

```lua
local bus = component.me_exportbus
bus.setExportConfiguration(sides.north, 1, {name = "minecraft:glass"})
print(bus.exportIntoSlot(sides.north, 1))
```

## `me_importbus`

`me_importbus` configures an AE import bus on a multipart host.

### `getImportConfiguration`

- Syntax: `bus.getImportConfiguration(side[, slot])`
- Returns: configured item descriptor table or `nil`
- Purpose: reads one import filter slot

### `setImportConfiguration`

- Syntax: `bus.setImportConfiguration(side[, slot][, databaseAddress, entry])`
- Syntax: `bus.setImportConfiguration(side[, slot][, detail])`
- Returns: `true`
- Purpose: sets or clears one import filter slot

### `getImportSlotSize`

- Syntax: `bus.getImportSlotSize(side)`
- Returns: number
- Purpose: returns the number of usable import filter slots

### `getImportOreFilter`

- Syntax: `bus.getImportOreFilter(side)`
- Returns: string
- Purpose: reads the ore dictionary filter string

### `setImportOreFilter`

- Syntax: `bus.setImportOreFilter(side, filter)`
- Returns: `true`
- Purpose: changes the ore dictionary filter

Example:

```lua
local bus = component.me_importbus
bus.setImportConfiguration(sides.up, 1, {name = "minecraft:clay_ball"})
print(bus.getImportSlotSize(sides.up))
```

## `me_storagebus`

`me_storagebus` configures an AE storage bus on a multipart host.

### `getStorageConfiguration`

- Syntax: `bus.getStorageConfiguration(side[, slot])`
- Returns: configured item descriptor table or `nil`
- Purpose: reads one storage bus filter slot

### `setStorageConfiguration`

- Syntax: `bus.setStorageConfiguration(side[, slot][, databaseAddress, entry])`
- Syntax: `bus.setStorageConfiguration(side[, slot][, detail])`
- Returns: `true`
- Purpose: sets or clears one storage bus filter slot

### `getStorageOreFilter`

- Syntax: `bus.getStorageOreFilter(side)`
- Returns: string
- Purpose: reads the storage bus ore filter

### `setStorageOreFilter`

- Syntax: `bus.setStorageOreFilter(side, filter)`
- Returns: `true`
- Purpose: changes the storage bus ore filter

### `getStorageSlotSize`

- Syntax: `bus.getStorageSlotSize(side)`
- Returns: number
- Purpose: returns the number of usable configuration slots

Storage buses start with more slots than import or export buses, and capacity upgrades increase the limit further.

## `me_interface`

`me_interface` is exposed for both full AE interface blocks and multipart interfaces.
Block interfaces omit the leading `side` argument.
Multipart interfaces require it.

### `getInterfaceConfiguration`

- Syntax: `iface.getInterfaceConfiguration([slot])`
- Syntax: `iface.getInterfaceConfiguration(side[, slot])`
- Parameters:
  - `slot`: optional one-based configuration slot, defaults to `1`
- Returns: item descriptor table or `nil`
- Purpose: reads one interface config slot

### `setInterfaceConfiguration`

- Syntax: `iface.setInterfaceConfiguration([slot][, databaseAddress, entry[, size]])`
- Syntax: `iface.setInterfaceConfiguration([slot][, detail])`
- Syntax: `iface.setInterfaceConfiguration(side[, slot][, databaseAddress, entry[, size]])`
- Syntax: `iface.setInterfaceConfiguration(side[, slot][, detail])`
- Returns: `true`
- Purpose: sets or clears one interface config slot

Clearing is done by omitting the descriptor argument.

### `getInterfacePattern`

- Syntax: `iface.getInterfacePattern([slot])`
- Syntax: `iface.getInterfacePattern(side[, slot])`
- Parameters:
  - `slot`: one-based pattern slot, defaults to `1`
- Returns: encoded pattern item stack or `nil`
- Purpose: reads one stored pattern from the interface

### `setInterfacePatternInput`

- Syntax: `iface.setInterfacePatternInput(slot, index[, databaseAddress, entry, size])`
- Syntax: `iface.setInterfacePatternInput(slot, index[, detail, type])`
- Purpose: overwrites one pattern input entry

Parameters:

- `slot`: one-based pattern slot inside the interface
- `index`: one-based input index inside the encoded pattern
- `databaseAddress`, `entry`, `size`: source database entry
- `detail`: item or fluid descriptor table
- `type`: AE stack type id such as the registered item or fluid type

Returns `true` on success.
If no replacement descriptor is supplied, the selected entry is cleared.

Failure cases:

- invalid interface slot
- invalid pattern index
- no encoded pattern in the selected slot
- unknown AE stack type

### `setInterfacePatternOutput`

- Syntax: `iface.setInterfacePatternOutput(slot, index[, databaseAddress, entry, size])`
- Syntax: `iface.setInterfacePatternOutput(slot, index[, detail, type])`
- Returns: `true`
- Purpose: overwrites one pattern output entry

Behavior is the same as `setInterfacePatternInput`, but it edits the output list.

### `storeInterfacePatternInput`

- Syntax: `iface.storeInterfacePatternInput(slot, index, databaseAddress, entry)`
- Returns: `true`
- Purpose: copies one pattern input entry into a database slot

The copied stack is normalized to size `1`.

### `storeInterfacePatternOutput`

- Syntax: `iface.storeInterfacePatternOutput(slot, index, databaseAddress, entry)`
- Returns: `true`
- Purpose: copies one pattern output entry into a database slot

### `clearInterfacePatternInput`

- Syntax: `iface.clearInterfacePatternInput(slot, index)`
- Returns: `true`
- Purpose: removes one input entry from a block interface pattern

This callback exists only on block interfaces.

### `clearInterfacePatternOutput`

- Syntax: `iface.clearInterfacePatternOutput(slot, index)`
- Returns: `true`
- Purpose: removes one output entry from a block interface pattern

This callback exists only on block interfaces.

Example:

```lua
local iface = component.me_interface
local pattern = iface.getInterfacePattern(1)
if pattern then
  print(pattern.label)
end
```

## `me_interface_terminal`

`me_interface_terminal` discovers interfaces visible to the AE network and moves encoded patterns between them.

### `getInterfaces`

- Syntax: `term.getInterfaces()`
- Returns: iterator userdata
- Purpose: snapshots all visible interfaces

### `getInterfacesByName`

- Syntax: `term.getInterfacesByName(filter)`
- Parameters:
  - `filter`: exact display name
- Returns: iterator userdata
- Purpose: snapshots interfaces whose displayed name matches exactly

### `getInterfacesByLocation`

- Syntax: `term.getInterfacesByLocation(location[, side])`
- Parameters:
  - `location`: table with `x`, `y`, `z`, and optional `dimId`
  - `side`: optional numeric or string Forge side
- Returns: iterator userdata
- Purpose: snapshots interfaces at one exact location

### `send`

- Syntax: `term.send(source, target)`
- Parameters:
  - `source`: table with `location`, optional `side`, and required `slot`
  - `target`: table with `location`, optional `side`, and optional `slot`
- Returns: `true, slot` on success or `false, error`
- Purpose: moves one encoded pattern from one interface to another

If `target.slot` is omitted, the first empty target slot is used.

Common errors:

- `Source slot cannot be empty`
- `Source slot out of bounds`
- `Can't find source pattern`
- `Target slot out of bounds`
- `Target slot is not empty`
- `No empty slot in target machine`
- `Source machine not found`
- `Target machine not found`
- `Both machines not found`

### `sendBatch`

- Syntax: `term.sendBatch(tasks)`
- Parameters:
  - `tasks`: array of `{source = ..., target = ...}` tables
- Returns: array of result tuples in the same order as the input tasks
- Purpose: performs multiple transfers in one call

### Iterator userdata

The userdata returned by the three discovery callbacks exposes the following behaviors.

### `iterator()`

- Syntax: `iterator()`
- Returns: one interface record, then the next one on each call, and finally `nil`
- Purpose: sequential iteration

Each record contains:

- `name`
- `location`
- `side`
- `patterns`: map of zero-based pattern slot index to pattern stack when pattern inclusion is enabled

### `iterator(index)`

- Syntax: `iterator(index)`
- Parameters:
  - `index`: one-based record index
- Returns: the matching interface record or `nil`
- Purpose: random access into the snapshot

### `iterator("n")`

- Syntax: `iterator("n")`
- Returns: number
- Purpose: returns the snapshot length using the userdata call operator

### `reset`

- Syntax: `iterator.reset()`
- Returns: nothing
- Purpose: rewinds sequential iteration

### `count`

- Syntax: `iterator.count()`
- Returns: number
- Purpose: returns the number of interfaces in the snapshot

### `getAll`

- Syntax: `iterator.getAll([includePatterns])`
- Parameters:
  - `includePatterns`: optional boolean, defaults to `false`
- Returns: array of all interface records
- Purpose: materializes the entire snapshot into Lua

`getAll(true)` can be expensive on large networks because every pattern inventory is serialized at once.

Example:

```lua
local term = component.me_interface_terminal
local interfaces = term.getInterfaces()
print("visible:", interfaces.count())

local first = interfaces(1)
if first then
  print(first.name, first.location.x, first.location.y, first.location.z)
end
```

## `upgrade_me`

`upgrade_me` is the AE upgrade installed in a robot or drone.
Besides the shared network callbacks, it also provides direct item and fluid transfer with the currently selected inventory slot or tank.

The upgrade must be linked to an AE security terminal and be within wireless range of a valid AE wireless access point.

### `sendItems`

- Syntax: `up.sendItems([amount])`
- Parameters:
  - `amount`: optional maximum item count, default `64`
- Returns: number of items actually inserted
- Purpose: sends items from the selected inventory slot into the AE network

If the selected slot is empty or the upgrade is not linked, the result is `0`.

### `requestItems`

- Syntax: `up.requestItems(databaseAddress, entry[, amount])`
- Syntax: `up.requestItems(detail)`
- Parameters:
  - `databaseAddress`, `entry`: database item descriptor
  - `amount`: optional request amount
  - `detail`: item descriptor table with at least `name` or `id`, and optional `damage` and `size`
- Returns: number of items actually extracted
- Purpose: pulls matching items into the selected inventory slot

Important behavior:

- if the selected slot already contains a different item type, the result is `0`
- when no size is supplied, the request grows to the largest legal stack for that slot
- extraction is capped by the selected slot's remaining free space

### `sendFluids`

- Syntax: `up.sendFluids([amount])`
- Parameters:
  - `amount`: optional maximum millibuckets, defaults to the tank content or capacity limit
- Returns: number of millibuckets actually inserted
- Purpose: sends fluid from the selected tank into the AE network

### `requestFluids`

- Syntax: `up.requestFluids(databaseAddress, entry[, amount])`
- Syntax: `up.requestFluids(detail)`
- Parameters:
  - `databaseAddress`, `entry`: database entry describing a filled fluid container
  - `amount`: optional millibuckets
  - `detail`: fluid descriptor table with at least `name` or `id`, and optional `size`
- Returns: number of millibuckets actually extracted
- Purpose: fills the selected tank from the AE network

When no size is given in a direct detail table, the default request amount is one bucket, `1000 mB`.

### `isLinked`

- Syntax: `up.isLinked()`
- Returns: boolean
- Purpose: reports whether the upgrade can currently reach its linked AE network

Example:

```lua
local me = component.upgrade_me
if me.isLinked() then
  print("sent", me.sendItems(16))
end
```

## Returned Userdata

### `Craftable`

Returned by `getCraftable` and `getCraftables`.

#### `getStack`

- Syntax: `craftable.getStack()`
- Returns: converted item table
- Purpose: returns the recipe output represented by this craftable handle

#### `request`

- Syntax: `craftable.request([amount[, prioritizePower[, cpuName]]])`
- Parameters:
  - `amount`: optional number of output items to craft, defaults to `1`
  - `prioritizePower`: optional boolean, defaults to `true`
  - `cpuName`: optional exact crafting CPU name
- Returns: `CraftingStatus` userdata, or `nil, "no controller"` when the source controller is gone
- Purpose: starts an asynchronous AE crafting request

The request is computed asynchronously first, then submitted to the network.
Typical failure reasons later reported through the status object include missing resources or internal AE submission errors.

### `CPU`

Returned in each `getCpus` descriptor under the `cpu` field.

#### `cancel`

- Syntax: `cpu.cancel()`
- Returns: boolean
- Purpose: cancels the current job running on that CPU

Returns `false` when the CPU is not busy.

#### `isActive`

- Syntax: `cpu.isActive()`
- Returns: boolean
- Purpose: checks whether the CPU cluster is currently active

#### `isBusy`

- Syntax: `cpu.isBusy()`
- Returns: boolean
- Purpose: checks whether the CPU currently has a job assigned

#### `activeItems`

- Syntax: `cpu.activeItems()`
- Returns: array of converted AE stack tables
- Purpose: lists items currently being crafted

#### `pendingItems`

- Syntax: `cpu.pendingItems()`
- Returns: array of converted AE stack tables
- Purpose: lists items still waiting to be processed

#### `storedItems`

- Syntax: `cpu.storedItems()`
- Returns: array of converted AE stack tables
- Purpose: lists items already stored inside the crafting job buffer

#### `finalOutput`

- Syntax: `cpu.finalOutput()`
- Returns: output stack table, or `nil, "No crafting monitor"`, or `nil, "Nothing is crafted"`
- Purpose: reads the final crafting monitor output if the CPU cluster has one

### `CraftingStatus`

Returned by `Craftable.request`.

#### `isComputing`

- Syntax: `status.isComputing()`
- Returns: boolean
- Purpose: checks whether AE is still computing the crafting plan

#### `hasFailed`

- Syntax: `status.hasFailed()`
- Returns: `boolean, reason`
- Purpose: reports whether the request failed and why

Typical reasons look like `request failed (missing resources)` or another submission error string.

#### `isCanceled`

- Syntax: `status.isCanceled()`
- Returns: `boolean[, reason]`
- Purpose: checks whether the crafting link was canceled

While the request is still being computed, the method returns `false, "computing"`.

#### `isDone`

- Syntax: `status.isDone()`
- Returns: `boolean[, reason]`
- Purpose: checks whether the crafting request completed

While the request is still being computed, the method returns `false, "computing"`.

## Full Example

```lua
local component = require("component")
local event = require("event")

local me = component.me_controller

print("Power:", me.getStoredPower(), "/", me.getMaxStoredPower())

for _, stack in ipairs(me.getItemsInNetwork({label = "Iron Ingot"})) do
  print(stack.label, stack.size)
end

local craftables = me.getCraftables({label = "Printed Silicon"})
if #craftables > 0 then
  local status = craftables[1].request(1, true)
  while status.isComputing() do
    os.sleep(0.1)
  end
  print("done:", status.isDone())
end

me.setItemEventSubscription(true)
local _, _, changed = event.pull(1, "network_item_changed")
if changed then
  print("changed:", changed.label, changed.size)
end
```

## Related

- `ae2fc`
- `thaumicenergistics`
- `opencomputers-data-card`
