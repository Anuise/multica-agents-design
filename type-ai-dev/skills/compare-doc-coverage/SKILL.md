# compare-doc-coverage

## Description

比對參照清單與目標 Markdown 文件目錄，回報覆蓋缺口與內容不一致之處。

## Input

- `refs`：來自 `parse-workflow-markdown` 的 `api_refs` 或 `db_refs` 陣列
- `docs_dir`：目標文件目錄路徑（例如 `{docs_root}/docs/api/` 或 `{docs_root}/docs/db-schema/`）

## Steps

1. 讀取 `docs_dir/*.md` 所有文件，建立可搜尋的文字索引
2. **覆蓋率檢查**：對每個 ref，在索引中搜尋對應描述是否存在
   - API ref：搜尋 `{method} {path}` 或語意相近的表達
   - DB ref：搜尋 `{table}` 與每個 `{field.name}` 是否都有描述
3. **一致性檢查**：若找到對應描述，比對：
   - API：method、path params、功能描述是否語意吻合
   - DB：欄位名稱、型別是否與文件一致
4. 輸出結果：

   > **欄位說明**：
   >
   > - **item**：API 參照格式 `{method} {path}`（如 `POST /orders`），或 DB 參照格式 `{table}.{field}`（如 `orders.status`）
   > - **source**：來源 workflow 檔案名稱

   ```json
   {
     "missing": [
       { "item": "POST /orders", "source": "create-order.md" },
       { "item": "orders.created_at", "source": "create-order.md" }
     ],
     "inconsistent": [
       {
         "item": "orders.status",
         "expected": "enum",
         "found": "string",
         "source": "create-order.md",
         "detail": "workflow 描述為 enum，但 schema 文件標示為 string"
       }
     ]
   }
   ```

   > **注意**：`detail` 欄位僅出現在 `inconsistent` 項目中，用於說明期望值與實際值的差異。
