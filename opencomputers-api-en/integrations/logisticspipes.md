# logisticspipes

## Summary

This page documents the OpenComputers integration exposed by LogisticsPipes. Unlike simpler integrations, LogisticsPipes does not publish one small handwritten callback set per block. Instead, it registers two OpenComputers components and then exposes most of its callable API through a wrapper system generated from LogisticsPipes' own `@CCType` and `@CCCommand` metadata.

## Availability

- Dependency: `logisticspipes`
- Label: `integration-required`
- Source repositories audited for this refresh:
  - `LogisticsPipes`
  - `OpenComputers`

## Component Names

- `logisticspipe`
- `logisticssolidblock`

## Integration Model

The actual OpenComputers bridge is created in `logisticspipes.proxy.opencomputers.OpenComputersProxy`.

Key behavior from source:

- routed pipes create an OC node with component name `logisticspipe`
- LP solid blocks create an OC node with component name `logisticssolidblock`
- ordinary OpenComputers signals are sent with `computer.signal`
- LP network messages are forwarded as the event `LP_MESSAGE`
- most callable methods come from generated wrapper objects backed by LP's internal `@CCType` / `@CCCommand` metadata

In practice this means the component itself is an entry point into a larger wrapped object model.

## Signal And Addressing Model

### Component addresses

Each LP OC node has a normal OpenComputers node address. Source code uses that address when forwarding messages over the LP network.

### `LP_MESSAGE`

When one LP pipe sends a network message to another computer endpoint, the OpenComputers side receives it as:

```lua
event.pull("LP_MESSAGE")
```

The payload is:

- event name: `LP_MESSAGE`
- source id:
  - a ComputerCraft numeric id when the sender was a CC computer
  - an OpenComputers component address string when the sender was an OC node
- message:
  - the arbitrary Lua-compatible payload supplied by the sender

### Broadcasts

The routed pipe source also defines a broadcast path for LP-wide computer messages. The current OC bridge clearly forwards `LP_MESSAGE`; treat LP broadcast behavior as related but verify exact event naming against runtime before relying on it for production automation.

## Discoverability Helpers

Wrapped LP values expose two important discovery helpers through the generic wrapper base class.

### `help([page])`

- Syntax: `value.help([page: number]): string`
- Purpose: Return a human-readable command list for the wrapped LP value.
- Usage notes:
  - methods marked as direct calls are flagged with `D`
  - methods marked as queued calls are flagged with `Q`
  - large command sets are paginated

### `helpCommand(index)`

- Syntax: `value.helpCommand(index: number): string`
- Purpose: Return the name, parameter types, return type, and description for one wrapped command.
- Usage notes:
  - command numbering comes from the generated wrapper metadata, not from Lua method order

These two helpers are the most reliable way to inspect large wrapped LP objects at runtime.

## `logisticspipe`

`logisticspipe` is exposed by routed LogisticsPipes pipe tiles.

The concrete callable surface depends on the wrapped pipe type. Source metadata confirms at least these wrapped families:

- `LogisticsPipes:Normal`
- `LogisticsPipes:Request`
- `LogisticsPipes:Crafting`
- `LogisticsChassiePipe`

### Shared routed-pipe methods

The base routed pipe type exposes methods such as:

- `getRouterId()`
  Return the runtime-stable numeric router id.
- `getRouterUUID()`
  Return the persistent router UUID string.
- `getRouterUUID(id)`
  Resolve another reachable router's UUID from its numeric router id.
- `setTurtleConnect(flag)`
  Change the turtle-connect flag on the current pipe.
- `getTurtleConnect()`
  Read the turtle-connect flag.
- `canAccess()`
  Check whether the current computer is permitted to interact with the pipe.
- `sendMessage(computerId, message)`
  Send an LP network computer message that becomes an `LP_MESSAGE` event on the destination.
- `sendBroadcast(message)`
  Send a network-wide computer broadcast through LP routing.
- `getPipeForUUID(uuid)`
  Return access to another reachable pipe when the remote-control upgrade is installed.
