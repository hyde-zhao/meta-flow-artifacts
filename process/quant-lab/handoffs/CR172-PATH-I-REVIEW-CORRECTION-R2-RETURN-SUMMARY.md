---
handoff_id: "CR172-PATH-I-REVIEW-CORRECTION-R2-RETURN"
from_agent: "meta-pm"
to_agent: "host-orchestrator"
change_id: "CR-172"
phase: "requirement-clarification"
checkpoint: "CP2"
status: "completed"
created_at: "2026-07-17T15:50:00+08:00"
---

# CR172 PATH-I review correction R2 return summary

## 结论

correction R2 已按最新 PATH-I 评审意见完成。只增量修改 8 个产品文档并保留全部修订记录、既有需求与场景 ID；未修改 CR-172、checkpoint/result、STATE、ledger、设计文档或代码。

## 产品范围修订

### DQ/REQ-015 收缩

- 默认信号由执行机基于批准策略包与本地行情生成；默认跨机信号传输为 `0`。
- PATH-I/CP3 对低频/EOD 只冻结可选 immutable `SignalBatch/TargetPositionBatch` 的精确 8 个最小字段：
  1. `schema_version`
  2. `batch_id`
  3. `strategy_id`
  4. `strategy_package_hash`
  5. `content_sha256`
  6. `signature/key_id`
  7. `valid_from/valid_until`
  8. `sequence_no`
- broker credential、account secret、broker order command 的允许数为 `0`。
- 七级状态机、具体 mailbox 物理路径、ack 状态机、idempotency/replay、真实 sync/transport/消费实现全部退出 PATH-I 当前规范。
- `SC-CR172-S01~S06` 保留 `6/6`，其中涉及 path/ack/idempotency/replay 的场景已重解释为 deferred-boundary 与 `implementation=0`，未删除或重编号。

### DQ/REQ-014 强化

- 登记 `FU-CR173-001`：Empirical dependency-matrix methodology v2 and sampling-error validation。
- 它不是重开 PATH-C/A 设计的绝对前置；DQ-003 的 `effective_trial_count=typed_unavailable` 降级路径仍可进入设计。
- 凡要输出 available effective trial count 或声明 `c1_computable=true`，`FU-CR173-001` 是硬前置，最小范围包含：
  - sampling-error/uncertainty evidence
  - method version 2 + hash
  - 有效域/偏差界限
  - 独立验证
- 未来 PATH-C/A CP2 必须显式三选一：完成 v2、拆为 future activation、采用 DQ-003 降级。

### Backlog 候选

| ID | 范围 | 当前状态 |
|---|---|---|
| `FU-CR173-001` | Empirical dependency-matrix methodology v2 and sampling-error validation | candidate；未建正式 CR |
| `DF-CR172-SIGNAL-BATCH-EXCHANGE` | 低频 immutable SignalBatch/TargetPositionBatch exchange 详细设计与实现 | deferred candidate；未建正式 CR |
| `DF-CR172-INTRADAY-REALTIME-SIGNAL` | intraday realtime signal transport | deferred candidate；未建正式 CR |

未提前建立 `FU-CR172-001~003`；两个 CR172 信号项是否转 FU，留待 CR172 CP8 决定。

### PATH-I 阶段守卫

| Gate | 允许的最高动作 | 必须保持 |
|---|---|---|
| CP2 | 只批准产品范围并解锁 CP3 | 不创建目录、不写 NAS、不运行 multi-trial、不传信号 |
| CP3 | 只冻结设计 | 目录创建、NAS write、multi-trial runtime、signal generation/transport=`0` |
| CP5 | repository-local code/test/fixture | 六类真实动作授权=`0/6`，真实 sync/pull/signal/runtime=`0` |
| CP7 | fixture/static 与零操作验证 | 六类真实动作执行计数=`0/6` |
| CP8 | 最高 `path_i_design_ready=true` 或等价 verified contract-ready | `stage3_entry_ready=false`、`c1_computable=false`、`real_data_authorized=false`、`multi_trial_runtime_authorized=false`、`signal_transport_authorized=false` |

## 不变项

- DQ/REQ-009~013 的产品决策语义不变。
- PATH-I 推荐不变。
- 四组件主权不变：研究机本地 active canonical、NAS verified replica/backup/distribution、执行机 verified local immutable cache、GitHub metadata-only。
- 新路径与 legacy read-only/no-migration/no-rewrite 规则不变。
- CR172 场景总量和 ID 不变。

## 修改文件

1. `docs/product/USE-CASES.md`
2. `docs/product/REQUIREMENTS.md`
3. `docs/product/SCENARIOS.yaml`
4. `docs/product/TEST-MATRIX.md`
5. `docs/product/STORY-MAP.md`
6. `docs/product/MVP-SCOPE.md`
7. `docs/product/RELEASE-SLICES.md`
8. `docs/product/BACKLOG.md`
9. 本 return summary。

## 精确计数与验证结果

| 检查 | 结果 |
|---|---|
| 产品文档增量更新 | PASS，`8/8` |
| `SCENARIOS.yaml` 解析 | PASS |
| 全局 scenario 总数 / coverage total | `152/152` |
| 全局 scenario ID 唯一 | PASS |
| CR172 P0 scenario | `27/27` |
| `SC-CR172-S01~S06` 保留并进入矩阵 | `6/6` |
| CR172 scenario → TEST-MATRIX | `27/27` |
| REQ/DQ-009~015 | `7/7` |
| REQ-009~015 负向或边界语义覆盖 | `7/7` |
| SignalBatch 精确最小字段 | `8/8` |
| R2 candidate ID | `3/3` |
| 错误 `FU-CR172-001~003` | `0` |
| 过期 empirical/intraday candidate ID | `0` |
| CP2/CP3/CP5/CP7/CP8 phase guard | `5/5` |
| CP7 六类真实动作 | `0/6` |
| 真实 sync/pull/signal/runtime | `0/0/0/0` |

## 未授权项

本次只修改产品基线。未执行或授权真实 data-lake read、multi-trial runtime/workspace write、trial-return generation、empirical-R computation、NAS replica sync、execution pull/verify/materialize、signal generation/exchange/transport/consume、trading、migration、deploy/publish 或 Git remote write。

## 阻塞

- meta-pm 阻塞：无。
- 正式 CP2 仍由 host-orchestrator 发起；本 correction 不代替人工 approval。
