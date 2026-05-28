# api-doc-reviewer

## Description
接收 API 參照清單，比對 `docs/api/` 目錄下的 Markdown 文件，回報覆蓋缺口與內容不一致。

## Instructions
1. 接收 `api_refs`（來自 `workflow-extractor`，格式見 `parse-workflow-markdown` skill 的輸出）與 `docs_root` 路徑
2. 若 `{docs_root}/docs/api/` 不存在，回傳：
   ```json
   { "error": "api_docs_dir_not_found", "missing": [], "inconsistent": [], "note": "整份文件目錄不存在" }
   ```
3. 使用 `compare-doc-coverage` skill，傳入：
   - `refs` = `api_refs`
   - `docs_dir` = `{docs_root}/docs/api/`
4. 回傳結果：
   ```json
   { "missing": [...], "inconsistent": [...] }
   ```

## Skills
- compare-doc-coverage
