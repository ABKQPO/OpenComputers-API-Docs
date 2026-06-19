# appeng

## 概述

本页说明 Applied Energistics 2 与 OpenComputers 集成后暴露出来的 Lua API。
内容包括固定式 ME 网络组件、总线与接口配置、接口终端样板转移、合成控制、网络监控，以及机器人或无人机使用的 AE 升级。

## 可用性

- 依赖：`appeng`
- 标签：`integration-required`

## 组件名

- `me_controller`
- `me_exportbus`
- `me_importbus`
- `me_storagebus`
- `me_interface`
- `me_interface_terminal`
- `upgrade_me`

## 本集成常用的数据结构

### 物品表

大多数物品相关接口都会返回或接受基于 OpenComputers 标准物品转换器的表。
常见字段包括：

- `name`：注册名，例如 `minecraft:iron_ingot`
- `label`：本地化显示名称
- `damage`：元数据或耐久值
- `size`：堆叠数量
- `maxSize`：最大堆叠数量
- `maxDamage`：最大耐久
- `hasTag`：是否带有 NBT
- `id`：数值物品 id，仅在 OC 配置允许时出现
- `oreNames`：矿辞名数组，仅在 OC 配置允许时出现
- `enchantments`、`lore`、`tag`：存在且配置允许时出现

AE 物品表还会额外包含：

- `isCraftable`：该条目是否代表一个可合成目标

### 流体表

常见字段包括：

- `name`：流体内部名，例如 `water`
- `label`：本地化流体名
- `amount`：毫桶数
- `hasTag`：是否带有 NBT
- `id`：数值流体 id，仅在 OC 配置允许时出现

AE 流体表还会额外包含：

- `size`：AE 栈大小
- `isCraftable`：该条目是否代表一个可合成目标

### 位置表

接口终端使用的位置表字段如下：

- `x`
- `y`
- `z`
- `dimId`：可选，默认使用当前终端所在维度

### 样板表

`getInterfacePattern` 返回编码样板物品。
它转换后的数据通常包含：

- `inputs`：输入物品数组
- `outputs`：输出物品数组
- `isCraftable`：该样板是否为合成样板

## 共享网络控制接口

以下接口既可在固定式 `me_controller` 上使用，也可在 `upgrade_me` 已成功接入 AE 网络后使用。

### `getCpus`

- 语法：`me.getCpus()`
- 返回：CPU 描述表数组
- 用途：列出当前网络可见的全部 AE 合成 CPU 集群

每个描述表通常包含：

- `name`
- `storage`
- `coprocessors`
- `busy`
- `cpu`：用于进一步查询的 userdata 句柄

示例：

```lua
for _, info in ipairs(component.me_controller.getCpus()) do
  print(info.name, info.storage, info.busy)
end
```

### `getCraftables`

- 语法：`me.getCraftables([filter])`
- 参数：
  - `filter`：可选过滤表，会与合成结果转换后的字段进行匹配
- 返回：`Craftable` userdata 数组
- 用途：列出当前网络已知的可合成目标

过滤采用精确匹配。常用键通常是 `name`、`label`、`damage`，以及在启用对应 OC 配置时可见的 `id`。

示例：

```lua
local craftables = component.me_controller.getCraftables({label = "Printed Silicon"})
print(#craftables)
```

### `getCraftable`

- 语法：`me.getCraftable(detail, type)`
- 参数：
  - `detail`：一个 AE 栈描述表
  - `type`：已注册的 AE 栈类型 id
- 返回：`Craftable` userdata 或 `nil`
- 用途：直接解析一个明确的可合成目标

说明：

- 源码会根据 `type` 指定的已注册 AE 栈类型，对 `detail` 做对应解析。
- 在普通 AE2 物品网络里，`type` 通常就是物品栈类型 id。
- 其他集成还可以继续注册新的 AE 栈类型，因此这个接口并不只限于普通物品。

示例：

```lua
local craftable = component.me_controller.getCraftable(
  {name = "minecraft:glass", damage = 0, size = 1},
  "item"
)
```

### `getItemsInNetwork`

- 语法：`me.getItemsInNetwork([filter])`
- 参数：
  - `filter`：可选物品过滤表
- 返回：转换后的物品表数组
- 用途：列出网络中当前存储的物品

示例：

