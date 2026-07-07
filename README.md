# harness-template

Claude Code 治理 harness 模板：一套可複製到新專案的「主憲法 + 作業守則」，讓主對話專注於**拆解、派工、整合、決策**，把執行外包給 subagent，並以機器可驗證證據取代模型自評。

## 解決的問題

- **Context 浪費**：主對話自己讀大量檔案，context 被原文佔滿 → 規則：指揮官不下場，讀寫掃查一律派 subagent
- **自驗閉環缺失**：實作者自己宣告完成、自己驗收 → 規則：驗證不自驗，一律派 fresh-context agent 驗收
- **制度膨脹與退化**：規則越寫越多、跨 session 遺忘教訓 → 維護協議 + 教訓回寫機制

## 使用方式

1. 複製整個模板到新專案根目錄
2. 依 `SETUP.md` 的初始化檢查清單，填妥 `CLAUDE.md` 內所有 `<TODO>`（build/test/lint 指令、目錄結構、subagents 清單等）
3. 完成前視為未上線；之後由 Claude Code session 依路由表按需載入各守則

## 檔案結構

| 路徑 | 用途 |
|---|---|
| `CLAUDE.md` | 主憲法：硬規則 7 條、情境路由表、預設行為（模板，含 `<TODO>` 佔位） |
| `SETUP.md` | 新專案初始化檢查清單 |
| `CHANGELOG.md` | 模板版本記錄 |
| `ops/model-dispatch.md` | 模型調度守則：何時派 subagent、model 選型、升降級、驗證不自驗 |
| `ops/judgment-rubrics.md` | 判斷力外化：升級模型／算不算完成／該不該問使用者等 5 個判準 |
| `ops/delegation-templates.md` | 派工 prompt 範本（搜尋／實作／重構／研究／審查）與回報合約 |
| `ops/maintenance-protocol.md` | 治理檔維護協議：修改程序、教訓回寫、精簡閾值、回退規則 |
| `ops/LESSONS.md` | 踩坑教訓檔（append-only） |
| `docs/A-diagnosis.md` | 歷史文件：三大根本問題診斷，各規則的立法理由 |
| `docs/letter-to-future-sessions.md` | 歷史文件：給未來 session 的交接信 |
