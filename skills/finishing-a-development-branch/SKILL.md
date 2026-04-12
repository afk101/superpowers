---
name: finishing-a-development-branch
description: Use when implementation is complete, all tests pass, and you need to decide how to integrate the work - guides completion of development work by presenting structured options for merge, PR, or cleanup
---

# 完成开发分支

## 概述

通过提供清晰的选项并处理选定的工作流程来指导完成开发工作。

**核心原则：** 验证测试 → 提供选项 → 执行选择 → 清理。

**开始时宣布：** "我正在使用 finishing-a-development-branch 技能来完成这项工作。"

## 流程

### 步骤 1：验证测试

**在提供选项之前，验证测试是否通过：**

```bash
# 运行项目的测试套件
npm test / cargo test / pytest / go test ./...
```

**如果测试失败：**
```
测试失败（<N> 个失败）。必须在完成之前修复：

[显示失败信息]

在测试通过之前无法继续进行 merge/PR。
```

停止。不要继续到步骤 2。

**如果测试通过：** 继续到步骤 2。

### 步骤 2：确定 Base Branch

```bash
# 尝试常见的 base branches
git merge-base HEAD main 2>/dev/null || git merge-base HEAD master 2>/dev/null
```

或者询问："此分支从 main 分支分离 - 这是否正确？"

### 步骤 3：提供选项

提供确切的 4 个选项：

```
实现已完成。您想要做什么？

1. 本地合并回 <base-branch>
2. 推送并创建 Pull Request
3. 保持分支原样（稍后我会处理）
4. 丢弃此工作

选择哪个选项？
```

**不要添加解释** - 保持选项简洁。

### 步骤 4：执行选择

#### 选项 1：本地合并

```bash
# 切换到 base branch
git checkout <base-branch>

# 拉取最新代码
git pull

# 合并 feature branch
git merge <feature-branch>

# 在合并结果上验证测试
<test command>

# 如果测试通过
git branch -d <feature-branch>
```

然后：清理 worktree（步骤 5）

#### 选项 2：推送并创建 PR

```bash
# 推送分支
git push -u origin <feature-branch>

# 创建 PR
gh pr create --title "<title>" --body "$(cat <<'EOF'
## 摘要
<2-3 个变更要点>

## 测试计划
- [ ] <验证步骤>
EOF
)"
```

然后：清理 worktree（步骤 5）

#### 选项 3：保持原样

报告："保持分支 <name>。Worktree 保留在 <path>。"

**不要清理 worktree。**

#### 选项 4：丢弃

**首先确认：**
```
这将永久删除：
- 分支 <name>
- 所有提交：<commit-list>
- <path> 处的 worktree

输入 'discard' 以确认。
```

等待精确确认。

如果确认：
```bash
git checkout <base-branch>
git branch -D <feature-branch>
```

然后：清理 worktree（步骤 5）

### 步骤 5：清理 Worktree

**对于选项 1、2、4：**

检查是否在 worktree 中：
```bash
git worktree list | grep $(git branch --show-current)
```

如果是：
```bash
git worktree remove <worktree-path>
```

**对于选项 3：** 保留 worktree。

## 快速参考

| 选项 | 合并 | 推送 | 保留 Worktree | 清理分支 |
|--------|-------|------|---------------|----------------|
| 1. 本地合并 | ✓ | - | - | ✓ |
| 2. 创建 PR | - | ✓ | ✓ | - |
| 3. 保持原样 | - | - | ✓ | - |
| 4. 丢弃 | - | - | - | ✓ (强制) |

## 常见错误

**跳过测试验证**
- **问题：** 合并损坏的代码，创建失败的 PR
- **修复：** 在提供选项之前始终验证测试

**开放式问题**
- **问题：** "我接下来该做什么？" → 模糊不清
- **修复：** 提供确切的 4 个结构化选项

**自动清理 worktree**
- **问题：** 在可能需要时删除 worktree（选项 2、3）
- **修复：** 仅对选项 1 和 4 进行清理

**丢弃时无确认**
- **问题：** 意外删除工作
- **修复：** 要求输入 "discard" 确认

## 危险信号

**绝不：**
- 在测试失败时继续
- 不在结果上验证测试就合并
- 不经确认删除工作
- 未经明确请求强制推送

**始终：**
- 在提供选项之前验证测试
- 提供确切的 4 个选项
- 对选项 4 获取输入确认
- 仅对选项 1 和 4 清理 worktree

## 集成

**调用者：**
- **subagent-driven-development**（步骤 7）- 所有任务完成后
- **executing-plans**（步骤 5）- 所有批次完成后

**配对：**
- **using-git-worktrees** - 清理由该技能创建的 worktree
