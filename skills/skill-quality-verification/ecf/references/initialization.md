# Initialization Check

## 触发时机

在 Intent Recognition Flow 之前执行。

## 检查序列

1. `docs/solutions/` exists (knowledge base) - Required
2. `.claude/ecf_config.yaml` exists (project config) - Required  
3. `docs/openspec/` exists (OpenSpec artifacts) - Optional
4. `docs/plans/` exists (work plans) - Optional

## Missing Required Directories

```
⚠️ Project not initialized
Missing: [missing directories list]
```

Ask user: "Would you like me to initialize now?"
- "Yes" → Execute `Skill("ecf-init")`
- "No" → Create `.claude/.ecf-degraded.flag` and proceed

## Degraded Mode

**Trigger**: `.claude/.ecf-degraded.flag` exists

**Behavior**:
- Knowledge retrieval disabled
- OpenSpec storage disabled  
- Intent recognition continues normally
- Warning: `⚠️ Knowledge retrieval unavailable`

**Exit**: Run `/ecf-init --force`

**Manual**: User can run `/ecf-init` explicitly