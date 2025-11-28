---
title: EpicFight 动画系统高级教程
tags:
  - epicfight
  - mc
  - mod
categories:
  - 教程
abbrlink: f5a02cd3
date: 2025-11-25 18:26:19
password: Ab040510
abstract: 有东西被加密了, 请输入密码查看.
message: 您好, 这里需要密码.
wrong_pass_message: 抱歉, 这个密码看着不太对, 请再试试.
wrong_hash_message: 抱歉, 这个文章不能被校验, 不过您还是能看看解密后的内容.
---
# EpicFight 动画系统高级教程

## 目录

1. [动画碰撞体系统](#动画碰撞体系统)
2. [动画事件系统](#动画事件系统)
3. [动画属性与自定义函数](#动画属性与自定义函数)
4. [自定义动画类](#自定义动画类)
5. [完整实战案例](#完整实战案例)

---

## 动画碰撞体系统

### 碰撞体类型

EpicFight 提供了多种碰撞体类型用于攻击判定:

#### 1. OBBCollider (定向包围盒)
最常用的碰撞体,适合剑类武器。

```java
// 创建一个 OBB 碰撞体
// 参数: 顶点坐标(x,y,z), 中心坐标(x,y,z)
OBBCollider collider = new OBBCollider(
    0.5, 0.0, 0.0,  // 顶点位置
    0.0, 0.0, 0.0   // 中心位置
);
```

#### 2. MultiOBBCollider (多重OBB碰撞体)
用于创建拖尾效果,适合快速挥砍动作。

```java
// 创建多重碰撞体
// 参数: 碰撞体数量, 顶点坐标, 中心坐标
MultiOBBCollider multiCollider = new MultiOBBCollider(
    5,              // 碰撞体数量
    0.5, 0.0, 0.0,  // 顶点
    0.0, 0.0, 0.0   // 中心
);
```

#### 3. LineCollider (线性碰撞体)
适合长矛、长枪等线性武器。

```java
// 创建线性碰撞体
// 参数: 起点坐标, 方向向量
LineCollider lineCollider = new LineCollider(
    0.0, 0.0, 0.0,  // 起点
    0.0, 0.0, 2.0   // 方向向量
);
```

#### 4. PlaneCollider (平面碰撞体)
适合范围攻击。

```java
// 创建平面碰撞体
PlaneCollider planeCollider = new PlaneCollider(
    0.0, 0.0, 0.0,  // 中心
    1.0, 0.0, 0.0,  // 第一个边向量
    0.0, 0.0, 1.0   // 第二个边向量
);
```

### 在动画中使用碰撞体

#### 方法一:使用预设碰撞体

```java
// EpicFight 提供了预设的碰撞体
public class ColliderPreset {
    // 拔刀术碰撞体 - 大范围前方扇形
    public static final Collider BATTOJUTSU = new MultiOBBCollider(
        8, 
        0.5, 0.0, 0.0,
        0.0, 0.0, -2.0
    );
    
    // 长剑碰撞体
    public static final Collider LONGSWORD = new OBBCollider(
        0.6, 0.0, 0.0,
        0.0, 0.0, 0.0
    );
}

// 在动画中使用
CUSTOM_ATTACK = builder.nextAccessor("custom/attack", (accessor) ->
    new BasicAttackAnimation(
        0.1F, 0.0F, 0.2F, 0.5F,
        ColliderPreset.BATTOJUTSU,  // 使用预设碰撞体
        Armatures.BIPED.get().toolR,
        accessor,
        Armatures.BIPED
    )
);
```

#### 方法二:创建自定义碰撞体

```java
// 创建自定义碰撞体类
public class CustomColliders {
    // 巨剑碰撞体 - 更大范围
    public static final Collider GREATSWORD_HEAVY = new MultiOBBCollider(
        10,             // 更多碰撞体数量
        0.8, 0.0, 0.0,  // 更大的顶点
        0.0, 0.0, -1.5  // 中心偏移
    );
    
    // 旋风斩碰撞体 - 360度范围
    public static final Collider WHIRLWIND = new PlaneCollider(
        0.0, 1.0, 0.0,  // 中心在腰部
        2.0, 0.0, 0.0,  // X轴方向
        0.0, 0.0, 2.0   // Z轴方向
    );
    
    // 突刺碰撞体 - 长距离线性
    public static final Collider THRUST = new LineCollider(
        0.0, 1.0, 0.0,  // 起点
        0.0, 0.0, 3.0   // 3格距离
    );
}

// 使用自定义碰撞体
WHIRLWIND_ATTACK = builder.nextAccessor("custom/whirlwind", (accessor) ->
    new AttackAnimation(
        0.15F, 0.2F, 0.5F, 0.7F, 1.2F,
        CustomColliders.WHIRLWIND,  // 使用自定义碰撞体
        Armatures.BIPED.get().torso,  // 绑定到躯干
        accessor,
        Armatures.BIPED
    )
    .addProperty(AttackPhaseProperty.MAX_STRIKES_MODIFIER, ValueModifier.adder(5))
    .addProperty(AttackPhaseProperty.DAMAGE_MODIFIER, ValueModifier.multiplier(0.8F))
);
```

#### 方法三:多阶段不同碰撞体

```java
// 使用 Phase 系统为不同阶段设置不同碰撞体
COMBO_ATTACK = builder.nextAccessor("custom/combo", (accessor) ->
    new AttackAnimation(
        0.1F,
        accessor,
        Armatures.BIPED,
        // 第一段 - 快速横斩
        new Phase(
            0.0F,           // 开始时间
            0.1F,           // 攻击开始
            0.2F,           // 攻击结束
            0.3F,           // 接触开始
            0.5F,           // 接触结束
            Float.MAX_VALUE,
            InteractionHand.MAIN_HAND,
            AttackAnimation.JointColliderPair.of(
                Armatures.BIPED.get().toolR,
                new OBBCollider(0.5, 0.0, 0.0, 0.0, 0.0, 0.0)  // 小碰撞体
            )
        ),
        // 第二段 - 重击
        new Phase(
            0.5F,
            0.6F,
            0.8F,
            0.9F,
            1.2F,
            Float.MAX_VALUE,
            InteractionHand.MAIN_HAND,
            AttackAnimation.JointColliderPair.of(
                Armatures.BIPED.get().toolR,
                new MultiOBBCollider(8, 0.8, 0.0, 0.0, 0.0, 0.0, -1.0)  // 大碰撞体
            )
        )
    )
);
```

### 碰撞体调试

```java
// 启用碰撞体可视化调试
// 在游戏中按 F3 + B 可以看到碰撞体
ATTACK_ANIMATION = builder.nextAccessor("debug/attack", (accessor) ->
    new BasicAttackAnimation(
        0.1F, 0.0F, 0.2F, 0.5F,
        new OBBCollider(0.6, 0.0, 0.0, 0.0, 0.0, 0.0),
        Armatures.BIPED.get().toolR,
        accessor,
        Armatures.BIPED
    )
);

// 碰撞体会在攻击时显示为红色框
```

---

## 动画事件系统

### 事件类型

#### 1. InTimeEvent - 在特定时间点触发

```java
// 在动画的 0.3 秒时播放音效
ATTACK_WITH_SOUND = builder.nextAccessor("attack/sound", (accessor) ->
    new BasicAttackAnimation(
        0.1F, 0.2F, 0.3F, 0.6F,
        null, Armatures.BIPED.get().toolR,
        accessor, Armatures.BIPED
    )
    // 在 0.3 秒时播放挥砍音效
    .addEvents(InTimeEvent.create(
        0.3F,                               // 时间点
        ReusableSources.PLAY_SOUND,         // 事件类型
        Side.CLIENT                         // 执行端
    ).params(EpicFightSounds.WHOOSH.get()))  // 参数
);
```

#### 2. InPeriodEvent - 在时间段内持续触发

```java
// 在 0.2-0.5 秒期间持续生成粒子
ATTACK_WITH_PARTICLES = builder.nextAccessor("attack/particles", (accessor) ->
    new AttackAnimation(
        0.1F, 0.2F, 0.5F, 0.6F, 0.8F,
        null, Armatures.BIPED.get().toolR,
        accessor, Armatures.BIPED
    )
    // 持续生成粒子
    .addEvents(InPeriodEvent.create(
        0.2F,                           // 开始时间
        0.5F,                           // 结束时间
        (entitypatch, animation, params) -> {
            // 每帧执行
            Vec3 pos = entitypatch.getOriginal().position();
            entitypatch.getOriginal().level().addParticle(
                ParticleTypes.FLAME,
                pos.x, pos.y + 1.0, pos.z,
                0, 0.1, 0
            );
        },
        Side.CLIENT
    ))
);
```

#### 3. SimpleEvent - 动画开始/结束时触发

```java
// 动画开始和结束时的事件
BUFF_ATTACK = builder.nextAccessor("attack/buff", (accessor) ->
    new AttackAnimation(
        0.1F, 0.2F, 0.4F, 0.5F, 0.8F,
        null, Armatures.BIPED.get().toolR,
        accessor, Armatures.BIPED
    )
    // 动画开始时添加buff
    .addEvents(StaticAnimationProperty.ON_BEGIN_EVENTS,
        SimpleEvent.create((entitypatch, animation, params) -> {
            entitypatch.getOriginal().addEffect(
                new MobEffectInstance(MobEffects.DAMAGE_BOOST, 40, 0)
            );
        }, Side.SERVER)
    )
    // 动画结束时移除buff
    .addEvents(StaticAnimationProperty.ON_END_EVENTS,
        SimpleEvent.create((entitypatch, animation, params) -> {
            entitypatch.getOriginal().removeEffect(MobEffects.DAMAGE_BOOST);
        }, Side.SERVER)
    )
);
```

### 常用事件示例

#### 播放音效

```java
.addEvents(InTimeEvent.create(
    0.2F,
    ReusableSources.PLAY_SOUND,
    Side.CLIENT
).params(SoundEvents.PLAYER_ATTACK_STRONG))
```

#### 生成粒子

```java
.addEvents(InTimeEvent.create(
    0.3F,
    (entitypatch, animation, params) -> {
        Vec3 pos = entitypatch.getOriginal().position();
        Vec3 look = entitypatch.getOriginal().getLookAngle();
        
        // 在前方生成粒子
        for (int i = 0; i < 20; i++) {
            entitypatch.getOriginal().level().addParticle(
                ParticleTypes.SWEEP_ATTACK,
                pos.x + look.x * i * 0.2,
                pos.y + 1.0,
                pos.z + look.z * i * 0.2,
                0, 0, 0
            );
        }
    },
    Side.CLIENT
))
```

#### 造成范围伤害

```java
.addEvents(InTimeEvent.create(
    0.4F,
    (entitypatch, animation, params) -> {
        Entity entity = entitypatch.getOriginal();
        AABB aabb = entity.getBoundingBox().inflate(3.0);
        
        // 对范围内所有敌人造成伤害
        entity.level().getEntities(entity, aabb, (e) -> 
            e instanceof LivingEntity && !e.is(entity)
        ).forEach(target -> {
            target.hurt(
                entity.damageSources().mobAttack((LivingEntity) entity),
                5.0F
            );
        });
    },
    Side.SERVER
))
```

#### 震屏效果

```java
.addEvents(InTimeEvent.create(
    0.5F,
    (entitypatch, animation, params) -> {
        if (entitypatch.getOriginal() instanceof Player player) {
            // 发送震屏数据包给客户端
            NetworkHandler.sendToPlayer(
                new PacketCameraShake(0.5F, 10),
                (ServerPlayer) player
            );
        }
    },
    Side.SERVER
))
```

#### 地面裂纹效果

```java
.addEvents(InTimeEvent.create(
    0.6F,
    Animations.ReusableSources.FRACTURE_GROUND_SIMPLE,
    Side.CLIENT
).params(
    new Vec3f(0.0F, -0.5F, -1.0F),  // 位置偏移
    Armatures.BIPED.get().toolR,     // 关节
    2.0D,                             // 范围
    0.8F                              // 强度
))
```

#### 召唤实体

```java
.addEvents(InTimeEvent.create(
    0.3F,
    (entitypatch, animation, params) -> {
        Entity entity = entitypatch.getOriginal();
        Vec3 pos = entity.position();
        Vec3 look = entity.getLookAngle();
        
        // 在前方召唤一个火球
        Fireball fireball = new Fireball(
            entity.level(),
            (LivingEntity) entity,
            look.x, look.y, look.z
        );
        fireball.setPos(pos.x + look.x * 2, pos.y + 1.5, pos.z + look.z * 2);
        entity.level().addFreshEntity(fireball);
    },
    Side.SERVER
))
```

### 多个事件组合

```java
ULTIMATE_ATTACK = builder.nextAccessor("ultimate/attack", (accessor) ->
    new AttackAnimation(
        0.15F, 0.3F, 0.6F, 0.8F, 2.0F,
        CustomColliders.ULTIMATE,
        Armatures.BIPED.get().toolR,
        accessor,
        Armatures.BIPED
    )
    // 0.1秒 - 蓄力音效
    .addEvents(InTimeEvent.create(0.1F, ReusableSources.PLAY_SOUND, Side.CLIENT)
        .params(SoundEvents.ENDER_DRAGON_GROWL))
    
    // 0.1-0.3秒 - 蓄力粒子
    .addEvents(InPeriodEvent.create(0.1F, 0.3F, (entitypatch, animation, params) -> {
        Vec3 pos = entitypatch.getOriginal().position();
        for (int i = 0; i < 5; i++) {
            double angle = Math.random() * Math.PI * 2;
            double radius = 1.5;
            entitypatch.getOriginal().level().addParticle(
                ParticleTypes.ENCHANT,
                pos.x + Math.cos(angle) * radius,
                pos.y + Math.random() * 2,
                pos.z + Math.sin(angle) * radius,
                -Math.cos(angle) * 0.1,
                0.1,
                -Math.sin(angle) * 0.1
            );
        }
    }, Side.CLIENT))
    
    // 0.3秒 - 释放音效
    .addEvents(InTimeEvent.create(0.3F, ReusableSources.PLAY_SOUND, Side.CLIENT)
        .params(SoundEvents.GENERIC_EXPLODE))
    
    // 0.3秒 - 爆炸粒子
    .addEvents(InTimeEvent.create(0.3F, (entitypatch, animation, params) -> {
        Vec3 pos = entitypatch.getOriginal().position();
        entitypatch.getOriginal().level().addParticle(
            ParticleTypes.EXPLOSION_EMITTER,
            pos.x, pos.y + 1.0, pos.z,
            0, 0, 0
        );
    }, Side.CLIENT))
    
    // 0.3秒 - 范围伤害
    .addEvents(InTimeEvent.create(0.3F, (entitypatch, animation, params) -> {
        Entity entity = entitypatch.getOriginal();
        AABB aabb = entity.getBoundingBox().inflate(5.0);
        entity.level().getEntities(entity, aabb, (e) -> 
            e instanceof LivingEntity && !e.is(entity)
        ).forEach(target -> {
            target.hurt(
                entity.damageSources().explosion(entity, (LivingEntity) entity),
                20.0F
            );
            // 击退
            Vec3 knockback = target.position().subtract(entity.position()).normalize();
            target.setDeltaMovement(knockback.x * 2, 1.0, knockback.z * 2);
        });
    }, Side.SERVER))
    
    // 0.3秒 - 地面裂纹
    .addEvents(InTimeEvent.create(0.3F, 
        Animations.ReusableSources.FRACTURE_GROUND_SIMPLE, Side.CLIENT)
        .params(new Vec3f(0.0F, -0.5F, 0.0F), Armatures.BIPED.get().rootJoint, 5.0D, 1.0F))
);
```

---

## 动画属性与自定义函数

### 动画属性系统

#### 1. 播放速度修改器

```java
CHARGING_ANIMATION = builder.nextAccessor("skill/charging", (accessor) ->
    new ActionAnimation(0.15F, accessor, Armatures.BIPED)
        // 自定义播放速度
        .addProperty(StaticAnimationProperty.PLAY_SPEED_MODIFIER, 
            (self, entitypatch, speed, prevElapsedTime, elapsedTime) -> {
                // 动画播放到一半后停止
                if (elapsedTime > self.getTotalTime() * 0.5F) {
                    return 0.0F;  // 停止播放
                }
                return 1.0F;  // 正常速度
            })
);
```

#### 2. 姿态修改器

```java
AIMING_ANIMATION = builder.nextAccessor("combat/aiming", (accessor) ->
    new StaticAnimation(true, accessor, Armatures.BIPED)
        // 根据玩家视角修改姿态
        .addProperty(StaticAnimationProperty.POSE_MODIFIER, 
            (self, pose, entitypatch, elapsedTime, partialTicks) -> {
                // 获取玩家俯仰角
                float xRot = entitypatch.getOriginal().getXRot();
                
                // 修改胸部关节旋转
                JointTransform chest = pose.orElseEmpty("Chest");
                Quaternion rotation = QuaternionUtils.XP.rotationDegrees(xRot);
                MathUtils.mulQuaternion(rotation, chest.rotation(), chest.rotation());
                
                // 修改头部关节
                JointTransform head = pose.orElseEmpty("Head");
                Quaternion headRot = QuaternionUtils.XP.rotationDegrees(xRot * 0.5F);
                MathUtils.mulQuaternion(headRot, head.rotation(), head.rotation());
            })
);
```

#### 3. 移动坐标修改器

```java
DASH_ANIMATION = builder.nextAccessor("skill/dash", (accessor) ->
    new ActionAnimation(0.1F, 0.5F, accessor, Armatures.BIPED)
        // 设置移动坐标
        .addProperty(ActionAnimationProperty.COORD_SET_BEGIN, 
            (self, entitypatch, transformSheet) -> {
                // 动画开始时设置向前冲刺
                Vec3 look = entitypatch.getOriginal().getLookAngle();
                return new Vec3(look.x * 3.0, 0, look.z * 3.0);
            })
        .addProperty(ActionAnimationProperty.COORD_SET_TICK,
            (self, entitypatch, transformSheet) -> {
                // 每帧更新位置
                return MoveCoordFunctions.TRACE_LOC_TARGET.apply(self, entitypatch, transformSheet);
            })
);
```

#### 4. 攻击属性修改

```java
CRITICAL_ATTACK = builder.nextAccessor("attack/critical", (accessor) ->
    new AttackAnimation(
        0.1F, 0.2F, 0.4F, 0.5F, 0.8F,
        null, Armatures.BIPED.get().toolR,
        accessor, Armatures.BIPED
    )
    // 伤害倍率
    .addProperty(AttackPhaseProperty.DAMAGE_MODIFIER, ValueModifier.multiplier(2.5F))
    // 护甲穿透
    .addProperty(AttackPhaseProperty.ARMOR_NEGATION_MODIFIER, ValueModifier.adder(50.0F))
    // 最大打击数
    .addProperty(AttackPhaseProperty.MAX_STRIKES_MODIFIER, ValueModifier.adder(3))
    // 冲击力
    .addProperty(AttackPhaseProperty.IMPACT_MODIFIER, ValueModifier.multiplier(2.0F))
    // 眩晕类型
    .addProperty(AttackPhaseProperty.STUN_TYPE, StunType.KNOCKDOWN)
    // 粒子效果
    .addProperty(AttackPhaseProperty.PARTICLE, EpicFightParticles.EVISCERATE)
    // 击中音效
    .addProperty(AttackPhaseProperty.HIT_SOUND, SoundEvents.PLAYER_ATTACK_CRIT)
    // 挥砍音效
    .addProperty(AttackPhaseProperty.SWING_SOUND, EpicFightSounds.WHOOSH_SHARP.get())
    // 额外伤害
    .addProperty(AttackPhaseProperty.EXTRA_DAMAGE, 
        Set.of(ExtraDamageInstance.SWEEPING_EDGE_ENCHANTMENT.create()))
    // 伤害标签
    .addProperty(AttackPhaseProperty.SOURCE_TAG, 
        Set.of(EpicFightDamageTypeTags.WEAPON_INNATE))
);
```

### 自定义函数示例

#### 1. 条件伤害加成

```java
BACKSTAB_ATTACK = builder.nextAccessor("attack/backstab", (accessor) ->
    new AttackAnimation(
        0.05F, 0.1F, 0.2F, 0.3F, 0.6F,
        null, Armatures.BIPED.get().toolR,
        accessor, Armatures.BIPED
    )
    // 根据是否背刺调整伤害
    .addProperty(AttackPhaseProperty.DAMAGE_MODIFIER, 
        new ValueModifier() {
            @Override
            public float modify(float original) {
                // 这个会在攻击时调用
                return original * 3.0F;  // 背刺3倍伤害
            }
        })
    // 添加背刺检测事件
    .addEvents(StaticAnimationProperty.ON_BEGIN_EVENTS,
        SimpleEvent.create((entitypatch, animation, params) -> {
            // 检测是否从背后攻击
            Entity attacker = entitypatch.getOriginal();
            Entity target = attacker.level().getNearestEntity(
                LivingEntity.class,
                TargetingConditions.DEFAULT,
                (LivingEntity) attacker,
                attacker.getX(),
                attacker.getY(),
                attacker.getZ(),
                attacker.getBoundingBox().inflate(3.0)
            );
            
            if (target != null) {
                Vec3 toTarget = target.position().subtract(attacker.position()).normalize();
                Vec3 targetLook = target.getLookAngle();
                double dot = targetLook.dot(toTarget);
                
                if (dot < -0.5) {
                    // 是背刺,添加标记
                    attacker.getPersistentData().putBoolean("IsBackstab", true);
                }
            }
        }, Side.SERVER))
);
```

#### 2. 连击系统

```java
public class ComboAnimation extends BasicAttackAnimation {
    private int comboCount = 0;
    
    public ComboAnimation(...) {
        super(...);
        
        // 添加连击计数器
        this.addProperty(AttackPhaseProperty.DAMAGE_MODIFIER, 
            new ValueModifier() {
                @Override
                public float modify(float original) {
                    // 每次连击增加10%伤害
                    return original * (1.0F + comboCount * 0.1F);
                }
            });
        
        // 击中时增加连击数
        this.addEvents(StaticAnimationProperty.ON_BEGIN_EVENTS,
            SimpleEvent.create((entitypatch, animation, params) -> {
                comboCount++;
                if (comboCount > 10) comboCount = 10;  // 最多10连击
            }, Side.SERVER));
    }
    
    public void resetCombo() {
        this.comboCount = 0;
    }
}
```

#### 3. 生命值消耗技能

```java
BLOOD_STRIKE = builder.nextAccessor("skill/blood_strike", (accessor) ->
    new AttackAnimation(
        0.1F, 0.2F, 0.4F, 0.5F, 0.8F,
        null, Armatures.BIPED.get().toolR,
        accessor, Armatures.BIPED
    )
    // 消耗生命值
    .addEvents(StaticAnimationProperty.ON_BEGIN_EVENTS,
        SimpleEvent.create((entitypatch, animation, params) -> {
            LivingEntity entity = entitypatch.getOriginal();
            float healthCost = entity.getMaxHealth() * 0.1F;  // 消耗10%最大生命
            entity.hurt(entity.damageSources().magic(), healthCost);
        }, Side.SERVER))
    // 根据消耗的生命值增加伤害
    .addProperty(AttackPhaseProperty.DAMAGE_MODIFIER,
        new ValueModifier() {
            @Override
            public float modify(float original) {
                // 消耗的生命越多,伤害越高
                return original * 2.0F;
            }
        })
);
```

---

## 自定义动画类

### 基础自定义动画类

```java
package com.yourmod.animation;

import yesman.epicfight.api.animation.types.AttackAnimation;
import yesman.epicfight.api.animation.AnimationAccessor;
import yesman.epicfight.api.model.Armature;
import yesman.epicfight.api.collider.Collider;
import yesman.epicfight.world.capabilities.entitypatch.LivingEntityPatch;

public class CustomAttackAnimation extends AttackAnimation {
    
    public CustomAttackAnimation(
            float convertTime,
            float antic,
            float preDelay,
            float contact,
            float recovery,
            Collider collider,
            String joint,
            AnimationAccessor<?> accessor,
            Armature armature) {
        super(convertTime, antic, preDelay, contact, recovery, 
              collider, joint, accessor, armature);
    }
    
    @Override
    public void begin(LivingEntityPatch<?> entitypatch) {
        super.begin(entitypatch);
        // 动画开始时的自定义逻辑
        System.out.println("Custom attack animation started!");
    }
    
    @Override
    public void tick(LivingEntityPatch<?> entitypatch) {
        super.tick(entitypatch);
        // 每帧执行的自定义逻辑
    }
    
    @Override
    public void end(LivingEntityPatch<?> entitypatch, 
                    AssetAccessor<? extends DynamicAnimation> nextAnimation, 
                    boolean isEnd) {
        super.end(entitypatch, nextAnimation, isEnd);
        // 动画结束时的自定义逻辑
    }
}
```

### 高级示例:元素攻击动画

```java
package com.yourmod.animation;

import net.minecraft.world.entity.LivingEntity;
import net.minecraft.world.effect.MobEffectInstance;
import net.minecraft.world.effect.MobEffects;
import net.minecraft.core.particles.ParticleTypes;
import yesman.epicfight.api.animation.types.AttackAnimation;
import yesman.epicfight.world.capabilities.entitypatch.LivingEntityPatch;

public class ElementalAttackAnimation extends AttackAnimation {
    
    public enum Element {
        FIRE, ICE, LIGHTNING, POISON
    }
    
    private final Element element;
    private final int elementLevel;
    
    public ElementalAttackAnimation(
            float convertTime,
            float antic,
            float preDelay,
            float contact,
            float recovery,
            Collider collider,
            String joint,
            AnimationAccessor<?> accessor,
            Armature armature,
            Element element,
            int level) {
        super(convertTime, antic, preDelay, contact, recovery, 
              collider, joint, accessor, armature);
        this.element = element;
        this.elementLevel = level;
        
        // 根据元素类型添加效果
        this.setupElementalEffects();
    }
    
    private void setupElementalEffects() {
        switch (this.element) {
            case FIRE:
                // 火焰效果
                this.addEvents(InTimeEvent.create(this.contact, 
                    (entitypatch, animation, params) -> {
                        // 生成火焰粒子
                        Vec3 pos = entitypatch.getOriginal().position();
                        for (int i = 0; i < 20; i++) {
                            entitypatch.getOriginal().level().addParticle(
                                ParticleTypes.FLAME,
                                pos.x + (Math.random() - 0.5) * 2,
                                pos.y + Math.random() * 2,
                                pos.z + (Math.random() - 0.5) * 2,
                                0, 0.1, 0
                            );
                        }
                    }, Side.CLIENT));
                
                // 点燃敌人
                this.addProperty(AttackPhaseProperty.EXTRA_DAMAGE,
                    Set.of(new ExtraDamageInstance(
                        (damagesource, target) -> {
                            target.setSecondsOnFire(5 * elementLevel);
                            return 0.0F;
                        }
                    )));
                break;
                
            case ICE:
                // 冰冻效果
                this.addEvents(InTimeEvent.create(this.contact,
                    (entitypatch, animation, params) -> {
                        // 冰霜粒子
                        Vec3 pos = entitypatch.getOriginal().position();
                        for (int i = 0; i < 20; i++) {
                            entitypatch.getOriginal().level().addParticle(
                                ParticleTypes.SNOWFLAKE,
                                pos.x + (Math.random() - 0.5) * 2,
                                pos.y + Math.random() * 2,
                                pos.z + (Math.random() - 0.5) * 2,
                                0, -0.1, 0
                            );
                        }
                    }, Side.CLIENT));
                
                // 减速效果
                this.addProperty(AttackPhaseProperty.EXTRA_DAMAGE,
                    Set.of(new ExtraDamageInstance(
                        (damagesource, target) -> {
                            if (target instanceof LivingEntity living) {
                                living.addEffect(new MobEffectInstance(
                                    MobEffects.MOVEMENT_SLOWDOWN,
                                    100 * elementLevel,
                                    elementLevel - 1
                                ));
                            }
                            return 2.0F * elementLevel;
                        }
                    )));
                break;
                
            case LIGHTNING:
                // 雷电效果
                this.addEvents(InTimeEvent.create(this.contact,
                    (entitypatch, animation, params) -> {
                        Vec3 pos = entitypatch.getOriginal().position();
                        // 召唤闪电
                        LightningBolt lightning = EntityType.LIGHTNING_BOLT.create(
                            entitypatch.getOriginal().level()
                        );
                        if (lightning != null) {
                            lightning.moveTo(pos.x, pos.y, pos.z);
                            entitypatch.getOriginal().level().addFreshEntity(lightning);
                        }
                    }, Side.SERVER));
                break;
                
            case POISON:
                // 中毒效果
                this.addProperty(AttackPhaseProperty.EXTRA_DAMAGE,
                    Set.of(new ExtraDamageInstance(
                        (damagesource, target) -> {
                            if (target instanceof LivingEntity living) {
                                living.addEffect(new MobEffectInstance(
                                    MobEffects.POISON,
                                    200 * elementLevel,
                                    elementLevel - 1
                                ));
                            }
                            return 1.0F * elementLevel;
                        }
                    )));
                break;
        }
    }
    
    @Override
    public void tick(LivingEntityPatch<?> entitypatch) {
        super.tick(entitypatch);
        
        // 持续生成元素粒子
        if (entitypatch.isLogicalClient()) {
            Vec3 pos = entitypatch.getOriginal().position();
            ParticleOptions particle = switch (this.element) {
                case FIRE -> ParticleTypes.FLAME;
                case ICE -> ParticleTypes.SNOWFLAKE;
                case LIGHTNING -> ParticleTypes.ELECTRIC_SPARK;
                case POISON -> ParticleTypes.MYCELIUM;
            };
            
            entitypatch.getOriginal().level().addParticle(
                particle,
                pos.x, pos.y + 1.0, pos.z,
                (Math.random() - 0.5) * 0.1,
                0.1,
                (Math.random() - 0.5) * 0.1
            );
        }
    }
}
```

### 使用自定义动画类

```java
// 注册元素攻击动画
FIRE_SLASH = builder.nextAccessor("elemental/fire_slash", (accessor) ->
    new ElementalAttackAnimation(
        0.1F, 0.2F, 0.3F, 0.4F, 0.8F,
        new OBBCollider(0.6, 0.0, 0.0, 0.0, 0.0, 0.0),
        Armatures.BIPED.get().toolR,
        accessor,
        Armatures.BIPED,
        ElementalAttackAnimation.Element.FIRE,  // 火元素
        3  // 等级3
    )
);

ICE_SLASH = builder.nextAccessor("elemental/ice_slash", (accessor) ->
    new ElementalAttackAnimation(
        0.1F, 0.2F, 0.3F, 0.4F, 0.8F,
        new OBBCollider(0.6, 0.0, 0.0, 0.0, 0.0, 0.0),
        Armatures.BIPED.get().toolR,
        accessor,
        Armatures.BIPED,
        ElementalAttackAnimation.Element.ICE,  // 冰元素
        2  // 等级2
    )
);
```

---

## 完整实战案例

### 案例:创建一个完整的连招系统

```java
package com.yourmod.animation;

import yesman.epicfight.api.animation.types.AttackAnimation;
import yesman.epicfight.api.animation.AnimationAccessor;
import yesman.epicfight.world.capabilities.entitypatch.LivingEntityPatch;

public class ComboSystemAnimation extends AttackAnimation {
    
    private final int comboStage;  // 连招阶段
    private final AnimationAccessor<? extends AttackAnimation> nextCombo;  // 下一段
    
    public ComboSystemAnimation(
            float convertTime,
            float antic,
            float preDelay,
            float contact,
            float recovery,
            Collider collider,
            String joint,
            AnimationAccessor<?> accessor,
            Armature armature,
            int comboStage,
            AnimationAccessor<? extends AttackAnimation> nextCombo) {
        super(convertTime, antic, preDelay, contact, recovery, 
              collider, joint, accessor, armature);
        this.comboStage = comboStage;
        this.nextCombo = nextCombo;
        
        this.setupComboEffects();
    }
    
    private void setupComboEffects() {
        // 根据连招阶段设置不同效果
        float damageMultiplier = 1.0F + (comboStage * 0.2F);
        this.addProperty(AttackPhaseProperty.DAMAGE_MODIFIER, 
            ValueModifier.multiplier(damageMultiplier));
        
        // 连招越高,粒子越多
        this.addEvents(InTimeEvent.create(this.contact,
            (entitypatch, animation, params) -> {
                Vec3 pos = entitypatch.getOriginal().position();
                for (int i = 0; i < comboStage * 5; i++) {
                    entitypatch.getOriginal().level().addParticle(
                        ParticleTypes.CRIT,
                        pos.x, pos.y + 1.0, pos.z,
                        (Math.random() - 0.5) * 0.5,
                        Math.random() * 0.5,
                        (Math.random() - 0.5) * 0.5
                    );
                }
            }, Side.CLIENT));
        
        // 最后一段连招有特殊效果
        if (nextCombo == null) {
            this.addProperty(AttackPhaseProperty.STUN_TYPE, StunType.LONG);
            this.addProperty(AttackPhaseProperty.IMPACT_MODIFIER, 
                ValueModifier.multiplier(2.0F));
            
            // 终结技爆炸
            this.addEvents(InTimeEvent.create(this.contact,
                (entitypatch, animation, params) -> {
                    Vec3 pos = entitypatch.getOriginal().position();
                    entitypatch.getOriginal().level().addParticle(
                        ParticleTypes.EXPLOSION_EMITTER,
                        pos.x, pos.y + 1.0, pos.z,
                        0, 0, 0
                    );
                }, Side.CLIENT));
        }
    }
    
    @Override
    public void end(LivingEntityPatch<?> entitypatch, 
                    AssetAccessor<? extends DynamicAnimation> nextAnimation, 
                    boolean isEnd) {
        super.end(entitypatch, nextAnimation, isEnd);
        
        // 如果有下一段连招,记录连招数据
        if (nextCombo != null && entitypatch.getOriginal() instanceof Player player) {
            player.getPersistentData().putInt("ComboStage", comboStage);
            player.getPersistentData().putLong("ComboTime", 
                player.level().getGameTime());
        }
    }
    
    public AnimationAccessor<? extends AttackAnimation> getNextCombo() {
        return nextCombo;
    }
    
    public int getComboStage() {
        return comboStage;
    }
}
```

### 注册连招动画

```java
@Mod.EventBusSubscriber(modid = "yourmod", bus = Bus.MOD)
public class ComboAnimations {
    
    public static AnimationAccessor<ComboSystemAnimation> COMBO_1;
    public static AnimationAccessor<ComboSystemAnimation> COMBO_2;
    public static AnimationAccessor<ComboSystemAnimation> COMBO_3;
    public static AnimationAccessor<ComboSystemAnimation> COMBO_FINISHER;
    
    @SubscribeEvent
    public static void registerAnimations(AnimationRegistryEvent event) {
        event.newBuilder("yourmod", ComboAnimations::build);
    }
    
    public static void build(AnimationBuilder builder) {
        // 先注册最后一段(因为需要引用)
        COMBO_FINISHER = builder.nextAccessor("combo/finisher", (accessor) ->
            new ComboSystemAnimation(
                0.15F, 0.3F, 0.5F, 0.7F, 1.2F,
                new MultiOBBCollider(10, 0.8, 0.0, 0.0, 0.0, 0.0, -1.5),
                Armatures.BIPED.get().toolR,
                accessor,
                Armatures.BIPED,
                4,      // 第4段
                null    // 没有下一段
            )
        );
        
        // 第3段
        COMBO_3 = builder.nextAccessor("combo/stage3", (accessor) ->
            new ComboSystemAnimation(
                0.1F, 0.2F, 0.3F, 0.4F, 0.7F,
                new OBBCollider(0.7, 0.0, 0.0, 0.0, 0.0, 0.0),
                Armatures.BIPED.get().toolR,
                accessor,
                Armatures.BIPED,
                3,
                COMBO_FINISHER  // 下一段是终结技
            )
        );
        
        // 第2段
        COMBO_2 = builder.nextAccessor("combo/stage2", (accessor) ->
            new ComboSystemAnimation(
                0.08F, 0.15F, 0.25F, 0.35F, 0.6F,
                new OBBCollider(0.6, 0.0, 0.0, 0.0, 0.0, 0.0),
                Armatures.BIPED.get().toolR,
                accessor,
                Armatures.BIPED,
                2,
                COMBO_3
            )
        );
        
        // 第1段
        COMBO_1 = builder.nextAccessor("combo/stage1", (accessor) ->
            new ComboSystemAnimation(
                0.08F, 0.1F, 0.2F, 0.3F, 0.5F,
                new OBBCollider(0.5, 0.0, 0.0, 0.0, 0.0, 0.0),
                Armatures.BIPED.get().toolR,
                accessor,
                Armatures.BIPED,
                1,
                COMBO_2
            )
        );
    }
}
```

---

## 总结

本教程涵盖了 EpicFight 动画系统的高级特性:

1. **碰撞体系统**: 
   - 多种碰撞体类型(OBB, Line, Plane, Multi)
   - 自定义碰撞体创建
   - 多阶段碰撞体配置

2. **事件系统**:
   - InTimeEvent(特定时间点)
   - InPeriodEvent(时间段)
   - SimpleEvent(开始/结束)
   - 各种实用事件示例

3. **动画属性**:
   - 播放速度控制
   - 姿态修改
   - 移动坐标
   - 攻击属性

4. **自定义动画类**:
   - 继承基础动画类
   - 实现自定义逻辑
   - 元素系统示例
   - 连招系统示例

通过这些技术,你可以创建出复杂而精彩的战斗动画系统!
