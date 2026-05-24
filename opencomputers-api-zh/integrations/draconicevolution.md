# draconicevolution

## 概述

本页说明 Draconic Evolution 自带的 OpenComputers 集成。这个模组并不是只暴露一个统一适配器，而是会根据不同方块暴露不同组件，因此可用接口取决于你连接的是哪一种 Draconic 方块。

## 可用性

- 依赖: `draconicevolution`
- 标签: `integration-required`

## 组件名

- `particle_generator`
- `draconic_rf_storage`
- `flux_gate`
- `fluid_gate`
- `draconic_reactor`

## 主要用途

- 通过能量塔读取 Draconic Evolution 储能核心的能量。
- 用 Lua 精细调整粒子发生器的粒子和光束效果。
- 用程序控制 Flux Gate 与 Fluid Gate，而不只依赖红石。
- 通过稳定器或注能器监控并控制 Draconic Reactor。

## 组件

### particle_generator

`particle_generator` 由粒子发生器方块暴露。

#### `setGeneratorProperty(property, value)`

- 语法: `setGeneratorProperty(property: string, value: boolean|number): boolean`
- 用途: 按属性名修改粒子发生器的一项设置。
- 参数:
  - `property: string`
    要修改的内部属性名。
  - `value: boolean|number`
    新值。不同属性要求的值类型不同。
- 返回:
  - `true` 表示属性名和值类型都被接受。
  - `false` 表示属性名未知、值类型错误，或参数数量不是正好两个。
- 说明:
  - 数值参数会按 Lua number 传入，再按需要转换成整数或浮点数。
  - 每次成功修改后都会立即应用并同步到方块。
  - 无效属性不会产生部分更新。

可设置的粒子属性:

- `particles_enabled: boolean`
  是否启用粒子生成。
- `red`, `green`, `blue: number`
  基础 RGB 颜色通道，范围会限制到 `0..255`。
- `random_red`, `random_green`, `random_blue: number`
  颜色随机增量，范围会限制到 `0..255`。
- `motion_x`, `motion_y`, `motion_z: number`
  基础速度向量，范围会限制到 `-5..5`。
- `random_motion_x`, `random_motion_y`, `random_motion_z: number`
  速度随机偏移，范围会限制到 `-5..5`。
- `scale: number`
  基础粒子大小，范围会限制到 `0.01..50`。
- `random_scale: number`
  随机附加大小，范围会限制到 `0.01..50`。
- `life: number`
  基础寿命，单位为 tick，范围会限制到 `0..1000`。
- `random_life: number`
  随机附加寿命，单位为 tick，范围会限制到 `0..1000`。
- `spawn_x`, `spawn_y`, `spawn_z: number`
  相对方块中心的基础生成偏移，范围会限制到 `-50..50`。
- `random_spawn_x`, `random_spawn_y`, `random_spawn_z: number`
  随机生成偏移，范围会限制到 `-50..50`。
- `fade: number`
  淡出时间，范围会限制到 `0..100`。
- `spawn_rate: number`
  生成间隔，范围会限制到 `1..200`。
- `collide: boolean`
  是否启用粒子碰撞。
- `selected_particle: number`
  粒子预设索引，范围会限制到 `1..3`。
- `gravity: number`
  粒子重力，范围会限制到 `-5..5`。

可设置的光束属性:

- `beam_enabled: boolean`
  是否启用光束渲染。
- `render_core: boolean`
  是否渲染光束核心。
- `beam_red`, `beam_green`, `beam_blue: number`
  光束 RGB 通道，范围会限制到 `0..255`。
- `beam_scale: number`
  光束宽度倍数，范围会限制到 `0..5`。
- `beam_pitch: number`
  光束上下角度，范围会限制到 `-180..180`。
- `beam_yaw: number`
  光束左右角度，范围会限制到 `-180..180`。
- `beam_length: number`
  光束长度，范围会限制到 `0..320`。
- `beam_rotation: number`
  光束纹理旋转速度，范围会限制到 `-1..1`。

#### `getGeneratorState()`

- 语法: `getGeneratorState(): table`
- 用途: 返回当前完整的发生器配置表。
- 返回:
  - 一个包含 `setGeneratorProperty` 所有可设置属性的键值表。
- 说明:
  - 返回表里的键名与上面的内部属性名完全一致。
  - 适合在修改前先做一次完整快照。

#### `resetGeneratorState()`

- 语法: `resetGeneratorState(): boolean`
- 用途: 将所有发生器设置恢复为默认值。
- 返回:
  - 重置完成后返回 `true`。
- 说明:
  - 这个调用会同时重置粒子设置和光束设置。
  - 重置后的关键默认值包括:
    `particles_enabled=true`、`scale=1`、`life=100`、`spawn_rate=1`、`selected_particle=1`、`beam_enabled=false`、`beam_scale=1`。

#### 示例

