---
title: 实体API(cbs)
tags:
  - pfc
  - ca
  - mc
  - mod
categories:
  - pfc
  - ca
date: 2025-12-12 18:12:45
---
# ClassBioArsenal 实体机制（Entity System）API 说明

本文档介绍如何针对 ClassBioArsenal 的 **实体机制系统** 进行二次开发，包括：

- 使用 Java 端 `MechanicsAPI` 读取 / 修改实体等级、实体词条等运行时数据；
- 基于 JSON（`entity_system`）定义的机制在 Java 中的等价操作；
- 在 Java 中动态注册新的实体词条定义、实体配置和掉落规则。

> 说明：以下内容基于 Forge 1.20.1。

---

## 1. Java MechanicsAPI

对外入口类：

- 包名：`site.backrer.classbioarsenal.api`
- 类名：`MechanicsAPI`

只要在运行时依赖 ClassBioArsenal 的 jar，就可以在你的模组中直接调用这些静态方法。

### 1.1 实体基础数据与 EntityData 能力

- `@Nullable String getEntityId(LivingEntity entity)`  
  返回实体注册 ID（例如 `minecraft:zombie`、`classbioarsenal:xxx`）。

- `@Nullable EntityData getEntityData(LivingEntity entity)`  
  返回实体附加的 `EntityData` 能力对象；仅对 **非玩家 LivingEntity** 附加，
  玩家不会有该能力。

- `boolean hasEntityData(LivingEntity entity)`  
  检查实体是否拥有 `EntityData` 能力。

### 1.2 运行时：实体等级与词条

- `int getEntityLevel(LivingEntity entity)`  
  获取实体等级；如果未初始化或无 `EntityData`，返回 `0`。

- `void setEntityLevel(LivingEntity entity, int level, boolean refreshAttributes)`  
  设置实体等级，并可选是否立即调用内部的 `EntityEventHandler.applyAttributes` 来刷新等级带来的属性加成。

- `Map<String, Integer> getEntityAffixLevels(LivingEntity entity)`  
  获取实体当前所有词条的 `(affixId -> level)` 映射；
  词条 ID 一般为形如 `classbioarsenal:tag_cold` 的字符串。

- `boolean hasEntityAffix(LivingEntity entity, String affixId)`  
  检查实体是否拥有某个词条。

- `int getEntityAffixLevel(LivingEntity entity, String affixId)`  
  获取实体指定词条等级；不存在则返回 `0`。

- `void addOrSetEntityAffix(LivingEntity entity, String affixId, int level)`  
  为实体添加或覆盖一个词条等级（直接操作 `EntityData`）。

- `void removeEntityAffix(LivingEntity entity, String affixId)`  
  从实体上移除指定词条。

> 注意：
> - 大部分“行为类”实体词条（如 COLD、BRUTAL 等）会在事件中按需检查 `hasAffix` 并生效，
>   因此通过 API 新增这些词条通常立即生效。  
> - 少数“属性类”词条（如 SWIFT / UNMATCHED）的属性加成是在实体生成时应用一次，
>   若你在运行时通过 API 修改这些词条，可能需要自行重新应用属性（例如重新调用 `applyAttributes` 或编写自定义逻辑）。

---

## 2. 配置级别：等级规则（LevelConfig）

实体等级规则由 `entity_system/level_config.json` 等 JSON 定义，对应 Java 结构为 `EntityConfig.LevelConfig`。

### 2.1 读取与设置维度等级规则

- `@Nullable EntityConfig.DimensionRule getLevelRule(String dimensionId)`  
  获取某维度的等级规则；维度 ID 例如：
  - `minecraft:overworld`
  - `minecraft:the_nether`

- `void setLevelRule(String dimensionId, int baseLevel, double levelMultiplier, int distanceThreshold)`  
  为某个维度设置或更新等级规则：

  - `baseLevel`：基础等级；
  - `levelMultiplier`：随距离增长的等级倍率；
  - `distanceThreshold`：距离阈值（决定多少距离提升一级）。

通过该 API 修改的规则，会影响后续新刷出的实体在 `EntityEventHandler.calculateLevel` 中的等级计算。

