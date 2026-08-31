# storagedrawers

## 概述

本页说明 OpenComputers 对 Storage Drawers 的集成接口。实现了抽屉组接口的方块会作为 Lua 组件暴露，可用于读取抽屉数量、库存占用和存储物品信息。

> 源码审计说明（2026-08-31）：当前 StorageDrawers 提交增加了控制器/从属方块对通用 OpenComputers 转置器的兼容。下述专用 `drawer` 组件为保持现有文档集兼容而保留，实际安装整合包中的可用接口仍应在运行时确认。

## 可用性

- 依赖: `storagedrawers`
- 标签: `integration-required`

## 源码审计

- 当前 OpenComputers 源码的 `src/main` 下没有 `storagedrawers` 或 `drawer` 驱动。
- 当前 StorageDrawers 源码没有专用的 OpenComputers API 导入、回调或组件注册。
- StorageDrawers 提交 `59b527f` 让控制器/从属方块可被通用 OpenComputers 转置器使用；它没有注册 `drawer` 组件，也没有增加 Storage Drawers 专用 Lua API。

## 运行时确认说明

下方 `drawer` API 保留自现有文档集。由于本次源码扫描没有找到其专用驱动，请使用 `component.list()` 和 `component.methods(address)` 确认当前安装整合包是否暴露这些接口。

## 组件

### drawer

`drawer` 组件对应实现了 Storage Drawers `IDrawerGroup` 接口的方块。

#### `getDrawerCount()`

- 语法: `getDrawerCount(): number`
- 用途: 返回当前方块暴露出来的抽屉槽总数。
- 返回值:
  - 抽屉槽数量。

#### `getItemCount(drawerSlot)`

- 语法: `getItemCount(drawerSlot: number): number`
- 用途: 返回指定抽屉槽当前存放的物品数量。
- 参数:
  - `drawerSlot: number`
    抽屉槽位编号，Lua 侧从 `1` 开始计数。
- 返回值:
  - 该槽位中的当前物品数量。
- 说明:
  - Lua 侧的一基索引会在驱动内部换算成 Java 的零基索引。
  - 访问不存在或被禁用的抽屉槽位会抛出 `no drawer found at slot N`。

#### `getMaxCapacity(drawerSlot)`

- 语法: `getMaxCapacity(drawerSlot: number): number`
- 用途: 返回指定抽屉槽理论上最多能容纳多少物品。
- 参数:
  - `drawerSlot: number`
    抽屉槽位编号，Lua 侧从 `1` 开始计数。
- 返回值:
  - 该槽位的最大存储容量。
- 说明:
  - 非法槽位同样会抛出 `no drawer found at slot N`。

#### `getItemName(drawerSlot)`

- 语法: `getItemName(drawerSlot: number): string | nil, string?`
- 用途: 返回指定抽屉槽当前物品的未本地化名称。
- 参数:
  - `drawerSlot: number`
    抽屉槽位编号，Lua 侧从 `1` 开始计数。
- 返回值:
  - 非空时返回未本地化名称，例如 `item.ingotIron`。
  - 空抽屉时返回 `nil` 和错误提示。
- 说明:
  - 这个接口返回的不是玩家界面里看到的本地化名称，而是源码使用的未本地化标识。
  - 空抽屉不会抛错，而是返回 `nil` 加消息。

#### `getItemDamage(drawerSlot)`

- 语法: `getItemDamage(drawerSlot: number): number | nil, string?`
- 用途: 返回指定抽屉槽当前物品的 damage 值或元数据值。
- 参数:
  - `drawerSlot: number`
    抽屉槽位编号，Lua 侧从 `1` 开始计数。
- 返回值:
  - 非空时返回数值 damage 或 metadata。
  - 空抽屉时返回 `nil` 和错误提示。
- 说明:
  - 这个值适合用来区分共用同一物品 ID 的不同子类型。
  - 某些极短暂的过渡状态下，非空抽屉也可能出现数量读取异常；如果要做稳健判断，最好把 `getItemName` 与 `getItemCount` 结合起来看。

#### 示例

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

仅当所安装整合包暴露了对应库存能力时，才可通过通用库存转置器访问。请在运行时用 `component.list()` 和 `component.methods(address)` 确认实际可用方法。

## 相关内容

- `betterstorage`
- `appeng`
