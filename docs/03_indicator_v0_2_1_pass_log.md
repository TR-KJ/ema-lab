# Indicator v0.2.1 Pass Log

## 開発テーマ

HTF EMA200 Touch / LTF EMA Break

上位足200EMAに価格が接触した後、下位足EMA20/50ブレイクでエントリー候補を出すインジケーター。

---

## 対象ファイル

src/htf_ema200_ltf_ema_break/indicator_v0_2_1.pine

---

## v0.2.1 の目的

v0.2で改善したApproach Direction方式を維持しつつ、以下の問題を修正する。

1. HTF EMA200より下でLONG Triggerが出る問題
2. HTF EMA200付近でもみ合った時にシグナルがごちゃつく問題
3. v0.1.1で理想的だったLong Setup → LONGの流れを維持すること

---

## v0.2.1 の主な仕様

### Setup方向

Setup方向は、接触足の終値ではなく、HTF EMA200へどちら側から接近したかで判定する。

- 上からHTF EMA200へ接触 = Long Setup
- 下からHTF EMA200へ接触 = Short Setup

### Long Setup

close[1] > htfEma[1]
and
low <= htfEma
and
high >= htfEma

### Short Setup

close[1] < htfEma[1]
and
low <= htfEma
and
high >= htfEma

### Trigger位置フィルタ

Long Triggerは、Trigger足の終値がHTF EMA200より上にある場合のみ有効。

longTriggerPositionOk = close > htfEma

Short Triggerは、Trigger足の終値がHTF EMA200より下にある場合のみ有効。

shortTriggerPositionOk = close < htfEma

### Cooldown

Trigger後、一定本数は新規Setupを禁止する。

初期値：

Cooldown Bars After Trigger = 12

5分足の場合、約60分のCooldown。

---

## 確認結果

### 1. HTF EMA200より下でのLONG誤発火

確認結果：

OK

内容：

v0.2では、上からHTF EMA200に接触した後、価格がHTF EMA200より下にある状態でもLTF EMAを上抜けるとLONGが出るケースがあった。

v0.2.1では、Long Triggerに close > htfEma 条件を追加したため、HTF EMA200より下でのLONG誤発火は解消した。

---

### 2. EMA200付近のごちゃつき

確認結果：

OK

内容：

v0.2では、HTF EMA200付近でもみ合う場面で、SetupやTriggerが密集しやすかった。

v0.2.1では、Trigger後にCooldownを追加したことで、再Setupが抑制され、シグナルのごちゃつきが軽減された。

---

### 3. 理想的なLong Setup → LONGの維持

確認結果：

OK

内容：

v0.1.1で理想的だった、以下の流れはv0.2.1でも維持できている。

価格がHTF EMA200の上側から下落
↓
HTF EMA200に接触
↓
Long Setup
↓
反発
↓
LTF EMA20/50ブレイク
↓
LONG Trigger

---

## 判定

Indicator v0.2.1 は合格。

---

## 合格理由

v0.2.1では、以下をすべて満たした。

1. HTF EMA200より下でのLONG誤発火が消えた
2. EMA200付近のごちゃつきがCooldownで軽減された
3. 理想的なLong Setup → LONGの流れが残った
4. Approach Direction方式が裁量イメージに近い
5. Strategy化に進める最低限のロジック整理ができた

---

## 現時点の採用ロジック

### Setup

- 上からHTF EMA200へ接触したらLong Setup
- 下からHTF EMA200へ接触したらShort Setup
- Setup有効中は反対方向Setupを無視
- SetupはTriggerまたはExpireで解除

### Trigger

- Long Setup有効中にLTF EMAを上抜け
- かつ、終値がHTF EMA200より上
- Short Setup有効中にLTF EMAを下抜け
- かつ、終値がHTF EMA200より下

### Cooldown

- Trigger後、一定本数は新規Setup禁止
- 初期値は12本

---

## 初期パラメータ

| Parameter | Value |
|---|---|
| Chart TF | 5m |
| HTF | 1H |
| HTF EMA | 200 |
| LTF EMA Fast | 20 |
| LTF EMA Slow | 50 |
| Trigger EMA | 20 or 50 |
| Setup Expire Bars | 12 |
| Cooldown Bars After Trigger | 12 |
| Setup Direction | Approach Direction |
| Trigger Position Filter | Enabled |

---

## 次の作業

次は Strategy v1.0 に進む。

予定ファイル：

src/htf_ema200_ltf_ema_break/strategy_v1_0.pine

---

## Strategy v1.0 初期案

### Entry

Indicator v0.2.1 の LONG / SHORT を使用する。

### SL候補

初期案：

- Long SL = Setup発生以降からTrigger足までの最安値
- Short SL = Setup発生以降からTrigger足までの最高値

### TP候補

RR固定で検証する。

- RR 1.0
- RR 1.3
- RR 1.5
- RR 2.0

### Position

同時ポジションは1つのみ。

### 検証目的

まずは以下を確認する。

- 総トレード数
- 勝率
- PF
- 最大DD
- 平均損益
- Long / Short 別の成績
- RR別の成績

---

## 備考

Indicator v0.2.1 は、まだ勝てるロジックとして確定したものではない。

この段階での合格は、裁量イメージとシグナル位置が大きくズレておらず、Strategy化して検証する価値がある、という意味での合格。