```lua
local component = require("component")
local gen = component.particle_generator

assert(gen.resetGeneratorState())
assert(gen.setGeneratorProperty("particles_enabled", true))
assert(gen.setGeneratorProperty("red", 32))
assert(gen.setGeneratorProperty("green", 180))
assert(gen.setGeneratorProperty("blue", 255))
assert(gen.setGeneratorProperty("spawn_y", 0.75))
assert(gen.setGeneratorProperty("motion_y", 0.03))
assert(gen.setGeneratorProperty("life", 40))

local state = gen.getGeneratorState()
print("Particle preset:", state.selected_particle)
print("RGB:", state.red, state.green, state.blue)
```

### draconic_rf_storage

`draconic_rf_storage` 由已连接储能核心的能量塔暴露。这个组件在源码中的真实名称就是 `draconic_rf_storage`，不是 `draconic_storage`。

#### `getEnergyStored()`

- 语法: `getEnergyStored(): number`
- 用途: 返回已连接储能核心的当前能量。
- 返回:
  - 当前存储能量。
- 说明:
  - 如果能量塔没有连到储能核心，会返回 `0`。
  - 返回值直接来自已连接核心，容量可以超过 32 位 RF 数值。

#### `getMaxEnergyStored()`

- 语法: `getMaxEnergyStored(): number`
- 用途: 返回已连接储能核心的总容量。
- 返回:
  - 最大可存储能量。
- 说明:
  - 如果能量塔没有连到储能核心，会返回 `0`。

#### 示例

```lua
local component = require("component")
local pylon = component.draconic_rf_storage

local stored = pylon.getEnergyStored()
local capacity = pylon.getMaxEnergyStored()

print(("Stored: %d / %d"):format(stored, capacity))
if capacity > 0 then
  print(("Fill: %.2f%%"):format(stored / capacity * 100))
end
```

### flux_gate

`flux_gate` 由 Flux Gate 方块暴露。

#### `getFlow()`

- 语法: `getFlow(): number`
- 用途: 返回 Gate 当前实际使用的流量值。
- 返回:
  - 当前生效的 RF/t 传输上限。
- 说明:
  - 如果启用了覆盖模式，返回覆盖值。
  - 否则会根据当前红石强度 `0..15`，在低流量与高流量之间做线性插值。

#### `setOverrideEnabled(enabled)`

- 语法: `setOverrideEnabled(enabled: boolean)`
- 用途: 开启或关闭软件覆盖模式。
- 参数:
  - `enabled: boolean`
    `true` 表示使用覆盖流量，`false` 表示使用红石插值流量。
- 返回:
  - 无返回值。

#### `getOverrideEnabled()`

- 语法: `getOverrideEnabled(): boolean`
- 用途: 返回当前是否启用了覆盖模式。
- 返回:
  - `true` 表示当前使用覆盖流量。
  - `false` 表示当前使用红石插值。

#### `setFlowOverride(flow)`

- 语法: `setFlowOverride(flow: number)`
- 用途: 设置软件直接控制的流量值。
- 参数:
  - `flow: number`
    覆盖模式启用时使用的 RF/t 上限。
- 返回:
  - 无返回值。
- 说明:
  - 这个值在源码中不会被自动限制。
  - 建议始终使用非负值，避免出现不可预期的传输行为。

#### `setSignalHighFlow(flow)`

- 语法: `setSignalHighFlow(flow: number)`
- 用途: 设置红石强度为 `15` 时使用的流量。
- 参数:
  - `flow: number`
    高信号 RF/t 上限。
- 返回:
  - 无返回值。

#### `getSignalHighFlow()`

- 语法: `getSignalHighFlow(): number`
- 用途: 返回当前配置的高信号流量。
- 返回:
  - 当前高信号 RF/t 值。

#### `setSignalLowFlow(flow)`

- 语法: `setSignalLowFlow(flow: number)`
- 用途: 设置红石强度为 `0` 时使用的流量。
- 参数:
  - `flow: number`
    低信号 RF/t 上限。
- 返回:
  - 无返回值。

#### `getSignalLowFlow()`

- 语法: `getSignalLowFlow(): number`
- 用途: 返回当前配置的低信号流量。
- 返回:
  - 当前低信号 RF/t 值。

#### 示例

```lua
local component = require("component")
local gate = component.flux_gate

gate.setSignalLowFlow(0)
gate.setSignalHighFlow(50000)
gate.setOverrideEnabled(true)
gate.setFlowOverride(12000)

print("Override active:", gate.getOverrideEnabled())
print("Current flow:", gate.getFlow(), "RF/t")
```

### fluid_gate

`fluid_gate` 由 Fluid Gate 方块暴露。

#### `getFlow()`

- 语法: `getFlow(): number`
- 用途: 返回 Gate 当前实际使用的流量值。
- 返回:
  - 当前生效的 mB/t 传输上限。
- 说明:
  - 控制逻辑与 `flux_gate` 完全相同，只是单位改成了毫桶每 tick。

#### `setOverrideEnabled(enabled)`

