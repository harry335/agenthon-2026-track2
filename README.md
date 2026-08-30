# Agenthon 2026 Track 2 Team Workspace

本工作區用來管理三人團隊的程式、實驗與提交流程。

## 文件入口

- [專案總計畫](docs/TEAM_PLAN.md)
- [實驗紀錄](docs/EXPERIMENT_LOG.md)
- [決策紀錄](docs/DECISION_LOG.md)
- [提交檢查表](docs/SUBMISSION_CHECKLIST.md)
- [協作與 PR 規範](CONTRIBUTING.md)

## GitHub 工作流程

1. 從 GitHub Project 選擇一張 Issue。
2. 建立 `feat/<issue>-<description>`、`fix/<issue>-<description>` 或 `docs/<issue>-<description>` 分支。
3. 完成變更、驗證與文件後開 Pull Request。
4. 至少一位成員核准且 CI 通過後，以 squash merge 合併。

## 團隊原則

1. `main` 必須維持可建置、可執行、可產生三個必要輸出檔案。
2. 每項實驗必須綁定 Git commit、設定、seed、執行時間與結論。
3. 先消除 DNF，再改善預測分數。
4. Numeric-only 與 full-text 必須能用設定切換，以支援 text ablation。
5. 禁止使用 `asof` 之後的資料，也禁止跨 unit 查找特定答案。
