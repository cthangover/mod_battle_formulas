# Battle Formulas —— 以数据书写的战斗技能

Battle Formulas 让战斗动作可以完全用数据来写：动作造成多少伤害或治疗，是一个以行动者和目标属性为变量的**公式**；动作还可以消耗魔力、无视护盾或造成眩晕。无需任何代码，同一个技能就能在 FF Battle 和 Card Battle 两套战斗系统中通用。Classic RPG Rules 的 Power strike、Fireball、Heal 和 Guard break 就是这样写的技能。

它还会根据属性推算敌人的战斗数值，并把魔力之类的资源以进度条形式画在战斗卡牌上。

公式使用 Stats Engine 的公式语言；战斗画面则属于各战斗系统。这个模组就是两者之间的桥梁。

## 逐行拆解一个技能

Classic RPG Rules 的 Power strike：

```json
{ "Id": "classic/power_strike", "Type": "ToEnemy",
  "Values": [ "RequiredPoint=1", "Formula=a.atk * 1.5 + rand(0, a.str / 2)" ] }
```

在 FF 战斗菜单里，卡牌上会显示公式，提示行则是技能的描述：

![英雄的动作菜单，已选中 Power strike；卡牌显示 Target enemy、1 AP 和公式 a.atk * 1.5 + rand(0, a.str / 2)，并附描述「全力一击：1.5 倍攻击，再加一点力量加成」。](assets/formula-menu.jpg)

RPG Plus 的战士攻击为 12、力量为 9，因此这一击造成 12 × 1.5 = 18，再加 0 到 4.5 的随机值。在截图的这场战斗中它打出了 22：

![战斗日志：Hero: Power strike → Wolf；Wolf's shield absorbed 2 · left 0；Power strike: Hero → Wolf · -20 HP；随后是狼的 Defence，+4 DEF（共 4）。](assets/formula-log.jpg)

狼的护盾（2）先吸收了一部分——Power strike 并不无视护盾——狼因此损失 20 点生命，从 31 降到 11。

同一张卡在 Card Battle 中：

![Card Battle 中的 Power strike 动作卡：Target enemy、1 AP、公式 a.atk * 1.5 + rand(0, a.str / 2)、RequiredPoint 1。](assets/formula-card.jpg)

## 敌人的数值来自属性

狼的 31 点生命、攻击 7 和护盾 2 并不是写死的数字。RPG Plus 通过补丁赋予狼属性：

```json
{ "Id": "rpg_wolf", "Stats": { "str": 5, "vit": 5, "agi": 7 } }
```

战斗开始时，本模组按 Classic RPG Rules 为 2 级的狼计算面板：生命 `12 + vit * 3 + level * 2` = 31，攻击 `str + level` = 7，防御 `floor(vit / 2)` = 2。没有属性数据的敌人则继续使用其角色文件中的数值。

## 公式动作的取值

一个带有 `Formula=` 值的动作实体（`action` 类型，参见 Characters Engine）：

- `Formula=`——数值大小；永不低于 0。`a.` 是行动者，`b.` 是目标：可用它们任意属性，也有 `hp`、`hp_max`、`attack`、`defence`、`point`。`rand(lo, hi)`、`floor`、`min`、`max` 等函数来自 Stats Engine。
- `Effect=damage`（默认）或 `Effect=heal`。
- `Cost=mp:4`——从某项资源属性中扣除 4；队友的存量不足时动作会被拒绝，什么也不消耗。敌人无需支付。
- `RequiredPoint=1`——行动点，与所有战斗系统一致。
- `Pierce=1`——无视目标的护盾。
- `State=effect/physics/stun` 配合 `StateTurns=1` 和 `StateChance=35`——以 35% 几率给目标施加状态效果，持续相应回合数。
- `Type`——`ToEnemy`、`ToSelf` 或 `ToAlias`（队友），与所有动作一致。

例如 Classic RPG Rules 的 Fireball：

```json
"Values": [ "RequiredPoint=1", "Cost=mp:4", "Pierce=1", "Formula=a.mag * 2 + rand(1, 6)" ]
```

## 出问题怎么办

1. **技能打出 0。** 公式中的某个名字并不是游戏里的属性（未知名字按 0 计），或者结果为负。
2. **技能被拒绝。** 角色的行动点不足，或 `Cost=` 指定的资源属性存量不够。
3. **敌人用的是文件里的数值，而不是属性推算值。** 它没有 `Stats`、`Archetype` 或 `Traits`。
