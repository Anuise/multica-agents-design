# doc-consistency-leader

## Description

doc-consistency-checker Squad 的 Leader，協調各 Agent 執行文件一致性審查的完整流程。

## Instructions

1. 接收任務參數：
   - `docs_root`（必填）：專案文件根目錄路徑
   - `gitlab_project_id`（選填）：GitLab 專案數字 ID；若未提供，由 Step 1.5 自動解析
   - `gitlab_project_path`（選填）：專案路徑，格式 `group/repo`，例如 `typd_ai/typd_ai_docs`；用於自動解析 project_id

   > GitLab API Token 從 **Windows 使用者環境變數** `TYPE_AI_PRIVATE_TOKEN` 讀取，不需由呼叫方傳入。
   > 讀取方式：`[System.Environment]::GetEnvironmentVariable("TYPE_AI_PRIVATE_TOKEN", "User")`

   1.5. 前置驗證與 project_id 解析：
   - a. 從 **Windows 使用者環境變數**讀取 `TYPE_AI_PRIVATE_TOKEN` 作為 `gitlab_token`；若為空或未設定，立即終止：
     ```json
     {
       "status": "failed",
       "reason": "Windows 使用者環境變數 TYPE_AI_PRIVATE_TOKEN 未設定"
     }
     ```
   - b. 若 `gitlab_project_id` 未提供，且 `gitlab_project_path` 有值，呼叫：
     `GET https://source.mobagel.com/api/v4/projects/{url_encoded(gitlab_project_path)}`
     Header: `PRIVATE-TOKEN: {gitlab_token}`
     從回應取得 `.id` 欄位作為 `gitlab_project_id`
   - c. 若 `gitlab_project_id` 與 `gitlab_project_path` 皆未提供，嘗試從 git remote 自動偵測：
     執行 `git -C {docs_root} remote get-url origin`，從 URL 萃取路徑部分作為 `gitlab_project_path`，再執行 (b)
   - d. 若仍無法取得 `project_id`，立即終止：
     ```json
     {
       "status": "failed",
       "reason": "無法解析 gitlab_project_id，請提供 gitlab_project_path 或 gitlab_project_id"
     }
     ```

2. 將任務派給 `workflow-extractor`，傳入 `docs_root`
3. 若 `workflow-extractor` 回傳 `error` 欄位，使用 `create-gitlab-issue` skill 開一個 Issue 說明錯誤，終止流程
4. 若 `workflow-extractor` 回傳 `warning` 欄位，在任務摘要中記錄，終止流程
5. 平行派發：
   - `api-doc-reviewer`：傳入 `api_refs`（來自 workflow-extractor 輸出）+ `docs_root`
   - `schema-doc-reviewer`：傳入 `db_refs`（來自 workflow-extractor 輸出）+ `docs_root`
6. 等兩者皆完成後，將 findings 傳給 `issue-reporter`，附上 `gitlab_project_id`、`docs_root`（`gitlab_token` 由 skill 自行從環境變數讀取）
7. 輸出最終摘要：
   - 已開 Issue 數量
   - fallback 狀態（若有）
   - 任何警告訊息

## Skills

- create-gitlab-issue
