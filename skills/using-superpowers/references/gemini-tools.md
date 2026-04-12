# Gemini CLI 工具映射

Skills 使用 Claude Code 的工具名称。当你在 skill 中遇到这些工具时，请使用你所在平台的对应工具：

| Skill 引用 | Gemini CLI 对应工具 |
|-----------------|----------------------|
| `Read` (文件读取) | `read_file` |
| `Write` (文件创建) | `write_file` |
| `Edit` (文件编辑) | `replace` |
| `Bash` (运行命令) | `run_shell_command` |
| `Grep` (搜索文件内容) | `grep_search` |
| `Glob` (按名称搜索文件) | `glob` |
| `TodoWrite` (任务跟踪) | `write_todos` |
| `Skill` tool (调用 skill) | `activate_skill` |
| `WebSearch` | `google_web_search` |
| `WebFetch` | `web_fetch` |
| `Task` tool (分发子代理) | 无对应工具 — Gemini CLI 不支持子代理 |

## 无子代理支持

Gemini CLI 没有与 Claude Code 的 `Task` 工具对应的功能。依赖于子代理分发（`subagent-driven-development`、`dispatching-parallel-agents`）的 Skills 将通过 `executing-plans` 回退到单会话执行模式。

## Gemini CLI 额外工具

以下工具在 Gemini CLI 中可用，但在 Claude Code 中没有对应工具：

| 工具 | 用途 |
|------|---------|
| `list_directory` | 列出文件和子目录 |
| `save_memory` | 将事实持久化保存到 GEMINI.md 中，跨会话保持 |
| `ask_user` | 向用户请求结构化输入 |
| `tracker_create_task` | 丰富的任务管理功能（创建、更新、列出、可视化） |
| `enter_plan_mode` / `exit_plan_mode` | 在进行更改前切换到只读研究模式 |
