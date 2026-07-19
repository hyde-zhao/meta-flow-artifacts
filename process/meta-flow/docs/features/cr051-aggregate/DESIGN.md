---
feature_id: "FEAT-AW-04"
feature_name: "Aggregate Gate & Evidence"
change_id: "CR-051"
story_ids: ["ST-AW-004"]
status: "ready-for-story-design"
applicability: "required"
lld_policy: "full-lld"
version: "1.2"
updated_at: "2026-07-19"
---

# FEAT-AW-04 聚合完成门实现设计

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-18 | meta-se-critical | 固化单写 AggregateResult、correlation validator、状态优先级、PARTIAL 非成功语义与 projection hard gate。 |
| 1.1 | 2026-07-18 | meta-dev-debugger | CP5 R2：关闭 F03；aggregate 只消费 reread-validated PublishedLegResultHandle，并固化 payload→receipt→handle→aggregate→receipt→projection DAG。 |
| 1.2 | 2026-07-19 | Host Orchestrator（inline-fallback） | CP8 终验回修：增加 consumer-side canonical target policy、staged projection receipt、PARTIAL production 语义与幂等 retry。 |

## 1. 目标与适用性

FEAT-AW-04 只消费 FEAT-AW-03 输出的 `PublishedLegResultHandle`。validator必须从handle的result_ref重读immutable `LegResultPayload`，验证external write receipt、payload/ref/digest/single-write key与correlation后，才以纯函数计算逻辑CR整体结果。固定状态优先级为：

```text
BLOCKED > FAIL > IN_PROGRESS > PASS
```

只有 source/artifact 两条 required leg 均为匹配当前 attempt 的 terminal PASS 时，overall 才能 PASS。单腿成功、缺失结果、陈旧结果、PARTIAL effect 或 evidence 不可验证均不得推进完成投影。

该 Feature 必须使用 `full-lld`：它拥有 overall 单写、跨 Story schema、CR/state/current projection guard 和失败恢复路由，属于高风险状态/持久化/跨模块契约。

量化成功标准：

- 16 组 source/artifact 四状态笛卡尔组合的 overall 与固定优先级一致率 100%。
- `PASS` 的必要充分条件测试覆盖率 100%；required leg 非 2/2 terminal PASS 时 PASS 数=0。
- raw payload/ref、unpublished outcome、stale/duplicate/wrong CR/wrong attempt/wrong project/wrong required set或receipt/ref/digest/key错配的拒绝率100%。
- artifact 单腿成功导致 CR close/state-current completion 的次数=0。
- 相同 validated input 重算结果 digest 一致率 100%，聚合过程 Git adapter 调用次数=0。

## 2. 边界

### 2.1 In Scope

- 定义 `AggregateRequest`、`ValidatedLegSet`、`AggregateResult` 与错误枚举。
- 重读并验证published handle对应payload schema/version/digest、external receipt、derived single-write key、operation/CR/project/attempt correlation、required leg set和唯一性。
- 纯函数计算 overall、progress/effect、blockers、next route。
- 通过既有 writer adapter 写入唯一 aggregate result；仅 validated all-PASS 才允许完成投影。
- 为 CLI/上层 workflow 提供只读 status/decision 接口和显式 resume/abort 建议。

### 2.2 Out of Scope 与相邻边界

| 相邻对象 | 本 Feature 不负责 | 正确衔接 |
|---|---|---|
| FEAT-AW-03 | 执行Git、重试/修复leg、生成/发布payload | 只接收PublishedLegResultHandle并经reader重读payload；raw/unpublished拒绝 |
| FEAT-AW-02 | worktree switch/recover | 不反向调用 lifecycle mutation API |
| CR lifecycle | 自动关闭失败/部分完成 CR、自动 rollback | 只输出 route；既有 writer 在 hard gate 后投影 |
| Manual Sync | main↔integration 同步 | 不调用、不建议自动执行，仅可说明 CR 外人工边界 |
| Publication binding | 把 receipt 绑定到未来承载自身的 commit OID | 由后置 event 处理，避免自引用 |

## 3. 冻结数据契约

### 3.1 `AggregateRequest`

