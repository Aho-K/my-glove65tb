# 片手運用（右半分のみ）ミラータイピング — Claude Code 引き継ぎ書

> このファイルは、別セッション（Claude Web）で詰めた設計方針を、ローカル clone 後の
> Claude Code に引き継ぐためのものです。`config/glove65tb.keymap` の編集が主タスク。
> まずこのファイルを通読してから作業を始めてください。

---

## 0. ゴール

glove65tb（左右分割・右側が親機＝トラックボール側）の **右半分だけ** を使って、

1. 動画鑑賞・メディア操作
2. ネットサーフィン・スクロール
3. 軽いタイピング（URL・検索ワード・短い文程度を想定。長文前提ではない ※要確認）

を片手で完結させたい。**両手用の DEFAULT(L0) は壊さない。** 片手モードは独立して追加する。

---

## 1. 採用方針（確定）

### 1-1. タイピングは Half-QWERTY ミラー方式
左半分のキーを適当にレイヤーへ置くと位置を覚え直しになる。ミラー（鏡像）なら、
右手が「左手と相同（homologous）な動き」をするだけなので、既存のタッチタイピングの
指の記憶がそのまま転用でき、立ち上がりが速い（Matias の Half-QWERTY, 1993〜 の手法）。

- 参考: タッチタイピストで約8時間で両手速度の約50%到達という報告がある（目安）。
- 注意: ZMK には QMK の `swap-hands`（ワンキーで全体ミラー）に相当する機能が **まだ無い**。
  そのため **ミラーは手動でレイヤーとして組む**必要がある（本書の方針はこれ）。

### 1-2. トグルで入る「片手モード」として実装
普段の両手運用（L0）には一切触らない。`&tog` で入る独立した片手ベース層を足す。

### 1-3. 既存リポジトリの「消費レイヤー」側はほぼ完成している
トラックボール周りは実装済みなので、基本流用するだけ（下記 2-2 参照）。

---

## 2. リポジトリ現状の分析（再解析不要・ここを正とする）

### 2-1. 物理構成
- 分割キーボード、計 65 キー。matrix transform は 14 列 × 5 行。
- 左半分 = 列 0–6、右半分 = 列 7–13（右 overlay で `col-offset = <7>`）。
- 右側が central（親機）。トラックボールは PMW3610（`config/.../glove65tb_R.overlay`, `_R.conf`）。

### 2-2. トラックボール（消費側）は設定済み
`boards/shields/glove65tb/glove65tb_R.overlay` の trackball ノードに:
- `automouse-layer = <4>;` … 動かすと自動で MOUSE(L4) へ
- `scroll-layers = <5>;` … SCROLL(L5) 中はボール操作がスクロールになる
- `snipe-layers = <6>;` … SNIPE(L6) 中は低 CPI 精密モード

`config/glove65tb.keymap` 側に:
- コンボ `key-positions = <50 51>` → `&mo 5`（SCROLL モメンタリ）
- `lt_mouse_layer`（hold=`&mo`, tap=`&mkp`）を keypos 51 に使用
- `&msc` / `&mmv` のチューニング、`zip_wheel_scaler` 入力プロセッサあり

→ **動画鑑賞・ネットサーフィン用途は基本このまま動く。** 片手モードでも流用する。

### 2-3. 既に「部分ミラー」の痕跡がある
- 右側内側の列（col7 = keypos 19/33/47）に `T / G / B` が置いてある。これは右手 index 列の
  `Y / H / N`（col8 = keypos 20/34/48）の真隣に、左手 index 列の文字を置いた＝
  **index 指のミラーを既に手で実装している**状態。本方針と整合する。
- FUN / SYM / SCROLL の右側に矢印キーや Ctrl 系（Ctl+W, Ctl+C/X/V 等）が寄せてある。

### 2-4. レイヤー構成（現状）
```
DEFAULT=0  NUM=1  SYM=2  FUN=3  MOUSE=4  SCROLL=5  SNIPE=6   layer_7(=7, 空・全 trans)
```
- `conditional_layers` に "Test": `if-layers = <1 3>; then-layer = <7>;`
  → **layer_7 は「NUM(1)+FUN(3) 同時押しで発火する条件レイヤーの行き先」。**
  見た目は空だが**自由枠ではない**。MIRROR をここに置くと NUM+FUN 同時押しで誤発火する。
  ⚠ **新レイヤーは index 8 以降を使うこと。**（or 未使用なら "Test" 条件を整理してから）

### 2-5. 右半分の keypos → DEFAULT(L0) バインド（ground truth）
```
行0(col8-13):  6=N6  7=N7  8=N8  9=N9  10=N0  11=&lt4 MINUS
行1(col7-13):  19=&lt3 T  20=Y  21=U  22=I  23=O  24=P  25=&lt2 LBKT
行2(col7-13):  33=&lt2 G  34=H  35=J  36=K  37=L  38=&lt3 SEMI  39=&lt1 APOS
行3(col7-13):  47=&lt1 B  48=N  49=M  50=&mkp LCLK  51=lt_mouse(5,MB3)  52=&mkp RCLK  53=RSHIFT
行4(thumb等):  61=BACKSPACE  62=&lt2 ENTER  63=&lt1 RIGHT_ALT  64=RCTRL
```
（左半分 = keypos 0–5, 12–18, 26–32, 40–46, 54–60。片手運用時は物理的に存在しない。）

---

## 3. 今回の確定事項（ユーザー回答済み）

