---
name: ecf
description: Use when user requests require multi-skill orchestration or multi-agent coordination. Triggers for feature development, bug fixes, refactoring, code review, documentation, test coverage, or ambiguous requests needing intent classification. Make sure to use this skill whenever the user mentions developing new features, fixing bugs, refactoring code, reviewing code, updating documentation, adding tests, or coordinating multiple agents, even if they don't explicitly ask for 'ecf.'
---

# EasyCodingFlow 编排

## Overview

统一的 Agent 协作编排 skill，整合 Superpowers、OpenSpec、Compound Engineering。

**Core principle**: 每个请求必须先经过意图识别，再路由到工作流。

## When to Use

Use when:
- User requests need multi-skill orchestration across multiple layers (Contract → Execution → Verification → Knowledge)
- Tasks involving feature development, bug fixes, refactoring, code review, documentation, test coverage
- Ambiguous requests needing automatic intent classification and workflow routing
- Skill development tasks requiring full standard workflow enforcement

**Do NOT use** for: Direct implementation of small, single-step tasks that don't need orchestration.

## Entry Point

User provided arguments: `$ARGUMENTS`

### Pre-flight Check (REQUIRED FIRST)

#### Step 1: Empty Arguments Check

If `$ARGUMENTS` is empty or whitespace-only:
```
🔍 Agent-Teams 帮手
请提供具体任务，例如：
• bug fix [问题描述] - 修复某个问题
• 开发新功能 [功能描述] - 实现某个功能
• review [范围] - 代码审查
• 重构 [模块] - 代码重构
• 文档更新 - 更新文档
• 测试补齐 - 补充测试用例
输入 /ecf-init 可初始化项目结构。
```
**STOP** - wait for user input. Do NOT proceed.

#### Step 2: Initialization Check

Check: `docs/solutions/` (knowledge base) and `.claude/ecf_config.yaml` (project config) exist.
- Missing → prompt user to run `/ecf-init`, or create `.claude/.ecf-degraded.flag` for degraded mode
- Degraded mode: knowledge retrieval disabled, OpenSpec storage disabled, intent recognition continues

#### Step 3: Dependency Environment Check

Detect 4 dependencies (commands from `ecf-init`, details in `references/dependency-check.md`):

| 依赖 | 检测位置 | 影响层级 |
|------|----------|----------|
| OpenSpec CLI + Skills | `.claude/skills/openspec-*` | Contract Layer |
| Compound Engineering | `~/.claude/plugins/cache/compound-engineering-plugin/` | Knowledge Layer |
| Superpowers@frad-dotclaude | `CLAUDE_PLUGIN_ROOT` + `setup-superpower-loop.sh` | Execution Layer |
| skill-creator | `~/.claude/skills/skill-creator/` | Skills Development |

缺失依赖时引导用户运行 `ecf-init --auto-install`。Superpowers 完全缺失为 **Critical**，需阻断流程。

#### Step 4: Process Execution Rules (MANDATORY)

**Core Rule**: After Pre-flight Check completes, ecf MUST continue executing the full standard workflow (Intent Recognition → Routing → Contract → Execution → Verification → Knowledge) in the current agent. DO NOT stop after just outputting SKILL.md content and return control to caller early.

**Exception 1: Trivial Small Task Skip**

ecf MAY skip full workflow and execute directly ONLY when ALL of the following criteria are met:
1. Modifies exactly **one file**
2. Total changes (add + modify + delete) **fewer than 50 lines**
3. Does **not** change core algorithm or routing logic
4. Only adds rules, corrects documentation, or supplements test/eval cases

If the above criteria are met and ecf estimates skipping full workflow is more efficient:
1. **Explain why**: State the specific reasons and how criteria are met
2. **Ask user confirmation**: Explicitly prompt "Do you confirm skipping full workflow and executing directly?"
3. **Wait for answer**: Only proceed directly after user explicitly confirms "yes" / "confirm"

If user answers "no" or "follow standard process", ecf MUST continue full standard workflow from current point.

**Exception 2: User Interaction Points**

When you reach a step that requires user input (asking workflow mode selection, confirming trivial task skip permission, etc.), you MUST output the prompt exactly as required, **THEN STOP EXECUTION** and return control to the user. Wait for the user to answer before proceeding.

#### Step 5: Proceed to Intent Recognition

