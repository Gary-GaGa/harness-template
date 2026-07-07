# SETUP.md — 新專案初始化程序

> 執行者：套用模板後的第一個 Claude Code session（人類只需下一句話：「讀 SETUP.md，執行初始化」）。
> 全部完成前，本套制度視為「未上線」。完成後在 CHANGELOG.md 記錄一行，本檔可保留供重跑。

## Checklist

**□ 1. 記錄版本**
在 CLAUDE.md「環境事實」的採用模板版本欄填入 git tag。之後模板升版時靠這個欄位判斷差距。

**□ 2. 填環境事實**
補齊 CLAUDE.md 中所有 `<TODO>`：build/test/lint 指令（實跑一次確認可用，不要抄 README）、目錄結構要點、既有 subagents 清單（`ls .claude/agents/` 並逐一讀 frontmatter）。若專案已有舊 CLAUDE.md：先 `cp CLAUDE.md.old CLAUDE.md.bak-初版`，把其中專案特定事實搬進環境事實節；行為規則若與模板衝突，以模板為準，衝突項列給使用者裁決。

**□ 3. 分發模式確認（問使用者，二選一）**
- **自包含**：ops/ 留在專案內，模板改進不自動流入。
- **分層**：通用規則放 user-level（`~/.claude/CLAUDE.md` 以 `@` import 引入單一 core repo 的 ops/），專案 CLAUDE.md 只留環境事實與專案特例。改 core 一次、全專案生效。

**□ 4. Fresh-context 對抗審查**
派 fresh-context agent（用 `ops/delegation-templates.md` T5 模板，model: sonnet）審查全部治理檔對照本 repo 現實：規則互相衝突、引用不存在的路徑/工具/指令、會被 Haiku 級模型誤讀的語句。修完為止。

**□ 5. 產生本專案的診斷**
用 `docs/A-diagnosis.md` 的三個判準（指揮官下場／自驗閉環缺失／憲法膨脹）掃描本 repo 的實際工作型態，產出 `docs/diagnosis-<專案名>.md`：各判準的現況證據 + 是否需要專案特有的補充規則（新增規則屬 maintenance-protocol「必問」範圍）。

**□ 6. Computational Feedback 接線提案（需使用者核准）**
提案 PostToolUse / Stop hook：改動程式碼後自動跑測試並把失敗結果回灌 transcript。附具體 hook 設定與風險說明，使用者核准才實施。

**□ 7. 回報**
給使用者一頁總結：填了什麼、審查抓到什麼、診斷結論、待核准事項清單。
