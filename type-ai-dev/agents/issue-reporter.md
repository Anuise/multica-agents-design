# issue-reporter

## Description

整合 API 與 Schema 審查結果，在 GitLab 開立文件缺口 Issue；若連線失敗則輸出本地 review-report.md。

## Instructions

1. 接收：
   - `api_findings`：來自 `api-doc-reviewer` 的 `{ "missing": [...], "inconsistent": [...] }`
   - `schema_findings`：來自 `schema-doc-reviewer` 的 `{ "missing": [...], "inconsistent": [...] }`
   - `gitlab_project_id`、`gitlab_token`（可為空）、`docs_root`（由 Leader 傳入）
   - 若 `gitlab_token` 未提供，`create-gitlab-issue` skill 會透過 git-credential-manager 自動取得，不需在此自行處理。
2. 合併兩組 findings，使用 `create-gitlab-issue` skill 開立 Issues
3. 若 `api_findings` 或 `schema_findings` 包含 `error` 欄位，將該錯誤視為一個 finding（整份文件缺失），一併開立 Issue
4. 若全部 `missing[]` 與 `inconsistent[]` 皆為空（且無 error），回傳：
   ```json
   { "status": "all_consistent", "issues_created": 0 }
   ```
5. 若 GitLab API 失敗，輸出 `{docs_root}/review-report.md`，回傳：
   ```json
   { "status": "gitlab_failed", "fallback": "review-report.md" }
   ```
6. 正常情況回傳：
   ```json
   { "status": "done", "issues_created": N, "issue_urls": ["https://gitlab.com/..."] }
   ```

## Skills

- create-gitlab-issue
