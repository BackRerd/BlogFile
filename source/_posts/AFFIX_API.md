---
title: 装备/武器API(cbs)
tags:
  - pfc
  - ca
  - mc
  - mod
categories:
  - pfc
  - ca
abbrlink: 433c116d
date: 2025-12-12 18:12:45
---
# ClassBioArsenal 武器 / 护甲词条系统 API 说明

本文档介绍如何针对 ClassBioArsenal 的 **武器 / 护甲词条系统** 进行二次开发，包括：

- 使用 Java 端 `AffixAPI` 查询装备当前词条信息；
- 基于 JSON 新增词条，并通过代码实现词条的“详细效果”；
- 在 Java 中动态注册新的词条定义。

> 说明：以下内容基于 Forge 1.20.1。

---

## 1. Java AffixAPI

对外入口类：

- 包名：`site.backrer.classbioarsenal.api`
- 类名：`AffixAPI`

只要在运行时依赖 ClassBioArsenal 的 jar，就可以在你的模组中直接调用这些静态方法。

### 1.1 基础：判断是否参与词条系统

- `boolean isWeapon(ItemStack stack)`  
  判断该物品是否被视为“武器”（由 `weapon_config.json` 和默认规则决定）。

- `boolean isArmor(ItemStack stack)`  
  判断该物品是否被视为“护甲”（由 `armor_config.json` 和默认规则决定）。

### 1.2 读取装备上的词条

- `@Nullable WeaponStats getWeaponStats(ItemStack stack)`  
  返回武器内部的词条数据对象；不是有效武器或尚未初始化时返回 `null`。

- `@Nullable ArmorStats getArmorStats(ItemStack stack)`  
  返回护甲内部的词条数据对象；不是有效护甲或尚未初始化时返回 `null`。

- `Map<String, Integer> getWeaponAffixLevels(ItemStack stack)`  
  获取武器所有词条的 `(id -> level)` 映射；无词条或无效武器时返回空 Map。

- `Map<String, Integer> getArmorAffixLevels(ItemStack stack)`  
  获取护甲所有词条的 `(id -> level)` 映射。

- `int getWeaponAffixLevel(ItemStack stack, String affixId)`  
  获取指定武器词条的等级，不存在则返回 `0`。

- `int getArmorAffixLevel(ItemStack stack, String affixId)`  
  获取指定护甲词条的等级，不存在则返回 `0`。

- `double getWeaponAffixValue(ItemStack stack, String affixId)`  
  按当前等级和配置，计算某武器词条的“实际数值”（与模组内部战斗公式一致）。

- `double getArmorAffixValue(ItemStack stack, String affixId)`  
  计算某护甲词条的“实际数值”。

### 1.3 等级与经验 API

- `int getWeaponLevel(ItemStack stack)`  
  获取武器等级；未初始化或非有效武器时返回 0。

- `int getWeaponXp(ItemStack stack)`  
  获取武器当前经验；未初始化或非有效武器时返回 0。

- `int getWeaponXpToNextLevel(ItemStack stack)`  
  获取武器距离下一级所需经验值；若已达最高等级或配置缺失则返回 0。

- `boolean addWeaponXp(ItemStack stack, Player player, int amount)`  
  为武器增加经验值，内部会调用原有的 `WeaponEventHandler.checkLevelUp`：
  - 处理多级升级；
  - 触发词条解锁 / 升级逻辑；
  - 保存回 NBT。  
  若本次调用导致武器等级发生变化，则返回 `true`。

- `int getArmorLevel(ItemStack stack)`  
  获取护甲等级；未初始化或非有效护甲时返回 0。

- `int getArmorXp(ItemStack stack)`  
  获取护甲当前经验；未初始化或非有效护甲时返回 0。

- `int getArmorXpToNextLevel(ItemStack stack)`  
  获取护甲距离下一级所需经验值；若已达最高等级或配置缺失则返回 0。

- `boolean addArmorXp(ItemStack stack, Player player, int amount)`  
  为护甲增加经验值，内部会调用原有的 `ArmorEventHandler.checkLevelUp` 完成升级与词条解锁/升级，若等级变化则返回 `true`。

