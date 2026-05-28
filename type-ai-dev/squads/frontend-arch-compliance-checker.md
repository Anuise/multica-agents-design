# frontend-arch-compliance-checker

## Description

掃描前端程式碼，比對 `ARCHITECTURE_PLAN.md` 與 `DEVELOPMENT_RULES.md` 兩份規劃文件，回報違反之處並自動開立 GitLab Issue。

## Leader Agent

frontend-arch-compliance-leader

## Members

- architecture-plan-extractor（解析 ARCHITECTURE_PLAN.md，萃取架構/邊界/相依規則）
- development-rules-extractor（解析 DEVELOPMENT_RULES.md，萃取命名/語法/Pattern 規則）
- frontend-architecture-reviewer（依架構規則掃描前端原始碼，回報違規）
- frontend-rules-reviewer（依開發規範掃描前端原始碼，回報違規）
- issue-reporter（整合審查結果，開立 GitLab Issues）
