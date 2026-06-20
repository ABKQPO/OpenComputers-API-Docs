# OpenComputers Lua API 文档

## 文档目录

- [核心运行时](./core-runtime/README.md)
- [组件](./components/README.md)
- [库](./libraries/README.md)
- [命令](./commands/README.md)
- [系统](./systems/README.md)
- [示例](./examples/README.md)
- [集成](./integrations/README.md)
- [附录](./appendix/README.md)
## 组件 API 索引

下面给出文档树中常用组件、扩展卡、升级与集成页面的快速入口，便于按组件名或模组名快速跳转。
如果你还需要查阅 OpenComputers 本体的基础概念与上手说明，也可以参考[开放式电脑官方文档](https://ocdoc.cil.li/start:zh)。
### 电脑组件
- 所有设备：[computer](./components/opencomputers-machine.md)
- EEPROM：[eeprom](./components/opencomputers-eeprom.md)
- 硬盘与软盘：[filesystem](./components/opencomputers-file-system.md)
- 机器人：[robot](./components/robot)、[world-control](./components/opencomputers-world-control.md)
- 无人机：[drone](./components/drone)、[world-control](./components/opencomputers-inventory-world-control.md)
- 平板电脑：[tablet](./components/tablet)
### 方块
连接在同一个电脑网络的方块所提供的组件
- 屏幕：[screen](./components/screen)、[玩家交互](./components/opencomputers-screen.md)
- 转运器：[inventory-transfer](./components/opencomputers-inventory-transfer.md)、[fluid-container-transfer](./components/opencomputers-fluid-container-transfer.md)、[inventory-analytics](./components/opencomputers-world-inventory-analytics.md)、[tank-analytics](./components/opencomputers-world-tank-analytics.md)、~~[fluid-container-analytics](./components/opencomputers-world-fluid-container-analytics.md)~~
- 红石I/O端口：[redstone-vanilla](./components/opencomputers-redstone-vanilla.md)、[redstone-signaller](./components/opencomputers-redstone-signaller.md)、[redstone-bundled](./components/opencomputers-redstone-bundled.md)、[redstone-wireless](./components/opencomputers-redstone-wireless.md)
- 地质分析仪：[geolyzer](./components/geolyzer.md)
- 运动传感器：[motion_sensor](./components/motion-sensor.md)
- 软盘驱动器：[disk_drive](./components/disk-drive.md)
- 网络分配器：[net_splitter](./components/opencomputers-net-splitter.md)
- 中继器：[relay](./components/relay.md)
- 3D打印机：[printer3d](./components/printer3d.md)
- 全息投影仪：[hologram](./components/hologram.md)
- 路径点：[waypoint](./components/waypoint.md)
- 电子装配器：[assembler](./components/assembler.md)
- 机箱：[computer](./components/opencomputers-machine.md)
- 机器人：[robot](./components/opencomputers-robot-proxy.md)
- 微控制器：[microcontroller](./components/microcontroller.md)

Computronics:

- 机架式电容：[rack_capacitor](./components/computronics-driver-board-capacitor.md)
- 指示灯面板：[light_board](./components/computronics-driver-board-light.md)
- 服务器自毁装置：[server_destruct](./components/server-destruct.md)
- 开关面板：[switch_board](./components/computronics-driver-board-switch.md)
- 摄像头：[camera](./components/computronics-tile-camera.md)
- 聊天盒 & 创造模式聊天盒：[chat_box](./components/computronics-tile-chat-box.md)
- 加密机：[cipher](./components/computronics-tile-cipher-block.md)
- 高级加密机：[advanced_cipher](./components/)、[回调对象](./components/computronics-rsavalue.md)
- 变色灯：[colorful_lamp](./components/computronics-tile-colorful-lamp.md)
- 铁音符盒：[iron_noteblock](./components/computronics-tile-iron-note.md)
- 雷达：[radar](./components/computronics-tile-radar.md)
- 语音盒：[speech_box](./components/computronics-tile-speech-box.md)
- 磁带驱动器：[tape_drive](./components/tape-rewind.md)

OpenPrinter:

- 开放式打印机：[openprinter](./components/openprinter.md)

OpenSecurity:

- 警铃：[os_alarm](./components/os-alarm.md)
- 生物计量读写器：[os_biometric](./components/os-biometric.md)
- 写卡器：[os_cardwriter](./components/os-cardwriter.md)
- 数据块：[os_datablock](./components/os-datablock.md)
- 门控器：[os_door](./components/os-door.md)
- 能源炮塔：[os_energyturret](./components/os-energyturret.md)
- 实体探测器：[os_entdetector](./components/os-entdetector.md)
- 按键：[os_keypad](./components/os-keypad.md)
- 磁卡片输入机：[os_magreader](./components/os-magreader.md)
- RFID读写器：[os_rfidreader](./components/os-rfidreader.md)
- 可切换集线器：[os_switchinghub](./components/os-switchinghub.md)
### 扩展卡
- 显卡：[gpu](./components/gpu.md)
- 网卡 & 无线网卡 & 连接卡 & 欺骗攻击卡 & 网络安全卡：[modem](./components/opencomputers-network-card.md)、[无线](./components/opencomputers-wireless-network-card.md)、[tunnel](./components/opencomputers-linked-card.md)、[欺骗攻击卡](./components/computronics-driver-card-spoof.md)、[wake-message](./components/opencomputers-wake-message-aware.md)、[网络安全卡](./components/opensecurity-secure-network-card-driver.md)
- T1红石卡：[redstone-vanilla](./components/opencomputers-redstone-vanilla.md)、[redstone-signaller](./components/opencomputers-redstone-signaller.md)
- T2红石卡：[redstone-vanilla](./components/opencomputers-redstone-vanilla.md)、[redstone-signaller](./components/opencomputers-redstone-signaller.md)、[redstone-bundled](./components/opencomputers-redstone-bundled.md)、[redstone-wireless](./components/opencomputers-redstone-wireless.md)
- 调试卡：[debug](./components/debug.md)
- 因特网卡：[internet](./components/opencomputers-internet-card.md)
- 数据卡：[data](./components/opencomputers-data-card.md)
- TPS卡：[tps_card](./components/opencomputers-tps-card.md)
- 世界传感器卡：[world_sensor](./integrations/gc.md)
- 蜂鸣卡：[beep](./components/beep.md)
- 噪音卡：[noise](./components/noise.md)
- 粒子效果卡：[particle](./components/particle.md)
- 自毁卡：[self_destruct](./components/self-destruct.md)
- 声卡：[sound](./components/sound.md)
### 适配器
- 原版内容：[vanilla](./integrations/vanilla.md) (命令方块/酿造台/熔炉/信标/红石比较器/音符盒/唱片机/刷怪笼)、[告示牌](./components/opencomputers-upgrade-sign-in-adapter.md)
- 应用能源2：[本体](./integrations/appeng.md)、[流体合成](./integrations/ae2fc.md)、[神秘能源](./integrations/thaumicenergistics.md)
- 无尽贪婪附属：[AvaritiaAddons](./integrations/avaritiaaddons.md) (梦魇工作台)
- 血魔法：[Blood Magic](./integrations/bloodmagic.md) (血之祭坛/主仪式石)
- BC：[BuildCraft](./integrations/buildcraft.md)
- 龙之研究：[Draconic Evolution](./integrations/draconicevolution.md)
- 抽屉：[drawer](./integrations/storagedrawers.md)
- 末影接口：[Ender IO](./integrations/enderio.md)
- 末影存储：[EnderStorage](./integrations/enderstorage.md) (高级末影箱子/高级末影储罐)
- 林业：[Forestry](./integrations/forestry.md) (分析台/简易蜂房/蜂箱/蜂箱组/魔法蜂箱)
- GT机器：[gt_machine](./integrations/gregtech.md)
- IC2：[IC2](./integrations/ic2.md)
- 物流管道：[Logistics Pipes](./integrations/logisticspipes.md)
- RC：[Railcraft](./integrations/railcraft.md)
- SGCraft：[stargate](./integrations/sgcraft.md) (星门OC接口)
- Storage Drawers：[storagedrawers](./integrations/storagedrawers.md) (历史集成审计说明)
- 神秘时代：[Thaumcraft](./integrations/thaumcraft.md)
### 升级
可加装在机器人、无人机或平板电脑的组件
- 物品栏升级：[inventory-control](./components/opencomputers-inventory-control.md)、[inventory-world-control](./components/opencomputers-inventory-world-control.md)
- 储罐升级：[tank-control](./components/opencomputers-tank-control.md)、[tank-world-control](./components/opencomputers-tank-world-control.md)
- 物品栏交互升级：[inventory_controller](./components/opencomputers-common.md#机器人上的-inventory_controller-接口)、[item-inventory-control](./components/opencomputers-item-inventory-control.md)、[inventory-world-control](./components/opencomputers-inventory-world-control-mk2.md)、[inventory-analytics](./components/opencomputers-inventory-analytics.md)、[world-inventory-analytics](./components/opencomputers-world-inventory-analytics.md)
- 储罐交互升级：[tank-inventory-control](./components/opencomputers-tank-inventory-control.md)、[tank-analytics](./components/opencomputers-world-tank-analytics.md)、~~[fluid-container-analytics](./components/opencomputers-world-fluid-container-analytics.md)~~
- 数据库升级：[database](./components/database.md)
- 合成升级：[crafting](./components/crafting.md)
- ME升级：[upgrade_me](./integrations/appeng.md)、[流体](./integrations/ae2fc.md)
- 发电机升级：[generator](./components/opencomputers-upgrade-generator.md)
- 导航升级：[navigation](./components/opencomputers-upgrade-navigation.md)
- 区块加载升级：[chunkloader](./components/chunkloader.md)
- 养蜂员升级：[bee_keeper](./integrations/forestry.md)
- 配置器升级：[configurator](./integrations/enderio.md)
- 牵引光束升级：[tractor_beam](./components/tractor-beam.md)
- 活塞升级：[piston](./components/piston.md)
- 告示牌读写升级：[sign](./components/opencomputers-upgrade-sign-in-rotatable.md)
- 经验升级：[experience](./components/experience.md)
- 拴绳升级：[leash](./components/leash.md)
- 交易升级：[trading](./components/trading.md)、[回调对象](./components/opencomputers-trade.md)
- 变色升级：[colors](./components/computronics-robot-upgrade-colorful.md)
- 摄像头升级：[camera](./components/computronics-robot-upgrade-camera.md)
- 聊天升级：[chat](./components/computronics-robot-upgrade-chat-box.md)
- 雷达升级：[radar](./components/computronics-robot-upgrade-radar.md)
- 语音升级：[speech](./components/speech.md)
