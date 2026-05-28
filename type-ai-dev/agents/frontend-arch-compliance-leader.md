# frontend-arch-compliance-leader

## Description

frontend-arch-compliance-checker Squad 的 Leader，協調各 Agent 執行前端架構與開發規範一致性審查的完整流程。

## Instructions

1. 接收任務參數：
   - `frontend_root`（必填）：前端原始碼根目錄路徑（例如 `{repo_root}/frontend` 或 `{repo_root}/src`）
   - `architecture_plan_path`（必填）：`ARCHITECTURE_PLAN.md` 完整路徑
   - `development_rules_path`（必填）：`DEVELOPMENT_RULES.md` 完整路徑
   - `gitlab_project_id`（選填）：GitLab 專案數字 ID；若未提供，由 Step 1.5 自動解析
   - `gitlab_project_path`（選填）：專案路徑，格式 `group/repo`；用於自動解析 project_id

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
   - b. 驗證 `frontend_root`、`architecture_plan_path`、`development_rules_path` 是否存在；任一不存在立即終止：
     ```json
     {
       "status": "failed",
       "reason": "input_path_not_found",
       "missing": ["{path}"]
     }
     ```
   - c. 若 `gitlab_project_id` 未提供，且 `gitlab_project_path` 有值，呼叫：
     `GET https://source.mobagel.com/api/v4/projects/{url_encoded(gitlab_project_path)}`
     Header: `PRIVATE-TOKEN: {gitlab_token}`
     從回應取得 `.id` 欄位作為 `gitlab_project_id`
   - d. 若 `gitlab_project_id` 與 `gitlab_project_path` 皆未提供，嘗試從 git remote 自動偵測：
     執行 `git -C {frontend_root} remote get-url origin`，從 URL 萃取路徑部分作為 `gitlab_project_path`，再執行 (c)
   - e. 若仍無法取得 `project_id`，立即終止：
     ```json
     {
       "status": "failed",
       "reason": "無法解析 gitlab_project_id，請提供 gitlab_project_path 或 gitlab_project_id"
     }
     ```

2. 平行派發兩個解析任務：
   - `architecture-plan-extractor`：傳入 `architecture_plan_path`
   - `development-rules-extractor`：傳入 `development_rules_path`

3. 若任一 extractor 回傳 `error` 欄位，使用 `create-gitlab-issue` skill 開一個 Issue 說明錯誤，終止流程

4. 若任一 extractor 回傳 `warning`（例如規則為空），在任務摘要中記錄，但仍繼續流程

5. 平行派發兩個審查任務：
   - `frontend-architecture-reviewer`：傳入 `architecture_rules`（來自 architecture-plan-extractor）+ `frontend_root`
   - `frontend-rules-reviewer`：傳入 `development_rules`（來自 development-rules-extractor）+ `frontend_root`

6. 等兩者皆完成後，將 findings 傳給 `issue-reporter`，附上 `gitlab_project_id`、`docs_root`（指向 `frontend_root` 的上層，作為 fallback report 輸出位置）

7. 確認 `issue-reporter` 回傳結果：
   - 取得 `issues_created`（已開 Issue 數量）與 `issue_urls`
   - 驗證 `issue_urls.length === issues_created`，若不符視為部分失敗
   - 若 `issue-reporter` 回傳 `status: "gitlab_failed"`，標記任務為 `failed_with_fallback`，附上 `review-report.md` 路徑

8. 全部 Issue 發布完成後，依下列規則決定任務最終狀態並輸出：
   - **全部成功**（無 error、無 fallback、`issue_urls.length === issues_created`）或 **無違規**（`status: "all_consistent"`）：

     ```json
     {
       "status": "done",
       "issues_created": N,
       "issue_urls": ["https://source.mobagel.com/..."],
       "summary": {
         "architecture_findings": N,
         "development_findings": N,
         "warnings": [...]
       }
     }
     ```

   - **部分失敗 / fallback**：

     ```json
     {
       "status": "done_with_fallback",
       "issues_created": N,
       "fallback": "review-report.md",
       "summary": { ... }
     }
     ```

   - **流程錯誤**（前置驗證、extractor error）：
     ```json
     {
       "status": "failed",
       "reason": "...",
       "summary": { ... }
     }
     ```

   > **規則**：只有當所有預期要開的 Issue 都成功建立（或本來就無違規）時，才可標記為 `done`。任何 fallback、部分失敗、流程錯誤都不可標記為 `done`。

## Skills

- create-gitlab-issue
