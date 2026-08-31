# Track 2 官方規格｜白話短版

> 最後核對：2026-08-30
> 官方版本：`639614b61db1952834787d2d6b3632330bdf2a19`

## 概述

我們要提交一個 Docker 預測程式。程式讀取截止日以前的金融時間序列與公開文字資料，輸出許多組彼此相關的未來情境，也就是一個機率分布，而不是單一預測值。
正式測試沒有開放網路，只能使用主辦方提供的模型端點，或使用事先包進 Docker 的模型。

## Docker input

主辦方會把整個題目目錄唯讀掛載到 '''/input'''：

'''/input/
├── card.toml
├── manifest.json
├── panels/
│   └── *.parquet
└── text/
    └── ...
'''

主要輸入為：
- '''/input/panels'''：截止日以前的金融時間序列。
- '''/input/text'''：截止日以前公開的新聞、央行文件及評論。
- '''/input/card.toml'''：題目 ID、目標資產、預測期限、單位與環境限制。
- '''/input/manifest.json'''：輸入檔案清單、來源與完整性資訊。
- '''--asof'''：資訊截止日，禁止使用這一天之後的資料。
  

## 程式會收到什麼？

- `/input/panels`：過去的金融數字。
- `/input/text`：新聞、央行文件等文字資料。
- `--asof`：資料截止日。只能使用這一天以前的資訊。

主辦方會執行：

```bash
forecast \
  --panels /input/panels \
  --text /input/text \
  --asof YYYY-MM-DD \
  --out /output/forecast.parquet
```

所以 Docker 必須看得懂 `forecast` 這個指令，成功時要正常結束。

## 程式要交出什麼？

三個檔案缺一不可：

| 檔案 | 用途 |
|---|---|
| `forecast.parquet` | 多次抽樣的預測數字 |
| `forecast_meta.json` | 題目、日期、資產、預測天數與抽樣次數 |
| `forecast_rationale.md` | 用白話簡述預測理由，不可空白 |

重要規則：

- 至少產生 200 組預測；一般先用 500 組。
- 每個資產和預測天數都不能漏。
- `forecast_meta.json` 要寫 `unit_id`，不能寫成 `card_id`。
- 不可出現空值、無限大或完全一樣的假抽樣。

## 怎樣才算通過？

官方有四項基本檢查，文件中會看到 `g0`～`g3`：

| 名稱 | 白話意思 |
|---|---|
| g0 | 程式和檔案是否完整、能否正常執行 |
| g1 | 三個輸出檔的格式是否正確 |
| g2 | 有沒有偷用未來資料、超時或超出資源 |
| g3 | 預測數量是否足夠、內容是否合理且沒有漏項 |

任何一項失敗，這題就不會正常計分。

本機檢查指令：

```bash
python scoring/scoring.py score \
  --card units/t2-EXAMPLE-ust-curve-1m/card.toml \
  --forecast output/forecast.parquet
```

公開資料沒有真正答案，所以本機只能確認格式和流程，不能知道正式分數。

## 分數怎麼看？

不用先理解公式，只要記得：

- 分數越低越好。
- 不只看猜得準不準，也看不確定範圍是否合理。
- 多個資產一起預測時，也會看它們是否合理地一起漲跌。
- 極端事件不能完全忽略。
- 每題都算分；漏跑或出錯會記 `4.0`，也就是最差分數。
- 官方純數字模型的基準是 `1.0`，目標是低於它。

官方共有 103 題練習題。先追求 103 題全部能完成，再追求更好的分數。

## 網路和模型限制

正式測試不能自由上網：

- 不可抓即時行情、新聞或其他外部資料。
- 不可直接呼叫 OpenAI、Anthropic 或 Google API。
- 只能使用主辦方提供的模型服務，或把自己的模型放進 Docker。
- 本機測試要使用 `--network=none`，確認斷網也能完成基本流程。

目前先不要決定要用多大的模型。先讓沒有模型 API 的基本版本也能完成預測，之後再選方案。

## 環境規格

先固定下面幾件事：

- Python 3.13 或更新版本。
- `qfbench2-common` 固定使用 `v2.3.1`。
- Docker 加上 `qfbench2.interface_version="2.0"`。
- 沒有 GPU 時，程式也必須能啟動或改用簡單版本。

官方每題最多提供 16 顆 CPU、128 GB 記憶體和 30 分鐘，但整批 103 題只有 12 小時。平均每題大約只能用 7 分鐘，所以不能讓每題都跑滿 30 分鐘。

## 官方範例目前的問題

官方 Python 範例可以跑，但官方 Dockerfile 少裝了一個必要套件。自己的 Dockerfile 必須加入：

```text
qfbench2-common @ git+https://github.com/Agenthon-2026/Agenthon2026-public.git@v2.3.1#subdirectory=common
```

不要直接從會變動的 branch 安裝。

## 重要日期

| 日期 | 要做什麼 |
|---|---|
| 2026-09-28 前 | 完成報名與開發階段 |
| 2026-09-29～10-12 | Final；每個參賽項目只有一次正式提交 |
| 2026-10-13～10-25 | 主辦方重新執行並檢查能否重現 |

## 不可踩的紅線

1. 不可使用 `--asof` 日期之後的資料。
2. 不可在正式執行時上網找答案。
3. 不可從其他練習題查出這一題的特定答案。
4. 必須固定套件、模型版本和隨機種子，讓結果可以重跑。

## 官方來源

- [Agenthon 2026 Protocol](https://www.agenthon.net/#protocol)
- [Track 2 官方 repository](https://github.com/Agenthon-2026/track2-forecasting-public)
- [完整 CLI 規格](https://github.com/Agenthon-2026/track2-forecasting-public/blob/main/SUBMISSION_CLI.md)
