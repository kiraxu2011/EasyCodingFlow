# Phase Completion Validation

## Overview

统一的阶段完成验证框架，用于在每个工作流步骤完成后验证该步骤确实已执行，预期产物确实存在，防止模型幻觉跳步。

**Core Principle:** 每个步骤完成后 **立即验证**，验证通过才能进入下一阶段。验证基于文件系统客观检查，不依赖模型记忆。

## Uniform Validation Template

所有验证使用统一输出格式：

### Success Template
```
🔍 [步骤名称] 完成验证
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ [检查项 1]: <description> ✅
✓ [检查项 2]: <description> ✅
...
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ 验证通过，进入下一阶段
```

### Failure Template
```
🔍 [步骤名称] 完成验证
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ [检查项 1]: <description> ✅
✗ [检查项 2]: <description> ❌
  预期: <expected-path-or-condition>
  实际: <actual-observation>
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
❌ 验证失败，请修复后继续。原因：<brief-reason>
```

**Rules:**
- 使用 `🔍` 开头，`━━━━━━━━━━━━━━━━━━━━━━━━━━━━━` 分隔线
- 每个检查项用 `✓` 或 `✗` 前缀，后面跟着 `✅` 或 `❌` 后缀
- 失败项必须说明预期和实际情况
- 验证失败后**必须中断流程**，不允许继续下一阶段

## Validation Check Types

### 1. Directory Existence
```bash
if [ -d "path/to/dir" ]; then
  # pass
else
  # fail
fi
```

### 2. File Existence
```bash
if [ -f "path/to/file" ]; then
  # pass
else
  # fail
fi
```

### 3. File Non-Empty
```bash
if [ -s "path/to/file" ]; then
  # pass (file size > 0)
else
  # fail (file empty)
fi
```

### 4. Variable Non-Empty
```bash
if [ -n "$variable" ]; then
  # pass
else
  # fail
fi
```

## Per-Step Validation Checklist (ecf main orchestration)

| Step | Validation Checks | Type |
|------|-------------------|------|
| **Pre-flight Check** | | |
| | Arguments not empty | Variable check |
| | Required directories initialized | Directory check |
| | Dependency check completed | Variable check |
| | Missing dependencies handled | Variable check |
| **Intent Recognition** | | |
| | Intent analysis output generated | Already done by current flow |
| | Scenario type identified | Variable check |
| | Contract entry point determined | Variable check |
| **/opsx:propose** | | |
| | Change directory created | Directory check: `openspec/changes/<name>/` |
| | proposal.md created and non-empty | File check: `openspec/changes/<name>/proposal.md` |
| | design.md created and non-empty | File check: `openspec/changes/<name>/design.md` |
| | tasks.md created and non-empty | File check: `openspec/changes/<name>/tasks.md` |
| **brainstorming** | | |
| | Brainstorm output completed | Already done by skill |
| **writing-plans** | | |
| | Plan directory created | Directory check: `docs/plans/*-design/` |
| | _index.md exists | File check |
| | architecture.md exists | File check |
| **skill-creator** | | |
| | Skill-creator process completed | Already done by skill |
| | Evaluation workspace created | Directory check |
| | evals.json created and non-empty | File check: `*/evals/evals.json` |
| | Eval verification completed (no remaining FAIL items) | Status check |
| **ecf-execute** | | |
| | Plan path exists and valid | Directory check |
| | All tasks in batch completed | Status check (all PASS) |
| | Execution summary output | Already done |
| **ecf-verify** | | |
| | Design path exists | Directory check |
| | Verification report written | File check: `*/verification-report.md` |
| **skill-quality-verification** | | |
| | All three check categories completed | Already done by skill |
| | Verification result output | Already done |
| **/opsx:archive** | | |
| | Change archived to openspec/changes/archive/ | Directory/file check |
| | Archive metadata recorded | File check |
| **ce:compound** | | |
| | Solution document written to docs/solutions/ | File check |

## Per-Skill Self-Validation Checklist

### ecf-init
| Check | Type |
|-------|------|
| docs/solutions/ directory exists | Directory |
| docs/plans/ directory exists | Directory |
| docs/openspec/ directory structure exists | Directory |
| .claude/ecf_config.yaml exists and non-empty | File non-empty |
| .claude/.ecf-init.local.md exists and non-empty | File non-empty |

### ecf-execute
| Check | Type |
|-------|------|
| Environment check completed | Done |
| Plan path resolved and exists | Directory |
| Tasks analyzed | Done |
| Execution mode selected | Done |
| All tasks in batch completed with PASS | Status |
| Execution summary output | Done |

### ecf-verify
| Check | Type |
|-------|------|
| Design path resolved and exists | Directory |
| Three parallel analyzers completed | Done |
| Verification report written and non-empty | File non-empty |
| Architecture-doc check completed (if needed) | Done |

### skill-quality-verification
| Check | Type |
|-------|------|
| Frontmatter format check completed | Done |
| Description CSO check completed | Done |
| Content structure check completed | Done |
| Verification result output generated | Done |

## Failure Handling Policy

1. **Fail-fast**: 验证失败立即中断流程，不允许继续下一阶段
2. **Clear reporting**: 清晰报告哪个检查项失败，预期是什么，实际是什么
3. **No automatic recovery**: 不尝试自动修复，由用户决定如何修复
4. **Knowledge retrieval**: 如果失败是已知问题，触发知识检索查找历史修复方案

## Integration with Exception Knowledge Retrieval

当验证失败时，自动触发异常知识检索流程：

```bash
# 验证失败后
if [ ! -f ".claude/.ecf-degraded.flag" ]; then
  # Extract keywords from failure context
  # Search docs/solutions/
  # If match found, show solution recommendation
fi
```

详见 [knowledge-retrieval.md](./knowledge-retrieval.md)。
