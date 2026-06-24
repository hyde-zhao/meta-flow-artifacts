---
check_id: "CR138-FOLLOW-UP-CR-COVERAGE-AUDIT-2026-06-24"
change_id: "CR-138"
status: "PASS_WITH_UPDATES"
created_at: "2026-06-24T18:55:00+08:00"
owner: "host-orchestrator"
source_request: "USER-20260624-FOLLOW-UP-CR-COVERAGE-AUDIT"
scope:
  - "process/baseline/CURRENT-REQUIREMENT-BASELINE.yaml"
  - "process/changes/CR-INDEX.yaml"
  - "process/changes/CR-091-FOLLOW-UP-TRACKING-2026-06-18.md"
  - "process/changes/CR-098-FOLLOW-UP-TRACKING-2026-06-19.md"
  - "process/changes/CR-126-QUANT-LAB-RUNNER-DEVELOPMENT-READINESS-RUNNER-ENTRY-DESIGN-2026-06-23.md"
  - "process/changes/CR-137-OFFLINE-RUNNER-RUN-REGISTRY-2026-06-23.md"
  - "process/STORY-BACKLOG-CR138.md"
  - "process/DEVELOPMENT-PLAN-CR138.yaml"
---

# CR138 Follow-up CR Coverage Audit

## 1. 结论

CR138 的 8 个 Story 可以吸收一部分原计划 / 后续跟踪项，但不能把真实 QMT / MiniQMT / XtQuant / gateway runtime、NAS、账户 / 行情 / 订单真实读取、submit/cancel、simulation/live 或 Git remote 写入并入当前开发任务。

本次处理采用以下规则：

| 分类 | 含义 | 处理 |
|---|---|---|
| `absorbed_by_CR138` | 当前 CR138 Story 已覆盖其后续开发价值 | 不再作为后续 CR 推进；保留历史审计，不物理删除旧台账 |
| `covered_as_boundary_only` | CR138 只覆盖合同、拒绝路径、授权模板或 fixture，不覆盖真实动作 | 保留为未来独立授权 gate 或新 CR |
| `closed_before_CR138` | 旧候选已由 CR102/103/104/126/128/137 等关闭 | 从 active / next 推荐中移除，仅作为历史证据 |
| `not_suitable_for_CR138` | 外部权限域、交易写、NAS、迁移、治理 warning 等不适合并入本轮功能实现 | 保留或降级为独立后续计划 |

## 2. 覆盖矩阵

| 原计划 / 后续项 | 来源 | 当前状态判断 | CR138 覆盖 | 处理结论 |
|---|---|---|---|---|
| CR126 Runner Entry Design / Scope Gate | CR125 后续、CR126 正式 CR | `closed_before_CR138` | S02 覆盖 Runner 控制壳，S03/S04/S08 覆盖后续运营状态、证据和 guardrail | 不再作为后续 CR；CR-INDEX stale item 应改为 closed |
| CR126 规划中的 Runner Core MVP Implementation | CR126 正文中旧称 CR127；实际落地为 CR128 | `closed_before_CR138` | CR138 不重做 offline runner core，只在 S02/S03 只读消费 strategy-runner-core / registry | 不再作为后续 CR；CR128 保留为依赖基线 |
| CR137 后续 offline batch run | CR137 summary / context | `absorbed_by_CR138` | S02 增加本地 `RunPlanBatch` / batch preflight；S03 增加 batch OpsSummary 聚合；仍为 local / fixture / registry 级能力 | 加入当前 S02/S03 设计证据；不再排独立 CR |
| FU-CR137-001 / WARN-CR137-CR-TRACKING | CP8-CR137 风险接受项 | `not_suitable_for_CR138` | 无业务功能覆盖 | 保留为低优先级治理候选；不放入 CR138 功能开发 |
| RA-CR097-001 Non-empty / trading-day readonly retest | CR098 follow-up | `closed_before_CR138` | CR104 已覆盖 nonzero readonly retest；CR138 S06/S07 覆盖未来查询/订阅合同 | 标记为 closed / superseded_by_CR104，不再作为后续 CR |
| RA-CR101-001 QMT direct-run validation gate | CR101 后续 | `covered_as_boundary_only` | S01/S08 覆盖授权模板和 no-real-op guardrail；不覆盖真实 QMT terminal direct-run | 保留为未来可选 runtime gate；不并入 CR138 |
| RA-CR101-002 MiniQMT gateway adapter validation gate | CR101 后续，CR104 已关闭 readonly runtime | `closed_before_CR138` + `covered_as_boundary_only` | S05/S06/S07 覆盖 Gateway service layer 合同；真实 adapter runtime 仍不授权 | 旧验证 gate 不再跟踪；未来若要实现 xtquant adapter，应新开 CR138-FU-RT-01 |
| RA-CR101-003 NAS real package exchange gate | CR101 后续，CR102 已关闭 | `closed_before_CR138` | 无；CR138 明确禁止 NAS | 从 CR138 后续计划中排除；额外 NAS 动作需独立 gate |
| FU-CR101-001 / proposed CR105 Order-write / simulation / live | CR101 后续 | `covered_as_boundary_only` | S07 覆盖 hard-reject、ExecutionReport、RecoveryPlan 边界；不覆盖真实 submit/cancel/sim/live | 保留为未来高风险交易写 CR；须在 readonly adapter 稳定后再评审 |
| CR063 NAS archive / data migration | CR062 后续 | `not_suitable_for_CR138` | 无 | 归入仓储 / 迁移计划，不进入 Runner/QMT Gateway P0 |
| CR064 Data lake root / catalog migration | CR062 后续 | `not_suitable_for_CR138` | 无 | 归入数据湖 / catalog 迁移计划，不进入 CR138 |
| CR065 Runtime / trading authorization | CR062 后续旧泛化项 | `superseded_by_scoped_runtime_policy` | CR138 S08 提供按需 runtime_authorization 模板 | 不保留泛化 CR；未来按具体 action scope 新建 |