Only after Steps 1-4 pass. Intent recognition runs in the current agent (no external API calls).

### Intent Recognition Output Template

```
意图识别
任务分析: <task description>
  - 关键词: <keywords>
  - 场景类型: <type> (<description>)
  - 标准工作流: <workflow>

路由决策: <reason>
  → 契约层入口: <entry>
```

#### 关键词匹配优先级规则

- **最高优先级**: 包含 ANY skill 关键词 → 强制匹配 `skill_development`
  - 关键词: `skill`, `skills`, `技能`, `SKILL.md`, `优化技能`, `技能优化`, `技能开发`, `技能修复`, `技能bug`, `技能评估`, `skill-creator`, `skill-quality-verification`
- 仅含"优化"无 skill 关键词 → 按上下文匹配 incremental/refactor

#### 契约层入口路由决策表

| 场景类型 | 契约层入口 | 说明 |
|----------|------------|------|
| new_feature | `/opsx:propose` | 需要 OpenSpec 变更管理 |
| skill_development | `/opsx:propose` | 需要 OpenSpec 变更管理 |
| incremental | `/opsx:propose` | 需要 OpenSpec 变更管理 |
| refactor | `Skill("agent-team-orchestrator", {mode: "brainstorming"})` | 不需要 OpenSpec；orchestrator 内部包装 brainstorming |
| bug_fix | **跳过契约层** | 直接执行层 |
| code_review | **跳过契约层** | 直接执行层 |
| test_coverage | **跳过契约层** | 直接执行层 |
| documentation | **跳过契约层** | 直接执行 |

完整关键词映射见 `references/intent-keywords.md`，意图识别流程见 `references/intent-recognition.md`。

#### Step 6: Ask Workflow Mode (MANDATORY - CANNOT SKIP)

**AFTER intent recognition completes, BEFORE proceeding to contract layer:**

You **MUST** output the following prompt exactly as written to ask the user for workflow mode selection:

```
🔀 请选择工作流流转模式:
1. 手动确认模式（默认）- 每个阶段完成后展示摘要并等待您确认
2. 自动流转模式 - 阶段完成后自动进入下一跳
```

**DO NOT** proceed to the next step (invoking contract layer skill) until the user selects a mode.

This step is **enforced by the process** - any execution that skips this is a process violation.

After outputting the prompt, you **MUST STOP** execution here and return control to the caller. Do NOT proceed to contract layer until the user selects a mode.

---

### 阶段完成验证

每个步骤完成后必须验证产物再进入下一阶段。统一验证规则和检查清单见 `references/phase-completion-validation.md`。

---

## 标准工作流

| 场景 | 工作流（完整定义见 `references/workflow-templates.md`） |
|------|--------------------------------------------------------|
| new_feature | `/opsx:propose` → **agent-team-orchestrator**(brainstorming) → writing-plans → **ecf-execute** → **ecf-verify** → **/opsx:archive** → ce:compound |
| bug_fix | systematic-debugging → fix → **ecf-verify** → ce:compound |
| refactor | **agent-team-orchestrator**(brainstorming) → writing-plans → **ecf-execute** → **ecf-verify** → ce:compound |
| code_review | **agent-team-orchestrator**(code_review) → ce:compound |
| skill_development | `/opsx:propose` → **skill-creator** → **skill-quality-verification** → **/opsx:archive** → ce:compound |
| incremental | `/opsx:propose` → **ecf-execute** → **ecf-verify** → **/opsx:archive** → ce:compound |
| test_coverage | BDD → **ecf-verify** → ce:compound |
| documentation | Direct execution |

> **agent-team-orchestrator**: brainstorming 和 code_review 阶段的多模型编排包装层。启用时自动将单一 agent 执行升级为 N 个 agent 独立分析→交叉校准→共识收敛。可通过配置禁用降级为直接调用开源技能。详见 `references/agent-team-orchestrator-integration.md`。

**⚠️ ce:compound 不可跳过**: 所有工作流最后一步必须调用。

详见 `references/workflow-completion-checklist.md`。

## After Contract Layer Complete (CRITICAL)

**禁止使用 `/opsx:apply`**。OpenSpec 返回的通用提示必须按场景路由。

