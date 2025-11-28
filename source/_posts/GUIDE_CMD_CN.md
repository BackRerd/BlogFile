---
title: 词条介绍(cbs)
tags:
  - pfc
  - ca
  - mc
  - mod
categories:
  - pfc
  - ca
abbrlink: f2bfe1a
date: 2025-11-25 09:12:28
---
# ⚡ ClassBioArsenal 指令系统教程

> 服务器管理员与玩家指令指南

---

## 🗡️ 武器防具系统指令

### 🔄 重载配置：`reload`

```text
 /cba_weapon reload
```

- **功能**：重载武器/防具系统配置
- **场景**：修改 JSON 后无需重启服务器
- 🔄 **同步**：执行后会将最新的武器/护甲配置通过数据包同步给所有在线玩家客户端
- 💡 **建议**：每次修改配置后执行一次

### 📊 设置等级：`level set`

```text
/cba_weapon level set <level>
```

- **功能**：直接设置武器/防具等级
- **参数**：目标等级（建议 1~100）
- ⚠️ **注意**：需手持支持系统的武器/防具

**示例**：
```text
/cba_weapon level set 10    # 设置为 10 级
```

### 💎 设置稀有度：`rarity set`

```text
/cba_weapon rarity set <rarityId>
```

- **功能**：设置武器/防具品质（稀有度）
- **参数**：稀有度 ID（如 `common`、`epic`）
- 💡 **提示**：Tab 键可补全有效 ID

**示例**：
```text
/cba_weapon rarity set epic    # 设置为史诗品质
```

### ⭐ 经验值操作：`xp set` / `xp add`

```text
/cba_weapon xp set <amount>
/cba_weapon xp add <amount>
```

- **作用对象**：当前主手武器/防具
- **功能**：
  - `set`：直接设置经验值
  - `add`：增加经验值
- **参数**：整数（`set`≥0，`add`≥1）
- ⚡ **自动升级**：修改经验后自动判断升级

**示例**：
```text
/cba_weapon xp set 0        # 重置经验
/cba_weapon xp add 500      # 增加 500 经验
```

### ✨ 词缀管理：`affix set` / `affix remove`

#### 🔧 设置/修改词缀：`affix set`

```text
/cba_weapon affix set <affixId> <level>
```

- **功能**：添加或修改词缀
  - 无词缀 → 添加指定等级词缀
  - 已有词缀 → 修改为指定等级
- **参数**：
  - `<affixId>`：词缀 ID（如 `fire_damage`）
  - `<level>`：词缀等级（建议从 1 开始）
- 💡 **提示**：Tab 键查看可用词缀 ID

**示例**：
```text
/cba_weapon affix set fire_damage 3    # 添加/修改火焰伤害 3 级
```

#### 🗑️ 移除词缀：`affix remove`

```text
/cba_weapon affix remove <affixId>
```

- **功能**：移除指定词缀
- **参数**：要移除的词缀 ID
- **结果**：
  - ✅ 存在词缀 → 移除成功
  - ❌ 不存在词缀 → 提示找不到

**示例**：
```text
/cba_weapon affix remove fire_damage    # 移除火焰伤害词缀
```

---

## 👥 职业系统指令

### 📋 查看职业列表：`list`

```text
/profession list
```

- **功能**：列出所有已注册职业
- **显示**：按基础/进阶职业分组
- **用途**：获取可用职业 ID

### 📊 查看职业详情：`info`

```text
/profession info <职业ID>
```

- **功能**：查看职业详细信息
- **参数**：职业 ID（非显示名）
- **显示内容**：
  - 🏷️ 职业 ID、显示名
  - 📊 基础/进阶职业类型
  - 🌳 上级职业（进阶职业）
  - 📈 等级上限、基础经验、成长倍率
  - ⚔️ 属性加成概览

**示例**：
```text
/profession info warrior    # 查看战士职业详情
```

### 🔄 重载配置：`reload`

```text
/profession reload all          # 重载全部
/profession reload professions   # 仅职业配置
/profession reload items         # 仅物品职业配置
```

- **`all`**：同时重载职业和物品职业配置
- **`professions`**：仅重载职业数值、结构等
- **`items`**：仅重载物品绑定职业配置
 - 🔄 **同步**：执行后会自动将职业与物品职业配置通过数据包同步到所有在线玩家客户端

> 💡 **建议**：修改数据包后使用 `reload all`

### 📦 导出职业配置：`export`

```text
/profession export <职业ID> <nbtMode>
```