> 说明：这些等级 API 全部复用你现有的升级与词条解锁逻辑，只是提供了一个方便的外部入口，不会改变原有系统行为。

### 1.4 读取词条配置定义

- `@Nullable AffixConfig.AffixEntry getWeaponAffixDefinition(String affixId)`  
  从武器词条配置中读取指定 ID 的定义（字段与 `weapon_system/affixes.json` 一致）。

- `@Nullable AffixConfig.AffixEntry getArmorAffixDefinition(String affixId)`  
  从护甲词条配置中读取指定 ID 的定义。

> 提示：词条定义本身来自 JSON（可以被外部数据包覆盖），AffixAPI 只是提供一个统一的 Java 访问入口。

### 1.5 Java 端动态注册新词条

- `boolean registerWeaponAffix(AffixConfig.AffixEntry entry)`  
- `boolean registerArmorAffix(AffixConfig.AffixEntry entry)`  

  直接向当前词条配置中追加一条新的词条定义。ID 已存在或配置未加载时返回 `false`。

- `boolean registerWeaponAffix(String id, String displayName, String description, String type, double minValue, double maxValue, int maxLevel, double growthPerLevel, String color, int weight, @Nullable Map<String, Double> params)`  
- `boolean registerArmorAffix(String id, String displayName, String description, String type, double minValue, double maxValue, int maxLevel, double growthPerLevel, String color, int weight, @Nullable Map<String, Double> params)`  

  使用字段参数构造并注册一个新的词条定义，便于在其它模组中直接调用。

> 建议：在 **配置加载完成之后**（例如 CommonSetup 阶段）调用注册方法，避免在配置为 null 时注册失败。

动态注册的词条会被加入到：

- 武器：`ConfigManager.AFFIX_CONFIG.affixes`
- 护甲：`ConfigManager.ARMOR_AFFIX_CONFIG.affixes`

而武器 / 护甲初始化与升级逻辑中，正是从这两个列表中：

- 初始化时：在 `initializeWeaponIfNeeded` / `initializeArmorIfNeeded` 里，使用 `new ArrayList<>(...affixes)` 作为随机候选词条池；
- 升级时：在 `handleLevelUpBonuses` 中通过 `ConfigManager.AFFIX_CONFIG.affixes` / `ARMOR_AFFIX_CONFIG.affixes` 以及 `ConfigManager.getAffix(...)` 选择解锁或升级的词条。

因此：

- **之后新获得或升级的装备**，会把你注册的新词条视为正常候选；
- **已经生成且已 roll 好词条的旧装备**，不会自动重新洗词条，只会在今后的“升级解锁/升级词条”时有机会抽到新词条。

---

## 2. 通过 JSON 新增词条，再用 AffixAPI 实现详细效果

词条基础信息由 JSON 定义，详细效果由 Java 代码实现。整体流程：

1. 在数据包 / 模组资源中新增一个词条定义（JSON）。  
2. 在你的模组代码中，通过 `AffixAPI` 读取该词条的等级与数值，在对应事件中写具体效果。

### 2.1 在 weapon_system/affixes.json 中新增武器词条

假设我们想新增一个 **流血 (BLEED)** 词条，对被攻击目标施加持续掉血效果：

```jsonc
{
  "affixes": [
    // ... 原有配置 ...
    {
      "id": "BLEED",
      "display_name": "流血",
      "description": "攻击时使目标流血，持续掉血",
      "type": "PASSIVE_DOT",
      "min_value": 1.0,
      "max_value": 5.0,
      "max_level": 10,
      "growth_per_level": 0.4,
      "color": "§4",
      "weight": 30
    }
  ]
}
```

> 实际使用时建议通过 **外部数据包** 或复制原 JSON 后合并，而不是直接修改模组 jar 内部文件。

### 2.2 使用 AffixAPI 在攻击事件中实现流血效果

在你的模组中监听 `LivingHurtEvent`，检测当前武器是否拥有 `BLEED` 词条，并根据其数值施加持续伤害：