```lua
local items = component.me_controller.getItemsInNetwork({name = "minecraft:iron_ingot"})
for _, stack in ipairs(items) do
  print(stack.label, stack.size)
end
```

### `getItemInNetwork`

- 语法：`me.getItemInNetwork(nameOrId[, damage[, nbtJson]])`
- 语法：`me.getItemInNetwork(detail)`
- 参数：
  - `nameOrId`：注册名字串或数值物品 id
  - `damage`：可选元数据，默认 `0`
  - `nbtJson`：可选 SNBT 字符串，默认 `{}`
  - `detail`：通过已注册 AE 物品栈解析器读取的物品描述表
- 返回：转换后的物品表，找不到时返回 `nil`
- 用途：精确查询某一种物品变体

说明：

- 字符串 / id 形式会先构造临时 `ItemStack`，再把第三个参数按 Minecraft 的 SNBT 规则解析。
- 表描述符形式是当前源码已经明确支持的新路径，适合直接拿现有描述表做精确查找。
- 精确查询因此不再必须只用“名称 + damage + NBT 文本”这一种表达方式。

示例：

```lua
local stack = component.me_controller.getItemInNetwork("minecraft:wool", 14)
if stack then
  print(stack.label, stack.size)
end
```

```lua
local exact = component.me_controller.getItemInNetwork({
  name = "minecraft:wool",
  damage = 14,
  size = 1
})
```

### `getItemsInNetworkById`

- 语法：`me.getItemsInNetworkById(filterIds)`
- 参数：
  - `filterIds`：由注册名或数值 id 组成的数组
- 返回：转换后的物品表数组
- 用途：按基础物品类型批量过滤网络中的存储条目

这个接口只按“物品种类”过滤，不区分元数据和 NBT。

示例：

```lua
local items = component.me_controller.getItemsInNetworkById({
  "minecraft:cobblestone",
  "minecraft:stone"
})
```

### `getFluidInNetwork`

- 语法：`me.getFluidInNetwork([name])`
- 语法：`me.getFluidInNetwork(detail)`
- 参数：
  - `name`：可选流体名，例如 `water`
  - `detail`：通过已注册 AE 流体栈解析器读取的流体描述表
- 返回：转换后的流体表，找不到时返回 `nil`
- 用途：查询网络中的某一种流体

说明：

- 描述符表形式和物品精确查找保持一致，适合把之前转换过的流体描述再次传回源码。
- 如果指定名称的流体不存在，则返回 `nil`。

示例：

```lua
local water = component.me_controller.getFluidInNetwork("water")
print(water and water.amount or 0)
```

```lua
local fluid = component.me_controller.getFluidInNetwork({name = "water", amount = 1})
```

### `allItems`

- 语法：`me.allItems()`
- 返回：迭代器 userdata
- 用途：按条目逐个遍历网络物品，而不是一次性构造巨大的 Lua 数组

每次调用迭代器会返回一个转换后的物品表，直到返回 `nil` 为止。
在非常大的 AE 网络里，这种方式比一次性拉完整数组更稳妥，也更省内存。

示例：

```lua
local it = component.me_controller.allItems()
while true do
  local stack = it()
  if not stack then break end
  print(stack.label, stack.size)
end
```

### `store`

- 语法：`me.store(filter, databaseAddress[, startSlot[, count]])`
- 参数：
  - `filter`：物品过滤表
  - `databaseAddress`：目标数据库组件地址
  - `startSlot`：可选的一基起始槽位，默认 `1`
  - `count`：可选的最大复制条目数
- 返回：`true`
- 用途：把网络中匹配的物品描述复制到数据库里

写入时会跳过已经占用的数据库槽位，继续向后寻找空槽。

示例：

```lua
local ok = component.me_controller.store(
  {label = "Iron Ingot"},
  database.address,
  1,
  8
)
```

### `getFluidsInNetwork`

- 语法：`me.getFluidsInNetwork()`
- 返回：转换后的流体表数组
- 用途：列出网络中当前可见的全部流体

部分隐藏或虚拟流体条目会被兼容层过滤掉。

示例：

```lua
for _, fluid in ipairs(component.me_controller.getFluidsInNetwork()) do
  print(fluid.label, fluid.amount)
end
```

### `getAvgPowerInjection`