| 字段 | 约束 |
|---|---|
| `operation_id` / `logical_attempt` | 与当前 CR operation 一致；不可用最新文件猜测 |
| `cr_id` / `project_id` | 与 route context 一致 |
| `required_legs` | shared-artifact mode 固定且有序为 `source`,`artifact`；缺失/重复非法 |
| `published_leg_handles` | exactly source+artifact两个handle；含result_ref、payload_digest、external receipt，不接受raw payload/ref或executor state |
| `policy_version` | 冻结 precedence 与 projection guard 版本 |

### 3.2 `ValidatedLegSet`

Published Result Validator 对每个 handle：从result_ref重读payload；重算payload digest与derived single-write key；验证receipt digest绑定key/ref/payload digest/writer/written_at；比对handle/receipt；再检查schema/version、唯一性、operation/attempt/CR/project、leg kind/mode、base/target policy、terminal/status。验证失败不是业务FAIL，而是聚合可信度`BLOCKED(invalid-published-result-set)`。

### 3.3 `AggregateResult`

Immutable `AggregateResult` payload最小字段：schema/version、aggregate ID、operation/attempt/CR/project、required published handle refs及payload digests、validated correlation、precedence version、overall、terminal、progress/effect、blockers、next route、projection decision、input digest、created_at。它不含自己的aggregate write receipt/ref/writer ID；这些由external `AggregateWriteReceipt`承载。

`progress/effect` 可以表达 `PARTIAL`，例如 source 已完成而 artifact 失败；`overall` 不存在 PARTIAL 枚举。`terminal=true` 仅表示当前 attempt 已得到稳定结论，不等于成功；BLOCKED/FAIL 也可以 terminal。

## 4. 决策表

按下列顺序短路，前序规则优先：

| Order | 条件 | overall | terminal | projection | next route |
|---|---|---|---|---|---|
| 1 | schema/digest/correlation/required-set/唯一性验证失败 | BLOCKED | true | hold | 修复 evidence 或人工仲裁 |
| 2 | 任一 required leg=`BLOCKED` | BLOCKED | 所有 required result 已稳定时 true，否则 false | hold | resolve blocker / resume |
| 3 | 无 BLOCKED，任一 required leg=`FAIL` | FAIL | 所有 required result 已稳定时 true，否则 false | hold | preserve facts / explicit resume-or-abort |
| 4 | 无 BLOCKED/FAIL，任一 result 缺失、非 terminal 或 `IN_PROGRESS` | IN_PROGRESS | false | hold | 等待或 resume |
| 5 | required source+artifact 均 terminal PASS | PASS | true | eligible | 允许既有 writer 进行完成投影 |

无论 progress/effect 是否为 PARTIAL，只要不满足第 5 条，`projection=hold`。

## 5. 单写与调用契约

| 调用方 → 被调用方 | 时机 | 输入 | 输出 | 失败/降级 | 同步范围 |
|---|---|---|---|---|---|
| Coordinator → Published Result Validator | status/finish/恢复时 | AggregateRequest + published handles | reread-validated payload set / errors | raw/unpublished或任一receipt/ref/digest/key不可信均BLOCKED | 与FEAT-AW-03冻结published contract |
| Aggregate Pure Function → ValidatedLegSet | validation 后 | typed set + policy version | deterministic decision | 不访问文件/Git/网络 | 单元测试 16 组合 |
| Aggregate Writer → existing result/evidence adapter | decision 后 | immutable aggregate payload | append receipt/ref | 写入失败不得投影 PASS | 不新增第二套 ledger |
| Projection Guard → existing CR/state/current writers | aggregate persisted 后 | validated PASS ref | completion projection | 非 PASS、写入失败、stale aggregate 均 hold | 复用既有 controlled writer |
| CLI → Coordinator | 显式 status/aggregate | CR/project/attempt selector | 结果与审计 refs | 不隐式 resume/sync/close | CR-051 CLI 唯一 merge owner=ST-AW-004 |

Aggregate 模块不得 import native Git adapter 或 leg executor；依赖只能从 coordinator/validator 指向 result model 和既有 writer adapter。

## 6. 模块与文件影响候选

