# frontend-rules-reviewer

## Description

依開發規範掃描前端原始碼，回報違反 `DEVELOPMENT_RULES.md` 之處（命名、檔案組織、語法慣例、禁用 API 等）。

## Instructions

1. 接收：
   - `development_rules`（來自 `development-rules-extractor`）
   - `frontend_root`：前端原始碼根目錄

2. 若 `frontend_root` 不存在，回傳：

   ```json
   {
     "error": "frontend_root_not_found",
     "missing": [],
     "inconsistent": [],
     "note": "前端原始碼目錄不存在"
   }
   ```

3. 若 `development_rules` 為空，回傳：

   ```json
   {
     "missing": [],
     "inconsistent": [],
     "note": "no_development_rules_to_check"
   }
   ```

4. 使用 `scan-frontend-against-rules` skill，傳入：
   - `rules` = `development_rules`
   - `source_root` = `frontend_root`
   - `rule_category` = `"development"`

5. 回傳結果：
   ```json
   {
     "missing": [
       {
         "item": "src/components/MyButton.tsx 缺少對應的 .test.tsx",
         "source": "DEVELOPMENT_RULES.md#測試規範"
       }
     ],
     "inconsistent": [
       {
         "item": "src/utils/parse_data.ts",
         "expected": "camelCase 檔名（parseData.ts）",
         "found": "snake_case 檔名（parse_data.ts）",
         "source": "DEVELOPMENT_RULES.md#命名規則",
         "detail": "違反前端檔名 camelCase 規則"
       }
     ]
   }
   ```

## Skills

- scan-frontend-against-rules
