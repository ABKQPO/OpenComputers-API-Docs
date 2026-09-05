# ocglasses

## 概述

本页说明 OCGlasses 提供的 OpenComputers 集成。`glasses` 组件用于控制已连接的 OpenGlasses Terminal，并创建供佩戴已连接眼镜的玩家查看的 2D 或 3D widget。

## 可用性

- 依赖: `OCGlasses` 和 `OpenComputers`
- 可选依赖: 安装 `Computronics` 后才会启用聊天回调和聊天事件。
- 组件名: `glasses`
- 标签: `integration-required`

终端会根据 widget 数量消耗能量。默认配置的计算公式是 `(widget数量 / 10) * energyMultiplier`。

## 组件回调

### `getBindPlayers()`

- 语法: `getBindPlayers(): string...`
- 用途: 列出当前佩戴并连接到此终端的玩家名称。

### `getObjectCount()`

- 语法: `getObjectCount(): number`
- 用途: 返回当前创建的 widget 数量。

### `removeObject(id)`

- 语法: `removeObject(id: number): boolean`
- 用途: 删除指定 ID 的 widget。
- 返回值: 删除成功返回 `true`，否则返回 `false`。

### `removeAll()`

- 语法: `removeAll()`
- 用途: 删除全部 widget，并将下一个 widget ID 重置为 `0`。

### `sendChatAs(playerName, message)`

- 语法: `sendChatAs(playerName: string, message: string): boolean, string`
- 用途: 以已连接玩家的身份发送聊天消息。需要 Computronics 和已连接眼镜中的 ChatBox Upgrade。
- 返回值:
  - 成功返回 `true`。
  - 玩家未连接时返回 `false, "Failed to find the player."`。
  - 缺少升级时返回 `false, "Missing ChatBox Upgrade on glasses."`。
  - 斜杠命令不被允许时返回 `false, "Command forbidden."`。
- 说明: 默认允许的命令是 `/msg`、`/w` 和 `/tell`，列表可以配置。

### `sendMessageTo(playerName, message)`

- 语法: `sendMessageTo(playerName: string, message: string): boolean, string`
- 用途: 向已连接玩家发送私聊消息。需要 Computronics 和 ChatBox Upgrade。
- 返回值: 成功返回 `true`；失败时返回与 `sendChatAs` 相同的玩家或升级错误。

### `newUniqueKey()`

- 语法: `newUniqueKey(): number`
- 用途: 生成新的终端密钥，取消使用旧密钥的玩家订阅，并返回新密钥。

### `addRect()`

- 语法: `addRect(): Rect2D`
- 用途: 创建 2D 矩形 widget。

### `addDot()`

- 语法: `addDot(): Dot2D`
- 用途: 创建 2D 点 widget。

### `addItem()`

- 语法: `addItem(): ItemIcon`
- 用途: 创建 2D 物品图标 widget。

### `addCube3D()`

- 语法: `addCube3D(): Cube3D`
- 用途: 创建 3D 立方体 widget。

### `addFloatingText()`

- 语法: `addFloatingText(): Text3D`
- 用途: 创建 3D 浮动文本 widget。

### `addTriangle()`

- 语法: `addTriangle(): Triangle2D`
- 用途: 创建 2D 三角形 widget。

### `addDot3D()`

- 语法: `addDot3D(): Dot3D`
- 用途: 创建 3D 点 widget。

### `addTextLabel()`

- 语法: `addTextLabel(): Text2D`
- 用途: 创建 2D 文本标签 widget。

### `addLine3D()`

- 语法: `addLine3D(): Line3D`
- 用途: 创建 3D 线段 widget。

### `addTriangle3D()`

- 语法: `addTriangle3D(): Triangle3D`
- 用途: 创建 3D 三角形 widget。

### `addQuad3D()`

- 语法: `addQuad3D(): Quad3D`
- 用途: 创建 3D 四边形 widget。

### `addQuad()`

- 语法: `addQuad(): Quad2D`
- 用途: 创建 2D 四边形 widget。

## Widget 对象

创建回调返回的是由终端和 widget ID 支持的 Lua 表，而不是 widget 的副本。每个 widget 表都包含下面的通用方法；属性方法只会出现在兼容的 widget 类型上。

### `getID()`

- 语法: `getID(): number`
- 用途: 返回 widget 在当前终端中的 ID。

### `isVisible()` / `setVisible(enabled)`

- 语法: `isVisible(): boolean`；`setVisible(enabled: boolean)`
- 用途: 读取或修改 widget 是否渲染。

### `getAlpha()` / `setAlpha(alpha)`

- 语法: `getAlpha(): number`；`setAlpha(alpha: number)`
- 用途: 读取或设置透明度。适用于支持透明度的 widget。

### `getColor()` / `setColor(red, green, blue)`

- 语法: `getColor(): number, number, number`；`setColor(red: number, green: number, blue: number)`
- 用途: 读取或设置 RGB 颜色通道。适用于支持颜色的 widget。

### `getPosition()` / `setPosition(x, y)`

- 语法: `getPosition(): number, number`；`setPosition(x: number, y: number)`
- 用途: 读取或设置 2D 坐标。

### `getSize()` / `setSize(width, height)`

- 语法: `getSize(): number, number`；`setSize(width: number, height: number)`
- 用途: 读取或设置矩形尺寸。适用于 `Rect2D`。

