# 貢獻與協作規範

## Branch

- `main` 必須維持可建置、可執行。
- 禁止直接推送至 `main`；所有變更使用 Pull Request。
- 分支名稱使用 `feat/<issue>-<description>`、`fix/<issue>-<description>`、`docs/<issue>-<description>` 或 `chore/<issue>-<description>`。

## Commit

- 每個 commit 只處理一個可描述的目的。
- 訊息使用祈使句，例如 `Add forecast schema validation`。
- 不提交 `.env`、API key、token、資料集或大型產出檔。

## Pull Request

- 每個 PR 連結一張 Issue，並使用 `Closes #<number>`。
- 至少一位成員核准後才能合併。
- 共用介面、Docker 或依賴變更必須由受影響模組的成員 review。
- CI、測試與格式檢查全部通過後，使用 squash merge。
- 建議將實質變更控制在 400 行內；較大的工作先拆介面與實作。

## Definition of Done

- 驗收條件已完成。
- 有自動測試或明確的重現指令。
- 相關文件與實驗紀錄已更新。
- 沒有降低既有 gates pass rate。

