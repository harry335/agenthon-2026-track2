# 提交檢查表

## 身分與版本

- [ ] Submission category 已確認。
- [ ] Docker image digest 已固定並由三人核對。
- [ ] 所有模型版本與 training cutoff 已揭露。
- [ ] 依賴使用固定 tag／version，沒有安裝 branch head。
- [ ] Seed、temperature 與 prompt version 已固定。

## CLI 與檔案

- [ ] Image 接受 leading `forecast` verb。
- [ ] `/input` 為 read-only 時可正常運作。
- [ ] 正確寫出指定的 `--out` 路徑。
- [ ] 同目錄存在 `forecast_meta.json`。
- [ ] 同目錄存在非空白 `forecast_rationale.md`。
- [ ] Metadata 使用 `unit_id`，且與 card 完全一致。
- [ ] Asset、horizon、型別與單位完全正確。
- [ ] Draw index 從 0 開始且完整。

## 分布品質

- [ ] 所有 target cells 都有至少 200 draws。
- [ ] Joint cards 每個 draw 都包含所有 target cells。
- [ ] 沒有 NaN、Inf、重複列或漏列。
- [ ] F3 使用 coherent joint sampling。
- [ ] F4 有足夠 draws 與 tail mass。
- [ ] LLM 失敗時 fallback 仍能輸出完整 forecast。

## 合規與資源

- [ ] 沒有讀取 `asof` 之後的資料。
- [ ] 沒有跨 unit 查找特定 target 答案。
- [ ] `--network=none` smoke test 通過。
- [ ] 正式模式只呼叫允許的 model endpoint。
- [ ] 沒有 vendor-side web search／retrieval 工具。
- [ ] 無 GPU 時 container 仍能啟動。
- [ ] Peak memory 低於限制。
- [ ] 全批預估 runtime 低於 phase budget，保留至少 20% 緩衝。

## 最終驗證

- [ ] g0–g3 全部通過。
- [ ] 代表性 F1、F2、F3、F4 regression 全部通過。
- [ ] 103 practice units completion rate 為 100%。
- [ ] 第二台乾淨環境成功 build/run。
- [ ] 三位成員確認提交的是同一 image digest。