## 3. 加入 CR138 当前开发任务的内容

| 加入项 | 放入 Story | 说明 | 权限边界 |
|---|---|---|---|
| Local batch run plan | CR138-S02 | 增加 `RunPlanBatch`、`batch_id`、批量 preflight 聚合和本地 CLI plan/preflight 支持 | 只读 strategy-runner registry / bundle refs；不启动 runtime |
| Batch ops summary | CR138-S03 | OpsSummary 增加 batch 汇总视图，统计 planned / blocked / manual_review / local summaries | 只读取本地 RunState / registry refs / redacted counters |

## 4. 不并入 CR138 的后续计划

| 优先级 | 后续计划 | 触发条件 | 为什么不放入 CR138 |
|---:|---|---|---|
| 1 | CR138-FU-RT-01 Gateway xtquant adapter Spike / readonly integration design | CR138 S05/S06/S07 fixture 实现和 CP7 通过后，用户明确需要真实 Gateway adapter | 需要 runtime / credential / host window / redaction 重新授权 |
| 2 | CR138-FU-RT-02 QMT terminal direct-run validation gate | 用户仍需要证明 QMT terminal 能加载 direct-run target | CR138 是 Runner/Gateway control plane，不是 QMT terminal direct-run 运行 gate |
| 3 | CR138-FU-TRD-01 Order-write / simulation / live design gate | readonly adapter 和 risk/OMS hard-reject 已稳定，用户接受高风险交易写门禁 | 交易写风险最高，必须独立 CP2/CP3/CP5/CP8 |
| 4 | CR138-FU-GOV-01 CR tracking warning cleanup | 用户要求零 warning 或 cr-tracking 输出收敛 | 治理项，不影响 Runner/Gateway 功能开发 |
| 5 | Storage / NAS / data-lake migration CR | 用户重新启动 NAS、数据湖或 catalog 迁移 | 外部数据 / 存储权限域，非 CR138 P0 |

## 5. 更新要求

- `process/stories/CR138-S02-...` 与 `CR138-S03-...` 必须补充 batch run 设计证据。
- `process/STORY-BACKLOG-CR138.md`、`process/DEVELOPMENT-PLAN-CR138.yaml` 必须记录 batch run 已并入 S02/S03。
- `process/changes/CR-INDEX.yaml` 中 stale 的 CR126 / CR103 / RA-CR101 / RA-CR097 状态应从 active / blocked / candidate 语义收敛为 closed / deferred / future scoped gate。
- `process/baseline/CURRENT-REQUIREMENT-BASELINE.yaml` 的 `next_recommended_candidates` 必须改为 CR138 后的 scoped future plan，不再列已关闭的 RA-CR101-002/003 作为推荐下一步。

## 6. 不授权声明

本审计和后续文档更新不授权实现、测试运行之外的真实外部动作，不授权 QMT / MiniQMT / XtQuant / gateway runtime、端口绑定、凭据读取、账户 / 行情 / 订单 / 成交真实查询、submit/cancel、simulation/live、NAS、provider/lake/catalog 或 Git remote 写入。
