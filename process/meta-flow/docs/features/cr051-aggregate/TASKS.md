---
feature_id: "FEAT-AW-04"
change_id: "CR-051"
story_ids: ["ST-AW-004"]
status: "ready-for-story-design"
version: "1.1"
---

# FEAT-AW-04 任务清单

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-18 | meta-se-critical | 将聚合 schema、validator、single writer、projection 与验证拆为稳定 TASK-ID。 |
| 1.1 | 2026-07-19 | Host Orchestrator（inline-fallback） | 增加 006–008 终验回修任务，关闭 target-policy、PARTIAL 与关键测试缺口。 |

## 1. Story 与设计证据

全部任务归属 `ST-AW-004`，`lld_policy=full-lld`。可与 ST-AW-003 并行起草 LLD/实现，但必须先冻结 `LegResult` schema，且 ST-AW-004 不修改 ST-AW-003 primary 文件；CR-051 的 `meta_flow/cli.py` 接线由本 Story 单独合并。

## 2. 任务 DAG

```text
TASK-AW-004-01
  ├─> TASK-AW-004-02
  └─> TASK-AW-004-03
       TASK-AW-004-02 + TASK-AW-004-03
                    └─> TASK-AW-004-04 ─> TASK-AW-004-05
```

| TASK-ID | 状态 | depends_on | 动作与产物 | 完成准则 |
|---|---|---|---|---|
| TASK-AW-004-01 | completed | FEAT-AW-03 contract frozen | full LLD 固化 AggregateRequest/ValidatedLegSet/AggregateResult、precedence、错误枚举、single-writer key | 所有字段/状态/owner/失败路径明确；PARTIAL 只在 progress/effect |
| TASK-AW-004-02 | completed | TASK-AW-004-01 | 实现 result validator 与纯聚合函数候选切片 | invalid correlation fail closed；16 状态组合 100% 一致；Git import/call=0 |
| TASK-AW-004-03 | completed | TASK-AW-004-01 | 实现 aggregate persistence、idempotency/conflict detection 与 evidence ref | append/readback 后才可投影；冲突 payload BLOCKED |
| TASK-AW-004-04 | completed | TASK-AW-004-02、TASK-AW-004-03 | 实现 projection guard、next route 与 CR-051 CLI 接线 | 仅 2/2 PASS 调用 controlled writer；非 PASS 不 close/rollback/sync |
| TASK-AW-004-05 | completed | TASK-AW-004-04 | 建立决策表、correlation、concurrency、projection、legacy regression tests | TEST-PLAN P0/P1 100% 执行，hard invariant failure=0 |
| TASK-AW-004-06 | completed | TASK-AW-004-02 | 在 consumer 重读后独立验证 canonical mode/base/target/active ref | producer/consumer 漂移 100% BLOCKED；Git 调用=0 |
| TASK-AW-004-07 | completed | TASK-AW-004-04 | 实现 staged projection receipt、PARTIAL 与幂等 retry | 已发生 state fact 不丢失；retry 不重跑 Git/aggregate |
| TASK-AW-004-08 | completed | TASK-AW-004-05 | 补 dependency boundary、exact DAG、target policy 与 PARTIAL 测试 | 关键用例全 PASS；同构期望测试不再是唯一判据 |

## 3. 文件所有权

| 分类 | 路径 | owner / 规则 |
|---|---|---|
| primary | `meta_flow/workflow/artifact_aggregate.py` | ST-AW-004 独占写 |
| primary | `tests/test_artifact_aggregate.py` | ST-AW-004 独占写 |
| shared merge owner | `meta_flow/cli.py` | ST-AW-004 是 CR-051 唯一接线 owner |
| contract read-only | `meta_flow/workflow/artifact_leg_lifecycle.py` | ST-AW-003 owner；本 Story 不修改 |
| existing controlled writer | CR/state/current adapter | 只通过已批准 API 调用，不复制 truth source |
| forbidden | Git adapter/executor mutation、artifact main、manual sync、自动 CR close/rollback、`process/quant-lab/**` | 任何依赖即设计偏差 |

## 4. 门禁与完成定义

- Dev Entry：full LLD 与全量 CP5 已批准；LegResult contract frozen；CLI/file conflict 检查通过。
- Dev Exit：实现证据逐项映射 DESIGN；16 组合、invalid result、single-writer、projection tests 通过；没有超出批准路径。
- Verification：meta-qa 独立确认 pure-function dependency boundary、input digest、non-PASS projection=0 与 legacy behavior。

## 5. 升级条件与 Gotchas

若实现需要聚合器直接调用 Git/worktree executor、从 artifact 单腿推断 overall PASS、把 PARTIAL 加入 overall enum、按 mtime 选择 current attempt、或新增第二套 CR/state writer，立即返回 `NEEDS_DESIGN_CLARIFICATION`。聚合结果持久化成功与 completion projection 成功是两个独立阶段，不得用一次“成功返回”合并。
