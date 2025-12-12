---
title: 职业系统数据包配置(cbs)
tags:
  - pfc
  - ca
  - mc
  - mod
categories:
  - pfc
  - ca
date: 2025-11-23 018:26:19
---
# 👥 ClassBioArsenal 职业系统配置

> 职业定义与物品职业限制指南

---

## 📁 目录结构

### 🎓 职业定义：`professions/`

**路径**：`data/classbioarsenal/professions/`

- 📄 每个 `*.json` 定义一个职业
- 🏷️ 文件名与职业 ID 相同：
  - `berserker.json` - 狂战士
  - `dual_swordsman.json` - 双剑士
  - `heavy_swordmaster.json` - 重剑圣

> 🔧 由 `ProfessionManager` 加载

### ⚔️ 物品职业限制：`item_professions/`

**路径**：`data/classbioarsenal/item_professions/`

- 📄 每个 `*.json` 定义物品的职业限制
- 🎯 示例文件：
  - `diamond_sword.json`
  - `minecraft_bow.json`
  - `minecraft_diamond_chestplate.json`

> 🔧 由 `ItemProfessionConfigManager` 加载

---

## 🎓 职业配置：`professions/*.json`

以 `berserker.json`（狂战士）为例：

```json
{
  "name": "berserker",
  "displayName": "狂战士",
  "isNormal": false,
  "upperProfession": "warrior",
  "professionLevel": 2,
  "maxLevel": 15,
  "maxExp": 200,
  "multiplier": 3,
  "attributeBonuses": [
    {
      "attribute": "minecraft:generic.max_health",
      "amountPerLevel": 30,
      "operation": "add"
    },
    {
      "attribute": "minecraft:generic.armor",
      "amountPerLevel": 3,
      "operation": "add"
    },
    {
      "attribute": "minecraft:generic.attack_damage",
      "amountPerLevel": 6,
      "operation": "add"
    },
    {
      "attribute": "minecraft:generic.attack_speed",
      "amountPerLevel": 1,
      "operation": "add"
    }
  ],
  "startingGear": [
    {
      "itemId": "minecraft:iron_sword",
      "count": 1,
      "slot": "mainhand"
    },
    {
      "itemId": "minecraft:iron_chestplate",
      "count": 1,
      "slot": "chest"
    }
  ]
}
```

### 🆔 基础身份字段

- **`name`**（必填）：职业 ID（内部识别）
  - 🏷️ 例：`"berserker"`、`"dual_swordsman"`

- **`displayName`**（必填）：显示名称
  - 📝 例：`"狂战士"`、`"双剑士"`

- **`isNormal`**（必填）：是否基础职业
  - ✅ `true`：基础职业（可直接选择）
  - ⬆️ `false`：进阶职业（需转职）

- **`upperProfession`**（可空）：上级职业 ID
  - 🌳 例：`"warrior"`（进阶职业的上级）

- **`professionLevel`**（必填）：职业层级
  - 📊 1 = 基础职业，2 = 二阶职业...

### 📈 等级与经验成长

- **`maxLevel`**（必填）：最大等级

- **`maxExp`**（必填）：1→2 级基础经验

- **`multiplier`**（必填）：成长倍率
  - 📊 经验需求：`下一级 ≈ 上一级 × multiplier`
  - 💪 属性成长：参考此倍率放大

> 💡 **示例**：`maxExp = 200`，`multiplier = 3`
> - 1→2 级：200 EXP
> - 2→3 级：600 EXP
> - 3→4 级：1800 EXP

### ⚔️ 属性加成字段

- **`attributeBonuses`**：属性加成列表（可空）
  - 每一项代表“某个 Attribute 在每级增加多少”
  - 支持原版与其他 Mod 的自定义 Attribute（只要写正确的注册名）

#### ✅ attributeBonuses 单项结构

```json
{
  "attribute": "minecraft:generic.max_health",
  "amountPerLevel": 30,
  "operation": "add"
}
```

#### 🔧 字段说明

- **`attribute`**：Attribute 注册名（ResourceLocation）
  - 推荐写“标准注册名”，例如：
    - `minecraft:generic.max_health`
    - `minecraft:generic.armor`
    - `minecraft:generic.attack_damage`
    - `minecraft:generic.attack_speed`
  - 你也可以写其他 Mod 的 Attribute，例如：`othermod:my_attribute`

- **`amountPerLevel`**：每级加成数值（double）

- **`operation`**：加成方式（字符串）
  - `add`：固定值加法
  - `multiply_base`：按基础值比例增加
  - `multiply_total`：按最终值比例增加

> 💡 **基础面板**：HEALTH=10, ARMOR=0, DAMAGE=1, DAMAGE_SPEED=1
> 📊 使用 `/profession info <职业ID>` 查看实际数值

### 🎒 初始装备：`startingGear`

- **`startingGear`**（可选）：职业初始装备

```json
{
  "itemId": "minecraft:iron_sword",
  "count": 1,
  "slot": "mainhand"
}
```

#### 🔧 字段说明
- **`itemId`**：物品 ID
- **`count`**：数量
- **`slot`**：装备位置
  - 🗡️ `"mainhand"` - 主手
  - 🛡️ `"offhand"` - 副手
  - 🎩 `"head"` - 头盔
  - 👕 `"chest"` - 胸甲
  - 👖 `"legs"` - 护腿
  - 👢 `"feet"` - 靴子

> 💡 **建议**：基础职业给简单装备，高阶职业给强力装备

### 🧾 首次选择职业指令：`firstSelectCommands`

- **`firstSelectCommands`**（可选）：玩家**第一次获得该职业**时，在服务器端执行的一组指令

