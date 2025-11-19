# Level Generation Flow

该文档概述 `Dungeon.newLevel()` 触发的楼层生成流程，并聚焦 `levels/Level.create()` 中的模板逻辑。不同楼层（如 `SewerLevel`, `PrisonLevel` 等）只需覆写 `build/decorate/createMobs/createItems`，即可在统一骨架上定制地图与内容。

## Mermaid 流程图

```mermaid
flowchart TD
    A["Dungeon.newLevel()"] --> B[depth++ stats 更新 清空 Actor]
    B --> C{当前深度}
    C -->|1-4| L1[SewerLevel]
    C -->|5| L2[SewerBossLevel]
    C -->|6-9| L3[PrisonLevel]
    C -->|10| L4[PrisonBossLevel]
    C -->|11-14| L5[CavesLevel]
    C -->|15| L6[CavesBossLevel]
    C -->|16-19| L7[CityLevel]
    C -->|20| L8[CityBossLevel]
    C -->|21| L9[LastShopLevel]
    C -->|22-24| L10[HallsLevel]
    C -->|25| L11[HallsBossLevel]
    C -->|26| L12[LastLevel]
    C -->|默认| L13[DeadEndLevel]

    subgraph Level.create 模板
        direction TB
        L1 & L2 & L3 & L4 & L5 & L6 & L7 & L8 & L9 & L10 & L11 & L12 & L13 --> D[初始化 map/visited/mapped 数组, 清空 mobs/heaps/blobs]
        D --> E{是否 Boss 层?}
        E -- 否 --> F[加入保底掉落: 食物、力量药水、升级卷轴、附魔卷轴]
        E -- 是 --> G
        F --> G["若深度>1 则掷骰确定 Feeling (Chasm/Water/Grass)"]
        G --> H[根据上一层弱地板状态计算 pitNeeded]
        H --> I[将地图填充为基本地形 感觉为 CHASM 时填坑]
        I --> J[同步 pitRoomNeeded/weakFloor 标志]
        J --> K{子类 build 成功?}
        K -- 否 --> I
        K -- 是 --> L[decorate 添加装饰/细节]
        L --> M[buildFlagMaps 计算 passable/losBlocking 等布尔图]
        M --> N[cleanWalls 清理墙面细节]
        N --> O[createMobs 放置怪物]
        O --> P[createItems 放置普通掉落]
    end

    P --> Q[Statistics.qualifiedForNoKilling 设置]
    Q --> R["Dungeon.switchLevel(level, entrance)"]
```

## 关键节点说明

- **保底掉落**：`Generator.random(FOOD)` 以及按需插入的力量药水、升级卷轴、附魔卷轴通过 `itemsToSpawn` 延迟到 `createItems()` 中实际落地。
- **Feeling 机制**：在非 Boss 层随机选择 CHASM/WATER/GRASS，驱动 `build()` 的初始填充与后续装饰氛围。
- **循环生成**：`build()` 返回 `false` 时会重新填充地图并再次尝试，直到满足拓扑约束（房间数量、入口出口连通等）。
- **Flag Maps**：`buildFlagMaps()` 根据最终地形计算 `passable`, `pit`, `water`, `losBlocking` 等静态数组，为寻路与视野提供 O(1) 查询。
- **内容注入**：`createMobs()` 与 `createItems()` 是子类最主要的自定义点，负责将 `itemsToSpawn` 和额外生成逻辑落地。

通过以上流程，任何新楼层类型都能在不修改全局逻辑的前提下插入到 `Dungeon.newLevel()` 的分支，并重用 `Level.create()` 的公共步骤。
