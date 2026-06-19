# thaumicenergistics

## 概述

本页说明 OpenComputers 向 Lua 暴露的 Thaumic Energistics 集成接口。它在 AE 风格集成模型的基础上，加入了面向 essentia 的总线与 essentia 网络查询能力。

## 可用性

- 依赖: `thaumicenergistics`
- 标签: `integration-required`

## 组件名

- `essentia_exportbus`
- `essentia_importbus`
- `essentia_storagebus`
- `me_controller`
- `me_interface`

## 主要用途

- 配置 essentia 导出总线、导入总线和存储总线。
- 读取已连接 Thaumic Energistics 网络中的 essentia。
- 订阅 essentia 变化事件。
- 在处理 essentia 时继续沿用 AE 风格的控制器与接口环境。

## Essentia 描述表

返回的 essentia 栈表通常包含:

- `name`
  Thaumcraft aspect id。
- `amount`
  当前数量。
- `label`
  本地化后的 aspect 名称。

当接口接受描述表时，关键字段是:

- `name: string`
  必填的 aspect id。
- `amount: number`
  可选数量。省略时默认是 `0`。

如果给出的 aspect 不存在，调用会触发参数错误。

## 组件

### essentia_exportbus

`essentia_exportbus` 用于 Thaumic Energistics 的 essentia 导出总线部件。

#### `getExportConfiguration(side[, slot])`

- 语法: `getExportConfiguration(side: number[, slot: number]): table`
- 用途: 返回某个导出槽中配置的 aspect。
- 参数:
  - `side: number`
    安装该导出总线的 multipart 方向。
  - `slot: number`
    可选配置槽编号，默认是 `0`。
- 返回值:
  - 该槽位的 essentia 描述表。

#### `setExportConfiguration(side[, slot][, detail])`

- 语法: `setExportConfiguration(side: number[, slot: number][, detail: table]): boolean`
- 用途: 设置或清空一个导出配置槽。
- 参数:
  - `side: number`
    安装该导出总线的 multipart 方向。
  - `slot: number`
    可选槽位编号，默认是 `0`。
  - `detail: table`
    可选 essentia 描述表。
- 返回值:
  - `true` 表示成功。
- 说明:
  - 省略 `detail` 时会清空该槽位。

#### `getExportSlotSize(side)`

- 语法: `getExportSlotSize(side: number): number`
- 用途: 返回当前有效的导出配置槽数量。
- 参数:
  - `side: number`
    安装该导出总线的 multipart 方向。
- 返回值:
  - 在 AE 容量升级规则作用后的槽位数量。

#### `getExportOreFilter(side)`

- 语法: `getExportOreFilter(side: number): string`
- 用途: 返回当前过滤字符串。
- 参数:
  - `side: number`
    安装该导出总线的 multipart 方向。
- 返回值:
  - 当前过滤字符串。

#### `setExportOreFilter(side, filter)`

- 语法: `setExportOreFilter(side: number, filter: string): boolean`
- 用途: 修改过滤字符串。
- 参数:
  - `side: number`
    安装该导出总线的 multipart 方向。
  - `filter: string`
    新过滤字符串。
- 返回值:
  - `true` 表示成功。

#### `getVoidAllowed(side)`

- 语法: `getVoidAllowed(side: number): boolean`
- 用途: 返回导出到虚空罐时是否允许多余 essentia 被直接丢弃。
- 参数:
  - `side: number`
    安装该导出总线的 multipart 方向。
- 返回值:
  - `true` 表示允许虚空处理。
  - `false` 表示不允许。

#### `setVoidAllowed(side, allowed)`

- 语法: `setVoidAllowed(side: number, allowed: boolean): boolean`
- 用途: 修改虚空罐处理模式。
- 参数:
  - `side: number`
    安装该导出总线的 multipart 方向。
  - `allowed: boolean`
    目标虚空处理状态。
- 返回值:
  - 如果这次调用真的切换了状态，则返回 `true`。
  - 如果原本就已经是目标状态，则返回 `false`。

### essentia_importbus

`essentia_importbus` 用于 Thaumic Energistics 的 essentia 导入总线部件。

#### `getImportConfiguration(side[, slot])`

- 语法: `getImportConfiguration(side: number[, slot: number]): table`
- 用途: 返回某个导入槽中配置的 aspect。
- 参数:
  - `side: number`
    安装该导入总线的 multipart 方向。
  - `slot: number`
    可选槽位编号，默认是 `0`。
- 返回值:
  - essentia 描述表。

#### `setImportConfiguration(side[, slot][, detail])`

- 语法: `setImportConfiguration(side: number[, slot: number][, detail: table]): boolean`
- 用途: 设置或清空一个导入配置槽。
- 参数:
  - `side: number`
    安装该导入总线的 multipart 方向。
  - `slot: number`
    可选槽位编号，默认是 `0`。
  - `detail: table`
    可选 essentia 描述表。
- 返回值:
  - `true` 表示成功。
- 说明:
  - 省略 `detail` 时会清空该槽位。

#### `getImportSlotSize(side)`

- 语法: `getImportSlotSize(side: number): number`
- 用途: 返回当前有效的导入配置槽数量。
- 参数:
  - `side: number`
    安装该导入总线的 multipart 方向。
- 返回值:
  - 槽位数量。

#### `getImportOreFilter(side)`

- 语法: `getImportOreFilter(side: number): string`
- 用途: 返回当前导入过滤字符串。
- 参数:
  - `side: number`
    安装该导入总线的 multipart 方向。
- 返回值:
  - 当前过滤字符串。

#### `setImportOreFilter(side, filter)`

