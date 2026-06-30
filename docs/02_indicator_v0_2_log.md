# Indicator v0.2 Log

## 開発テーマ

HTF EMA200 Touch / LTF EMA Break

上位足200EMAに価格が接触した後、反発を確認し、下位足EMA20/50ブレイクでエントリー候補を出す。

---

## v0.1

初期版。

### 判定

- HTF EMA200を表示
- LTF EMA20 / EMA50を表示
- HTF EMA200への接触を検出
- 接触足の終値がHTF EMA200より上ならLong Setup
- 接触足の終値がHTF EMA200より下ならShort Setup
- Setup後、一定本数内にLTF EMAをブレイクしたらTrigger

### 問題

`longTouch = low <= htfEma` のため、価格がHTF EMA200より下にあるだけでLTが大量発生した。

---

## v0.1.1

タッチ判定を修正。

### 変更点

```pine
htfEmaTouched = low <= htfEma and high >= htfEma
```

5分足の高値〜安値の中にHTF EMA200が入っている場合だけ、タッチと判定。

### 結果

LT大量発生は解消。

### 残課題

接触足の終値でLong / Shortを決めるため、裁量的にはLong Setupに見える場面でも、終値がHTF EMA200の下に出るとShort Setupになる。

---

## v0.2

Setup方向を「接触足の終値」ではなく、「HTF EMA200へどちら側から接近したか」で決める方式に変更。

### 新判定

```text
上からHTF EMA200へ接触 = Long Setup
下からHTF EMA200へ接触 = Short Setup
```

### Long Setup

```pine
close[1] > htfEma[1]
and
low <= htfEma
and
high >= htfEma
```

### Short Setup

```pine
close[1] < htfEma[1]
and
low <= htfEma
and
high >= htfEma
```

---

## v0.2 の目的

裁量的に以下のような場面をLong Setupとして扱う。

```text
価格がHTF EMA200の上側から下落
↓
HTF EMA200に接触
↓
サポート反発候補
↓
LTF EMA20/50ブレイクでLong Trigger
```

逆に、価格がHTF EMA200の下側から上昇して接触した場合はShort Setupとする。

---

## v0.2 の追加仕様

一度Setupが発生したら、TriggerまたはExpireまで反対方向のSetupを無視する。

これにより、HTF EMA200付近でもみ合った場面で以下のような連発を防ぐ。

```text
LS
SS
LS
SS
```

---

## 初期設定

| Item | Value |
|---|---|
| Chart TF | 5m |
| HTF | 1H |
| HTF EMA | 200 |
| LTF EMA | 20 / 50 |
| Trigger EMA | 20 or 50 |
| Expire | 12 bars |
| Setup Direction | Approach Direction |
| Reset | Trigger or Expire |

---

## 確認ポイント

- 上からEMA200に触れた場面がLong Setupになるか
- 下からEMA200に触れた場面がShort Setupになるか
- EMA200付近でもみ合った時にLS/SSが連発しないか
- Setup有効中の背景表示が自然か
- Triggerが遅すぎないか
- Trigger EMAは20と50のどちらが自然か

---

## 次の作業

v0.2をチャートで確認する。

確認後、必要に応じて以下を検討する。

- Trigger条件の改善
- Setup足高値/安値ブレイクTrigger
- EMA20/50並びフィルタ
- HTF EMA200傾きフィルタ
- Strategy v1.0化

## v0.2


v0.2のApproach Direction方式：採用

2枚目の理想形：再現OK

赤矢印：Trigger位置フィルタで修正

黄色矢印：Cooldownで軽減、必要なら次にHTF EMA傾きフィルタ

## v0.2.1

v0.1
タッチ判定が甘く、LT大量発生

v0.1.1
実タッチ判定に修正
ただし、接触足終値でLong/Shortを決めるため裁量とズレる場面あり

v0.2
Approach Direction方式に変更
上からEMA200へ接触 = Long Setup
下からEMA200へ接触 = Short Setup

v0.2.1
Trigger位置フィルタ追加
LongはHTF EMA200より上のみ
ShortはHTF EMA200より下のみ
Trigger後Cooldown追加
