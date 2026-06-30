# ema-lab

EMA系トレード手法の検証・開発用リポジトリ。

このリポジトリでは、TradingView Pine Script v6でインジケーターを作成し、その後Strategy化して検証する。優位性のあるロジックが確認できた場合、最終的にMT5/MQL5 EA化を目指す。

---

## 開発テーマ

### HTF EMA200 Touch / LTF EMA Break

上位足の200EMAに価格が接触した後、反発を確認し、下位足のEMA20またはEMA50をブレイクしたところでエントリーする手法を検証する。

基本イメージ：

```text
上位足200EMAに接触
↓
反発を確認
↓
下位足EMA20/EMA50をブレイク
↓
エントリー候補
```

---

## 開発ロードマップ

```text
Step 1: 手法仕様をMarkdown化
Step 2: Indicator v0.1 作成
Step 3: チャート上でSetup / Triggerを目視確認
Step 4: Indicator v0.2 で条件調整
Step 5: Strategy v1.0 作成
Step 6: パラメータ探索
Step 7: 優位性チェック
Step 8: EA仕様書作成
Step 9: MQL5 EA化
Step 10: デモフォワード検証
```

---

## 初期想定仕様

| Item | Value |
|---|---|
| Chart TF | 5m |
| HTF | 1H |
| HTF EMA | 200 |
| LTF EMA | 20 / 50 |
| Setup | HTF EMA200 touch and rebound |
| Trigger | LTF EMA breakout |
| Expire | 12 LTF bars |
| First Output | Indicator |
| Next Output | Strategy |
| Final Goal | MT5 EA |

---

## ディレクトリ構成予定

```text
ema-lab/
│
├── README.md
│
├── docs/
│   ├── 01_htf_ema200_ltf_ema_break_concept.md
│   ├── 02_indicator_spec.md
│   ├── 03_strategy_spec.md
│   ├── 04_backtest_policy.md
│   └── 05_ea_conversion_notes.md
│
├── src/
│   └── htf_ema200_ltf_ema_break/
│       ├── indicator_v0_1.pine
│       ├── indicator_v0_2.pine
│       ├── strategy_v1_0.pine
│       └── strategy_v1_1.pine
│
└── results/
    └── htf_ema200_ltf_ema_break/
        ├── parameter_log.md
        └── screenshots/
```

---

## Coding Rules

Pine Script v6を書くときは、`PINE202605-TR` の「ハマり集」を遵守する。

特に以下を必ず守る。

- 1行1文、セミコロン禁止
- `request.security()` は `barmerge.gaps_off` / `barmerge.lookahead_off`
- Pineの `or` / `and` に配列アクセスの安全確認を任せない
- 配列アクセス前にサイズ確認
- 複数配列は `min(size)` を境界にする
- `for i = a to b` は `a <= b` を確認してから回す
- `by -1` は使わない
- line / label は delete 後に必ず `:= na`
- `pine_check` 合格後もTradingView実機でRuntime Error確認を行う

---

## 検証方針

最初から完璧なStrategyを作らず、まずはIndicatorでSetupとTriggerの位置を確認する。

優先順位：

```text
1. 裁量イメージとシグナル位置が合っているか
2. 条件が機械的に定義できているか
3. Strategy化して検証できるか
4. パラメータ探索で優位性が残るか
5. EA化しても再現できるか
```

---

## 注意

このリポジトリは検証用であり、掲載コードや検証結果は投資助言ではない。
