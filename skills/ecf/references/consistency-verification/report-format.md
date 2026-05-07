# Report Format

Verification report structure for consistency verification.

## Report Template

```markdown
# 一致性验证报告

## 概要
- 验证时间: [ISO 8601]
- 设计文档: docs/plans/YYYY-MM-DD-*-design/
- OpenSpec源: docs/openspec/.../ (如存在，否则注明"不存在")

## 一致性统计

| 维度 | 状态 | 一致项 | 不一致项 |
|------|------|--------|----------|
| spec ↔ design | [状态图标] | N | M |
| design ↔ code | [状态图标] | N | M |
| spec ↔ tests | [状态图标] | N | M |

状态图标: ✅ 一致 / ⚠️ 有不一致 / ❌ 有缺失 / 🔍 待验证

## Traceability Matrix

### Spec-Design
| Spec Scenario | Design Section | Status |
|---------------|----------------|--------|
| Scenario 1 | Architecture Ch2 | ✅ |
| Scenario 2 | - | ❌ Missing |

### Design-Code
| Design Component | Code File | Status |
|------------------|-----------|--------|
| AuthModule | src/auth/ | ✅ |
| SessionModule | - | ❌ Not implemented |

### Spec-Test
| Spec Scenario | Test Function | Status |
|---------------|---------------|--------|
| Scenario 1 | test_scenario_1 | ✅ |
| Scenario 2 | - | ❌ No test |

## 不一致详情

### [序号]. [维度]: [类型]
- **不一致项**: [描述]
- **来源**: [文件 + 位置]
- **影响**: [高/中/低] - [原因]
- **修复建议**:
  - [A] [描述]
  - [B] [描述]
- **推荐**: [推荐选项] - [原因]

---

## 用户选择记录

| 序号 | 用户选择 | 执行结果 |
|------|----------|----------|
| 1 | A | ✅ 已修复 |
| 2 | 跳过 | ⚠️ 已记录 |

## 最终状态
[✅ 验证通过 / ⚠️ 验证完成但有跳过项]
```

## Status Icons

| Icon | Meaning | Evidence Required |
|------|---------|-------------------|
| ✅ | Fully consistent | Line number + content excerpt |
| ⚠️ | Has inconsistencies | Specific mismatch details |
| ❌ | Has missing elements | Search proof (where looked) |
| 🔍 | Needs re-verification | Reason for ambiguity |
| ⏭️ | Skipped by user | Justification recorded |

**Evidence Quality Requirements:**
- Line numbers or section anchors required
- Brief content excerpt proving match/mismatch
- Confidence level: 高/中/低

## Priority Levels

| Level | Criteria |
|-------|----------|
| **高** | Core functionality missing, security issue, data integrity |
| **中** | Interface mismatch, terminology confusion, partial coverage |
| **低** | Minor inconsistency, naming variation, cosmetic issue |

## Fix Recommendation Logic

```
if spec is source_of_truth:
    recommend: update code/design to match spec
else if code is reality:
    recommend: update docs to match code (with user confirmation)
else:
    present both options equally
```

## User Confirmation Requirements

**最终状态 "验证通过" 必须满足:**
- 所有高影响项已处理（不可跳过）
- 用户已确认所有不一致项处理方案
- 跳过项不超过 20%

**跳过限制:**
- 高影响（高）项: 不可跳过
- 中影响（中）项: 需记录理由
- 低影响（低）项: 可跳过，但需记录