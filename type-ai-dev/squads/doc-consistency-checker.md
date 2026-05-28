# doc-consistency-checker

## Description
掃描 workflow Markdown 文件，比對 API 文件與 DB schema 文件的覆蓋率與一致性，對缺漏自動開立 GitLab Issue。

## Leader Agent
doc-consistency-leader

## Members
- workflow-extractor（解析 workflow 文件，萃取 API 與 DB 參照清單）
- api-doc-reviewer（比對 API Markdown 文件，回報缺漏與不一致）
- schema-doc-reviewer（比對 DB schema Markdown 文件，回報缺漏與不一致）
- issue-reporter（整合審查結果，開立 GitLab Issues）
