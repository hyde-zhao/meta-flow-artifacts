---
handoff_id: "HO-CR051-CP4-META-SE-FEAT12-RETURN"
from_agent: "meta-se-critical"
to_agent: "host-orchestrator"
workflow_id: "meta-flow-self-dev"
change_id: "CR-051"
stage: "story-planning"
status: "completed"
completed_at: "2026-07-18T07:25:00Z"
context_ref: "process/context/CP4-CR051-STORY-PLANNING-CONTEXT.yaml"
source_handoff: "process/handoffs/CR051-CP4-META-SE-FEAT12.md"
write_lane: "feature-packs-01-02"
---

# CR-051 CP4 FEAT-AW-01/02 Return

## 结论

FEAT-AW-01 与 FEAT-AW-02 的 Feature DESIGN / TEST-PLAN / TASKS 已完成，可供 core planning lane 汇总 FEATURE-DESIGN-MATRIX、DEVELOPMENT-PLAN、Story 卡和 CP4 自动预检。两个 Story 均建议 `full-lld`，没有新增架构/安全/runtime 人工决策项，也没有绕过 CP5 的实现授权。

## 交付文件

| Feature | 文件 | 状态 |
|---|---|---|
| FEAT-AW-01 | `process/docs/features/cr051-routing/DESIGN.md` | complete-for-CP4 |
| FEAT-AW-01 | `process/docs/features/cr051-routing/TEST-PLAN.md` | complete-for-CP4 |
| FEAT-AW-01 | `process/docs/features/cr051-routing/TASKS.md` | complete-for-CP4 |
| FEAT-AW-02 | `process/docs/features/cr051-worktree/DESIGN.md` | complete-for-CP4 |
| FEAT-AW-02 | `process/docs/features/cr051-worktree/TEST-PLAN.md` | complete-for-CP4 |
| FEAT-AW-02 | `process/docs/features/cr051-worktree/TASKS.md` | complete-for-CP4 |

## Story 下游契约

| Story | feature_design_refs | lld_policy.required_level | 原因 |
|---|---|---|---|
| ST-AW-001 | `process/docs/features/cr051-routing/{DESIGN,TEST-PLAN,TASKS}.md` | `full-lld` | portable schema、legacy/new 冲突、跨模块唯一写目标 |
| ST-AW-002 | `process/docs/features/cr051-worktree/{DESIGN,TEST-PLAN,TASKS}.md` | `full-lld` | 非原子状态机、并发锁、filesystem durability/capacity、Git/authz 与恢复 |

## 冻结的模块与文件所有权建议

| Owner | 主文件 | Shared / 禁止边界 |
|---|---|---|
| ST-AW-001 | `meta_flow/workspace/project_artifact_routing.py`、`tests/test_cr051_project_artifact_routing.py` | `routing.py` 需单写兼容窗口；不得 import lifecycle/Git/aggregate |
| ST-AW-002 | `meta_flow/workspace/project_worktree.py`、`worktree_capacity.py`、`worktree_journal.py`、`tests/test_cr051_project_worktree.py`、`tests/test_cr051_worktree_faults.py` | `git_sync.py` shared 单写；`cli.py` 由 core 指定唯一 owner；默认不改 `git_branch_lifecycle.py` |

FEAT-AW-02 只读消费 FEAT-AW-01 PASS `RouteDecision` 与 owned-target proof；不得重新解析、改变 write target 或修改 FEAT-AW-01 主文件。

## O-AW-01 / O-AW-02 消费结果

| Open ID | 设计与测试闭环 | CP5 关闭条件 | 失败路由 |
|---|---|---|---|
| O-AW-01 | 规定 checkout/store filesystem 分开观测、deterministic upper bound、1.5x safety、bounded 512MiB 条件；TEST-PLAN 含 CAP-01..11，覆盖常见、误差、阈值、无权限、无法枚举、profile mismatch、跨 filesystem | 适用 bounded profile `false-safe=0`、`underestimate=0`；无权限/无法枚举必须 fail closed | 禁用 auto switch；不得用固定 512MiB 静默放行 |
| O-AW-02 | 固定 store-local temp → write/file-fsync → atomic replace → dir-fsync → checksum/readback；TEST-PLAN 含 DUR-01..14，覆盖 ENOSPC/EACCES/fsync/replace/corruption/kill/cross-device | persistence fault 前 Git mutation=0；损坏 record 不消费；resume 10 次保持幂等 | mutation 前 BLOCKED，切 manual-only，禁止 destructive fallback |

CP3-DC-01 的 fresh post-observation、conditional rollback、第三态 preserve/RECOVERY_REQUIRED 和禁止 reset/clean/stash/force/delete/overwrite 已分别落到接口、状态机、失败表、测试矩阵和 TASK-AW-W08/W11。

## TASK/DAG 摘要

- FEAT-AW-01：`TASK-AW-R01..R06`；schema → route → ownership/adapter → fixtures/audit。
- FEAT-AW-02：`TASK-AW-W01..W11`；W03 capacity 与 W04 durable store 可并行；W08 switch 必须等待 W02/W03/W04/W05；W09 集成等待 bootstrap/remove/switch 收敛。
- shared file：`routing.py`、`git_sync.py`、条件式 `cli.py` 必须由 core Development Plan 给予明确单写窗口。

## 验证与门禁建议

| 检查 | 期望 |
|---|---|
| Feature pack 结构 | 2/2 Feature 均具备 DESIGN/TEST-PLAN/TASKS |
| Requirement/Scenario trace | ST-AW-001 回链 REQ-AW-001..003/013/NF001..002 与 TC-AW-001..003/010/012；ST-AW-002 回链 REQ-AW-004..007/013/C002..003/NF003..005 与 TC-AW-004/005/007/010/011/012/014/015 |
| 安全边界 | 真实 artifact、source、link、ref、remote mutation=0；所有实现/fixture 仍受 CP5 与临时目录边界控制 |
| Feature Matrix | 本 lane 未写；core lane 必须把两个 Feature 标为 `required` 并回填上述 refs/policy |

## 阻塞、开放项与授权

- 新 blocker：0。
- 新人工 decision-item：0；CP3 DQ-01..03/ADR-AW-001..007 只被消费，没有重开。
- non-blocking-open：O-AW-01/02 继续存在，已转为 CP5 机器可验证义务；不能以风险接受替代 fixture 证明。
- 未执行：源码/测试实现、真实 Git/worktree/ref/remote/link/migration/main-sync mutation、STATE/ledger/checkpoint/CP4 result 修改。
