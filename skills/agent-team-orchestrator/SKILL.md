---
name: agent-team-orchestrator
description: Use when the user requests brainstorming, code review, design discussion, architecture review, or any multi-perspective analysis. Also use when higher quality analysis through diverse model perspectives would be valuable, or when the user mentions wanting independent analysis or multiple opinions on code, design, or architecture decisions — even if they don't explicitly ask for agent teams. Automatically handles configuration loading, state machine flow control, and automatic file output for all phases.
---

# Agent Team Orchestrator

## Overview

用多模型 agent 团队的认知多样性提升分析质量。核心理念：N 个独立大脑各自思考 → 交叉挑战 → 收敛共识。

**核心改进**: 配置加载、流程控制、文件输出完全自动化，skill 自身闭环，不依赖主LLM记忆。每个阶段完成立即写盘，不会遗漏。

## When to Use

- **brainstorming 模式**: new_feature、refactor 场景的头脑风暴阶段
- **code_review 模式**: 代码审查阶段，需要多视角、明确立场的审查

**调用方式**:
- Slash command: `/agent-team-orchestrator [--key=value ...] <task description>`
- Skill invocation: `Skill("agent-team-orchestrator", {mode: "brainstorming"|"code_review", task: "<任务描述>"})`

Command-line arguments (optional) override configuration:
```
/agent-team-orchestrator --consensus_strategy=decentralized_debate --cross_calibration_rounds=2 我的任务描述
```

## Execution Model: State Machine

This skill uses an internal state machine to automatically progress through phases. You (the caller) only need to reinvoke the skill after each batch of agent completions — the skill decides what to do next based on the current state.

### States

| State | Description | Next Step |
|-------|-------------|-----------|
| `INIT` | Starting fresh | Load config → validate → confirm (if needed) → create output dir → launch Phase 1 |
| `PHASE1_DONE` | Phase 1 completed | Write output → launch Phase 2 rounds |
| `PHASE2_DONE` | Phase 2 all rounds completed | Write output → proceed to Phase 3 based on consensus_strategy |
| `DEBATE_ROUND_DONE` | One debate round completed | Write output → check completion → next round or finish → generate final report |
| `DONE` | All phases completed | Output summary with file paths → done |

## Step 1: Automatic Configuration Loading

Configuration is loaded automatically by the skill in this priority order (lowest to highest):

1. **Built-in defaults** (see below)
2. **Global config**: `~/.claude/skills/ecf/ecf_config.yaml`
3. **Project config**: `.claude/ecf_config.yaml`
4. **Environment variables**: `ATO_*` prefix
5. **Command-line arguments**: `--key=value` from invocation line (highest priority)

### Default Configuration

```python
default_config = {
    "enabled": True,
    "models": ["opus", "sonnet", "haiku"],
    "cross_calibration_rounds": 1,
    "cross_calibration_strategy": "distributed",
    "consensus_strategy": "centralized",
    "consensus_max_rounds": 2,
    "consensus_early_stop": True,
    "enable_file_output": True,
    "output_dir": ".agent-team-output",
}
```

### Configuration Schema

See `references/config-schema.md` for complete documentation of all configuration options.

### Environment Variable Mapping

| Environment Variable | Config Key | Example |
|----------------------|------------|---------|
| `ATO_ENABLED` | `enabled` | `ATO_ENABLED=false` |
| `ATO_MODELS` | `models` | `ATO_MODELS=opus,sonnet` |
| `ATO_CROSS_CALIBRATION_ROUNDS` | `cross_calibration_rounds` | `ATO_CROSS_CALIBRATION_ROUNDS=2` |
| `ATO_CROSS_CALIBRATION_STRATEGY` | `cross_calibration_strategy` | `ATO_CROSS_CALIBRATION_STRATEGY=centralized` |
| `ATO_CONSENSUS_STRATEGY` | `consensus_strategy` | `ATO_CONSENSUS_STRATEGY=decentralized_debate` |
| `ATO_CONSENSUS_MAX_ROUNDS` | `consensus_max_rounds` | `ATO_CONSENSUS_MAX_ROUNDS=3` |
| `ATO_CONSENSUS_EARLY_STOP` | `consensus_early_stop` | `ATO_CONSENSUS_EARLY_STOP=false` |
| `ATO_ENABLE_FILE_OUTPUT` | `enable_file_output` | `ATO_ENABLE_FILE_OUTPUT=false` |
| `ATO_OUTPUT_DIR` | `output_dir` | `ATO_OUTPUT_DIR=/tmp/agent-output` |

