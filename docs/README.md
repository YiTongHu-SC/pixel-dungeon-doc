# docs

项目文档，理解开发框架。

## 1. 项目概览

- **类型**：基于 Java/Android 的 Rogue-like 探索游戏，使用 Noosa 轻量渲染框架。
- **核心目标**：通过 `com.watabou.pixeldungeon` 包实现从 UI、关卡生成到战斗逻辑的完整客户端。
- **模块组成**：场景 (`scenes`)、地城与关卡 (`levels`)、实体系统 (`actors`)、物品系统 (`items`)、UI 组件 (`ui`/`windows`)、资源封装 (`Assets`, `res/`)。

## 2. 启动与场景切换

- **入口**：`PixelDungeon` 继承 Noosa 的 `Game`，在构造函数中指定首个 `TitleScene` 并注册保存兼容所需的别名。
- **生命周期**：`onCreate` 中按照偏好设置屏幕方向、沉浸式 UI，并加载 `Assets` 中定义的音效。`Preferences` 类保存所有玩家设置或运行时 flag。
- **场景体系**：所有画面继承自 `PixelScene`，常见有 `TitleScene`（主菜单）、`StartScene`（角色选择）、`GameScene`（游戏主循环）等，通过 `Game.switchScene` 在 `PixelDungeon` 中切换。`GameScene` 负责驱动关卡渲染、UI 层叠以及事件处理。

## 3. 地城与关卡结构

- **Dungeon 管理器**：`Dungeon` 持有全局运行态（当前层数、金币、可见范围、挑战模式、掉落池等），并负责初始化英雄、生成新关卡、保存/读取进度。
- **Level 抽象**：`levels/Level` 定义了 32x32 的网格地图及 `build/decorate/createMobs/createItems` 四个模板方法，不同楼层（例如 `SewerLevel`, `PrisonLevel`, `CityLevel`）通过子类实现生成策略。Boss、商店、终点等特殊楼层同样通过子类建模。
- **地形与特性**：`Terrain`、`rooms/`, `features/`, `painters/`, `traps/` 等子目录实现地形常量、房间模板、装饰与陷阱放置。`Level` 还维护 `passable`, `losBlocking`, `pit` 等静态布尔数组用于寻路与视野计算。

## 4. 实体与行动系统

- **Actor 调度器 (`actors/Actor`)**：维护全局时间轮（`time` 字段）与 `process()` 方法，按最小冷却顺序依次调用实体的 `act()`。`Actor.add` 将角色、 Buff、环境效果统一纳入调度。
- **Char 层 (`actors/Char`)**：扩展 `Actor`，提供生命值、速度、命中/闪避、 Buff 集合管理与基础战斗行为。玩家角色 (`hero/`) 和怪物 (`mobs/`) 均继承自此。
- **战斗与感知**：`Dungeon.level.updateFieldOfView` 使用 `ShadowCaster` 计算视野；`Ballistica` 处理射线与飞行物路径；`Buff`/`Blob` 用于持续状态与场景效果。

## 5. 物品与资源

- **Item 体系**：`items/Item` 为抽象基类，派生出装备、武器、卷轴、药水、植物等子模块。`Generator` 依据类别和楼层深度决定掉落权重。
- **背包与快捷栏**：`ui/QuickSlot`, `items/bags/` 定义装备/物品容器与 UI 绑定；`Heap` 表示地面堆叠掉落。
- **资源定义**：`Assets` 中集中声明音频/纹理路径，对应文件位于 `res/` 和 `assets/`。

## 6. UI 与交互

- **界面层**：`ui/` 包含 HUD、日志、按钮、进度条；`windows/` 提供模态对话框（如物品详情、商店、复活对话）。
- **场景效果**：`effects/` 与 `sprites/` 存放视觉特效、角色动画，配合 `GameScene` 的渲染顺序执行。

## 7. 数据持久化与配置

- **Bundle 序列化**：依赖 `com.watabou.utils.Bundle`/`Bundlable` 保存对象状态；`Dungeon.saveGame/restoreGame` 负责游戏与楼层(`level`, `depthX.dat`)文件写读。
- **统计与进度**：`Statistics`, `Rankings`, `GamesInProgress`, `Badges` 追踪玩家表现；`Journal` 记录剧情/任务提示。
- **偏好设置**：`Preferences` 映射到 Android `SharedPreferences`，对场景切换和 UI 配置（音乐、震动、亮度）即刻生效。

## 8. 推荐阅读顺序

1. `PixelDungeon.java` —— 了解生命周期与场景注册。
2. `Dungeon.java`、`levels/Level.java` —— 理解关卡生成与世界状态。
3. `actors/Actor.java`, `actors/Char.java`, `actors/hero/Hero.java` —— 掌握行动与战斗调度。
4. `items/Item.java` 与 `Generator.java` —— 熟悉物品架构。
5. `scenes/GameScene.java`, `ui/` —— 查看 UI 组合方式。

## 9. 详细文档

- **[关卡生成流程](level-generation.md)** - Mermaid 流程图展示地城生成的完整过程
- **[UML 架构草图](uml-architecture.md)** - 核心类图与设计模式分析
- **[地牢生成算法](dungeon-generation-algorithm.md)** - BSP 分割与连通性算法详解

## 10. 后续扩展建议

- 按模块建立更细的笔记（例如 Buff/Blob 生命周期、战斗计算公式）。
- 补充具体子系统的序列图，展示方法调用链。
- 若需调试可视化，可先在 `GameScene` 中添加日志，再深入具体子系统。