| 类型 | 候选路径 | 责任 / owner | 并行约束 |
|---|---|---|---|
| 新模块 | `meta_flow/workflow/artifact_aggregate.py` | ST-AW-004：validator、pure aggregate、projection guard | primary |
| 测试候选 | `tests/test_artifact_aggregate.py` | ST-AW-004 | primary |
| CLI | `meta_flow/cli.py` | ST-AW-004 统一 CR-051 status/coordinator 接线 | CR-051 唯一 merge owner |
| 只读 contract | `meta_flow/workflow/artifact_leg_lifecycle.py` | FEAT-AW-03 `LegResult` | ST-AW-004 不修改；CP5 前冻结 contract |
| existing writer | controlled CR/state/current adapter | 仅 guarded invocation | 不复制 writer 或 truth source |

## 7. 幂等、并发和持久化

- evidence DAG固定为 `LegResultPayload → LegResultWriteReceipt → validated PublishedLegResultHandle → immutable AggregateResult → AggregateWriteReceipt → controlled projection`；各payload均不含自己的receipt/ref，禁止二次覆盖。
- aggregate ID由operation/attempt/policy version与两个validated payload digest确定；相同输入重复计算返回相同payload digest。
- 单个 operation/attempt 只能有一个 current aggregate writer lease；重复 append 必须识别 same-payload idempotent 与 conflicting-payload BLOCKED。
- 新 attempt 不覆盖旧 attempt 事实；current selector 由显式 workflow state 指定，不按 mtime 选择。
- leg result 追加后才能聚合，aggregate 持久化且 readback 通过后才能投影。
- 投影部分失败不重写 aggregate；记录 projection failure 并保持可重试，不重新执行 Git leg。

## 8. Story 下游契约

`ST-AW-004` full LLD必须细化PublishedLegResultHandle reader/validator、payload/receipt/key校验、pure function、aggregate external receipt、single-writer/idempotency、projection boundary、CLI和fixture。它可以与ST-AW-003并行实现，前提是published-result contract在CP5批准前冻结、ST-AW-004不写ST-AW-003 primary，且CLI只由ST-AW-004合并。

## 9. 风险与 Gotchas

| 风险 | 缓解 | 验证 |
|---|---|---|
| artifact 单腿成功误推进 | all-required-PASS hard gate | projection negative matrix |
| 陈旧 result 混入 | explicit attempt/correlation + digest | stale/duplicate fixtures |
| 聚合时产生 mutation | pure function + import boundary | dependency/command spy |
| 重试产生冲突结果 | deterministic ID + conflicting-payload BLOCKED | concurrency fixture |

Gotchas：状态优先级只在可信的 matching result set 上计算，validation failure 必须先路由 BLOCKED；`terminal` 与 `success` 是两个维度；PARTIAL 是 progress/effect，不是第五种 overall；聚合 PASS 只表示允许投影，不能绕过既有 writer 的 controlled update 规则。

## 10. CP8 终验回修设计增量（ST-AW-004）

1. `AggregateRequest` 只接受 canonical mode：source=`source-default`，artifact=`shared-artifact-project-first`。CLI 以枚举约束输入，不提供会漂移到旧 `artifact-integration` 的默认值。
2. validator 在重读并验证 receipt/digest/correlation 后，必须使用共享纯 `artifact_policy.target_policy_errors()` 独立重算 mode、base、target 与 active ref；producer 产出的“自洽但错误 target”必须 `BLOCKED(TARGET_POLICY_MISMATCH)`。
3. controlled projection 是 staged side effect：先写 state/current projection fact，再追加 CR ledger/current selector。每个成功阶段返回 receipt；后续阶段失败不得删除或伪装前序事实。
4. state 写入成功、ledger/current 未完成时返回 `ProjectionStatus.PARTIAL`，coordinator 路由 `retry-controlled-projection`；retry 只补未完成投影，不重写 aggregate、不重跑 Git leg。
5. state 写入前失败返回 `FAILED`；全部完成返回 `COMPLETE`；重复 retry 对已存在且 digest 相同的 state fact 幂等。`PARTIAL` 仍不是 aggregate overall，也不允许把非 2/2 PASS 投影为完成。
