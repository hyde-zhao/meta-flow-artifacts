---
status: ready
version: "1.0"
release_artifact_profile: compact
release_decision: READY_WITH_RISK
---

# CR066 Migration

## 1. 迁移结论

| 项目 | 内容 |
|---|---|
| 是否需要迁移 | no |
| 是否自动迁移 | N/A |
| 是否保留兼容路径 | N/A |
| 是否可逆 | yes |

## 2. 兼容性判断表

| 对象 | 是否变化 | 兼容性 | 需要迁移 | 验证方式 | 回滚方式 |
|---|---|---|---|---|---|
| `STATE.md` schema | no | compatible | no | CR tracking consistency check。 | 回填状态即可。 |
| 模板字段 | no | N/A | no | N/A | N/A |
| 配置格式 | no | N/A | no | N/A | N/A |
| 安装路径 | no | N/A | no | N/A | N/A |
| Agent frontmatter | no | N/A | no | N/A | N/A |
| Skill 输出格式 | no | N/A | no | N/A | N/A |
| 命令参数 | no | N/A | no | N/A | N/A |
| 数据存储结构 | no | N/A | no | N/A | N/A |

## 3. 迁移步骤

| Step | 操作 | 前置条件 | 验证 | 回退 |
|---|---|---|---|---|
| 1 | N/A | CR066 不做 physical move、bulk import rewrite 或 Git publication。 | CP8 checklist。 | N/A |

## 4. N/A 说明

| 项目 | 原因 | 后续触发条件 |
|---|---|---|
| 文件 / 数据迁移 | CR066 是 publication strategy gate，不迁移文件或数据。 | 用户启动 physical move / NAS / lake CR。 |
| Git publication migration | CR066 不执行 Git 写动作。 | CR067 execution gate approved。 |
