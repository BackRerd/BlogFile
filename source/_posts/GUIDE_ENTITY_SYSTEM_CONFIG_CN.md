---
title: 生物与世界数据包配置(cbs)
tags:
  - pfc
  - ca
  - mc
  - mod
categories:
  - pfc
  - ca
date: 2025-11-25 06:52:01
---
# 👾 ClassBioArsenal 实体系统配置

> 怪物等级、词缀与强度调整指南

---

## 📁 目录结构

**路径**：`data/classbioarsenal/entity_system/`

### 📄 主要配置文件 / 目录

- 📊 `level_config.json` - 按维度+距离计算怪物等级
- 👾 `entity_settings.json` - 生物强度倍率与词缀限制
- ✨ `affix_config.json` - 怪物词缀池与全局规则
- 📁 `drops/` 目录 - 每个生物一个 JSON，配置其额外掉落与指令

> 💡 由 `EntitySystemManager` 统一加载，控制整个实体系统

---

## 📊 等级配置：`level_config.json`

示例：

```json
{
  "dimensions": {
    "minecraft:overworld": {
      "base_level": 1,
      "level_multiplier": 1.0,
      "distance_threshold": 64
    },
    "minecraft:the_nether": {
      "base_level": 10,
      "level_multiplier": 2.0,
      "distance_threshold": 32
    },
    "minecraft:the_end": {
      "base_level": 30,
      "level_multiplier": 3.0,
      "distance_threshold": 32
    }
  }
}
```

### 🌍 维度规则

- **`dimensions`**：以维度 ID 为键的规则集
  - 🌍 主世界：`"minecraft:overworld"`
  - 🔥 下界：`"minecraft:the_nether"`
  - 🌌 末地：`"minecraft:the_end"`
  - ➕ 自定义维度：`"modid:my_dimension"`

### ⚙️ 维度参数

- **`base_level`**：维度基础等级
  - 🌍 主世界 `1`：出生点附近 1 级怪物
  - 🔥 下界 `10`：默认 10 级起步，危险度更高

- **`level_multiplier`**：距离相关的等级增长倍率
  - 📏 `额外等级 ≈ (距离 / distance_threshold) × level_multiplier`

- **`distance_threshold`**：距离阈值（每多少格算 1 个等级单位）
  - 📍 距离：怪物到世界出生点的欧氏距离

> 大致上：
>
> - 越远离出生点，等级越高。
> - `base_level` 决定起点，`distance_threshold` 和 `level_multiplier` 决定增长速度。

### 💡 调参建议

- 🌿 **温和主世界**：`base_level` 1~3，`level_multiplier` 0.5~1.0
- ⚡ **危险维度**：提高 `base_level`（30+），调高 `level_multiplier`
- 📈 **陡峭曲线**：降低 `distance_threshold`（更短距离就涨一级）

> 🔧 **默认规则**：未配置维度使用 `"default"` 规则，否则使用内置计算（距离/64）

---

## 👾 生物配置：`entity_settings.json`

示例：

```json
{
  "excluded_mobs": [
    "minecraft:villager",
    "minecraft:iron_golem",
    "minecraft:snow_golem"
  ],
  "mobs": {
    "minecraft:zombie": {
      "enabled": false,
      "health_multiplier": 0.1,
      "damage_multiplier": 0.05,
      "armor_multiplier": 0.05,
      "max_affixes": 3
    },
    "minecraft:zombified_piglin": {
      "enabled": true,
      "health_multiplier": 1.2,
      "damage_multiplier": 0.1,
      "armor_multiplier": 0.1,
      "max_affixes": 5,
      "min_level": 10,
      "max_level": 50,
      "allowed_affixes": [
        "classbioarsenal:tag_teleport"
      ],
      "blocked_affixes": [
        "classbioarsenal:tag_resurrect"
      ],
      "boss_bar": {
        "enabled": true,
        "min_level": 20,
        "color": "RED",
        "overlay": "NOTCHED_10",
        "darken_screen": false,
        "play_boss_music": false,
        "create_world_fog": false
      }
    }
  }
}
```

