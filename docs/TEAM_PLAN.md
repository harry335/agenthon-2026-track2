# Track 2 三人團隊總計畫

## 0. 已確認時程與尚待決策

| 項目 | 目前狀態 |
|---|---|
| 成員姓名與專長 | TBD |
| Registration | 2026-08-17～09-28 |
| Development | 2026-08-28～09-28 |
| Final | 2026-09-29～10-12；每 track 一次 final submission |
| Verification | 2026-10-13～10-25 |
| 每週可投入時數 | TBD |
| 可用本機／雲端 GPU | TBD |
| Submission category | TBD：`api`／`byo-small`／`byo-large` |

官方規格最後核對日期為 2026-08-30；詳細來源與實作合約見 [OFFICIAL_REQUIREMENTS.md](OFFICIAL_REQUIREMENTS.md)。

## 1. 專案目標

交付一個可重現的 Docker forecasting agent，讀取截至 `asof` 的金融 panel 與文字 corpus，輸出校準良好的 Monte Carlo 聯合分布，並在所有 sealed cards 穩定完成。

成功優先順序：

1. 零結構性 DNF：所有輸出通過 g0–g3。
2. 零漏題：在 phase wall-clock 內完成全部 cards。
3. Numeric baseline 穩定：不依賴文字也能合理預測。
4. Joint calibration：保留跨資產、跨 horizon 的共同變動。
5. Text uplift：文字能調整方向、波動、偏態與 tail risk。
6. 可重現：固定依賴、模型版本、training cutoff、seed 與 temperature。

## 2. 不可違反的硬性規格

- Container 必須接受 `forecast --panels ... --text ... --asof ... --out ...`。
- Image 必須包含 `LABEL qfbench2.interface_version="2.0"`，並以 Python `>=3.13` 對齊官方 package。
- 必須產生 `forecast.parquet`、`forecast_meta.json`、非空白的 `forecast_rationale.md`。
- `forecast_meta.json` 使用 `unit_id`，不是 `card_id`。
- 最少 200 draws；預設 500；F4 預設 1,000，除非效能測試顯示無法負擔。
- Joint cards 的同一 draw 必須包含全部 targets，不可逐資產獨立抽樣後任意拼接。
- 正式環境不得擷取外部市場／新聞資料；只能呼叫主辦方 audited proxy 上的 house model endpoint，第三方 vendor API 不可用。
- 模型必須能在沒有實體 GPU 的 worker 上正常啟動或安全 fallback。
- `qfbench2-common` 固定為官方要求版本；變更前必須留下決策紀錄。
- 每張卡上限為 16 vCPU、128 GB、1,800 秒，但 Development 全 submission 只有 12 小時，設計目標應低於約 419 秒／unit。

## 3. 三人角色與責任

將 Person 1–3 替換為真實姓名。每個工作項目只有一位最終負責人。

| 角色 | 暫定成員 | Accountable | 主要交付 |
|---|---|---|---|
| PM／整合負責人 | Person 1 | 排程、規格、整合、提交 | Roadmap、interface contract、release candidate |
| Forecasting Lead | Person 2 | 數值模型與分布品質 | Numeric baseline、uncertainty、joint sampler |
| Text Reasoning Lead | Person 3 | 文字訊號與事件推理 | Corpus parser、event schema、text ablation |

交叉 review：

- Forecasting PR：Text Lead 或 PM review。
- Text reasoning PR：Forecasting Lead review其量化語意。
- Docker／submission PR：至少另一位成員在不同機器重跑。
- Release candidate：三人共同簽核。

## 4. 模組邊界

```text
Panel Loader ──> Numeric Forecaster ───────────────┐
                                                   ├─> Distribution Combiner
Text Loader ──> Text Reasoner ─> Event Signal ────┘
                                                        │
                                                        v
                                               Joint/Tail Sampler
                                                        │
                                      forecast + metadata + rationale
```

Text Reasoner 不直接寫最終 forecast，先輸出可測試的結構化訊號：

```json
{
  "regime": "hawkish",
  "direction": {"UST_2Y": 1.0, "UST_10Y": 0.4},
  "location_shift": {},
  "scale_multiplier": 1.3,
  "tail_event_probability": 0.08,
  "confidence": 0.72,
  "evidence_ids": []
}
```

在介面定版前，任何成員不得各自創造不相容的訊號格式。

## 5. 里程碑與退出條件

### M0 — 團隊啟動（第 0～2 天）

- 填完成員、期限、時數與硬體資訊。
- 三人確認角色與 review 規則。
- 官方 repository 固定到明確 commit/tag。
- 固定 Python 3.13、`qfbench2-common@v2.3.1` 與 Docker interface version 2.0。
- 建立 GitHub Project：Backlog / This Week / In Progress / Review / Done。

退出條件：每個 P0 工作都有 owner、deadline 與驗收條件。

### M1 — Golden Pipeline（2–3 天）

- 官方 example 在 Python 與 Docker 中跑通。
- 正確安裝 pinned `qfbench2-common`。
- 產生三個必要輸出。
- 通過 g0–g3。
- 用 `--network=none` 完成 end-to-end smoke run。

退出條件：兩台不同電腦從乾淨 checkout 可依 README 重現。

### M2 — Numeric Safety Baseline（第 1 週）

