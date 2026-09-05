# ocglasses

## Summary

This page documents the OpenComputers integration provided by OCGlasses. The `glasses` component controls a linked OpenGlasses Terminal and creates 2D or 3D widgets for players wearing linked glasses.

## Availability

- Dependency: `OCGlasses` and `OpenComputers`
- Optional dependency: `Computronics` enables the chat callbacks and chat event.
- Component name: `glasses`
- Label: `integration-required`

The terminal consumes energy according to its widget count. Its default configured formula is `(number of widgets / 10) * energyMultiplier`.

## Component Callbacks

### `getBindPlayers()`

- Syntax: `getBindPlayers(): string...`
- Purpose: List players currently wearing glasses linked to this terminal.

### `getObjectCount()`

- Syntax: `getObjectCount(): number`
- Purpose: Return the number of widgets currently instantiated.

### `removeObject(id)`

- Syntax: `removeObject(id: number): boolean`
- Purpose: Remove the widget with the specified ID.
- Returns: `true` when removed, otherwise `false`.

### `removeAll()`

- Syntax: `removeAll()`
- Purpose: Remove all widgets and reset the next widget ID to `0`.

### `sendChatAs(playerName, message)`

- Syntax: `sendChatAs(playerName: string, message: string): boolean, string`
- Purpose: Send a chat message as a linked player. Requires Computronics and a ChatBox Upgrade in the linked glasses.
- Returns:
  - `true` on success.
  - `false, "Failed to find the player."` if the player is not linked.
  - `false, "Missing ChatBox Upgrade on glasses."` if the upgrade is absent.
  - `false, "Command forbidden."` if a slash command is not allowed.
- Notes: The default allowed commands are `/msg`, `/w`, and `/tell`; the list is configurable.

### `sendMessageTo(playerName, message)`

- Syntax: `sendMessageTo(playerName: string, message: string): boolean, string`
- Purpose: Send a private message to a linked player. Requires Computronics and a ChatBox Upgrade.
- Returns: `true` on success, or the same player and upgrade errors as `sendChatAs`.

### `newUniqueKey()`

- Syntax: `newUniqueKey(): number`
- Purpose: Generate a new terminal key, unsubscribe players using the old key, and return the new key.

### `addRect()`

- Syntax: `addRect(): Rect2D`
- Purpose: Create a 2D rectangle widget.

### `addDot()`

- Syntax: `addDot(): Dot2D`
- Purpose: Create a 2D dot widget.

### `addItem()`

- Syntax: `addItem(): ItemIcon`
- Purpose: Create a 2D item-icon widget.

### `addCube3D()`

- Syntax: `addCube3D(): Cube3D`
- Purpose: Create a 3D cube widget.

### `addFloatingText()`

- Syntax: `addFloatingText(): Text3D`
- Purpose: Create a floating 3D text widget.

### `addTriangle()`

- Syntax: `addTriangle(): Triangle2D`
- Purpose: Create a 2D triangle widget.

### `addDot3D()`

- Syntax: `addDot3D(): Dot3D`
- Purpose: Create a 3D dot widget.

### `addTextLabel()`

- Syntax: `addTextLabel(): Text2D`
- Purpose: Create a 2D text-label widget.

### `addLine3D()`

- Syntax: `addLine3D(): Line3D`
- Purpose: Create a 3D line widget.

### `addTriangle3D()`

- Syntax: `addTriangle3D(): Triangle3D`
- Purpose: Create a 3D triangle widget.

### `addQuad3D()`

- Syntax: `addQuad3D(): Quad3D`
- Purpose: Create a 3D quadrilateral widget.

### `addQuad()`

- Syntax: `addQuad(): Quad2D`
- Purpose: Create a 2D quadrilateral widget.

## Widget Objects

Creation callbacks return a Lua table backed by a terminal and widget ID, not a copy of the widget. Every widget table has the common methods below. Attribute-specific methods are present only on compatible widget types.

### `getID()`

- Syntax: `getID(): number`
- Purpose: Return the terminal-local widget ID.

### `isVisible()` / `setVisible(enabled)`

- Syntax: `isVisible(): boolean`; `setVisible(enabled: boolean)`
- Purpose: Read or change whether the widget is rendered.

### `getAlpha()` / `setAlpha(alpha)`

- Syntax: `getAlpha(): number`; `setAlpha(alpha: number)`
- Purpose: Read or set alpha. Available on alpha-capable widgets.

### `getColor()` / `setColor(red, green, blue)`

- Syntax: `getColor(): number, number, number`; `setColor(red: number, green: number, blue: number)`
- Purpose: Read or set RGB channels. Available on color-capable widgets.

### `getPosition()` / `setPosition(x, y)`

- Syntax: `getPosition(): number, number`; `setPosition(x: number, y: number)`
- Purpose: Read or set a 2D position.

