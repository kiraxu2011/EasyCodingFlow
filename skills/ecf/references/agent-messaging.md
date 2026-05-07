# Agent Messaging

## Communication Scenarios

| Scenario | From | To | Content |
|----------|------|-----|---------|
| API contract | Backend | Frontend | API definition |
| Task complete | Agent | Coordinator | Status, artifacts |
| Dependency unlock | Agent-A | Agent-B | "Ready" + context |

## Message Format

```yaml
Message:
  from: [sender_id]
  to: [recipient_id] | "all" | "coordinator"
  type: "artifact_share" | "completion" | "dependency_unlock"
  content:
    summary: "[Summary]"
    artifacts: ["path1", "path2"]
    context: "[Context]"
```

## Implementation

Use Agent tool's SendMessage:
```markdown
SendMessage to: '[agent_id]'
内容: YAML message format
```

## Message Types

**artifact_share**: Share artifacts
**completion**: Task done
**dependency_unlock**: Ready for dependent task

## Red Flags

- No notification after task completion
- Missing artifacts
- Non-standard format
- Start dependent before unlock