### `get3DPos()` / `set3DPos(x, y, z)`

- 语法: `get3DPos(): number, number, number`；`set3DPos(x: number, y: number, z: number)`
- 用途: 读取或设置 3D 坐标。适用于支持 3D 位置的 widget。

### `getText()` / `setText(text)`

- 语法: `getText(): string`；`setText(text: string)`
- 用途: 读取或设置 widget 文本。适用于 `Text2D` 和 `Text3D`。

### `getScale()` / `setScale(scale)`

- 语法: `getScale(): number`；`setScale(scale: number)`
- 用途: 读取或设置缩放值。对于 `Dot2D` 和 `Line3D`，该值控制渲染尺寸或线宽。

### `isVisibleThroughObjects()` / `setVisibleThroughObjects(enabled)`

- 语法: `isVisibleThroughObjects(): boolean`；`setVisibleThroughObjects(enabled: boolean)`
- 用途: 读取或设置兼容 3D widget 的穿透渲染。

### `getVertexCount()`

- 语法: `getVertexCount(): number`
- 用途: 返回三角形、四边形或线段接受的顶点数量。

### `setVertex(index, x, y[, z])`

- 语法: `setVertex(index: number, x: number, y: number[, z: number])`
- 用途: 使用从 `1` 开始的顶点索引设置顶点。2D widget 使用 `x, y`；3D widget 使用 `x, y, z`。
- 错误: 索引超出 `1..getVertexCount()` 范围时抛出 `Vertex not exist!`。

### `getViewDistance()` / `setViewDistance(distance)`

- 语法: `getViewDistance(): number`；`setViewDistance(distance: number)`
- 用途: 读取或设置兼容 3D widget 的最大可视距离。

### `getLookingAt()` / `setLookingAt(value)`

- 语法: `getLookingAt(): number, number, number, boolean`；`setLookingAt(enabled: boolean) OR setLookingAt(x: number, y: number, z: number)`
- 用途: 读取注视目标，启用/禁用注视目标，或设置目标坐标。

### `getItem()` / `setItem(database, slot)`

- 语法: `getItem(): string`；`setItem(database: string, slot: number)`
- 用途: 读取显示物品的未本地化名称，或从 OpenComputers 数据库的 1-based 槽位加载物品。适用于 `ItemIcon`。
- 错误: 地址不存在或不是数据库时抛出 `Not a database`。

### `getRotation()` / `setRotation(rotation)`

- 语法: `getRotation(): number`；`setRotation(rotation: number)`
- 用途: 读取或设置可旋转 widget 的旋转角度。

## Widget 类型

所有 widget 表都包含 `getID`、`isVisible` 和 `setVisible`。

- `Rect2D`（`addRect`）: 2D 位置、尺寸、颜色和透明度。
- `Dot2D`（`addDot`）: 2D 位置、颜色、透明度和缩放。
- `ItemIcon`（`addItem`）: 物品、2D 位置、缩放和旋转。
- `Cube3D`（`addCube3D`）: 3D 位置、透明度、颜色、缩放、穿透渲染、可视距离和注视目标。
- `Text3D`（`addFloatingText`）: 3D 位置、文本、颜色、缩放、透明度、穿透渲染、可视距离和注视目标。
- `Triangle2D`（`addTriangle`）: 颜色、透明度和 3 个 2D 顶点。
- `Dot3D`（`addDot3D`）: 3D 位置、透明度、颜色、缩放、穿透渲染和可视距离。
- `Text2D`（`addTextLabel`）: `Dot2D` 的方法，以及文本和旋转。
- `Line3D`（`addLine3D`）: 颜色、透明度、缩放、穿透渲染和 2 个 3D 顶点。
- `Triangle3D`（`addTriangle3D`）: 颜色、透明度、穿透渲染和 3 个 3D 顶点。
- `Quad3D`（`addQuad3D`）: `Triangle3D` 的方法，但有 4 个顶点。
- `Quad2D`（`addQuad`）: `Triangle2D` 的方法，但有 4 个 2D 顶点。

## 事件

终端会通过 `computer.signal` 向可到达的电脑发送以下信号:

- `glasses_on(user, width, height)`: 已连接玩家戴上眼镜。
- `glasses_off(user)`: 已连接玩家摘下眼镜。
- `hud_click(user, x, y, button)`: 已连接玩家点击 HUD。
- `hud_drag(user, x, y, button)`: 已连接玩家拖动 HUD。
- `hud_keyboard(user, character, key)`: 已连接玩家触发键盘交互。
- `block_interact(user, x, y, z, side)`: 已连接玩家通过覆盖层与方块交互。
- `overlay_opened(user)`: 已连接玩家的覆盖层打开。
- `overlay_closed(user)`: 已连接玩家的覆盖层关闭。
- `glasses_resized(user, width, height)`: 已连接玩家的视口尺寸改变。
- `chat_message(username, message)`: Computronics 集成启用且已连接玩家安装 ChatBox Upgrade 时，该玩家发送聊天消息。

## 示例

```lua
local component = require("component")

if component.isAvailable("glasses") then
  local glasses = component.glasses
  local label = glasses.addTextLabel()
  label.setPosition(10, 10)
  label.setText("Online")
  label.setColor(0, 1, 0)
end
```

## 相关内容

- `opencomputers`
- `computronics`