结构对应：`EntityConfig.EntitySettings` / `MobConfig`：

### 🚫 完全排除的生物

- **`excluded_mobs`**：完全跳过实体系统的生物列表
  - 🤖 友方生物：村民、铁傀儡、雪傀儡
  - ⚖️ 平衡性实体：维护村庄稳定的关键单位

### ⚙️ 特定生物配置

- **`mobs`**：以实体 ID 为键的配置集
  - 🧟 原版生物：`"minecraft:zombie"`、`"minecraft:zombified_piglin"`
  - 🎮 自定义生物：`"modid:my_mob"`

每个生物的配置字段：

- **`enabled`**：
  - ✅ `true`：该生物启用实体系统（会生成等级和词缀）。
  - ❌ `false`：禁用实体系统，即使不在 `excluded_mobs` 中也不会生成等级/词缀。

- **`health_multiplier`**：
  - 每级额外生命值加成系数。
  - 实际计算大致为：
    - `额外生命倍率 ≈ health_multiplier × (等级 - 1)`，
    - 并以“乘法修正”的方式加到最大生命属性上。

- **`damage_multiplier`**：
  - 每级额外攻击力加成系数，计算类似：
    - `额外伤害倍率 ≈ damage_multiplier × (等级 - 1)`。

- **`armor_multiplier`**：
  - 每级额外护甲加成系数，计算方式类似。

- **`max_affixes`**：
  - 该生物**最多可以拥有的怪物词缀数量**。
  - 实际生成时，会在“全局词缀数量规则”和具体生物 `max_affixes` 之间取一个较小值。


- **`min_level`**（可选）：
  - 该生物的**最小等级限制**。
  - 如果维度规则计算出的等级低于此值，则使用此值作为最终等级。
  - 不配置此字段则不应用最小等级限制。

- **`max_level`**（可选）：
  - 该生物的**最大等级限制**。
  - 如果维度规则计算出的等级高于此值，则使用此值作为最终等级。
  - 不配置此字段则不应用最大等级限制。

>  **使用场景**：
> -  **限制新手区 Boss**：设置 `max_level: 10` 防止出生点附近的特殊生物过强
> -  **保证精英怪强度**：设置 `min_level: 20` 确保某些生物始终具有威胁性
> -  **固定等级生物**：同时设置 `min_level: 30` 和 `max_level: 30` 使生物始终为 30 级
> -  **区域等级控制**：配合维度规则，在特定维度限制生物等级范围

- **可选字段：`allowed_affixes` / `blocked_affixes`**：
  - `allowed_affixes`（白名单）：
    - 若配置此字段，该生物**只会从列表中的词缀 ID 中抽取**。
    - 若未配置，则默认可以从全局词缀池中任意抽取。
  - `blocked_affixes`（黑名单）：
    - 若配置此字段，列表中的词条 ID **绝不会出现在该生物身上**。
  - 两者同时存在时：
    - 先按 `allowed_affixes` 过滤候选池 → 再从中移除 `blocked_affixes`。
    - 若过滤后候选池为空，则该生物本次不会生成任何词缀。

- **`boss_bar`**（可选）：
  - 该生物的 **Boss 血条配置**。
  - **`enabled`**：是否启用 Boss 血条。
  - **`min_level`**：显示 Boss 血条所需的最小等级。
  - **`color`**：血条颜色。可选值：`PINK`, `BLUE`, `RED`, `GREEN`, `YELLOW`, `PURPLE`, `WHITE`。
  - **`overlay`**：血条样式。可选值：`PROGRESS`, `NOTCHED_6`, `NOTCHED_10`, `NOTCHED_12`, `NOTCHED_20`。
  - **`darken_screen`**：是否调暗天空背景。
  - **`play_boss_music`**：是否播放 Boss 音乐。
  - **`create_world_fog`**：是否产生世界迷雾。

