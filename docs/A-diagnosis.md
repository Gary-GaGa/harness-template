> **[歷史文件]** 本檔是初版制度的立法理由書，診斷對象為 2026-07 的原始環境。新專案的診斷由 `SETUP.md` 步驟產生，不要把本檔內容當作你專案的現況。

# A. Harness 快速診斷（2026-07-06，Fable 5 產出）

> 讀者：後續接手的主模型（Sonnet/Opus/Haiku）與使用者本人。
> 依據：使用者既有 harness 的已知結構——CLAUDE.md constitution、三個 read-only reviewer subagents（spec-reviewer / quant-skeptic / data-integrity-auditor）、minimal-permission settings.json、Harness Engineering 2×2 框架（Feedforward/Feedback × Computational/Inferential）分析中已識別的缺口。
> 誠實標註：本診斷未能直接讀取 repo 內實際檔案（產出於 claude.ai 環境），屬「結構性診斷」而非「逐行診斷」。新專案套用模板時，由 `SETUP.md` 的初始化程序產生該專案自己的診斷。

---

## 第一名：指揮官下場親自讀取（最大 token 漏 + 失焦主因）

**症狀**：主對話直接 Read/Grep/WebFetch 大量檔案，raw content 全部進主 context。一次掃 repo 就吃掉 30–60k tokens；到 context 三分之二滿之後，訊噪比崩壞、模型開始忘記早期指令與驗收條件——這就是「失焦」的機制本質，不是模型變笨，是 context 被垃圾填滿。

**判準（何時算違規）**：主對話單次工具呼叫預期回傳 > 200 行，或需要連續讀 ≥ 3 個檔案才能回答的問題，卻沒有派 subagent。

**修法**（已制度化於 `ops/model-dispatch.md` 第 1 條）：
- 大量讀取、掃 repo、查網頁、批次改檔 → 一律派 subagent（內建 Explore agent 或自訂 agent）。
- 主對話只接收：結論 + `檔案:行號` 引用 + 落檔路徑。禁止 subagent 回傳原文 dump。
- 正例：「派 Explore（model: haiku）找出所有呼叫 `loadUser()` 的位置，回報格式：每處一行 `路徑:行號 — 一句話用途`，上限 30 行。」
- 反例：主對話自己 `grep -r loadUser` 然後把 400 行輸出留在 context 裡。

## 第二名：驗證閉環缺失——自驗 + Computational Feedback 沒有回灌（最大錯誤來源）

**症狀**：實作者在同一個 context 裡寫完程式碼、自己宣告「完成」。三個 reviewer subagents 是「被動建議者」而非「驗收關卡」——它們的意見可以被忽略，且沒有機器可驗證的證據要求。先前 Harness Engineering 分析已識別的缺口正是：Computational Feedback（測試、lint、build 結果）沒有自動回灌到 agent 自我修正迴圈。結果是：錯誤在同一 context 裡延續多輪，越修越歪。

**判準（何時算違規）**：任何「完成」宣告，沒有附上以下至少一項——測試輸出、實跑結果、fresh-context read-back 確認。

**修法**（制度化於 `ops/model-dispatch.md` 第 6 條「驗證不自驗」+ `ops/judgment-rubrics.md` Rubric 2）：
- 完成定義 = 機器可驗證證據，不是模型自評。程式碼 → 測試或實跑；檔案 → fresh-context agent read-back；高風險判斷 → 第二意見或多答案評審。
- 驗收一律派 **fresh-context** agent：它沒看過實作過程，不會繼承實作者的盲點。
- 建議追加（需使用者同意，屬 F 級變更）：PostToolUse / Stop hook 自動跑測試並把失敗結果印回 transcript，讓 Computational Feedback 強制回灌。

## 第三名：CLAUDE.md 膨脹與規則衝突（固定稅 + 慢性失焦）

**症狀**：CLAUDE.md 是每個 session 的固定成本——每一行都在每一次對話開頭付一次 token 稅，而且規則越多、彼此衝突的機率越高。能力較弱的模型（Haiku、低 effort Sonnet）遇到衝突規則時會隨機選一條執行，行為不可預測。常見膨脹來源：把「一次性教訓」直接塞進 CLAUDE.md、同一規則用不同措辭寫兩次、過時工具名沒清。

**判準（何時算違規）**：CLAUDE.md 超過 120 行；或存在兩條規則對同一情境給出不同指示；或引用了 repo 中不存在的檔案/工具。

**修法**（制度化於新版 `CLAUDE.md` 本身的結構 + `ops/maintenance-protocol.md`）：
- CLAUDE.md 只放三種東西：硬規則（≤10 條）、路由表（何時讀哪個 ops 檔）、環境事實。其他一律抽成引用檔。
- 教訓寫進 `ops/LESSONS.md`（append-only，有格式），累積到閾值才由使用者核准合併進規則。
- 寫法原則：「**能力較弱的模型需要明確，能力較強的模型需要留白**」——硬規則寫成 Haiku 也不會誤讀的祈使句 + 判準 + 範例；策略性內容標註「由主模型自行判斷」。

---

## 三名共同根因（一句話版）

主 context 被當成無限資源使用，而它是整個 harness 最稀缺的資產。CLAUDE.md 與 ops/ 下所有制度檔都是圍繞「保護主 context、外包執行、機器驗收」三件事設計的。
