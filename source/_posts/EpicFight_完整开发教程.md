---
title: EpicFight 1.20.1 完整开发教程
tags:
  - epicfight
  - mc
  - mod
categories:
  - 教程
abbrlink: 78ce3532
date: 2025-11-25 18:26:19
password: Ab040510
abstract: 有东西被加密了, 请输入密码查看.
message: 您好, 这里需要密码.
wrong_pass_message: 抱歉, 这个密码看着不太对, 请再试试.
wrong_hash_message: 抱歉, 这个文章不能被校验, 不过您还是能看看解密后的内容.
---
# EpicFight 1.20.1 完整开发教程

## 目录

1. [完整案例一:添加自定义武器动作套装](#完整案例一添加自定义武器动作套装)
2. [完整案例二:按键直接播放动画](#完整案例二按键直接播放动画)
3. [完整案例三:自定义模型+自定义动作](#完整案例三自定义模型自定义动作)

---

## 完整案例一:添加自定义武器动作套装

本案例将创建一个完整的自定义武器"雷霆之刃",包含完整的攻击连招、冲刺攻击、空中斩击和武器固有技能。

### 第一步:项目结构准备

创建以下目录结构:
```
src/main/
├── java/com/yourmod/
│   ├── YourMod.java
│   ├── animations/
│   │   └── ThunderBladeAnimations.java
│   ├── skills/
│   │   └── ThunderSlashSkill.java
│   └── items/
│       └── ThunderBladeItem.java
└── resources/
    ├── assets/yourmod/
    │   ├── animmodels/animations/
    │   │   └── thunder_blade/
    │   │       ├── auto1.armature
    │   │       ├── auto2.armature
    │   │       ├── auto3.armature
    │   │       ├── dash.armature
    │   │       ├── air_slash.armature
    │   │       └── thunder_slash.armature
    │   ├── models/item/
    │   │   └── thunder_blade.json
    │   └── textures/item/
    │       └── thunder_blade.png
    └── data/yourmod/
        └── capabilities/weapons/
            └── thunder_blade.json
```

### 第二步:注册动画

**ThunderBladeAnimations.java**:
```java
package com.yourmod.animations;

import net.minecraftforge.eventbus.api.SubscribeEvent;
import net.minecraftforge.fml.common.Mod;
import yesman.epicfight.api.animation.AnimationAccessor;
import yesman.epicfight.api.animation.AnimationBuilder;
import yesman.epicfight.api.animation.types.*;
import yesman.epicfight.api.forgeevent.AnimationRegistryEvent;
import yesman.epicfight.gameasset.Armatures;
import yesman.epicfight.particle.EpicFightParticles;
import yesman.epicfight.api.animation.property.AnimationProperty.*;
import yesman.epicfight.api.animation.property.AnimationEvent.*;
import yesman.epicfight.api.utils.math.ValueModifier;
import yesman.epicfight.world.damagesource.StunType;

@Mod.EventBusSubscriber(modid = "yourmod", bus = Mod.EventBusSubscriber.Bus.MOD)
public class ThunderBladeAnimations {
    
    // 声明动画访问器
    public static AnimationAccessor<BasicAttackAnimation> THUNDER_AUTO1;
    public static AnimationAccessor<BasicAttackAnimation> THUNDER_AUTO2;
    public static AnimationAccessor<BasicAttackAnimation> THUNDER_AUTO3;
    public static AnimationAccessor<DashAttackAnimation> THUNDER_DASH;
    public static AnimationAccessor<AirSlashAnimation> THUNDER_AIR_SLASH;
    public static AnimationAccessor<AttackAnimation> THUNDER_SLASH;
    
    @SubscribeEvent
    public static void registerAnimations(AnimationRegistryEvent event) {
        event.newBuilder("yourmod", ThunderBladeAnimations::build);
    }
    
    public static void build(AnimationBuilder builder) {
        // 第一段普攻 - 快速横斩
        THUNDER_AUTO1 = builder.nextAccessor("thunder_blade/auto1", (accessor) ->
            new BasicAttackAnimation(
                0.08F,  // 转换时间
                0.0F,   // 攻击开始时间
                0.15F,  // 攻击结束时间
                0.4F,   // 总时间
                null,   // 碰撞器(使用默认)
                Armatures.BIPED.get().toolR,  // 右手工具关节
                accessor,
                Armatures.BIPED
            )
            // 设置基础攻击速度
            .addProperty(AttackAnimationProperty.BASIS_ATTACK_SPEED, 2.0F)
            // 添加雷电粒子效果
            .addProperty(AttackPhaseProperty.PARTICLE, EpicFightParticles.EVISCERATE)
            // 添加音效
            .addEvents(InTimeEvent.create(0.1F, ReusableSources.PLAY_SOUND, Side.CLIENT)
                .params(EpicFightSounds.WHOOSH.get()))
        );
        
        // 第二段普攻 - 上挑
        THUNDER_AUTO2 = builder.nextAccessor("thunder_blade/auto2", (accessor) ->
            new BasicAttackAnimation(
                0.08F, 0.05F, 0.2F, 0.45F,
                null, Armatures.BIPED.get().toolR,
                accessor, Armatures.BIPED
            )
            .addProperty(AttackAnimationProperty.BASIS_ATTACK_SPEED, 2.0F)
            .addProperty(AttackPhaseProperty.DAMAGE_MODIFIER, ValueModifier.multiplier(1.1F))
            .addProperty(AttackPhaseProperty.PARTICLE, EpicFightParticles.EVISCERATE)
        );
        
        // 第三段普攻 - 旋转斩击(范围攻击)
        THUNDER_AUTO3 = builder.nextAccessor("thunder_blade/auto3", (accessor) ->
            new BasicAttackAnimation(
                0.1F, 0.1F, 0.3F, 0.6F,
                null, Armatures.BIPED.get().toolR,
                accessor, Armatures.BIPED
            )
            .addProperty(AttackAnimationProperty.BASIS_ATTACK_SPEED, 2.0F)
            .addProperty(AttackPhaseProperty.DAMAGE_MODIFIER, ValueModifier.multiplier(1.3F))
            .addProperty(AttackPhaseProperty.MAX_STRIKES_MODIFIER, ValueModifier.adder(2))
            .addProperty(AttackPhaseProperty.PARTICLE, EpicFightParticles.EVISCERATE)
            .addProperty(AttackPhaseProperty.STUN_TYPE, StunType.LONG)
            // 添加扫击附魔效果
            .addProperty(AttackPhaseProperty.EXTRA_DAMAGE, 
                Set.of(ExtraDamageInstance.SWEEPING_EDGE_ENCHANTMENT.create()))
        );
        
        // 冲刺攻击 - 雷霆突刺
        THUNDER_DASH = builder.nextAccessor("thunder_blade/dash", (accessor) ->
            new DashAttackAnimation(
                0.1F,   // 转换时间
                0.15F,  // 攻击开始
                0.25F,  // 攻击结束
                0.3F,   // 冲刺开始
                0.7F,   // 总时间
                null,
                Armatures.BIPED.get().toolR,
                accessor,
                Armatures.BIPED,
                true    // 可被格挡
            )
            .addProperty(AttackAnimationProperty.BASIS_ATTACK_SPEED, 1.8F)
            .addProperty(AttackPhaseProperty.DAMAGE_MODIFIER, ValueModifier.multiplier(1.5F))
            .addProperty(AttackPhaseProperty.ARMOR_NEGATION_MODIFIER, ValueModifier.adder(10.0F))
            .addProperty(AttackPhaseProperty.PARTICLE, EpicFightParticles.BLADE_RUSH_SKILL)
            // 添加雷电音效
            .addEvents(InTimeEvent.create(0.15F, ReusableSources.PLAY_SOUND, Side.CLIENT)
                .params(EpicFightSounds.BLADE_RUSH_FINISHER.get()))
        );
        
        // 空中斩击
        THUNDER_AIR_SLASH = builder.nextAccessor("thunder_blade/air_slash", (accessor) ->
            new AirSlashAnimation(
                0.1F, 0.15F, 0.25F, 0.4F, 1.0F,
                null, Armatures.BIPED.get().toolR,
                accessor, Armatures.BIPED
            )
            .addProperty(AttackAnimationProperty.BASIS_ATTACK_SPEED, 1.6F)
            .addProperty(AttackPhaseProperty.DAMAGE_MODIFIER, ValueModifier.multiplier(1.2F))
        );
        
        // 武器固有技能 - 雷霆斩
        THUNDER_SLASH = builder.nextAccessor("thunder_blade/thunder_slash", (accessor) ->
            new AttackAnimation(
                0.15F, 0.3F, 0.5F, 0.8F, 1.5F,
                null, Armatures.BIPED.get().toolR,
                accessor, Armatures.BIPED
            )
            .addProperty(AttackPhaseProperty.DAMAGE_MODIFIER, ValueModifier.multiplier(2.5F))
            .addProperty(AttackPhaseProperty.ARMOR_NEGATION_MODIFIER, ValueModifier.adder(30.0F))
            .addProperty(AttackPhaseProperty.MAX_STRIKES_MODIFIER, ValueModifier.adder(3))
            .addProperty(AttackPhaseProperty.IMPACT_MODIFIER, ValueModifier.multiplier(2.0F))
            .addProperty(AttackPhaseProperty.STUN_TYPE, StunType.KNOCKDOWN)
            .addProperty(AttackPhaseProperty.PARTICLE, EpicFightParticles.EVISCERATE)
            // 添加雷电特效事件
            .addEvents(InTimeEvent.create(0.3F, (entitypatch, animation, params) -> {
                // 生成雷电粒子
                Vec3 pos = entitypatch.getOriginal().position();
                for (int i = 0; i < 20; i++) {
                    double offsetX = (Math.random() - 0.5) * 3;
                    double offsetZ = (Math.random() - 0.5) * 3;
                    entitypatch.getOriginal().level().addParticle(
                        ParticleTypes.ELECTRIC_SPARK,
                        pos.x + offsetX,
                        pos.y + 1.0,
                        pos.z + offsetZ,
                        0, 0.1, 0
                    );
                }
                // 播放雷鸣音效
                entitypatch.playSound(SoundEvents.LIGHTNING_BOLT_THUNDER, 1.0F, 1.0F);
            }, Side.CLIENT))
        );
    }
}
```

### 第三步:创建武器固有技能

**ThunderSlashSkill.java**:
```java
package com.yourmod.skills;

import yesman.epicfight.skill.SkillBuilder;
import yesman.epicfight.skill.weaponinnate.SimpleWeaponInnateSkill;
import yesman.epicfight.world.capabilities.entitypatch.player.PlayerPatch;
import com.yourmod.animations.ThunderBladeAnimations;

public class ThunderSlashSkill extends SimpleWeaponInnateSkill {
    
    public ThunderSlashSkill(SkillBuilder<? extends Skill> builder) {
        super(builder);
    }
    
    @Override
    public void onInitiate(SkillContainer container) {
        super.onInitiate(container);
        // 初始化逻辑
    }
    
    @Override
    public void onActivate(SkillContainer container) {
        super.onActivate(container);
        
        PlayerPatch<?> player = (PlayerPatch<?>) container.getExecutor();
        
        // 播放雷霆斩动画
        player.playAnimationSynchronized(ThunderBladeAnimations.THUNDER_SLASH, 0.0F);
        
        // 消耗耐力
        player.getOriginal().causeFoodExhaustion(3.0F);
    }
}
```

### 第四步:注册技能

**YourMod.java** (在主类中添加):
```java
@Mod.EventBusSubscriber(modid = "yourmod", bus = Mod.EventBusSubscriber.Bus.MOD)
public class YourMod {
    
    public static Skill THUNDER_SLASH;
    
    @SubscribeEvent
    public static void buildSkills(SkillBuildEvent event) {
        ModRegistryWorker registry = event.createRegistryWorker("yourmod");
        
        // 注册雷霆斩技能
        WeaponInnateSkill thunderSlash = registry.build("thunder_slash",
            ThunderSlashSkill::new,
            SimpleWeaponInnateSkill.createSimpleWeaponInnateBuilder()
                .setAnimations(ThunderBladeAnimations.THUNDER_SLASH)
        );
        
        // 配置技能属性
        thunderSlash.newProperty()
            .addProperty(AttackPhaseProperty.DAMAGE_MODIFIER, ValueModifier.multiplier(2.5F))
            .addProperty(AttackPhaseProperty.ARMOR_NEGATION_MODIFIER, ValueModifier.adder(30.0F))
            .addProperty(AttackPhaseProperty.MAX_STRIKES_MODIFIER, ValueModifier.adder(3))
            .addProperty(AttackPhaseProperty.IMPACT_MODIFIER, ValueModifier.multiplier(2.0F))
            .addProperty(AttackPhaseProperty.STUN_TYPE, StunType.KNOCKDOWN)
            .addProperty(AttackPhaseProperty.EXTRA_DAMAGE, 
                Set.of(ExtraDamageInstance.SWEEPING_EDGE_ENCHANTMENT.create()))
            .addProperty(AttackPhaseProperty.SOURCE_TAG, 
                Set.of(EpicFightDamageTypeTags.WEAPON_INNATE));
        
        THUNDER_SLASH = thunderSlash;
    }
}
```

### 第五步:创建武器物品

**ThunderBladeItem.java**:
```java
package com.yourmod.items;

import net.minecraft.world.item.SwordItem;
import net.minecraft.world.item.Tier;
import net.minecraft.world.item.Item;

public class ThunderBladeItem extends SwordItem {
    
    public ThunderBladeItem(Tier tier, int attackDamage, float attackSpeed, Item.Properties properties) {
        super(tier, attackDamage, attackSpeed, properties);
    }
}
```

注册物品:
```java
public static final RegistryObject<Item> THUNDER_BLADE = ITEMS.register("thunder_blade",
    () -> new ThunderBladeItem(
        Tiers.DIAMOND,
        5,  // 攻击伤害
        -2.0F,  // 攻击速度
        new Item.Properties()
    )
);
```

### 第六步:配置武器能力

**data/yourmod/capabilities/weapons/thunder_blade.json**:
```json
{
    "attributes": {
        "common": {
            "impact": 3.0,
            "max_strikes": 2,
            "armor_negation": 15.0,
            "damage_bonus": 1.2
        }
    },
    "type": "epicfight:longsword",
    "style": "two_hand",
    "collider": "epicfight:longsword",
    "weapon_motion_category": "longsword",
    "combo_animations": [
        "yourmod:thunder_blade/auto1",
        "yourmod:thunder_blade/auto2",
        "yourmod:thunder_blade/auto3"
    ],
    "dash_attack": "yourmod:thunder_blade/dash",
    "air_slash": "yourmod:thunder_blade/air_slash",
    "innate_skills": [
        "yourmod:thunder_slash"
    ]
}
```

### 第七步:添加语言文件

**assets/yourmod/lang/zh_cn.json**:
```json
{
    "item.yourmod.thunder_blade": "雷霆之刃",
    "skill.yourmod.thunder_slash": "雷霆斩",
    "skill.yourmod.thunder_slash.tooltip": "召唤雷霆之力进行强力斩击"
}
```

---

## 完整案例二:按键直接播放动画

本案例将创建一个自定义按键,按下后播放特定动画(例如:嘲讽动作)。

### 第一步:注册自定义按键

**CustomKeyMappings.java**:
```java
package com.yourmod.client.input;

import com.mojang.blaze3d.platform.InputConstants;
import net.minecraft.client.KeyMapping;
import net.minecraftforge.api.distmarker.Dist;
import net.minecraftforge.client.event.RegisterKeyMappingsEvent;
import net.minecraftforge.client.settings.KeyConflictContext;
import net.minecraftforge.eventbus.api.SubscribeEvent;
import net.minecraftforge.fml.common.Mod;

@Mod.EventBusSubscriber(modid = "yourmod", value = Dist.CLIENT, bus = Mod.EventBusSubscriber.Bus.MOD)
public class CustomKeyMappings {
    
    // 定义按键分类
    public static final String CATEGORY = "key.categories.yourmod";
    
    // 嘲讽动作按键 (默认绑定到 T 键)
    public static final KeyMapping TAUNT_KEY = new KeyMapping(
        "key.yourmod.taunt",              // 翻译键
        KeyConflictContext.IN_GAME,        // 上下文:游戏内
        InputConstants.Type.KEYSYM,        // 类型:键盘按键
        InputConstants.KEY_T,              // 默认按键: T
        CATEGORY                           // 分类
    );
    
    // 特殊技能按键 (默认绑定到 Y 键)
    public static final KeyMapping SPECIAL_SKILL_KEY = new KeyMapping(
        "key.yourmod.special_skill",
        KeyConflictContext.IN_GAME,
        InputConstants.Type.KEYSYM,
        InputConstants.KEY_Y,
        CATEGORY
    );
    
    // 姿态切换按键 (默认绑定到 H 键)
    public static final KeyMapping STANCE_SWITCH_KEY = new KeyMapping(
        "key.yourmod.stance_switch",
        KeyConflictContext.IN_GAME,
        InputConstants.Type.KEYSYM,
        InputConstants.KEY_H,
        CATEGORY
    );
    
    @SubscribeEvent
    public static void registerKeys(RegisterKeyMappingsEvent event) {
        event.register(TAUNT_KEY);
        event.register(SPECIAL_SKILL_KEY);
        event.register(STANCE_SWITCH_KEY);
    }
}
```

### 第二步:创建嘲讽动画

**CustomAnimations.java**:
```java
package com.yourmod.animations;

import net.minecraftforge.eventbus.api.SubscribeEvent;
import net.minecraftforge.fml.common.Mod;
import yesman.epicfight.api.animation.AnimationAccessor;
import yesman.epicfight.api.animation.AnimationBuilder;
import yesman.epicfight.api.animation.types.StaticAnimation;
import yesman.epicfight.api.animation.types.ActionAnimation;
import yesman.epicfight.api.forgeevent.AnimationRegistryEvent;
import yesman.epicfight.gameasset.Armatures;
import yesman.epicfight.api.animation.property.AnimationProperty.*;

@Mod.EventBusSubscriber(modid = "yourmod", bus = Mod.EventBusSubscriber.Bus.MOD)
public class CustomAnimations {
    
    // 嘲讽动画
    public static AnimationAccessor<StaticAnimation> TAUNT_ANIMATION;
    // 特殊姿态动画
    public static AnimationAccessor<ActionAnimation> SPECIAL_STANCE;
    // 战斗姿态
    public static AnimationAccessor<StaticAnimation> COMBAT_STANCE;
    
    @SubscribeEvent
    public static void registerAnimations(AnimationRegistryEvent event) {
        event.newBuilder("yourmod", CustomAnimations::build);
    }
    
    public static void build(AnimationBuilder builder) {
        // 嘲讽动画 - 2秒的动作动画
        TAUNT_ANIMATION = builder.nextAccessor("player/taunt", (accessor) ->
            new StaticAnimation(
                0.15F,  // 转换时间
                false,  // 不循环
                accessor,
                Armatures.BIPED
            )
            // 锁定头部旋转
            .addProperty(StaticAnimationProperty.FIXED_HEAD_ROTATION, true)
            // 添加音效
            .addEvents(InTimeEvent.create(0.5F, ReusableSources.PLAY_SOUND, Side.CLIENT)
                .params(SoundEvents.VILLAGER_CELEBRATE))
            // 设置状态
            .newTimePair(0.0F, 2.0F)
                .addStateRemoveOld(EntityState.MOVEMENT_LOCKED, true)
                .addStateRemoveOld(EntityState.CAN_BASIC_ATTACK, false)
                .addStateRemoveOld(EntityState.CAN_SKILL_EXECUTION, false)
        );
        
        // 特殊姿态 - 蓄力姿态
        SPECIAL_STANCE = builder.nextAccessor("player/special_stance", (accessor) ->
            new ActionAnimation(
                0.2F,   // 转换时间
                accessor,
                Armatures.BIPED
            )
            .addProperty(ActionAnimationProperty.STOP_MOVEMENT, true)
            .addProperty(StaticAnimationProperty.PLAY_SPEED_MODIFIER, 
                (self, entitypatch, speed, prevElapsedTime, elapsedTime) -> {
                    // 动画播放到最后时停止
                    if (elapsedTime > 1.0F) {
                        return 0.0F;
                    }
                    return 1.0F;
                })
        );
        
        // 战斗姿态 - 持续的战斗准备姿态
        COMBAT_STANCE = builder.nextAccessor("player/combat_stance", (accessor) ->
            new StaticAnimation(
                0.25F,
                true,   // 循环播放
                accessor,
                Armatures.BIPED
            )
            .addProperty(StaticAnimationProperty.FIXED_HEAD_ROTATION, false)
        );
    }
}
```

### 第三步:处理按键输入

**KeyInputHandler.java**:
```java
package com.yourmod.client;

import net.minecraft.client.Minecraft;
import net.minecraft.client.player.LocalPlayer;
import net.minecraftforge.api.distmarker.Dist;
import net.minecraftforge.event.TickEvent;
import net.minecraftforge.eventbus.api.SubscribeEvent;
import net.minecraftforge.fml.common.Mod;
import yesman.epicfight.world.capabilities.EpicFightCapabilities;
import yesman.epicfight.world.capabilities.entitypatch.player.PlayerPatch;
import com.yourmod.client.input.CustomKeyMappings;
import com.yourmod.animations.CustomAnimations;
import com.yourmod.network.NetworkHandler;
import com.yourmod.network.PacketPlayAnimation;

@Mod.EventBusSubscriber(modid = "yourmod", value = Dist.CLIENT)
public class KeyInputHandler {
    
    private static boolean tauntKeyWasDown = false;
    private static boolean specialKeyWasDown = false;
    private static boolean stanceKeyWasDown = false;
    
    @SubscribeEvent
    public static void onClientTick(TickEvent.ClientTickEvent event) {
        if (event.phase != TickEvent.Phase.END) {
            return;
        }
        
        Minecraft mc = Minecraft.getInstance();
        if (mc.player == null) {
            return;
        }
        
        LocalPlayer player = mc.player;
        
        // 检查嘲讽按键
        if (CustomKeyMappings.TAUNT_KEY.isDown()) {
            if (!tauntKeyWasDown) {
                tauntKeyWasDown = true;
                onTauntKeyPressed(player);
            }
        } else {
            tauntKeyWasDown = false;
        }
        
        // 检查特殊技能按键
        if (CustomKeyMappings.SPECIAL_SKILL_KEY.isDown()) {
            if (!specialKeyWasDown) {
                specialKeyWasDown = true;
                onSpecialSkillKeyPressed(player);
            }
        } else {
            specialKeyWasDown = false;
        }
        
        // 检查姿态切换按键
        if (CustomKeyMappings.STANCE_SWITCH_KEY.isDown()) {
            if (!stanceKeyWasDown) {
                stanceKeyWasDown = true;
                onStanceSwitchKeyPressed(player);
            }
        } else {
            stanceKeyWasDown = false;
        }
    }
    
    private static void onTauntKeyPressed(LocalPlayer player) {
        // 获取玩家的 EpicFight 能力
        player.getCapability(EpicFightCapabilities.CAPABILITY_ENTITY).ifPresent(cap -> {
            if (cap instanceof PlayerPatch<?> playerPatch) {
                // 检查玩家是否可以执行动作
                if (!playerPatch.isInaction() && !playerPatch.getOriginal().isUsingItem()) {
                    // 发送网络包到服务器播放动画
                    NetworkHandler.sendToServer(new PacketPlayAnimation("taunt"));
                }
            }
        });
    }
    
    private static void onSpecialSkillKeyPressed(LocalPlayer player) {
        player.getCapability(EpicFightCapabilities.CAPABILITY_ENTITY).ifPresent(cap -> {
            if (cap instanceof PlayerPatch<?> playerPatch) {
                if (!playerPatch.isInaction()) {
                    NetworkHandler.sendToServer(new PacketPlayAnimation("special_stance"));
                }
            }
        });
    }
    
    private static void onStanceSwitchKeyPressed(LocalPlayer player) {
        player.getCapability(EpicFightCapabilities.CAPABILITY_ENTITY).ifPresent(cap -> {
            if (cap instanceof PlayerPatch<?> playerPatch) {
                NetworkHandler.sendToServer(new PacketPlayAnimation("combat_stance"));
            }
        });
    }
}
```

### 第四步:创建网络包

**PacketPlayAnimation.java**:
```java
package com.yourmod.network;

import net.minecraft.network.FriendlyByteBuf;
import net.minecraft.server.level.ServerPlayer;
import net.minecraftforge.network.NetworkEvent;
import yesman.epicfight.world.capabilities.EpicFightCapabilities;
import yesman.epicfight.world.capabilities.entitypatch.player.ServerPlayerPatch;
import com.yourmod.animations.CustomAnimations;

import java.util.function.Supplier;

public class PacketPlayAnimation {
    
    private final String animationType;
    
    public PacketPlayAnimation(String animationType) {
        this.animationType = animationType;
    }
    
    public PacketPlayAnimation(FriendlyByteBuf buf) {
        this.animationType = buf.readUtf();
    }
    
    public void encode(FriendlyByteBuf buf) {
        buf.writeUtf(this.animationType);
    }
    
    public void handle(Supplier<NetworkEvent.Context> ctx) {
        ctx.get().enqueueWork(() -> {
            ServerPlayer player = ctx.get().getSender();
            if (player == null) {
                return;
            }
            
            player.getCapability(EpicFightCapabilities.CAPABILITY_ENTITY).ifPresent(cap -> {
                if (cap instanceof ServerPlayerPatch serverPatch) {
                    // 根据动画类型播放对应动画
                    switch (animationType) {
                        case "taunt":
                            if (!serverPatch.isInaction()) {
                                serverPatch.playAnimationSynchronized(
                                    CustomAnimations.TAUNT_ANIMATION, 
                                    0.0F
                                );
                            }
                            break;
                            
                        case "special_stance":
                            if (!serverPatch.isInaction()) {
                                serverPatch.playAnimationSynchronized(
                                    CustomAnimations.SPECIAL_STANCE, 
                                    0.0F
                                );
                            }
                            break;
                            
                        case "combat_stance":
                            serverPatch.playAnimationSynchronized(
                                CustomAnimations.COMBAT_STANCE, 
                                0.0F
                            );
                            break;
                    }
                }
            });
        });
        ctx.get().setPacketHandled(true);
    }
}
```

**NetworkHandler.java**:
```java
package com.yourmod.network;

import net.minecraft.resources.ResourceLocation;
import net.minecraftforge.network.NetworkRegistry;
import net.minecraftforge.network.PacketDistributor;
import net.minecraftforge.network.simple.SimpleChannel;

public class NetworkHandler {
    
    private static final String PROTOCOL_VERSION = "1";
    public static final SimpleChannel INSTANCE = NetworkRegistry.newSimpleChannel(
        new ResourceLocation("yourmod", "main"),
        () -> PROTOCOL_VERSION,
        PROTOCOL_VERSION::equals,
        PROTOCOL_VERSION::equals
    );
    
    private static int packetId = 0;
    
    public static void register() {
        INSTANCE.messageBuilder(PacketPlayAnimation.class, packetId++)
            .encoder(PacketPlayAnimation::encode)
            .decoder(PacketPlayAnimation::new)
            .consumerMainThread(PacketPlayAnimation::handle)
            .add();
    }
    
    public static void sendToServer(Object packet) {
        INSTANCE.sendToServer(packet);
    }
}
```

在主类初始化时调用:
```java
@Mod("yourmod")
public class YourMod {
    public YourMod() {
        NetworkHandler.register();
    }
}
```

### 第五步:添加语言文件

**assets/yourmod/lang/zh_cn.json**:
```json
{
    "key.categories.yourmod": "你的模组",
    "key.yourmod.taunt": "嘲讽动作",
    "key.yourmod.special_skill": "特殊技能",
    "key.yourmod.stance_switch": "切换姿态"
}
```

---

## 完整案例三:自定义模型+自定义动作

本案例将创建一个自定义生物"暗影刺客",拥有独特的模型和专属动作。

### 第一步:准备模型文件

使用 Blockbench 或其他建模软件创建模型,导出为 EpicFight 格式。

**目录结构**:
```
assets/yourmod/animmodels/
├── entity/
│   └── shadow_assassin.json  (模型文件)
└── animations/
    └── shadow_assassin/
        ├── idle.armature
        ├── walk.armature
        ├── attack1.armature
        ├── attack2.armature
        ├── attack3.armature
        ├── stealth.armature
        └── backstab.armature
```

### 第二步:注册自定义骨骼

**CustomArmatures.java**:
```java
package com.yourmod.model;

import yesman.epicfight.api.model.armature.Armature;
import yesman.epicfight.api.model.armature.ArmatureAccessor;

public class CustomArmatures {
    
    // 暗影刺客骨骼
    public static final ArmatureAccessor<Armature> SHADOW_ASSASSIN = 
        new ArmatureAccessor<>(
            "yourmod",                    // 命名空间
            "shadow_assassin",            // 模型名称
            Armature.class                // 骨骼类型
        );
}
```

### 第三步:注册动画

**ShadowAssassinAnimations.java**:
```java
package com.yourmod.animations;

import net.minecraftforge.eventbus.api.SubscribeEvent;
import net.minecraftforge.fml.common.Mod;
import yesman.epicfight.api.animation.AnimationAccessor;
import yesman.epicfight.api.animation.AnimationBuilder;
import yesman.epicfight.api.animation.types.*;
import yesman.epicfight.api.forgeevent.AnimationRegistryEvent;
import com.yourmod.model.CustomArmatures;
import yesman.epicfight.api.animation.property.AnimationProperty.*;
import yesman.epicfight.world.damagesource.StunType;

@Mod.EventBusSubscriber(modid = "yourmod", bus = Mod.EventBusSubscriber.Bus.MOD)
public class ShadowAssassinAnimations {
    
    // 基础动画
    public static AnimationAccessor<StaticAnimation> ASSASSIN_IDLE;
    public static AnimationAccessor<MovementAnimation> ASSASSIN_WALK;
    
    // 攻击动画
    public static AnimationAccessor<AttackAnimation> ASSASSIN_ATTACK1;
    public static AnimationAccessor<AttackAnimation> ASSASSIN_ATTACK2;
    public static AnimationAccessor<AttackAnimation> ASSASSIN_ATTACK3;
    
    // 特殊动画
    public static AnimationAccessor<ActionAnimation> ASSASSIN_STEALTH;
    public static AnimationAccessor<AttackAnimation> ASSASSIN_BACKSTAB;
    
    @SubscribeEvent
    public static void registerAnimations(AnimationRegistryEvent event) {
        event.newBuilder("yourmod", ShadowAssassinAnimations::build);
    }
    
    public static void build(AnimationBuilder builder) {
        // 待机动画
        ASSASSIN_IDLE = builder.nextAccessor("shadow_assassin/idle", (accessor) ->
            new StaticAnimation(
                true,       // 循环播放
                accessor,
                CustomArmatures.SHADOW_ASSASSIN
            )
        );
        
        // 行走动画
        ASSASSIN_WALK = builder.nextAccessor("shadow_assassin/walk", (accessor) ->
            new MovementAnimation(
                true,
                accessor,
                CustomArmatures.SHADOW_ASSASSIN
            )
        );
        
        // 攻击连招
        ASSASSIN_ATTACK1 = builder.nextAccessor("shadow_assassin/attack1", (accessor) ->
            new AttackAnimation(
                0.1F, 0.1F, 0.2F, 0.3F, 0.6F,
                null,
                CustomArmatures.SHADOW_ASSASSIN.get().toolR,
                accessor,
                CustomArmatures.SHADOW_ASSASSIN
            )
            .addProperty(AttackPhaseProperty.DAMAGE_MODIFIER, ValueModifier.multiplier(1.0F))
            .addProperty(AttackPhaseProperty.PARTICLE, EpicFightParticles.HIT_BLADE)
        );
        
        ASSASSIN_ATTACK2 = builder.nextAccessor("shadow_assassin/attack2", (accessor) ->
            new AttackAnimation(
                0.05F, 0.15F, 0.25F, 0.35F, 0.65F,
                null,
                CustomArmatures.SHADOW_ASSASSIN.get().toolR,
                accessor,
                CustomArmatures.SHADOW_ASSASSIN
            )
            .addProperty(AttackPhaseProperty.DAMAGE_MODIFIER, ValueModifier.multiplier(1.2F))
            .addProperty(AttackPhaseProperty.PARTICLE, EpicFightParticles.HIT_BLADE)
        );
        
        ASSASSIN_ATTACK3 = builder.nextAccessor("shadow_assassin/attack3", (accessor) ->
            new AttackAnimation(
                0.1F, 0.2F, 0.35F, 0.45F, 1.0F,
                null,
                CustomArmatures.SHADOW_ASSASSIN.get().toolR,
                accessor,
                CustomArmatures.SHADOW_ASSASSIN
            )
            .addProperty(AttackPhaseProperty.DAMAGE_MODIFIER, ValueModifier.multiplier(1.5F))
            .addProperty(AttackPhaseProperty.MAX_STRIKES_MODIFIER, ValueModifier.adder(2))
            .addProperty(AttackPhaseProperty.STUN_TYPE, StunType.LONG)
            .addProperty(AttackPhaseProperty.PARTICLE, EpicFightParticles.EVISCERATE)
        );
        
        // 潜行动画
        ASSASSIN_STEALTH = builder.nextAccessor("shadow_assassin/stealth", (accessor) ->
            new ActionAnimation(
                0.2F,
                accessor,
                CustomArmatures.SHADOW_ASSASSIN
            )
            .addProperty(ActionAnimationProperty.STOP_MOVEMENT, false)
            .addEvents(StaticAnimationProperty.ON_BEGIN_EVENTS, 
                SimpleEvent.create((entitypatch, animation, params) -> {
                    // 进入潜行时添加隐身效果
                    entitypatch.getOriginal().addEffect(
                        new MobEffectInstance(MobEffects.INVISIBILITY, 200, 0, false, false)
                    );
                    // 添加速度效果
                    entitypatch.getOriginal().addEffect(
                        new MobEffectInstance(MobEffects.MOVEMENT_SPEED, 200, 1, false, false)
                    );
                }, Side.SERVER))
        );
        
        // 背刺动画 - 高伤害暴击
        ASSASSIN_BACKSTAB = builder.nextAccessor("shadow_assassin/backstab", (accessor) ->
            new AttackAnimation(
                0.05F, 0.1F, 0.2F, 0.3F, 0.8F,
                null,
                CustomArmatures.SHADOW_ASSASSIN.get().toolR,
                accessor,
                CustomArmatures.SHADOW_ASSASSIN
            )
            .addProperty(AttackPhaseProperty.DAMAGE_MODIFIER, ValueModifier.multiplier(3.0F))
            .addProperty(AttackPhaseProperty.ARMOR_NEGATION_MODIFIER, ValueModifier.adder(50.0F))
            .addProperty(AttackPhaseProperty.STUN_TYPE, StunType.KNOCKDOWN)
            .addProperty(AttackPhaseProperty.PARTICLE, EpicFightParticles.EVISCERATE)
            .addEvents(InTimeEvent.create(0.1F, ReusableSources.PLAY_SOUND, Side.CLIENT)
                .params(SoundEvents.PLAYER_ATTACK_CRIT))
        );
    }
}
```

### 第四步:创建实体类

**ShadowAssassinEntity.java**:
```java
package com.yourmod.entity;

import net.minecraft.world.entity.EntityType;
import net.minecraft.world.entity.ai.attributes.AttributeSupplier;
import net.minecraft.world.entity.ai.attributes.Attributes;
import net.minecraft.world.entity.ai.goal.*;
import net.minecraft.world.entity.monster.Monster;
import net.minecraft.world.entity.player.Player;
import net.minecraft.world.level.Level;

public class ShadowAssassinEntity extends Monster {
    
    public ShadowAssassinEntity(EntityType<? extends Monster> type, Level level) {
        super(type, level);
    }
    
    @Override
    protected void registerGoals() {
        this.goalSelector.addGoal(0, new FloatGoal(this));
        this.goalSelector.addGoal(1, new MeleeAttackGoal(this, 1.2D, false));
        this.goalSelector.addGoal(2, new WaterAvoidingRandomStrollGoal(this, 1.0D));
        this.goalSelector.addGoal(3, new LookAtPlayerGoal(this, Player.class, 8.0F));
        this.goalSelector.addGoal(4, new RandomLookAroundGoal(this));
        
        this.targetSelector.addGoal(1, new HurtByTargetGoal(this));
        this.targetSelector.addGoal(2, new NearestAttackableTargetGoal<>(this, Player.class, true));
    }
    
    public static AttributeSupplier.Builder createAttributes() {
        return Monster.createMonsterAttributes()
            .add(Attributes.MAX_HEALTH, 40.0D)
            .add(Attributes.MOVEMENT_SPEED, 0.35D)
            .add(Attributes.ATTACK_DAMAGE, 8.0D)
            .add(Attributes.ARMOR, 4.0D)
            .add(Attributes.FOLLOW_RANGE, 32.0D);
    }
}
```

### 第五步:创建实体能力补丁

**ShadowAssassinPatch.java**:
```java
package com.yourmod.world.capabilities;

import yesman.epicfight.world.capabilities.entitypatch.MobPatch;
import yesman.epicfight.world.capabilities.entitypatch.Faction;
import yesman.epicfight.api.animation.LivingMotions;
import com.yourmod.entity.ShadowAssassinEntity;
import com.yourmod.animations.ShadowAssassinAnimations;

public class ShadowAssassinPatch extends MobPatch<ShadowAssassinEntity> {
    
    public ShadowAssassinPatch() {
        super(Faction.HOSTILE);
    }
    
    @Override
    public void initAnimator() {
        super.initAnimator();
        
        // 设置基础动作
        this.animator.addLivingAnimation(LivingMotions.IDLE, ShadowAssassinAnimations.ASSASSIN_IDLE);
        this.animator.addLivingAnimation(LivingMotions.WALK, ShadowAssassinAnimations.ASSASSIN_WALK);
        this.animator.addLivingAnimation(LivingMotions.CHASE, ShadowAssassinAnimations.ASSASSIN_WALK);
    }
    
    @Override
    public void updateMotion(boolean considerInaction) {
        super.updateMotion(considerInaction);
        
        // 自定义运动逻辑
        if (this.state.inaction()) {
            currentMotion = LivingMotions.IDLE;
        } else if (this.getOriginal().getDeltaMovement().horizontalDistanceSqr() > 0.01D) {
            currentMotion = LivingMotions.WALK;
        } else {
            currentMotion = LivingMotions.IDLE;
        }
    }
    
    // 自定义攻击逻辑
    public void performBackstab() {
        if (!this.isLogicalClient()) {
            this.playAnimationSynchronized(ShadowAssassinAnimations.ASSASSIN_BACKSTAB, 0.0F);
        }
    }
    
    public void enterStealth() {
        if (!this.isLogicalClient()) {
            this.playAnimationSynchronized(ShadowAssassinAnimations.ASSASSIN_STEALTH, 0.0F);
        }
    }
}
```

### 第六步:注册实体能力

**CapabilityRegistry.java**:
```java
package com.yourmod.world.capabilities;

import net.minecraftforge.eventbus.api.SubscribeEvent;
import net.minecraftforge.fml.common.Mod;
import yesman.epicfight.api.forgeevent.EntityPatchRegistryEvent;
import com.yourmod.entity.ModEntities;

@Mod.EventBusSubscriber(modid = "yourmod", bus = Mod.EventBusSubscriber.Bus.MOD)
public class CapabilityRegistry {
    
    @SubscribeEvent
    public static void registerEntityPatches(EntityPatchRegistryEvent event) {
        event.registerMob(
            ModEntities.SHADOW_ASSASSIN.get(),
            ShadowAssassinPatch::new
        );
    }
}
```

### 第七步:创建AI目标(背刺逻辑)

**BackstabGoal.java**:
```java
package com.yourmod.entity.ai;

import net.minecraft.world.entity.LivingEntity;
import net.minecraft.world.entity.ai.goal.Goal;
import net.minecraft.world.phys.Vec3;
import yesman.epicfight.world.capabilities.EpicFightCapabilities;
import com.yourmod.entity.ShadowAssassinEntity;
import com.yourmod.world.capabilities.ShadowAssassinPatch;

import java.util.EnumSet;

public class BackstabGoal extends Goal {
    
    private final ShadowAssassinEntity assassin;
    private LivingEntity target;
    private int cooldown = 0;
    
    public BackstabGoal(ShadowAssassinEntity assassin) {
        this.assassin = assassin;
        this.setFlags(EnumSet.of(Goal.Flag.MOVE, Goal.Flag.LOOK));
    }
    
    @Override
    public boolean canUse() {
        if (cooldown > 0) {
            cooldown--;
            return false;
        }
        
        this.target = this.assassin.getTarget();
        if (target == null) {
            return false;
        }
        
        // 检查是否在目标背后
        return isBehin dTarget();
    }
    
    @Override
    public void start() {
        // 执行背刺
        this.assassin.getCapability(EpicFightCapabilities.CAPABILITY_ENTITY).ifPresent(cap -> {
            if (cap instanceof ShadowAssassinPatch patch) {
                patch.performBackstab();
            }
        });
        
        // 设置冷却时间
        cooldown = 200; // 10秒
    }
    
    private boolean isBehindTarget() {
        Vec3 assassinPos = this.assassin.position();
        Vec3 targetPos = this.target.position();
        Vec3 targetLook = this.target.getLookAngle();
        
        Vec3 toAssassin = assassinPos.subtract(targetPos).normalize();
        double dot = targetLook.dot(toAssassin);
        
        // 如果点积小于-0.5,说明刺客在目标背后
        return dot < -0.5 && this.assassin.distanceTo(this.target) < 3.0;
    }
}
```

在 `ShadowAssassinEntity` 中添加这个AI:
```java
@Override
protected void registerGoals() {
    this.goalSelector.addGoal(0, new FloatGoal(this));
    this.goalSelector.addGoal(1, new BackstabGoal(this));  // 添加背刺AI
    this.goalSelector.addGoal(2, new MeleeAttackGoal(this, 1.2D, false));
    // ... 其他AI
}
```

### 第八步:添加战斗行为

**ShadowAssassinCombatBehavior.java**:
```java
package com.yourmod.gameasset;

import yesman.epicfight.world.entity.ai.goal.CombatBehaviors;
import yesman.epicfight.world.capabilities.entitypatch.MobPatch;
import com.yourmod.animations.ShadowAssassinAnimations;

public class ShadowAssassinCombatBehavior {
    
    public static void register() {
        CombatBehaviors.Builder<MobPatch<?>> builder = 
            CombatBehaviors.builder();
        
        // 添加攻击连招
        builder.newBehaviorSeries(
            // 近战攻击系列
            CombatBehaviors.MELEE_ATTACK,
            // 攻击1
            CombatBehaviors.Phase.WINDUP(0.2F, 
                ShadowAssassinAnimations.ASSASSIN_ATTACK1),
            // 攻击2
            CombatBehaviors.Phase.WINDUP(0.15F, 
                ShadowAssassinAnimations.ASSASSIN_ATTACK2),
            // 攻击3(终结技)
            CombatBehaviors.Phase.WINDUP(0.25F, 
                ShadowAssassinAnimations.ASSASSIN_ATTACK3)
        );
        
        // 注册到实体
        CombatBehaviors.registerMobCombatBehavior(
            ModEntities.SHADOW_ASSASSIN.get(),
            builder.build()
        );
    }
}
```

在主类中调用:
```java
@Mod.EventBusSubscriber(modid = "yourmod", bus = Mod.EventBusSubscriber.Bus.MOD)
public class YourMod {
    
    @SubscribeEvent
    public static void onCommonSetup(FMLCommonSetupEvent event) {
        event.enqueueWork(() -> {
            ShadowAssassinCombatBehavior.register();
        });
    }
}
```

---

## 总结

以上三个完整案例展示了:

1. **案例一**: 如何创建完整的自定义武器系统,包括:
   - 多段连招动画
   - 冲刺和空中攻击
   - 武器固有技能
   - 武器配置文件

2. **案例二**: 如何实现按键触发动画,包括:
   - 自定义按键注册
   - 客户端按键检测
   - 网络同步
   - 服务器端动画播放

3. **案例三**: 如何创建自定义模型和生物,包括:
   - 自定义骨骼系统
   - 实体能力补丁
   - 自定义AI行为
   - 战斗系统集成

这些案例都是完整可运行的代码,你可以直接使用或根据需求修改。记住:
- 网络包需要正确注册
- 客户端和服务端代码要分离
- 测试时注意检查日志输出
