# 專案多代理人協作總規範

本文件定義本專案之團隊分工原則、標準開發工作流、除錯鐵律與任務交接協定。所有參與本專案開發之 AI Agents（Sub-agents）與工程人員均須嚴格遵守本規範。

---

## 1. 團隊角色矩陣與技能隔離原則

為避免職責混淆與上下文污染，專案嚴格落實**技能隔離（Skill Scoping）**與**行為邊界管制**，各角色僅能使用專屬掛載之 Skills：

| 角色設定檔 | 角色名稱 / 職位 | 掛載 Skills | 核心職責 | 嚴格行為邊界（禁止行為） |
| :--- | :--- | :--- | :--- | :--- |
| [architect.md](file:///.agents/architect.md) | **@architect**<br>(架構師 / PM) | `ponytail`<br>`codegraph` | • 需求釐清與拆解任務清單<br>• 維護專案進度與驗收標準 (DoD)<br>• 制定前後端 API Contract<br>• 設計資料庫 Schema | **只做規劃、設計與任務派發。**<br>❌ 嚴禁直接撰寫或修改任何具體的前後端業務程式碼。 |
| [frontend.md](file:///.agents/frontend.md) | **@frontend**<br>(前端工程師) | `impeccable`<br>`codegraph` | • UI 元件開發與頁面切版<br>• 客戶端狀態管理與互動體驗<br>• 嚴格遵循 `impeccable` 質感規範<br>• 依照 API Contract 串接資料 | **嚴格依照架構師定義之契約串接。**<br>❌ 嚴禁擅自修改後端 API 路由或資料庫結構。<br>⚠️ 缺漏欄位必須向 `@architect` 提出變更申請。 |
| [backend.md](file:///.agents/backend.md) | **@backend**<br>(後端工程師) | `codegraph` | • API 端點實作與商業邏輯<br>• 資料庫操作 (ORM/Migration)<br>• 身分驗證、授權與安全性防護<br>• 核心單元測試與效能調校 | **修改既有函式前必先用 `codegraph` 查影響。**<br>❌ 嚴禁修改前端 UI 樣式、HTML/CSS 與元件。<br>❌ 禁止未通知架構師即私自破壞 API Contract。 |
| [qa-reviewer.md](file:///.agents/qa-reviewer.md) | **@qa-reviewer**<br>(測試與品質工程師) | `playwright-cli`<br>`open-code-review`<br>`codegraph` | • Git 變更之自動化 Code Review<br>• `playwright-cli` 啟動無頭瀏覽器 E2E 驗證<br>• 監控 Console 報錯與 Network 狀態<br>• 定位 Bug 根因與截圖舉證 | **只負責審查、測試、舉證與定位。**<br>❌ 嚴禁直接大幅重寫業務功能程式碼。<br>⚠️ 發現問題必須撰寫結構化報告退回工程師修復。 |

---

## 2. 標準開發工作流程（三階段管線）

專案開發嚴格遵循「契約先行、平行開發、雙重驗收」標準工作流程：

1. **階段一：架構與契約（@architect）**
   - 需求釐清（`ponytail` 拒絕過度設計）➔ 拓撲分析（`codegraph`）➔ 鎖定 API Contract 與 DB Schema ➔ 拆解任務派發。
2. **階段二：平行實作（@frontend & @backend）**
   - **前端**：遵循 `impeccable` 質感規範切版，依 API Contract 串接 Mock / API。
   - **後端**：動工前以 `codegraph` 評估 Blast Radius，實作商業邏輯、API 與單元測試。
3. **階段三：品質審查與驗收（@qa-reviewer）**
   - **審查驗收**：`open-code-review` 代碼審查 + `playwright-cli` 無頭瀏覽器 E2E 驗證與 Console 監控。
   - **流程分流**：
     - **驗收通過** ➔ 產出驗收報告並結案。
     - **發現缺陷** ➔ 產出標準 Bug Report 退回修復（⚠️ **同一缺陷修復 ≤ 2 次允許重試；超過 2 次強制觸發熔斷，喚醒 `@architect` 介入仲裁與架構重估**）。

---

## 3. 除錯鐵律
全體 Agent 在排查問題與撰寫修復程式碼時，必須無條件遵守以下三大鐵律：

### 鐵律一：動工前先看 `ponytail` 與 `codegraph`
1. **先看 `ponytail`（極簡約束）**：
   - 在新增任何功能或修復 Bug 前，優先檢查：專案內是否已有現成函式或工具？能否用最少的新程式碼解決問題？
   - 拒絕為了微小功能引入全新的大型第三方依賴套件。
2. **先看 `codegraph`（衝擊半徑分析）**：
   - 在修改任何已存在的函式、類別、介面或資料庫欄位前，**必須執行 `codegraph explore` 或 `codegraph callers / impact`**。
   - 確認受影響的上下游呼叫者（Callers）、被呼叫者（Callees）與關聯檔案，評估變更風險（Blast Radius），嚴禁盲目修改函式簽名。

### 鐵律二：遇到 Bug 禁止盲猜修復
1. **嚴禁憑直覺盲猜亂改**：禁止在沒有實質證據的情況下隨意更動程式碼，這會破壞既有穩定邏輯並引入次生 Bug。
2. **標準除錯排查鏈**：
   - **Step 1 - 現場取證**：收集完整的報錯訊息、Stack Trace、瀏覽器 Console Log 或 HTTP Network 回應。
   - **Step 2 - 拓撲定位**：透過 `codegraph` 順著調用路徑向上或向下反查，找出觸發異常的確切程式節點。
   - **Step 3 - 根因分析 (RCA)**：釐清問題屬於「輸入邊界未校驗」、「狀態非同步競爭」、「資料型別不合」還是「API Contract 認知分歧」。
   - **Step 4 - 最小代價修復**：以最小異動量精確修復問題，不更動無關邏輯。
   - **Step 5 - 回歸驗證**：修復後透過單元測試或 `playwright-cli` 重現原操作路徑，確認 Bug 已消除且無副作用。
3. **兩次失敗熔斷機制（Circuit Breaker - 防死循環關鍵）**：
   - **若同一缺陷退回修復超過 2 次仍未通過，禁止繼續在工程師與 QA 之間循環**。
   - 連續 2 次修復失敗通常意味著底層 API Contract、資料庫 Schema 或規格設計本身存在邏輯矛盾，繼續踢皮球只會無謂浪費大量的上下文與 Token。
   - 一旦達標，必須**強制觸發熔斷**，直接標記並喚醒 `@architect` 介入仲裁，重新評估系統架構與契約設計後再行分派。

### 鐵律三：嚴守權限邊界，禁止越權干涉
1. **前端不越權修後端**：前端發現資料結構錯誤或缺少欄位時，嚴禁自行修改後端 Controller、ORM 或資料庫，必須提交變更申請由 `@architect` 裁定。
2. **後端不越權改前端**：後端嚴禁觸碰前端樣式、CSS 或元件模板。
3. **QA 不越權寫業務**：QA 角色僅提供詳實的 Bug 重現步驟、截圖與根因指引，由負責該模組的工程師進行代碼修復。

---

## 4. 任務交接協定與標準格式

為確保非同步多 Agent 協作間的資訊完整透明，每當任務完成、需要下游接手或發現 Bug 退回時，**必須使用以下標準 Markdown 格式發布交接摘要**：

### 4.1 標準任務完成交接格式

```markdown
### 📋 任務完成交接報告

- **任務名稱 / ID**：[例如：AUTH-002 使用者登入 API 與前端表單串接]
- **執行角色**：[@frontend / @backend]
- **交接目標角色**：[@qa-reviewer 進行驗收]

#### 1. 完成項目清單
- [x] 完成項目 A：依據 API Contract 實作登入驗證端點
- [x] 完成項目 B：依照 impeccable 規範實作登入視窗與防呆校驗

#### 2. 變更檔案與衝擊半徑分析
- `src/controllers/auth.ts`: 新增 `loginHandler`
- `src/components/LoginForm.tsx`: 切版與表單驗證
- **codegraph 衝擊評估**：經檢查呼叫者僅限於 `src/routes/auth.ts`，無破壞性影響。

#### 3. 自我檢驗結果
- 單元測試：`npm test` 通過 (5/5 passed)
- 本地驗證：手動測試成功登入與密碼錯誤情境

#### 4. 已知限制與跳過項目
- *未實作項目*：第三方 OAuth 登入（留待第二階段迭代）
- *潛在風險*：目前速率限制（Rate Limit）採記憶體計數，重啟後重置

#### 5. 下一步行動建議
- 請 `@qa-reviewer` 使用 `playwright-cli` 進行無頭瀏覽器端對端驗收，並執行 `open-code-review`。
```

---

### 4.2 標準缺陷退回報告格式

```markdown
### 🚨 缺陷退回報告

- **缺陷標題**：[例如：密碼包含特殊字元時登入請求回傳 500 錯誤]
- **提報人**：@qa-reviewer
- **指派修復角色**：[@backend]
- **嚴重等級**：[Critical / Major / Minor]
- **退回次數計數**：[第 1 次 / 第 2 次（⚠️ 連續 2 次未通過將強制觸發熔斷並標記 @architect）]

#### 1. 重現步驟
1. 開啟首頁並點擊「登入」按鈕。
2. 於帳號輸入 `test@example.com`，密碼輸入 `Pass&Word<123>`。
3. 點擊「送出」按鈕。

#### 2. 預期結果 vs 實際結果
- **預期結果**：正常驗證通過，回傳 Token 或提示密碼錯誤 (HTTP 401)。
- **實際結果**：伺服器崩潰回傳 HTTP 500 Internal Server Error。

#### 3. 舉證資料
- **Console Log**：`Uncaught (in promise) Error: Request failed with status code 500`
- **Network Payload**：POST `/api/v1/auth/login` -> 500 (見截圖 `artifacts/login_error.png`)

#### 4. codegraph 根因定位分析
- 透過 `codegraph callers parseCredentials` 追蹤發現，`src/utils/sanitizer.ts` 未跳脫 XML/特殊字元，導致 SQL 查詢建構器拋出未捕獲之例外。

#### 5. 修復要求
- 請 `@backend` 於 `src/utils/sanitizer.ts` 補齊特殊字元轉義處理，並為該字元組合新增單元測試用例。
```
