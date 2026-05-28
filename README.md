# Multica Agents Design

> 為 [multica](https://github.com/multica-ai/multica) 平台設計可重複使用的 AI Agent 的設計庫與參考架構。

## 這個 Repo 是做什麼的

[multica](https://github.com/multica-ai/multica) 是一個 Managed Agents 平台，讓 AI Agent 像真實的團隊成員一樣接收 Issue、自主執行任務、回報進度。

本 Repo 的用途是：

- 設計並記錄可在 multica 中部署的 Agent 行為與職責
- 建立可重複使用的 Skill，供整個團隊的 Agent 複用
- 定義 Agent 之間的協作模式（Squads、Orchestration）
- 作為 Agent 設計的單一事實來源（Source of Truth）

## 核心概念對應

| 本 Repo 的概念        | multica 對應概念                          |
| --------------------- | ----------------------------------------- |
| Agent 設計文件        | Agent 在 multica 中的 Profile 與職責      |
| Skill 定義            | multica Reusable Skills                   |
| Orchestration Pattern | multica Squads + Squad Leader             |
| Autopilot 規格        | multica Autopilots（cron / webhook 觸發） |

## 目錄結構

```
multica-agents-design/
└── workspace/           # 工作空間根目錄
    ├── agents/          # 各 Agent 的設計規格（職責、輸入輸出、限制）
    ├── skills/          # 可重複使用的 Skill 定義（每個 Skill 一個資料夾，內含 SKILL.md）
    ├── squads/          # Squad 組成與任務路由設計
    └── autopilots/      # 定期自動化任務規格
```

## 設計一個 Agent 的流程

1. **定義職責** — 在 `workspace/agents/` 下建立設計文件，描述這個 Agent 做什麼、不做什麼
2. **設計 Skills** — 將可重用的能力抽成 `workspace/skills/`，供多個 Agent 共用
3. **組成 Squad** — 在 `workspace/squads/` 下定義多個 Agent 如何協作，並指定 Squad Leader 的路由邏輯
4. **設定 Autopilot**（選用）— 在 `workspace/autopilots/` 下定義觸發條件與排程

## 支援的 Agent Runtime

multica 目前支援以下 Agent CLI，本設計庫的規格應對應其中一種或多種：

- Claude Code
- OpenAI Codex
- GitHub Copilot CLI
- Gemini
- OpenClaw / OpenCode
- Cursor Agent、Kimi、Kiro CLI 等

## 相關資源

- [multica GitHub](https://github.com/multica-ai/multica)
- [multica CLI & Daemon 指南](https://github.com/multica-ai/multica/blob/main/CLI_AND_DAEMON.md)
- [multica 架構規範 CLAUDE.md](https://github.com/multica-ai/multica/blob/main/CLAUDE.md)

## License

MIT — see [LICENSE](LICENSE).