---

## 3. 配置级别：实体词条定义（AffixSettings.affixes）

实体词条定义由 `entity_system/affix_config.json` 中的 `affixes` 字段管理，对应 Java 结构为 `EntityConfig.AffixSettings.affixes`。

### 3.1 读取实体词条定义

- `Map<String, EntityConfig.AffixDefinition> getAllEntityAffixDefinitions()`  
  获取所有实体词条定义（key 为词条 ID，如 `classbioarsenal:tag_cold`）。

- `@Nullable EntityConfig.AffixDefinition getEntityAffixDefinition(String affixId)`  
  获取单个实体词条定义。

`AffixDefinition` 结构中包含：

- `min_level`：出现所需最低实体等级；
- `base_chance`：基础概率；
- `growth_per_level`：随实体等级增加的权重 / 概率成长；
- `max_level`：该词条最大等级；
- `params`：可扩展的参数 Map（由具体词条逻辑解释）；
- `conflicts`：冲突词条 ID 列表。

> 当前版本中，`AffixHelper.generateAffixes` 只使用了 `max_level` 来限制生成等级，
> 但你依然可以通过 `params` 和 `conflicts` 在自定义逻辑中使用这些字段。

此外，`affix_config.json` 中还包含 `global_settings`：

- `global_max_affixes`：全局最大实体词条数量；
- `count_by_level`：按实体等级区间控制词条数量，例如 `"1-5": 1` 表示 1～5 级只生成 1 个词条。

对应的 Java API 为：

- `@Nullable EntityConfig.GlobalAffixSettings getGlobalAffixSettings()` /  
  `void setGlobalAffixSettings(@Nullable EntityConfig.GlobalAffixSettings settings)`  
  直接读写整个 `global_settings` 对象。

- `int getGlobalMaxAffixes()` / `void setGlobalMaxAffixes(int max)`  
  读写 `global_max_affixes` 字段。

- `Map<String, Integer> getAffixCountByLevel()` /  
  `void setAffixCountByLevel(@Nullable Map<String, Integer> counts)`  
  读写 `count_by_level` 映射（如 `"1-5" -> 1`）。

### 3.2 在 Java 中注册实体词条定义

- `boolean registerEntityAffix(String id, EntityConfig.AffixDefinition def)`  
  直接注册一个完整的实体词条定义。ID 已存在时返回 `false`，不会覆盖原有配置。

- `boolean registerEntityAffix(String id, int minLevel, double baseChance, double growthPerLevel, int maxLevel, @Nullable Map<String, Object> params, @Nullable List<String> conflicts)`  
  使用字段参数构造并注册一个新的词条定义。

> 建议：在配置加载完成之后（资源重载完成或服务器启动后）调用注册方法，
> 例如在你的模组 CommonSetup 阶段中 `enqueueWork` 执行。

注册成功后：

- 该词条会被加入 `EntitySystemManager.AFFIX_CONFIG.affixes`；
- `AffixHelper.generateAffixes` 在为实体生成词条时会把它作为候选之一（受 allowed/blocked 等过滤影响）；
- 你可以在事件中通过 `hasEntityAffix` / `getEntityAffixLevel` 编写该词条的行为逻辑。

---

## 4. 配置级别：每个实体的 MobConfig 与掉落规则

实体专属配置与额外掉落规则由 `entity_system/entity_settings.json` 和 `entity_system/drops/*.json` 管理，
对应 Java 结构为：

- `EntitySystemManager.ENTITY_CONFIG.mobs`（内联设置）；
- `EntitySystemManager.EXTRA_DROPS`（按实体 ID 分组的额外掉落规则）。

### 4.1 读取与设置 MobConfig

- `@Nullable EntityConfig.MobConfig getMobConfig(String entityId)`  
  获取某实体类型的配置（如 `minecraft:zombie`）。

