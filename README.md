# Agenthon 2026 Track 2 團隊工作區

## 我們要做什麼？

做一個可以放進 Docker 的預測程式。主辦方會給它兩種資料：

- 過去的金融數字
- 當時已經公開的新聞與文件

程式要根據這些資料，產生多種可能的未來結果，而不是只猜一個數字。

## 現在先做這 5 件事

1. 下載官方範例。
2. 用 Python 跑通範例。
3. 修好並跑通 Docker 版本。
4. 確認每次都會產生三個必要檔案。
5. 用官方檢查工具確認全部通過。

先把整條流程跑通，再研究更好的預測方法。

## 完成時應該看到什麼？

主辦方會用類似下面的指令執行程式：

```bash
docker run <我們的-image> \
  forecast \
  --panels /input/panels \
  --text /input/text \
  --asof YYYY-MM-DD \
  --out /output/forecast.parquet
```

程式成功後，`/output` 必須有：

```text
forecast.parquet       預測結果
forecast_meta.json     這次預測的基本資料
forecast_rationale.md  這次預測的簡短理由
```

## 三條底線

1. 不可使用指定日期之後的資料。
2. 正式比賽不能上網抓新聞或行情。
3. 每一題都要完成；漏掉一題會被記最差分數。

## 文件怎麼看？

第一次加入專案，只要依序看：

1. [白話版官方規格](docs/OFFICIAL_REQUIREMENTS.md)
2. [團隊總計畫](docs/TEAM_PLAN.md)
3. [提交前檢查表](docs/SUBMISSION_CHECKLIST.md)

需要時再查：

- [實驗紀錄](docs/EXPERIMENT_LOG.md)
- [決策紀錄](docs/DECISION_LOG.md)
- [GitHub 協作規則](CONTRIBUTING.md)

## GitHub 怎麼用？

1. 從 Project 選一張工作卡。
2. 建立自己的 branch。
3. 完成後開 Pull Request。
4. 另一人確認、檢查通過後合併。

正式內容以 GitHub `main` 為準。
