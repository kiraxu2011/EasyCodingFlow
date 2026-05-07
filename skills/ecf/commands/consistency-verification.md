---
description: Execute consistency verification - DEPRECATED, use ecf-verify skill instead
---

# Consistency Verification Command

**⚠️ 此命令已整改为独立技能**

原 `/ecf:consistency-verification` 命令已迁移为独立技能 `/ecf-verify`。

## 迁移说明

| 旧命令 | 新技能 |
|--------|--------|
| `/ecf:consistency-verification` | `/ecf-verify` |

## 使用方式

调用新技能:
```
/ecf-verify [design-folder-path]
```

或通过 Skill tool:
```
Skill("ecf-verify")
```

## 技能位置

`skills/ecf-verify/SKILL.md`

## 迁移原因

Commands 目录下的命令格式不被 skill 系统正确解析，已整改为标准 skill 目录格式。

## 参考

- [ecf-verify/SKILL.md](../ecf-verify/SKILL.md) - 新技能实现
- [ecf-init/SKILL.md](../ecf-init/SKILL.md) - 已整改的 init 技能