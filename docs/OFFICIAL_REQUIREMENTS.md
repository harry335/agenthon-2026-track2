# Agenthon 2026 Track 2 官方規格摘要

> 最後核對：2026-08-30（Asia/Taipei）
> 官方 Track 2 source commit：`639614b61db1952834787d2d6b3632330bdf2a19`（2026-08-29）

本文件濃縮實作、測試與提交時會直接影響成敗的規格。若本文件與官方來源衝突，以官方網站、規則及 public repository 的最新版本為準。

## 1. 官方來源

- [Agenthon 2026 Protocol 與時間表](https://www.agenthon.net/#protocol)
- [Track 2 public repository](https://github.com/Agenthon-2026/track2-forecasting-public)
- [Track 2 participant guide](https://github.com/Agenthon-2026/track2-forecasting-public/blob/main/README.md)
- [Submission CLI contract](https://github.com/Agenthon-2026/track2-forecasting-public/blob/main/SUBMISSION_CLI.md)
- [F1–F4 card families](https://github.com/Agenthon-2026/track2-forecasting-public/blob/main/docs/CATEGORIES.md)
- [Scoring concepts](https://github.com/Agenthon-2026/track2-forecasting-public/blob/main/docs/CONCEPTS.md)

## 2. 競賽時間

| 階段 | 日期 | 意義 |
|---|---|---|
| Registration | 2026-08-17～09-28 | 最晚 09-28 完成團隊註冊 |
| Development | 2026-08-28～09-28 | Public practice、迭代與 validation leaderboard |
| Final | 2026-09-29～10-12 | 每個參賽 track 只有一次 final submission；sealed units |
| Verification | 2026-10-13～10-25 | 主辦方以新 seed 重跑頂尖 submission 並檢查可重現性 |
| NeurIPS | 2026-12-09～12-13 | Atlanta workshop；確切 session 待公告 |

團隊可有 1～3 人。只有已註冊團隊能取得 starter package、提交與進入 leaderboard。

## 3. 任務與評分

Track 2 使用數值 time-series panels 加上截至 `asof` 的 frozen text corpus，輸出未來結果的完整機率分布，以 Monte Carlo draws 表示。

多 cell 卡的 composite：

```text
S = 0.5 × marginal CRPS + 0.3 × joint variogram + 0.2 × tail penalty
```

分數越低越好。單 cell 卡沒有 joint term，因此權重改為 `0.714 × CRPS + 0.286 × tail`。

- 每張卡的 normalized text-blind baseline 為 `1.0`。
- 每張卡等權重。
- 漏跑、錯誤或 inadmissible 的卡記 `4.0`，沒有值得跳過的卡。
- Text uplift 是科學診斷，不是獨立的排名項目；仍應固定執行 full-text／text-ablated 對照。

### Card families

| Family | 核心問題 | 主要壓力 |
|---|---|---|
| F1 Continuation-with-context | 文字能否替熟悉序列增加增量訊號 | Marginal CRPS |
| F2 Text-cued regime shift | 文字能否在數字改變前辨識 regime shift | CRPS + tail |
| F3 Cross-asset reasoning | 文字能否推導相關市場並維持 joint dependence | Joint variogram |
| F4 Tail/shock-from-text | 文字能否提前反映 shock 與 fat tails | Tail penalty |

Public repository 有 103 個 practice units：F1 23、F2 27、F3 22、F4 31。Practice units 沒有 realized outcomes，因此本機只能驗證執行與 g0–g3，不能算正式 CRPS composite。

## 4. Docker 與 CLI 合約

Image 必須實作：

```bash
forecast \
  --panels /input/panels/ \
  --text /input/text/ \
  --asof YYYY-MM-DD \
  --out /output/forecast.parquet
```

Harness 會將整個 unit directory 以唯讀方式掛載到 `/input`，而不是分別掛載 panels 與 text。Image 必須：

- 接受 `forecast` 作為 image 後的第一個參數，或將它做成 `PATH` 上的 executable。
- 成功時 exit code 為 `0`。
- 包含 `LABEL qfbench2.interface_version="2.0"`。
- 只讀 `/input`，只寫 `/output`。
- 能在沒有實體 GPU 的 worker 啟動或安全 fallback。
- 使用 Python `>=3.13`，與官方 package 及 CI 對齊。

## 5. 必要輸出

三個檔案必須位於同一個 output directory：

### `forecast.parquet`

| 欄位 | 型別 | 規則 |
|---|---|---|
| `draw` | int32 | 從 0 開始的 sample index |
| `asset` | string | 必須完全符合 card target asset ID |
| `horizon` | int32 | business days，不是 calendar days |
| `value` | float64 | 使用 card 指定單位，不可有 NaN／Inf |

- 至少 200 draws；一般建議 500+，F4 建議 1,000+。
- F3／F4 的同一 draw 必須包含所有 target assets 與 horizons。
- 不可逐資產獨立抽樣後任意拼接成 joint forecast。

### `forecast_meta.json`

至少包含：

```json
{
  "unit_id": "exact-card-task-id",
  "asof": "YYYY-MM-DD",
  "asset_ids": ["..."],
  "horizons": [21],
  "representation": "samples",
  "n_draws": 500
}
```

`unit_id` 與 `asof` 必須完全符合 `card.toml`。Metadata 使用 `unit_id`，不可寫成 scorer report 使用的 `card_id`。

### `forecast_rationale.md`

- 必須存在且不可空白，否則 g1 失敗。
- 不會進入數值評分，但會供人工 review。
- 應記錄 numeric anchor、文字造成的調整、scale／shape、依據文件與日期，以及最後 adjustment ledger。

## 6. Admissibility gates

只有通過 g0–g3 才會計算排名分數：

| Gate | 重點 |
|---|---|
| g0 integrity | image、manifest、執行與完整性 |
| g1 schema | parquet schema、metadata、非空白 rationale |
| g2 cutoff/resource | `asof`、text timestamps、網路與資源限制 |
| g3 domain semantics | draw 數、完整 cells、非退化分布、joint／tail 合理性 |

本機 smoke scorer：

```bash
qfbench2-smoke units/t2-EXAMPLE-ust-curve-1m/ output/ --track forecasting
```

或使用官方 scorer：

```bash
python scoring/scoring.py score \
  --card units/t2-EXAMPLE-ust-curve-1m/card.toml \
  --forecast output/forecast.parquet
```

沒有 `--realized` 時只會回報 gates，不會產生可比較的正式分數。

## 7. 網路與模型

Official scoring 使用 `restricted` network，沒有 open internet。唯一允許的 egress 是 organizer audited proxy 上的 house model endpoint。

- `api.openai.com`、Anthropic、Google 等 vendor API 都會被拒絕。
- Harness 不提供、也沒有機制傳入 participant API key。
- Runtime 由環境提供 `HTTP_PROXY`、`HTTPS_PROXY`、`NO_PROXY`、`MODEL_ENDPOINT`、`MODEL_NAME`、`QFBENCH_NETWORK`；不可 hardcode proxy host。
- 所有 model calls 必須停用 vendor-side web search、code execution 與 retrieval。
- 本機 structural smoke run 使用 `--network=none`。

Submission category 必須三選一：

| Category | 模型方式 |
|---|---|
| `api` | 使用 organizer house endpoint；CPU tier |
| `byo-small` | image 內自帶約 ≤8B 權重；CPU 或 small GPU |
| `byo-large` | image 內自帶大型權重；80GB-class GPU tier |

所有 category 共用同一 leaderboard。官方 provisional API budget 為每 unit 1,000,000 input + 100,000 output tokens；在正式公告前不得當成最終值。

## 8. 資源與時間

每張 Track 2 card 目前宣告：

- 16 vCPU
- 128 GB memory
- `gpu = true`，但不保證 worker 一定掛載實體 GPU
- 1,800 秒 per-unit ceiling
- Development 全 submission 43,200 秒（12 小時）
- Final／Verification 全 submission 86,400 秒（24 小時）

103 units 在 Development 平均只有約 419 秒／unit。必須依 phase budget 設計，不可把 1,800 秒當成每題可用預算。Docker image cold pull 也計時，因此 image 大小會直接消耗預算。

## 9. 可重現性與 leakage

- 模型版本與 bundled weights revision 必須固定。
- 揭露每個模型的 training cutoff。
- API 支援時固定 temperature 與 seed；讀取 harness 提供的 `QFBENCH_SEED`。
- 固定 Docker image digest。
- Panel 與 text 皆不可使用 `asof` 後資料。
- 推論時不可下載市場資料、新聞或其他外部資料。
- 不可從其他 practice unit 把特定 target answer 帶入本 unit；practice data 可訓練／調參，但不可建立跨 unit 答案查找表。

## 10. 官方 reference 的已知注意事項

截至上述 source commit：

- Python CLI 可在本機跑通 exemplar，產生三個檔案並通過 g0–g3。
- 官方 root Dockerfile 可 build，但仍未安裝 `qfbench2-common`，container 執行時會出現 `ModuleNotFoundError`；自己的 Dockerfile 必須補上固定版本。
- 必須固定：

```text
qfbench2-common @ git+https://github.com/Agenthon-2026/Agenthon2026-public.git@v2.3.1#subdirectory=common
```

- 不可安裝 branch head；`v2.3.1` 是目前官方指定、包含必要 contracts 的版本。

## 11. 第一個可驗收目標

在模型研究開始前，先完成：

1. 固定官方 source commit 與 `qfbench2-common@v2.3.1`。
2. 使用 Python 3.13 跑通官方 exemplar。
3. 修正自己的 Docker image，加入 interface label 與 shared toolkit。
4. 使用單一 `/input` 唯讀 mount 執行 `--network=none`。
5. 產生三個必要輸出並讓 g0–g3 全部 pass。
6. 記錄 image digest、runtime、memory、seed 與完整命令。
