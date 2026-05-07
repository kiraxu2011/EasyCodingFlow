# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

EasyCodingFlow is a Claude Code skill for orchestrating multi-agent development workflows. It integrates Superpowers skills, OpenSpec, and Compound Engineering into a unified four-layer architecture plus verification layer.

## Architecture

```
Layer 0: Orchestration (Intent → Route → Team → Monitor)
Layer 1: Contract (OpenSpec / Brainstorming)
Layer 2: Execution (Writing-Plans → Executing-Plans → BDD/Debugging)
Layer 2.5: Verification (Consistency-Verification)
Layer 3: Knowledge (Compound Engineering)
```

**Execution order**: Layers must execute sequentially. Bug fixes can skip Layer 1 (Contract).

## Skill Structure

```
skills/
├── ecf/
│   ├── SKILL.md                      # Main skill entry point (~292 lines, optimized)
│   ├── ecf_config.yaml       # Configuration
│   ├── commands/                     # Migration reference docs (DEPRECATED)
│   │   ├── init.md                   # Points to ecf-init
│   │   ├── execute.md                # Points to ecf-execute
│   │   └ consistency-verification.md # Points to ecf-consistency-verification
│   └── references/
│       ├── dependency-check.md       # Unified dependency detection (NEW)
│       ├── intent-keywords.md        # Keyword mappings for intent recognition
│       ├── workflow-templates.md     # Workflow templates by scenario
│       ├── converter/SKILL.md        # OpenSpec→Superpowers artifact conversion
│       └ consistency-verification/   # Verification analyzer templates
│       └── ...                       # Other reference docs
├── ecf-init/SKILL.md         # Standalone: Initialize project (~261 lines, optimized)
├── ecf-execute/SKILL.md      # Standalone: Execute plans (~177 lines)
└── ecf-consistency-verification/SKILL.md  # Standalone: Verify alignment (~179 lines)
```

**Note**: Commands in `commands/` directory are DEPRECATED migration reference docs. Use standalone skills instead.

**Optimization (2026-05-04)**: Skills refactored for reduced redundancy. Environment detection unified to `dependency-check.md`, Pre-flight Check simplified with quick scripts + reference links.

## Workflow Templates

| Scenario | Workflow |
|----------|----------|
| New feature | OpenSpec → Brainstorming → Writing-Plans → Executing-Plans → Verification → **Archive** → Compound |
| Incremental | OpenSpec → Executing-Plans → Verification → **Archive** → Compound |
| Skills development | OpenSpec → **Writing-Skills** → Skill-Quality-Verification → **Archive** → Compound |
| Bug fix | Systematic-Debugging → Fix → Verification → Compound (skips Contract) |
| Refactor | Brainstorming → Writing-Plans → Executing-Plans → Verification → Compound |
| Code review | Requesting-Review → Receiving-Review → Compound |
| Test coverage | BDD → Verification → Compound |

**Archive step**: 使用 OpenSpec 创建变更的工作流，必须在 Verification 之后调用 `/opsx:archive` 完成变更生命周期闭环 (propose → apply → archive)。

**Skills development workflow** differs from new feature:
- Execution layer uses `writing-skills` (TDD flow), NOT `superpowers:writing-plans`
- Verification uses `skill-quality-verification`, NOT `consistency-verification`

## Key Skills

| Skill | Invocation | Function |
|-------|------------|----------|
| ecf-init | `/ecf-init` or `Skill("ecf-init")` | Initialize project structure |
| ecf-execute | `/ecf-execute` or `Skill("ecf-execute")` | Execute plans with concurrency |
| ecf-consistency-verification | `/ecf-consistency-verification` or `Skill("ecf-consistency-verification")` | Verify spec↔design↔code↔test alignment |

**Note**: Old command format `/ecf:init`, `/ecf:execute` etc. are deprecated. Use standalone skills instead.

## Testing Methodology

