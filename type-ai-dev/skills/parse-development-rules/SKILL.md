# parse-development-rules

## Description

解析 `DEVELOPMENT_RULES.md`，萃取出可機械化檢查的開發規範並輸出結構化 JSON。

## Input

- `development_rules_path`：`DEVELOPMENT_RULES.md` 完整路徑

## Steps

1. 讀取 `development_rules_path` 全文
2. 依 Markdown 標題分段，識別與下列規則相關的章節：
   - **命名規則**（naming）：檔案、變數、函式、元件、CSS class 等命名慣例
   - **檔案組織**（file_organization）：檔案副檔名、配對檔案（如 `.tsx` 必有 `.test.tsx`）、單一檔案責任範圍
   - **語法慣例**（syntax_conventions）：必須/禁止的語法（如 `const` over `let`、不得使用 `any`）
   - **Pattern 約束**（patterns）：必須遵循的設計 pattern（如 hooks 命名 `useXxx`、Container/Presentational 分離）
   - **禁用 API**（forbidden_apis）：禁止呼叫的 API、套件方法或全域物件（如 `console.log` in production）
   - **其他**（other）：可檢查但無法歸類的條目
3. 對每條規則保留章節錨點作為 `source`
4. 若文件未包含任何可機械化檢查的條目，回傳空結構並由呼叫端的 agent 轉為 `warning`

## Output

```json
{
  "development_rules": {
    "naming": [
      {
        "rule_id": "NAME-001",
        "target": "file:ts",
        "pattern": "camelCase",
        "source": "DEVELOPMENT_RULES.md#命名規則"
      },
      {
        "rule_id": "NAME-002",
        "target": "react_component",
        "pattern": "PascalCase",
        "source": "DEVELOPMENT_RULES.md#命名規則"
      }
    ],
    "file_organization": [
      {
        "rule_id": "FILE-001",
        "rule": "每個 .tsx 必須有同名 .test.tsx",
        "source": "DEVELOPMENT_RULES.md#測試規範"
      }
    ],
    "syntax_conventions": [
      {
        "rule_id": "SYN-001",
        "forbid": "any 型別",
        "source": "DEVELOPMENT_RULES.md#TypeScript 規範"
      }
    ],
    "patterns": [
      {
        "rule_id": "PAT-001",
        "rule": "custom hook 名稱必須以 use 開頭",
        "source": "DEVELOPMENT_RULES.md#React Pattern"
      }
    ],
    "forbidden_apis": [
      {
        "rule_id": "API-001",
        "api": "console.log",
        "context": "production code",
        "source": "DEVELOPMENT_RULES.md#禁用 API"
      }
    ],
    "other": []
  }
}
```

> **欄位說明**
>
> - **rule_id**：本次解析自動產生的唯一識別碼
> - **source**：規則所在的 Markdown 章節錨點
> - 各類別內欄位依規則類型而定；下游 reviewer 需能容忍未知欄位
