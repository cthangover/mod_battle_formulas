# Battle Formulas — kỹ năng chiến đấu viết bằng dữ liệu

Battle Formulas cho phép viết một hành động chiến đấu hoàn toàn bằng dữ liệu: hành động gây hay hồi bao nhiêu là một **công thức** dựa trên chỉ số của người hành động và mục tiêu; nó có thể tốn mana, bỏ qua lớp khiên hoặc gây choáng. Không cần viết mã, và cùng một kỹ năng chạy được trên cả hai hệ thống chiến đấu, FF Battle lẫn Card Battle. Power strike, Fireball, Heal và Guard break của Classic RPG Rules đều là những kỹ năng như thế.

Mod còn suy ra các con số chiến đấu của kẻ địch từ chỉ số của chúng, và vẽ các nguồn tài nguyên như mana thành thanh trên thẻ chiến đấu.

Công thức dùng ngôn ngữ công thức của Stats Engine; còn màn hình chiến đấu thuộc về các hệ thống chiến đấu. Mod này chính là cầu nối giữa hai phần đó.

## Một kỹ năng, từng bước

Power strike của Classic RPG Rules:

```json
{ "Id": "classic/power_strike", "Type": "ToEnemy",
  "Values": [ "RequiredPoint=1", "Formula=a.atk * 1.5 + rand(0, a.str / 2)" ] }
```

Trong menu chiến đấu của FF, thẻ hiển thị công thức, còn dòng gợi ý chính là mô tả của kỹ năng:

![Menu hành động của anh hùng với Power strike đang chọn; thẻ hiển thị Target enemy, 1 AP và công thức a.atk * 1.5 + rand(0, a.str / 2), kèm mô tả "Vung trọn đòn: một lần rưỡi sát thương và thêm một chút từ sức mạnh".](assets/formula-menu.jpg)

Chiến binh RPG Plus có sát thương 12 và sức mạnh 9, nên đòn này gây 12 × 1.5 = 18 cộng một giá trị ngẫu nhiên từ 0 đến 4.5. Trong trận được chụp lại ở đây nó gây 22:

![Nhật ký chiến đấu: Hero: Power strike → Wolf; Wolf's shield absorbed 2 · left 0; Power strike: Hero → Wolf · -20 HP; rồi đến Defence của sói, +4 DEF (tổng 4).](assets/formula-log.jpg)

Lớp khiên 2 điểm của sói đỡ phần đầu — Power strike không bỏ qua khiên — nên con sói mất 20 máu, từ 31 xuống 11.

Cùng thẻ đó trong Card Battle:

![Thẻ hành động Power strike trong Card Battle: Target enemy, 1 AP, công thức a.atk * 1.5 + rand(0, a.str / 2), RequiredPoint 1.](assets/formula-card.jpg)

## Kẻ địch lấy số từ chỉ số của chúng

Máu 31, sát thương 7 và lớp khiên 2 của con sói không hề được ghi dưới dạng con số ở bất kỳ đâu. RPG Plus trao chỉ số cho con sói bằng một bản vá:

```json
{ "Id": "rpg_wolf", "Stats": { "str": 5, "vit": 5, "agi": 7 } }
```

Khi trận chiến bắt đầu, mod này tính bảng chỉ số của con sói theo Classic RPG Rules ở cấp 2: máu `12 + vit * 3 + level * 2` = 31, sát thương `str + level` = 7, phòng thủ `floor(vit / 2)` = 2. Kẻ địch không có dữ liệu chỉ số sẽ giữ nguyên các con số từ tệp nhân vật của nó.

## Các giá trị của hành động công thức

Một thực thể hành động (loại `action`, xem Characters Engine) mang giá trị `Formula=`:

- `Formula=` — lượng tác động; không bao giờ nhỏ hơn 0. `a.` là người hành động, `b.` là mục tiêu: có thể dùng bất kỳ chỉ số nào của họ, cùng `hp`, `hp_max`, `attack`, `defence`, `point`. Các hàm như `rand(lo, hi)`, `floor`, `min`, `max` đến từ Stats Engine.
- `Effect=damage` (mặc định) hoặc `Effect=heal`.
- `Cost=mp:4` — trừ 4 điểm từ một chỉ số dạng nguồn; nếu thành viên đội không đủ, hành động bị từ chối và không tốn gì. Kẻ địch không phải trả giá.
- `RequiredPoint=1` — điểm hành động, như ở mọi hệ thống chiến đấu.
- `Pierce=1` — bỏ qua lớp khiên của mục tiêu.
- `State=effect/physics/stun` cùng `StateTurns=1` và `StateChance=35` — 35% cơ hội tròng lên mục tiêu một hiệu ứng trạng thái đúng số lượt đó.
- `Type` — `ToEnemy`, `ToSelf` hoặc `ToAlias` (đồng đội), như mọi hành động khác.

Lấy Fireball của Classic RPG Rules làm ví dụ:

```json
"Values": [ "RequiredPoint=1", "Cost=mp:4", "Pierce=1", "Formula=a.mag * 2 + rand(1, 6)" ]
```

## Khi có sự cố

1. **Kỹ năng gây đúng 0.** Có tên trong công thức không phải chỉ số của game (tên lạ được tính là 0), hoặc kết quả ra số âm.
2. **Kỹ năng bị từ chối.** Nhân vật thiếu điểm hành động hoặc thiếu chỉ số nguồn được nêu trong `Cost=`.
3. **Kẻ địch mang số từ tệp, chứ không từ chỉ số.** Nó không có `Stats`, `Archetype` hay `Traits`.