- **功能**：根据当前玩家背包与装备，导出对应职业的 JSON 配置文件到 `logs` 目录（如：`logs/profession_warrior.json`）
- **职业ID**：可使用 Tab 自动补全（与 `/profession info` 相同）
- **`nbtMode` 选项**：控制是否导出物品 NBT，以及导出哪些槽位的 NBT
  - `none`：不导出任何 NBT
  - `all`：所有导出的物品都包含 NBT
  - `equip`：仅护甲槽位（`head/chest/legs/feet`）导出 NBT
  - `equip_main`：护甲 + 主手（`mainhand`）导出 NBT
  - `equip_main_off`：护甲 + 主手 + 副手（`offhand`）导出 NBT

> ⚠️ **重要说明**：由于数据包在不同环境中的存放路径各不相同，指令**无法直接修改数据包内的职业 JSON**，只能将当前背包内容**导出为 JSON 文件**，再由你手动复制/调整后放入对应数据包位置使用。

### 🎮 玩家职业操作：`player ...`

#### 🔧 设置/重置职业：`set` / `reset`

```text
/profession player set <职业ID> [玩家名]
/profession player reset [玩家名]
```

- **目标玩家**：省略玩家名 = 对自己生效
- **`set`**：直接设置职业（初始等级 1）
- **`reset`**：清除职业（变为无职业）

**示例**：
```text
/profession player set warrior          # 自己设为战士
/profession player set mage Steve       # Steve 设为法师
/profession player reset                # 重置自己职业
/profession player reset Alex           # 重置 Alex 职业
```

#### ⭐ 添加职业经验：`addxp` / `addxpradius`

```text
/profession player addxp <数量> [玩家名]
/profession player addxpradius <数量> <半径>
```

- **前置条件**：目标玩家必须有职业
- **`addxp`**：给单个玩家加经验
- **`addxpradius`**：给范围内所有有职业玩家加经验

**示例**：
```text
/profession player addxp 100            # 自己 +100 经验
/profession player addxp 200 Steve      # Steve +200 经验
/profession player addxpradius 50 32    # 半径32格内所有有职业玩家 +50 经验
```

#### 📊 查看自身信息：`info`

```text
/profession player info
```

- **功能**：查看自己当前职业状态
- **显示内容**：
  - 🏷️ 职业名称
  - 📊 当前等级
  - ⭐ 当前经验/升级所需经验
  - 📈 进度百分比

> 💡 **玩家常用**：建议告诉玩家这个指令

### ❓ 查看帮助：根命令

```text
/profession
```

- **功能**：显示简要指令帮助列表
- **用途**：临时忘记子命令时快速查看

---

## 🧬 实体系统指令

### 🔄 重载实体配置：`/cba_entity reload`

```text
/cba_entity reload
```

- **功能**：重载实体系统的配置文件（例如怪物/生物的自定义属性、标签、掉落等）
- **场景**：修改实体系统相关 JSON 配置后，无需重启服务器即可让新配置生效
- 🔄 **同步**：只需在服务器执行一次，新的实体配置会在服务端生效，并通过同步机制影响所有在线玩家

## 🎯 常见使用场景

### 🗡️ 修改武器等级和词缀
1. 🤏 手持目标武器
2. 📊 `/cba_weapon level set 20` - 设置为 20 级
3. ✨ `/cba_weapon affix set fire_damage 3` - 添加 3 级火焰伤害

### 🔄 测试新配置
1. ✏️ 修改数据包配置
2. ⚡ `/cba_weapon reload` + `/profession reload all`
3. 🎮 测试体验效果

### 👥 给玩家设置职业
1. 🔧 `/profession player set warrior Steve` - 设置 Steve 为战士
2. ⭐ `/profession player addxp 500 Steve` - 给 Steve 加 500 经验
3. 📊 提示 Steve 用 `/profession player info` 查看状态

### 🎁 活动奖励经验
1. 📍 站在活动区域中央
2. 🎉 `/profession player addxpradius 1000 64` - 半径 64 格内所有有职业玩家 +1000 经验

---

## 🔒 权限与安全建议

### 👤 管理员权限
- 🛡️ **仅给信任的管理员**使用这些指令
- ⚠️ 大量经验/高等级/高品质词缀会破坏平衡

### 🎮 生存服务器建议
- 🧪 **测试环境**：仅用于测试或活动奖励
- 🚫 **普通玩家**：不要开放 `/cba_weapon` 和 `/profession player set`
- 📝 **记录改动**：重要服务器建议记录关键指令执行

### 💡 最佳实践
- 🎯 **谨慎使用**：避免随意修改玩家数据
- 🔄 **定期备份**：重要数据修改前备份
- 📋 **文档记录**：在管理群记录重要操作

---

> 💡 **需要更多帮助？**
> - 📖 新增指令文档更新
> - 🎯 具体使用场景指导
