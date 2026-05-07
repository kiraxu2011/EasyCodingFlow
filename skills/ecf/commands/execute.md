---
description: Execute plans with agent teams - DEPRECATED, use ecf-execute skill instead
---

# Execute Command

**⚠️ 此命令已整改为独立技能**

原 `/ecf:execute` 命令已迁移为独立技能 `/ecf-execute`。

## 迁移说明

| 旧命令 | 新技能 |
|--------|--------|
| `/ecf:execute` | `/ecf-execute` |

## 使用方式

调用新技能:
```
/ecf-execute [plan-folder-path]
```

或通过 Skill tool:
```
Skill("ecf-execute")
```

## 技能位置

`skills/ecf-execute/SKILL.md`

## 迁移原因

Commands 目录下的命令格式不被 skill 系统正确解析，已整改为标准 skill 目录格式。

## 参考

- [ecf-execute/SKILL.md](../ecf-execute/SKILL.md) - 新技能实现
- [ecf-init/SKILL.md](../ecf-init/SKILL.md) - 已整改的 init 技能