- 语法：`me.getAvgPowerInjection()`
- 返回：数字
- 用途：读取网络平均供能输入

### `getAvgPowerUsage`

- 语法：`me.getAvgPowerUsage()`
- 返回：数字
- 用途：读取网络平均耗能

### `getIdlePowerUsage`

- 语法：`me.getIdlePowerUsage()`
- 返回：数字
- 用途：读取网络空闲耗能

### `getMaxStoredPower`

- 语法：`me.getMaxStoredPower()`
- 返回：数字
- 用途：读取网络总能量容量

### `getStoredPower`

- 语法：`me.getStoredPower()`
- 返回：数字
- 用途：读取当前已存储的 AE 能量

示例：

```lua
local me = component.me_controller
print(me.getStoredPower(), "/", me.getMaxStoredPower())
```

### `setFluidEventSubscription`

- 语法：`me.setFluidEventSubscription(enabled)`
- 参数：
  - `enabled`：布尔值
- 返回：无
- 用途：订阅或取消订阅 `network_fluid_changed` 事件

启用后，组件会通过 `computer.signal` 发出事件。
第一个参数是事件名，后续参数是发生变化的流体条目描述表。

### `isFluidEventSubscription`

- 语法：`me.isFluidEventSubscription()`
- 返回：布尔值
- 用途：检查当前是否启用了流体变化订阅

### `setItemEventSubscription`

- 语法：`me.setItemEventSubscription(enabled)`
- 参数：
  - `enabled`：布尔值
- 返回：无
- 用途：订阅或取消订阅 `network_item_changed` 事件

### `isItemEventSubscription`

- 语法：`me.isItemEventSubscription()`
- 返回：布尔值
- 用途：检查当前是否启用了物品变化订阅

示例：

```lua
local event = require("event")
local me = component.me_controller

me.setItemEventSubscription(true)
local _, _, changed = event.pull("network_item_changed")
print(changed.label, changed.size)
```

## `me_controller`

`me_controller` 只暴露上文列出的共享网络控制接口。
它适合安装在固定机器上，用来直接读取或控制一个 AE 网络。

## `me_exportbus`

`me_exportbus` 用于配置安装在 multipart 宿主上的 AE 导出总线。
涉及方向参数时，使用普通 Forge 方向编号 `0` 到 `5`。

### `getExportConfiguration`

- 语法：`bus.getExportConfiguration(side[, slot])`
- 参数：
  - `side`：导出总线安装方向
  - `slot`：可选的一基配置槽位，默认 `1`
- 返回：该槽位的物品描述表，未配置时返回 `nil`
- 用途：读取一个导出过滤槽
- 说明：
  - 非法槽位会抛出 `invalid slot`。

### `setExportConfiguration`

- 语法：`bus.setExportConfiguration(side[, slot][, databaseAddress, entry])`
- 语法：`bus.setExportConfiguration(side[, slot][, detail])`
- 参数：
  - `side`：总线方向
  - `slot`：可选的一基配置槽位，默认 `1`
  - `databaseAddress`、`entry`：从数据库复制物品描述
  - `detail`：直接传入物品描述表
- 返回：`true`
- 用途：设置或清空一个导出过滤槽

省略描述参数时会清空该槽位。
非法槽位会抛出 `invalid slot`。

### `getExportSlotSize`

- 语法：`bus.getExportSlotSize(side)`
- 返回：数字
- 用途：读取当前可用的配置槽数量

可用槽位数会受到 AE 容量升级影响。

### `getExportOreFilter`

- 语法：`bus.getExportOreFilter(side)`
- 返回：字符串
- 用途：读取矿辞过滤字符串

### `setExportOreFilter`

- 语法：`bus.setExportOreFilter(side, filter)`
- 参数：
  - `filter`：AE2 可接受的矿辞名或过滤字符串
- 返回：`true`
- 用途：修改导出总线的矿辞过滤规则

### `exportIntoSlot`

- 语法：`bus.exportIntoSlot(side, slot)`
- 参数：
  - `side`：总线方向
  - `slot`：相邻库存中的一基目标槽位
- 返回：成功导出时为 `true`，没有导出任何东西时为 `false`，没有相邻库存时返回 `nil, "no inventory"`
- 用途：立即执行一次定向导出，强制尝试把内容塞入指定库存槽位

