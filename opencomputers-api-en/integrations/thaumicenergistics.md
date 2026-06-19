# thaumicenergistics

## Summary

This page documents the Thaumic Energistics integration exposed to Lua by OpenComputers. It adds essentia-aware buses and essentia network inspection on top of the AE-style integration model.

## Availability

- Dependency: `thaumicenergistics`
- Label: `integration-required`

## Component Names

- `essentia_exportbus`
- `essentia_importbus`
- `essentia_storagebus`
- `me_controller`
- `me_interface`

## What This Integration Is Good For

- Configuring essentia export, import, and storage buses.
- Reading stored essentia from a connected Thaumic Energistics network.
- Subscribing to essentia change events.
- Reusing AE-style controllers and interfaces while working with essentia instead of items or fluids.

## Essentia Detail Tables

Returned essentia stack tables normally contain:

- `name`
  Thaumcraft aspect id.
- `amount`
  Stored amount.
- `label`
  Localized aspect name.

When a callback accepts a detail table, the key field is:

- `name: string`
  Required aspect id.
- `amount: number`
  Optional amount. Defaults to `0` when omitted.

If the named aspect does not exist, the callback raises an argument error.

## Components

### essentia_exportbus

`essentia_exportbus` is exposed on Thaumic Energistics essentia export bus parts.

#### `getExportConfiguration(side[, slot])`

- Syntax: `getExportConfiguration(side: number[, slot: number]): table`
- Purpose: Return the configured aspect in one export slot.
- Parameters:
  - `side: number`
    Multipart side containing the export bus.
  - `slot: number`
    Optional configuration slot index. Defaults to `0`.
- Returns:
  - Essentia descriptor table for the slot.

#### `setExportConfiguration(side[, slot][, detail])`

- Syntax: `setExportConfiguration(side: number[, slot: number][, detail: table]): boolean`
- Purpose: Set or clear one export configuration slot.
- Parameters:
  - `side: number`
    Multipart side containing the export bus.
  - `slot: number`
    Optional slot index. Defaults to `0`.
  - `detail: table`
    Optional essentia descriptor table.
- Returns:
  - `true` on success.
- Usage notes:
  - Omitting `detail` clears the slot.

#### `getExportSlotSize(side)`

- Syntax: `getExportSlotSize(side: number): number`
- Purpose: Return how many export config slots are currently valid.
- Parameters:
  - `side: number`
    Multipart side containing the export bus.
- Returns:
  - Slot count after AE capacity upgrade rules are applied.

#### `getExportOreFilter(side)`

- Syntax: `getExportOreFilter(side: number): string`
- Purpose: Return the current filter string.
- Parameters:
  - `side: number`
    Multipart side containing the export bus.
- Returns:
  - Filter string.

#### `setExportOreFilter(side, filter)`

- Syntax: `setExportOreFilter(side: number, filter: string): boolean`
- Purpose: Change the filter string.
- Parameters:
  - `side: number`
    Multipart side containing the export bus.
  - `filter: string`
    New filter string.
- Returns:
  - `true` on success.

#### `getVoidAllowed(side)`

- Syntax: `getVoidAllowed(side: number): boolean`
- Purpose: Return whether exporting into a void jar is allowed to discard excess essentia.
- Parameters:
  - `side: number`
    Multipart side containing the export bus.
- Returns:
  - `true` if voiding is allowed.
  - `false` otherwise.

#### `setVoidAllowed(side, allowed)`

- Syntax: `setVoidAllowed(side: number, allowed: boolean): boolean`
- Purpose: Change void-jar behavior.
- Parameters:
  - `side: number`
    Multipart side containing the export bus.
  - `allowed: boolean`
    Desired voiding state.
- Returns:
  - `true` if the call actually toggled the mode.
  - `false` if the part was already in the requested state.

### essentia_importbus

`essentia_importbus` is exposed on Thaumic Energistics essentia import bus parts.

#### `getImportConfiguration(side[, slot])`

- Syntax: `getImportConfiguration(side: number[, slot: number]): table`
- Purpose: Return the configured aspect in one import slot.
- Parameters:
  - `side: number`
    Multipart side containing the import bus.
  - `slot: number`
    Optional slot index. Defaults to `0`.
- Returns:
  - Essentia descriptor table.

#### `setImportConfiguration(side[, slot][, detail])`

- Syntax: `setImportConfiguration(side: number[, slot: number][, detail: table]): boolean`
- Purpose: Set or clear one import configuration slot.
- Parameters:
  - `side: number`
    Multipart side containing the import bus.
  - `slot: number`
    Optional slot index. Defaults to `0`.
  - `detail: table`
    Optional essentia descriptor table.
- Returns:
  - `true` on success.
- Usage notes:
  - Omitting `detail` clears the slot.

#### `getImportSlotSize(side)`

- Syntax: `getImportSlotSize(side: number): number`
- Purpose: Return how many import config slots are currently valid.
- Parameters:
  - `side: number`
    Multipart side containing the import bus.
- Returns:
  - Slot count.

#### `getImportOreFilter(side)`

- Syntax: `getImportOreFilter(side: number): string`
- Purpose: Return the current import filter string.
- Parameters:
  - `side: number`
    Multipart side containing the import bus.
- Returns:
  - Filter string.

#### `setImportOreFilter(side, filter)`

