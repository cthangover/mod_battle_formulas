# Battle Formulas —— データとして書くバトルスキル

Battle Formulas を使うと、バトルアクションを完全にデータとして書けます。どれだけ傷つけ、どれだけ回復するかは、行動者と対象のステータスから成る**式**で決まり、マナの消費、シールドの無視、スタンも設定できます。コードは不要で、同じスキルが FF Battle と Card Battle の両方のシステムで機能します。Classic RPG Rules の Power strike・Fireball・Heal・Guard break は、まさにそのようなスキルです。

さらに、敵の戦闘値をステータスから計算し、マナのようなリソース（プール）を戦闘カードのバーとして表示します。

式は Stats Engine の式言語で書かれ、戦闘画面はバトルシステムのものです。このModはその二つをつなぐ役割です。

## スキルを一つずつ

Classic RPG Rules の Power strike：

```json
{ "Id": "classic/power_strike", "Type": "ToEnemy",
  "Values": [ "RequiredPoint=1", "Formula=a.atk * 1.5 + rand(0, a.str / 2)" ] }
```

FF のバトルメニューではカードに式が表示され、ヒントにはスキルの説明が出ます：

![Power strike を選んだ英雄のアクションメニュー。カードには Target enemy、1 AP、式 a.atk * 1.5 + rand(0, a.str / 2)、そして説明「渾身の一撃：攻撃の1.5倍に力の値を少し上乗せ」。](assets/formula-menu.jpg)

RPG Plus の戦士は攻撃 12・力 9 なので、この一撃は 12 × 1.5 = 18 に 0〜4.5 の乱数を加えた値になります。ここで記録した戦闘では 22 を出しました：

![戦闘ログ：Hero: Power strike → Wolf／Wolf's shield absorbed 2 · left 0／Power strike: Hero → Wolf · -20 HP／続いて狼の Defence、+4 DEF（合計 4）。](assets/formula-log.jpg)

狼のシールド（2）が最初の部分を受け止めました——Power strike はシールドを無視しません——体力は 20 減り、31 から 11 へ。

同じカードを Card Battle で見ると：

![Card Battle での Power strike アクションカード：Target enemy、1 AP、式 a.atk * 1.5 + rand(0, a.str / 2)、RequiredPoint 1。](assets/formula-card.jpg)

## 敵の数値はステータスから

狼の体力 31、攻撃 7、シールド 2 は、どこにも数値として書かれていません。RPG Plus はパッチで狼にステータスを与えます：

```json
{ "Id": "rpg_wolf", "Stats": { "str": 5, "vit": 5, "agi": 7 } }
```

戦闘が始まると、このModは Classic RPG Rules に従ってレベル 2 の狼のシートを計算します。体力 `12 + vit * 3 + level * 2` = 31、攻撃 `str + level` = 7、防御 `floor(vit / 2)` = 2。ステータスデータのない敵は、キャラクターファイルの数値を使い続けます。

## 式アクションの値

`Formula=` 値を持つアクションエンティティ（種類 `action`。Characters Engine を参照）：

- `Formula=`——量。決して 0 未満にはなりません。`a.` は行動者、`b.` は対象：両者の任意のステータスに加え、`hp`、`hp_max`、`attack`、`defence`、`point` が使えます。`rand(lo, hi)`、`floor`、`min`、`max` などの関数は Stats Engine 由来です。
- `Effect=damage`（既定）または `Effect=heal`。
- `Cost=mp:4`——プール型ステータスから 4 消費します。パーティメンバーが足りなければ行動は拒否され、何も消費されません。敵は支払い不要です。
- `RequiredPoint=1`——行動ポイント。どのバトルシステムでも同じです。
- `Pierce=1`——対象のシールドを無視します。
- `State=effect/physics/stun` に `StateTurns=1` と `StateChance=35` を合わせると——35% の確率で対象に状態異常をそのターン数の間与えます。
- `Type`——`ToEnemy`、`ToSelf`、`ToAlias`（味方）。どのアクションでも同じです。

たとえば Classic RPG Rules の Fireball：

```json
"Values": [ "RequiredPoint=1", "Cost=mp:4", "Pierce=1", "Formula=a.mag * 2 + rand(1, 6)" ]
```

## うまくいかないときは

1. **スキルが 0 になる。** 式の中の名前がゲームのステータスではない（未知の名前は 0 として数えられます）か、結果が負になりました。
2. **スキルが拒否される。** キャラクターに行動ポイント、または `Cost=` で指定したプール型ステータスが足りません。
3. **敵の数値がファイル由来で、ステータス由来でない。** `Stats`・`Archetype`・`Traits` を持っていません。
