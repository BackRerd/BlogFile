---
title: 职业系统API(cbs)
tags:
  - pfc
  - ca
  - mc
  - mod
categories:
  - pfc
  - ca
abbrlink: 3719d53
date: 2025-12-12 18:12:45
---
# ClassBioArsenal 职业系统二次开发说明

本文档介绍如何基于 ClassBioArsenal 模组的职业系统进行二次开发（以 **Java API** 为主），包括：

- 使用 **Java API** 在其它模组中操作玩家职业

职业本身依然由 JSON / 数据包驱动加载，这里不展开数据包教程，仅关注 Java 端调用方式。

> 说明：以下内容基于 Forge 1.20.1。

---

## 1. Java ProfessionAPI

对外入口类：

- 包名：`site.backrer.classbioarsenal.api`
- 类名：`ProfessionAPI`

只要在运行时依赖 ClassBioArsenal 的 jar，就可以在你的模组代码里直接调用这些静态方法。

### 1.1 基础查询

- `boolean hasProfession(Player player)`  
  检查玩家是否已经拥有某个职业。

- `@Nullable String getProfessionId(Player player)`  
  获取玩家当前职业 ID（即 JSON 中的 `name` 字段），无职业时返回 `null`。

- `@Nullable Profession getProfession(Player player)`  
  获取玩家当前职业模板，仅包含配置数据，不包含玩家等级/经验。

- `@Nullable ProfessionHelper.PlayerProfessionInfo getProfessionInfo(Player player)`  
  获取包含模板 + 等级 + 经验的综合信息对象；无职业时其中 `profession` 为 `null`，等级/经验为 0。

- `int getLevel(Player player)`  
  获取玩家职业等级，无职业返回 0。

- `int getCurrentExp(Player player)`  
  获取当前职业经验，无职业返回 0。

- `int getMaxExp(Player player)`  
  获取当前等级升级所需最大经验，无职业返回 0。

- `float getExpProgress(Player player)`  
  获取经验进度（0.0–1.0）。

> 建议：这些方法应在 **服务器端逻辑** 调用；客户端调用只适用于纯显示用途。

### 1.2 玩家职业操作

- `boolean setProfession(Player player, String professionId)`  
  为玩家设置职业。内部会：

  - 校验职业 ID 是否存在
  - 重置该玩家的职业等级/经验
  - 发放职业初始装备
  - 执行首选职业时的指令（firstSelectCommands）
  - 应用职业属性加成

  返回值：设置成功（职业存在）返回 `true`，否则返回 `false`。

- `void clearProfession(Player player)`  
  清除玩家职业，重置为“无职业”状态，并移除所有职业属性加成。

- `boolean addProfessionExperience(Player player, int amount)`  
  为玩家增加职业经验，内部会自动处理：

  - 经验累加
  - 循环升级（可多级）
  - 刷新职业属性加成
  - 触发升级指令（levelCommands）

  返回值：本次调用中如果玩家至少升级一次，则返回 `true`。

- `void refreshProfessionAttributes(Player player)`  
  根据当前职业与等级，重新应用玩家的职业属性加成。  
  一般仅在你手动修改了经验/等级后，希望立即刷新属性时使用。

### 1.3 职业配置访问

- `@Nullable Profession getProfessionById(String id)`  
  通过职业 ID 获取职业模板。

- `Map<String, Profession> getAllProfessions()`  
  获取所有已注册职业的不可变 Map 副本（key 为职业 ID）。

- `Collection<String> getAllProfessionIds()`  
  获取所有职业 ID。

- `List<Profession> getNormalProfessions()`  
  获取所有基础职业（一级职业）。

- `List<Profession> getAdvancedProfessions(String baseProfessionId)`  
  获取指定基础职业的所有进阶职业。

- `void reloadProfessions(MinecraftServer server)`  
  通过服务器资源管理器手动重载职业配置。  
  一般推荐使用 `/profession reload professions` 指令，仅在你做服务端管理工具时会用到此方法。

