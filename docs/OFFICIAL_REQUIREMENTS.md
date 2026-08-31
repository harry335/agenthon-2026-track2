# Track 2 官方規格

> 最後核對：2026-08-30
> 官方版本：`639614b61db1952834787d2d6b3632330bdf2a19`

## 概述

我們要提交一個 Docker 預測程式。程式讀取截止日以前的金融時間序列與公開文字資料，輸出許多組彼此相關的未來情境，也就是一個機率分布，而不是單一預測值。
正式測試沒有開放網路，只能使用主辦方提供的模型端點，或使用事先包進 Docker 的模型。

## Docker input

主辦方會把整個題目目錄唯讀掛載到 ```/input```：

```/input/
├── card.toml
├── manifest.json
├── panels/
│   └── *.parquet
└── text/
    └── ...
```

主要輸入為：
- ```/input/panels```：截止日以前的金融時間序列。
- ```/input/text```：截止日以前公開的新聞、央行文件及評論。
- ```/input/card.toml```：題目 ID、目標資產、預測期限、單位與環境限制。
- ```/input/manifest.json```：輸入檔案清單、來源與完整性資訊。
- ```--asof```：資訊截止日，禁止使用這一天之後的資料。

主辦方會執行：
```
forecast \
  --panels /input/panels \
  --text /input/text \
  --asof YYYY-MM-DD \
  --out /output/forecast.parquet
```

Docker 必須能正確接受 ```forecast``` 這個 verb：
- 可以把 ```forecast``` 安裝成 PATH 上的可執行指令。
- 或由 ```ENTRYPOINT``` 指向的程式接收 forecast 作為第一個位置參數。
- 成功時必須以 exit code ```0``` 結束。
公開 exemplar 有一個例外：Parquet 位於 unit 根目錄，不在 panels/ 中；官方參考程式會向父目錄尋找。正式 staged unit 仍會使用 /input/panels，所以正式介面不可改掉。

## 金融數值資料格式

Panel 使用 Parquet 長格式。官方主要規格為：

| 欄位 | 型別 | 意義 |
|---|---|---|
| `date` | `date32` | 觀察日期 |
| `asset` | string | 資產或序列 ID |
| `value` | `float64` | 當天數值 |
| `panel_id` | string | 所屬 panel |

公開範例部分文件使用 ```asset_id```，實作時應檢查實際 schema，不要只寫死單一欄名。
目前公開的資料類別包括：
- 美國公債殖利率。
- G10 外匯。
- CPI、PCE、NFP、GDP 等總體經濟發布資料。
- MKT、SMB、HML、MOM、BAB、QMJ 等因子報酬。
程式不可寫死資產名稱、預測期限或單位，應從實際輸入與 ```card.toml``` 取得。

## Output

```/output``` 根目錄必須正好包含以下三個檔案：

| 檔案 | 用途 |
|---|---|
| `forecast.parquet` | Monte Carlo 預測抽樣 |
| `forecast_meta.json` | 題目、截止日、資產、期限與抽樣數 |
| `forecast_rationale.md` | 給人工審查者看的簡短預測理由 |

不要把 log、暫存檔或其他檔案寫入 ```/output``` 根目錄，否則可能在 g0 被拒絕。Log 應輸出到 stdout／stderr。

```forecast.parquet```
必須包含：

| 欄位 | 型別 | 意義 |
|---|---|---|
| `draw` | `int32` | 抽樣編號，從 0 開始 |
| `asset` | string | 必須完全符合目標資產 ID |
| `horizon` | `int32` | 未來第幾個營業日 |
| `value` | `float64` | 預測值 |

重要規則：
- 只接受 Monte Carlo samples，不接受未實作的 parametric 格式。
- 抽樣數必須介於 200 和 20,000。
- 一般建議先使用 500 組。
- F4 尾部風險題建議使用至少 1,000 組。
- ```draw``` 必須從 0 開始且連續。
- 每個 ```(draw, asset, horizon)``` 必須恰好有一列。
- 不可有重複列、漏項、NaN 或無限大。
- 不可用全部相同或幾乎沒有變化的數字冒充機率分布。
- 多資產題必須保留同一個 draw 內的共同漲跌關係，不能對每個資產完全獨立抽樣。

```forecast_meta.json```
格式大致如下：
```
{
  "unit_id": "t2-F3-example",
  "asof": "2024-06-28",
  "asset_ids": ["UST_2Y", "UST_10Y"],
  "horizons": [21],
  "representation": "samples",
  "n_draws": 500
}
```
注意：
- 必須寫 ```unit_id```，不能寫成 ```card_id```。
- ```unit_id``` 必須符合 ```card.toml [task].id```。
- ```asof```、資產、期限和抽樣數都必須與實際輸出及題目一致。
- ```representation``` 目前必須是 ```samples```。
官方 ```scorer``` 的報告會使用 ```card_id```，但那是 scorer 自己的輸出欄位，不能因此把 metadata 的 ```unit_id``` 改名。

```forecast_rationale.md```

這個檔案：
- 必須存在。
- 不可只有空白。
- 不會影響 CRPS 分數或排名。
- 內容是給人工審查者了解預測依據。
  