成功完成一轮导出后，这个调用会暂停大约 `0.25` 秒。

示例：

```lua
local bus = component.me_exportbus
bus.setExportConfiguration(sides.north, 1, {name = "minecraft:glass"})
print(bus.exportIntoSlot(sides.north, 1))
```

## `me_importbus`

`me_importbus` 用于配置安装在 multipart 宿主上的 AE 导入总线。

### `getImportConfiguration`

- 语法：`bus.getImportConfiguration(side[, slot])`
- 返回：该槽位的物品描述表，未配置时返回 `nil`
- 用途：读取一个导入过滤槽
- 说明：
  - 非法槽位会抛出 `invalid slot`。

### `setImportConfiguration`

- 语法：`bus.setImportConfiguration(side[, slot][, databaseAddress, entry])`
- 语法：`bus.setImportConfiguration(side[, slot][, detail])`
- 返回：`true`
- 用途：设置或清空一个导入过滤槽
- 说明：
  - 省略描述参数时会清空对应槽位。
  - 非法槽位会抛出 `invalid slot`。

### `getImportSlotSize`

- 语法：`bus.getImportSlotSize(side)`
- 返回：数字
- 用途：读取当前可用的导入过滤槽数量

### `getImportOreFilter`

- 语法：`bus.getImportOreFilter(side)`
- 返回：字符串
- 用途：读取矿辞过滤字符串

### `setImportOreFilter`

- 语法：`bus.setImportOreFilter(side, filter)`
- 返回：`true`
- 用途：修改矿辞过滤规则

示例：

```lua
local bus = component.me_importbus
bus.setImportConfiguration(sides.up, 1, {name = "minecraft:clay_ball"})
print(bus.getImportSlotSize(sides.up))
```

## `me_storagebus`

`me_storagebus` 用于配置安装在 multipart 宿主上的 AE 存储总线。

### `getStorageConfiguration`

- 语法：`bus.getStorageConfiguration(side[, slot])`
- 返回：该槽位的物品描述表，未配置时返回 `nil`
- 用途：读取一个存储总线过滤槽
- 说明：
  - 非法槽位会抛出 `invalid slot`。

### `setStorageConfiguration`

- 语法：`bus.setStorageConfiguration(side[, slot][, databaseAddress, entry])`
- 语法：`bus.setStorageConfiguration(side[, slot][, detail])`
- 返回：`true`
- 用途：设置或清空一个存储总线过滤槽
- 说明：
  - 省略描述参数时会清空对应槽位。
  - 非法槽位会抛出 `invalid slot`。

### `getStorageOreFilter`

- 语法：`bus.getStorageOreFilter(side)`
- 返回：字符串
- 用途：读取存储总线矿辞过滤字符串

### `setStorageOreFilter`

- 语法：`bus.setStorageOreFilter(side, filter)`
- 返回：`true`
- 用途：修改存储总线矿辞过滤规则

### `getStorageSlotSize`

- 语法：`bus.getStorageSlotSize(side)`
- 返回：数字
- 用途：读取当前可用的配置槽数量

存储总线的基础配置槽通常比导入或导出总线更多，容量升级还能继续增加上限。

## `me_interface`

`me_interface` 同时覆盖完整 AE 接口方块和 multipart 接口。
完整方块接口不带前置 `side` 参数，multipart 接口则必须显式传入。

### `getInterfaceConfiguration`

- 语法：`iface.getInterfaceConfiguration([slot])`
- 语法：`iface.getInterfaceConfiguration(side[, slot])`
- 参数：
  - `slot`：可选的一基配置槽位，默认 `1`
- 返回：物品描述表，未配置时返回 `nil`
- 用途：读取一个接口配置槽

### `setInterfaceConfiguration`

- 语法：`iface.setInterfaceConfiguration([slot][, databaseAddress, entry[, size]])`
- 语法：`iface.setInterfaceConfiguration([slot][, detail])`
- 语法：`iface.setInterfaceConfiguration(side[, slot][, databaseAddress, entry[, size]])`
- 语法：`iface.setInterfaceConfiguration(side[, slot][, detail])`
- 返回：`true`
- 用途：设置或清空一个接口配置槽

不传描述参数时会清空该槽位。

### `getInterfacePattern`

