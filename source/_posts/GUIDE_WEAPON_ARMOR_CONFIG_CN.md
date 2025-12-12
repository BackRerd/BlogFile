---
title: 防具与武器数据包配置(cbs)
tags:
  - pfc
  - ca
  - mc
  - mod
categories:
  - pfc
  - ca
date: 2025-11-23 22:22:22
---
# ⚔️ ClassBioArsenal 武器防具系统配置

> 武器稀有度、词缀与升级系统指南

---

## 📁 目录结构

### 🗡️ 武器系统 `weapon_system`

**路径**：`data/classbioarsenal/weapon_system/`

- 📋 `weapon_config.json` - 武器白名单/黑名单
- 💎 `rarities.json` - 稀有度配置（普通、稀有、史诗）
- ✨ `affixes.json` - 词缀配置（攻击力、暴击、火伤）
- 📈 `leveling.json` - 升级与经验曲线

### 🛡️ 防具系统 `armor_system`

**路径**：`data/classbioarsenal/armor_system/`

- 📋 `armor_config.json` - 防具白名单/黑名单
- 💎 `armor_rarities.json` - 稀有度配置
- ✨ `armor_affixes.json` - 词缀配置（反伤、限伤、回血）
- 📈 `armor_leveling.json` - 升级与经验曲线

---

## 🗡️ 武器系统配置

### 📋 武器配置：`weapon_config.json`

示例：

```json
{
  "allowed_weapons": [
    "minecraft:diamond_sword",
    "minecraft:netherite_sword",
    "classbioarsenal:kkkk",
    "classbioarsenal:example_sword"
  ],
  "blocked_weapons": []
}
```

- **`allowed_weapons`**：允许参与的武器列表
  - ✅ 列表内物品视为有效武器
  - 🗡️ 例：`"minecraft:diamond_sword"`、`"modid:custom_blade"`

- **`blocked_weapons`**：禁止参与的武器列表
  - ❌ 即使是剑、斧、三叉戟也会被排除

#### 📜 默认行为
- 不在 `blocked_weapons` 中：
  - ✅ `allowed_weapons` 中的物品 → 有效武器
  - 🎮 原版剑、斧、三叉戟 → 默认兼容

#### 💡 使用建议

- 🚫 **完全禁用**：加入 `blocked_weapons`
- ➕ **添加自定义**：加入 `allowed_weapons`，确认不在黑名单

#### 🎯 按物品定制：`item_rules`

> 用于为**某个具体物品**单独指定：
> - 最低稀有度
> - 必定刷出的词缀
> - 永远不会出现的词缀

在 `weapon_config.json` 中增加：

```jsonc
{
  "allowed_weapons": [
    "minecraft:diamond_sword",
    "minecraft:netherite_sword",
    "classbioarsenal:example_sword"
  ],
  "blocked_weapons": [],
  "item_rules": {
    "minecraft:diamond_sword": {
      "min_rarity": ["rare"],           // 只有一个值 = 固定 rare 品质
      "forced_affixes": [
        "ATTACK_DAMAGE",                  // 必定拥有：攻击力
        "CRITICAL_RATE"                   // 必定拥有：暴击率
      ],
      "blocked_affixes": [
        "DAMAGE_PER_SECOND",              // 不会出现：秒伤
        "LIGHTBURST"                      // 不会出现：光爆
      ]
    }
  }
}
```

字段说明：

- **`item_rules`**：`物品ID -> 规则对象` 的映射
- **`min_rarity`**：稀有度区间列表，需与 `rarities.json` 中的 `id` 一致
  - `["rare"]`：固定刷新为 `rare`
  - `["rare", "epic"]`：在 `rare` ~ `epic` 区间内随机（含两端），按各自权重
- **`forced_affixes`**：必定添加的词缀 ID 列表（来自 `affixes.json` 的 `id`）
- **`blocked_affixes`**：禁止出现的词缀 ID 列表

行为细节：

- 总词缀数仍由稀有度的 `min_affixes/max_affixes` 随机决定
- 系统会先添加 `forced_affixes`，再从剩余候选池中随机补足
- `blocked_affixes` 会从候选池中移除，确保该物品永远不会 roll 到这些词缀
- 若随机出的稀有度低于 `min_rarity`，则自动提升到 `min_rarity`

> ✅ 适合做**专属武器**：例如某把故事向武器总是带吸血/暴击，不会带 DOT/控制类词缀

---

### 💎 稀有度配置：`rarities.json`

示例（节选）：

```json
{
  "rarities": [
    {
      "id": "common",
      "display_name": "普通",
      "color_code": "§7",
      "stat_multiplier": 1.0,
      "weight": 100,
      "min_affixes": 0,
      "max_affixes": 1
    },
    {
      "id": "epic",
      "display_name": "史诗",
      "color_code": "§5",
      "stat_multiplier": 1.5,
      "weight": 20,
      "min_affixes": 2,
      "max_affixes": 3
    }
  ]
}
```

#### 🔑 稀有度字段

