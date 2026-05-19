# Agent Team Orchestrator - 多模型 Agent 团队编排器

> 用多模型认知多样性提升分析质量 —— **独立思考 → 交叉挑战 → 收敛共识**

## 📋 目录

- [概述](#概述)
- [核心原理](#核心原理)
- [快速开始](#快速开始)
- [配置说明](#配置说明)
- [使用场景](#使用场景)
  - [头脑风暴模式](#头脑风暴模式brainstorming)
  - [代码审查模式](#代码审查模式code_review)
- [工作流详解](#工作流详解)
  - [Phase 1: 独立思考](#phase-1-independent-thinking独立思考)
  - [Phase 2: 交叉校准](#phase-2-cross-calibration交叉校准)
  - [Phase 3: 共识收敛](#phase-3-consensus-convergence共识收敛)
- [输出报告结构](#输出报告结构)
- [常见问题](#常见问题)
- [开发参考](#开发参考)

## 概述

Agent Team Orchestrator 是 EasyCodingFlow (ECF) 生态中的核心编排技能，它利用**不同模型的认知多样性**来提升复杂分析任务的质量。

**支持两种分析场景：**
- 🧠 **头脑风暴** —— 技术方案设计、架构决策、技术选型
- 🔍 **代码审查** —— PR 审查、代码质量检查、安全审计

**核心优势：**
- ✅ 消除单一模型的认知盲区
- ✅ 通过交叉挑战暴露隐藏假设
- ✅ 基于论据强度而非投票做决策
- ✅ 完全可配置的模型组合和工作流
- ✅ 自动降级支持单次快速跳过

## 核心原理

> "多个独立大脑比一个大脑更聪明，前提是它们先独立思考，再互相挑战。"

### 为什么需要多模型编排？

不同模型有不同的认知特性：
| 模型 | 特性 | 优势 | 劣势 |
|------|------|------|------|
| Opus | 深度推理 | 复杂问题分析透彻 | 成本高、速度慢 |
| Sonnet | 平衡 | 速度与质量均衡 | - |
| Haiku | 快速直觉 | 反应快、成本低 | 深度不足 |

通过让**不同模型独立思考**，再**互相挑战**，最终**收敛共识**，可以得到比单一模型更好的分析结果。

### 三阶段流水线

```
┌─────────────────────────────────────────────────────────────┐
│  Phase 1: Independent Thinking  独立思考                     │
│  ────────────────────────────────────────────────────────  │
│  N 个 agent 隔离环境中独立分析，互不影响                      │
│  避免锚定效应，保证每个视角真正独立                            │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│  Phase 2: Cross-Calibration  交叉校准                       │
│  ────────────────────────────────────────────────────────  │
│  每个 agent 阅读其他 agent 的产出                            │
│  大胆质疑、发现盲区、挑战假设                                │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│  Phase 3: Consensus Convergence  共识收敛                   │
│  ────────────────────────────────────────────────────────  │
│  综合所有独立分析 + 交叉挑战意见                             │
│  输出结构化共识报告                                          │
└─────────────────────────────────────────────────────────────┘
```

## 快速开始

### 1. 调用方式

在 ECF 工作流中，通过 `Skill` 调用：

```javascript
// 头脑风暴模式
Skill("agent-team-orchestrator", {
  mode: "brainstorming",
  task: "我需要为高并发订单系统设计库存扣减方案，解决超卖问题..."
})
```

```javascript
// 代码审查模式
Skill("agent-team-orchestrator", {
  mode: "code_review",
  task: "审查这个 PR 中新增的 Redis 缓存层，关注缓存失效竞态条件...",
  scope: "src/auth/session.py (200 行变更)"
})
```

### 2. 启用/禁用

在 `.claude/ecf_config.yaml` 中配置：

```yaml
agent_team_orchestrator:
  enabled: true  # 设置 false 禁用，降级为直接调用基础技能
```

### 3. 快速模式（单次跳过）

在用户请求中包含以下关键词，会**单次跳过**编排，直接调用基础技能：
- `--no-orchestrator`
- `快速模式`
- `跳过编排`

示例：
```
快速模式 -- 帮我简单想一下这个 API 的分页方案应该用 offset 还是 cursor...
```

## 配置说明

### 完整配置 Schema

配置从以下位置按优先级加载（高优先级覆盖低优先级）：

1. **环境变量**（最高优先级）
2. **项目配置** → `.claude/ecf_config.yaml`
3. **全局配置** → `~/.claude/skills/ecf/ecf_config.yaml`
4. **内置默认值**（最低优先级）

```yaml
agent_team_orchestrator:
  # 是否启用 orchestrator
  # false 时，brainstorming/code_review 直接调用基础技能
  # 默认: true
  enabled: true

  # Phase 1 使用的模型列表
  # 模型数量决定 agent 数量（N 个模型 = N 个 agent）
  # 有效值: opus, sonnet, haiku
  # 默认: [opus, sonnet, haiku]
  models:
    - opus
    - sonnet
    - haiku

  # Phase 2 交叉校准轮次
  # 1 = 单轮交叉审查（默认，大多数场景足够）
  # 2 = 交叉审查后再交叉审查一次，更深入但成本更高
  # 默认: 1
  cross_calibration_rounds: 1

  # Phase 2 交叉校准策略
  # "distributed" (默认) - 每个 agent 审查所有其他 agent 的输出
  # "centralized" - 单个综合 agent 对比所有输出（更快，但细节较少）
  # 默认: "distributed"
  cross_calibration_strategy: "distributed"

  # Phase 3 共识收敛策略
  # "centralized" (默认) - orchestrator 中心化综合生成共识报告（原有行为）
  # "decentralized_debate" - 所有 agent 参与多轮辩论共同达成共识（新增，更高质量但成本更高）
  # 默认: "centralized" (保持向后兼容)
  consensus_strategy: "centralized"

  # 集体辩论最大轮次（仅 decentralized_debate）
  # 达到轮次后强制终止，如实记录各方最终立场
  # 默认: 2
  consensus_max_rounds: 2

  # 提前终止检测（仅 decentralized_debate）
  # true = 全部分歧消除后提前停止，节省成本
  # false = 强制完成所有轮次
  # 默认: true
  consensus_early_stop: true
```

完整配置见：[references/config-schema.md](./references/config-schema.md)

### 环境变量覆盖

| 环境变量 | 格式 | 示例 | 覆盖配置项 |
|----------|------|------|-----------|
| `ATO_ENABLED` | `true`/`false` | `ATO_ENABLED=false` | `enabled` |
| `ATO_MODELS` | 逗号分隔 | `ATO_MODELS=opus,sonnet` | `models` |
| `ATO_CROSS_CALIBRATION_ROUNDS` | 整数 | `ATO_CROSS_CALIBRATION_ROUNDS=2` | `cross_calibration_rounds` |
| `ATO_CONSENSUS_STRATEGY` | `centralized`/`decentralized_debate` | `ATO_CONSENSUS_STRATEGY=decentralized_debate` | `consensus_strategy` |
| `ATO_CONSENSUS_MAX_ROUNDS` | 整数 | `ATO_CONSENSUS_MAX_ROUNDS=3` | `consensus_max_rounds` |
| `ATO_CONSENSUS_EARLY_STOP` | `true`/`false` | `ATO_CONSENSUS_EARLY_STOP=false` | `consensus_early_stop` |

### 常用配置示例

#### 默认配置（推荐大多数场景）
```yaml
agent_team_orchestrator:
  enabled: true
  models: [opus, sonnet, haiku]
  cross_calibration_rounds: 1
  cross_calibration_strategy: "distributed"
  consensus_strategy: "centralized"
```

#### 最高质量分析（启用集体辩论，成本较高）
```yaml
agent_team_orchestrator:
  enabled: true
  models: [opus, opus, sonnet]  # 两个 Opus + 一个 Sonnet 保证深度
  cross_calibration_rounds: 1
  consensus_strategy: "decentralized_debate"  # 启用集体辩论收敛
  consensus_max_rounds: 2          # 最多两轮辩论
  consensus_early_stop: true       # 分歧消除提前停止
```

#### 快速分析
```yaml
agent_team_orchestrator:
  enabled: true
  models: [sonnet, haiku]  # 只使用 Sonnet + Haiku
  cross_calibration_rounds: 1
  consensus_strategy: "centralized"
```

#### 禁用编排（完全降级）
```yaml
agent_team_orchestrator:
  enabled: false
```

## 使用场景

### 头脑风暴模式 (brainstorming)

#### 适用场景
- 新技术方案设计
- 架构决策讨论
- 技术选型评估
- 重构方案探讨

#### 每个 Phase 1 Agent 的输出要求

Agent 必须按以下结构输出：

1. **需求理解与约束分析**
   - 核心问题是什么
   - 关键约束条件
   - 隐含假设（你做了哪些假设）

2. **方案设计（至少 2 个）**
   - 方案描述（架构思路、技术选型）
   - 优势分析
   - 劣势与风险
   - 适用场景

3. **方案对比**
   | 维度 | 方案 A | 方案 B | ... |
   |------|--------|--------|-----|
   | 复杂度 | | | |
   | 可扩展性 | | | |
   | 风险 | | | |
   | 开发成本 | | | |

4. **推荐方案**
   - 首选方案及理由
   - 关键风险与缓解措施
   - 分析中最不确定的部分

5. **自我质疑**
   - 推荐方案在什么情况下会失败？
   - 最没有把握的判断是什么？
   - 如果必须反对自己的推荐方案，你会怎么说？

> **关键点**："自我质疑" 要求 agent 在独立阶段就挑战自己的结论，为交叉校准提供靶点。

#### 示例调用

```javascript
Skill("agent-team-orchestrator", {
  mode: "brainstorming",
  task: "当前订单系统在每秒 1000 单时会出现超卖问题，需要设计一个既能保证一致性又能支撑高并发的库存扣减架构。"
})
```

---

### 代码审查模式 (code_review)

#### 适用场景
- Pull Request 审查
- 新增代码质量检查
- 安全漏洞审计
- 架构合规性检查

#### 立场分类标准

每个发现**必须**标注明确立场，**禁止模糊表述**：

| 立场 | 适用场景 | 示例 |
|------|----------|------|
| **MUST_FIX** | 必须修复 | 安全漏洞、数据损坏风险、逻辑错误、会导致生产事故的问题 |
| **SHOULD_FIX** | 建议修复 | 代码异味、可维护性问题、性能隐患、违反团队规范 |
| **OK** | 可接受 | 风格偏好、可接受的权衡、主观判断差异 |

**严格禁止**以下模糊表述：
- "修不修都行"
- "可选"
- "建议考虑"
- "可以改进但也不是必须的"
- 任何不给出确切立场的表述

如果一个问题你确实认为修不修复都可以接受 → 标注为 **OK** 并说明为什么可以接受。

#### 每个 Phase 1 Agent 的输出要求

Agent 必须按以下结构输出：

1. **审查概览**
   - 审查范围和变更规模
   - 整体代码质量判断
   - 主要风险领域

2. **MUST_FIX 发现**
   对每个发现：
   - 位置：`文件:行号`
   - 问题描述：具体什么问题
   - 立场：`MUST_FIX`
   - 理由：为什么必须修复
   - 修复建议：具体如何修复

3. **SHOULD_FIX 发现**（同上结构）

4. **OK 发现**
   （如果你注意到某些代码可能被质疑但你认为可以接受）
   - 位置：`文件:行号`
   - 观察：注意到的模式或写法
   - 立场：`OK`
   - 理由：为什么可以接受

5. **审查盲区自评**
   - 你认为自己的审查在哪些方面可能不全面？
   - 哪些类型的 bug 或问题你不太擅长发现？
   - 如果你必须挑自己审查报告中最可能出错的一个判断，是哪个？

> **关键点**："审查盲区自评" 让 agent 暴露自己的弱点，为交叉校准提供靶点。

#### 模糊立场检测

Orchestrator 在 Phase 1 收集结果后会自动扫描每个 agent 的产出，如果检测到模糊表述，会要求该 agent **重新明确立场**后才能进入 Phase 2。

#### 示例调用

```javascript
Skill("agent-team-orchestrator", {
  mode: "code_review",
  task: "我们在用户会话服务中添加了 Redis 缓存层，我担心缓存失效的竞态条件和内存压力问题。",
  scope: "src/auth/session.py，约 200 行变更"
})
```

## 工作流详解

### Phase 1: Independent Thinking（独立思考）

**目标**：N 个 agent 各自在隔离上下文中独立完成分析，互不知晓对方存在。

**为什么隔离？** 如果 agent 在思考阶段就能看到其他人的结论，会产生**锚定效应**——过早统一思路导致后续碰撞无意义。隔离确保每个视角都是真正独立的。

**执行方式：**
- 并行启动 N 个 `Agent` tool 调用（一次发送多个调用）
- 每个 agent 使用配置列表中对应模型
- 每个 agent 的 prompt 中明确写入隔离指令："当前阶段禁止与其他 agent 交流，请基于你自己的判断形成完整结论"
- Orchestrator 等待**所有** agent 完成后再进入 Phase 2

**Brainstorming 模式要点：**
- Agent 内部可调用 `Skill("superpowers:brainstorming")` 辅助思考
- 要求产出完整的多方案分析，包含自我质疑

**Code Review 模式要点：**
- Agent 内部可调用 `Skill("compound-engineering:ce-review")` 辅助审查
- 要求每个发现必须标注明确立场，禁止模糊表述

### Phase 2: Cross-Calibration（交叉校准）

**目标**：让各 agent 阅读彼此的产出，大胆质疑，发现盲区和矛盾。

**为什么交叉？** 独立分析后最容易忽视的是**集体盲区**——所有 agent 都没想到的问题。交叉审查通过让每个 agent 阅读其他人的结论来暴露这些盲区。

#### 分布式策略（distributed，默认推荐）

每个 agent 审查**所有其他 agent**的产出：

```
Agent 阅读所有 Phase 1 产出后需要回答：
  1. 你不同意的观点有哪些？为什么？
  2. 所有产出中共同的盲区是什么？
  3. 哪些重要问题被所有 agent 忽略了？
  4. 针对其他 agent 的结论，提出直接挑战——不要客气。

Brainstorming 附加问题：如果将不同方案的最优部分组合，会是什么样？
Code Review 附加问题：哪些发现的立场你不同意？
```

适用于：agent 数量 ≤ 4（大多数场景）

#### 集中式策略（centralized）

单个综合 agent 对比所有 Phase 1 产出，找出分歧点和共识点。

适用于：agent 数量 > 4，或需要更快完成的场景

#### 多轮交叉校准

如果配置 `cross_calibration_rounds: 2`，第一轮交叉完成后会再进行一轮：每个 agent 阅读其他 agent 的挑战意见，再做一轮回应。这会产生更深层次的辩论，但成本也更高。

### Phase 3: Consensus Convergence（共识收敛）

**目标**：综合所有独立分析 + 交叉审查意见，产出最终报告。支持两种策略：

#### 中心化收敛（centralized，默认）

- orchestrator（主调用者）综合所有 Phase 1 和 Phase 2 产出生成最终报告
- 成本低，速度快，保持原有行为
- **收敛原则：**
  1. **多数一致 + 无有效挑战** → 采纳为共识
  2. **有分歧** → 分析论据质量（**不是**按 agent 投票），基于论据强度而非 agent 数量做决策
  3. **单个 agent 提出独特观点且论证充分** → 即使其他 agent 未提及也要纳入推荐
  4. **低质量模型输出**（如 haiku 分析明显浅于 opus）→ 在综合时给予较低权重

#### 集体辩论收敛（decentralized_debate，可选，高质量）

- 所有 agent 保持原有身份和模型，参与多轮辩论
- 每个 agent 可以基于辩论更新自己的立场（坚持/改变/挑战）
- **成本**：额外需要 `N × R` 次 agent 调用（N = agent 数量，R = 最大轮次），默认配置约增加 6 倍成本
- **流程：**
  1. 启动辩论前向用户提示成本增量并请求确认
  2. 每轮每个 agent 看到完整历史（Phase 1 + Phase 2 + 所有先前辩论），更新立场
  3. 如果开启提前停止且全部分歧消除 → 提前终止
  4. 达到最大轮次强制终止，如实记录各方最终立场
  5. 最终报告基于所有 agent 最终立场生成，仍有分歧时如实呈现由用户决策

## 输出报告结构

### 最终共识报告结构

```markdown
# [Brainstorming / Code Review] 共识报告

## 参与 Agent
| Agent | 模型 | 角色 |
|-------|------|------|
| Agent A | opus | 独立分析 → 交叉校准 → 辩论 |
| Agent B | sonnet | 独立分析 → 交叉校准 → 辩论 |
| Agent C | haiku | 独立分析 → 交叉校准 → 辩论 |

## 共识点
<!-- 所有 agent 一致同意的结论 -->

## 分歧点
| 议题 | Agent A | Agent B | Agent C | 辩论结果 |
|------|---------|---------|---------|----------|
| ... | ... | ... | ... | ... |

## 盲区发现
<!-- 交叉校准中新发现的，所有 agent 独立阶段都遗漏的问题 -->

## 最终推荐

### Brainstorming 模式
- **推荐方案**: <综合多方视角后的最优方案>
- **方案对比矩阵**: 各方案在关键维度上的对比
- **保留的备选方案**: <虽然未选中但有价值保留的方案>
- **风险提示**: 综合所有 agent 视角的风险清单

### Code Review 模式
- **MUST_FIX 清单** (合并去重): <必须修复的发现列表>
- **SHOULD_FIX 清单** (合并去重): <建议修复的发现列表>
- **立场分歧专项**: <同一发现但不同 agent 给出不同立场的详细辩论>
- **审查覆盖盲区**: <可能被遗漏的审查维度>
```

## 常见问题

### Q: 什么时候应该使用 Agent Team Orchestrator？

**A:** 当你需要更高质量的分析，且能接受额外成本时：
- ✅ 重要架构决策 → 使用
- ✅ 关键 PR 审查 → 使用
- ✅ 安全审计 → 使用
- ✗ 简单问题、快速问答 → 用快速模式跳过

### Q: 成本增加多少？

**A:** 默认 3-agent 配置（中心化收敛）的 Token 消耗大约是单一分析的 2~3 倍。但获得的质量提升通常值得这个成本。你可以通过配置 `models: [sonnet, haiku]` 来降低成本。

**如果启用 `decentralized_debate` 集体辩论模式**，成本会进一步增加：默认配置（3 agents × 2 轮辩论）大约是中心化收敛的 6 倍。启用前会提示你确认成本。

### Q: 什么时候应该用 `decentralized_debate` 集体辩论模式？

**A:** 对于**非常重要、高风险的决策**（比如核心架构决策、安全关键部分的代码审查），值得付出额外成本来获得更高质量的共识。对于一般任务，默认中心化收敛已经足够好。

### Q: 如果所有 agent 结论都一致怎么办？

**A:** 这是好事！共识点就是最终结论，报告中明确体现即可。但这种情况较少发生，不同模型确实会有不同视角。

### Q: 如果结论分歧很大无法收敛怎么办？

**A:** Orchestrator 会如实呈现所有分歧点和各自论据，由人做最终决策。Orchestrator 是**辅助决策**，不是**代替决策**。

### Q: 可以只使用一个模型吗？

**A:** 技术上可以（配置 `models: [opus]`），但这失去了多模型认知多样性的意义。如果只想要单一模型分析，建议直接禁用 orchestrator 或使用快速模式。

### Q: 什么是"锚定效应"，为什么要避免？

**A:** 如果第一个 agent 的结论被后面的 agent 看到，后面的 agent 很容易顺着第一个的思路走，不会产生真正独立的想法。隔离就是为了避免锚定效应，保证每个视角的独立性。

### Q: 为什么要求自我质疑和盲区自评？

**A:** 这是为了让 agent 在第一阶段就主动暴露不确定性，让交叉校准更有针对性。如果一个 agent 已经认识到自己哪里不确定，其他 agent 可以重点审查这些地方。

## 开发参考

### 文件结构

```
skills/agent-team-orchestrator/
├── README.md                          # 你正在阅读的操作指导
├── SKILL.md                           # 技能定义（给 Claude 看的执行逻辑）
├── evals/
│   └── evals.json                     # 评估用例（skill-creator TDD）
└── references/
    ├── brainstorming-prompt.md        # Phase 1 头脑风暴 prompt 模板
    ├── code-review-prompt.md          # Phase 1 代码审查 prompt 模板
    ├── decentralized-debate-prompt.md # Phase 3 集体辩论轮次 prompt 模板
    └── config-schema.md               # 配置 Schema 文档
```

### 参考技能

- `superpowers:brainstorming` —— 基础头脑风暴技能（Phase 1 agent 内部调用）
- `compound-engineering:ce-review` —— 基础代码审查技能（Phase 1 agent 内部调用）
- `ecf` —— ECF 顶层编排技能

### 评估测试

项目包含 3 个预定义评估用例：

1. **ID 1**: 头脑风暴场景验证 → 验证三阶段流水线输出结构
2. **ID 2**: 代码审查场景验证 → 验证立场标注和去重
3. **ID 3**: 快速模式验证 → 验证关键词检测和跳过逻辑

运行 `skill-quality-verification` 时会自动执行这些评估。

## 许可证

作为 EasyCodingFlow 项目的一部分，遵循项目许可证。
