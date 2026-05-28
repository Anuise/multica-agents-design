# frontend-architecture-reviewer

## Description

依架構規則掃描前端原始碼，回報違反 `ARCHITECTURE_PLAN.md` 之處（如目錄結構錯誤、跨層相依、模組邊界違反、技術選型偏離）。

## Instructions

1. 接收：
   - `architecture_rules`（來自 `architecture-plan-extractor`）
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

3. 若 `architecture_rules` 為空（或所有子陣列皆空），回傳：

   ```json
   {
     "missing": [],
     "inconsistent": [],
     "note": "no_architecture_rules_to_check"
   }
   ```

4. 使用 `scan-frontend-against-rules` skill，傳入：
   - `rules` = `architecture_rules`
   - `source_root` = `frontend_root`
   - `rule_category` = `"architecture"`

5. 回傳結果：
   ```json
   {
     "missing": [
       {
         "item": "目錄 src/features 不存在",
         "source": "ARCHITECTURE_PLAN.md#目錄結構"
       }
     ],
     "inconsistent": [
       {
         "item": "src/pages/Home.tsx 直接 import src/infrastructure/db.ts",
         "expected": "pages 層不得直接相依 infrastructure 層",
         "found": "存在跨層 import",
         "source": "ARCHITECTURE_PLAN.md#層級相依",
         "detail": "違反 pages → application → infrastructure 的相依方向"
       }
     ]
   }
   ```

## Skills

- scan-frontend-against-rules
