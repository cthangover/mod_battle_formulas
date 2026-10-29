# Battle Formulas — battle skills written as data

Battle Formulas lets a battle action be written entirely in data: how much it hurts or heals is a **formula** of the stats of the one acting and of the target, it may cost mana, ignore the shield or stun. No code is needed, and the same skill works in both battle systems, FF Battle and Card Battle. Classic RPG Rules' Power strike, Fireball, Heal and Guard break are such skills.

It also gives enemies their battle numbers from their stats, and shows pools such as mana as bars on the battle cards.

The formulas use the formula language of Stats Engine; the battle screens are those of the battle systems. This mod is the link between them.

## A skill, step by step

Power strike from Classic RPG Rules:

```json
{ "Id": "classic/power_strike", "Type": "ToEnemy",
  "Values": [ "RequiredPoint=1", "Formula=a.atk * 1.5 + rand(0, a.str / 2)" ] }
```

In the FF battle menu the card shows the formula, and the hint is the skill's description:

![The Hero's actions menu with Power strike selected; its card shows Target enemy, 1 AP and Formula a.atk * 1.5 + rand(0, a.str / 2), with the description "A full swing: one and a half times the attack and a bit more from strength."](assets/formula-menu.jpg)

The RPG Plus warrior has attack 12 and strength 9, so the strike does 12 × 1.5 = 18 plus a random 0 to 4.5. In the battle captured here it did 22:

![The battle log: Hero: Power strike → Wolf; Wolf's shield absorbed 2 · left 0; Power strike: Hero → Wolf · -20 HP; then the wolf's Defence, +4 DEF (total 4).](assets/formula-log.jpg)

The wolf's shield of 2 took the first part — Power strike does not ignore shields — and the wolf lost 20 health, from 31 to 11.

The same card in Card Battle:

![The Power strike action card in Card Battle: Target enemy, 1 AP, Formula a.atk * 1.5 + rand(0, a.str / 2), RequiredPoint 1.](assets/formula-card.jpg)

## Enemies get numbers from their stats

The wolf's 31 health, attack 7 and shield 2 are not written anywhere as numbers. RPG Plus gives the wolf stats with a patch:

```json
{ "Id": "rpg_wolf", "Stats": { "str": 5, "vit": 5, "agi": 7 } }
```

When the battle starts, this mod computes the wolf's sheet with Classic RPG Rules at its level 2: health `12 + vit * 3 + level * 2` = 31, attack `str + level` = 7, defence `floor(vit / 2)` = 2. An enemy without stats data keeps the numbers from its character file.

## Values of a formula action

An action entity (kind `action`, see Characters Engine) with a `Formula=` value:

- `Formula=` — the amount; never below 0. `a.` is the one acting, `b.` the target: any of their stats, and also `hp`, `hp_max`, `attack`, `defence`, `point`. Functions such as `rand(lo, hi)`, `floor`, `min`, `max` come from Stats Engine.
- `Effect=damage` (the default) or `Effect=heal`.
- `Cost=mp:4` — spends 4 of a pool stat; when a party member has too little, the action is refused and nothing is spent. Enemies pay nothing.
- `RequiredPoint=1` — action points, as in every battle system.
- `Pierce=1` — ignores the target's shield.
- `State=effect/physics/stun` with `StateTurns=1` and `StateChance=35` — a 35% chance to put a status effect on the target for that many turns.
- `Type` — `ToEnemy`, `ToSelf` or `ToAlias` (an ally), as for any action.

Classic RPG Rules' Fireball, for example:

```json
"Values": [ "RequiredPoint=1", "Cost=mp:4", "Pierce=1", "Formula=a.mag * 2 + rand(1, 6)" ]
```

## If something goes wrong

1. **A skill does 0.** A name in the formula is not a stat of the game (unknown names count as 0), or the result was negative.
2. **A skill is refused.** The character lacks the action points or the pool stat named in `Cost=`.
3. **An enemy has the numbers of its file, not of its stats.** It has no `Stats`, `Archetype` or `Traits`.
