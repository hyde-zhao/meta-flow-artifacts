---
project_id: "meta-flow"
release_scope: "meta-flow-artifact-routing-and-eval-governance"
created_at: "2026-06-17T13:49:25+08:00"
---

# Migration

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-06-11 | host-orchestrator | 新增 workflow eval governance 迁移说明 |
| 1.1 | 2026-06-17 | host-orchestrator | 增加 process/docs 外置迁移、artifact routing metadata 和 eval runner 扩展 |

## 状态字段

| 字段 | 迁移影响 | 处理 |
|---|---|---|
| `STATE.md.artifact_routing` | 新增并成为 process health source | 记录 `routing_mode=symlink`、artifact root、project process root 和 hard-stop 策略 |
| `STATE.md.target_project_profile` | 新增可选字段 | 旧 STATE 缺失时按模板补齐，默认 `project_kind=unknown` |
| Story `verification_gate.validation_target` | 新增可选字段 | 旧 Story 缺失时按 `validation_mode` 和项目类型推断或写 N/A |
| Story `verification_gate.workflow_eval` | 新增可选字段 | 纯代码项目默认 N/A，workflow / prompt / mixed 项按 CP7 要求补证据 |

## 文件系统迁移

| 对象 | 原路径 | 新真实路径 | 源码仓库路径 |
|---|---|---|---|
| process artifacts | `<project-root>/process` 普通目录 | `<artifact-root>/process/meta-flow`，`artifact_root=../meta-flow-artifacts` | `process -> ../meta-flow-artifacts/process/meta-flow` |
| docs artifacts | `<project-root>/docs` 普通目录 | `<artifact-root>/docs/meta-flow`，`artifact_root=../meta-flow-artifacts` | 内部 docs 子路径指向 `../meta-flow-artifacts/docs/meta-flow/...` |

## 兼容性

- 旧纯代码项目无需迁移到 workflow eval。
- 旧验证报告仍可作为 legacy evidence，但新 CP7 建议补 `validation_target`。
- 外部 adapter 不自动启用。
- 如果 symlink 缺失、断链或指向错误项目，工作流必须 hard-stop，不得静默重建 `process/STATE.md`。
