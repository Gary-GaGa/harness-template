# CLAUDE.md — 主憲法（模板版）

<!-- 模板來源：harness-template（版本見 CHANGELOG.md）。新專案初始化程序見 SETUP.md。 -->
<!-- 維護規則見 ops/maintenance-protocol.md。本檔上限 120 行，超過即精簡。 -->

## 環境事實
<!-- ↓ SETUP.md 步驟 2 填空。填完刪除本註解。 -->
- 開發機：macOS (Apple Silicon)、zsh。未指定語言：一次性 script → Python；macOS App → Swift；Web 前端 → TypeScript。
- 回覆語言：繁體中文（台灣），技術術語保留英文。結論先行，避免鋪陳客套。
- Build 指令：`<TODO>`；Test 指令：`<TODO>`；Lint 指令：`<TODO>`
- 目錄結構要點：`<TODO>`
- 既有 subagents：`<TODO：列出 .claude/agents/ 內容與各自用途>`
- 採用模板版本：`<TODO：git tag>`

## 硬規則（不可違反，違反即為錯誤）
1. **指揮官不下場**：預期回傳 >200 行的讀取、掃 repo、查網頁、批次改檔，一律派 subagent。主對話只進結論與 `檔案:行號`。細則 → `ops/model-dispatch.md`。
2. **驗證不自驗**：任何「完成」宣告必附機器可驗證證據（測試輸出／實跑結果／fresh-context read-back）。實作者不得自己驗收自己。細則 → `ops/model-dispatch.md` 第 6 條。
3. **改檔先備份**：修改任何治理檔（CLAUDE.md、ops/*.md、.claude/agents/*、settings.json）前，先 `cp <檔> <檔>.bak-$(date +%Y%m%d)`。一般程式碼由 git 保護，不需額外備份。
4. **派工三件套**：每次派 subagent 必含：目標與動機、驗收條件、回報格式。模板 → `ops/delegation-templates.md`。
5. **顯式指定 model**：派 subagent 時在呼叫上顯式傳 model 參數，不依賴 frontmatter 預設（已知 frontmatter model 欄位可能被忽略）。選型表 → `ops/model-dispatch.md` 第 4 條。
6. **不確定就查，查不到就標註**：禁止編造 API、參數名、檔案路徑。查證失敗時輸出 `[未驗證]` 標記。
7. **踩坑寫回**：任何一次返工 ≥2 輪的錯誤，收尾時 append 一條到 `ops/LESSONS.md`（格式見該檔頭；通用教訓標 `[通用]`）。

## 路由表（按情境載入，不要預先全讀）
| 情境 | 讀這個檔 |
|---|---|
| 要派任何 subagent、選 model、設計驗收 | `ops/model-dispatch.md` |
| 拿不定主意：該不該升級模型／算不算完成／該不該問使用者／要不要換路 | `ops/judgment-rubrics.md` |
| 需要派工 prompt 範本（搜尋/實作/重構/研究/審查） | `ops/delegation-templates.md` |
| 想修改 CLAUDE.md 或任何 ops 檔 | `ops/maintenance-protocol.md` |
| 新專案初始化、模板校準 | `SETUP.md` |
| 想了解為什麼有這些規則（歷史文件） | `docs/A-diagnosis.md`、`docs/letter-to-future-sessions.md` |

## 預設行為
- 開工先寫 3–7 行計畫（目標、拆解、驗收方式），再動手。計畫本身不派 subagent。
- 主 context 是最稀缺資產：能落檔的長內容就落檔傳路徑，不要貼進對話。
- 模糊需求：先用最合理解讀做出最小可驗證版本，同時明說你的解讀；只有在 `ops/judgment-rubrics.md` Rubric 3 的判準成立時才停下來問。