- `void setMobConfig(String entityId, EntityConfig.MobConfig config)`  
  为某实体类型设置或覆盖 MobConfig。你可以手动构造一个 `MobConfig`：

  - `enabled`：是否启用实体机制；
  - `health_multiplier` / `armor_multiplier` / `damage_multiplier`：等级对属性的倍率；
  - `max_affixes`：该实体最多拥有的词条数；
  - `allowed_affixes` / `blocked_affixes`：白名单 / 黑名单；
  - `extra_drops`：内联额外掉落规则；
  - `min_level` / `max_level`：该实体等级最小 / 最大值约束。

通过 API 设置的 MobConfig 会影响后续新刷出的实体在 `EntityEventHandler` 与 `AffixHelper` 中的：

- 等级计算（最小 / 最大等级约束）；
- 属性加成倍数；
- 词条生成的数量和候选集；
- 内联掉落规则。

### 4.2 读取与追加额外掉落规则

- `List<EntityConfig.ExtraDropRule> getExtraDropRules(String entityId)`  
  返回该实体当前综合的掉落规则：

  - `MobConfig.extra_drops` 中配置的规则；
  - `entity_system/drops/*.json` 加载到 `EXTRA_DROPS` 的规则。

- `void addExtraDropRule(String entityId, EntityConfig.ExtraDropRule rule, boolean asInline)`  
  为某实体追加一个额外掉落规则：

  - `asInline = true`：追加到 `MobConfig.extra_drops` 中，等价于在 `entity_settings` 中内联配置；
  - `asInline = false`：追加到 `EXTRA_DROPS` 中，等价于在 `entity_system/drops/*.json` 中新增规则。

`ExtraDropRule` 支持：

- 按整体概率 `chance` 控制触发；
- 按实体等级区间 `entity_level.min/max` 限制；
- 按实体词条 / 词条等级条件 `required_affixes` 限制；
- 掉落多个物品 `items`（支持 min/max 数量和单独概率）；
- 添加额外经验 `extra_experience`；
- 执行一系列命令 `commands`（支持 console / position / killer 等执行模式）。

---

## 5. 示例：为僵尸追加一个自定义实体词条与掉落

> 以下为伪代码示意，请根据你的模组包名/事件注册方式自行调整。

### 5.0 通过数据包注册实体词条与掉落（可选）

如果你更习惯用 **数据包** 扩展实体机制，可以通过在数据包中添加 `entity_system` 目录下的 JSON：

1. 在 `data/yourmod/entity_system/affix_config.json` 中新增实体词条定义：

```jsonc
{
  "affixes": {
    "yourmod:tag_shadow": {
      "min_level": 1,
      "base_chance": 0.1,
      "growth_per_level": 0.02,
      "max_level": 5,
      "params": {
        "extra_damage_per_level": 0.05
      },
      "conflicts": []
    }
  }
}
```

2. 在 `data/yourmod/entity_system/entity_settings.json` 中为僵尸开启实体机制并允许该词条：

```jsonc
{
  "mobs": {
    "minecraft:zombie": {
      "enabled": true,
      "health_multiplier": 0.1,
      "armor_multiplier": 0.05,
      "damage_multiplier": 0.05,
      "max_affixes": 3,
      "allowed_affixes": [
        "yourmod:tag_shadow"
      ],
      "blocked_affixes": []
    }
  },
  "excluded_mobs": []
}
```

3. 在 `data/yourmod/entity_system/drops/zombie_shadow.json` 中为满足条件的僵尸追加额外掉落：

```jsonc
{
  "entity_id": "minecraft:zombie",
  "extra_drops": [
    {
      "chance": 0.5,
      "entity_level": { "min": 10 },
      "required_affixes": [
        {
          "affix_id": "yourmod:tag_shadow",
          "min_level": 2
        }
      ],
      "items": [
        {
          "item": "minecraft:diamond",
          "min_count": 1,
          "max_count": 2,
          "chance": 1.0
        }
      ]
    }
  ]
}
```

加载数据包后（`/reload` 或重启世界）：

- 实体系统会通过 `EntitySystemManager` 读取这些 JSON 并加入内存配置；
- 实体生成时由 `AffixHelper.generateAffixes` 决定是否给僵尸赋予 `yourmod:tag_shadow`；
- 掉落时由 `EntityDropHandler` 根据 `extra_drops` 规则判断是否额外掉落钻石。