- 语法：`iface.getInterfacePattern([slot])`
- 语法：`iface.getInterfacePattern(side[, slot])`
- 参数：
  - `slot`：一基样板槽位，默认 `1`
- 返回：编码样板物品，未放入样板时返回 `nil`
- 用途：读取接口中的一个编码样板

### `setInterfacePatternInput`

- 语法：`iface.setInterfacePatternInput(slot, index[, databaseAddress, entry, size])`
- 语法：`iface.setInterfacePatternInput(slot, index[, detail, type])`
- 用途：覆盖编码样板中的一个输入项

参数说明：

- `slot`：接口内的一基样板槽位
- `index`：样板输入列表中的一基索引
- `databaseAddress`、`entry`、`size`：从数据库槽位复制描述
- `detail`：物品或流体描述表
- `type`：AE 栈类型 id，例如已注册的物品类型或流体类型

返回 `true` 表示写入成功。
如果不传新的描述参数，则会清空对应输入项。

常见失败情况：

- 接口槽位非法
- 样板内索引非法
- 指定槽位里没有编码样板
- AE 栈类型字符串无法识别

### `setInterfacePatternOutput`

- 语法：`iface.setInterfacePatternOutput(slot, index[, databaseAddress, entry, size])`
- 语法：`iface.setInterfacePatternOutput(slot, index[, detail, type])`
- 返回：`true`
- 用途：覆盖编码样板中的一个输出项

行为与 `setInterfacePatternInput` 相同，只是修改输出列表。

### `storeInterfacePatternInput`

- 语法：`iface.storeInterfacePatternInput(slot, index, databaseAddress, entry)`
- 返回：`true`
- 用途：把样板中的一个输入项复制到数据库槽位

复制到数据库时，堆叠数量会被规范为 `1`。

### `storeInterfacePatternOutput`

- 语法：`iface.storeInterfacePatternOutput(slot, index, databaseAddress, entry)`
- 返回：`true`
- 用途：把样板中的一个输出项复制到数据库槽位

### `clearInterfacePatternInput`

- 语法：`iface.clearInterfacePatternInput(slot, index)`
- 返回：`true`
- 用途：删除完整方块接口样板中的一个输入项

这个接口只存在于完整方块接口版本。

### `clearInterfacePatternOutput`

- 语法：`iface.clearInterfacePatternOutput(slot, index)`
- 返回：`true`
- 用途：删除完整方块接口样板中的一个输出项

这个接口只存在于完整方块接口版本。

示例：

```lua
local iface = component.me_interface
local pattern = iface.getInterfacePattern(1)
if pattern then
  print(pattern.label)
end
```

## `me_interface_terminal`

`me_interface_terminal` 用于发现网络中可见的接口，并在它们之间转移编码样板。

### `getInterfaces`

- 语法：`term.getInterfaces()`
- 返回：迭代器 userdata
- 用途：抓取当前网络中全部可见接口的快照

### `getInterfacesByName`

- 语法：`term.getInterfacesByName(filter)`
- 参数：
  - `filter`：精确显示名
- 返回：迭代器 userdata
- 用途：按显示名称筛选接口快照

### `getInterfacesByLocation`

- 语法：`term.getInterfacesByLocation(location[, side])`
- 参数：
  - `location`：包含 `x`、`y`、`z` 以及可选 `dimId` 的位置表
  - `side`：可选的 Forge 方向，既可传数字也可传字符串
- 返回：迭代器 userdata
- 用途：查找某个精确位置上的接口

### `send`

- 语法：`term.send(source, target)`
- 参数：
  - `source`：包含 `location`、可选 `side`、以及必填 `slot` 的表
  - `target`：包含 `location`、可选 `side`、以及可选 `slot` 的表
- 返回：成功时返回 `true, slot`，失败时返回 `false, error`
- 用途：把一个编码样板从源接口移动到目标接口

如果 `target.slot` 省略，则会自动寻找目标接口中的第一个空槽。

常见错误包括：

- `Source slot cannot be empty`
  源数据里没有提供样板槽位。
- `Source slot out of bounds`
  源接口槽位越界。
- `Can't find source pattern`
  源槽位中没有编码样板。
- `Target slot out of bounds`
  目标接口槽位越界。
- `Target slot is not empty`
  指定目标槽位已经被占用。