#### 🔄 生效顺序

1️⃣ 检查 `excluded_mobs` → 完全跳过
2️⃣ 检查 `mobs` 配置
   - ❌ `enabled == false` → 跳过
   - ✅ `enabled == true` → 应用配置
3️⃣ 未配置生物 → 使用默认值

#### 💡 调参建议

- ☠️ **致命怪物**：提高 `health_multiplier`、`damage_multiplier`、`max_affixes`
- 🎯 **平衡怪物**：`enabled: true`，`max_affixes: 0~1`
- 🚫 **完全禁用**：加入 `excluded_mobs`

### 🎁 额外掉落与指令：`extra_drops`

> 从实体等级 + 实体词缀条件出发，附加额外掉落、经验和执行指令。

`MobConfig` 中新增字段：

- `extra_drops`: 额外掉落规则列表，每条规则结构：`ExtraDropRule`

#### 🔧 ExtraDropRule 字段

- `chance` *(可选)*：整条规则触发概率 `0.0 ~ 1.0`，不写视为 `1.0`
- `entity_level` *(可选)*：实体等级条件
  - `min`：最小等级（含）
  - `max`：最大等级（含）
- `required_affixes` *(可选)*：所需实体词缀条件列表（并且关系，需要全部满足）
  - 每个元素：`AffixCondition`
    - `affix_id`：词缀 ID，例如 `classbioarsenal:tag_teleport`
    - `min_level` / `max_level`：该词缀的等级范围
- `items` *(可选)*：额外掉落物品列表
  - 每个元素：`ExtraDropItem`
    - `item`：物品 ID，例如 `minecraft:gold_ingot`
    - `min_count` / `max_count`：数量区间（随机整型）
    - `chance`：该物品自身掉落概率 `0.0 ~ 1.0`，不写视为 `1.0`
- `extra_experience` *(可选)*：额外经验值（整数），在死亡位置生成额外经验球
- `commands` *(可选)*：规则触发后执行的一组指令动作列表，元素为 `CommandAction`

#### 🕹️ CommandAction 字段

- `type`：指令执行方式
  - `"console"`：
    - 以**服务器控制台**身份执行
    - 执行位置 = 怪物死亡位置
  - `"position"`：
    - 以**控制台**身份执行
    - 执行位置由 `x/y/z` 决定（支持占位符）
  - `"killer"`：
    - 以击杀者实体身份执行
    - 若击杀者是玩家，则使用该玩家的命令源
- `as_op`：是否提升为 OP 权限执行
  - `true`：以权限等级 4 执行
  - `false` 或未配置：保持原始权限
- `command`：实际执行的命令字符串
  - 可以带或不带 `/`，内部会自动去掉前导 `/`
- `x` / `y` / `z`：仅在 `type == "position"` 时生效
  - 支持**常数字符串**，例如：`"0"`、`"80"`
  - 也支持占位符：`"{entity_x}"`、`"{entity_y}"`、`"{entity_z}"`

#### 🔣 指令占位符

在任意 `command` 字段中，可以使用以下占位符，在执行前会被替换为实际值：

- `{killer_name}`：击杀者名称（玩家名或实体显示名）
- `{entity_name}`：被击杀实体的显示名
- `{entity_level}`：被击杀实体的等级（`EntityData` 中的 Level）
- `{entity_x}` / `{entity_y}` / `{entity_z}`：实体死亡时所在方块坐标

> `x/y/z` 字段本身也可以使用 `{entity_x}` 等占位符，用来让命令在实体当前位置执行。

#### 示例：为猪灵僵尸添加额外掉落与指令（drops 文件）

推荐将额外掉落配置拆分到 `drops/` 目录下的独立文件中，例如：

- 路径：`data/classbioarsenal/entity_system/drops/minecraft/zombified_piglin.json`

