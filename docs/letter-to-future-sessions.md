> **[歷史文件]** 本檔記錄 2026-07 初版立法時的原始環境與交接內容，僅供理解規則由來。現行初始化程序見 `SETUP.md`；本檔 Task 0 一節已由 SETUP.md 取代。

# docs/letter-to-future-sessions.md — 給未來 session 的信（歷史文件）

> 寫於 2026-07-06，claude.ai 環境（非使用者 repo 內）。作者是本套制度的初版撰寫者。
> 你不需要每次都讀這封信。該讀的時機：進入新專案、對制度存在的理由感到困惑、或使用者說「照交接做」。

---

## 先做：Task 0（第一個 Claude Code session 的待辦，做完可刪本節）

本套檔案產出於 claude.ai，撰寫時**無法讀取實際 repo**。以下校準工作被刻意留給你，因為你在真實環境裡：

1. 備份舊 CLAUDE.md（`cp CLAUDE.md CLAUDE.md.bak-初版`），按新版 CLAUDE.md 頭部的合併指引合併，補齊「環境事實」節的 TODO（build/test 指令、目錄結構、既有 subagents 清單）。
2. 派一個 fresh-context agent 用 `ops/delegation-templates.md` T5 模板，對全部治理檔做對抗審查：規則衝突、路徑錯誤、與 repo 現實不符之處。撰寫環境裡只能做同 context 的自審＋機械檢查，**真正的 fresh-context 審查是你的責任**。
3. 用 `docs/A-diagnosis.md` 的三個判準對照 repo 實況，修正診斷中與現實不符的描述。
4. 向使用者提案（需核准，屬 `ops/maintenance-protocol.md` 第 1 節「必問」範圍）：PostToolUse/Stop hook 自動跑測試並回灌結果——這是先前 Harness Engineering 分析已識別、至今未補的 Computational Feedback 缺口。

## 三件沒被問、但對這個環境最重要的事

**1. Session 續命靠落檔，不靠記憶。** 這個環境最確定的事實是：你會被中斷，context 會滿，下一個你什麼都不記得。所以任何超過一小時的工作，維持一份 `PROGRESS.md`（目前狀態／已決事項與理由／下一步），每個里程碑更新。接手的 session 讀它，十行內回到現場。記憶功能與 context 都不可靠，檔案永遠在。

**2. 既有的三個 reviewer subagents（spec-reviewer / quant-skeptic / data-integrity-auditor）目前是「建議者」不是「關卡」。** 它們 read-only、意見可被忽略——這正是自驗問題的另一面。最高價值的一步改造：把它們接進 T5 驗收流程，讓相應類型的產出（spec → spec-reviewer、數據 → quant-skeptic/data-integrity-auditor）必須過關卡才算完成。這比新增任何 subagent 都便宜且有效。

**3. 使用者本人是這個 harness 裡最強的驗收 agent，但頻寬最貴。** 他是資深 Data Architect，判斷力遠超模型在品味與領域題上的水準——所以高風險決策不要給單一答案，給 2–3 個選項＋trade-offs 讓他裁決（這用他 30 秒，替你省 3 輪返工）；但低風險決策絕不要拿去煩他（Rubric 3 有判準）。他的偏好是結論先行、零客套——表演性的提問與過度確認會消耗他對這套制度的信任。

## 這套制度最可能的退化方式（按可能性排序）與預防

**退化 1：規則通膨。** 每次踩坑直接往 CLAUDE.md 加一條，兩個月後 CLAUDE.md 變成 300 行的許願牆，固定 token 稅暴漲、規則開始互咬。**預防**：教訓一律先進 `LESSONS.md` 緩衝區，累積成模式才升格（`ops/maintenance-protocol.md` 第 3–4 節）；CLAUDE.md 的 120 行上限是硬的。

**退化 2：驗收儀式化。** fresh-context 驗收慢慢變成走過場——驗收 prompt 附上了實作者的自評，驗收 agent 照抄 PASS。**預防**：T5 模板明文「不要向實作方查證」且只附原始驗收條件；建議使用者每月一次故意埋一個已知錯誤測驗收是否真的會抓（制度的 smoke test）。

**退化 3：例外累積。** 「這個任務很小，就不派工／不驗收了吧」——第一次是效率，第十次是常態，制度名存實亡。**預防**：豁免範圍已明文定義（dispatch 第 1 條「可以自己做的情境」），範圍外零例外。行為判準：當你發現自己在**尋找理由**跳過某條規則，這個尋找動作本身就是該遵守它的訊號。

**退化 4：檔案漂移。** repo 的指令、路徑、工具換了，ops 檔沒跟上，規則引用不存在的東西，弱模型會照著壞掉的指令硬跑。**預防**：季度健檢（`ops/maintenance-protocol.md` 第 4 節）＋ CLAUDE.md 硬規則 6（查不到就標註，禁止硬掰）。

## 誠實條款：這套 harness 的極限

拆解、驗證、多樣本評審能補的是**執行品質**。以下補不了，遇到照此處理：

- **品味與模糊題**（文案語氣、架構優雅度、「哪個方案比較好」的無標準題）：多答案評審提升期望值，但上限是評審模型的品味。處理順序：Rubric 5 → 升級最強可用模型 → 給使用者選項而非答案 → 明說做不到。
- **領域縱深**（台灣金融法遵、公司內部政治、未公開的組織脈絡）：模型沒有可靠資訊。一律標註 [推測] 並建議人類確認，禁止以流利語氣掩蓋不確定。
- **本套制度自身的正確性**：它出自單一 session 的結構性推理，未經 repo 實測。前三次使用時對它保持懷疑，摩擦點記進 LESSONS.md——制度是用來修的，不是用來供的。

## 交接：本 session 未完成事項

- 未能讀取實際 repo → Task 0 第 1–3 項。
- Hook 回灌 Computational Feedback → Task 0 第 4 項（需使用者核准）。
- 收尾的 fresh-context 對抗審查在本環境不可行（無 subagent 工具），以同 context 對抗性重讀＋bash 機械交叉檢查替代（已確認：跨檔閾值一致、條號引用全部有效、無失效路徑）；真正的 fresh-context 審查 → Task 0 第 2 項。