- **`id`**：稀有度 ID（内部使用）
  - 🏷️ 例：`"common"`、`"epic"`

- **`display_name`**：显示名称
  - 📝 例：`"普通"`、`"史诗"`

- **`color_code`**：前缀颜色码
  - 🎨 使用 `§` 开头：`§7`、`§9` 等

- **`stat_multiplier`**：基础伤害倍率
  - 📊 `1.0` = 不加成，`1.5` = 伤害 × 1.5

- **`weight`**：抽取权重
  - 🎲 数值越大越常见
  - 📊 例：`common` 100，`legendary` 5

- **`min_affixes` / `max_affixes`**：词缀数量范围
  - ✨ 随机词缀数量范围

**使用建议：**

- 调低 `legendary.weight`，可以让传说武器更稀有。
- 提高 `stat_multiplier` 能显著增强该稀有度武器整体强度。
- `min_affixes` / `max_affixes` 决定了同稀有度武器的“词缀数量档位”。

---

### ✨ 词缀配置：`affixes.json`

示例（节选）：

```json
{
  "affixes": [
    {
      "id": "ATTACK_DAMAGE",
      "display_name": "攻击力",
      "description": "提高基础攻击伤害",
      "type": "STAT_FLAT",
      "min_value": 2.0,
      "max_value": 20.0,
      "max_level": 10,
      "growth_per_level": 2.0,
      "color": "§c",
      "weight": 100
    },
    {
      "id": "FIRE_DAMAGE",
      "display_name": "火焰伤害",
      "description": "附加火焰伤害并点燃目标",
      "type": "EFFECT_DAMAGE",
      "min_value": 2.0,
      "max_value": 12.0,
      "max_level": 10,
      "growth_per_level": 1.0,
      "color": "§6",
      "weight": 60
    }
  ]
}
```

公共字段说明：

- **`id`**：词缀 ID（内部名称），与代码中的效果逻辑一一对应。
- **`display_name`**：词缀显示名（物品描述中显示）。
- **`description`**：词缀效果说明文本，用于详细说明界面。
- **`type`**：词缀类别标识，用于区分“数值型”“效果型”等（不建议随意乱填）。
- **`min_value`**：
  - 1 级词缀的基础数值。
  - 例如 `ATTACK_DAMAGE` 1 级 = +2.0 伤害。
- **`max_value`**：
  - 设计上的最高数值，用于你自己把握平衡（代码主要通过 `min_value + (等级-1)*growth_per_level` 推导实际值）。
- **`max_level`**：词缀最大等级（例如 10 级）。
- **`growth_per_level`**：
  - 每升 1 级增加的数值。实际值约为：
  - `当前值 ≈ min_value + (当前等级 - 1) * growth_per_level`
- **`color`**：显示颜色码，类似 `§c`、`§6` 等。
- **`weight`**：
  - 抽取该词缀时的随机权重，数值越大越容易出现。
- **`params`**（可选）：
  - 少部分词缀有额外参数（如恢复间隔等），以 `key: value` 的形式记录。

**常见武器词缀示例：**

- `ATTACK_DAMAGE`：额外平砍伤害。
- `CRITICAL_RATE`：暴击概率（%）。
- `CRITICAL_DAMAGE`：暴击伤害倍率。
- `FIRE_DAMAGE`：额外火焰伤害，并点燃目标。
- `ICE_DAMAGE`：减速 + 冰伤。
- `SOUL_DAMAGE`：对“灵魂类”生物额外伤害。
- `DAMAGE_PER_SECOND`：持续伤害（当前实现为凋零等）
- `LIFESTEAL_RATE` / `LIFESTEAL_MULTIPLIER`：吸血概率与吸血倍率。
- `LIGHTBURST`：攻击时施加失明和黑暗。

#### 💡 调参建议

- 🎲 **更常见**：提高 `weight`
- 💪 **更强力**：提高 `min_value`/`growth_per_level`/`max_level`
- 🚫 **临时禁用**：设 `weight: 0` 或移除

> ⚠️ **注意**：避免多重加成叠加导致伤害失控

---

### 📈 升级配置：`leveling.json`

示例：

```json
{
  "max_level": 10,
  "xp_curve": [100, 250, 500, 900, 1500, 2300, 3500, 5000, 7000, 10000],
  "damage_per_level": 0.5,
  "affix_unlock_chance": {
    "3": 0.3,
    "6": 0.5,
    "9": 0.8,
    "10": 1.0
  },
  "affix_upgrade_chance": 0.4
}
```

#### 🔧 配置字段

- **`max_level`**：武器最高等级

- **`xp_curve`**：每级所需经验列表
  - 📊 第1个值 = 1→2级所需经验
  - 📈 列表长度 ≥ `max_level`

- **`damage_per_level`**：每级伤害加成
  - ⚔️ 例：`0.5` = 每级 +0.5 伤害

- **`affix_unlock_chance`**：特定等级解锁新词缀概率
  - 🔑 键 = 等级（字符串）
  - 🎲 值 = 概率（0~1）
  - ✨ 成功时随机添加新词缀

