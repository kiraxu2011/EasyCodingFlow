# EasyCodingFlow

统一的 Agent 协作编排技能，整合 Superpowers、OpenSpec、Compound Engineering 为四层架构。

## 快速开始

### 前置条件

- [Claude Code CLI](https://github.com/anthropics/claude-code) - AI 编程助手
- Git - 版本控制工具

### 安装步骤

**1. 克隆项目**

```bash
git clone https://github.com/kiraxu2011/EasyCodingFlow
cd EasyCodingFlow
```

**2. 安装技能**

```bash
# 复制到 Claude Code 技能目录
mkdir -p ~/.claude/skills

cp -r skills/ecf ~/.claude/skills/
cp -r skills/ecf-init ~/.claude/skills/
cp -r skills/ecf-execute ~/.claude/skills/
cp -r skills/ecf-verify ~/.claude/skills/
```

**3. 初始化项目**

在你的项目目录中运行：

```bash
/ecf-init
```

这将创建：
- `docs/solutions/` - 知识库
- `docs/architectures/` - 架构文档
- `.claude/ecf_config.yaml` - 项目配置

### 验证安装

```bash
ls ~/.claude/skills/
# 应显示: ecf/ ecf-init/ ecf-execute/ ecf-verify/
```

## 工作原理

### 四层架构

```
┌─────────────────────────────────────────┐
│ Layer 0: 编排层 (Orchestration)          │
│ 意图识别 → 流程路由 → 团队组建 → 进度监控  │
├─────────────────────────────────────────┤
│ Layer 1: 规范契约层 (Contract)           │
│ OpenSpec (propose → apply → archive)     │
│ Brainstorming                            │
├─────────────────────────────────────────┤
│ Layer 2: 执行层 (Execution)              │
│ Writing-Plans → Executing-Plans → Test   │
├─────────────────────────────────────────┤
│ Layer 2.5: 验证层 (Verification)         │
│ Consistency-Verification                 │
├─────────────────────────────────────────┤
│ Layer 3: 知识沉淀层 (Knowledge)          │
│ Compound Engineering                     │
└─────────────────────────────────────────┘
```

**核心流程**: 每个请求首先经过意图识别，自动路由到对应工作流，按层顺序执行。

### 支持场景

| 场景 | 关键词 | 工作流 |
|------|--------|--------|
| 新需求开发 | 开发、新功能、实现 | OpenSpec → Brainstorming → Plans → Execute → Verify → Archive |
| 增量开发 | 扩展、迭代 | OpenSpec → Execute → Verify → Archive |
| Bug 修复 | bug、报错、修复 | Debugging → Fix → Verify |
| 代码重构 | 重构、优化结构 | Brainstorming → Plans → Execute → Verify |
| Code Review | review、审查 | Review → Compound |
| 文档更新 | 文档、readme | 直接执行 |
| 知识检索 | 之前、类似 | Search docs/solutions/ |

## 功能一览

### 核心功能

| 功能 | 描述 |
|------|------|
| **意图识别** | 自动识别请求类型，置信度 ≥0.7 直接路由 |
| **Skill Detection** | 检测技能相关任务，强制 TDD 流程 |
| **并发执行** | DAG 调度，>6 任务使用 Agent Team 并行 |
| **一致性验证** | spec↔design↔code↔tests 四层一致性检查 |
| **知识沉淀** | 自动归档解决方案，支持历史检索 |
| **产物转换** | OpenSpec 产物自动转换为执行层输入 |

### 关键技能

| 技能 | 命令 | 功能 |
|------|------|------|
| ecf | `/ecf` | 主入口，意图识别与流程编排 |
| ecf-init | `/ecf-init` | 初始化项目结构 |
| ecf-execute | `/ecf-execute` | 并发执行计划 |
| ecf-verify | `/ecf-verify` | 一致性验证 |

## 使用方式

### 基本用法

直接向 Claude 描述任务，技能会自动识别并路由：

```
用户: "开发一个用户认证系统"
Claude 自动执行: 意图识别 → OpenSpec → Brainstorming → Plans → Execute → Verify → Archive

用户: "修复登录页面的 bug"
Claude 自动执行: Debugging → Fix → Verify

用户: "之前怎么处理过认证问题"
Claude 自动执行: 知识检索 docs/solutions/
```

### 配置

配置文件：`.claude/ecf_config.yaml`

```yaml
defaults:
  language: zh-CN
  max_parallel_agents: 5

models:
  brainstorming: sonnet
  execution: sonnet
  review: opus
```

### 项目结构

```
your-project/
├── docs/
│   ├── solutions/           # 知识库（必需）
│   ├── architectures/       # 架构文档
│   └── plans/               # 设计计划
├── openspec/                # OpenSpec CLI 管理
│   ├── changes/             # 变更目录
│   └── changes/archive/     # 归档目录
└── .claude/
    └── ecf_config.yaml      # 项目配置
```

## 参与贡献

欢迎贡献代码、报告问题或提出建议。

### 开发流程

1. Fork 项目
2. 创建特性分支 (`git checkout -b feature/amazing-feature`)
3. 提交更改 (`git commit -m 'feat: add amazing feature'`)
4. 推送分支 (`git push origin feature/amazing-feature`)
5. 创建 Pull Request

### 技能修改规范

技能修改必须遵循 TDD 流程：

```
/opsx:propose → skill-creator → skill-quality-verification → /opsx:archive → ce:compound
```

**禁止**直接编辑 SKILL.md 文件。

详见：[Skills开发工作流规范](docs/solutions/workflow-issues/skills-development-workflow-enforcement-2026-05-04.md)

## 许可证

MIT License

Copyright (c) 2026 xuguoliang3

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

---

**注意**: 此技能需要与 Claude Code CLI 配合使用。