| 意图识别结果 | 执行层入口 | 禁止 |
|--------------|------------|------|
| skill_development | `Skill("skill-creator")` | `/opsx:apply` |
| new_feature | `/ecf-execute` | `/opsx:apply` |
| refactor | `/ecf-execute` | `/opsx:apply` |
| bug_fix | `Skill("systematic-debugging")` | `/opsx:apply` |
| code_review | `Skill("agent-team-orchestrator", {mode: "code_review"})` | `/opsx:apply` |

OpenSpec 产物需转换，详见 `references/converter/SKILL.md`。

---

## 工作流自动流转机制

意图识别完成后、进入契约层前，询问用户选择模式：

```
🔀 请选择工作流流转模式:
1. 手动确认模式（默认）- 每个阶段完成后展示摘要并等待您确认
2. 自动流转模式 - 阶段完成后自动进入下一跳
```

各场景完成信号和流转规则定义在 `references/workflow-templates.md` 的"场景流转规则表"章节。

**Writing-plans 输出自动纠正**: 如果输出引导至 `/superpowers:executing-plans`，自动纠正为 `/ecf-execute`。

---

## 异常处理与知识检索

工作流执行异常时（Task FAIL、Skill 调用异常、验证不通过），优先检索 `docs/solutions/` 知识库。完整流程见 `references/knowledge-retrieval.md` 的"自动异常场景检索"章节。

### 快速处理规则

1. degraded 模式 → 跳过检索，直接报告异常
2. Top-1 tags 匹配 ≥1 → 自动应用并验证
3. 多个匹配 → 展示 Top-3 供用户选择
4. 无匹配 → 展示错误上下文，用户确认处理方式

---

## 并行执行

详见 `references/parallel-execution.md`。

| 任务数量 | 执行模式 | Agent 通信 |
|----------|----------|------------|
| >6 独立任务 | Agent Team | ✅ 可互发消息 |
| ≤6 独立任务 | Subagent | ❌ 仅返回 |
| 1 任务 | Linear | ❌ 无 |

**执行入口**: `/ecf-execute`（强制并发）。

## Team Building

| Complexity | Agents | Model |
|------------|--------|-------|
| simple | 1 | haiku |
| medium | 2-3 | sonnet |
| complex | 4-6 | mixed |

## 知识沉淀层

每个工作流完成后必须调用知识沉淀。检查 CE 插件存在性，存在则调用 `Skill("ce:compound")`，否则使用简化流程直接写入 `docs/solutions/`。详见 `references/knowledge-writing.md`。

## Red Flags - STOP

- 直接编码未意图识别
- **意图识别完成后未询问流转模式就进入契约层**（process violation，必须停止并询问）
- 新需求跳过 brainstorming
- Bug修复不用 systematic-debugging
- 工作流完成但未调用 ce:compound
- 契约层完成后调用 `/opsx:apply`
- 使用 `superpowers:executing-plans` 而非 `/ecf-execute`
- Critical 依赖缺失未阻断流程
- 工作流异常时未进行知识检索直接报错
- **degraded 模式下仍尝试知识检索**
- 琐碎小任务未用户确认就跳过完整流程直接执行

## References

- `references/workflow-templates.md` - 工作流模板（权威源）
- `references/intent-keywords.md` - 关键词映射
- `references/intent-recognition.md` - 意图识别流程（含输出格式规范）
- `references/phase-completion-validation.md` - 阶段完成验证
- `references/workflow-completion-checklist.md` - 工作流完成检查清单
- `references/parallel-execution.md` - 并发执行策略（含 Agent 通信格式）
- `references/dependency-check.md` - 依赖检查完整参考
- `references/knowledge-retrieval.md` - 知识检索流程（含异常处理、任务失败处理）
- `references/knowledge-writing.md` - 知识沉淀流程
- `references/converter/SKILL.md` - OpenSpec→Superpowers 产物转换
- `references/config-schema.md` - 配置 schema

## Skills

| 技能 | 调用方式 | 功能 |
|------|----------|------|
| ecf-init | `/ecf-init` | 初始化项目结构 |
| ecf-execute | `/ecf-execute` | 并发执行计划 |
| ecf-verify | `/ecf-verify` | 一致性验证 |
| agent-team-orchestrator | `Skill("agent-team-orchestrator")` | 多模型 agent 编排（brainstorming/code_review） |

> **迁移说明**: 旧 `/ecf:init`, `/ecf:execute`, `/ecf:consistency-verification` 已迁移为独立技能 `ecf-init`, `ecf-execute`, `ecf-verify`。
