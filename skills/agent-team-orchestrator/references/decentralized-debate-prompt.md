# Decentralized Debate Round Prompt Template

`decentralized_debate` 模式下，每一轮辩论每个 agent 的 prompt 模板。

## Template

```
你是 Agent {agent_id}，在 Phase 1 使用 {model} 模型完成了独立分析，在 Phase 2 完成了交叉校准。现在进入多轮共识辩论阶段。

## 完整上下文

### Phase 1: 所有 Agent 独立分析结果

{phase_1_full_output}

### Phase 2: 交叉校准结果

{phase_2_full_output}

### 之前辩论轮次历史

{debate_history}

## 当前未解决分歧

{open_disputes_list}

## 你的身份

你依然是 Agent {agent_id}，使用 {model} 模型。请基于完整上下文，针对每个未解决的分歧，更新你的立场。

## 指令

1. 仔细阅读所有上下文：Phase 1 独立分析、Phase 2 交叉校准、以及之前所有辩论轮次的发言。
2. 针对每个**当前未解决的分歧**：
   - 给出你**更新后的最终立场**
   - 如果你被其他 agent 的论据说服了，请**明确说明改变立场**，接受对方观点
   - 如果你仍然坚持自己原来的立场，请**回应对你的挑战**，解释为什么你的观点仍然成立
   - 如果你仍然不认同其他 agent 的某个立场，请**继续提出你的挑战**，给出你的论据
3. **对事不对人**：基于论据辩论，不纠结立场，接受合理说服。如果对方的论据更充分，大方承认并改变立场是完全可以接受的。

## 输出要求

请按以下结构输出：

### 你的更新立场

对于每个未解决分歧：

#### 分歧: <分歧主题>
- **你的最终立场**: <坚持原有立场 / 改变立场接受 XXX 观点 / 继续挑战 XXX 的观点>
- **论据**: <你的理由，回应之前的挑战>

### 新发现的盲区（如有）
如果你在辩论中发现了仍然被所有人忽略的重要问题，请列出来。

### 当前你认同的共识
如果你认为某些分歧已经解决，可以明确标记出来。
```

## Notes

- `{agent_id}`, `{model}`, `{phase_1_full_output}`, `{phase_2_full_output}`, `{debate_history}`, `{open_disputes_list}` 由 orchestrator 在启动辩论轮次时填充
- 每个 agent 在每一轮辩论中都能看到完整的历史，保证信息对称
