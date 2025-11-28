---
title: EpicFight 1.20.1 API 使用文档
tags:
  - epicfight
  - mc
  - mod
categories:
  - 教程
abbrlink: 3d053e04
date: 2025-11-25 18:26:19
password: Ab040510
abstract: 有东西被加密了, 请输入密码查看.
message: 您好, 这里需要密码.
wrong_pass_message: 抱歉, 这个密码看着不太对, 请再试试.
wrong_hash_message: 抱歉, 这个文章不能被校验, 不过您还是能看看解密后的内容.
---
# EpicFight 1.20.1 API 使用文档

## 项目概述

EpicFight 是一个 Minecraft Forge 1.20.1 模组,为游戏添加了动作战斗系统。该项目提供了完整的 API,允许开发者:
- 添加自定义动画
- 创建自定义模型和骨骼
- 添加新技能和技能书
- 配置武器能力

## 目录

1. [动画系统](#动画系统)
2. [模型系统](#模型系统)
3. [技能系统](#技能系统)
4. [武器能力配置](#武器能力配置)

---

## 动画系统

### 核心类

#### AnimationManager
- **位置**: `yesman.epicfight.api.animation.AnimationManager`
- **功能**: 管理所有动画的注册、加载和访问
- **关键方法**:
  - `getInstance()` - 获取单例实例
  - `byKey(ResourceLocation)` - 通过资源位置获取动画
  - `byId(int)` - 通过ID获取动画

#### StaticAnimation
- **位置**: `yesman.epicfight.api.animation.types.StaticAnimation`
- **功能**: 所有动画的基类
- **子类型**:
  - `MovementAnimation` - 移动动画
  - `AttackAnimation` - 攻击动画
  - `DodgeAnimation` - 闪避动画
  - `GuardAnimation` - 格挡动画
  - `ActionAnimation` - 动作动画

### 添加自定义动画

#### 步骤 1: 注册动画事件

```java
@Mod.EventBusSubscriber(modid = YourMod.MODID, bus = Bus.MOD)
public class YourAnimations {
    // 声明动画访问器
    public static AnimationAccessor<StaticAnimation> YOUR_ANIMATION;
    
    @SubscribeEvent
    public static void registerAnimations(AnimationRegistryEvent event) {
        event.newBuilder(YourMod.MODID, YourAnimations::build);
    }
    
    public static void build(AnimationBuilder builder) {
        YOUR_ANIMATION = builder.nextAccessor("path/to/animation", 
            (accessor) -> new StaticAnimation(true, accessor, Armatures.BIPED)
        );
    }
}
```

#### 步骤 2: 创建动画文件

动画文件位置: `src/main/resources/assets/yourmod/animmodels/animations/`

**示例: 基础攻击动画**
```java
SWORD_AUTO1 = builder.nextAccessor("biped/combat/sword_auto1", (accessor) ->
    new BasicAttackAnimation(
        0.1F,  // 转换时间
        0.0F,  // 攻击开始时间
        0.1F,  // 攻击结束时间
        0.4F,  // 总时间
        null,  // 碰撞器
        Armatures.BIPED.get().toolR,  // 关节
        accessor,
        Armatures.BIPED
    )
    .addProperty(AttackAnimationProperty.BASIS_ATTACK_SPEED, 1.6F)
);
```

#### 步骤 3: 添加动画属性

```java
YOUR_ANIMATION = builder.nextAccessor("your/animation", (accessor) ->
    new StaticAnimation(true, accessor, Armatures.BIPED)
        // 添加声音事件
        .addEvents(InTimeEvent.create(0.15F, ReusableSources.PLAY_SOUND, Side.CLIENT)
            .params(EpicFightSounds.SWORD_IN.get()))
        // 添加粒子效果
        .addProperty(AttackPhaseProperty.PARTICLE, EpicFightParticles.HIT_BLUNT)
        // 设置播放速度
        .addProperty(StaticAnimationProperty.PLAY_SPEED_MODIFIER, (self, entitypatch, speed, prevElapsedTime, elapsedTime) -> {
            return 1.0F; // 自定义速度逻辑
        })
);
```

### 动画数据文件格式

动画数据文件位置: `src/main/resources/assets/yourmod/animmodels/animations/path/to/data/animation.json`

```json
{
    "trail_effects": [
        {
            "start_time": 0.05,
            "end_time": 0.15,
            "joint": "Tool_R",
            "item_skin_hand": "main_hand"
        }
    ]
}
```

### 常用动画类型

#### BasicAttackAnimation - 基础攻击
```java
new BasicAttackAnimation(
    float convertTime,      // 转换时间
    float attackStart,      // 攻击开始时间
    float attackEnd,        // 攻击结束时间
    float totalTime,        // 总时间
    InteractionHand hand,   // 手部(可选)
    Collider collider,      // 碰撞器
    String joint,           // 关节名称
    AnimationAccessor accessor,
    Armature armature
)
```

#### DashAttackAnimation - 冲刺攻击
```java
new DashAttackAnimation(
    float convertTime,
    float attackStart,
    float attackEnd,
    float rushStart,        // 冲刺开始时间
    float totalTime,
    Collider collider,
    String joint,
    AnimationAccessor accessor,
    Armature armature,
    boolean canBeBlocked    // 是否可被格挡
)
```

#### DodgeAnimation - 闪避动画
```java
new DodgeAnimation(
    float convertTime,
    AnimationAccessor accessor,
    float iframeStart,      // 无敌帧开始
    float iframeEnd,        // 无敌帧结束
    Armature armature
)
```

---

## 模型系统

### 核心概念

#### Armature (骨骼)
骨骼定义了模型的骨骼结构和关节层次。

**位置**: `yesman.epicfight.api.model.armature.Armature`

#### 实体模型文件

模型文件位置: `src/main/resources/assets/yourmod/animmodels/entity/`

**biped.json 结构**:
```json
{
    "vertices": {
        "positions": { "stride": 3, "count": 260, "array": [...] },
        "uvs": { "stride": 2, "count": 321, "array": [...] },
        "normals": { "stride": 3, "count": 6, "array": [...] },
        "parts": {
            "rightArm": { "stride": 3, "count": 108, "array": [...] },
            "leftArm": { "stride": 3, "count": 108, "array": [...] },
            "head": { "stride": 3, "count": 36, "array": [...] },
            "torso": { "stride": 3, "count": 240, "array": [...] }
        }
    },
    "armature": {
        "joints": ["Root", "Thigh_R", "Leg_R", ...],
        "hierarchy": [...]
    }
}
```

### 添加自定义模型

#### 步骤 1: 创建模型文件

在 `assets/yourmod/animmodels/entity/` 创建 JSON 文件,定义顶点、UV、法线和骨骼层次。

#### 步骤 2: 注册骨骼

```java
public class YourArmatures {
    public static final ArmatureAccessor<Armature> YOUR_ARMATURE = 
        new ArmatureAccessor<>("yourmod", "your_model", Armature.class);
}
```

#### 步骤 3: 模型转换器

如果需要支持原版模型或其他模型格式:

```java
public class YourModelTransformer extends HumanoidModelTransformer {
    @Override
    public SkinnedMesh transformArmorModel(HumanoidModel<?> humanoidModel) {
        // 实现模型转换逻辑
    }
}
```

### 模型部件

常用模型部件名称:
- `head` - 头部
- `torso` - 躯干
- `chest` - 胸部
- `rightArm` / `leftArm` - 右臂/左臂
- `rightLeg` / `leftLeg` - 右腿/左腿
- `rightSleeve` / `leftSleeve` - 右袖/左袖
- `rightPants` / `leftPants` - 右裤/左裤

---

## 技能系统

### 核心类

#### Skill
- **位置**: `yesman.epicfight.skill.Skill`
- **功能**: 所有技能的基类

#### SkillBuilder
- **位置**: `yesman.epicfight.skill.SkillBuilder`
- **功能**: 构建技能的建造者模式类

### 技能类型

#### 激活类型 (ActivateType)
- `ONE_SHOT` - 单次激活
- `DURATION` - 持续时间
- `TOGGLE` - 切换开关
- `HELD` - 按住激活

#### 资源类型 (Resource)
- `NONE` - 无消耗
- `STAMINA` - 消耗耐力
- `COOLDOWN` - 冷却时间

#### 技能分类 (SkillCategory)
- `BASIC_ATTACK` - 基础攻击
- `DODGE` - 闪避
- `GUARD` - 格挡
- `PASSIVE` - 被动
- `WEAPON_INNATE` - 武器固有技能
- `IDENTITY` - 身份技能
- `MOVER` - 移动技能

### 添加自定义技能

#### 步骤 1: 创建技能类

```java
public class YourSkill extends Skill {
    public YourSkill(SkillBuilder<? extends Skill> builder) {
        super(builder);
    }
    
    @Override
    public void onInitiate(SkillContainer container) {
        super.onInitiate(container);
        // 初始化逻辑
    }
    
    @Override
    public void onUpdate(SkillContainer container) {
        super.onUpdate(container);
        // 更新逻辑
    }
    
    @Override
    public void onActivate(SkillContainer container) {
        // 激活时的逻辑
    }
}
```

#### 步骤 2: 注册技能

```java
@Mod.EventBusSubscriber(modid = YourMod.MODID, bus = Bus.MOD)
public class YourSkills {
    public static Skill YOUR_SKILL;
    
    @SubscribeEvent
    public static void buildSkillEvent(SkillBuildEvent event) {
        ModRegistryWorker registry = event.createRegistryWorker(YourMod.MODID);
        
        YOUR_SKILL = registry.build("your_skill", YourSkill::new,
            Skill.createBuilder()
                .setCategory(SkillCategories.WEAPON_INNATE)
                .setActivateType(ActivateType.ONE_SHOT)
                .setResource(Resource.STAMINA)
        );
    }
}
```

#### 步骤 3: 配置技能属性

```java
WeaponInnateSkill yourSkill = registry.build("your_skill", 
    SimpleWeaponInnateSkill::new, 
    SimpleWeaponInnateSkill.createSimpleWeaponInnateBuilder()
        .setAnimations(YOUR_ANIMATION)
);

yourSkill.newProperty()
    .addProperty(AttackPhaseProperty.DAMAGE_MODIFIER, ValueModifier.multiplier(2.0F))
    .addProperty(AttackPhaseProperty.ARMOR_NEGATION_MODIFIER, ValueModifier.adder(20.0F))
    .addProperty(AttackPhaseProperty.MAX_STRIKES_MODIFIER, ValueModifier.adder(1))
    .addProperty(AttackPhaseProperty.STUN_TYPE, StunType.LONG)
    .addProperty(AttackPhaseProperty.EXTRA_DAMAGE, 
        Set.of(ExtraDamageInstance.SWEEPING_EDGE_ENCHANTMENT.create()));
```

### 技能书系统

#### SkillBookItem
- **位置**: `yesman.epicfight.world.item.SkillBookItem`
- **功能**: 技能书物品

#### 创建技能书

```java
// 设置技能书包含的技能
ItemStack skillBook = new ItemStack(EpicFightItems.SKILLBOOK.get());
SkillBookItem.setContainingSkill(YOUR_SKILL, skillBook);

// 或者通过技能名称
SkillBookItem.setContainingSkill("yourmod:your_skill", skillBook);
```

#### 获取技能书中的技能

```java
Skill skill = SkillBookItem.getContainSkill(itemStack);
```

### 技能数据参数

技能参数配置位置: `src/main/resources/data/yourmod/skill_parameters/`

```json
{
    "max_level": 5,
    "cooldown": 10.0,
    "stamina_cost": 5.0,
    "damage_multiplier": 1.5
}
```

---

## 武器能力配置

### 武器能力文件

位置: `src/main/resources/data/yourmod/capabilities/weapons/`

#### 基础配置

**diamond_longsword.json**:
```json
{
    "attributes": {
        "common": {
            "impact": 2.5,
            "max_strikes": 2
        }
    },
    "type": "epicfight:longsword"
}
```

### 武器类型

常用武器类型:
- `epicfight:sword` - 单手剑
- `epicfight:longsword` - 长剑
- `epicfight:greatsword` - 大剑
- `epicfight:tachi` - 太刀
- `epicfight:uchigatana` - 打刀
- `epicfight:spear` - 长矛
- `epicfight:dagger` - 匕首
- `epicfight:axe` - 斧头
- `epicfight:fist` - 拳套

### 武器属性

#### 通用属性 (common)
- `impact` - 冲击力
- `max_strikes` - 最大连击数
- `armor_negation` - 护甲穿透
- `damage_bonus` - 伤害加成

#### 添加自定义武器配置

```json
{
    "attributes": {
        "common": {
            "impact": 3.0,
            "max_strikes": 3,
            "armor_negation": 10.0,
            "damage_bonus": 1.5
        }
    },
    "type": "epicfight:longsword",
    "style": "two_hand",
    "collider": "epicfight:longsword",
    "weapon_motion_category": "longsword"
}
```

### 关键字匹配

使用关键字自动应用配置到多个物品:

位置: `data/yourmod/capabilities/weapons/item_keyword/`

```json
{
    "items": [
        "yourmod:custom_sword_1",
        "yourmod:custom_sword_2"
    ],
    "attributes": {
        "common": {
            "impact": 2.5,
            "max_strikes": 2
        }
    },
    "type": "epicfight:sword"
}
```

---

## 完整示例

### 示例 1: 添加自定义攻击动画和技能

```java
// 1. 注册动画
@Mod.EventBusSubscriber(modid = "yourmod", bus = Bus.MOD)
public class ModAnimations {
    public static AnimationAccessor<AttackAnimation> CUSTOM_SLASH;
    
    @SubscribeEvent
    public static void registerAnimations(AnimationRegistryEvent event) {
        event.newBuilder("yourmod", ModAnimations::build);
    }
    
    public static void build(AnimationBuilder builder) {
        CUSTOM_SLASH = builder.nextAccessor("biped/combat/custom_slash", 
            (accessor) -> new BasicAttackAnimation(
                0.1F, 0.05F, 0.2F, 0.5F,
                null, Armatures.BIPED.get().toolR,
                accessor, Armatures.BIPED
            )
            .addProperty(AttackAnimationProperty.BASIS_ATTACK_SPEED, 1.8F)
            .addProperty(AttackPhaseProperty.DAMAGE_MODIFIER, ValueModifier.multiplier(1.5F))
            .addEvents(InTimeEvent.create(0.1F, ReusableSources.PLAY_SOUND, Side.CLIENT)
                .params(EpicFightSounds.WHOOSH.get()))
        );
    }
}

// 2. 创建技能
public class CustomSlashSkill extends SimpleWeaponInnateSkill {
    public CustomSlashSkill(SkillBuilder<? extends Skill> builder) {
        super(builder);
    }
}

// 3. 注册技能
@SubscribeEvent
public static void buildSkills(SkillBuildEvent event) {
    ModRegistryWorker registry = event.createRegistryWorker("yourmod");
    
    WeaponInnateSkill customSlash = registry.build("custom_slash",
        CustomSlashSkill::new,
        SimpleWeaponInnateSkill.createSimpleWeaponInnateBuilder()
            .setAnimations(ModAnimations.CUSTOM_SLASH)
    );
    
    customSlash.newProperty()
        .addProperty(AttackPhaseProperty.DAMAGE_MODIFIER, ValueModifier.multiplier(2.0F))
        .addProperty(AttackPhaseProperty.ARMOR_NEGATION_MODIFIER, ValueModifier.adder(15.0F))
        .addProperty(AttackPhaseProperty.STUN_TYPE, StunType.LONG);
}
```

### 示例 2: 配置自定义武器

**data/yourmod/capabilities/weapons/custom_blade.json**:
```json
{
    "attributes": {
        "common": {
            "impact": 3.5,
            "max_strikes": 3,
            "armor_negation": 20.0
        }
    },
    "type": "epicfight:longsword",
    "style": "two_hand",
    "collider": "epicfight:longsword",
    "innate_skills": [
        "yourmod:custom_slash"
    ]
}
```

---

## 调试和测试

### 动画调试

启用动画调试渲染:
```java
animation.renderDebugging(poseStack, buffer, entitypatch, playTime, partialTicks);
```

### 技能调试

使用命令测试技能:
```
/epicfight skill <player> <slot> <skill>
```

### 常见问题

#### 动画不播放
1. 检查动画文件路径是否正确
2. 确认骨骼类型匹配
3. 验证动画注册事件是否触发

#### 技能不生效
1. 检查技能分类是否正确
2. 确认技能容器已初始化
3. 验证激活条件是否满足

#### 武器配置无效
1. 检查 JSON 语法
2. 确认物品 ID 正确
3. 验证武器类型存在

---

## 资源链接

- **官方 Wiki**: [EpicFight Wiki](https://github.com/Yesssssman/epicfightmod)
- **示例模组**: 查看 EpicFight 源码中的 `gameasset` 包

---

## 版本信息

- **Minecraft**: 1.20.1
- **Forge**: 47.x+
- **EpicFight**: 20.x.x

---

## 许可证

本文档基于 EpicFight 项目,遵循其原始许可证。