- **`affix_upgrade_chance`**：升级已有词缀概率
  - ⬆️ 未解锁新词缀时的升级概率（0~1）

#### 💡 调参建议

- ⚡ **升级更快**：降低 `xp_curve` 数值
- 📈 **高级差距更大**：提高 `damage_per_level` 或高级XP
- ✨ **词缀更频繁**：增加 `affix_unlock_chance` 档位和概率

---

## 🛡️ 防具系统配置

> 结构与武器系统类似，字段名稍有不同

### 📋 防具配置：`armor_config.json`

- **`allowed_armors`**：允许加入的防具列表
- **`blocked_armors`**：强制排除的防具列表

> 💡 **建议**：显式列出希望参与系统的防具

#### 🎯 按物品定制：`item_rules`

与武器的 `item_rules` 类似，可为**每一件防具**单独指定：

- 最低稀有度区间（`min_rarity` 列表，对应 `armor_rarities.json`）
- 必定拥有的防具词缀（`forced_affixes`，对应 `armor_affixes.json`）
- 永远不会出现的防具词缀（`blocked_affixes`）

示例：让钻石胸甲至少为稀有，并且必定带“重甲 + 恢复”，永远不会出现“狂躁”：

```jsonc
{
  "allowed_armors": [
    "minecraft:diamond_helmet",
    "minecraft:diamond_chestplate",
    "minecraft:diamond_leggings",
    "minecraft:diamond_boots"
  ],
  "blocked_armors": [],
  "item_rules": {
    "minecraft:diamond_chestplate": {
      "min_rarity": ["rare"],
      "forced_affixes": [
        "HEAVY_ARMOR",   // 重甲：受伤前减固定数值
        "REGEN"          // 恢复：持续回血
      ],
      "blocked_affixes": [
        "BERSERK"        // 狂躁：提高攻击但自伤，这里禁止在胸甲上出现
      ]
    }
  }
}
```

> ⚠️ 注意：
> - 词缀 ID 必须存在于 `armor_affixes.json` 中
> - 稀有度 ID 必须存在于 `armor_rarities.json` 中
> - 规则仅对**初始化时**的新装备生效，已有 NBT 的装备不会自动重塑

---

### 💎 防具稀有度：`armor_rarities.json`

> 与武器稀有度结构相同，可独立调整权重和加成

- **`stat_multiplier`**：防具减伤或相关数值倍率
- **`min_affixes` / `max_affixes`**：初始化词缀数量范围
- **`weight`**：出现概率

---

### ✨ 防具词缀：`armor_affixes.json`

> 字段含义与武器词缀相同

#### 🛡️ 常见防具词缀

- 🛡️ `HEAVY_ARMOR` - 重甲：受伤前减固定数值
- ⚡ `DAMAGE_LIMIT` - 限伤：单次伤害上限
- 🎯 `FIXED_DAMAGE` - 定伤：只承受固定伤害
- 🌵 `THORNS` - 反甲：反弹伤害给攻击者
- ❤️ `REGEN` - 恢复：持续回血（可调间隔）
- 🔧 `REPAIR` - 修复：定期恢复耐久
- 💨 `DODGE` - 闪避：几率完全闪避伤害
- ⚔️ `SLOW/STUN/EXPLOSIVE/SHOCK` - 控制/反击效果
- 👻 `SOUL_IMMUNE` - 免疫灵魂/魔法伤害
- 🔥 `BERSERK` - 狂躁：提高伤害但自损生命

> 💡 **调参**：通过 `min_value`、`growth_per_level`、`max_level`、`weight` 控制

---

### 📈 防具升级：`armor_leveling.json`

> 与武器升级配置完全一致，作用对象为防具

- **`max_level`**：防具最大等级
- **`xp_curve`**：每级所需经验
- **`damage_per_level`**：每级减伤或相关数值加成
- **`affix_unlock_chance`**：解锁新防具词缀概率
- **`affix_upgrade_chance`**：提升已有词缀等级概率


---

## 🔄 配置生效方式

1. 💾 **编辑保存** JSON 文件
2. ⚡ **执行指令**：
   ```text
   /cba_weapon reload              # 重载武器/防具
   /profession reload all          # 重载职业（如需）
   ```
3. 👀 **观察效果**：新生成物品或已有物品表现

---

## 🎯 调参小贴士

### 🔰 从小改起
- 微调数值，测试几件装备，逐步调整

### 🎲 权重控制稀有度
- `weight` 越低越稀有
- 💡 建议：普通词缀权重大，变态词缀权重极低

### ⚠️ 避免多重加成失控
- 总伤害 ≈ 基础伤害 × 稀有度倍率 + 等级加成 + 词缀加成
- 🚫 不要同时拉满所有倍率

### 📊 先设计再实现
- 表格计算不同等级/稀有度/词缀的伤害区间
- 再写入 JSON 配置

---

> 💡 **需要更多帮助？**
> - 📖 英文版配置教程
> - 🎯 具体词缀/稀有度平衡建议
