---
name: skill-quality-verification
description: Use when skill creation completes and verification layer triggers, or when checking skill quality before deployment. Checks frontmatter format, description CSO quality, and content structure.
---

# Skill Quality Verification

## Overview

**Core principle**: Skills 验证不同于代码验证。检查 frontmatter、description CSO、内容结构，而非 spec↔design↔code↔test 一致性。

**REQUIRED**: 在 skill_development 工作流验证层调用，替代 consistency-verification。

## When to Use

**Triggering conditions**:
- Skill creation 完成，进入验证层
- Skill deployment 前质量检查
- 修改现有 skill 后验证

**NOT for**:
- 代码一致性验证（使用 consistency-verification）
- 非 skill 文档验证

## Verification Checks

### 1. Frontmatter Format Check

| Check | Requirement | Error Message |
|-------|-------------|---------------|
| `name` exists | Required field | "frontmatter must have 'name' field" |
| `name` format | Letters, numbers, hyphens only | "name must use letters, numbers, hyphens only (no parentheses, special chars)" |
| `description` exists | Required field | "frontmatter must have 'description' field" |
| `description` length | < 500 chars | "description must be under 500 characters" |

### 2. Description CSO Check

| Check | Requirement | Error Message |
|-------|-------------|---------------|
| Starts with "Use when" | Required format | "description must start with 'Use when'" |
| No workflow summary | Description ≠ process | "description must describe triggers, not workflow" |
| Third-person voice | No "I", "we", "you" | "description must be in third person" |
| Technology-agnostic | Unless skill is tech-specific | "description should describe problem, not technology-specific symptoms" |

**CSO Violation Examples**:

```yaml
# ❌ BAD: Summarizes workflow
description: Use when creating skills - follows RED-GREEN-REFACTOR cycle with pressure scenarios

# ✅ GOOD: Just triggering conditions
description: Use when creating new skills, editing existing skills, or verifying skills work before deployment
```

### 3. Content Structure Check

| Check | Requirement | Error Message |
|-------|-------------|---------------|
| `## Overview` exists | Required section | "skill must have Overview section" |
| `## When to Use` exists | Required section | "skill must have When to Use section" |
| One excellent example | Not multi-language | "one excellent example beats multi-language dilution" |

## Quick Reference

```bash
# Verification checklist
1. Read skill SKILL.md
2. Parse YAML frontmatter
3. Check: name format, description format
4. Check: content has Overview, When to Use
5. Check: examples are single-language
6. Report: PASS or list violations
```

## Output Format

```
🔍 Skill Quality Verification
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Skill: <skill-name>

Frontmatter:
✅ name: <value>
✅ description: <value>

CSO Quality:
✅ Starts with "Use when"
✅ No workflow summary
✅ Third-person voice

Content Structure:
✅ Overview section
✅ When to Use section
✅ Single-language examples

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Result: PASS / FAIL
Violations: [list if FAIL]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| description 描述流程 | 改为描述触发条件 |
| name 包含括号 | 改用 hyphens 连接 |
| 多语言示例 | 选择一个最佳语言示例 |
| 缺少 When to Use | 添加触发条件和场景 |

## Red Flags - STOP

- skill 验证使用 consistency-verification（应使用 skill-quality-verification）
- description 包含 "step by step", "process", "workflow"
- name 包含 parentheses, underscores, spaces
- 缺少 Overview 或 When to Use section

**遇到以上**: 停止，修复 skill 文件后重新验证。