- 语法: `setOverrideEnabled(enabled: boolean)`
- 用途: 开启或关闭软件覆盖模式。
- 参数:
  - `enabled: boolean`
    `true` 表示使用覆盖流量，`false` 表示使用红石插值流量。
- 返回:
  - 无返回值。

#### `getOverrideEnabled()`

- 语法: `getOverrideEnabled(): boolean`
- 用途: 返回当前是否启用了覆盖模式。
- 返回:
  - `true` 表示当前使用覆盖流量。
  - `false` 表示当前使用红石插值。

#### `setFlowOverride(flow)`

- 语法: `setFlowOverride(flow: number)`
- 用途: 设置软件直接控制的流量值。
- 参数:
  - `flow: number`
    覆盖模式启用时使用的 mB/t 上限。
- 返回:
  - 无返回值。
- 说明:
  - 建议使用非负值，源码本身不会帮你校验。

#### `setSignalHighFlow(flow)`

- 语法: `setSignalHighFlow(flow: number)`
- 用途: 设置红石强度为 `15` 时使用的流量。
- 参数:
  - `flow: number`
    高信号 mB/t 上限。
- 返回:
  - 无返回值。

#### `getSignalHighFlow()`

- 语法: `getSignalHighFlow(): number`
- 用途: 返回当前配置的高信号流量。
- 返回:
  - 当前高信号 mB/t 值。

#### `setSignalLowFlow(flow)`

- 语法: `setSignalLowFlow(flow: number)`
- 用途: 设置红石强度为 `0` 时使用的流量。
- 参数:
  - `flow: number`
    低信号 mB/t 上限。
- 返回:
  - 无返回值。

#### `getSignalLowFlow()`

- 语法: `getSignalLowFlow(): number`
- 用途: 返回当前配置的低信号流量。
- 返回:
  - 当前低信号 mB/t 值。

#### 示例

```lua
local component = require("component")
local gate = component.fluid_gate

gate.setOverrideEnabled(false)
gate.setSignalLowFlow(0)
gate.setSignalHighFlow(4000)

print("Low signal flow:", gate.getSignalLowFlow(), "mB/t")
print("High signal flow:", gate.getSignalHighFlow(), "mB/t")
```

### draconic_reactor

`draconic_reactor` 由反应堆稳定器和反应堆能量注入器暴露。这两种方块都会把相同的四个方法转发到已连接的反应堆核心。

#### `getReactorInfo()`

- 语法: `getReactorInfo(): table`
- 用途: 返回当前反应堆运行状态表。
- 返回:
  - 一个包含以下键的表:
    - `temperature`
      反应堆温度，保留两位小数。
    - `fieldStrength`
      当前护盾场强，保留两位小数。
    - `maxFieldStrength`
      最大护盾场强，保留两位小数。
    - `energySaturation`
      当前内部能量饱和度。
    - `maxEnergySaturation`
      最大内部能量饱和度。
    - `fuelConversion`
      已转换燃料量，保留三位小数。
    - `maxFuelConversion`
      本轮可转换的总燃料量。
    - `generationRate`
      当前 RF/t 发电量。
    - `fieldDrainRate`
      当前 RF/t 护盾耗能。
    - `fuelConversionRate`
      当前燃料转换速率，数值等于真实速率乘以 `1,000,000`。
    - `status`
      可能的值为 `offline`、`charging`、`charged`、`online`、`stopping`、`invalid`。
- 说明:
  - 如果稳定器或注入器没有连接到反应堆核心，这个调用会返回 `nil`。
  - 在启动阶段中，状态会显示为 `charging` 或 `charged`。

#### `chargeReactor()`

- 语法: `chargeReactor(): boolean`
- 用途: 让反应堆进入启动充能阶段。
- 返回:
  - `true` 表示已接受请求。
  - `false` 表示当前状态不允许开始充能。
- 说明:
  - 只有在结构有效、反应堆尚未在线且燃料总量至少达到 `144` 时，才能开始充能。

#### `activateReactor()`

- 语法: `activateReactor(): boolean`
- 用途: 将已充能的反应堆切换为在线运行。
- 返回:
  - `true` 表示成功进入在线状态。
  - `false` 表示启动条件尚未满足。
- 说明:
  - 需要结构有效、护盾场强至少达到一半、能量饱和度至少达到一半、温度达到 `2000` 以上，并且总燃料不少于 `144`。

#### `stopReactor()`

- 语法: `stopReactor(): boolean`
- 用途: 启动受控停机流程。
- 返回:
  - `true` 表示成功接受停机请求。
  - `false` 表示当前无法停机。
- 说明:
  - 这会进入安全停机状态，不是瞬间断电。

#### 示例

```lua
local component = require("component")
local reactor = component.draconic_reactor

local info = reactor.getReactorInfo()
if not info then
  error("No linked reactor core")
end

print("Status:", info.status)
print("Temperature:", info.temperature)
print("Shield:", info.fieldStrength, "/", info.maxFieldStrength)

if info.status == "charged" then
  assert(reactor.activateReactor())
end
```

## 相关内容

- `mekanism`
- `cofh`
