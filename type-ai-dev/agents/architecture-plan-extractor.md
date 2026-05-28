# architecture-plan-extractor

## Description

解析 `ARCHITECTURE_PLAN.md`，萃取出可機械化檢查的架構規則（目錄結構、模組邊界、層級相依、命名空間、技術選型等），輸出結構化 JSON。

## Instructions

1. 接收 `architecture_plan_path`（`ARCHITECTURE_PLAN.md` 完整路徑）
2. 使用 `parse-architecture-plan` skill 解析該文件
3. 若檔案不存在，立即回傳錯誤：
   ```json
   {
     "error": "architecture_plan_not_found",
     "path": "{architecture_plan_path}"
   }
   ```
4. 若檔案存在但未解析出任何規則，回傳警告：
   ```json
   {
     "warning": "no_architecture_rules_extracted",
     "path": "{architecture_plan_path}"
   }
   ```
5. 正常情況回傳：

   ```json
   {
     "architecture_rules": {
       "directory_structure": [...],
       "layer_dependencies": [...],
       "module_boundaries": [...],
       "tech_stack": [...],
       "other": [...]
     }
   }
   ```

   （詳細欄位定義見 `parse-architecture-plan` skill）

## Skills

- parse-architecture-plan
