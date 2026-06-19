# logisticspipes

## 概述

本页说明 LogisticsPipes 提供给 OpenComputers 的集成接口。它和一般“每个方块只写几条 `@Callback`”的集成不同：LogisticsPipes 只注册了两个 OpenComputers 组件入口，大部分可调用 API 都来自它自己的 `@CCType` / `@CCCommand` 元数据，再通过包装层动态暴露给 OpenComputers。

## 可用性

- 依赖: `logisticspipes`
- 标签: `integration-required`
- 本次刷新审计的源码仓库:
  - `LogisticsPipes`
  - `OpenComputers`

## 组件名

- `logisticspipe`
- `logisticssolidblock`

## 集成模型

真正的 OpenComputers 桥接代码位于 `logisticspipes.proxy.opencomputers.OpenComputersProxy`。

从源码确认到的关键行为有：

- 路由管道会创建名为 `logisticspipe` 的 OC 组件节点
- LP 实体方块会创建名为 `logisticssolidblock` 的 OC 组件节点
- 普通 OpenComputers 信号通过 `computer.signal` 发送
- LP 网络内部的计算机消息会以事件名 `LP_MESSAGE` 转发
- 大多数可调用方法不是手写 `@Callback`，而是来自 LP 包装层生成的包装对象

这意味着组件本身更像是进入 LP 包装对象体系的入口。

## 信号与寻址模型

### 组件地址

每个 LP 的 OC 节点都有标准的 OpenComputers 节点地址。源码在 LP 网络转发消息时会使用这个地址。

### `LP_MESSAGE`

当一条 LP 管道向另一台计算机端点发送网络消息时，OpenComputers 侧会收到：

```lua
event.pull("LP_MESSAGE")
```

载荷结构为：

- 事件名: `LP_MESSAGE`
- 来源 id:
  - 如果发送方是 ComputerCraft，则是数值计算机 id
  - 如果发送方是 OpenComputers，则是字符串形式的组件地址
- 消息本体:
  - 发送时提供的 Lua 兼容载荷

### 广播

路由管道源码里还定义了 LP 范围内的广播路径。当前 OC 桥接对 `LP_MESSAGE` 的转发是明确的；广播事件名与行为应视为相关能力，但在生产自动化里依赖前最好再做运行时确认。

## 运行时自检辅助

LP 包装值通过通用包装基类暴露了两个很重要的自检接口。

### `help([page])`

- 语法: `value.help([page: number]): string`
- 用途: 返回当前包装值的人类可读命令列表。
- 说明:
  - 源码里标记为 direct call 的方法会显示 `D`
  - 标记为 queued call 的方法会显示 `Q`
  - 命令太多时会分页输出

### `helpCommand(index)`

- 语法: `value.helpCommand(index: number): string`
- 用途: 返回某条包装命令的名称、参数类型、返回类型和描述。
- 说明:
  - 命令编号来自生成出来的包装元数据，不是 Lua 层的方法顺序

对于较大的 LP 包装对象，这两个接口是最可靠的运行时自检手段。

## `logisticspipe`

`logisticspipe` 对应 LogisticsPipes 的路由管道方块实体。

具体可调用接口取决于被包装的管道类型。源码元数据至少确认了这些包装家族：

- `LogisticsPipes:Normal`
- `LogisticsPipes:Request`
- `LogisticsPipes:Crafting`
- `LogisticsChassiePipe`

### 共享路由管道方法

基础路由管道类型会暴露诸如：

- `getRouterId()`
  返回运行期稳定的数值路由器 id。
- `getRouterUUID()`
  返回生命周期稳定的路由器 UUID 字符串。
- `getRouterUUID(id)`
  根据另一个可达路由器的数值 id 解析其 UUID。
- `setTurtleConnect(flag)`
  修改当前管道上的 turtle 连接标志。
- `getTurtleConnect()`
  读取 turtle 连接标志。
- `canAccess()`
  检查当前计算机是否有权限操作这条管道。
- `sendMessage(computerId, message)`
  通过 LP 网络发送一条计算机消息，目标端会收到 `LP_MESSAGE` 事件。
- `sendBroadcast(message)`
  在 LP 网络里发送广播消息。