> 提示：职业/物品限制的具体数值仍然来自数据包 JSON，本节只介绍 Java 访问入口。

---

## 2. 示例：某职业拥有常驻药水和额外 10% 伤害

本节给出一个完整示例：

- 当玩家拥有某个职业（例如 `berserker` 狂战士）时：
  - 持续获得一个常驻药水效果（例如 `力量`）；
  - 攻击其它生物时，额外造成 **10% 伤害**。

### 2.1 事件处理类示例（伪代码）

以下为一个事件处理类示例，展示如何只通过 `ProfessionAPI` 与 Forge 事件实现上述效果：

```java
@Mod.EventBusSubscriber(modid = "your_modid")
public class ExampleProfessionHooks {

    private static final String TARGET_PROFESSION_ID = "berserker"; // 目标职业 ID

    /**
     * 玩家拥有指定职业时，保持一个常驻药水效果。
     */
    @SubscribeEvent
    public static void onPlayerTick(TickEvent.PlayerTickEvent event) {
        if (event.phase != TickEvent.Phase.END) {
            return;
        }

        Player player = event.player;
        if (player.level().isClientSide()) {
            return; // 只在服务端处理逻辑
        }

        String professionId = ProfessionAPI.getProfessionId(player);
        if (!TARGET_PROFESSION_ID.equals(professionId)) {
            return; // 只对指定职业生效
        }

        // 例：给予力量药水效果（MobEffects.DAMAGE_BOOST），
        // 持续时间设置得稍长一些，每 tick 检查一次，可实现近似常驻效果。
        int minDuration = 200; // 剩余时间少于该值则重新施加
        var effect = player.getEffect(MobEffects.DAMAGE_BOOST);
        if (effect == null || effect.getDuration() < minDuration) {
            player.addEffect(new MobEffectInstance(
                    MobEffects.DAMAGE_BOOST,
                    220,           // 持续时间（tick）
                    0,             // 药水等级（0 = I）
                    true,          // ambient
                    false          // 是否显示粒子，可按需调整
            ));
        }
    }

    /**
     * 拥有指定职业的玩家对其他实体造成额外 10% 伤害。
     */
    @SubscribeEvent
    public static void onLivingHurt(LivingHurtEvent event) {
        if (!(event.getSource().getEntity() instanceof Player player)) {
            return;
        }

        if (player.level().isClientSide()) {
            return; // 只在服务端修改伤害
        }

        String professionId = ProfessionAPI.getProfessionId(player);
        if (!TARGET_PROFESSION_ID.equals(professionId)) {
            return; // 只对指定职业生效
        }

        float base = event.getAmount();
        float extra = base * 0.1F; // 10% 额外伤害
        event.setAmount(base + extra);
    }
}
```

要点说明：

- **不直接访问能力 / NBT**，统一通过 `ProfessionAPI.getProfessionId` 判断职业；
- 常驻药水效果通过定期检查并重新施加实现；
- 伤害加成为在 `LivingHurtEvent` 中修改 `event.setAmount` 完成；
- 你可以把 `TARGET_PROFESSION_ID` 换成任何在数据中存在的职业 ID。

### 2.2 适配到你自己的模组

- 把 `your_modid` 替换成你的模组 ID；
- 根据需要选择合适的药水效果、持续时间和等级；
- 也可以根据职业等级调整倍率，例如：
  - 使用 `ProfessionAPI.getLevel(player)` 获取等级；
  - 将伤害倍率改为 `1.0F + 0.1F * level` 等等。

---

## 3. 热加载与调试

- 使用指令 `/profession reload professions` 可在不重启服务器的情况下重载职业配置。
- 使用指令 `/profession reload items` 可重载物品职业配置。
- 使用指令 `/profession list` / `/profession info <id>` 可以查看当前已注册职业及其属性。

建议在开发/调试时：

1. 打开日志，关注职业加载相关输出与错误。  
2. 使用 `/profession reload professions` 快速验证 JSON 修改是否生效。  
3. 使用 `/profession player set <id>` 和 `/profession player addxp <amount>` 进行实际游戏内测试。