```java
@Mod.EventBusSubscriber(modid = "your_modid")
public class BleedAffixHandler {

    private static final String BLEED_AFFIX_ID = "BLEED";

    @SubscribeEvent
    public static void onLivingHurt(LivingHurtEvent event) {
        if (!(event.getSource().getEntity() instanceof Player player)) {
            return;
        }

        ItemStack weapon = player.getMainHandItem();
        if (!AffixAPI.isWeapon(weapon)) {
            return;
        }

        // 当前武器上 BLEED 的“数值”，例如 1.0 ~ 5.0（可按等级成长）
        double bleedValue = AffixAPI.getWeaponAffixValue(weapon, BLEED_AFFIX_ID);
        if (bleedValue <= 0.0) {
            return; // 没有该词条或数值为 0
        }

        LivingEntity target = event.getEntity();
        if (target.level().isClientSide()) {
            return; // 只在服务端实现效果
        }

        // 简单示例：根据数值决定流血持续时间和每跳伤害
        int duration = (int) (40 + bleedValue * 10); // 2s+ 持续
        int amplifier = 0;

        // 这里为了示例，复用凋零效果代表“流血”，你也可以实现自定义效果
        target.addEffect(new MobEffectInstance(MobEffects.WITHER, duration, amplifier));
    }
}
```

要点：

- 词条是否存在、等级和最终数值，统一通过 `AffixAPI` 读取；
- JSON 只负责“数值配置”和“抽取权重”，具体怎么用这些数值由你的代码决定；
- 可以通过 `AffixAPI.getWeaponAffixLevel(weapon, BLEED_AFFIX_ID)` 再根据等级做额外逻辑。

---

## 3. 在 Java 中注册新的词条（不改 JSON）

如果你希望纯粹通过 Java 追加词条定义（例如只在某个服务器环境中临时增加一个词条），可以使用注册 API：

### 3.1 在 CommonSetup 阶段注册武器词条

```java
public class YourMod {

    public static final String MODID = "your_modid";

    public YourMod() {
        // 在 mod 初始化中注册 CommonSetup 回调
        net.minecraftforge.fml.javafmlmod.FMLJavaModLoadingContext.get().getModEventBus()
            .addListener(this::onCommonSetup);
    }

    private void onCommonSetup(final net.minecraftforge.fml.event.lifecycle.FMLCommonSetupEvent event) {
        event.enqueueWork(() -> {
            // 注册一个简单的“额外伤害百分比”词条 EXTRA_DAMAGE_PERCENT
            AffixAPI.registerWeaponAffix(
                    "EXTRA_DAMAGE_PERCENT",   // id
                    "额外伤害",                 // displayName
                    "提高攻击伤害百分比",       // description
                    "STAT_PERCENT",           // type，自行约定或复用现有类型
                    5.0,                        // minValue
                    25.0,                       // maxValue
                    10,                         // maxLevel
                    2.0,                        // growthPerLevel
                    "§c",                      // color
                    50,                         // weight 抽取权重
                    null                        // 额外参数（可为 null）
            );
        });
    }
}
```

之后，你可以在自己的事件中使用：

```java
double extraPercent = AffixAPI.getWeaponAffixValue(weapon, "EXTRA_DAMAGE_PERCENT");
if (extraPercent > 0) {
    float base = event.getAmount();
    float extra = base * (float) (extraPercent / 100.0);
    event.setAmount(base + extra);
}
```

> 注意：如果同一个词条 ID 既在 JSON 中存在、又在 Java 里注册，第二次注册会失败（返回 false），以避免冲突。

---

## 4. 调试与重载

- 词条配置来自：
  - 武器：`data/classbioarsenal/weapon_system/affixes.json`
  - 护甲：`data/classbioarsenal/armor_system/armor_affixes.json`
- 这些文件可以被外部数据包覆盖，`ConfigManager` 会优先加载外部数据包中的版本。

推荐的调试方式：

1. 使用指令 `/reload` 或重启世界以确保外部数据包生效；  
2. 使用自带调试手段（例如在事件里 `System.out.println(AffixAPI.getWeaponAffixLevels(stack))`）查看当前装备的词条；  
3. 调整 JSON / 注册代码后，再次重载并在游戏内实战测试效果。
