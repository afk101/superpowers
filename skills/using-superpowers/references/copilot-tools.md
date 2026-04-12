# Copilot CLI Tool Mapping

Skills 使用 Claude Code 工具名称。当你在 skill 中遇到这些工具时，请使用你所对应平台的等效工具：

| Skill 引用 | Copilot CLI 等效工具 |
|-----------------|----------------------|
| `Read` (文件读取) | `view` |
| `Write` (文件创建) | `create` |
| `Edit` (文件编辑) | `edit` |
| `Bash` (运行命令) | `bash` |
| `Grep` (搜索文件内容) | `grep` |
| `Glob` (按名称搜索文件) | `glob` |
| `Skill` 工具 (调用 skill) | `skill` |
| `WebFetch` | `web_fetch` |
| `Task` 工具 (分发子代理) | `task` (参见 [Agent types](#agent-types)) |
| 多个 `Task` 调用 (并行) | 多个 `task` 调用 |
| Task 状态/输出 | `read_agent`, `list_agents` |
| `TodoWrite` (任务跟踪) | `sql` 及内置的 `todos` 表 |
| `WebSearch` | 无等效工具 — 使用 `web_fetch` 配合搜索引擎 URL |
| `EnterPlanMode` / `ExitPlanMode` | 无等效工具 — 保持在主会话中 |

## Agent types

Copilot CLI 的 `task` 工具接受一个 `agent_type` 参数：

| Claude Code 代理 | Copilot CLI 等效代理 |
|-------------------|----------------------|
| `general-purpose` | `"general-purpose"` |
| `Explore` | `"explore"` |
| 命名插件代理 (例如 `superpowers:code-reviewer`) | 从已安装的插件中自动发现 |

## Async shell sessions

Copilot CLI 支持持久化的异步 shell 会话，这在 Claude Code 中没有直接对应的等效功能：

| 工具 | 用途 |
|------|---------|
| `bash` 并设置 `async: true` | 在后台启动一个长时间运行的命令 |
| `write_bash` | 向运行中的异步会话发送输入 |
| `read_bash` | 从异步会话读取输出 |
| `stop_bash` | 终止异步会话 |
| `list_bash` | 列出所有活动的 shell 会话 |

## Additional Copilot CLI tools

| 工具 | 用途 |
|------|---------|
| `store_memory` | 持久化存储关于代码库的事实，供未来会话使用 |
| `report_intent` | 用当前意图更新 UI 状态栏 |
| `sql` | 查询会话的 SQLite 数据库 (todos, metadata) |
| `fetch_copilot_cli_documentation` | 查阅 Copilot CLI 文档 |
| GitHub MCP 工具 (`github-mcp-server-*`) | 原生 GitHub API 访问 (issues, PRs, 代码搜索) |