- 所有 practice units 都能完成。
- 無 schema、NaN、資產遺漏、horizon 遺漏。
- 記錄每題與整批 wall time、peak memory、draw 數。
- 建立簡單但 coherent 的 joint sampling。

退出條件：103 units 零 DNF，且預估能在 phase budget 內完成。

### M3 — Text MVP（第 2 週）

- Corpus parser 能處理空資料、壞文件與重複文件。
- Text Reasoner 輸出固定 schema。
- 提供 `numeric_only` / `full_text` 開關。
- F1–F4 各選代表題完成 qualitative sanity check。

退出條件：關閉文字時輸出等同 baseline；開啟文字時差異可解釋並寫入 rationale。

### M4 — Calibration & Robustness（第 3 週起）

- F3：改善跨資產 covariance／共同 shock。
- F4：建立 mixture 或 heavy-tail sampling。
- 對 draw 數、scale、tail probability 做敏感度分析。
- 模擬 API timeout、無 GPU、空 corpus、單資產、超大 target set。

退出條件：完整 regression 無 DNF，錯誤時能 fallback 至 numeric baseline。

### M5 — Release Candidate（Final 前至少 5 天）

- 冻結新功能，只修 correctness 或 reliability 問題。
- 固定 image digest、依賴、模型與 prompts。
- 三人各自完成一次 clean-room run。
- 完成提交檢查表與 model disclosure。

退出條件：三人一致同意提交同一 image digest。

## 6. 初始 Backlog

### P0：本週必做

| ID | 工作 | Owner | 驗收標準 |
|---|---|---|---|
| P0-01 | 確認競賽期限與 submission category | PM | 文件內有來源與確認日期 |
| P0-02 | 固定官方 repo commit 與共同 toolkit | PM | Lockfile／Docker 使用固定版本 |
| P0-03 | 跑通官方 Python example | Forecast | 三檔產生且 g0–g3 pass |
| P0-04 | 修好並跑通 Docker example | Integrator | `--network=none` pass |
| P0-05 | 建立 103-unit batch runner | Integrator | 輸出成功率、時間與錯誤摘要 |
| P0-06 | 建立 numeric baseline | Forecast | 全題可輸出有效 samples |
| P0-07 | 盤點 text corpus 格式 | Text | 文件類型、metadata、異常案例清單 |
| P0-08 | 定義 event signal schema | 全員；Text 執筆 | 三人 review 通過 |

### P1：Pipeline 穩定後

- Joint covariance sampler。
- F4 tail mixture sampler。
- Text ablation runner。
- API timeout／rate-limit fallback。
- Prompt 與模型版本 registry。
- Runtime budget dashboard。
- Representative unit regression set。

### 暫不做

- 在 pipeline 未穩定前嘗試大型多代理架構。
- 只為 Development leaderboard 排名做特定 unit lookup。
- 無法在無 GPU 環境 fallback 的大型模型方案。
- 沒有 ablation 或可重現設定的 prompt tweaking。

## 7. 每週工作節奏

- 每日 15 分鐘：完成項目、今日目標、阻塞點、是否影響 milestone。
- 週一：選定本週最多 3 個團隊級成果。
- 週三：架構與實驗 review；停止沒有可驗收結論的支線。
- 週五：完整 regression、更新風險表、決定下週優先順序。
- PR 盡量小於 400 行實質變更；超過時先拆介面與實作。

## 8. 風險表

| 風險 | 機率 | 衝擊 | 預防／fallback | Owner |
|---|---|---|---|---|
| Docker 本機可跑、正式環境失敗 | 中 | 極高 | offline smoke、pinned deps、第二台機器重跑 | PM |
| 全體超過 phase 時限 | 中 | 極高 | batch profiling、numeric fallback、限制 API calls | Integrator |
| LLM 輸出無法穩定解析 | 高 | 高 | JSON schema、重試一次、失敗回 numeric baseline | Text |
| F3 資產獨立抽樣 | 中 | 高 | shared latent shock、covariance tests | Forecast |
| F4 tail 過窄 | 高 | 高 | mixture distribution、增加 draws、tail calibration | Forecast |
| 文字沒有真實 uplift | 中 | 中 | mandatory ablation、低信心時縮小 text adjustment | Text |
| 時間洩漏／跨 unit lookup | 低 | 極高 | asof tests、code review、禁止 target lookup cache | 全員 |
| 只有一台電腦能建置 | 中 | 高 | clean-room run、文件化命令 | PM |

## 9. PM 每週儀表板

每週只追以下指標：

- Practice completion rate。
- g0–g3 pass rate。
- 全批預估 wall time。
- P50／P95 單題 runtime。
- Numeric-only 與 full-text 的差異覆蓋率。
- F3 joint consistency tests pass rate。
- F4 tail sample effective count。
- 可重現的有效實驗數，而非總實驗數。
- 尚未關閉的 P0 blockers。

## 10. Definition of Done

工作只有在以下條件全部成立才算完成：

- 程式已合併且有 review。
- 有自動測試或明確的重現命令。
- 更新相關文件。
- 實驗有 commit、設定、runtime、結果與結論。
- 不降低既有 gates pass rate。
- 若改動 forecast distribution，已跑 numeric-only/full-text 對照。