### `getSize()` / `setSize(width, height)`

- Syntax: `getSize(): number, number`; `setSize(width: number, height: number)`
- Purpose: Read or set rectangle dimensions. Available on `Rect2D`.

### `get3DPos()` / `set3DPos(x, y, z)`

- Syntax: `get3DPos(): number, number, number`; `set3DPos(x: number, y: number, z: number)`
- Purpose: Read or set a 3D position. Available on 3D-positionable widgets.

### `getText()` / `setText(text)`

- Syntax: `getText(): string`; `setText(text: string)`
- Purpose: Read or set widget text. Available on `Text2D` and `Text3D`.

### `getScale()` / `setScale(scale)`

- Syntax: `getScale(): number`; `setScale(scale: number)`
- Purpose: Read or set scale. On `Dot2D` and `Line3D`, scale controls rendered size or line width.

### `isVisibleThroughObjects()` / `setVisibleThroughObjects(enabled)`

- Syntax: `isVisibleThroughObjects(): boolean`; `setVisibleThroughObjects(enabled: boolean)`
- Purpose: Read or set through-object rendering for compatible 3D widgets.

### `getVertexCount()`

- Syntax: `getVertexCount(): number`
- Purpose: Return the number of vertices accepted by a triangle, quad, or line.

### `setVertex(index, x, y[, z])`

- Syntax: `setVertex(index: number, x: number, y: number[, z: number])`
- Purpose: Set a 1-based vertex. 2D widgets use `x, y`; 3D widgets use `x, y, z`.
- Errors: An index outside `1..getVertexCount()` raises `Vertex not exist!`.

### `getViewDistance()` / `setViewDistance(distance)`

- Syntax: `getViewDistance(): number`; `setViewDistance(distance: number)`
- Purpose: Read or set maximum view distance on compatible 3D widgets.

### `getLookingAt()` / `setLookingAt(value)`

- Syntax: `getLookingAt(): number, number, number, boolean`; `setLookingAt(enabled: boolean) OR setLookingAt(x: number, y: number, z: number)`
- Purpose: Read or enable/disable the look-at target, or set its coordinates.

### `getItem()` / `setItem(database, slot)`

- Syntax: `getItem(): string`; `setItem(database: string, slot: number)`
- Purpose: Read the displayed item's unlocalized name, or load an item from a one-based OpenComputers database slot. Available on `ItemIcon`.
- Errors: A missing/non-database address raises `Not a database`.

### `getRotation()` / `setRotation(rotation)`

- Syntax: `getRotation(): number`; `setRotation(rotation: number)`
- Purpose: Read or set rotation on rotatable widgets.

## Widget Types

All widget tables include `getID`, `isVisible`, and `setVisible`.

- `Rect2D` (`addRect`): 2D position, size, color, and alpha.
- `Dot2D` (`addDot`): 2D position, color, alpha, and scale.
- `ItemIcon` (`addItem`): item, 2D position, scale, and rotation.
- `Cube3D` (`addCube3D`): 3D position, alpha, color, scale, through-object visibility, view distance, and look-at target.
- `Text3D` (`addFloatingText`): 3D position, text, color, scale, alpha, through-object visibility, view distance, and look-at target.
- `Triangle2D` (`addTriangle`): color, alpha, and three 2D vertices.
- `Dot3D` (`addDot3D`): 3D position, alpha, color, scale, through-object visibility, and view distance.
- `Text2D` (`addTextLabel`): `Dot2D` methods plus text and rotation.
- `Line3D` (`addLine3D`): color, alpha, scale, through-object visibility, and two 3D vertices.
- `Triangle3D` (`addTriangle3D`): color, alpha, through-object visibility, and three 3D vertices.
- `Quad3D` (`addQuad3D`): `Triangle3D` methods with four vertices.
- `Quad2D` (`addQuad`): `Triangle2D` methods with four 2D vertices.

## Events

The terminal sends these signals to reachable computers through `computer.signal`:

- `glasses_on(user, width, height)`: A linked player put on the glasses.
- `glasses_off(user)`: A linked player removed the glasses.
- `hud_click(user, x, y, button)`: A linked player clicked the HUD.
- `hud_drag(user, x, y, button)`: A linked player dragged on the HUD.
- `hud_keyboard(user, character, key)`: A linked player sent a keyboard interaction.
- `block_interact(user, x, y, z, side)`: A linked player interacted with a block through the overlay.
- `overlay_opened(user)`: A linked player's overlay opened.
- `overlay_closed(user)`: A linked player's overlay closed.
- `glasses_resized(user, width, height)`: A linked player's viewport was resized.
- `chat_message(username, message)`: A linked player with ChatBox Upgrade sent chat while Computronics integration is active.

## Example

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

## Related

- `opencomputers`
- `computronics`