1. **Space/ミラー兼用キー = keypos 62**（現状 `&lt 2 ENTER`）。
   - タップ = `SPACE`、ホールド = MIRROR 層モメンタリ。
   - これは Half-QWERTY 本家の「スペース押しっぱで鏡像」と同じ作法。
   - 右半分には現状 Space が無い（左親指 keypos 59 にあるため）。このキーで Space 供給も兼ねる。
2. **`F R UP A P` ブロックは現状のレイヤー（FUN/SYM/SCROLL）にだけ残す。**
   MIRROR 層には入れない・複製しない。（中身の意味には踏み込まない方針）

---

## 4. 実装プラン（Claude Code 向け・主タスク）

対象: `config/glove65tb.keymap`

### 4-1. レイヤー追加（index は 8 以降。2-4 の理由）
```c
#define ONEHAND 8   // 片手ベース層（&tog で入る）
#define MIRROR  9   // ミラー層（ONEHAND 中に keypos62 ホールドで momentary）
```

### 4-2. ONEHAND（8）= 片手ベース層
- `&tog 8` で入る／抜ける。トグルキーは右手だけで届く場所に。
  - 推奨: keypos 63+64 あたりのコンボ → `&tog 8`（既存コンボ作法 50+51 と同様）。要相談。
- **差分だけ置き、他は `&trans`**（トグル層なので trans は L0 へフォールスルー＝右側の通常文字を継承できる）。
  - keypos 62 → `&lt MIRROR SPACE`（= `&lt 9 SPACE`。tap=Space / hold=MIRROR）
  - keypos 61 → `BACKSPACE`（維持）
  - Enter の置き場所が未決（62 が Space になるため）。候補: MIRROR 層側のどこか／別コンボ／keypos 64 を hold-tap 化。→ 5. の Open items 参照。

### 4-3. MIRROR（9）= 鏡像アルファ層（momentary）
keypos 62 ホールド中だけ有効。**右手の各キーが「左右対称の相手＝左半分の文字」を出す。**
他は基本 `&trans`。`F R UP A P` は入れない（3-2）。

QWERTY のミラーペア（左端↔右端）:
```
上段: Q↔P  W↔O  E↔I  R↔U  T↔Y
中段: A↔;  S↔L  D↔K  F↔J  G↔H
下段: Z↔/  X↔.  C↔,  V↔M  B↔N
```

右手キー（keypos）→ MIRROR で出す文字:
```
上段:  20(Y)→T   21(U)→R   22(I)→E   23(O)→W   24(P)→Q
中段:  34(H)→G   35(J)→F   36(K)→D   37(L)→S   38(;)→A
下段:  48(N)→B   49(M)→V   50→C      51→X      52→Z
```
- 下段の 50/51/52 は L0 ではマウスボタン。MIRROR は momentary なので、その間だけ C/X/Z に
  上書きされる（問題なし）。
- col7（19/33/47）は L0 で既に T/G/B（＝左 index 列）。MIRROR では `&trans` のままで
  ONEHAND→L0 を継承させれば T/G/B が出て整合する。20/34/48 にも T/G/B を割当てるので
  index 文字が二重に打てる形になるが実害なし。気になるなら col7 側を別用途にしてもよい。要相談。

### 4-4. 動作確認・反映
- push すると `.github/workflows/draw.yml` が keymap-drawer の SVG を再生成する。図で配列を目視確認。
- まず `ZMK Studio`（USB 有線・ライブ編集）で MIRROR の出力を試すと速い。
- ⚠ README 記載の注意: 「ビルド経由の変更」と「ZMK Studio の変更」を併用すると競合することがある。
  おかしくなったら `settings_reset.uf2`（右→左の順）で解消。
- ファーム書き込みは右=`glove65tb_R.uf2` / 左=`glove65tb_L.uf2`、reset ボタン2回でブートモード。

---

## 5. 未決定・要確認（CC かユーザーと詰める）

1. **片手モードの入り方**: `&tog 8`（推奨・トグル）か、「L0 から直接ミラーをホールド」だけで足りるか。
2. **Enter の置き場所**（keypos 62 が Space/Mirror になるため）。
3. **片手モードでの数字・記号入力**: 既存 NUM(1) のテンキーは左半分依存で右手だけでは届きにくい。
   - 暫定: 数字は上段右 6/7/8/9/0 が native。1–5 は当面諦めるか、MIRROR/別層で上段をミラーする拡張も可。
   - 優先度は「軽いタイピング」想定なので低め。後回し可。
4. **ミラータイピングの想定用途**: URL・検索ワード・短文で合っているか（長文を打つ気があるなら配列の追い込みを増やす）。
5. **トグルキーの物理位置**（4-2 のコンボ位置）。

---

## 6. やらないこと / 禁止事項
- DEFAULT(L0) を含む既存の両手用バインドは変更しない（差分は新レイヤー側に閉じる）。
- `F R UP A P` ブロックを MIRROR に複製しない。
- layer_7(index 7) を MIRROR に流用しない（NUM+FUN の条件レイヤー行き先のため）。

---

## 7. 完了の定義（DoD）
- ONEHAND(8) と MIRROR(9) が追加され、keymap-drawer SVG が正しく再生成される。
- 右半分だけで: ①トラックボールでカーソル/スクロール ②keypos62 ホールドで A–Z が右手のみで打てる
  ③tap で Space、61 で Backspace が打てる。
- 両手用 L0 の挙動が変わっていないことを確認。
