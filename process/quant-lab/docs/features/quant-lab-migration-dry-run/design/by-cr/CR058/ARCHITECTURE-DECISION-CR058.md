---
status: "draft-for-review"
version: "0.1"
change_id: "CR-058"
created_by: "host-orchestrator"
created_at: "2026-06-14T13:59:00+08:00"
---

# Architecture Decision CR058

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 0.1 | 2026-06-14 | host-orchestrator | 新增 CR058 ADR 草案。 |

## ADR-CR058-001: CR058 不继承 CR053 真实迁移授权

| 字段 | 内容 |
|---|---|
| 状态 | proposed |
| 决策 | CR053 `READY_WITH_RISK` 只作为静态 inventory / dry-run 输入，不授权 CR058 执行真实动作。 |
| 理由 | CR053 CP8 明确不授权 repo-local move、NAS、lake、trading、credential、git remote 或自动启动 CR058。 |
| 后果 | CR058 必须重新走 CP2 / CP3 / CP5 / CP8 和后续 runtime_authorization。 |
| 回退 | 若用户要求从 CR053 继承授权，必须重开 CR053 CP8 决策，不得静默继承。 |

## ADR-CR058-002: Preserve-audit 优先于机械替换

| 字段 | 内容 |
|---|---|
| 状态 | proposed |
| 决策 | `process/**`、checkpoint、handoff、历史 Story、DEV-LOG、legacy checkpoints 默认 `preserve-audit`。 |
| 理由 | 历史记录中的 `local_backtest` 是当时事实；机械替换会破坏审计语境。 |
| 后果 | CR058 candidate list 只能包含 future-facing docs 或人工明确允许对象。 |
| 回退 | 如需改写历史对象，必须单独列出 allowlist 行、理由、reviewer 和 rollback_ref。 |

## ADR-CR058-003: rollback_ref 三件套为真实执行前置

| 字段 | 内容 |
|---|---|
| 状态 | proposed |
| 决策 | 真实 move / rewrite 前必须具备 `pre_cr058_commit`、`pre_cr058_git_bundle`、`pre_reference_rewrite_manifest`。 |
| 理由 | repo-local relayout 虽是本地仓库内动作，但会影响大量路径和审计引用，必须可回滚。 |
| 后果 | 缺任一 ref 时 CR058 只能停留在设计 / dry-run。 |
| 回退 | 用户可显式豁免 bundle，但必须形成 `risk_acceptance` 决策项。 |

## ADR-CR058-004: 当前批次不新增 mover 脚本

| 字段 | 内容 |
|---|---|
| 状态 | proposed |
| 决策 | CP5 前只输出 design、Story technical-note、candidate schema 和 no-op gate，不新增或运行 mover/rewrite 脚本。 |
| 理由 | 用户本轮明确禁止真实 move / rename / delete 和路径替换；脚本实现容易被误当作执行授权。 |
| 后果 | 后续如需脚本，必须在 CP5 通过后另行 Story / CP6，并仍默认 no-op / dry-run。 |
| 回退 | 若用户只要求人工清单，不需要脚本 Story；若用户要求自动化，新增 implementation DQ。 |

