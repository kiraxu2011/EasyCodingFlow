# Error Handling

## Task Failure Response

When task verification fails:
1. Output failure details (⚠️ format)
2. Record reason to task document
3. Offer user options

## User Options

| Option | Description | Use Case |
|--------|-------------|----------|
| Retry | Re-execute task | Temporary failure |
| Skip | Mark as skipped, continue | Non-critical failure |
| Abort | Stop, await human intervention | Critical failure |

## Interaction Format

```
⚠️ 任务 [ID] 执行失败
原因: [Reason]

请选择处理方式:
1. 重试 (剩余次数: [N]/max_retry)
2. 跳过并继续后续任务
3. 终止执行
```

## Max Retry Exhausted

- Escalate to human intervention
- Output: "已达到最大重试次数 [max_retry]，需要人工介入"
- Pause dependent tasks
- Save failure context

## Red Flags

- Auto-skip critical tasks
- Continue without user choice
- Untracked retry count
- Unrecorded failure reason