```json
"firstSelectCommands": [
  "say %player% 成为了铁剑卫！",
  "give %player% minecraft:golden_apple 1"
]
```

#### 🔧 行为与占位符
- 触发时机：
  - 玩家从“无职业” → 第一次变成这个职业时触发（包括 `/profession player set`）。
  - 之后通过升级 / 重登不会再次触发。
- 占位符：
  - **`%player%`**：当前选择该职业的玩家名称。

> 💡 **说明**：字段为可选，未配置时不会执行任何首次指令。

### 📈 升级触发指令：`levelCommands`

- **`levelCommands`**（可选）：当玩家该职业**等级提升到指定等级**时执行的指令列表

```json
"levelCommands": [
  {
    "level": 5,
    "commands": [
      "say %player% 的铁剑卫已升到 %level% 级，获得阶段奖励！",
      "give %player% minecraft:iron_ingot 8"
    ]
  },
  {
    "level": 10,
    "commands": [
      "say %player% 的铁剑卫达到了 %level% 级，获得特别奖励！",
      "give %player% minecraft:diamond 1"
    ]
  }
]
```

#### 🔧 行为与细节
- 触发时机：
  - 只有在职业经验通过系统升级（如 `/profession player addxp`）时触发。
  - 每次升级时，内部按等级 **一档一档循环** 调用 `levelUp`：
    - 如果一次获得大量经验从 1 级直接升到 10 级，且配置了 5 和 10 级，**5 级和 10 级的指令都会依次触发**，不会被跳过。
- 占位符：
  - **`%player%`**：当前升级的玩家名称。
  - **`%level%`**：玩家升级后达到的职业等级（整数）。

> 💡 **说明**：`levelCommands` 完全可选，未配置时升级只会改变属性，不会执行额外指令。

---

## ⚔️ 物品职业限制：`item_professions/*.json`

以 `diamond_sword.json` 为例：

```json
{
  "itemId": "minecraft:diamond_sword",
  "professions": [
    "warrior",
    "berserker"
  ],
  "descriptionPrefix": "职业限制: ",
  "showInLore": true,
  "loreColor": "aqua"
}
```

以及 `minecraft_bow.json`：

```json
{
  "itemId": "minecraft:bow",
  "professions": [
    "archer",
    "ranger"
  ],
  "descriptionPrefix": "职业限制: ",
  "showInLore": true,
  "loreColor": "green"
}
```

### 🆔 基础字段

- **`itemId`**（必填）：物品 ID
  - 🏷️ 格式：`"modid:itemname"`
  - 🗡️ 例：`"minecraft:diamond_sword"`

- **`professions`**（必填）：允许使用的职业列表
  - 👥 必须与职业配置中的 `name` 一致
  - 🔄 内部统一转为小写匹配

> ⚖️ **使用规则**：职业在列表中 ✅ 可使用 | 不在列表中 ❌ 受限制

### 🎨 展示控制字段

- **`descriptionPrefix`**（可选）：前缀文本
  - 📝 例：`"职业限制: "` → 显示 `职业限制: 战士, 狂战士`
  - 🎯 高阶装备：`"仅限职业: "`

- **`showInLore`**（可选）：是否在物品说明中显示
  - 👁️ `true`：显示限制信息（默认）
  - 🔒 `false`：仅逻辑限制，不显示

- **`loreColor`**（可选）：文本颜色
  - 🎨 支持 Minecraft 颜色名：`"white"`、`"green"`、`"red"` 等
  - ⚠️ 警告色：`"red"`、`"dark_red"`

> 💡 **建议**：高阶武器用醒目文字+警告色，通用装备设多职业

---

## 🔄 配置生效方式

### 🎓 职业配置重载

1. 💾 **编辑保存** `professions/*.json`
2. ⚡ **执行指令**：
   ```text
   /profession reload professions  # 仅职业
   /profession reload all          # 全部
   ```

### 🧾 导出当前已注册的 Attribute 列表（排查用）

当你不确定某个 Attribute 的“注册名”到底应该写什么时，可以在游戏内执行：

```text
/profession dump_attributes
```

它会把服务器当前已注册的所有 Attribute 注册名导出到运行目录的：

```text
classbioarsenal_dumps/
```

### ⚔️ 物品职业配置重载

1. 💾 **编辑保存** `item_professions/*.json`
2. ⚡ **执行指令**：
   ```text
   /profession reload items       # 仅物品
   /profession reload all          # 全部
   ```

---

## 🎯 设计与调参建议

### 🌳 规划职业树
- 🗺️ 先设计"基础职业 → 进阶职业"结构
- 📝 填写 `isNormal` / `upperProfession` / `professionLevel`

### 📈 经验与数值平衡
- 🔰 初版：`maxLevel` 10~20，`maxExp` 适中
- 🎮 根据玩家体验微调 `multiplier` 与 `maxLevel`

### ⚔️ 职业特色定位
- 🛡️ **坦克类**：高 `minecraft:generic.max_health`、中 `minecraft:generic.armor`、低 `minecraft:generic.attack_damage`
- ⚔️ **输出类**：中 `minecraft:generic.max_health`、低 `minecraft:generic.armor`、高 `minecraft:generic.attack_damage`/`minecraft:generic.attack_speed`
- 🔄 **混合类**：各项平衡

### 🎨 物品限制策略
- 🎯 引导风格而非强制锁死
- 👥 避免所有好装备只给少数职业

### 🛠️ 热加载调试
- ⚡ `/profession reload professions` + `/profession reload items`
- 🧪 `/profession player set` + `/profession player addxp` 测试

---
