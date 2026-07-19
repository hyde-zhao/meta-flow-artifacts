---
feature_id: "FEAT-AW-03"
change_id: "CR-051"
story_ids: ["ST-AW-003"]
status: "ready-for-story-design"
version: "1.1"
---

# FEAT-AW-03 任务清单

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-18 | meta-se-critical | 将异构 leg 契约拆为可执行 TASK-ID，并固定文件 owner、依赖和完成准则。 |
| 1.1 | 2026-07-19 | Host Orchestrator（inline-fallback） | 增加 006–008 终验回修任务并记录 scoped CAS/policy/高优回归完成状态。 |

## 1. Story 映射

全部任务归属 `ST-AW-003`，设计证据级别为 `full-lld`。CP5 全量设计证据确认前所有任务状态保持 `planned`，不得修改源码或执行真实 Git mutation。

## 2. 任务 DAG

```text
TASK-AW-003-01
  ├─> TASK-AW-003-02 ─┐
  └─> TASK-AW-003-03 ─┼─> TASK-AW-003-04 ─> TASK-AW-003-05
```

| TASK-ID | 状态 | depends_on | 动作与产物 | 完成准则 |
|---|---|---|---|---|
| TASK-AW-003-01 | completed | ST-AW-001、ST-AW-002 contracts | 在 full LLD 固化 `LegRequest/Plan/Attempt/Result` schema、mode dispatch、状态/错误枚举、CR-050 applicability table | 字段、owner、调用方向、failure route 100% 明确；artifact main target 明确不可达 |
| TASK-AW-003-02 | completed | TASK-AW-003-01 | 实现 source-default leg policy/executor 候选切片 | fresh source default OID、typed authz、argv-only、post-proof 与 result 单写满足设计 |
| TASK-AW-003-03 | completed | TASK-AW-003-01 | 实现 artifact-integration leg policy/executor 候选切片 | base/target 只为 integration；main/control/sibling mutation=0 |
| TASK-AW-003-04 | completed | TASK-AW-003-02、TASK-AW-003-03 | 实现 resume/abort、fresh drift、evidence adapter 与 no-cross-leg-rollback | stale/partial/evidence failure 均不产生 overall PASS；恢复幂等 |
| TASK-AW-003-05 | completed | TASK-AW-003-04 | 建立 bare-remote、command-spy、authz、correlation、legacy regression tests | TEST-PLAN P0/P1 100% 执行；所有 hard invariant 通过 |
| TASK-AW-003-06 | completed | TASK-AW-003-01 | 抽取 canonical artifact policy 供 leg producer 与 aggregate consumer 共用 | mode/base/target/active ref 单一纯规则源；main/default hard deny |
| TASK-AW-003-07 | completed | TASK-AW-003-03、TASK-AW-003-04 | 实现 integration containment proof 与 expected-OID CAS 清理 | 非 containment、错 ref/OID/step mutation=0；ordinary force=0 |
| TASK-AW-003-08 | completed | TASK-AW-003-05 | 补 TP-AW03-004/006/008/014 高优测试 | 4/4 PASS；legacy/sibling/cross-leg/main 边界均有回归保障 |

## 3. 文件所有权

| 分类 | 路径 | owner / 规则 |
|---|---|---|
| primary | `meta_flow/workflow/artifact_leg_lifecycle.py` | ST-AW-003 独占写 |
| primary | `tests/test_artifact_leg_lifecycle.py` | ST-AW-003 独占写 |
| shared-read/limited-write | `meta_flow/workspace/git_sync.py` | 仅 native Git 通用 probe/adapter；业务状态机禁止进入 |
| shared-read | `meta_flow/workflow/git_branch_lifecycle.py` | 复用安全 contract；不得改 paired-default 默认语义 |
| merge owner | `meta_flow/workflow/artifact_leg_lifecycle.py` | ST-AW-003 |
| external merge owner | `meta_flow/cli.py` | ST-AW-004；ST-AW-003 不并行写 |
| forbidden | artifact main/default refs、control/sibling checkout、`process/quant-lab/**`、凭据/production remote | 读写均受 DESIGN 限制；真实 mutation 不在 Story 默认授权内 |

## 4. 开发与验证门

- Dev Entry：ST-AW-003 full LLD 已确认；ST-AW-001 route contract 与 ST-AW-002 worktree health contract 已冻结；与 ST-AW-004 的 `LegResult` 消费契约已确认；文件冲突检查通过。
- Dev Exit：实现对象清单、设计契约映射、测试 fixture、local validation 与 design delta 全部有证据；源码变更限于批准路径。
- Verification：由 meta-qa 独立验证 target matrix、mutation denylist、correlation、drift、authz、resume 与 legacy regression。

## 5. 回退与升级条件

若 CR-050 policy 无法在不改变 legacy 默认行为的情况下显式 override artifact target，或实现必须更新 artifact main、跨 leg rollback、共享 CLI 并行写，则停止任务并返回 `NEEDS_DESIGN_CLARIFICATION`；不得把 target 偏差降级为技术债。

## 6. Gotchas

- TASK 完成不等于 Story 可发布；仍需 CP6/CP7 证据。
- `git_sync.py` 是 adapter，不是放置 leg 状态机的便捷共享模块。
- ST-AW-004 可以与本 Story 并行实现的前提，是 `LegResult` contract 在 CP5 前冻结且 CLI 只有一个 merge owner。
