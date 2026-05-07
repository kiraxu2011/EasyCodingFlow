---
name: ecf-verify
description: Use when execution layer completes, or when verifying spec-design-code-test alignment after implementation, or when artifacts may be inconsistent after changes. Make sure to use this skill whenever the user mentions verification, consistency check, alignment check, artifact verification, or after executing-plans completes, even if they don't explicitly ask for 'consistency-verification.'
---

# Consistency Verification

Verify alignment between spec, design, code, and tests using parallel sub-agent analysis.

## Overview

Ensures traceability across all artifact layers - catching inconsistencies before they become production issues.

**Core principle:** Bidirectional verification - spec→design AND design→spec, not just presence checks.

## When to Use

- After executing-plans Phase 4 completes (automatic invocation)
- User requests verification of artifact consistency
- Suspected misalignment between specification and implementation
- User invokes `/ecf-verify`

## Arguments

- `[design-folder-path]`: Path to design folder containing spec and plan files
- If not provided: searches `docs/plans/` for latest `*-design/` directory

## Workflow

```
Parse path → Launch 3 parallel analyzers → Aggregate results → Phase 1 Report → Phase 2: Architecture-Doc Check → Final Report
```

## Verification Dimensions

| Analyzer | Checks |
|----------|--------|
| Spec-Design | Feature coverage, terminology consistency |
| Design-Code | Module existence, interface signatures |
| Spec-Test | Scenario coverage, test completeness |

## Execution Flow

**Step 1: Resolve Path**
```bash
# If argument provided
design_path="$ARGUMENTS"

# If no argument, find latest
design_path=$(ls -td docs/plans/*-design/ 2>/dev/null | head -1)

# Verify path exists
[ -d "$design_path" ] || echo "⚠️ Design folder not found: $design_path"
```

**Step 2: Launch Parallel Analyzers**

Use Agent tool to spawn 3 parallel subagents:
- Analyzer 1: Spec-Design alignment
- Analyzer 2: Design-Code alignment
- Analyzer 3: Spec-Test coverage

**Step 3: Aggregate Results**

Collect findings from all analyzers:
- Consistency score per dimension
- Inconsistency items with evidence (line numbers + excerpts)
- Severity classification (high/medium/low)

**Step 4: Output Report**

Save to `docs/plans/*-design/verification-report.md`:
```markdown
# Verification Report

## Summary
- Spec-Design: ✅/⚠️/❌
- Design-Code: ✅/⚠️/❌
- Spec-Test: ✅/⚠️/❌

## Findings
[List of inconsistencies with evidence]
```

## Phase 2: Architecture-Doc Consistency Check

总文档一致性检查，更新 docs/openspec/architectures/ 目录。

**触发条件**: Phase 1 完成后自动触发

**执行流程**:

```mermaid
flowchart TD
    A["Phase 1 完成"] --> B["获取 git diff"]
    B --> C["启动 Architecture-Doc Analyzer"]
    
    C --> D["分析变更类型"]
    D --> E{"变更类型判断"}
    
    E -->|"架构变更"| F["更新 architecture.md"]
    E -->|"模块变更"| G["更新 modules.md"]
    E -->|"任何变更"| H["追加 changes-index.md"]
    
    F --> I["验证文档完整性"]
    G --> I
    H --> I
    
    I --> J["输出更新报告"]
```

**变更类型判断**:

| 变更信号 | 触发更新 |
|---------|---------|
| 技术栈变更 | architecture.md 技术栈表 |
| 新增目录 | modules.md 新增模块条目 |
| 删除目录 | modules.md 模块状态改为 archived |
| 核心入口变更 | modules.md 模块入口更新 |
| 接口契约变更 | modules.md 接口契约 |
| 任何变更 | changes-index.md 追加条目 |

**Analyzer 模板**: 见 [architecture-doc-analyzer.md](../ecf/references/consistency-verification/architecture-doc-analyzer.md)

**错误处理**:

| 异常情况 | 处理方式 |
|---------|---------|
| architectures 目录不存在 | 调用 ecf-init 创建 |
| 文档格式损坏 | 重建空白模板 |

## Fix Options

Per inconsistency item:
- **[A] 更新代码**: Modify code to match spec/design
- **[B] 更新文档**: Modify docs to match code
- **[跳过]**: Record but don't fix (max 20%, high-impact items cannot skip)
- **[重新验证]**: Re-run analyzer

## Red Flags - STOP

- "Tests pass, so it must be consistent"
- "The code looks correct"
- "I can just scan quickly"
- "Developer just finished, probably fine"
- "I'll compare in my head"

**遇到以上情况**: 执行完整 traceability matrix 分析，不可跳过。

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Keyword matching instead of semantic analysis | Parse intent, not just text |
| Presence checks without correctness | Verify functionality matches |
| Single-direction verification | Bidirectional: spec→design AND design→spec |
| No evidence output | Line numbers + content excerpt required |

## Summary Output

```
📊 Verification Summary
━━━━━━━━━━━━━━━━━━━━
Phase 1 - Artifact Alignment:
  Spec-Design: [score]
  Design-Code: [score]
  Spec-Test: [score]

Phase 2 - Architecture-Doc:
  变更类型: [count] 个
  文档更新: architecture.md / modules.md / changes-index.md
  完整性: ✅/⚠️
━━━━━━━━━━━━━━━━━━━━
[✅ All aligned / ⚠️ N inconsistencies found]
```

## References

- [analyzer-templates.md](../ecf/references/consistency-verification/analyzer-templates.md) - Sub-agent prompt templates
- [report-format.md](../ecf/references/consistency-verification/report-format.md) - Report structure