文件内容示例：

```json
{
  "entity_id": "minecraft:zombified_piglin",
  "extra_drops": [
    {
      "chance": 1.0,
      "entity_level": { "min": 5 },
      "required_affixes": [
        {
          "affix_id": "classbioarsenal:tag_teleport",
          "min_level": 1
        }
      ],
      "items": [
        {
          "item": "minecraft:gold_ingot",
          "min_count": 1,
          "max_count": 3,
          "chance": 1.0
        }
      ],
      "extra_experience": 5,
      "commands": [
        {
          "type": "console",
          "as_op": true,
          "command": "say [CBA] {entity_name} Lv.{entity_level} died at {entity_x} {entity_y} {entity_z}"
        },
        {
          "type": "position",
          "as_op": true,
          "x": "{entity_x}",
          "y": "{entity_y}",
          "z": "{entity_z}",
          "command": "say [CBA] Command executed at entity xyz ({entity_x},{entity_y},{entity_z})"
        },
        {
          "type": "killer",
          "as_op": false,
          "command": "say [CBA] {killer_name} killed {entity_name} Lv.{entity_level}"
        },
        {
          "type": "killer",
          "as_op": true,
          "command": "say [CBA] {killer_name} executed this command as OP"
        }
      ]
    }
  ]
}
```

**效果说明：**

- 满足条件的猪灵僵尸死亡时：
  - 额外掉落 1~3 个金锭
  - 掉落额外 5 点经验
  - 控制台广播实体死亡信息（包含等级与坐标）
  - 再以实体当前位置为命令源执行一条测试指令
  - 再以击杀者身份分别执行普通权限与 OP 权限指令

> 你可以将这段配置拷贝/参考到自己的 `drops/<namespace>/<path>.json` 文件中，再按需调整实体 ID、词缀条件、掉落物与指令内容。

---

## ✨ 词缀配置：`affix_config.json`

示例：

```json
{
  "global_settings": {
    "global_max_affixes": 5,
    "count_by_level": {
      "1-5": 1,
      "6-10": 2,
      "11-20": 3,
      "21-50": 4,
      "50-1000": 5
    }
  },
  "affixes": {
    "classbioarsenal:tag_cold": { "min_level": 1, "base_chance": 0.1, "max_level": 5 },
    "classbioarsenal:tag_resurrect": { "min_level": 10, "base_chance": 0.05, "max_level": 5 },
    "classbioarsenal:tag_brutal": { "min_level": 5, "base_chance": 0.1, "max_level": 5 }
    // ... 其它词缀略
  }
}
```

结构对应：`EntityConfig.AffixSettings` / `GlobalAffixSettings` / `AffixDefinition`。

### 🌐 全局设置

- **`global_max_affixes`**：全局词缀数量上限
- **`count_by_level`**：按等级建议词缀数量
  - 📊 等级 1~5 → 1 个词缀
  - 📈 等级 6~10 → 2 个词缀
  - ⚡ 等级 11~20 → 3 个词缀
  - 🔥 等级 21~50 → 4 个词缀
  - 👑 等级 50+ → 5 个词缀

### 🎯 词缀定义

#### 🔑 词缀 ID
- 🧊 `tag_cold` - 冰冷效果
- 💀 `tag_resurrect` - 复活能力
- ⚔️ `tag_brutal` - 残暴伤害
- 💥 `tag_explosive` - 爆炸攻击
- 💰 `tag_greedy` - 额外掉落
- 🩸 `tag_vampire` - 吸血效果
- ✨ `tag_lightburst` - 光爆致盲
- 🏃 `tag_frenzy` - 狂乱加速
- 🌟 `tag_unmatched` - 极强综合
- 👑 `tag_apotheosis` - 神化终极

