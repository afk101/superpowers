# Codex 工具映射

Skills 使用 Claude Code 工具名称。当你在 skill 中遇到这些名称时,请使用你平台的等效工具:

| Skill 引用 | Codex 等效工具 |
|-----------------|------------------|
| `Task` 工具 (分派子代理) | `spawn_agent` (参见[命名代理分派](#named-agent-dispatch)) |
| 多个 `Task` 调用 (并行) | 多个 `spawn_agent` 调用 |
| Task 返回结果 | `wait` |
| Task 自动完成 | `close_agent` 以释放槽位 |
| `TodoWrite` (任务跟踪) | `update_plan` |
| `Skill` 工具 (调用 skill) | Skills 原生加载 — 直接遵循指令即可 |
| `Read`, `Write`, `Edit` (文件) | 使用你的原生文件工具 |
| `Bash` (运行命令) | 使用你的原生 shell 工具 |

## 子代理分派需要多代理支持

在你的 Codex 配置文件 (`~/.codex/config.toml`) 中添加:

```toml
[features]
multi_agent = true
```

这将为 `dispatching-parallel-agents` 和 `subagent-driven-development` 等 skills 启用 `spawn_agent`、`wait` 和 `close_agent` 功能。

## 命名代理分派

Claude Code skills 引用命名代理类型,如 `superpowers:code-reviewer`。
Codex 没有命名代理注册表 — `spawn_agent` 从内置角色(`default`、`explorer`、`worker`)创建通用代理。

当 skill 要求分派命名代理类型时:

1. 找到代理的提示文件 (例如 `agents/code-reviewer.md` 或 skill 的本地提示模板,如 `code-quality-reviewer-prompt.md`)
2. 读取提示内容
3. 填充任何模板占位符 (`{BASE_SHA}`, `{WHAT_WAS_IMPLEMENTED}` 等)
4. 使用填充后的内容作为 `message` 参数来生成一个 `worker` 代理

| Skill 指令 | Codex 等效操作 |
|-------------------|------------------|
| `Task tool (superpowers:code-reviewer)` | 使用 `code-reviewer.md` 内容的 `spawn_agent(agent_type="worker", message=...)` |
| `Task tool (general-purpose)` 及内联提示 | 使用相同提示的 `spawn_agent(message=...)` |

### 消息框架

`message` 参数是用户级输入,而非系统提示。按以下方式构建它以最大化指令遵循度:

```
Your task is to perform the following. Follow the instructions below exactly.

<agent-instructions>
[从代理的 .md 文件填充的提示内容]
</agent-instructions>

Execute this now. Output ONLY the structured response following the format
specified in the instructions above.
```

- 使用任务委派框架 ("Your task is...") 而非角色框架 ("You are...")
- 用 XML 标签包裹指令 — 模型会将标签块视为权威性的
- 以明确的执行指令结束,以防止指令被概括化

### 此变通方案何时可以移除

此方法弥补了 Codex 的插件系统尚不支持 `plugin.json` 中的 `agents` 字段的问题。当 `RawPluginManifest` 获得 `agents` 字段后,插件可以符号链接到 `agents/` (镜像现有的 `skills/` 符号链接),skills 就可以直接分派命名代理类型了。

## 环境检测

创建 worktree 或完成分支的 skills 应在继续之前使用只读 git 命令检测其环境:

```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
BRANCH=$(git branch --show-current)
```

- `GIT_DIR != GIT_COMMON` → 已在链接的 worktree 中 (跳过创建)
- `BRANCH` 为空 → detached HEAD (无法从沙箱进行分支/推送/PR 操作)

参见 `using-git-worktrees` 第 0 步和 `finishing-a-development-branch` 第 1 步,了解每个 skill 如何使用这些信号。

## Codex App 完成

当沙箱阻止分支/推送操作时 (在外部管理的 worktree 中处于 detached HEAD 状态),代理提交所有工作并通知用户使用 App 的原生控件:

- **"Create branch"** — 命名分支,然后通过 App UI 提交/推送/创建 PR
- **"Hand off to local"** — 将工作转移到用户的本地检出

代理仍然可以运行测试、暂存文件,并输出建议的分支名称、提交消息和 PR 描述供用户复制。
