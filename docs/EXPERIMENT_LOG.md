# 實驗紀錄

每次正式實驗新增一列。沒有 commit、設定或結論的執行不列為有效實驗。

| Exp ID | 日期 | Owner | Git commit | Unit set | Mode | Model／Prompt | Draws | Seed | Runtime | Gates | 主要結果 | 結論／下一步 |
|---|---|---|---|---|---|---|---:|---:|---:|---|---|---|
| E000 | YYYY-MM-DD | TBD | `abcdef0` | example | numeric_only | baseline-v0 | 500 | 42 | TBD | TBD | 初始 smoke test | TBD |

## 實驗前檢查

- 實驗問題是否能用一句話回答？
- 是否只有一個主要變因？
- 是否有 numeric-only 或上一最佳版本作為 control？
- 是否記錄 seed、模型版本、prompt 版本與 draw 數？
- 是否使用合法的 `asof` 資訊？

## 實驗後必填

- 結果是否可重現？
- 改善的是中心、分散、joint 還是 tail？
- Runtime／memory 是否惡化？
- 決定：採用、拒絕、或需要下一個實驗。

