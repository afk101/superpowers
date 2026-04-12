---
name: using-git-worktrees
description: Use when starting feature work that needs isolation from current workspace or before executing implementation plans - creates isolated git worktrees with smart directory selection and safety verification
---

# 使用 Git Worktrees

## 概述

Git worktrees 创建共享同一仓库的隔离工作空间，允许在不切换的情况下同时处理多个分支。

**核心原则：** 系统化的目录选择 + 安全验证 = 可靠的隔离。

**开始时宣布：** "I'm using the using-git-worktrees skill to set up an isolated workspace."

## 目录选择流程

按照以下优先级顺序：

### 1. 检查现有目录

```bash
# 按优先级顺序检查
ls -d .worktrees 2>/dev/null     # 首选（隐藏）
ls -d worktrees 2>/dev/null      # 备选
```

**如果找到：** 使用该目录。如果两者都存在，`.worktrees` 优先。

### 2. 检查 CLAUDE.md

```bash
grep -i "worktree.*director" CLAUDE.md 2>/dev/null
```

**如果指定了偏好：** 直接使用，无需询问。

### 3. 询问用户

如果没有目录存在且 CLAUDE.md 中没有偏好设置：

```
No worktree directory found. Where should I create worktrees?

1. .worktrees/ (项目本地，隐藏)
2. ~/.config/superpowers/worktrees/<project-name>/ (全局位置)

您希望使用哪个？
```

## 安全验证

### 对于项目本地目录（.worktrees 或 worktrees）

**必须在创建 worktree 之前验证目录已被忽略：**

```bash
# 检查目录是否被忽略（遵循 local、global 和 system gitignore）
git check-ignore -q .worktrees 2>/dev/null || git check-ignore -q worktrees 2>/dev/null
```

**如果未被忽略：**

按照 Jesse 的规则"立即修复问题"：
1. 向 .gitignore 添加适当的行
2. 提交该更改
3. 继续创建 worktree

**为什么至关重要：** 防止意外将 worktree 内容提交到仓库。

### 对于全局目录（~/.config/superpowers/worktrees）

无需 .gitignore 验证 - 完全在项目之外。

## 创建步骤

### 1. 检测项目名称

```bash
project=$(basename "$(git rev-parse --show-toplevel)")
```

### 2. 创建 Worktree

```bash
# 确定完整路径
case $LOCATION in
  .worktrees|worktrees)
    path="$LOCATION/$BRANCH_NAME"
    ;;
  ~/.config/superpowers/worktrees/*)
    path="~/.config/superpowers/worktrees/$project/$BRANCH_NAME"
    ;;
esac

# 使用新分支创建 worktree
git worktree add "$path" -b "$BRANCH_NAME"
cd "$path"
```

### 3. 运行项目设置

自动检测并运行适当的设置：

```bash
# Node.js
if [ -f package.json ]; then npm install; fi

# Rust
if [ -f Cargo.toml ]; then cargo build; fi

# Python
if [ -f requirements.txt ]; then pip install -r requirements.txt; fi
if [ -f pyproject.toml ]; then poetry install; fi

# Go
if [ -f go.mod ]; then go mod download; fi
```

### 4. 验证干净的基线

运行测试以确保 worktree 以干净状态开始：

```bash
# 示例 - 使用适合项目的命令
npm test
cargo test
pytest
go test ./...
```

**如果测试失败：** 报告失败，询问是继续还是调查。

**如果测试通过：** 报告准备就绪。

### 5. 报告位置

```
Worktree ready at <full-path>
Tests passing (<N> tests, 0 failures)
Ready to implement <feature-name>
```

## 快速参考

| 情况 | 操作 |
|-----------|--------|
| `.worktrees/` 存在 | 使用它（验证是否被忽略） |
| `worktrees/` 存在 | 使用它（验证是否被忽略） |
| 两者都存在 | 使用 `.worktrees/` |
| 都不存在 | 检查 CLAUDE.md → 询问用户 |
| 目录未被忽略 | 添加到 .gitignore + 提交 |
| 基线测试失败 | 报告失败 + 询问 |
| 没有 package.json/Cargo.toml | 跳过依赖安装 |

## 常见错误

### 跳过忽略验证

- **问题：** Worktree 内容被跟踪，污染 git 状态
- **修复：** 在创建项目本地 worktree 之前始终使用 `git check-ignore`

### 假设目录位置

- **问题：** 造成不一致，违反项目约定
- **修复：** 遵循优先级：现有 > CLAUDE.md > 询问

### 在测试失败时继续

- **问题：** 无法区分新 bug 和预先存在的问题
- **修复：** 报告失败，获得明确许可后继续

### 硬编码设置命令

- **问题：** 在使用不同工具的项目上会失败
- **修复：** 从项目文件（package.json 等）自动检测

## 示例工作流程

```
You: I'm using the using-git-worktrees skill to set up an isolated workspace.

[检查 .worktrees/ - 存在]
[验证是否被忽略 - git check-ignore 确认 .worktrees/ 已被忽略]
[创建 worktree: git worktree add .worktrees/auth -b feature/auth]
[运行 npm install]
[运行 npm test - 47 passing]

Worktree ready at /Users/jesse/myproject/.worktrees/auth
Tests passing (47 tests, 0 failures)
Ready to implement auth feature
```

## 危险信号

**永远不要：**
- 在未验证被忽略的情况下创建 worktree（项目本地）
- 跳过基线测试验证
- 在测试失败时未经询问就继续
- 在不明确时假设目录位置
- 跳过 CLAUDE.md 检查

**始终：**
- 遵循目录优先级：现有 > CLAUDE.md > 询问
- 验证目录是否被忽略（项目本地）
- 自动检测并运行项目设置
- 验证干净的测试基线

## 集成

**被以下技能调用：**
- **brainstorming**（第 4 阶段）- 当设计被批准且后续需要实施时为必需
- **subagent-driven-development** - 在执行任何任务之前为必需
- **executing-plans** - 在执行任何任务之前为必需
- 任何需要隔离工作空间的技能

**配对使用：**
- **finishing-a-development-branch** - 工作完成后清理时为必需