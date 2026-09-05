# openmodularturrets

## 概述

本页说明 Open Modular Turrets 的 OpenComputers 集成。每个炮塔基座等级都使用独立的组件名，但暴露相同的控制接口。

## 可用性

- 依赖: `openmodularturrets` 和 `OpenComputers`
- 附加要求: 炮塔基座中必须安装串口附加件（Serial Port Addon）。
- 标签: `integration-required`

## 组件名

- `tierOneTurretBase`
- `tierTwoTurretBase`
- `tierThreeTurretBase`
- `tierFourTurretBase`
- `tierFiveTurretBase`

这五个组件都暴露下面的回调。基座等级会影响库存、能量容量和炮塔能力，但不会改变 Lua 接口名称。

## 访问门控

只有在加载 OpenComputers 且基座安装串口附加件时，这些回调才会正常工作。电脑访问被禁用时，驱动会把 `Computer access deactivated!` 作为第一个返回值，而不会执行请求的操作。

## 回调

### `getOwner()`

- 语法: `getOwner(): string`
- 用途: 返回炮塔基座记录的拥有者名称。

### `isAttacksMobs()`

- 语法: `isAttacksMobs(): boolean`
- 用途: 返回基座是否配置为攻击敌对生物。

### `setAttacksMobs(value)`

- 语法: `setAttacksMobs(value: boolean)`
- 用途: 启用或禁用对敌对生物的攻击。
- 返回值: 成功时没有有意义的返回值。

### `isAttacksNeutrals()`

- 语法: `isAttacksNeutrals(): boolean`
- 用途: 返回基座是否配置为攻击中立生物。

### `setAttacksNeutrals(value)`

- 语法: `setAttacksNeutrals(value: boolean)`
- 用途: 启用或禁用对中立生物的攻击。
- 返回值: 成功时没有有意义的返回值。

### `isAttacksPlayers()`

- 语法: `isAttacksPlayers(): boolean`
- 用途: 返回基座是否配置为攻击玩家。

### `setAttacksPlayers(value)`

- 语法: `setAttacksPlayers(value: boolean)`
- 用途: 启用或禁用对玩家的攻击。
- 返回值: 成功时没有有意义的返回值。

### `getTrustedPlayers()`

- 语法: `getTrustedPlayers(): table`
- 用途: 返回信任玩家列表。
- 返回值:
  - 返回包含信任玩家记录的表。
  - 运行时转换为 Lua 后，记录会暴露公共字段 `name`、`canOpenGUI`、`canChangeTargeting`、`admin` 和 `uuid`。

### `addTrustedPlayer(name[, canOpenGUI[, canChangeTargeting[, admin]]])`

- 语法: `addTrustedPlayer(name: string[, canOpenGUI: boolean[, canChangeTargeting: boolean[, admin: boolean]]])`
- 用途: 将玩家加入信任列表，并可选设置权限。
- 参数:
  - `name`: 玩家名称。玩家必须符合服务器在线/离线模式的有效性规则。
  - `canOpenGUI`: 是否允许玩家打开基座 GUI，默认值为 `false`。
  - `canChangeTargeting`: 是否允许玩家修改目标设置，默认值为 `false`。
  - `admin`: 是否将玩家标记为管理员，默认值为 `false`。
- 返回值:
  - 成功时没有有意义的返回值。
  - 无法添加玩家时返回 `"Name not valid!"`。
- 实现说明: 当前 OpenComputers 回调实现会从第 2 个参数读取三个可选权限字段。分别传入 `canChangeTargeting` 或 `admin` 不会实际修改对应字段。

### `removeTrustedPlayer(name)`

- 语法: `removeTrustedPlayer(name: string)`
- 用途: 从信任列表中移除玩家。
- 参数:
  - `name`: 要移除的精确玩家名称。
- 返回值: 无论是否存在匹配记录，都没有有意义的返回值。

### `getMaxEnergyStorage()`

- 语法: `getMaxEnergyStorage(): number`
- 用途: 返回炮塔基座的最大能量容量，包括已配置的电力扩容件。

### `getCurrentEnergyStorage()`

- 语法: `getCurrentEnergyStorage(): number`
- 用途: 返回炮塔基座当前储存的能量。

### `getActive()`

- 语法: `getActive(): boolean`
- 用途: 返回炮塔基座当前是否处于激活状态。

### `setInverted(value)`

- 语法: `setInverted(value: boolean)`
- 用途: 设置基座的红石控制是否反转。
- 返回值: 成功时没有有意义的返回值。

### `getInverted()`

- 语法: `getInverted(): boolean`
- 用途: 返回红石控制是否反转。

### `getRedstone()`

- 语法: `getRedstone(): boolean`
- 用途: 返回基座当前检测到的红石状态。

## 示例

```lua
local component = require("component")

local name = "tierOneTurretBase"
if component.isAvailable(name) then
  local base = component[name]
  print("Owner:", base.getOwner())
  print("Energy:", base.getCurrentEnergyStorage(), "/", base.getMaxEnergyStorage())
  base.setAttacksMobs(true)
  base.setInverted(false)
end
```

## 相关内容

- `opencomputers`
- `gregtech`
