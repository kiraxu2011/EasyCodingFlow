---
name: ecf-execute
description: Use when ready to execute implementation plans with parallel agent teams, or after writing-plans generates batch tasks, or when user requests concurrent execution of planned work. Make sure to use this skill whenever the user mentions executing plans, running tasks concurrently, parallel execution, agent teams, or when writing-plans completes, even if they don't explicitly ask for 'ecf-execute.'
---

# Execute Plans with EasyCodingFlow

并发执行实现计划，使用 EasyCodingFlow 提升执行效率。

## Overview

此技能是 `superpowers:executing-plans` 的增强版本，强制启用并发执行能力。

**关键差异**:
- 原生 executing-plans: 可能使用 Linear 模式
- ecf-execute: 强制使用 Agent Team 并发

## When to Use

- After writing-plans generates implementation plan
- User requests `/ecf-execute`
- Ready to implement with parallel execution
- Need to execute multiple independent tasks concurrently

## Arguments

- `[plan-folder-path]`: Path to plan folder containing `_index.md` and task files
- If not provided: searches `docs/plans/` for latest `*-plan/` directory

## Workflow

```
Check environment → Load skills → Resolve plan → Analyze tasks → Choose mode → Execute → Verify → Complete
```

## Execution Flow

**Step 1: Environment Check**

**必须先检查环境**，否则并发功能会卡住。

```bash
export CLAUDE_PLUGIN_ROOT="${CLAUDE_PLUGIN_ROOT:-$(ls -d ~/.claude/plugins/marketplaces/frad-dotclaude/superpowers 2>/dev/null || ls -d ~/.claude/plugins/cache/frad-dotclaude/superpowers/*/ 2>/dev/null | head -1 || echo "")}"

if [[ -z "$CLAUDE_PLUGIN_ROOT" ]]; then
    echo "⚠️ 环境未就绪，使用官方版本 fallback"
    # 官方版本不支持 Agent Team，将使用 subagent
fi
```

**Step 2: Load Required Skills**

**并发执行必须加载以下 Skills**:

1. `superpowers:agent-team-driven-development` - 提供团队协调能力
2. `superpowers:behavior-driven-development` - 提供测试驱动能力

调用方式: `Skill("superpowers:agent-team-driven-development")`

**Step 3: Resolve Plan Path**

```bash
# If argument provided
plan_path="$ARGUMENTS"

# If no argument, find latest
plan_path=$(ls -td docs/plans/*-plan/ 2>/dev/null | head -1)

# Verify path exists
[ -d "$plan_path" ] || echo "⚠️ Plan folder not found: $plan_path"
```

**Step 4: Analyze Batch Tasks**

读取 `_index.md` 中的 YAML 任务元数据：

```yaml
tasks:
  - id: "001"
    subject: "Setup project"
    type: "setup"
    depends-on: []
  - id: "002"
    subject: "Auth test"
    type: "test"
    depends-on: ["001"]
  - id: "003"
    subject: "Auth impl"
    type: "impl"
    depends-on: ["002"]
```

**分析结果**:
- 任务数量: N
- Red-Green 配对: 识别 test+impl 配对
- 文件冲突: 检测同文件编辑

**Step 5: Choose Execution Mode**

| 条件 | 模式 | Agent 数 |
|------|------|----------|
| Red-Green 配对 | `pair` | 2/配对 |
| >6 独立任务 | `team` | 4-6 |
| ≤6 独立任务 | `subagent` | N(并行) |
| 1 任务/强依赖 | `linear` | 1 |

**设计原理**: 小规模任务集（≤6）使用 subagent 更高效，避免 Agent Team 的协调开销；大规模任务集（>6）需要 Agent Team 的消息通信能力来协调复杂依赖关系。

**Step 6: Execute with Concurrency**

### Agent Team Mode

调用 `superpowers:agent-team-driven-development`：

配置团队成员:
- Implementer 1: 负责 task-001, task-002
- Implementer 2: 负责 task-003, task-004
- Reviewer: 验证所有任务