- `No empty slot in target machine`
  未指定目标槽位时，目标接口里也没有空槽可用。
- `Source machine not found`
  没找到源接口。
- `Target machine not found`
  没找到目标接口。
- `Both machines not found`
  源接口和目标接口都没找到。

### `sendBatch`

- 语法：`term.sendBatch(tasks)`
- 参数：
  - `tasks`：由 `{source = ..., target = ...}` 组成的任务数组
- 返回：与输入顺序一致的结果元组数组
- 用途：一次调用中批量执行多条样板转移

### 迭代器 userdata

上面三个查询接口返回的 userdata 还带有以下行为。

### `iterator()`

- 语法：`iterator()`
- 返回：当前记录，然后依次返回下一条，最后返回 `nil`
- 用途：顺序迭代接口快照

每条记录包含：

- `name`
- `location`
- `side`
- `patterns`：当包含样板内容时，这是一个以零基样板槽位为键的映射表

### `iterator(index)`

- 语法：`iterator(index)`
- 参数：
  - `index`：一基记录索引
- 返回：对应的接口记录，越界时返回 `nil`
- 用途：按索引随机访问快照内容

### `iterator("n")`

- 语法：`iterator("n")`
- 返回：数字
- 用途：通过调用运算符读取快照长度

### `reset`

- 语法：`iterator.reset()`
- 返回：无
- 用途：把顺序迭代位置重置到开头

### `count`

- 语法：`iterator.count()`
- 返回：数字
- 用途：返回快照中接口数量

### `getAll`

- 语法：`iterator.getAll([includePatterns])`
- 参数：
  - `includePatterns`：可选布尔值，默认 `false`
- 返回：全部接口记录组成的数组
- 用途：一次性把整份快照展开到 Lua

`getAll(true)` 在大型网络里开销较大，因为它会把每个接口的样板库存一起序列化出来。

示例：

```lua
local term = component.me_interface_terminal
local interfaces = term.getInterfaces()
print("visible:", interfaces.count())

local first = interfaces(1)
if first then
  print(first.name, first.location.x, first.location.y, first.location.z)
end
```

## `upgrade_me`

`upgrade_me` 是安装在机器人或无人机上的 AE 升级。
除了共享网络控制接口之外，它还额外提供与当前选中物品槽或流体槽直接交换数据的接口。

要正常工作，它必须已经绑定到 AE 安全终端，并处于有效无线接入点的可通信范围内。

### `sendItems`

- 语法：`up.sendItems([amount])`
- 参数：
  - `amount`：可选最大物品数量，默认 `64`
- 返回：实际送入网络的物品数量
- 用途：把当前选中物品槽中的物品送入 AE 网络

如果当前槽为空，或者升级未连入网络，则结果为 `0`。

### `requestItems`

- 语法：`up.requestItems(databaseAddress, entry[, amount])`
- 语法：`up.requestItems(detail)`
- 参数：
  - `databaseAddress`、`entry`：数据库中的物品描述
  - `amount`：可选请求数量
  - `detail`：物品描述表，至少需要 `name` 或 `id`，可附带 `damage` 与 `size`
- 返回：实际拉取到当前槽位的物品数量
- 用途：把匹配物品从 AE 网络取到当前选中的物品槽

重要行为：

- 如果当前选中槽位已经有不同类型的物品，结果直接为 `0`
- 直接传表且未指定 `size` 时，会默认按该槽允许的最大合法堆叠请求
- 实际提取量还会受到当前槽剩余空间限制

### `sendFluids`

- 语法：`up.sendFluids([amount])`
- 参数：
  - `amount`：可选最大毫桶数，默认按当前流体量或槽容量上限处理
- 返回：实际送入网络的毫桶数
- 用途：把当前选中流体槽中的流体送入 AE 网络

### `requestFluids`

- 语法：`up.requestFluids(databaseAddress, entry[, amount])`
- 语法：`up.requestFluids(detail)`
- 参数：
  - `databaseAddress`、`entry`：数据库中描述某种已装填流体容器的条目
  - `amount`：可选毫桶数
  - `detail`：流体描述表，至少需要 `name` 或 `id`，可选 `size`
- 返回：实际灌入当前槽位的毫桶数
- 用途：从 AE 网络中提取流体并填入当前选中槽

