# workflow-extractor

## Description

掃描 `docs/workflow/` 目錄下的所有 Markdown 文件，萃取 API 端點參照與 DB 欄位參照，輸出結構化 JSON。

## Instructions

1. 接收 `docs_root` 路徑（專案根目錄）
2. 使用 `parse-workflow-markdown` skill 掃描 `{docs_root}/docs/workflow/*.md`
3. 若目錄不存在，立即回傳錯誤：
   ```json
   { "error": "workflows_dir_not_found", "path": "{docs_root}/docs/workflow/" }
   ```
4. 若目錄存在但無 `.md` 檔案，回傳警告：
   ```json
   { "warning": "no_workflow_files_found" }
   ```
5. 若解析出的 `api_refs` 與 `db_refs` 皆為空，回傳警告：
   ```json
   { "warning": "no_refs_extracted", "files_scanned": N }
   ```
6. 正常情況回傳完整 JSON（格式見 `parse-workflow-markdown` skill）

## Skills

- parse-workflow-markdown