### Subagent Mode

并行调用 Agent tool:

```
Agent({ description: "执行任务1", prompt: "...", subagent_type: "general-purpose" })
Agent({ description: "执行任务2", prompt: "...", subagent_type: "general-purpose" })
```

### Red-Green Pair Mode

顺序执行配对，多配对并行:
- Pair 1: task-002-test → task-002-impl
- Pair 2: task-003-test → task-003-impl
- Pair 1 和 Pair 2 可并行启动

**Step 7: Verification Gate**

每批次完成后强制验证:

```
📋 批次验证
━━━━━━━━━━━━━━━
Task-001: ✅ PASS
Task-002: ❌ FAIL (重试 1/2)
━━━━━━━━━━━━━━━
```

**规则**:
- 所有任务 PASS 才进入下一批次
- FAIL 任务保持在 `in_progress`
- 最多 2 次重试，失败后升级

**异常处理 - 知识检索集成**:

当任务 FAIL 时，在重试前先执行知识检索：

```bash
# 检查 degraded 模式
if [ -f "../.claude/.ecf-degraded.flag" ] || [ -f ".claude/.ecf-degraded.flag" ]; then
    echo "⚠️ 知识检索不可用，直接重试"
else
    # 提取失败上下文关键词
    error_keywords=$(echo "$task_error" | grep -oP '(error|fail|exception|missing|not found|permission denied)' | head -3)
    
    # 检索 docs/solutions/ 匹配历史方案
    for kw in $error_keywords; do
        results=$(grep -ril "$kw" docs/solutions/ 2>/dev/null)
        [ -n "$results" ] && break
    done
    
    if [ -n "$results" ]; then
        echo "📚 知识库匹配到历史方案，优先应用修复"
        # 应用 top-1 方案修复后重试
    fi
fi
```

**知识检索集成行为**:
| 情况 | 行为 |
|------|------|
| degraded 模式 | 跳过检索，直接重试 |
| 匹配到方案 | 应用历史方案修复后重试 |
| 无匹配方案 | 使用默认重试逻辑，失败后升级 |

**重试升级**:
- 2 次重试均 FAIL → 执行升级流程
- 升级时展示错误上下文和检索结果（如有）
- 向编排层报告失败详情

详细检索流程见 [knowledge-retrieval.md](../ecf/references/knowledge-retrieval.md) 的"自动异常场景检索"章节。

## Execution Completion Validation

After all batches complete, run final completion validation:

```bash
🔍 执行完成验证
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ 环境检查已完成 ✅
✓ 计划路径存在且有效 ✅
✓ 任务分析已完成 ✅
✓ 执行模式已确定 ✅
✓ 所有批次任务执行完成 ✅
✓ 批次验证门控已通过 ✅
✓ 执行摘要已输出 ✅
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ 验证通过，执行层完成
```

**If any check fails**:
- Interrupt workflow
- Report which check failed
- Do not proceed to verification layer

## Summary Output

```
📊 Execution Summary
━━━━━━━━━━━━━━━━━━━━
计划路径: [path]
任务总数: [N]
执行模式: [team/subagent/pair/linear]
完成状态: [count] ✅ / [count] ❌
━━━━━━━━━━━━━━━━━━━━
[✅ 全部完成 / ⚠️ 部分失败]
```

## Red Flags - STOP

- 跳过环境检查直接调用 skill
- >6 任务仍使用 linear 或 subagent 模式（应使用 team）
- ≤6 独立任务使用 team 模式（应使用 subagent）
- 未加载 agent-team-driven-development skill（当需要 team 模式时）
- 未执行批次验证门控

## References

- [phase-completion-validation.md](../ecf/references/phase-completion-validation.md) - 阶段完成验证统一规则
- [parallel-execution.md](../ecf/references/parallel-execution.md) - 并发执行策略
- [dag-scheduling.md](../ecf/references/dag-scheduling.md) - DAG 任务调度
- [superpowers:executing-plans](https://github.com/frad-dotclaude/superpowers) - 原生执行 skill