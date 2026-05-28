# parse-workflow-markdown

## Description

從 workflow Markdown 文件中萃取 API 端點參照與 DB 資料表欄位參照，輸出結構化 JSON。

## Input

- `docs_root` (string): 掃描目標的根目錄路徑。skill 將讀取 `{docs_root}/docs/workflow/` 下的所有 `*.md` 檔案。

## Steps

1. 讀取 `{docs_root}/docs/workflow/` 下的所有 `*.md` 檔案
2. 掃描每個檔案，以正規表達式識別 HTTP method + path 模式：
   - 匹配模式：`(GET|POST|PUT|PATCH|DELETE)\s+(/[\w/:.-]+)`
   - 擷取鄰近的描述文字作為 `description`
3. 掃描每個檔案，識別 DB 資料表與欄位描述：
   - 匹配模式：表名（`\w+ 表`、`\w+ table`）與欄位（`\w+ 欄位`、`表名.欄位名`）
   - 若有型別資訊（如 `string`、`integer`、`enum`），一併擷取
4. 合併相同 endpoint 或 table 的參照，去除重複：
   - API 參照去重鍵：`{method}+{path}` （例：`POST+/orders`）
   - DB 參照去重鍵：`{table}+{field.name}` （例：`orders+status`）
5. 輸出 JSON：
   - 若檔案無法讀取，跳過該檔案並繼續處理其他檔案
   - 如有任何檔案被跳過，將其檔名列入 `skipped_files` 陣列
   ```json
   {
     "api_refs": [
       {
         "method": "POST",
         "path": "/orders",
         "description": "建立訂單",
         "source": "create-order.md"
       }
     ],
     "db_refs": [
       {
         "table": "orders",
         "fields": [{ "name": "status", "type": "enum" }],
         "source": "create-order.md"
       }
     ],
     "skipped_files": []
   }
   ```
