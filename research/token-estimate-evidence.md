# Token 概算：匿名用量與計量方法

更新日期：2026-11-04（Asia/Taipei）。[返回分析](custom-app-vs-self-hosted-token-estimate.md)。

## 證據可支持的範圍

研究讀取原生 Claude Code 與 Codex 開發紀錄的用量欄位；下列三例為原生 Claude Code 的匿名彙總。
案例皆是既有系統增修或同步，不是同一需求的從零 A/B 實驗。它們只能說明請求數、上下文與快取的量級，不能用來計算平台的因果節省率。
逐階段的 100 → 76 表格及 65–90% 範圍是工程預算，不是由以下案例相除而來。

原始紀錄由分析者本地保留，不放入公開 repo。此處不提供 Session ID、客戶名稱、租戶、網址、原始提示、工具輸出或開發路徑。
公開讀者可檢查算術，但不能只靠此彙總獨立驗證原始事件；用量也未與供應商帳單對帳。

## 匿名用量

以下皆為 Token，非費用；input 已合併一般輸入、cache creation 與 cache read。

| 樣本 | 範圍 | 去重模型請求 | 完整 input | cache read | output | input＋output | 排除 cache read 後 |
|---|---|---:|---:|---:|---:|---:|---:|
| A | AI GO Custom 業務分頁增修 | 135 | 37,976,958 | 37,639,106 | 121,722 | 38,098,680 | 459,574 |
| B | AI GO Custom VFS／本地同步 | 33 | 3,951,527 | 3,861,538 | 13,553 | 3,965,080 | 103,542 |
| C | 既有官網改版並部署一般主機 | 119 | 30,333,943 | 29,929,579 | 140,040 | 30,473,983 | 544,404 |

A、B 的歷史工具紀錄同時包含 builder API、VFS、compile 與 publish，據此確認為 Custom App。
C 的歷史紀錄包含 main 合併後主機自動部署及正式網址／資產檢查，據此確認為直接主機部署。
C 的部署機制原本就存在，沒有從零建立登入、資料庫或部署腳本；A、B 同樣不是從零建立完整系統。
研究未重新驗證歷史案例目前的正式站狀態。

「排除 cache read 後」仍包含 cache creation 與 output，不能視為金額、帳單 Token 或訂閱額度。
不同案例的模型、上下文、任務與範圍不同，不應將 A／C 或 B／C 當作技術路線比較。

## 計量與去重

1. OpenAI／Codex：每次 input＋output；cached input 已在 input 中，reasoning 已在 output 中，不重複加回。
2. Claude：一般 input＋cache creation＋cache read＋output；thinking 不額外重加。
3. 原生逐回應紀錄依唯一 message／response ID 去重；重複 streaming 用量不逐片累加。累計快照不能逐筆相加。
4. 若使用量含多個 iteration，按供應商對該欄位的定義彙總，不能再與已彙總頂層值重加。本輪選樣未遇到需額外彙總的多 iteration 案例。
5. 匯入歷史不當作新的模型請求；分支、worktree 與跨檔重疊不能無條件相加。單一 Session 也不保證包含所有子代理及其他工作階段。
6. 模型請求、人類討論、工具呼叫分開計。紀錄中的 user 訊息可能包含摘要或系統事件，不能全部算成人類決策輪數。
7. 每次請求只依主要目的歸入一個階段，該次攜帶的 Skill、歷史及工具結果一併計入；階段加總須等於全案總量。
8. CI 執行、主機建置或等待本身不按時間產生模型 Token；模型撰寫設定、讀取日誌及修正才計用量。

## 可重做的配對實驗

使用同一份需求、合成資料、KPI 規則、UI、角色權限、正式部署與驗收清單。兩邊從空 App／repo 開始。
各工具內固定模型、推理強度、上下文壓縮與 Skill 載入策略；不跨模型直接相除。
保留人類討論紀錄，但不強制模型請求數相同，讓請求數成為平台流程差異的結果。
將所有必要請求、失敗重試、子代理及審查納入，以 response ID 跨檔去重。
按相同階段規則分組，再報告每條路線多次完成任務的中位數、範圍及失敗情形。

## 官方計量來源

- [OpenAI 用量與觀測](https://developers.openai.com/api/docs/guides/agents-api/observability)
- [OpenAI prompt caching](https://developers.openai.com/api/docs/guides/prompt-caching)
- [Claude prompt caching](https://platform.claude.com/docs/en/build-with-claude/prompt-caching)
- [Claude Code 用量](https://code.claude.com/docs/en/costs)

官方文件用於核對計量方式，並未提供或背書 AI GO 與直接開發的節省比例。
