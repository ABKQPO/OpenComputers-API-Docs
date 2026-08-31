# storagedrawers

## Summary

This page documents the Storage Drawers integration exposed by OpenComputers. Drawer controller blocks and drawer groups become queryable Lua components for reading occupancy and stored item identity.

> Source audit note (2026-08-31): the current StorageDrawers commit adds compatibility with the generic OpenComputers transposer for controller/slave blocks. The dedicated `drawer` component described below is retained for compatibility with the existing documentation set and should be verified against the installed modpack at runtime.

## Availability

- Dependency: `storagedrawers`
- Label: `integration-required`

## Source Audit

- The current OpenComputers tree has no `storagedrawers` or `drawer` driver under `src/main`.
- The current StorageDrawers tree has no dedicated OpenComputers API imports, callbacks, or component registration.
- StorageDrawers commit `59b527f` makes controller/slave blocks usable through the generic OpenComputers transposer. It does not register a `drawer` component or a Storage Drawers-specific Lua API.

## Runtime Verification Note

The `drawer` API below is retained from the existing documentation set. Because the current source scan does not locate its dedicated driver, verify that the installed modpack exposes it with `component.list()` and `component.methods(address)`.

## Component

### drawer

The `drawer` component is exposed on blocks implementing the Storage Drawers `IDrawerGroup` interface.

#### `getDrawerCount()`

- Syntax: `getDrawerCount(): number`
- Purpose: Return how many drawer slots the block exposes.
- Returns:
  - Drawer slot count.

#### `getItemCount(drawerSlot)`

- Syntax: `getItemCount(drawerSlot: number): number`
- Purpose: Return the number of stored items in one drawer slot.
- Parameters:
  - `drawerSlot: number`
    Drawer slot index starting at `1` in Lua.
- Returns:
  - Current stored item count for that slot.
- Usage notes:
  - Lua uses one-based indices here, but the driver converts them to zero-based Java indices internally.
  - Requesting a missing or disabled drawer raises `no drawer found at slot N`.

#### `getMaxCapacity(drawerSlot)`

- Syntax: `getMaxCapacity(drawerSlot: number): number`
- Purpose: Return the maximum item capacity of one drawer slot.
- Parameters:
  - `drawerSlot: number`
    Drawer slot index starting at `1` in Lua.
- Returns:
  - Maximum storage capacity for that slot.
- Usage notes:
  - Invalid slots raise `no drawer found at slot N`.

#### `getItemName(drawerSlot)`

- Syntax: `getItemName(drawerSlot: number): string | nil, string?`
- Purpose: Return the stored item's unlocalized name.
- Parameters:
  - `drawerSlot: number`
    Drawer slot index starting at `1` in Lua.
- Returns:
  - An unlocalized item name such as `item.ingotIron` when the drawer is not empty.
  - `nil` plus an explanatory message when the drawer is empty.
- Usage notes:
  - This is not the localized display label shown to players.
  - Empty drawers do not throw here; they return `nil` and a message.

#### `getItemDamage(drawerSlot)`

- Syntax: `getItemDamage(drawerSlot: number): number | nil, string?`
- Purpose: Return the stored item's damage value or metadata.
- Parameters:
  - `drawerSlot: number`
    Drawer slot index starting at `1` in Lua.
- Returns:
  - Damage / metadata for a non-empty drawer.
  - `nil` plus an explanatory message when the drawer is empty.
- Usage notes:
  - This is useful when multiple item variants share the same base item id.
  - A non-empty drawer can still briefly report unusual counts during transient updates, so combine this with `getItemCount` if you need a robust emptiness check.

#### Example

```lua
local component = require("component")
local drawer = component.drawer

for slot = 1, drawer.getDrawerCount() do
  local name, message = drawer.getItemName(slot)
  local count = drawer.getItemCount(slot)
  local capacity = drawer.getMaxCapacity(slot)

  if name then
    print(slot, name, count .. "/" .. capacity)
  else
    print(slot, "empty", message)
  end
end
```

Use a generic inventory transposer only when the installed modpack exposes the relevant inventory capability. Verify available methods at runtime with `component.list()` and `component.methods(address)`.

## Related

- `betterstorage`
- `appeng`
