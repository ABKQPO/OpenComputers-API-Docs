# gregtech

## 概述

本页说明 OpenComputers 与 Computronics 提供的 GregTech / TecTech 集成接口。可用组件覆盖机器基础状态、电池缓冲器、数字箱、可配置编程电路的机器、设备信息读取以及 TecTech 多方块参数。

## 可用性

- 依赖: `gregtech`
- 标签: `integration-required`

## 组件名

- `gt_machine`
- `gt_batterybuffer`
- `digital_chest`
- `tt_machine`
- `bec_diode`
- `bec_io_node`
- `bec_storage`

## 主要用途

- 读取 GregTech 机器中储存的 EU 和蒸汽。
- 监控工作进度、活动状态和拥有者信息。
- 在支持的机器上查询或修改编程电路配置。
- 检查电池缓冲器里的电池状态。
- 读取数字箱内容和 TecTech 参数组。

## 组件

### gt_machine

`gt_machine` 是多个 GregTech 驱动共用的组件名。具体一台机器会暴露哪一部分接口，取决于它匹配到了哪些驱动。

#### 基于 BaseMetaTileEntity 的方法

这一组接口来自 `BaseMetaTileEntity` 驱动，适用于 GregTech 基础 meta tile。

##### getEUStored()

- 语法: `getEUStored(): number`
- 用途: 返回方块当前储存的 EU。

##### getSteamStored()

- 语法: `getSteamStored(): number`
- 用途: 返回方块当前储存的蒸汽。

##### getEUMaxStored()

- 语法: `getEUMaxStored(): number`
- 用途: 返回方块的最大 EU 容量。

##### getSteamMaxStored()

- 语法: `getSteamMaxStored(): number`
- 用途: 返回方块的最大蒸汽容量。

##### getEUInputAverage()

- 语法: `getEUInputAverage(): number`
- 用途: 返回方块的平均 EU 输入值。

##### getEUOutputAverage()

- 语法: `getEUOutputAverage(): number`
- 用途: 返回方块的平均 EU 输出值。

##### getOwnerName()

- 语法: `getOwnerName(): string`
- 用途: 返回方块记录的拥有者名称。

#### 基于 IMachineProgress 的方法

这一组接口来自 `IMachineProgress` 驱动，适用于会暴露工作进度的 GregTech 机器。

##### hasWork()

- 语法: `hasWork(): boolean`
- 用途: 返回机器当前是否有工作可做。

##### getWorkProgress()

- 语法: `getWorkProgress(): number`
- 用途: 返回机器当前进度值。

##### getWorkMaxProgress()

- 语法: `getWorkMaxProgress(): number`
- 用途: 返回当前工作周期的最大进度值。

##### isWorkAllowed()

- 语法: `isWorkAllowed(): boolean`
- 用途: 返回机器当前是否被允许工作。

##### setWorkAllowed(work)

- 语法: `setWorkAllowed(work: boolean)`
- 用途: 启用或禁用机器的工作许可。
- 参数:
  - `work`: `true` 表示允许工作，`false` 表示禁止工作。
- 返回值:
  - 成功时没有有意义的返回值。
- 使用说明:
  - 驱动只有在恰好收到一个布尔参数时才会真正改状态。

##### isMachineActive()

- 语法: `isMachineActive(): boolean`
- 用途: 返回机器当前是否处于活动状态。

##### getCoordinates()

- 语法: `getCoordinates(): number, number, number`
- 用途: 返回机器坐标。
- 返回值:
  - 当 Computronics 的 GregTech 坐标选项启用时，返回 `x, y, z`。
  - 当该选项关闭时，返回空结果列表。

##### getName()

- 语法: `getName(): string`
- 用途: 返回 GregTech meta tile 名称。

#### 设备信息方法

这个接口来自 `IGregTechDeviceInformation` 驱动，只有在目标方块当前声明自己“正在提供信息”时才会暴露。

##### getSensorInformation()

- 语法: `getSensorInformation(): table`
- 用途: 返回 GregTech 设备信息文本行列表。
- 返回值:
  - `getInfoData()` 返回的字符串数组。

#### 编程电路配置方法

这一组接口只在机器实现了 `IConfigurationCircuitSupport` 时暴露。

##### getCircuitConfiguration()

- 语法: `getCircuitConfiguration(): number | nil, string?`
- 用途: 返回当前安装的编程电路配置值。
- 返回值:
  - 如果已安装合法的集成电路，则返回配置号。
  - 如果电路槽有效，但当前为空或不是集成电路，则返回 `-1`。
  - 如果机器不支持该功能，则返回 `nil, "Machine does not support circuit configuration."`。
  - 如果机器报告的电路槽下标超出库存范围，则返回 `nil, "Invalid circuit slot index."`。

##### setCircuitConfiguration(config)

- 语法: `setCircuitConfiguration(config: number): boolean | nil, string`
- 用途: 安装、替换或移除编程电路。
- 参数:
  - `config`: 传 `1` 到 `24` 安装对应配置的电路，传 `-1` 移除当前电路。
- 返回值:
  - 成功时返回 `true`。
  - 不支持该功能时返回 `nil, "Machine does not support circuit configuration."`。
  - 电路槽无效时返回 `nil, "Invalid circuit slot index."`。
  - 传入非法值时返回 `nil, "Invalid circuit configuration value: ..."`。

### gt_batterybuffer

`gt_batterybuffer` 组件对应 GregTech 基础电池缓冲器。

#### getBatteryCharge(slot)

- 语法: `getBatteryCharge(slot: number): number | nil, string?`
- 用途: 返回指定槽位电池当前储存的 EU。
- 参数:
  - `slot`: 从 `1` 开始的槽位编号。
