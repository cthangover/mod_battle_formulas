# Battle Formulas — 데이터로 쓰는 전투 스킬

Battle Formulas는 전투 행동을 전부 데이터로 쓸 수 있게 해 줍니다. 얼마나 아프게 하고 얼마나 회복하는지는 행동자와 대상의 능력치로 이뤄진 **공식**이 결정하며, 마나 소모, 방어막 무시, 기절도 지정할 수 있습니다. 코드가 필요 없고, 같은 스킬이 FF Battle과 Card Battle 어느 쪽 시스템에서도 작동합니다. Classic RPG Rules의 Power strike, Fireball, Heal, Guard break이 바로 그런 스킬입니다.

또한 적의 전투 수치를 능력치에서 계산하고, 마나 같은 자원을 전투 카드 위의 막대로 표시합니다.

공식은 Stats Engine의 공식 언어로 쓰이고, 전투 화면은 전투 시스템의 것입니다. 이 모드는 그 둘을 잇는 다리입니다.

## 스킬 하나, 단계별로

Classic RPG Rules의 Power strike:

```json
{ "Id": "classic/power_strike", "Type": "ToEnemy",
  "Values": [ "RequiredPoint=1", "Formula=a.atk * 1.5 + rand(0, a.str / 2)" ] }
```

FF 전투 메뉴에서는 카드에 공식이 보이고, 힌트 자리에는 스킬 설명이 나옵니다:

![Power strike를 선택한 영웅의 행동 메뉴. 카드에는 Target enemy, 1 AP, 공식 a.atk * 1.5 + rand(0, a.str / 2)와 설명 "전력으로 휘두르는 일격: 공격의 1.5배에 힘에서 조금 더"가 보입니다.](assets/formula-menu.jpg)

RPG Plus의 전사는 공격 12, 힘 9이므로 이 일격은 12 × 1.5 = 18에 0에서 4.5 사이의 무작위 값을 더합니다. 여기에 담긴 전투에서는 22가 나왔습니다:

![전투 로그: Hero: Power strike → Wolf; Wolf's shield absorbed 2 · left 0; Power strike: Hero → Wolf · -20 HP; 이어서 늑대의 Defence, +4 DEF(총 4).](assets/formula-log.jpg)

늑대의 방어막(2)이 첫 부분을 흡수했습니다 — Power strike는 방어막을 무시하지 않죠 — 그래서 늑대는 체력을 20 잃고 31에서 11이 됐습니다.

같은 카드를 Card Battle에서 보면:

![Card Battle의 Power strike 행동 카드: Target enemy, 1 AP, 공식 a.atk * 1.5 + rand(0, a.str / 2), RequiredPoint 1.](assets/formula-card.jpg)

## 적의 수치는 능력치에서 나옵니다

늑대의 체력 31, 공격 7, 방어막 2는 어디에도 숫자로 적혀 있지 않습니다. RPG Plus는 패치로 늑대에게 능력치를 부여합니다:

```json
{ "Id": "rpg_wolf", "Stats": { "str": 5, "vit": 5, "agi": 7 } }
```

전투가 시작되면 이 모드는 Classic RPG Rules에 따라 2레벨 늑대의 시트를 계산합니다. 체력 `12 + vit * 3 + level * 2` = 31, 공격 `str + level` = 7, 방어 `floor(vit / 2)` = 2. 능력치 데이터가 없는 적은 캐릭터 파일의 수치를 그대로 씁니다.

## 공식 행동의 값

`Formula=` 값을 가진 행동 엔티티(종류 `action`, Characters Engine 참조):

- `Formula=` — 그 양입니다. 절대 0 아래로 내려가지 않습니다. `a.`는 행동자, `b.`는 대상: 두 쪽의 어떤 능력치든 쓸 수 있고, `hp`, `hp_max`, `attack`, `defence`, `point`도 있습니다. `rand(lo, hi)`, `floor`, `min`, `max` 같은 함수는 Stats Engine에서 옵니다.
- `Effect=damage`(기본값) 또는 `Effect=heal`.
- `Cost=mp:4` — 풀 스탯에서 4를 소모합니다. 파티원의 잔량이 부족하면 행동이 거부되고 아무것도 소모되지 않습니다. 적은 지불하지 않습니다.
- `RequiredPoint=1` — 행동 포인트. 모든 전투 시스템에서 마찬가지입니다.
- `Pierce=1` — 대상의 방어막을 무시합니다.
- `State=effect/physics/stun`에 `StateTurns=1`과 `StateChance=35`를 함께 — 35% 확률로 대상에게 상태 효과를 그 턴 수만큼 겁니다.
- `Type` — `ToEnemy`, `ToSelf` 또는 `ToAlias`(아군). 어떤 행동이든 마찬가지입니다.

예컨대 Classic RPG Rules의 Fireball:

```json
"Values": [ "RequiredPoint=1", "Cost=mp:4", "Pierce=1", "Formula=a.mag * 2 + rand(1, 6)" ]
```

## 문제가 생겼을 때

1. **스킬이 0을 냄.** 공식 속 이름이 게임의 능력치가 아닌(모르는 이름은 0으로 셉니다) 경우이거나, 결과가 음수로 나왔습니다.
2. **스킬이 거부됨.** 캐릭터에게 행동 포인트나 `Cost=`에 적힌 풀 스탯이 부족합니다.
3. **적이 파일의 수치를 씀, 능력치의 수치가 아님.** `Stats`, `Archetype`, `Traits`가 없는 것입니다.