- 语法: `setImportOreFilter(side: number, filter: string): boolean`
- 用途: 修改导入过滤字符串。
- 参数:
  - `side: number`
    安装该导入总线的 multipart 方向。
  - `filter: string`
    新过滤字符串。
- 返回值:
  - `true` 表示成功。

### essentia_storagebus

`essentia_storagebus` 用于 Thaumic Energistics 的 essentia 存储总线部件。

#### `getStorageConfiguration(side[, slot])`

- 语法: `getStorageConfiguration(side: number[, slot: number]): table`
- 用途: 返回某个存储槽中配置的 aspect。
- 参数:
  - `side: number`
    安装该存储总线的 multipart 方向。
  - `slot: number`
    可选槽位编号，默认是 `0`。
- 返回值:
  - essentia 描述表。

#### `setStorageConfiguration(side[, slot][, detail])`

- 语法: `setStorageConfiguration(side: number[, slot: number][, detail: table]): boolean`
- 用途: 设置或清空一个存储配置槽。
- 参数:
  - `side: number`
    安装该存储总线的 multipart 方向。
  - `slot: number`
    可选槽位编号，默认是 `0`。
  - `detail: table`
    可选 essentia 描述表。
- 返回值:
  - `true` 表示成功。
- 说明:
  - 省略 `detail` 时会清空该槽位。

#### `getStorageOreFilter(side)`

- 语法: `getStorageOreFilter(side: number): string`
- 用途: 返回当前存储总线过滤字符串。
- 参数:
  - `side: number`
    安装该存储总线的 multipart 方向。
- 返回值:
  - 当前过滤字符串。

#### `setStorageOreFilter(side, filter)`

- 语法: `setStorageOreFilter(side: number, filter: string): boolean`
- 用途: 修改存储总线过滤字符串。
- 参数:
  - `side: number`
    安装该存储总线的 multipart 方向。
  - `filter: string`
    新过滤字符串。
- 返回值:
  - `true` 表示成功。

#### `getStorageSlotSize(side)`

- 语法: `getStorageSlotSize(side: number): number`
- 用途: 返回当前有效的存储配置槽数量。
- 参数:
  - `side: number`
    安装该存储总线的 multipart 方向。
- 返回值:
  - 在 AE 存储总线升级规则作用后的槽位数量。

### me_controller

安装 `thaumicenergistics` 后，`me_controller` 会额外获得一组 essentia 网络控制接口。

#### `getEssentiaInNetwork([filter])`

- 语法: `getEssentiaInNetwork([filter: string])`
- 语法: `getEssentiaInNetwork(detail: table)`
- 用途: 返回网络中当前可见的 essentia，或按精确描述符解析单个 essentia。
- 参数:
  - `filter: string`
    可选的 aspect id 过滤条件，会与 aspect 的未本地化名称匹配。
  - `detail: table`
    通过已注册 AE 栈解析器读取的精确 essentia 描述表。
- 返回值:
  - 字符串过滤形式: essentia 描述表数组。
  - 描述符表形式: 单个匹配的 essentia 描述表，或 `nil`。
- 说明:
  - 如果当前拿不到监视器，则返回 `nil`。

#### `setEssentiaEventSubscription(enabled)`

- 语法: `setEssentiaEventSubscription(enabled: boolean)`
- 用途: 启用或关闭 `network_essentia_changed` 事件订阅。
- 参数:
  - `enabled: boolean`
    目标订阅状态。
- 返回值:
  - 无返回值。

#### `isEssentiaEventSubscription()`

- 语法: `isEssentiaEventSubscription(): boolean`
- 用途: 返回当前是否已启用 essentia 变化事件订阅。
- 返回值:
  - `true` 表示已订阅。
  - `false` 表示未订阅。

### me_interface

在 Thaumic Energistics 网络中，方块形态的 `me_interface` 也会获得同样的 essentia 网络控制接口。

#### `getEssentiaInNetwork([filter])`

- 语法: `getEssentiaInNetwork([filter: string])`
- 语法: `getEssentiaInNetwork(detail: table)`
- 用途: 返回当前连接网络中可见的 essentia，或按精确描述符解析单个 essentia。
- 参数:
  - `filter: string`
    可选 aspect id 过滤条件。
  - `detail: table`
    通过已注册 AE 栈解析器读取的精确 essentia 描述表。
- 返回值:
  - 字符串过滤形式: essentia 描述表数组。
  - 描述符表形式: 单个匹配的 essentia 描述表，或 `nil`。

#### `setEssentiaEventSubscription(enabled)`

- 语法: `setEssentiaEventSubscription(enabled: boolean)`
- 用途: 启用或关闭 `network_essentia_changed` 事件订阅。
- 参数:
  - `enabled: boolean`
    目标订阅状态。
- 返回值:
  - 无返回值。

#### `isEssentiaEventSubscription()`

- 语法: `isEssentiaEventSubscription(): boolean`
- 用途: 返回当前是否已启用 essentia 变化事件订阅。
- 返回值:
  - `true` 表示已订阅。
  - `false` 表示未订阅。

#### 事件: `network_essentia_changed`

- 语法: `network_essentia_changed(...)`
- 用途: 当连接网络中的 essentia 发生变化时，向机器发送事件。
- 说明:
  - 这个事件负载由共享 AE 订阅系统管理，更适合做变更驱动自动化，而不是写死位置解析。

## 示例

```lua
local component = require("component")
local controller = component.me_controller

for _, aspect in ipairs(controller.getEssentiaInNetwork()) do
  print(aspect.label, aspect.amount)
end

controller.setEssentiaEventSubscription(true)
```

## 相关内容

- `appeng`
- `ae2fc`
- `thaumcraft`