如果你需要在 Java 端动态读 / 改这些配置，可以结合本文件前面介绍的 `MechanicsAPI` 接口（例如 `getEntityAffixDefinition`、`getMobConfig`、`getExtraDropRules` 等）。

### 5.1 注册一个实体词条定义 `yourmod:tag_shadow`

```java
public class YourMod {

    public static final String MODID = "your_modid";

    public YourMod() {
        net.minecraftforge.fml.javafmlmod.FMLJavaModLoadingContext.get().getModEventBus()
            .addListener(this::onCommonSetup);
    }

    private void onCommonSetup(final net.minecraftforge.fml.event.lifecycle.FMLCommonSetupEvent event) {
        event.enqueueWork(() -> {
            MechanicsAPI.registerEntityAffix(
                    "yourmod:tag_shadow",
                    1,      // minLevel
                    0.1,    // baseChance
                    0.02,   // growthPerLevel
                    5,      // maxLevel
                    null,   // params，可选
                    null    // conflicts，可选
            );
        });
    }
}
```

### 5.2 在攻击事件中实现 `yourmod:tag_shadow` 的效果

```java
@Mod.EventBusSubscriber(modid = "your_modid")
public class ShadowAffixHandler {

    private static final String SHADOW_AFFIX = "yourmod:tag_shadow";

    @SubscribeEvent
    public static void onLivingHurt(LivingHurtEvent event) {
        if (!(event.getSource().getEntity() instanceof LivingEntity attacker)) {
            return;
        }

        int level = MechanicsAPI.getEntityAffixLevel(attacker, SHADOW_AFFIX);
        if (level <= 0) {
            return; // 攻击者没有该词条
        }

        LivingEntity target = event.getEntity();
        if (target.level().isClientSide()) {
            return;
        }

        float base = event.getAmount();
        float extra = base * (0.05f * level); // 每级 +5% 伤害
        event.setAmount(base + extra);

        // 额外：施加短暂失明
        target.addEffect(new MobEffectInstance(MobEffects.DARKNESS, 40 + level * 10, 0));
    }
}
```

> 要点：
> - 词条生成由 `AffixHelper.generateAffixes` 决定是否赋予实体该 ID；
> - 你的代码只需要在事件中通过 `MechanicsAPI.getEntityAffixLevel` 判断并实现行为逻辑。

### 5.3 为僵尸追加一个额外掉落规则

```java
public class ZombieDropConfig {

    public static void registerExtraDrops() {
        EntityConfig.ExtraDropRule rule = new EntityConfig.ExtraDropRule();
        rule.chance = 0.5; // 50% 概率触发

        // 等级条件：实体等级 >= 10
        EntityConfig.LevelCondition lc = new EntityConfig.LevelCondition();
        lc.min = 10;
        rule.entity_level = lc;

        // 掉落一到两颗钻石
        EntityConfig.ExtraDropItem item = new EntityConfig.ExtraDropItem();
        item.item = "minecraft:diamond";
        item.min_count = 1;
        item.max_count = 2;
        item.chance = 1.0; // 规则触发后必定掉落

        rule.items = java.util.List.of(item);

        MechanicsAPI.addExtraDropRule("minecraft:zombie", rule, true); // 追加为内联配置
    }
}
```

---

## 6. 配置重载与调试

- `void reloadEntitySystemConfigs(MinecraftServer server)`  
  手动重载 `entity_system` 下的所有 JSON 配置。  
  一般推荐使用 Minecraft 原生的资源重载机制（例如 `/reload`），本方法主要给管理工具或测试使用。

调试建议：

1. 使用 `/reload` 或重启世界，确保外部数据包 / 配置被重新加载；  
2. 在服务器日志中观察 `EntitySystemManager` 输出的“Loaded Entity System Configs”统计信息；  
3. 在你的调试代码中打印 `MechanicsAPI.getEntityLevel`、`getEntityAffixLevels` 等，确认实体等级与词条是否按预期生成；  
4. 配合 `EntityDropHandler` 的日志或实际游戏掉落，验证额外掉落规则是否生效。