- Syntax: `setImportOreFilter(side: number, filter: string): boolean`
- Purpose: Change the import filter string.
- Parameters:
  - `side: number`
    Multipart side containing the import bus.
  - `filter: string`
    New filter string.
- Returns:
  - `true` on success.

### essentia_storagebus

`essentia_storagebus` is exposed on Thaumic Energistics essentia storage bus parts.

#### `getStorageConfiguration(side[, slot])`

- Syntax: `getStorageConfiguration(side: number[, slot: number]): table`
- Purpose: Return the configured aspect in one storage slot.
- Parameters:
  - `side: number`
    Multipart side containing the storage bus.
  - `slot: number`
    Optional slot index. Defaults to `0`.
- Returns:
  - Essentia descriptor table.

#### `setStorageConfiguration(side[, slot][, detail])`

- Syntax: `setStorageConfiguration(side: number[, slot: number][, detail: table]): boolean`
- Purpose: Set or clear one storage configuration slot.
- Parameters:
  - `side: number`
    Multipart side containing the storage bus.
  - `slot: number`
    Optional slot index. Defaults to `0`.
  - `detail: table`
    Optional essentia descriptor table.
- Returns:
  - `true` on success.
- Usage notes:
  - Omitting `detail` clears the slot.

#### `getStorageOreFilter(side)`

- Syntax: `getStorageOreFilter(side: number): string`
- Purpose: Return the current storage bus filter string.
- Parameters:
  - `side: number`
    Multipart side containing the storage bus.
- Returns:
  - Filter string.

#### `setStorageOreFilter(side, filter)`

- Syntax: `setStorageOreFilter(side: number, filter: string): boolean`
- Purpose: Change the storage bus filter string.
- Parameters:
  - `side: number`
    Multipart side containing the storage bus.
  - `filter: string`
    New filter string.
- Returns:
  - `true` on success.

#### `getStorageSlotSize(side)`

- Syntax: `getStorageSlotSize(side: number): number`
- Purpose: Return how many storage config slots are currently valid.
- Parameters:
  - `side: number`
    Multipart side containing the storage bus.
- Returns:
  - Slot count after AE storage bus upgrade rules are applied.

### me_controller

When `thaumicenergistics` is installed, `me_controller` gains shared essentia network control callbacks.

#### `getEssentiaInNetwork([filter])`

- Syntax: `getEssentiaInNetwork([filter: string])`
- Syntax: `getEssentiaInNetwork(detail: table)`
- Purpose: Return stored essentia visible in the network, or resolve one exact essentia descriptor.
- Parameters:
  - `filter: string`
    Optional aspect id to match against the aspect's unlocalized name.
  - `detail: table`
    Exact essentia descriptor table parsed through the registered AE stack parser.
- Returns:
  - String-filter form: array of essentia descriptor tables.
  - Detail-table form: one matching essentia descriptor table or `nil`.
- Usage notes:
  - If the monitor is unavailable, the callback returns `nil`.

#### `setEssentiaEventSubscription(enabled)`

- Syntax: `setEssentiaEventSubscription(enabled: boolean)`
- Purpose: Enable or disable the `network_essentia_changed` event subscription.
- Parameters:
  - `enabled: boolean`
    Desired subscription state.
- Returns:
  - No value.

#### `isEssentiaEventSubscription()`

- Syntax: `isEssentiaEventSubscription(): boolean`
- Purpose: Return whether essentia change events are currently enabled.
- Returns:
  - `true` if subscribed.
  - `false` otherwise.

### me_interface

Block `me_interface` components gain the same shared essentia network control callbacks in Thaumic Energistics networks.

#### `getEssentiaInNetwork([filter])`

- Syntax: `getEssentiaInNetwork([filter: string])`
- Syntax: `getEssentiaInNetwork(detail: table)`
- Purpose: Return stored essentia visible through the connected network, or resolve one exact essentia descriptor.
- Parameters:
  - `filter: string`
    Optional aspect id filter.
  - `detail: table`
    Exact essentia descriptor table parsed through the registered AE stack parser.
- Returns:
  - String-filter form: array of essentia descriptor tables.
  - Detail-table form: one matching essentia descriptor table or `nil`.

#### `setEssentiaEventSubscription(enabled)`

- Syntax: `setEssentiaEventSubscription(enabled: boolean)`
- Purpose: Enable or disable the `network_essentia_changed` event subscription.
- Parameters:
  - `enabled: boolean`
    Desired subscription state.
- Returns:
  - No value.

#### `isEssentiaEventSubscription()`

- Syntax: `isEssentiaEventSubscription(): boolean`
- Purpose: Return whether essentia change events are currently enabled.
- Returns:
  - `true` if subscribed.
  - `false` otherwise.

#### Event: `network_essentia_changed`

- Syntax: `network_essentia_changed(...)`
- Purpose: Notify the machine when stored essentia changes in the connected network.
- Usage notes:
  - The exact payload is managed by the shared AE subscription system and is intended for change-driven automation rather than fixed-position tuple parsing.

## Example

```lua
local component = require("component")
local controller = component.me_controller

for _, aspect in ipairs(controller.getEssentiaInNetwork()) do
  print(aspect.label, aspect.amount)
end

controller.setEssentiaEventSubscription(true)
```

## Related

- `appeng`
- `ae2fc`
- `thaumcraft`
