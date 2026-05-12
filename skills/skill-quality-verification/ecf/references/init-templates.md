# Init Templates - Agent-Teams 初始化模版

## 模版清单

| 模版 ID | 目标位置 | 用途 |
|---------|---------|------|
| solution-template | docs/solutions/.template.md | 解决方案归档格式 |
| solution-readme | docs/solutions/README.md | 知识库使用指南 |
| openspec-readme | docs/openspec/README.md | OpenSpec 目录指南 |
| openspec-proposal-template | docs/openspec/templates/proposal-template.md | Proposal 格式模版 |
| plans-readme | docs/plans/README.md | 计划目录指南 |
| project-config | .claude/ecf_config.yaml | 项目配置模版 |
| init-state | .claude/.ecf-init.local.md | 初始化状态文件 |
| architecture-template | docs/openspec/architectures/architecture.md | 系统整体架构 |
| modules-template | docs/openspec/architectures/modules.md | 功能模块划分 |
| changes-index-template | docs/openspec/architectures/changes-index.md | 变更历史索引 |

---

## Template: solution-template

**目标**: `docs/solutions/.template.md`

```markdown
# 解决方案: [标题]

## 问题背景
[描述遇到的问题]

## 解决方案
[详细的解决方案描述]

## 关键决策
- 决策1: ...
- 决策2: ...

## 经验总结
[可复用的经验点]

## 相关任务
- task-id: ...
- commit: ...

## 最后更新
YYYY-MM-DD
```

---

## Template: solution-readme

**目标**: `docs/solutions/README.md`

```markdown
# Knowledge Base

存储 ecf 的解决方案和经验总结。

## 使用方法

1. 创建新解决方案: YYYY-MM-DD-topic-solution.md
2. 参考 .template.md 了解格式

## 目录结构

- .template.md: 解决方案模版
- YYYY-MM-DD-*-solution.md: 已归档的解决方案
```

---

## Template: openspec-readme

**目标**: `docs/openspec/README.md`

```markdown
# OpenSpec 产物存储

存储 OpenSpec 工作流产物。

## 目录结构

- proposals/: /opsx:propose 生成的 proposal, spec, design 文件
- templates/: 可复用的 spec 模版
- changes/: 变更套件历史
```

---

## Template: openspec-proposal-template

**目标**: `docs/openspec/templates/proposal-template.md`

```markdown
# Proposal: [标题]

## Context
[背景信息]

## Requirements
[需求列表]

## Design
[设计方案]

## Tasks
[任务拆解]
```

---

## Template: plans-readme

**目标**: `docs/plans/README.md`

```markdown
# 工作计划目录

存储 ecf 生成的实施计划。

## 目录结构

- YYYY-MM-DD-*-plan.md: 实施计划文件
- YYYY-MM-DD-*-design/: 设计文档目录
```

---

## Template: project-config

**目标**: `.claude/ecf_config.yaml`

```yaml
# Agent-Teams 项目配置
# 继承全局配置: ~/.claude/skills/ecf/ecf_config.yaml

defaults:
  language: zh-CN
  knowledge_base: docs/solutions/
  max_parallel_agents: 5

# 项目级覆盖配置 (可选)
# models:
#   execution: sonnet

# custom_templates: []

# parameters:
#   timeout_per_task: 30m
```

---

## Template: init-state

**目标**: `.claude/.ecf-init.local.md`

```yaml
---
initialized_at: "YYYY-MM-DD HH:MM:SS"
version: "1.0"
directories_created:
  - docs/solutions/
  - docs/openspec/
  - docs/openspec/proposals/
  - docs/openspec/templates/
  - docs/openspec/changes/
  - docs/plans/
  - .claude/
templates_deployed: true
config_generated: true
---

# Initialization State

此文件记录 ecf 初始化状态。

**注意**: 此文件不纳入 Git 版本控制 (.gitignore: `.claude/*.local.md`)
```

---

## Minimal Mode

当使用 `--minimal` 参数时，不部署以下模版：
- docs/solutions/README.md
- docs/solutions/.template.md
- docs/openspec/README.md
- docs/openspec/templates/proposal-template.md
- docs/openspec/architectures/*.md  # NEW
- docs/plans/README.md

仅创建空目录结构和 .claude/ecf_config.yaml。

---

## Template: architecture-template

**目标**: `docs/openspec/architectures/architecture.md`

完整模板见 [architectures-templates.md](./architectures-templates.md)

---

## Template: modules-template

**目标**: `docs/openspec/architectures/modules.md`

完整模板见 [architectures-templates.md](./architectures-templates.md)

---

## Template: changes-index-template

**目标**: `docs/openspec/architectures/changes-index.md`

完整模板见 [architectures-templates.md](./architectures-templates.md)

---

## Force Mode

当使用 `--force` 参数时：
1. 备份现有文件到 `.backup/YYYYMMDD-HHMMSS/`
2. 重新部署所有模版
3. 重置配置文件为默认值