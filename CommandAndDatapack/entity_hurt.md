---
lang: zh-CN
title: 选择被玩家击中的实体
description: 研究如何精确地选择中被玩家伤害的实体
---

# 选择被玩家击中的实体

在游戏中，我们经常会遇到需要选择被玩家击中的实体的情况。本文将简述几种可以达到该目的的方法。  

选择被玩家击中的实体的方法有多种：
1. 对于活体，利用其`HurtTime`NBT标签，与`minecraft.custom:minecraft.damage_dealt`准则搭配使用
1. 利用`execute on`子命令
1. 使用谓词
1. 使用进度

以下将一一展开。

## 使用NBT标签与记分板

### 思路

对于Minecraft中的所有*活体*，其都拥有`short`类型的`HurtTime`标签，描述这个活体**受到伤害**后的**发红时间**。活体被攻击后这个标签被设为`10s`，然后每tick逐步递减，直至为`0`。  

> 生物也是活体。

同时，记分板系统中还存在一个准则`minecraft.custom:minecraft.damage_dealt`，在每次玩家造成伤害后为该玩家加上造成的伤害\*10分。我们可以利用这个准则来判断发起攻击的玩家。  

综上，我们可以先运行以下命令创建一个记分板：
```
scoreboard objectives add damageDealt minecraft.custom:minecraft.damage_dealt
```

然后高频运行如下命令用`dealt_damage`标记这次循环内造成了伤害的玩家：
```
tag @a remove dealt_damage
tag @a[scores={damageDealt=1..}] add dealt_damage
scoreboard players reset @a damageDealt
```

生存模式下玩家的攻击距离约为3格，可以高频运行以下命令标记刚造成伤害的玩家附近的受伤的生物。
```
tag @e remove player_damaged
execute as @a[tag=dealt_damage] at @s run tag @e[distance=..3,nbt={HurtTime:10s}] add player_damaged
```

标记成功！

### 优点

1. 版本兼容性相当好！即使是很久以前的版本也支持这套做法。*（以前的记分板准则叫`damageDealt`或`minecraft:damage_dealt`，没有`tag`可以用记分板标记来代替）*

### 缺点

1. 无法标记远程攻击和超出范围的横扫攻击(?)
1. 无法完美匹配攻击距离