### Command-line Argument Parsing

Any `--key=value` argument at the start of the invocation overrides configuration. Example:

```
/agent-team-orchestrator --consensus_strategy=decentralized_debate --cross_calibration_rounds=2 帮我设计一个高并发缓存架构
```

Supported arguments: all config keys above use the same name (snake_case). Boolean values: `true`/`false`.

## Step 2: Configuration Validation

After loading, the skill automatically validates configuration:

| Check | Rule |
|-------|------|
| `models` | All models must be one of `opus`, `sonnet`, `haiku` |
| `cross_calibration_rounds` | Must be integer between 1 and 3 inclusive |
| `consensus_max_rounds` | Must be integer between 1 and 5 inclusive |
| `output_dir` | Directory must be creatable and writable |

If validation fails, the skill reports the error immediately and stops.

## Step 3: User Confirmation for High-Cost Configurations

Before proceeding, the skill requests explicit confirmation if:

1. **`consensus_strategy = decentralized_debate`**: Calculates expected cost `N × R` (N = number of agents, R = max rounds) and prompts:
   ```
   配置指定使用 decentralized_debate 策略，预期需要 {N×R} 次额外 agent 调用，是否继续？
   ```

2. **`cross_calibration_rounds > 1`**: Prompts:
   ```
   配置指定交叉校准轮次 = {N}，是否继续？
   ```

Wait for user confirmation before proceeding.

## Step 4: Run Setup

After confirmation passes:

1. Generate unique `run_id`: `ato-YYYYMMDD-HHMMSS` (timestamp-based, collision-free)
2. Create output directory: `{config.output_dir}/{run_id}/`
3. Store state: all phase outputs accumulated in context for subsequent phases

Output directory isolation: each run gets its own directory, no overwrites.

---

## Phase 1: Independent Thinking

**Goal**: N agents think independently in isolation, no anchoring bias from seeing others' conclusions first.

### Execution

