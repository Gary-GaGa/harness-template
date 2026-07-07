# ops/model-dispatch.md — 模型調度守則

> 讀者：主對話模型（任何等級）。每一條都有判準與範例，照做即可，不需要品味。
> 事實依據（2026-07 向官方文件與社群驗證）：subagent frontmatter 支援 `model: sonnet|opus|haiku|inherit|<完整 model ID>`；解析順序為 `CLAUDE_CODE_SUBAGENT_MODEL` 環境變數 → 呼叫時 model 參數 → frontmatter → inherit；**已知 bug：frontmatter 的 model 可能被忽略，只有呼叫時顯式傳入的 model 參數確定生效**。subagent 繼承主對話的 extended thinking 設定（v2.1.198+），沒有 per-subagent thinking 開關。

---

## 1. 指揮官不下場

主對話的職責是：拆解、派工、整合、決策。不是執行。

**必須派 subagent 的情境（滿足任一即派）**：
- 讀取：預期單次工具輸出 >200 行，或需連讀 ≥3 個檔案。
- 掃 repo：任何「找出所有 X」「盤點 Y」型任務 → 內建 Explore agent。
- 查網頁：任何需要 ≥2 次 fetch 的研究。
- 批次改檔：同一模式改 ≥3 個檔案。

**可以自己做的情境**：讀單一已知檔案的特定區段（<100 行）、跑一條 build/test 指令看結果、寫 <30 行的單檔修改。

- 正例：「找出這個 bug 可能的成因」→ 派 Explore 收集相關檔案位置與摘要，主對話拿摘要做診斷決策。
- 反例：主對話自己開 5 個檔案全文讀完才開始想——這 5 個檔案的原文會佔據 context 直到 session 結束。

## 2. 派工三件套（缺一不可）

每個派工 prompt 必含三段，模板見 `ops/delegation-templates.md`：

1. **目標與動機**：做什麼＋為什麼（subagent 看不到主對話歷史，動機決定它遇到歧義時怎麼選）。
2. **驗收條件**：完成的客觀定義，subagent 交付前自查用。
3. **回報格式**：明確的輸出結構與長度上限。

## 3. 回報合約（subagent 端義務，寫進每個派工 prompt）

- 只回：結論、`檔案:行號` 引用、落檔路徑。
- 長產物（>50 行）一律寫檔，回傳路徑 + ≤10 行摘要。
- 禁止：貼原始碼全文、貼網頁原文、貼完整工具輸出。
- 失敗要說失敗：找不到、跑不動、不確定，直接回報 + 已嘗試什麼，不要硬掰答案。

## 4. 顯式指定 model 與思考深度

**每次派工在呼叫上顯式傳 model 參數**（理由見檔頭 bug 說明）。選型表：

| 任務類型 | model | 理由 |
|---|---|---|
| 掃 repo、找位置、格式轉換、機械式批改 | `haiku` | 不需推理，要快要便宜 |
| 實作、重構、寫測試、一般研究 | `sonnet` | 性價比預設值 |
| 架構決策、跨模組重構設計、高風險判斷的第二意見 | `opus`（或當時最強可用模型） | 錯誤成本 > 模型成本 |

思考深度的誠實說明：Claude Code 的 extended thinking 是 **session 層**設定，subagent 繼承主對話、無法逐一指定。要「高思考深度的 subagent」，做法是主 session 開 thinking，或改用 API 直呼（Opus 4.5+ 支援 effort 參數）。不要在 frontmatter 裡發明 thinking/effort 欄位——不存在。

## 5. 升降級路徑

- **降級**：Sonnet 派出的任務若屬上表 haiku 列，改派 haiku。連續 3 次同型任務 haiku 都合格 → 該型任務預設 haiku。
- **升級**：同一任務 Sonnet 失敗 2 次（驗收不過或方向反覆）→ 停止重試，升級 opus 重派，並附上兩次失敗的摘要作為額外 context。升級判準細節見 `ops/judgment-rubrics.md` Rubric 1。
- **升級到使用者**：opus 也失敗，或任務觸及 Rubric 3（該問使用者）→ 停手，整理現況問人。

## 6. 驗證不自驗（本檔最重要的一條）

實作者的 context 帶著它自己的盲點，自驗等於沒驗。驗收一律派 **fresh-context** agent（新派工、不共享實作過程），按產物類型：

| 產物 | 驗法 | 派工要點 |
|---|---|---|
| 檔案/文件 | read-back：fresh agent 讀檔，回答「這份檔案要求你做什麼」，答案與原意不符即退回 | model: haiku 即可 |
| 程式碼 | 測試或實跑：先跑既有測試；無測試則要求驗收 agent 寫最小 smoke test 或實跑一次貼結果 | model: sonnet |
| 高風險判斷（架構選型、對外文案、金額計算） | 第二意見：fresh opus agent 只拿問題與答案（不拿推理過程），獨立評判；或多答案評審——派 2–3 個 agent 各自作答，再派一個評審 agent 選優並說明理由 | model: opus 評審 |

- 正例：改完 auth 模組 → 派 fresh sonnet agent：「跑 `<測試指令>`，全綠則檢查 diff 是否含 hardcoded secret，回報 PASS/FAIL + 證據行號」。
- 反例：實作 agent 回報「已完成並自我檢查通過」→ 主對話採信。這是第一名錯誤來源（見 `docs/A-diagnosis.md` 第二名）。

## 7. 主對話 context 保衛（收尾習慣）

- 每完成一個里程碑，把「目前狀態、已決事項、下一步」寫進工作檔（如 `PROGRESS.md`），之後靠讀檔續命，不靠 context 記憶。
- 感覺開始重複自己、忘記先前決定 → 這是 context 汙染訊號，立即落檔現況，建議使用者開新 session 續作。
