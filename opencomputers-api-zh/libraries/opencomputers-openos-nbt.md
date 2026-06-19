# nbt

## 概述

`nbt` 是 OpenOS 提供的一个辅助库，用来把数据卡上的 NBT 编码 / 解码回调包装成更适合 Lua 使用的代理层。
它会把带类型信息的 NBT 表转换成可直接字段访问的代理对象，并提供一个补丁函数，把组件上的 `encodeNBT` / `decodeNBT` 变成对这些代理友好的版本。

## 可用性

- 仓库: `opencomputers`
- 系统: `openos`
- 类型: `library`

## 来源

- `OpenComputers/src\main\resources\assets\opencomputers\loot\openos\lib\nbt.lua`

## 用法

```lua
local component = require("component")
local nbt = require("nbt")

local data = nbt.patch(component.data)
```

## 作用

- `patch(component)` 会包装一个已经提供 `encodeNBT` 与 `decodeNBT` 的组件。
- `wrap(value)` 会把原始带类型 NBT 表转换成 Lua 代理对象。
- `unwrap(value)` 会把代理对象还原回底层原始带类型表。
- 包装后的 compound / list 值可以像普通 Lua 表一样用索引访问，并支持 `pairs` 遍历。

## 接口

### `nbt.patch(component)`

- 语法: `nbt.patch(component)`
- 返回: 打过补丁的组件代理，或原始组件本身
- 用途: 用支持 NBT 代理的版本替换 `encodeNBT` 与 `decodeNBT`。
- 说明:
  - 目标组件必须同时提供 `encodeNBT` 和 `decodeNBT`。
  - 如果缺少其中任意一个回调，这个辅助库会向 `stderr` 写出警告，并原样返回传入组件。

### `nbt.wrap(value)`

- 语法: `nbt.wrap(value)`
- 返回: 包装后的代理对象，或原始值本身
- 用途: 把带类型的 NBT 表转换成 Lua 更易用的代理对象。
- 说明:
  - 标量类型标签会直接解包成普通 Lua 标量值。
  - compound 和 list 标签会变成代理表。

### `nbt.unwrap(value)`

- 语法: `nbt.unwrap(value)`
- 返回: 原始带类型表，或原始值本身
- 用途: 去掉代理层，还原出底层编码器期望的原始 NBT 结构。

## 代理行为

包装后的 compound 与 list 代理支持：

- `proxy[key]`
  读取嵌套标签，并自动完成包装。
- `proxy[key] = value`
  替换已有标签值，或写入另一个带类型标签。
- `pairs(proxy)`
  遍历键和值，并自动包装子节点。
- `#proxy`
  当底层值是数组式列表时，返回其长度。

代理对象还会提供一组类型化 setter，可用于 compound 或 list 值：

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

这些 setter 会先校验 Lua 值类型，再写入对应类型的 NBT 条目。

## 示例

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

## 相关内容

- `opencomputers-data-card`
- `opencomputers-openos-serialization`
