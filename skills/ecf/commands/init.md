---
description: Initialize ecf project structure - DEPRECATED, use ecf-init skill instead
argument-hint: [--force] [--minimal]
---

# Agent-Teams Init Command

**⚠️ 此命令已整改为独立技能**

原 `/ecf:init` 命令已迁移为独立技能 `/ecf-init`。

## 迁移说明

| 旧命令 | 新技能 |
|--------|--------|
| `/ecf:init` | `/ecf-init` |

## 使用方式

调用新技能:
```
/ecf-init [--force] [--minimal]
```

或通过 Skill tool:
```
Skill("ecf-init")
```

## 技能位置

`skills/ecf-init/SKILL.md`

## 参数说明

- `--force`: 强制重新初始化，备份现有文件到 `.backup/`
- `--minimal`: 最小模式，仅创建目录不部署模板

## 迁移原因

Commands 目录下的命令格式不被 skill 系统正确解析，已整改为标准 skill 目录格式。

## 参考

- [ecf-init/SKILL.md](../ecf-init/SKILL.md) - 新技能实现