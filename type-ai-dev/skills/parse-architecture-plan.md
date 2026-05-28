# parse-architecture-plan

## Description

解析 `ARCHITECTURE_PLAN.md`，萃取出可機械化檢查的架構規則並輸出結構化 JSON。

## Input

- `architecture_plan_path`：`ARCHITECTURE_PLAN.md` 完整路徑

## Steps

1. 讀取 `architecture_plan_path` 全文
2. 依 Markdown 標題分段，識別與下列規則相關的章節：
   - **目錄結構**（directory_structure）：列出必須存在的資料夾與其用途
   - **層級相依**（layer_dependencies）：層與層之間允許/禁止的 import 方向
   - **模組邊界**（module_boundaries）：模組對外暴露的入口、不允許跨模組存取的內部檔案
   - **技術選型**（tech_stack）：必須使用 / 禁止使用的框架、套件、語言版本
   - **其他**（other）：無法歸類但具明確檢查條件的規則
3. 對每條規則，保留來源章節錨點作為 `source`（例：`ARCHITECTURE_PLAN.md#目錄結構`）
4. 若文件未包含任何可機械化檢查的條目，回傳空結構並由呼叫端的 agent 轉為 `warning`

## Output

```json
{
  "architecture_rules": {
    "directory_structure": [
      {
        "rule_id": "DIR-001",
        "must_exist": "src/features",
        "purpose": "功能模組根目錄",
        "source": "ARCHITECTURE_PLAN.md#目錄結構"
      }
    ],
    "layer_dependencies": [
      {
        "rule_id": "DEP-001",
        "from": "pages",
        "forbid_to": ["infrastructure"],
        "source": "ARCHITECTURE_PLAN.md#層級相依"
      }
    ],
    "module_boundaries": [
      {
        "rule_id": "MOD-001",
        "module": "features/auth",
        "public_entry": "index.ts",
        "source": "ARCHITECTURE_PLAN.md#模組邊界"
      }
    ],
    "tech_stack": [
      {
        "rule_id": "TECH-001",
        "required": "react@>=18",
        "source": "ARCHITECTURE_PLAN.md#技術選型"
      },
      {
        "rule_id": "TECH-002",
        "forbidden": "moment",
        "reason": "改用 date-fns",
        "source": "ARCHITECTURE_PLAN.md#技術選型"
      }
    ],
    "other": []
  }
}
```

> **欄位說明**
>
> - **rule_id**：本次解析自動產生的唯一識別碼，方便後續引用與去重
> - **source**：規則所在的 Markdown 章節錨點
> - 各類別內欄位依規則類型而定；下游 reviewer 需能容忍未知欄位
