# nbt

## Summary

The `nbt` OpenOS helper library wraps the data-card NBT encode/decode callbacks in a more Lua-friendly proxy layer.
It turns typed NBT tables into proxy objects with normal field access and provides helpers for patching a component so `encodeNBT` and `decodeNBT` automatically work with those proxies.

## Availability

- Repository: `opencomputers`
- System: `openos`
- Type: `library`

## Source

- `OpenComputers/src\main\resources\assets\opencomputers\loot\openos\lib\nbt.lua`

## Usage

```lua
local component = require("component")
local nbt = require("nbt")

local data = nbt.patch(component.data)
```

## What It Does

- `patch(component)` wraps a component that already exposes `encodeNBT` and `decodeNBT`.
- `wrap(value)` turns raw typed NBT tables into Lua-friendly proxy objects.
- `unwrap(value)` returns the underlying raw typed table again.
- Wrapped compound and list values can be read with normal Lua indexing and iterated with `pairs`.

## API

### `nbt.patch(component)`

- Syntax: `nbt.patch(component)`
- Returns: patched component proxy or the original component
- Purpose: Replace `encodeNBT` and `decodeNBT` with proxy-aware versions.
- Notes:
  - The target component must expose both `encodeNBT` and `decodeNBT`.
  - If either callback is missing, the helper writes a warning to `stderr` and returns the original component unchanged.

### `nbt.wrap(value)`

- Syntax: `nbt.wrap(value)`
- Returns: wrapped proxy or the original value
- Purpose: Convert typed NBT tables into Lua-friendly proxy objects.
- Notes:
  - Scalar typed tags unwrap to their plain Lua scalar value immediately.
  - Compound and list tags become proxy tables.

### `nbt.unwrap(value)`

- Syntax: `nbt.unwrap(value)`
- Returns: raw typed table or the original value
- Purpose: Strip a proxy back to the raw typed NBT structure expected by low-level encoders.

## Proxy Behavior

Wrapped compound and list proxies support:

- `proxy[key]`
  Read nested tags with automatic wrapping.
- `proxy[key] = value`
  Replace an existing tag value or assign another typed tag.
- `pairs(proxy)`
  Iterate keys and wrapped children.
- `#proxy`
  Return the list length when the wrapped tag contains an array-like value.

The proxy also exposes typed setter helpers on compound or list values:

- `set_string`
- `set_boolean`
- `set_byte`
- `set_short`
- `set_int`
- `set_long`
- `set_float`
- `set_double`
- `set_byte_array`
- `set_int_array`
- `set_list`
- `set_compound`

Each setter validates the Lua value type and stores a typed NBT entry.

## Example

```lua
local component = require("component")
local nbt = require("nbt")

local data = nbt.patch(component.data)
local root = nbt.wrap({
  __nbt_type = "compound",
  __value = {}
})

root:set_string("id", "minecraft:stone")
root:set_int("Count", 64)

local encoded = data.encodeNBT(root)
local decoded = data.decodeNBT(encoded)
print(decoded.id, decoded.Count)
```

## Related

- `opencomputers-data-card`
- `opencomputers-openos-serialization`
