# Knowledge Retrieval

## Trigger Keywords

| Keyword | Example | Target |
|---------|---------|--------|
| 之前/曾经 | "之前怎么处理过认证问题" | docs/solutions/ |
| 类似 | "有没有类似的重构经验" | Similar solutions |
| 历史/经验 | "查看历史的bug修复经验" | Bug solutions |
| 怎么处理 | "这种问题怎么处理" | Problem matching |

## Search Process

```dot
digraph knowledge_retrieval {
    rankdir=LR;
    trigger -> search -> rank -> present;
    present -> apply [shape=diamond];
    apply -> "引用方案执行" [label="是"];
}
```

**Implementation**:
- Search: `docs/solutions/*.md`
- Method: Grep keywords → relevance sort → top 3

## Output Format

```
📚 知识库检索结果
━━━━━━━━━━━━━━━━━━━━
查询: [Query]
找到 [N] 个相关方案:

1. [Title] ([Filename])
   概要: [Summary]
━━━━━━━━━━━━━━━━━━━━
选择方案编号应用，或输入 'n' 不应用
```

## Knowledge Base Structure

```
docs/solutions/
├── YYYY-MM-DD-topic-solution.md
├── auth-system-solution.md
└── ...
```

## Red Flags

- No retrieval when user asks about history
- Results not sorted by relevance
- No summary presentation
- No apply option