- `getLP()`
  Return the global LP helper object.
- `hasLogisticsModule()`
  Return whether this pipe has an internal module.

### `LogisticsPipes:Request`

Request pipes add network-query and request methods such as:

- `makeRequest(stack)`
- `makeRequest(stack, forceCrafting)`
- `makeRequest(item, amount)`
- `makeRequest(item, amount, forceCrafting)`
- `getAvailableItems()`
- `getCraftableItems()`
- `getItemAmount(item)`

The request methods are queued in source, so they are meant for logistics operations that may take time or need server-side scheduling.

### `LogisticsPipes:Crafting`

Crafting pipes add:

- `reimport()`
  Re-import the crafting recipe from the attached machine or crafting table.

The wider crafting state is still accessible through the wrapped routed-pipe and module object graph.

### `LogisticsChassiePipe`

Chassis pipes add:

- `getModuleInSlot(slot)`
  Return the wrapped module object installed in the one-based chassis slot.
- `getChassieSize()`
  Return the chassis module slot count.

Returned module objects can then be inspected with `help()` and `helpCommand()`.

## `logisticssolidblock`

`logisticssolidblock` is exposed by LogisticsPipes solid utility blocks that participate in the OC bridge.

The audited source confirms at least:

- `getRotation()`
  Return the LP rotation value of the block.

Like routed pipes, the block uses the wrapper system to expose additional behavior when the concrete block type carries LP computer metadata.

## Wrapped Module Families

Source metadata confirms these module-family wrappers are available through LP objects when the relevant module exists:

- `LogisticsModule`
- `Provider Module`
- `ItemSink Module`
- `Advanced Extractor Module`
- `Terminus Module`
- `EnchantmentSink Module MK2`

Typical module patterns include:

- returning filter inventories
- exposing boolean flags such as default-route behavior
- exposing module-specific request or provider controls

Because the exact module instance depends on installed hardware, use `help()` on the returned module object for the authoritative runtime surface.

## Wrapped Value Objects

Source metadata confirms these reusable wrapped object families:

- `LP Global Access`
- `ItemIdentifier`
- `ItemIdentifierStack`
- `ItemIdentifierBuilder`
- `FluidIdentifier`
- `FilterInventory`
- `CCItemSinkRequest`
- `Resource`
- `ItemResource`
- `DictResource`
- `FluidResource`
- `Pair`
- `Triplet`
- `Quartet`

Important examples:

- `LP Global Access`
  - `identify(object)`
  - `getItemIdentifierBuilder()`
- `ItemIdentifier`
  - item id, name, mod, tag, damage, and conversion helpers
- `ItemIdentifierStack`
  - stack identity plus size helpers
- `FilterInventory`
  - size, read-slot, and write-slot helpers
- `FluidIdentifier`
  - fluid id, name, tag, localized name, and conversion helpers

These wrapped values are how most non-trivial LP automation data moves through the OC bridge.

## Example

```lua
local component = require("component")
local event = require("event")

local pipe = component.logisticspipe

print("Router id:", pipe.getRouterId())
print("Router uuid:", pipe.getRouterUUID())
print("Can access:", pipe.canAccess())

local lp = pipe.getLP()
print(lp.identify(lp))

local helpText = lp.help()
print(helpText)

pipe.sendBroadcast("hello from oc")

local _, sourceId, payload = event.pull(0.5, "LP_MESSAGE")
if sourceId then
  print("LP message from", sourceId, payload)
end
```

## Usage Notes

- The wrapper layer is case-insensitive when matching method names internally, but documentation should still use the canonical source names.
- Some LP methods are direct-only in source. If a method refuses to run from the current context, inspect the wrapper help output and the pipe permissions.
- Remote-pipe access depends on LP upgrades and network reachability. A method existing on one pipe does not imply unrestricted access to every other LP node.
- LP object persistence is limited. Some wrapped values can be saved and restored, but others are effectively live references around current world state.

## Related

- `storagedrawers`
- `betterstorage`
- `appeng`
