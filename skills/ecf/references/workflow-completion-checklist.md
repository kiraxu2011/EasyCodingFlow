# Workflow Completion Checklist

> **用途**: 工作流执行完成后，必须使用此清单检查所有步骤是否完成。防止遗漏关键步骤。

## Checklist Template

每次工作流完成后，输出以下格式的完成状态：

```
📊 工作流完成检查
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
场景类型: <scenario_type>
标准工作流: <workflow from mapping table>

执行状态:
| 步骤 | 状态 | 备注 |
|------|------|------|
| <step 1> | ✅/❌ | <brief note> |
| <step 2> | ✅/❌ | <brief note> |
| ... | ... | ... |
| ce:compound | ✅/⚠️ | **必须完成** |

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
工作流状态: 完成/未完成
缺失步骤: <list if incomplete>
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

## Step Status Definitions

| 状态 | 说明 |
|------|------|
| ✅ | 步骤已完成 |
| ⚠️ | 步骤未完成（需要执行） |
| ❌ | 步骤失败或跳过（需要修复） |
| N/A | 步骤不适用于当前场景 |

## Critical Steps (Cannot Skip)

以下步骤**必须完成**，跳过会导致工作流断裂：

| 步骤 | 所属层 | 跳过的影响 |
|------|--------|------------|
| `/opsx:archive` | Contract | 变更未闭环，无法追踪 |
| `ecf-verify` | Verification | spec↔design↔code 不一致风险 |
| `ce:compound` | Knowledge | 知识无法沉淀，团队无法 compounding |

## Common Omission Patterns

**已观测到的遗漏模式**:

1. **skill_development**: archive 后忘记 ce:compound
   - 原因: opsx:archive 输出无后续提示
   - 防止: 使用此清单强制检查

2. **new_feature**: verify 后忘记 archive + compound
   - 原因: verify 输出引导到"继续工作"
   - 防止: 在 verify 输出中添加提醒

3. **refactor**: execute 后忘记 verify + compound
   - 原因: 执行完成即认为工作结束
   - 防止: 工作流表格明确显示后续步骤

## Usage Instructions

**何时使用**:
- Archive 完成后（skill_development, new_feature, incremental）
- Verify 完成后（所有涉及代码变更的场景）
- 用户说"完成了"时

**使用方式**:
1. 识别场景类型
2. 从 workflow-templates.md 获取标准工作流
3. 对照执行历史，填写各步骤状态
4. 如有 ⚠️ 或 ❌，提醒用户完成缺失步骤

## Example

**skill_development 工作流检查**:

```
📊 工作流完成检查
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
场景类型: skill_development
标准工作流: /opsx:propose → skill-creator → skill-quality-verification → /opsx:archive → ce:compound

执行状态:
| 步骤 | 状态 | 备注 |
|------|------|------|
| /opsx:propose | ✅ | proposal, design, specs, tasks 完成 |
| skill-creator | ✅ | intent-recognition.md 修改完成 |
| skill-quality-verification | ✅ | frontmatter/CSO 检查通过 |
| /opsx:archive | ✅ | 2026-05-08-intent-recognition-workflow-output 已归档 |
| ce:compound | ⚠️ | **缺失 - 需要执行知识沉淀** |

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
工作流状态: 未完成
缺失步骤: ce:compound
→ 请执行: Skill("ce:compound")
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

## Red Flags

如果用户说"完成了"但：
- ce:compound 状态为 ⚠️ 或 ❌
- archive 状态为 ⚠️（适用于 OpenSpec 场景）
- verify 状态为 ⚠️（适用于代码变更场景）

**必须提醒用户完成缺失步骤，不要默认工作流已结束。**