# Intent Keywords Mapping

## 场景关键词映射表

### 技能开发
- skill
- skills
- 技能
- SKILL.md
- 技能开发
- 优化技能
- 技能优化
- 修改技能
- 技能修复
- 技能bug
- 技能评估
- skill-creator
- skill-quality-verification

### 新需求开发
- 开发
- 新功能
- 实现
- 创建
- 新增
- 设计
- 构建
- 独立分析
- 多方案对比
- 头脑风暴
- 方案碰撞

### 增量开发
- 扩展
- 迭代
- 增强
- 优化
- 添加
- 升级
- 完善

### Bug修复
- bug
- 报错
- 失败
- 修复
- 错误
- 异常
- crash
- broken
- 不工作

### Code Review
- review
- 审查
- 检查
- 评审
- 审阅
- 检视
- 多视角
- 交叉审查
- 多模型审查

### 代码重构
- 重构
- 优化结构
- 模块化
- 重写
- 整理
- 清理
- 重组织

### 文档更新
- 文档
- readme
- 说明
- 注释
- 文档化
- 记录

### 用例补齐
- 测试
- 用例
- test
- coverage
- 测试覆盖
- 补齐测试

### 知识检索
- 之前
- 曾经
- 类似
- 历史
- 经验
- 怎么处理
- 查看历史
- 有没有

---

## 关键词匹配优先级

**优先级规则**（歧义处理）：

1. **最高优先级**: 如果匹配到 **skill_development** 场景的**任何关键词** → **强制优先分类为 skill_development**
   - **规则**: 即使请求同时包含 `bug`, `fix`, `修复`, `review`, `优化` 等其他场景关键词，**仍然强制分类为 skill_development**
   - 示例: "修复 ecf skill 中的 bug" → skill_development（**不是** bug_fix）
   - 示例: "优化 skills 技能" → skill_development（**不是** refactor）
   - 示例: "修改 SKILL.md 格式" → skill_development
   - 原因: 技能开发有特殊工作流要求（skill-creator + skill-quality-verification + archive + eval 强制验证），分类错误会导致跳过强制 eval 验证环节，违反项目核心规则

2. 如果不包含 skill_development 关键词 → 按 confidence 最高选择场景

---

## 置信度阈值与处理方式

- **>= 0.7**: 关键词匹配成功，当前 agent 直接路由到工作流
- **< 0.7**: 当前 agent 基于上下文深度分析意图（不发起额外 API 调用）

**重要**: 意图识别始终由当前 Claude session 执行。当前 agent 已具备完整的意图理解能力，无需调用外部 LLM API 或启动独立 Agent。

---

## 匹配算法

1. 应用优先级规则：skill_development 关键词优先匹配
2. 扫描用户输入中的关键词
3. 统计每个场景的匹配关键词数量
4. 计算 confidence = 匹配数 / 该场景关键词总数
5. 选择 confidence 最高的场景
6. 若最高 confidence < 0.7，当前 agent 深度分析意图