- `getPipeForUUID(uuid)`
  在安装远程控制升级的前提下，返回另一个可达管道的访问对象。
- `getLP()`
  返回全局 LP 辅助对象。
- `hasLogisticsModule()`
  返回这条管道是否有内部模块。

### `LogisticsPipes:Request`

请求管道还会增加以下网络查询与请求方法：

- `makeRequest(stack)`
- `makeRequest(stack, forceCrafting)`
- `makeRequest(item, amount)`
- `makeRequest(item, amount, forceCrafting)`
- `getAvailableItems()`
- `getCraftableItems()`
- `getItemAmount(item)`

这些请求方法在源码里是 queued 调用，适合用于可能需要调度或等待的物流请求。

### `LogisticsPipes:Crafting`

合成管道会增加：

- `reimport()`
  从相邻机器或工作台重新导入合成配方。

更完整的合成状态仍然要通过包装后的路由管道对象和模块对象继续深入查看。

### `LogisticsChassiePipe`

底盘管道会增加：

- `getModuleInSlot(slot)`
  返回指定一基槽位中的包装模块对象。
- `getChassieSize()`
  返回底盘模块槽数量。

返回的模块对象可以继续通过 `help()` 和 `helpCommand()` 进行自检。

## `logisticssolidblock`

`logisticssolidblock` 对应接入了 OC 桥接的 LogisticsPipes 实体方块。

从审计到的源码至少可以确认：

- `getRotation()`
  返回该方块的 LP 旋转值。

和路由管道一样，如果具体方块类型带有 LP 计算机元数据，它还会通过包装层暴露更多能力。

## 包装模块家族

源码元数据确认了这些模块类包装对象：

- `LogisticsModule`
- `Provider Module`
- `ItemSink Module`
- `Advanced Extractor Module`
- `Terminus Module`
- `EnchantmentSink Module MK2`

这类模块通常会提供：

- 过滤库存访问
- 默认路由之类的布尔开关
- 模块专有的请求或提供控制

由于具体可见接口依赖已安装模块，最权威的方式仍然是对返回对象调用 `help()`。

## 包装值对象

源码元数据确认了这些可复用包装值家族：

- `LP Global Access`
- `ItemIdentifier`
- `ItemIdentifierStack`
- `ItemIdentifierBuilder`
- `FluidIdentifier`
- `FilterInventory`
- `CCItemSinkRequest`
- `Resource`
- `ItemResource`
- `DictResource`
- `FluidResource`
- `Pair`
- `Triplet`
- `Quartet`

其中比较重要的例子有：

- `LP Global Access`
  - `identify(object)`
  - `getItemIdentifierBuilder()`
- `ItemIdentifier`
  - 物品 id、名称、模组、标签、damage 与转换辅助
- `ItemIdentifierStack`
  - 物品标识加堆叠数量辅助
- `FilterInventory`
  - 尺寸、读槽位、写槽位辅助
- `FluidIdentifier`
  - 流体 id、名称、标签、本地化名称与转换辅助

这些包装值就是 LP 在 OC 桥接里传递复杂数据的主要方式。

## 示例

```lua
local component = require("component")
local event = require("event")

local pipe = component.logisticspipe

print("Router id:", pipe.getRouterId())
print("Router uuid:", pipe.getRouterUUID())
print("Can access:", pipe.canAccess())

local lp = pipe.getLP()
print(lp.identify(lp))

local helpText = lp.help()
print(helpText)

pipe.sendBroadcast("hello from oc")

local _, sourceId, payload = event.pull(0.5, "LP_MESSAGE")
if sourceId then
  print("LP message from", sourceId, payload)
end
```

## 使用说明

- 包装层在内部匹配方法名时大小写不敏感，但文档里仍应使用源码里的规范名称。
- 某些 LP 方法在源码里属于 direct-only。如果某个方法在当前上下文拒绝执行，先查看包装对象的 `help()` 输出以及当前管道权限状态。
- 远程管道访问依赖 LP 升级和网络可达性。某条管道上存在的方法，不代表你一定能无条件操作网络里的所有其他 LP 节点。
- LP 对象的持久化能力有限。有些包装值可以保存恢复，有些本质上只是围绕当前世界状态的实时引用。

## 相关内容

- `storagedrawers`
- `betterstorage`
- `appeng`
