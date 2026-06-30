# HTF EMA200 Touch / LTF EMA Break Concept

## 1. 手法概要

この手法は、上位足の200EMAに価格が接触した後、反発を確認し、下位足のEMA20またはEMA50をブレイクしたところでエントリーを検討する。

狙いは、上位足200EMAを押し目・戻り目の基準として使い、下位足EMAブレイクをトリガーとして機械的にエントリー判定すること。

---

## 2. 開発目的

TradingView Pine Script v6で以下の順に開発する。

```text
Indicator
↓
Strategy
↓
Backtest
↓
Parameter Test
↓
EA Specification
↓
MQL5 EA
↓
Demo Forward
```

最初はインジケーターとしてSetupとTriggerを可視化し、裁量イメージと合っているかを確認する。

その後、Strategy化してバックテストを行い、優位性が確認できた場合はMT5 EA化を目指す。

---

## 3. 初期仕様

| 項目 | 内容 |
|---|---|
| 手法名 | HTF EMA200 Touch / LTF EMA Break |
| Chart TF | 5m |
| HTF | 1H |
| HTF EMA | 200 |
| LTF EMA | 20 / 50 |
| Setup | HTF EMA200接触後の反発 |
| Trigger | LTF EMA20/50ブレイク |
| Entry | Trigger確定後 |
| Expire | Setup後12本 |
| Long / Short | 両方検証 |
| First Version | Indicator v0.1 |

---

## 4. ロング条件の初期定義

### Long Setup

```text
HTF EMA200に価格が接触
かつ
終値がHTF EMA200より上に戻る
```

初期案：

```text
low <= HTF EMA200
and
close > HTF EMA200
```

または、HTF値をLTF上に展開して、LTF足ベースで判定する。

### Long Trigger

```text
Long Setup発生後、一定期間内にLTF closeがLTF EMA20を上抜け
```

初期案：

```text
ta.crossover(close, LTF EMA20)
```

後続バージョンで、以下のような強化条件も検討する。

```text
close > EMA20
and
close > high[1]
```

---

## 5. ショート条件の初期定義

### Short Setup

```text
HTF EMA200に価格が接触
かつ
終値がHTF EMA200より下に戻る
```

初期案：

```text
high >= HTF EMA200
and
close < HTF EMA200
```

### Short Trigger

```text
Short Setup発生後、一定期間内にLTF closeがLTF EMA20を下抜け
```

初期案：

```text
ta.crossunder(close, LTF EMA20)
```

後続バージョンで、以下のような強化条件も検討する。

```text
close < EMA20
and
close < low[1]
```

---

## 6. Indicator v0.1 の目的

Indicator v0.1では、売買検証は行わない。

目的は以下。

```text
1. HTF EMA200を表示する
2. LTF EMA20 / EMA50を表示する
3. HTF EMA200接触をマークする
4. 反発確認をマークする
5. Setup有効状態を可視化する
6. Trigger発生位置に矢印を表示する
```

この段階では、SL / TP / RR はまだ扱わない。

---

## 7. Indicator v0.1 の確認ポイント

チャート上で以下を確認する。

```text
自分が入りたい場所にTriggerが出ているか
Setupが遅すぎないか
余計なTriggerが多すぎないか
EMA200接触判定が厳しすぎないか
EMA200接触判定が緩すぎないか
Long / Short のどちらに偏りがあるか
```

目視でズレている場合、Strategy化する前にIndicator条件を調整する。

---

## 8. Strategy v1.0 の予定仕様

Indicator v0.1 / v0.2でシグナル位置に納得できたらStrategy化する。

初期予定：

| 項目 | 内容 |
|---|---|
| HTF | 1H |
| LTF | 5m |
| HTF EMA | 200 |
| Trigger EMA | 20 |
| Expire | 12 LTF bars |
| SL Mode | Setup Candle |
| TP Mode | RR固定 |
| RR | 1.0 / 1.3 / 1.5 / 2.0 |
| Position | 同時1ポジション |
| Direction | Long / Short 両方 |

---

## 9. SL候補

Strategy化時に、以下を比較候補にする。

```text
SL Mode A: Setup Candleの反対側
SL Mode B: Trigger前の直近スイング
SL Mode C: ATR基準
SL Mode D: 固定pips
```

初期版では、EA化しやすい `Setup Candle` を優先する。

---

## 10. TP候補

初期版ではRR固定で検証する。

```text
RR 1.0
RR 1.3
RR 1.5
RR 2.0
```

---

## 11. パラメータ探索候補

最初から探索範囲を広げすぎない。

第1段階の候補：

| Parameter | Values |
|---|---|
| HTF | 1H |
| LTF | 5m |
| HTF EMA | 200 |
| Trigger EMA | 20 / 50 |
| Expire | 6 / 12 / 24 |
| SL Mode | Setup Candle / ATR |
| RR | 1.0 / 1.3 / 1.5 / 2.0 |

---

## 12. 改善フィルタ候補

初期Strategyで優位性が弱い場合、以下を順に検討する。

```text
HTF EMA200の傾きフィルタ
HTF close と EMA200 の位置関係
LTF EMA20 / EMA50 の並び
時間帯フィルタ
曜日フィルタ
経済指標停止
ATRボラティリティフィルタ
```

---

## 13. EA化を見据えた設計方針

EA化を前提に、Pine段階から以下を守る。

```text
確定足ベースで判定する
未確定足に依存しない
request.securityはlookahead_off
Entry / SL / TP を明確な価格で出す
Setup理由とTrigger理由を分ける
配列ロジックは必要最小限にする
TradingViewでしか再現しにくい処理を避ける
```

---

## 14. Coding Rules

Pine Script v6を書くときは、`PINE202605-TR` の「ハマり集」を遵守する。

特に以下を重視する。

```text
1行1文
セミコロン禁止
短絡評価に頼らない
配列アクセス前にサイズ確認
forループ前に start <= end を確認
複数配列は min(size) で境界統一
delete後は := na
pine_check後にTradingView実機確認
```

---

## 15. 次の作業

次は `Indicator v0.1` を作成する。

作成予定ファイル：

```text
src/htf_ema200_ltf_ema_break/indicator_v0_1.pine
```

Indicator v0.1の目的：

```text
HTF EMA200を表示
LTF EMA20 / EMA50を表示
EMA200接触マーク
反発確認マーク
Setup有効状態
Trigger矢印
```
