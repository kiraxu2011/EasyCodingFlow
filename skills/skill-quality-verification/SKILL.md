---
name: skill-quality-verification
description: Use when skill creation completes, skill verification layer triggers, checking skill quality before deployment, or verifying SKILL.md frontmatter format, description CSO quality, and content structure. Also triggers for 技能验证, skill检查, frontmatter验证, CSO检查, skill quality check.
---

# Skill Quality Verification

## Overview

**Core principle**: Skills验证不同于代码验证。检查 frontmatter、description CSO、内容结构，而非 spec↔design↔code↔test 一致性。

**REQUIRED**: 在 skill_development 工作流验证层调用，替代 consistency-verification。

## When to Use

**Triggering conditions**:
- Skill creation 完成，进入验证层
- Skill deployment 前质量检查
- 修改现有 skill 后验证
- 技能验证、skill检查、frontmatter验证、CSO检查

**NOT for**:
- 代码一致性验证（使用 consistency-verification）
- 非 skill 文档验证

## Verification Checks

### Frontmatter Format

| Check | Rule | Error |
|-------|------|-------|
| `name` exists | Required | "frontmatter must have 'name' field" |
| `name` format | letters/numbers/hyphens | "no parentheses, special chars" |
| `description` exists | Required | "frontmatter must have 'description'" |
| `description` length | <500 chars | "description must be under 500 chars" |

### Description CSO Quality

| Check | Rule | Error |
|-------|------|-------|
| Starts with "Use when" | Required | "description must start with 'Use when'" |
| No workflow summary | description ≠ process | "describe triggers, not workflow" |
| Third-person voice | No "I/we/you" | "description must be third person" |
| Technology-agnostic | Unless tech-specific | "describe problem, not tech symptoms" |

### Content Structure

| Check | Rule | Error |
|-------|------|-------|
| `## Overview` | Required section | "skill must have Overview" |
| `## When to Use` | Required section | "skill must have When to Use" |
| Single-language example | Not multi-language | "one excellent example beats dilution" |

## Quick Reference

```bash
# Verification checklist
1. Parse YAML frontmatter
2. Check: name format, description format
3. Check: Overview, When to Use sections
4. Check: examples single-language
5. Report: PASS or list violations
```

## Output Format

```
🔍 Skill Quality Verification
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Skill: <name>

Frontmatter: ✅/❌ name, ✅/❌ description
CSO Quality: ✅/❌ Use when, ✅/❌ no workflow, ✅/❌ third-person
Structure: ✅/❌ Overview, ✅/❌ When to Use, ✅/❌ examples

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Result: PASS / FAIL
Violations: [list if FAIL]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| description 描述流程 | 改为触发条件 |
| name 包含括号 | 改用 hyphens |
| 多语言示例 | 单一最佳语言 |
| 缺少 When to Use | 添加触发场景 |

## Red Flags - STOP

- skill 验证使用 consistency-verification
- description 包含 "step by step", "process", "workflow"
- name 包含 parentheses, underscores, spaces
- 缺少 Overview 或 When to Use

**遇到以上**: 停止，修复 skill 后重新验证。