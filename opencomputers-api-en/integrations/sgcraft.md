# sgcraft

## Summary

This page documents the OpenComputers Stargate interface block provided by SGCraft. The interface exposes Stargate control callbacks through a component named `stargate` and can also relay OpenComputers network packets through an installed LAN card upgrade.

## Availability

- Dependency: `sgcraft`
- Label: `integration-required`

## Component Names

- `stargate`

## What This Integration Is Good For

- Reading local Stargate state, chevron count, and connection direction.
- Checking available energy and estimating the cost of a dial attempt.
- Dialing and disconnecting gates from Lua scripts.
- Controlling an iris upgrade.
- Sending structured messages to the remote gate and receiving Stargate events.
- Rebroadcasting OpenComputers wireless packets through connected gates when a LAN card is installed in the interface block.

## Setup Notes

- Place the OpenComputers Stargate Interface block adjacent to a merged Stargate.
- The component is exposed as `component.stargate`.
- The interface has one internal upgrade slot.
- Installing an OpenComputers `lanCard` item in that slot enables Stargate packet rebroadcast for OpenComputers network packets.

## Component

### stargate

#### `stargateState()`

- Syntax: `stargateState(): string, number, string`
- Purpose: Return the current Stargate state, number of engaged chevrons, and connection direction.
- Returns:
  - `state: string`
    Human-readable Stargate state description.
  - `chevrons: number`
    Number of engaged chevrons.
  - `direction: string`
    `Outgoing`, `Incoming`, or an empty string when not connected.
- Usage notes:
  - If no Stargate is connected to the interface, this returns `Offline, 0, ""`.

#### `energyAvailable()`

- Syntax: `energyAvailable(): number`
- Purpose: Return the energy currently available to the attached Stargate.
- Returns:
  - Available dial energy.

#### `energyToDial(address)`

- Syntax: `energyToDial(address: string): number[, string]`
- Purpose: Estimate the energy required to dial a normalized Stargate address.
- Parameters:
  - `address: string`
    Destination Stargate address.
- Returns:
  - Required energy on success.
  - `nil, message` if the local Stargate is missing, the address is invalid, or the destination cannot be found.
- Usage notes:
  - The address is normalized before lookup.
  - The cost depends on the coordinate distance between the local and remote gates.

#### `localAddress()`

- Syntax: `localAddress(): string`
- Purpose: Return the home address of the attached Stargate.
- Returns:
  - Local Stargate address, or an empty string if the interface is not attached to a gate.

#### `remoteAddress()`

- Syntax: `remoteAddress(): string[, string]`
- Purpose: Return the address of the currently connected remote gate.
- Returns:
  - Remote address when connected.
  - An empty string when the local gate is idle.
  - `nil, message` if no Stargate is connected to the interface.

#### `dial(address)`

- Syntax: `dial(address: string): boolean[, string]`
- Purpose: Start a dial attempt to the specified destination address.
- Parameters:
  - `address: string`
    Destination Stargate address.
- Returns:
  - `true` on success.
  - `nil, message` if dialing fails.
- Usage notes:
  - The address is normalized before dialing.
  - Any Stargate-side error is returned as the second result.

#### `disconnect()`

- Syntax: `disconnect(): boolean[, string]`
- Purpose: Close the current Stargate connection.
- Returns:
  - `true` on success.
  - `nil, message` if there is no connected gate or the disconnect attempt fails.

#### `irisState()`

- Syntax: `irisState(): string`
- Purpose: Return the current iris state description.
- Returns:
  - Iris state string.
- Usage notes:
  - If no iris upgrade is installed, this returns `Offline`.

#### `openIris()`

- Syntax: `openIris(): boolean[, string]`
- Purpose: Start opening the iris.
- Returns:
  - `true` on success.
  - `nil, message` if there is no connected Stargate or no iris upgrade is installed.

#### `closeIris()`

- Syntax: `closeIris(): boolean[, string]`
- Purpose: Start closing the iris.
- Returns:
  - `true` on success.
  - `nil, message` if there is no connected Stargate or no iris upgrade is installed.

#### `sendMessage(...)`

- Syntax: `sendMessage(...): boolean[, string]`
- Purpose: Send one Stargate application-layer message to the currently connected remote gate.
- Parameters:
  - `...`
    Zero or more Lua values. The interface forwards them as an object array.
- Returns:
  - `true` on success.
  - `nil, message` if the gate is not connected or the send fails.
- Usage notes:
  - The receiver gets the payload through the `sgMessageReceived` event.
  - Arguments are forwarded in order without an extra wrapper table.

## Events

The interface posts these events to reachable OpenComputers machines through `computer.signal`.

#### Event: `sgStargateStateChange`

- Syntax: `sgStargateStateChange(newState, oldState)`
- Purpose: Fired when the Stargate state description changes.
- Parameters:
  - `newState: string`
    New human-readable state.
  - `oldState: string`
    Previous human-readable state.

#### Event: `sgDialOut`

- Syntax: `sgDialOut(address)`
- Purpose: Fired when the local Stargate begins an outgoing connection to `address`.

#### Event: `sgDialIn`

- Syntax: `sgDialIn(address)`
- Purpose: Fired when the local Stargate receives an incoming connection from `address`.

#### Event: `sgChevronEngaged`

- Syntax: `sgChevronEngaged(chevronCount, symbol)`
- Purpose: Fired each time another chevron locks during dialing.
- Parameters:
  - `chevronCount: number`
    Number of chevrons engaged after the lock.
  - `symbol: string`
    The symbol that was just locked.

#### Event: `sgIrisStateChange`

- Syntax: `sgIrisStateChange(newState, oldState)`
- Purpose: Fired when the iris state description changes.

#### Event: `sgMessageReceived`

- Syntax: `sgMessageReceived(...)`
- Purpose: Fired when the remote Stargate sends a message through `sendMessage(...)`.
- Parameters:
  - `...`
    The exact values forwarded by the sender.

## OpenComputers Packet Bridging

- Every Stargate base tile joins the OpenComputers wireless network internally.
- If an interface block contains a LAN card, incoming `network.message` packets can be forwarded through an active Stargate to the connected remote gate.
- Packets are only forwarded while their TTL is still above zero.
- The default wireless rebroadcast strength used by the Stargate-side bridge is `50`.

## Example

```lua
local component = require("component")
local event = require("event")
local sg = component.stargate

local state, chevrons, direction = sg.stargateState()
print("State:", state, chevrons, direction)
print("Local address:", sg.localAddress())

local ok, err = sg.dial("ABCD-EFG-HI")
if not ok then
  error(err)
end

while true do
  local _, name, a, b = event.pull()
  if name == "sgChevronEngaged" then
    print("Chevron", a, "locked symbol", b)
  elseif name == "sgStargateStateChange" then
    print("State changed:", a, "(was", b .. ")")
  end
end
```

## Related

- `stargatetech2`
- `network`