1. All N agents are launched in parallel in a single message using the Agent tool
2. Each agent uses its assigned model
3. Each agent gets the prompt template from `references/{mode}-prompt.md`
4. Each agent works in complete isolation (doesn't know other agents exist)

### Prompt Requirements (from template):

**Brainstorming mode**:
- Full task description + context
- Independence declaration: "你正在独立完成分析。当前阶段禁止与其他 agent 交流。请基于你自己的判断形成完整结论。"
- Structured output requirement (understanding → multiple schemes → comparison → recommendation → self-questioning)

**Code Review mode**:
- Scope description (files/PR to review)
- Strict stance labeling requirement: every finding MUST be MUST_FIX / SHOULD_FIX / OK (no ambiguity)
- Prohibits vague phrases like "maybe consider", "optional", etc.

### Completion

After all agents complete:
- Collect all outputs with agent identity and model
- If `enable_file_output: true`, **automatically write** to:
  ```
  {output_dir}/{run_id}/phase-1-independent-thinking.md
  ```
- Transition to state `PHASE1_DONE`
- Next: skill automatically proceeds to Phase 2

---

## Phase 2: Cross-Calibration

**Goal**: Each agent reads everyone else's work, challenges weak arguments, finds blind spots.

### Execution

Loop for `round` from 1 to `cross_calibration_rounds`:

1. Distribute all previous phase outputs to each agent
2. Launch each agent in parallel (retains original model from Phase 1)
3. Each agent challenges disagreements and identifies blind spots
4. Collect all cross-review results
5. If `enable_file_output: true`, **automatically write** to:
   ```
   {output_dir}/{run_id}/phase-2-cross-calibration-round-{round}.md
   ```

After all rounds complete:
- Transition to state `PHASE2_DONE`
- Next: skill automatically proceeds to Phase 3 based on `consensus_strategy`

---

## Phase 3: Consensus Convergence

Two strategies supported:

### Strategy A: Centralized Convergence (default)

**When**: `consensus_strategy: centralized`

**Execution**:
- Orchestrator (this skill) synthesizes all Phase 1 + Phase 2 outputs in the current context
- Applies convergence principles: majority agreement + argument quality weighting
- Generates final consensus report following standard structure
- If `enable_file_output: true`, **automatically write** to:
  ```
  {output_dir}/{run_id}/phase-3-consensus-report.md
  ```
- Transition to state `DONE`

### Strategy B: Decentralized Debate Collective Convergence

**When**: `consensus_strategy: decentralized_debate`

**Execution**:

For `round` from 1 to `consensus_max_rounds`:

1. Each agent gets full history: Phase 1 + Phase 2 + all previous debate rounds + open disputes
2. Launch all agents in parallel (each retains original identity/model from Phase 1)
3. Prompt from `references/decentralized-debate-prompt.md`: each agent updates positions on open disputes
4. Collect updated positions
5. If `enable_file_output: true`, **automatically write** to:
   ```
   {output_dir}/{run_id}/phase-3-debate-round-{round}.md
   ```
6. Check for completion:
   - If `consensus_early_stop: true` AND all disputes resolved → break early
   - Else → continue to next round

After loop exits:
- Generate final consensus report from all agents' final positions
  - Consensus points (all agree)
  - Remaining disputes (record each agent's final position)
  - New blind spots found during debate
  - Final recommendation
- If `enable_file_output: true`, **automatically write** to:
  ```
  {output_dir}/{run_id}/phase-3-consensus-report.md
  ```
- Transition to state `DONE`

---

## Final Completion (State DONE)

Output summary to user:
- Confirm all phases completed successfully
- List all output files created with full paths
- Summary of key findings/recommendations

---

## Final Report Structure

Both strategies produce the same report structure:

```markdown
# [Brainstorming / Code Review] 共识报告

## 参与 Agent
| Agent | 模型 | 角色 |
|-------|------|------|
| Agent A | opus | 独立分析 → 交叉校准 → 辩论 |
| ... | ... | ... |

## 共识点
<!-- 所有 agent 一致同意的结论 -->

## 分歧点
| 议题 | Agent A | Agent B | Agent C | 最终状态 |
|------|---------|---------|---------|----------|
| ... | ... | ... | ... | 已达成共识 / 仍有分歧 → 各方立场如上 |

## 盲区发现
<!-- 辩论过程中新发现的，所有先前阶段都遗漏的问题 -->

## 最终推荐

### Brainstorming 模式
- **推荐方案**: <综合多方视角后的最优方案>
- **方案对比矩阵**: 各方案在关键维度上的对比
- **保留的备选方案**: <虽然未选中但有价值保留的方案>
- **风险提示**: 综合所有 agent 视角的风险清单

### Code Review 模式
- **MUST_FIX 清单** (合并去重): <必须修复的发现列表>
- **SHOULD_FIX 清单** (合并去重): <建议修复的发现列表>
- **立场分歧专项**: <同一发现但不同 agent 给出不同立场的详细辩论>
- **审查覆盖盲区**: <可能被遗漏的审查维度>
```

---

## Mode-Specific Instructions

### Brainstorming Mode

Each agent in Phase 1 must produce:
1. 需求理解与约束分析
2. 至少 2 个完整方案
3. 方案对比矩阵
4. 推荐方案 + 风险分析
5. 自我质疑（你的方案什么时候会失败）

### Code Review Mode

Each agent in Phase 1 must produce:
1. 审查概览
2. MUST_FIX 发现（每个发现必须有位置、描述、理由、修复建议）
3. SHOULD_FIX 发现（同上格式）
4. OK 发现（那些可能被质疑但你认为可接受的）
5. 审查盲区自评

Orchestrator scans Phase 1 outputs for ambiguous phrasing. If any found, requires the agent to re-clarify stances before Phase 2.

---

## Degradation Rules

Skip orchestration and directly call the underlying skill if any:
- `enabled: false` in configuration
- Only one model configured in `models`
- User prompt contains "快速模式", "跳过编排", or "--no-orchestrator"

When skipped, orchestrator directly invokes:
- Brainstorming → `Skill("superpowers:brainstorming")`
- Code Review → `Skill("compound-engineering:ce-review")`

---

## References

- `references/brainstorming-prompt.md` — Phase 1 brainstorming prompt template
- `references/code-review-prompt.md` — Phase 1 code review prompt template
- `references/decentralized-debate-prompt.md` — Phase 3 decentralized debate prompt template
- `references/config-schema.md` — Complete configuration schema documentation

## Skills Referenced

- `superpowers:brainstorming` — Brainstorming scenario target skill (called internally by agents)
- `compound-engineering:ce-review` — Code review scenario target skill (called internally by agents)
