# scan-frontend-against-rules

## Description

依給定的規則集，掃描前端原始碼目錄，回報缺漏（應存在而不存在）與違反（存在但與規則不符）之處。

## Input

- `rules`：來自 `parse-architecture-plan` 或 `parse-development-rules` 的規則 JSON
- `source_root`：前端原始碼根目錄
- `rule_category`：`"architecture"` 或 `"development"`，決定掃描策略

## Steps

1. 列出 `source_root` 下所有原始碼檔案（含子目錄），排除：
   - `node_modules/`
   - `dist/`、`build/`、`.next/`、`.cache/` 等建置產物
   - `.git/`
   - 依專案 `.gitignore` 中標示忽略者

2. 建立檔案索引：
   - 路徑
   - 副檔名
   - 內容（用於語法 / API / import 比對）
   - 透過 AST 或 regex 擷取的 `import`/`require` 清單

3. **依 `rule_category` 套用對應檢查策略：**

   ### architecture 類
   - `directory_structure`：檢查 `must_exist` 路徑是否存在 → 不存在計為 `missing`
   - `layer_dependencies`：分析每個檔案的 `import`，若來源層 import 到 `forbid_to` 列出的層 → 計為 `inconsistent`
   - `module_boundaries`：若有檔案跨模組 import 該模組的非 `public_entry` 檔 → 計為 `inconsistent`
   - `tech_stack`：
     - `required`：檢查 `package.json` 是否包含且版本符合 → 不符計為 `missing` 或 `inconsistent`
     - `forbidden`：檢查 `package.json` 或 import 中是否出現 → 出現計為 `inconsistent`

   ### development 類
   - `naming`：依 `target` 與 `pattern` 檢查檔名 / 識別符是否符合 → 不符計為 `inconsistent`
   - `file_organization`：依規則描述檢查配對檔案是否齊全 → 缺失計為 `missing`
   - `syntax_conventions`：以 regex 或語法檢查偵測 `forbid` 條件 → 命中計為 `inconsistent`
   - `patterns`：依規則描述檢查命名/結構是否符合 → 不符計為 `inconsistent`
   - `forbidden_apis`：以 regex 偵測 `api` 字串在原始碼中的出現 → 命中計為 `inconsistent`

4. 對於規則描述過於抽象、無法以靜態檢查驗證者，輸出到 `skipped` 並標註原因，**不**計入 `missing`/`inconsistent`

5. 輸出結果：

   > **欄位說明**：
   >
   > - **item**：違規對象（檔案路徑、識別符、import 來源 → 目標）
   > - **rule_id**：對應的規則 ID（來自 parse skill）
   > - **source**：規則出處（Markdown 章節錨點）
   > - **expected / found**：僅 `inconsistent` 使用，描述期望值與實際值
   > - **detail**：僅 `inconsistent` 使用，補充說明違規原因

   ```json
   {
     "missing": [
       {
         "item": "src/features",
         "rule_id": "DIR-001",
         "source": "ARCHITECTURE_PLAN.md#目錄結構"
       }
     ],
     "inconsistent": [
       {
         "item": "src/pages/Home.tsx -> src/infrastructure/db.ts",
         "rule_id": "DEP-001",
         "expected": "pages 不得直接相依 infrastructure",
         "found": "存在 import './infrastructure/db'",
         "source": "ARCHITECTURE_PLAN.md#層級相依",
         "detail": "違反 pages → application → infrastructure 的相依方向"
       }
     ],
     "skipped": [
       {
         "rule_id": "PAT-009",
         "reason": "規則描述過於抽象，無法自動驗證",
         "source": "DEVELOPMENT_RULES.md#其他"
       }
     ]
   }
   ```