This project uses **TDD-for-skills**: test skill behavior by running scenarios before and after skill changes, recording results in docs/plans/*-design/tests/.

Test result files:
- `baseline-scenarios.md` - Test scenarios
- `baseline-results.md` - Results before skill changes
- `refactor-results.md` - Results after skill changes

**Skill Optimization**: When optimizing existing skills, use skill-creator workflow:
1. Analyze current skill structure and identify redundancy
2. Refactor to reduce line count (<500 lines target)
3. Unify duplicate content to reference files
4. Enhance descriptions for better triggering
5. Create evals.json for key scenarios

## Configuration

Two-level configuration:
- Global: `~/.claude/skills/ecf/ecf_config.yaml`
- Project: `.claude/ecf_config.yaml` (overrides global)

Key settings: model selection (haiku/sonnet/opus by scenario), max_parallel_agents, knowledge_base path.

## Knowledge Base

`docs/solutions/` — documented solutions to past problems (bugs, workflow patterns, best practices), organized by category with YAML frontmatter (`module`, `tags`, `problem_type`). Relevant when implementing or debugging in documented areas.

**Key documents**:
- `workflow-issues/skills-development-workflow-enforcement-2026-05-04.md` - Skills开发工作流强制执行规范

## Directory Structure

```
docs/
├── solutions/           # Knowledge base (required)
├── architectures/       # Architecture docs (ecf specific)
│   ├── architecture.md  # System architecture, tech stack, design decisions
│   ├── modules.md       # Module breakdown, responsibilities, interfaces
│   └── changes-index.md # Change history index
└── plans/               # Work plans (YYYY-MM-DD-*-design/)

openspec/                # OpenSpec CLI managed (NOT created by init)
├── changes/<name>/      # Change proposals
├── specs/<capability>/  # Specifications
└── changes/archive/     # Archived changes

.claude/
├── ecf_config.yaml
└── skills/              # Project-level skill overrides
```

**Important**: 
- `openspec/` is managed by OpenSpec CLI, not by ecf-init
- `docs/architectures/` is ecf specific, updated by Consistency-Verification layer

## Design Documents

Design docs in `docs/plans/YYYY-MM-DD-*-design/` follow a consistent structure:
- `_index.md` - Overview
- `architecture.md` - Component design
- `bdd-specs.md` - BDD specifications
- `best-practices.md` - Implementation guidelines
- `tests/` - Test scenarios and results

## Working with Skills

- **Read skill files directly** - Use Read tool, NOT the Skill tool when editing/modifying skills
- **Skill tool for invocation** - Use Skill tool to test/invoke skills in Claude sessions
- **Follow frontmatter** - Each skill file has YAML frontmatter with name, description, and constraints

## Red Flags (Avoid These)

- Direct coding without intent recognition first
- Skipping brainstorming for new features
- Bug fixes without systematic-debugging skill
- **Using OpenSpec but missing /opsx:archive step**
- **Workflow complete but OpenSpec change still active**
- Missing verification report (📊 summary) at end

## Red Flags - Skills开发 (CRITICAL)

**技能修改任务必须遵循标准流程:**

- **禁止直接编辑 SKILL.md** - 必须通过 `writing-skills` TDD 流程
- **禁止跳过 baseline scenario** - 必须验证需求必要性
- **禁止跳过 skill-quality-verification** - 必须验证 frontmatter 和 CSO
- **禁止跳过 archive** - OpenSpec 变更必须闭环
- **禁止跳过 compound** - 知识沉淀必须完成

**标准流程**: `/opsx:propose → writing-skills → skill-quality-verification → /opsx:archive → ce:compound`

**触发关键词**: skill、技能、SKILL.md、优化技能、技能增强、技能修改

**详见**: `docs/solutions/workflow-issues/skills-development-workflow-enforcement-2026-05-04.md`

## Important
Default to reply in Chinese unless I explicitly specify the language
