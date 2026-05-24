# sgcraft

## 概述

本页说明 SGCraft 提供的 OpenComputers 星门接口方块。这个接口会通过名为 `stargate` 的组件暴露星门控制回调；如果在接口方块中安装了 LAN 卡升级，还可以通过星门中继 OpenComputers 网络数据包。

## 可用性

- 依赖: `sgcraft`
- 标签: `integration-required`

## 组件名

- `stargate`

## 主要用途

- 读取本地星门状态、已锁定符文数量和连接方向。
- 查询当前可用能量，并估算拨号所需能量。
- 用 Lua 拨号和断开连接。
- 控制安装在星门上的 Iris 升级。
- 向远端星门发送消息，并接收星门事件。
- 在接口方块安装 LAN 卡时，经由已连接星门中继 OpenComputers 无线网络数据包。

## 使用前提

- 将 OpenComputers Stargate Interface 方块放在已合并完成的星门旁边。
- 组件会以 `component.stargate` 的形式出现。
- 接口方块内部只有一个升级槽位。
- 在该槽位中放入 OpenComputers 的 `lanCard` 物品后，才能启用与星门相关的 OC 网络包中继功能。

## 组件

### stargate

#### `stargateState()`

- 语法: `stargateState(): string, number, string`
- 用途: 返回当前星门状态、已锁定符文数量以及连接方向。
- 返回:
  - `state: string`
    人类可读的星门状态描述。
  - `chevrons: number`
    当前已锁定的符文数量。
  - `direction: string`
    `Outgoing`、`Incoming`，或者未连接时返回空字符串。
- 说明:
  - 如果接口没有连到星门，会返回 `Offline, 0, ""`。

#### `energyAvailable()`

- 语法: `energyAvailable(): number`
- 用途: 返回当前附属星门可用的能量。
- 返回:
  - 当前可用于拨号的能量值。

#### `energyToDial(address)`

- 语法: `energyToDial(address: string): number[, string]`
- 用途: 估算拨打指定标准化地址所需的能量。
- 参数:
  - `address: string`
    目标星门地址。
- 返回:
  - 成功时返回所需能量。
  - 本地星门不存在、地址非法或找不到目标时，返回 `nil, message`。
- 说明:
  - 地址会在查找前先做标准化。
  - 能量消耗与本地门和目标门之间的距离有关。

#### `localAddress()`

- 语法: `localAddress(): string`
- 用途: 返回当前附属星门的本地地址。
- 返回:
  - 本地星门地址；如果接口未附属到星门，则返回空字符串。

#### `remoteAddress()`

- 语法: `remoteAddress(): string[, string]`
- 用途: 返回当前已连接远端星门的地址。
- 返回:
  - 已连接时返回远端地址。
  - 本地星门空闲时返回空字符串。
  - 若接口没有连接到星门，则返回 `nil, message`。

#### `dial(address)`

- 语法: `dial(address: string): boolean[, string]`
- 用途: 尝试向指定地址发起拨号。
- 参数:
  - `address: string`
    目标星门地址。
- 返回:
  - 成功时返回 `true`。
  - 拨号失败时返回 `nil, message`。
- 说明:
  - 地址在拨号前会先被标准化。
  - 星门内部返回的错误会作为第二返回值给出。

#### `disconnect()`

- 语法: `disconnect(): boolean[, string]`
- 用途: 断开当前星门连接。
- 返回:
  - 成功时返回 `true`。
  - 当前没有连接或断开失败时返回 `nil, message`。

#### `irisState()`

- 语法: `irisState(): string`
- 用途: 返回当前 Iris 状态描述。
- 返回:
  - Iris 状态字符串。
- 说明:
  - 如果星门没有安装 Iris 升级，会返回 `Offline`。

#### `openIris()`

- 语法: `openIris(): boolean[, string]`
- 用途: 开始打开 Iris。
- 返回:
  - 成功时返回 `true`。
  - 如果没有连接到星门，或者星门没有安装 Iris 升级，则返回 `nil, message`。

#### `closeIris()`

- 语法: `closeIris(): boolean[, string]`
- 用途: 开始关闭 Iris。
- 返回:
  - 成功时返回 `true`。
  - 如果没有连接到星门，或者星门没有安装 Iris 升级，则返回 `nil, message`。

#### `sendMessage(...)`

- 语法: `sendMessage(...): boolean[, string]`
- 用途: 向当前连接的远端星门发送一条星门应用层消息。
- 参数:
  - `...`
    零个或多个 Lua 值。接口会按对象数组原样转发。
- 返回:
  - 成功时返回 `true`。
  - 星门未连接或发送失败时返回 `nil, message`。
- 说明:
  - 接收端会通过 `sgMessageReceived` 事件收到这组参数。
  - 所有参数会按原始顺序传递，不会额外包成表。

## 事件

这个接口会通过 `computer.signal` 向可达的 OpenComputers 机器发送以下事件。

#### 事件: `sgStargateStateChange`

- 语法: `sgStargateStateChange(newState, oldState)`
- 用途: 星门状态描述发生变化时触发。
- 参数:
  - `newState: string`
    新的人类可读状态。
  - `oldState: string`
    旧的人类可读状态。

#### 事件: `sgDialOut`

- 语法: `sgDialOut(address)`
- 用途: 本地星门开始向 `address` 发起外拨连接时触发。

#### 事件: `sgDialIn`

- 语法: `sgDialIn(address)`
- 用途: 本地星门接收到来自 `address` 的入站连接时触发。

#### 事件: `sgChevronEngaged`

- 语法: `sgChevronEngaged(chevronCount, symbol)`
- 用途: 拨号过程中每锁定一个新符文时触发。
- 参数:
  - `chevronCount: number`
    本次锁定后已锁定的符文数量。
  - `symbol: string`
    本次刚刚锁定的符号。

#### 事件: `sgIrisStateChange`

- 语法: `sgIrisStateChange(newState, oldState)`
- 用途: Iris 状态描述发生变化时触发。

#### 事件: `sgMessageReceived`

- 语法: `sgMessageReceived(...)`
- 用途: 接收到远端星门通过 `sendMessage(...)` 发来的消息时触发。
- 参数:
  - `...`
    发送端原样传来的参数列表。

## OpenComputers 网络中继

- 每一个星门基座方块都会在内部加入 OpenComputers 无线网络。
- 如果接口方块中安装了 LAN 卡，那么进入接口的 `network.message` 数据包可以沿着当前已连接的星门转发到远端。
- 只有当数据包 TTL 仍然大于零时，数据包才会继续被转发。
- 星门侧无线中继默认使用的重广播强度为 `50`。

## 示例

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

## 相关内容

- `stargatetech2`
- `network`
