# Dependency Check Reference

## Overview

Pre-flight Check Step 3 检查四大依赖：
1. OpenSpec Skills (项目级)
2. Compound Engineering Plugin (用户级)
3. Superpowers@frad-dotclaude (用户级)
4. skill-creator (用户级，Skills Development 场景必需)

## Detection Commands

### OpenSpec Skills

检查项目级 `.claude/skills/` 目录下的 OpenSpec skills：

```bash
OPENSPEC_SKILLS=("openspec-explore" "openspec-propose" "openspec-apply-change" "openspec-archive-change")
MISSING_OPSX=()

for skill in "${OPENSPEC_SKILLS[@]}"; do
    if [[ ! -d ".claude/skills/${skill}" ]]; then
        MISSING_OPSX+=("$skill")
    fi
done

if [[ ${#MISSING_OPSX[@]} -eq 0 ]]; then
    echo "✅ OpenSpec Skills: 已安装 (opsx:explore, opsx:propose, opsx:apply, opsx:archive)"
else
    echo "⚠️ OpenSpec Skills: 缺失 ${#MISSING_OPSX[@]} 个 - ${MISSING_OPSX[*]}"
fi
```

**关键检查点**:

| Skill | 检查路径 | 用途 |
|-------|----------|------|
| openspec-explore | `.claude/skills/openspec-explore/` | Contract Layer - 需求探索 |
| openspec-propose | `.claude/skills/openspec-propose/` | Contract Layer - 提案创建 |
| openspec-apply-change | `.claude/skills/openspec-apply-change/` | Execution Layer - 变更应用 |
| openspec-archive-change | `.claude/skills/openspec-archive-change/` | Knowledge Layer - 变更归档 |

### Compound Engineering Plugin

检查用户级插件缓存目录：

```bash
if ls ~/.claude/plugins/cache/compound-engineering-plugin/ 2>/dev/null | grep -q .; then
    echo "✅ Compound Engineering: 已安装"
else
    echo "⚠️ Compound Engineering: 未安装"
fi
```

**关键检查点**:

| 检查项 | 路径/命令 | 说明 |
|--------|-----------|------|
| 插件缓存目录 | `~/.claude/plugins/cache/compound-engineering-plugin/` | 存在即可 |
| Skill 注册 | `ce:compound` 调用测试 | 运行时验证 |

### Superpowers@frad-dotclaude

检查环境变量和脚本存在性：

```bash
# Step 1: 检查 CLAUDE_PLUGIN_ROOT 是否已设置
if [[ -n "${CLAUDE_PLUGIN_ROOT:-}" && -f "$CLAUDE_PLUGIN_ROOT/scripts/setup-superpower-loop.sh" ]]; then
    echo "✅ Superpowers@frad-dotclaude: 已就绪 ($CLAUDE_PLUGIN_ROOT)"
    # 继续正常流程
else
    # Step 2: 尝试自动检测并设置
    export CLAUDE_PLUGIN_ROOT="${CLAUDE_PLUGIN_ROOT:-$(ls -d ~/.claude/plugins/marketplaces/frad-dotclaude/superpowers 2>/dev/null || ls -d ~/.claude/plugins/cache/frad-dotclaude/superpowers/*/ 2>/dev/null | head -1 || echo "")}"
    
    if [[ -n "$CLAUDE_PLUGIN_ROOT" && -f "$CLAUDE_PLUGIN_ROOT/scripts/setup-superpower-loop.sh" ]]; then
        echo "✅ Superpowers@frad-dotclaude: 自动检测并设置 ($CLAUDE_PLUGIN_ROOT)"
    elif ls ~/.claude/plugins/cache/claude-plugins-official/superpowers/*/ 2>/dev/null | grep -q .; then
        echo "⚠️ Superpowers: frad-dotclaude 未找到，将使用官方版本 fallback"
    else
        echo "❌ Superpowers: 未安装 (Critical)"
    fi
fi
```

**关键检查点**:

| 检查项 | 路径/命令 | 用途 |
|--------|-----------|------|
| CLAUDE_PLUGIN_ROOT | 环境变量 | Loop 脚本路径 |
| setup-superpower-loop.sh | `$CLAUDE_PLUGIN_ROOT/scripts/setup-superpower-loop.sh` | Superpower Loop 启动 |
| Marketplace 安装 | `~/.claude/plugins/marketplaces/frad-dotclaude/superpowers` | 优先检查 |
| Cache 安装 | `~/.claude/plugins/cache/frad-dotclaude/superpowers/*/` | 备用检查 |
| 官方版本 | `~/.claude/plugins/cache/claude-plugins-official/superpowers/*/` | Fallback |