- 返回值:
  - 对于电力物品，返回当前电量。
  - 槽位越界时返回 `nil, "slot does not exist"`。
  - 槽位为空时返回 `nil, "slot is empty"`。
  - 槽内物品不是电力物品时返回 `nil, "item in slot is not electric"`。

#### getMaxBatteryCharge(slot)

- 语法: `getMaxBatteryCharge(slot: number): number | nil, string?`
- 用途: 返回指定槽位电池的最大 EU 容量。
- 参数:
  - `slot`: 从 `1` 开始的槽位编号。
- 返回值:
  - 对于电力物品，返回最大电量。
  - 槽位越界时返回 `nil, "slot does not exist"`。
  - 槽位为空时返回 `nil, "slot is empty"`。
  - 槽内物品不是电力物品时返回 `nil, "item in slot is not electric"`。

### digital_chest

`digital_chest` 组件对应 GregTech 数字箱。

#### getContents()

- 语法: `getContents(): table`
- 用途: 返回数字箱当前储存物品的数据表。
- 返回值:
  - `getStoredItemData()` 返回的原始结构。
- 使用说明:
  - 这个结构本身就是 GregTech 数字箱使用的存储描述格式，所以字段布局遵循 GregTech 的数据格式。

### tt_machine

`tt_machine` 组件对应继承自 `TTMultiblockBase` 的 TecTech 多方块机器。

#### getParameters(hatch, id)

- 语法: `getParameters(hatch: number, id: number): number`
- 用途: 返回一个 TecTech 参数输入槽当前的值。
- 参数:
  - `hatch`: 参数组索引。
  - `id`: 参数组内参数索引。

#### setParameters(hatch, id, value)

- 语法: `setParameters(hatch: number, id: number, value: number)`
- 用途: 向一个 TecTech 参数槽写入新值。
- 参数:
  - `hatch`: 参数组索引。
  - `id`: 参数组内参数索引。
  - `value`: 新的浮点数值。
- 返回值:
  - 成功时没有有意义的返回值。

### `bec_diode`

`bec_diode` 组件对应 TecTech 的玻色-爱因斯坦凝聚态二极管 meta tile。

#### `getCondensateFilter()`

- 语法: `getCondensateFilter(): string | nil`
- 用途: 返回凝聚态过滤器使用的流体注册名；未设置过滤器时返回 `nil`。

#### `setCondensateFilter(fluidName)`

- 语法: `setCondensateFilter(fluidName: string | nil)`
- 用途: 按流体注册名设置凝聚态过滤器。传入 `nil` 会清除过滤器。
- 错误:
  - 未知流体名会抛出 `Unknown fluid: ...`。

### `bec_io_node`

`bec_io_node` 组件对应 TecTech BEC 输入/输出节点。

#### 配方状态读取

- `getRequiredCondensate(): table | nil`: 返回当前配方所需的流体注册名到数量映射。
- `getConsumedCondensate(): table | nil`: 返回当前配方已消耗的流体注册名到数量映射。
- `getProvidedTier(): table | nil`: 返回该节点提供的纳米等级 `{name, tier}`。
- `getRequiredTier(): table | nil`: 返回当前配方要求的纳米等级 `{name, tier}`。
- `getAvailableNanites(): number`: 返回可用纳米数量。
- `getSlowdowns(): number`: 返回当前应用的减速次数。
- `getParallelRecipesInProgress(): number`: 返回正在执行的并行配方数量。
- `getMinParallel(): number`: 返回配置的最小并行配方数。
- `getMaxParallel(): number`: 返回配置的最大并行配方数。
- `getManualSlowdown(): number`: 返回手动减速值。
- `getRecipeSteps(): table | nil`: 返回配方步骤表，每步包含 `nanite`、`start`、`end` 和 `index`。
- `getState(): string`: 返回 `idle`、`unpowered`、`assembler-offline`、`nanite-tier-too-low`、`paused-step`、`paused-immediate`、`crafting` 或 `internal-error`。

#### `setMinParallel(min)`

- 语法: `setMinParallel(min: number)`
- 用途: 设置最小并行配方数。

#### `setMaxParallel(max)`

- 语法: `setMaxParallel(max: number)`
- 用途: 设置最大并行配方数。

#### `setSpeedDivisor(divisor)`

- 语法: `setSpeedDivisor(divisor: number)`
- 用途: 设置配方速度除数。

凝聚态表使用流体注册名到数值数量的映射。纳米等级表包含 `name` 和数值 `tier`；配方步骤表包含纳米等级以及 `start`、`end`、`index`。

### `bec_storage`

`bec_storage` 组件对应 TecTech BEC 存储节点。

#### `getFieldStrength()`

- 语法: `getFieldStrength(): number`
- 用途: 返回存储节点的场强。

#### `setFieldStrength(strength)`

- 语法: `setFieldStrength(strength: number)`
- 用途: 设置存储节点的场强。

#### `getStoredCondensate()`

- 语法: `getStoredCondensate(): table`
- 用途: 返回流体注册名到已存储凝聚态数量的映射表。

## 示例

```lua
local component = require("component")

if component.isAvailable("gt_machine") then
  local machine = component.gt_machine
  print("Stored EU:", machine.getEUStored())
  print("Machine active:", machine.isMachineActive and machine.isMachineActive())
end

if component.isAvailable("gt_batterybuffer") then
  local buffer = component.gt_batterybuffer
  local charge, err = buffer.getBatteryCharge(1)
  print("Slot 1:", charge or err)
end
```

## 相关内容

- `ic2`
- `gregtech`