- **值（Value）**：`AffixDefinition` 对象，常用字段：

  - **`min_level`**：
    - 怪物至少达到多少级才有资格使用该词缀。
    - 例如 `min_level: 20` 表示只会出现在 20 级及以上的怪身上。

  - **`base_chance`**：
    - 该词缀作为候选时的基础概率（0 ~ 1），用于区分“常见词缀 / 稀有词缀”。
    - 如 `0.1` ≈ 10% 基础概率，`0.01` ≈ 1%（极稀有）。

  - **`max_level`**：
    - 该词缀本身的最高等级。
    - 实际生成时，会根据怪物等级简单推一个词缀等级，并用此字段进行上限裁剪。

  - **可选字段（当前 JSON 示例中未使用，但结构支持）：**

    - `growth_per_level`：
      - 设计用于规定该词缀每升一级在数值上的成长（由具体效果逻辑使用）。

    - `params`：
      - 自由参数表，键值对形式，用于为某些复杂词缀提供额外参数（例如半径、触发冷却等）。

    - `conflicts`：
      - 一个字符串列表，列出与该词缀**互斥**的其他词缀 ID。
      - 方便避免“逻辑上不该共存”的组合（例如同时拥有极端减伤与极端爆发）。

**现有怪物词缀（概念示例）：**

- `tag_cold`：冰冷，减速或冰冻效果。
- `tag_resurrect`：复活能力，死亡时可以复活一次 or 多次。
- `tag_brutal`：残暴，高额伤害提升。
- `tag_explosive`：爆炸攻击或死亡时爆炸。
- `tag_greedy`：掉落更多战利品或吸取玩家经验。
- `tag_vampire`：吸血效果，根据伤害量回复自身生命。
- `tag_lightburst`：光爆，施加失明/黑暗等效果。
- `tag_frenzy`：狂乱，攻速或移动速度大幅提升。
- `tag_swift`：极速移动。
- `tag_unmatched`：极强的综合加成（非常稀有）。
- `tag_soul`：与灵魂相关的特殊伤害或减免。
- `tag_teleport`：短距离传送、闪现。
- `tag_celestial`：更高层次的增益（终局向）。
- `tag_leader`：统领光环，加强附近小弟。
- `tag_desperate`：濒死狂化。
- `tag_combo`：连击相关效果。
- `tag_adapt`：对玩家攻击方式进行适应性防御。
- `tag_disarm`：缴械玩家。
- `tag_apotheosis`：神化/终极 Boss 方向的极强词缀。
- `tag_guerrilla`：游击战术类效果。
- `tag_domain`：领域展开，控制一定范围内的战斗环境。

> 注意：上面是概念说明，具体数值和效果取决于实际实现与后续更新。

#### 💡 调参建议

- 👑 **Boss 专属**：`min_level: 40+`，`base_chance: 0.01`
- 🎲 **常见增益**：降低 `min_level`，提高 `base_chance`
- 🚫 **临时禁用**：移除或设置极高 `min_level` + 极低 `base_chance`

---

## 🔄 配置生效方式

1. 💾 **编辑 JSON** → 保存文件
2. 🔄 **重载资源** → `/reload` 或重启世界
3. 👀 **观察效果** → 新怪物按新规则生成

---

## 🎯 整体调参思路

### 📐 先设计难度曲线
- 🌍 主世界、下界、末地的等级区间规划
- 👾 普通怪/精英怪/Boss 的预期强度
- ⚔️ 词缀数量分配策略

### 🛡️ 从安全配置开始
- 🔰 初版设低倍率和词缀数量
- 📈 逐步调整，观察战斗体验

### 🎨 精细控制
- 🏘️ 村庄功能实体 → 加入 `excluded_mobs`
- 👑 危险 Boss → 提高 `mobs` 倍率

### ⭐ 稀有词缀设计
- 🎆 `tag_apotheosis`、`tag_domain` → 终局事件级
- 💎 极高 `min_level` + 极低 `base_chance`

---

> 💡 **需要帮助？**
> - 📖 英文版教程
