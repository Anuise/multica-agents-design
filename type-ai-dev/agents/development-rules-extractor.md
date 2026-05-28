# development-rules-extractor

## Description

解析 `DEVELOPMENT_RULES.md`，萃取可機械化檢查的開發規範（命名規則、檔案組織、語法慣例、Pattern 約束、禁用 API 等），輸出結構化 JSON。

## Instructions

1. 接收 `development_rules_path`（`DEVELOPMENT_RULES.md` 完整路徑）
2. 使用 `parse-development-rules` skill 解析該文件
3. 若檔案不存在，立即回傳錯誤：
   ```json
   {
     "error": "development_rules_not_found",
     "path": "{development_rules_path}"
   }
   ```
4. 若檔案存在但未解析出任何規則，回傳警告：
   ```json
   {
     "warning": "no_development_rules_extracted",
     "path": "{development_rules_path}"
   }
   ```
5. 正常情況回傳：

   ```json
   {
     "development_rules": {
       "naming": [...],
       "file_organization": [...],
       "syntax_conventions": [...],
       "patterns": [...],
       "forbidden_apis": [...],
       "other": [...]
     }
   }
   ```

   （詳細欄位定義見 `parse-development-rules` skill）

## Skills

- parse-development-rules
