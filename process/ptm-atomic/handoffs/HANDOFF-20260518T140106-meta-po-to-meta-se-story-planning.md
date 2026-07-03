---
handoff_id: "HANDOFF-20260518T140106-meta-po-to-meta-se-story-planning"
from_agent: "meta-po"
to_agent: "meta-se"
status: "agent_completed"
created_at: "2026-05-18T14:01:06+0800"
workflow_phase: "story-planning"
workflow_id: "atomic-ops-ngfw-install-config-chain"
change_id: ""
story_id: ""
wave_id: ""
dispatch:
  required: true
  mode: "subagent"
  platform: "codex"
  agent_role: "meta-se"
  agent_path: ".codex/agents/meta-se.toml"
  tool_name: "spawn_agent"
  agent_id: "019e39ac-5f21-7063-a912-4afc2b0eb45f"
  agent_name: "Archimedes"
  thread_id: "019e39ac-5f21-7063-a912-4afc2b0eb45f"
  spawned_at: "2026-05-18T14:01:06+0800"
  resumed_at: ""
  completed_at: "2026-05-18T14:35:00+0800"
  evidence: "spawn_agent returned agent_id=019e39ac-5f21-7063-a912-4afc2b0eb45f nickname=Archimedes; actual tool agent_type=default, acting_role=meta-se"
  fallback_reason: ""
  approved_by: ""
  approved_at: ""
---

# Handoff: meta-po -> meta-se Story Planning

## 目标

在 CP3 HLD Review 已人工批准后，由 `meta-se` 执行 `story-planning` 阶段，输出 ADR、平台/交付面边界说明、Story Backlog、Development Plan、Story 卡片、Story 状态汇总和 CP4 自动预检。

## 输入上下文

| 路径 | 用途 |
|---|---|
| `AGENTS.md` | 当前仓库工作流、交付面和门控规则。 |
| `.codex/agents/meta-se.toml` | `meta-se` 角色职责与 story-planning 输出契约。 |
| `process/STATE.md` | 当前阶段为 `story-planning`，CP4 前 LLD 仍阻塞。 |
| `process/HLD.md` | v1.1 confirmed HLD，推荐方案 B。 |
| `process/USE-CASES.md` | 已确认场景基线。 |
| `process/REQUIREMENTS.md` | 已确认需求基线。 |
| `checkpoints/CP3-HLD-REVIEW.md` | CP3 人工批准证据。 |
| `README.md`、`docs/`、`schemas/`、`src/atomic_ops/`、`scripts/` | 仓库事实核验来源。 |

## 必须输出

| 产物 | 路径 |
|---|---|
| ADR | `process/ARCHITECTURE-DECISION.md` |
| 平台 / 交付面边界说明 | `process/PLATFORM-INSTALL-SPEC.md` |
| Story Backlog | `process/STORY-BACKLOG.md` |
| 开发计划 | `process/DEVELOPMENT-PLAN.yaml` |
| Story 状态汇总 | `process/STORY-STATUS.md` |
| Story 卡片 | `process/stories/STORY-*.md` |
| CP4 自动预检 | `process/checks/CP4-STORY-DAG-PARALLEL-SAFETY.md` |

## 禁止事项

- 不得创建 `process/stories/STORY-*-LLD.md`。
- 不得修改产品交付物目录。
- 不得启动或模拟 `meta-dev` / `meta-qa`。
- 不得把 `.input/` 内容直接复制进当前仓库交付面。

## CP4 关键决策下限

- 方案 B 已获 CP3 接受：冻结 `schema_version="1.1"` 与字段族 `risk`、`credential_ref`、`session_ref`、`state_ref`、`gate`、`verification`、`batch`。
- capacity 参考内容只抽象为 atomic 原生配置域契约，不整体搬运；首批至少覆盖 10 个配置域。
- 多设备批次配置需要独立 Story 或独立 TASK-ID，优先独立 Story。
- 命令命名规范、参数校验、安全 gate 与不新增真实设备执行命令必须进入 Story 验收。
- CP4 自动预检和人工确认通过前，不得拉起 LLD 设计子 agent。

## 完成结果

| 项目 | 结果 | 证据 |
|---|---|---|
| Story planning 产物 | 已完成 | `process/ARCHITECTURE-DECISION.md`、`process/PLATFORM-INSTALL-SPEC.md`、`process/STORY-BACKLOG.md`、`process/DEVELOPMENT-PLAN.yaml`、`process/STORY-STATUS.md`、`process/stories/STORY-*.md` |
| CP4 自动预检 | PASS | `process/checks/CP4-STORY-DAG-PARALLEL-SAFETY.md` |
| Story 数 | 6 | STORY-001 到 STORY-006 |
| 推荐首批 LLD | `STORY-001`、`STORY-002`、`STORY-003` | `process/DEVELOPMENT-PLAN.yaml` `first_lld_design_batch_recommendation` |
| LLD 文件 | 未生成 | `process/stories/` 无 `STORY-*-LLD.md` |
| 产品交付面 | 未修改 | meta-se 仅写入 `process/` 运行态产物 |
