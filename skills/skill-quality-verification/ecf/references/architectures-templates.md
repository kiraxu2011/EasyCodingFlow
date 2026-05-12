# Architectures Templates - 系统架构文档模板

## 模板清单

| 模板 ID | 目标位置 | 用途 |
|---------|---------|------|
| architecture-template | docs/openspec/architectures/architecture.md | 系统整体架构 |
| modules-template | docs/openspec/architectures/modules.md | 功能模块划分 |
| changes-index-template | docs/openspec/architectures/changes-index.md | 变更历史索引 |

---

## Template: architecture-template

**目标**: `docs/openspec/architectures/architecture.md`

```markdown
# 系统架构文档

## 概述
[系统整体描述、核心价值、用户群体]

## 技术栈
| 层级 | 技术选型 | 选型理由 |
|------|---------|---------|
| 前端 | ... | ... |
| 后端 | ... | ... |
| 数据层 | ... | ... |

## 架构图
[Mermaid 或 ASCII 架构图]

## 数据流
[系统数据流转路径]

## 核心设计决策
| 决策 | 背景 | 结果 |
|------|------|------|
| ... | ... | ... |

## 最后更新
YYYY-MM-DD
```

---

## Template: modules-template

**目标**: `docs/openspec/architectures/modules.md`

```markdown
# 功能模块文档

## 模块清单
| 模块名 | 职责 | 状态 | 负责人 |
|--------|------|------|--------|
| ... | ... | active/archived | ... |

## 模块依赖关系图
[Mermaid 依赖关系图]

## 模块详细说明
### [模块A]
- 职责: ...
- 核心入口: ...
- 数据流: ...
- 关键逻辑: ...
- 设计模式: ...
- 接口契约: ...
- 依赖: ...
- 设计文档: [链接](../../plans/YYYY-MM-DD-xxx-design/)
```

---

## Template: changes-index-template

**目标**: `docs/openspec/architectures/changes-index.md`

```markdown
# 变更索引

## 变更记录
| 日期 | 模块 | 变更类型 | 影响范围 | 设计文档链接 |
|------|------|---------|---------|-------------|
| YYYY-MM-DD | init | 初始化 | 项目初始化 | - |

## 统计
- 总变更次数: 1
- 最近更新: YYYY-MM-DD
```

---

## 存量项目生成规则

当 ecf-init 检测到存量项目时，模板内容需基于代码分析生成：

### architecture.md 生成规则

| 章节 | 生成方式 |
|------|---------|
| 技术栈 | 扫描 package.json/Gemfile/pom.xml 等 |
| 架构图 | 分析目录结构生成 Mermaid 图 |
| 数据流 | 分析核心模块入口和数据流转路径 |
| 设计决策 | 扫描 CLAUDE.md/README.md 提取关键决策 |

### modules.md 生成规则

| 章节 | 生成方式 |
|------|---------|
| 模块清单 | 分析 src/ 目录结构 |
| 依赖关系图 | 分析 import/require 关系 |
| 模块详细说明 | 分析每个模块的核心入口、接口、关键逻辑 |

---

## 更新触发规则

变更类型判断规则：

| 变更信号 | 触发更新 |
|---------|---------|
| 技术栈变更 | architecture.md 技术栈表 |
| 新增目录 | modules.md 新增模块条目 |
| 删除目录 | modules.md 模块状态改为 archived |
| 核心入口变更 | modules.md 模块入口更新 |
| 数据流变更 | architecture.md 数据流图 |
| 接口契约变更 | modules.md 接口契约 |
| 任何变更 | changes-index.md 追加条目 |