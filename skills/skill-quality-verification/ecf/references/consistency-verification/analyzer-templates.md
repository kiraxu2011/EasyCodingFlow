# Analyzer Templates

Sub-agent prompt templates for parallel consistency analysis.

## Common Requirements (MANDATORY)

**Every analyzer MUST:**
- Create traceability matrix (not just presence checks)
- Perform bidirectional verification (both directions)
- Use semantic analysis (not keyword matching)
- Output structured evidence with **line numbers AND content excerpt**
- Provide **meaningful fix options** (not superficial like "fix/don't fix")

**Evidence Quality Requirements:**
- Line numbers or section anchors required
- Brief content excerpt proving match/mismatch
- Confidence level: 高/中/低

**Status Options:**
- ✅ Consistent: Full match with evidence
- ❌ Missing: Not found with search proof
- ⚠️ Inconsistent: Partial match or mismatch
- 🔍 Ambiguous: Cannot determine with confidence (requires explicit flagging)

**Output format:**
```
✅ Consistent: [list with sources + line numbers]
❌ Missing: [list with details + search evidence]
⚠️ Inconsistent: [list with gaps + content excerpts]
🔍 Ambiguous: [list with reason + needs clarification]
💡 Recommended Fixes: [meaningful options describing WHAT to change]
```

---

## Spec-Design Analyzer

```markdown
你是 Spec-Design 一致性分析器。

输入:
- bdd-specs.md: [内容]
- architecture.md: [内容]
- _index.md: [内容]

分析任务:
1. 从 spec 提取所有 Feature/Scenario
2. 创建 traceability matrix: spec场景 → design章节
3. 双向验证:
   - spec→design: 每个场景是否在 design 有对应设计
   - design→spec: design 是否有超出 spec 的内容（over-engineering）
4. 语义分析: 术语一致性，需求完整性

输出:
- Traceability Matrix (场景 → 章节 → 状态)
- Missing Coverage (spec 有，design 无)
- Over-engineering (design 有，spec 无)
- Term Inconsistencies (概念术语不一致)

每个不一致项必须包含:
- 来源位置 (文件 + 章节/行号)
- 影响等级 (高/中/低)
- 修复建议 (至少 2 个选项)
```

---

## Design-Code Analyzer

```markdown
你是 Design-Code 一致性分析器。

输入:
- architecture.md: [内容]
- 代码目录: src/**/*

分析任务:
1. 解析 architecture.md 定义的所有模块/组件/接口
2. 搜索代码实现:
   - Glob 搜索目录结构
   - Grep 搜索接口定义
3. 创建 traceability matrix: design组件 → 代码文件
4. 双向验证:
   - design→code: 设计的组件是否实现
   - code→design: 代码是否有未设计的组件
5. 接口签名比对: 参数名、类型、返回值

输出:
- Matched Components (组件 → 文件路径)
- Missing Implementations (design 有，code 无)
- Undocumented Code (code 有，design 无)
- Interface Mismatches (签名差异详情)
```

---

## Spec-Test Analyzer

```markdown
你是 Spec-Test 一致性分析器。

输入:
- bdd-specs.md: [内容]
- 测试文件: tests/**/*_test.*

分析任务:
1. 解析所有 Gherkin Scenario
2. 搜索测试用例:
   - Glob 搜索测试文件
   - Grep 搜索测试函数
3. 创建 traceability matrix: Scenario → Test Function
4. 双向验证:
   - spec→tests: 每个场景是否有测试
   - tests→spec: 测试是否有对应场景
5. 测试质量检查: 每个测试是否验证 Then 条件

输出:
- Covered Scenarios (Scenario → Test → 状态)
- Uncovered Scenarios (无测试)
- Extra Tests (测试无对应场景)
- Incomplete Tests (未验证所有条件)
```

---

## Architecture-Doc Analyzer

```markdown
你是 Architecture-Doc 一致性分析器。

输入:
- git diff: [当前变更差异]
- docs/openspec/architectures/architecture.md: [现有内容]
- docs/openspec/architectures/modules.md: [现有内容]
- docs/openspec/architectures/changes-index.md: [现有内容]

分析任务:
1. 解析 git diff 获取变更文件列表
2. 分析变更类型（技术栈变更、模块变更、接口变更等）
3. 根据变更类型确定更新目标文档和章节
4. 验证文档完整性

输出:
- 变更类型列表（类型、文件、影响范围、等级）
- 更新内容建议（目标文件、章节、建议内容）
- 变更索引条目（日期、模块、变更类型、影响范围、链接）
```

完整模板见 [architecture-doc-analyzer.md](./architecture-doc-analyzer.md)

---

## OpenSpec Traceback (Optional)

如果 `docs/openspec/` 存在:

```markdown
OpenSpec 追溯验证:

输入:
- docs/openspec/proposals/*/spec.md
- docs/plans/*-design/bdd-specs.md

分析任务:
1. 比较 OpenSpec spec.md 与 superpowers bdd-specs.md
2. 确保 OpenSpec 的变更意图在 superpowers 中保留
3. 检查转换过程是否丢失关键信息

输出:
- Intent Preservation (意图是否保留)
- Lost Information (丢失的内容)
- Conversion Quality Assessment
`````

---

## OpenSpec Traceback Rules

**如果 `docs/openspec/` 存在，追溯验证为 MUST:**
- 必须执行 Intent Traceability Matrix
- 报告注明 "OpenSpec 追溯: 已执行"

**仅在不存在时可跳过:**
- 报告注明 "OpenSpec 追溯: 跳过（源文档不存在）"

---

## Fix Options Quality

**修复建议必须具体:**
- ❌ Bad: "[A] 修复 / [B] 不修复"
- ✅ Good: "[A] 在 architecture.md 添加 SessionModule 章节 / [B] 移除 spec SessionModule Scenario"

---

## Skip Limits

- 最大 20% 不一致项可跳过
- 高影响项不可跳过
- 跳过必须记录原因
