# 決策紀錄

重大架構、模型、依賴與提交決策都在此記錄，避免口頭決定失去背景。

## D000 — 初始架構

- 日期：YYYY-MM-DD
- 狀態：Proposed
- Owner：TBD
- 決策：Numeric Forecaster 與 Text Reasoner 分離，由 Distribution Combiner 將結構化文字訊號轉成位置、尺度、偏態與尾部調整。
- 原因：可獨立測試、可做 text ablation、LLM 失敗時可 fallback。
- 替代方案：LLM 直接輸出所有 Monte Carlo draws。
- 代價：需要維護 event signal schema 與 combiner。
- 重新評估條件：若 text signal 無法可靠映射，或正式 API 限制改變。

## 新決策模板

- 日期：
- 狀態：Proposed / Accepted / Superseded
- Owner：
- 決策：
- 原因：
- 替代方案：
- 代價：
- 重新評估條件：