### skill-creator

检查 Skills 目录和插件缓存目录：

```bash
SC_INSTALLED=false
if [[ -d ~/.claude/skills/skill-creator ]]; then
  SC_INSTALLED=true
fi
if [[ -d ~/.claude/plugins/cache/claude-plugins-official/skill-creator ]]; then
  SC_INSTALLED=true
fi

if $SC_INSTALLED; then
    echo "✅ skill-creator: 已安装"
else
    echo "⚠️ skill-creator: 未安装"
fi
```

**自动安装**（`--auto-install` 模式会自动执行）：
```bash
# 1. 检查/添加 marketplace
claude plugin marketplace list 2>/dev/null | grep -q claude-plugins-official || claude plugin marketplace add claude-plugins-official

# 2. 更新 marketplace 获取最新插件列表
claude plugin marketplace update claude-plugins-official

# 3. 安装 skill-creator
claude plugin install skill-creator@claude-plugins-official
```

**关键检查点**:

| 检查项 | 路径/命令 | 用途 |
|--------|-----------|------|
| Skills 目录 | `~/.claude/skills/skill-creator/` | 已安装到 skills 目录 |
| 缓存目录 | `~/.claude/plugins/cache/claude-plugins-official/skill-creator/` | 缓存安装位置 |
| marketplace | `claude plugin marketplace list` | 检查 claude-plugins-official 是否存在 |

## Fallback Matrix

| 依赖 | 状态 | Fallback 策略 | 影响 | 用户提示 |
|------|------|---------------|------|----------|
| **OpenSpec** | 未安装 | Degraded: 跳过 OpenSpec 流程，直接使用 Brainstorming | Contract Layer 功能受限，无法使用 `/opsx:*` | "⚠️ OpenSpec Skills 未安装，Contract Layer 将使用 Brainstorming 替代" |
| **OpenSpec** | 部分安装 | 仅使用已安装的 skills，缺失功能使用 Brainstorming 替代 | 部分 OpenSpec 功能可用 | "⚠️ OpenSpec Skills 部分缺失: [list]，相关功能将使用 Brainstorming" |
| **CE** | 未安装 | Degraded: 直接写入简化 solution 文件到 `docs/solutions/` | 知识沉淀功能降级 | "⚠️ Compound Engineering 未安装，知识沉淀使用简化模式" |
| **Superpowers** | frad 未安装，官方可用 | 使用官方版本 (无 Superpower Loop) | 无 Loop 协调，某些高级并发功能不同 | "⚠️ 使用 Superpowers 官方版本 (frad-dotclaude 未安装)" |
| **Superpowers** | 完全未安装 | **Critical**: 无法执行，提示用户安装 | Execution Layer 不可用 | "❌ Superpowers 未安装，请先安装 Superpowers skills" |
| **skill-creator** | 未安装 | Degraded: 仅 skill_development 场景受限，无法使用 skill-creator TDD 流程 | 仅技能开发工作流受影响 | "⚠️ skill-creator 未安装，Skills 开发工作流将受限。运行 ecf-init --auto-install 自动安装" |

## Dependency Status Summary Format

检查完成后输出汇总：

```
🔍 Dependency Check Summary
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
OpenSpec Skills:      ✅ 已安装 / ⚠️ 部分缺失 [list]
Compound Engineering: ✅ 已安装 / ⚠️ 未安装
Superpowers:          ✅ frad-dotclaude / ⚠️ 官方版本 / ❌ 未安装
skill-creator:        ✅ 已安装 / ⚠️ 未安装
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[如有缺失依赖，显示建议操作]
```

## Red Flags

- 跳过依赖检查直接调用 skill
- OpenSpec 缺失但未 fallback 到 Brainstorming
- CE 缺失但未使用简化知识沉淀
- 调用 superpowers skill 时卡住但未检查 CLAUDE_PLUGIN_ROOT
- Superpowers 完全未安装但未阻止流程
- skill-creator 未安装但未提示安装（仅 skill_development 场景必需）
- skill-creator 安装失败时未更新 marketplace

## Related

- [superpowers-environment-check.md](superpowers-environment-check.md) - Superpowers 专项检查详细流程
- [initialization.md](initialization.md) - 项目初始化检查
- [knowledge-writing.md](knowledge-writing.md) - CE Plugin 调用流程