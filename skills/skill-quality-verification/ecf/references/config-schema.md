# Config Schema

## ecf_config.yaml Schema

### 文件位置

- 全局配置: `~/.claude/skills/ecf/ecf_config.yaml`
- 项目配置: `.claude/ecf_config.yaml` (覆盖全局)

### 配置结构

```yaml
# 默认配置（开箱即用）
defaults:
  language: zh-CN
  knowledge_base: docs/solutions/
  max_parallel_agents: 5

# 模型配置
models:
  intent_analysis: haiku      # 意图识别默认用轻量模型
  brainstorming: sonnet       # 复杂设计用中等模型
  execution: sonnet           # 执行层默认模型
  review: opus                # Review用最强模型

  # 可按场景覆盖
  by_scenario:
    bug_fix: haiku
    new_feature: opus
    refactor: sonnet

# Skill映射配置
skills:
  contract_layer:
    openspec_commands: true  # 直接使用 /opsx:* 命令
    analysis_aux: superpowers:brainstorming

  execution_layer:
    planning: superpowers:writing-plans
    execution: superpowers:executing-plans
    testing: superpowers:behavior-driven-development
    debugging: superpowers:systematic-debugging

  knowledge_layer:
    primary: compound-engineering-plugin

# 自定义模板（用户可扩展）
custom_templates: []

# 参数调优
parameters:
  timeout_per_task: 30m
  max_retry: 3
  review_rounds: 2
```

---

## 字段说明

### defaults

| 字段 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| language | string | zh-CN | 工作语言 |
| knowledge_base | path | docs/solutions/ | 知识库目录 |
| max_parallel_agents | int | 5 | 最大并行Agent数 |

### models

| 字段 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| intent_analysis | string | haiku | 意图识别模型 |
| brainstorming | string | sonnet | Brainstorming阶段模型 |
| execution | string | sonnet | 执行层默认模型 |
| review | string | opus | Code Review模型 |
| by_scenario | map | {} | 按场景覆盖模型配置 |

### parameters

| 字段 | 类型 | 默认值 | 范围 | 说明 |
|------|------|--------|------|------|
| timeout_per_task | duration | 30m | 5m-120m | 单任务超时时间 |
| max_retry | int | 3 | 1-10 | 最大重试次数 |
| review_rounds | int | 2 | 1-5 | Review轮数 |

---

## 验证规则

配置加载时自动验证：

1. **必需字段**: defaults.language, defaults.knowledge_base
2. **范围检查**: max_retry 1-10, max_parallel_agents 1-10
3. **路径检查**: knowledge_base 必须是有效路径格式
4. **模型检查**: 模型名称必须是有效模型 ID

验证失败时:
- 输出详细错误信息
- 提示用户修正配置文件
- 询问是否使用默认配置继续

---

## 配置验证失败交互

当配置验证失败时的标准交互流程：

### 错误输出格式

```
❌ 配置验证失败
━━━━━━━━━━━━━━━━━━━━
字段: [字段路径]
错误: [具体错误描述]
期望值: [有效范围/格式]
当前值: [用户提供的值]
━━━━━━━━━━━━━━━━━━━━
修正建议: [如何修复]
```

**示例**:
```
❌ 配置验证失败
━━━━━━━━━━━━━━━━━━━━
字段: parameters.max_retry
错误: 值超出允许范围
期望值: 1-10
当前值: 100
━━━━━━━━━━━━━━━━━━━━
修正建议: 将 max_retry 设置为 1-10 之间的整数
```

### 用户交互格式

```
是否使用默认配置继续执行？

1. 使用默认配置继续 (max_retry=3)
2. 手动修正配置后重试
```

### Field Path Notation

使用点号表示配置层级：
- `defaults.language`
- `defaults.knowledge_base`
- `models.intent_analysis`
- `parameters.max_retry`
- `skills.contract_layer.openspec_commands`

### 验证触发时机

每次加载配置时必须执行验证：
1. Skill 初始化时
2. 用户指定自定义配置路径时
3. 配置文件修改后重新加载时

### Red Flags - Config Validation

- 未验证直接使用用户配置
- 验证失败后未提供选项
- 错误信息无 field path
- 未提供修正建议