可以簡短說明：
- 預測的大致方向與不確定性。
- 哪些數值資料影響判斷。
- 文字資料帶來什麼調整。
- 哪些情況可能使預測失效。

  
## 怎樣才算通過？

官方有四項基本檢查，文件中會看到 `g0`～`g3`：

| Gate | 官方意義 | 白話說明 |
|---|---|---|
| g0 | Integrity | `/output` 是否正好有三個必要檔案，而且 metadata 能正常解析 |
| g1 | Schema | metadata 是否符合 schema、抽樣數是否合法、representation 是否為 `samples`、rationale 是否非空 |
| g2 | Cutoff/resource | metadata 是否與可信任的 card 綁定，截止日宣告是否正確 |
| g3 | Domain semantics | 資產與期限是否完全符合題目，Parquet 是否完整填滿每個格子，數值是否有效 |

任何一項失敗，這題就不會正常計分。

本機檢查指令：

```bash
python scoring/scoring.py score \
  --card units/t2-EXAMPLE-ust-curve-1m/card.toml \
  --forecast output/forecast.parquet
```

公開資料沒有真正答案，所以本機只能確認格式和流程，不能知道正式分數。

理想結果應包含：
```
{
  "admissible": true,
  "gates": {
    "g0_integrity": "pass",
    "g1_schema": "pass",
    "g2_cutoff_resource": "pass",
    "g3_domain_semantics": "pass"
  },
  "scored": false
}
```

## 分數怎麼看？

正式分數由三部分組成：
```
0.5 × marginal CRPS + 0.3 × joint variogram + 0.2 × tail penalty
```

- 分數越低越好。
- CRPS 檢查單一資產的預測是否準確且不確定範圍是否合理。
- Joint variogram 檢查多個資產是否呈現合理的共同變動。
- Tail penalty 檢查是否忽略 1%、5%、95%、99% 等極端結果。
- 單一資產、單一期限的題目沒有 joint 關係，因此權重會重新分配。
- 每一題權重相同。
- 不合格、執行失敗或未完成的題目會記最差值 4.0。
- 正規化後 1.0 表示與文字盲基準相當。
- 目標是讓平均分數低於 1.0。

官方共有 103 題練習題，這 103 題主要用於測試程式是否能處理各種題型，不應把公開練習排名當成正式能力排名。

## 網路和模型限制

正式 Track 2 使用 restricted 網路：
- 沒有開放網路。
- 不可下載行情、新聞、文件或模型。
- 不可直接呼叫 OpenAI、Anthropic、Google 或其他第三方模型 API。
- 只能呼叫主辦方提供的 MODEL_ENDPOINT。
- 或使用建置時已包進 Docker 的本機模型。
- 所有連線都會被代理伺服器記錄。
- 模型端的網路搜尋、retrieval、程式執行等工具必須關閉。
- 主辦方不會注入參賽者自己的 API key。


## 環境規格

固定要求：
- Python >=3.13。
- qfbench2-common 固定使用 v2.3.1。
- Docker image 必須包含： ```LABEL qfbench2.interface_version=2.0```
- 依賴、模型版本與模型訓練截止日必須明確記錄。
- 能固定 temperature 和 seed 時必須固定。
- 沒有 GPU 裝置時，Docker 仍必須能啟動並使用替代方案。

## 資源與時間限制

指定版本中，每個 Track 2 card 的上限為：

| 資源 | 上限 |
|---|---:|
| CPU | 16 |
| 記憶體 | 128 GB |
| GPU 宣告 | `gpu = true` |
| 每題 wall time | 1,800 秒 |
| 網路 | `restricted` |

注意：
- 1,800 秒只是單題硬上限，不代表每題都能使用 30 分鐘。
- Development 整次 submission 的總時間只有 43,200 秒，也就是 12 小時。
- 官方建議以每題約 400 秒左右作為工程預算。
- Final 與 Verification 的總時間上限為 86,400 秒，也就是 24 小時。
- 題目依序執行；前面太慢會導致後面的題目沒有時間。
- 計時從 docker create 開始，第一次拉取大型 image 的時間也可能被計入。
- ```gpu = true``` 不保證實際 worker 一定附有可用 GPU，所以必須有無 GPU fallback。
  
## 官方範例目前的問題

官方 Python 範例可以跑，但官方 Dockerfile 少裝了一個必要套件。自己的 Dockerfile 必須加入：

```text
qfbench2-common @ git+https://github.com/Agenthon-2026/Agenthon2026-public.git@v2.3.1#subdirectory=common
```
不要直接從會變動的 branch 安裝。

## 注意事項

1. 不可使用 --asof 之後的金融資料或文字。
2. 不可在正式執行時上網取得行情、新聞或答案。
3. 不可利用其他練習 unit 的較晚 panel，查出目前 unit 的特定 target value。
4. 不可把特定 unit 的答案，以查表、程式常數、模型權重或其他形式包進 Docker。
5. 可以使用公開練習資料進行一般訓練、調參與校準，但不可攜帶某題的特定答案進入該題執行。
6. 模型與套件版本必須固定。
7. 模型訓練截止日必須揭露。
8. 在支援的情況下，temperature 與隨機種子必須固定。
9. 正式推論不得下載新的模型、套件或外部資料。
    
