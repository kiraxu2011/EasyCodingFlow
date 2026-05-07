# Output Format Specification

## Status Icons

| Icon | Meaning |
|------|---------|
| 📍 | Phase start |
| ✅ | Task completed |
| ⚠️ | Failure/warning |
| 📊 | Summary report |

## Phase Start

```
📍 [Phase Name] 启动
预期任务: [N] 个
```

## Task Complete

```
✅ [Role] 完成: [Summary]
```

## Failure

```
⚠️ 任务 [ID] 执行失败
原因: [Reason]
```

## Summary Report

```
📊 执行汇总
━━━━━━━━━━━━━━━━━━━━
总任务数: [N]
成功: [M] | 失败: [K] | 跳过: [J]
━━━━━━━━━━━━━━━━━━━━
结论: [Summary]
```