若直接传表且未指定 `size`，默认请求量为 `1000 mB`，也就是一桶。

### `isLinked`

- 语法：`up.isLinked()`
- 返回：布尔值
- 用途：判断当前升级是否能够连接到已绑定的 AE 网络

示例：

```lua
local me = component.upgrade_me
if me.isLinked() then
  print("sent", me.sendItems(16))
end
```

## 返回的 userdata

### `Craftable`

由 `getCraftable` 和 `getCraftables` 返回。

#### `getStack`

- 语法：`craftable.getStack()`
- 返回：转换后的结果物品表
- 用途：读取这个可合成目标对应的产物描述

#### `request`

- 语法：`craftable.request([amount[, prioritizePower[, cpuName]]])`
- 参数：
  - `amount`：可选产物数量，默认 `1`
  - `prioritizePower`：可选布尔值，默认 `true`
  - `cpuName`：可选的精确 CPU 名称
- 返回：`CraftingStatus` userdata；若源控制器已经失效，则返回 `nil, "no controller"`
- 用途：发起一次异步 AE 合成请求

请求会先异步计算合成计划，再提交给 AE 网络执行。
后续若失败，通常会在状态对象里看到类似资源不足或提交异常的原因。

### `CPU`

通过 `getCpus` 返回的描述表中的 `cpu` 字段取得。

#### `cancel`

- 语法：`cpu.cancel()`
- 返回：布尔值
- 用途：取消该 CPU 当前正在执行的合成任务

当 CPU 本身不忙时返回 `false`。

#### `isActive`

- 语法：`cpu.isActive()`
- 返回：布尔值
- 用途：判断 CPU 集群当前是否处于活动状态

#### `isBusy`

- 语法：`cpu.isBusy()`
- 返回：布尔值
- 用途：判断 CPU 当前是否已经分配了任务

#### `activeItems`

- 语法：`cpu.activeItems()`
- 返回：AE 栈表数组
- 用途：列出当前正在被合成的项目

#### `pendingItems`

- 语法：`cpu.pendingItems()`
- 返回：AE 栈表数组
- 用途：列出当前仍在排队等待处理的项目

#### `storedItems`

- 语法：`cpu.storedItems()`
- 返回：AE 栈表数组
- 用途：列出当前合成任务缓存中已经存放的项目

#### `finalOutput`

- 语法：`cpu.finalOutput()`
- 返回：最终输出物品表；没有合成监视器时返回 `nil, "No crafting monitor"`；没有实际产物时返回 `nil, "Nothing is crafted"`
- 用途：读取该 CPU 合成监视器当前显示的最终产物

### `CraftingStatus`

由 `Craftable.request` 返回。

#### `isComputing`

- 语法：`status.isComputing()`
- 返回：布尔值
- 用途：判断 AE 是否仍在计算本次合成计划

#### `hasFailed`

- 语法：`status.hasFailed()`
- 返回：`boolean, reason`
- 用途：判断请求是否失败，并返回失败原因

常见原因通常形如 `request failed (missing resources)`，表示 AE 在计算或提交时发现材料不足。

#### `isCanceled`

- 语法：`status.isCanceled()`
- 返回：`boolean[, reason]`
- 用途：判断这次合成请求是否被取消

当请求仍在计算中时，返回 `false, "computing"`。

#### `isDone`

- 语法：`status.isDone()`
- 返回：`boolean[, reason]`
- 用途：判断这次合成请求是否已经完成

当请求仍在计算中时，返回 `false, "computing"`。

## 完整示例

```lua
local component = require("component")
local event = require("event")

local me = component.me_controller

print("Power:", me.getStoredPower(), "/", me.getMaxStoredPower())

for _, stack in ipairs(me.getItemsInNetwork({label = "Iron Ingot"})) do
  print(stack.label, stack.size)
end

local craftables = me.getCraftables({label = "Printed Silicon"})
if #craftables > 0 then
  local status = craftables[1].request(1, true)
  while status.isComputing() do
    os.sleep(0.1)
  end
  print("done:", status.isDone())
end

me.setItemEventSubscription(true)
local _, _, changed = event.pull(1, "network_item_changed")
if changed then
  print("changed:", changed.label, changed.size)
end
```

## 相关内容

- `ae2fc`
- `thaumicenergistics`
- `opencomputers-data-card`
