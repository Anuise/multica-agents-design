# create-gitlab-issue

## Description

根據文件審查結果，在 GitLab 專案中開立格式統一的 Issue，若 API 失敗則 fallback 輸出本地報告。

## Input

- `findings`：包含 `missing[]` 與 `inconsistent[]` 的審查結果（來自 `compare-doc-coverage`）
- `gitlab_project_id`：GitLab 專案 ID
- `docs_root`：專案根目錄路徑（用於 fallback 報告輸出）

> GitLab API Token 從 **Windows 使用者環境變數** `TYPE_AI_PRIVATE_TOKEN` 讀取，需有 `api` scope 的 PAT。
> 設定方式：系統內容 → 進階 → 環境變數 → 使用者變數 → 新增 `TYPE_AI_PRIVATE_TOKEN`

## Steps

0. 從 **Windows 使用者環境變數**讀取 `TYPE_AI_PRIVATE_TOKEN` 作為 `gitlab_token`（`[System.Environment]::GetEnvironmentVariable("TYPE_AI_PRIVATE_TOKEN", "User")`）；若為空或未設定，立即終止並回傳錯誤：
   ```json
   {
     "status": "failed",
     "reason": "Windows 使用者環境變數 TYPE_AI_PRIVATE_TOKEN 未設定，請至系統環境變數新增具 api scope 的 GitLab PAT"
   }
   ```
   不進行任何 API 呼叫，也不輸出 review-report.md。
1. 對每個 finding，判斷問題類型並組合 Issue 標題：
   - `missing` 項目：`[Doc Gap] {文件類型} 缺少 {item} 的描述`
   - `inconsistent` 項目：`[Doc Inconsistency] {item} 描述與 workflow 不符`
   - 文件類型根據 item 格式判斷：`{method} {path}` 為 API 文件，`{table}.{field}` 為 DB schema
2. 組合 Issue 內文，包含：
   - 問題說明（描述缺什麼或哪裡不一致，`inconsistent` 項目附上 `detail`）
   - 來源 workflow 檔案（`source` 欄位）
   - 建議補齊的內容（根據 workflow 描述建議應有的文字）
3. 合併來自多個 workflow 的同一缺漏：若相同 `item` 出現在多個 `source`，合併為一個 Issue，列出所有來源
4. 呼叫 GitLab API 建立 Issue：
   - Endpoint：`POST /api/v4/projects/{gitlab_project_id}/issues`
   - Header：`PRIVATE-TOKEN: {gitlab_token}`
   - Body：`{ "title": "...", "description": "...", "labels": "doc-gap" }` 或 `"labels": "doc-inconsistency"`
5. 若 GitLab API 呼叫失敗，將所有待建 Issues 輸出為 `{docs_root}/review-report.md`，格式：

   ```markdown
   # Doc Consistency Review Report

   ## Doc Gaps

   - **[Doc Gap] API 文件缺少 POST /orders 的描述**
     來源：create-order.md

   ## Doc Inconsistencies

   - **[Doc Inconsistency] orders.status 描述與 workflow 不符**
     來源：create-order.md
     詳情：workflow 描述為 enum，但 schema 文件標示為 string
   ```
