---
status: "accepted-cp3"
version: "1.4"
change_id: "CR-172"
decision_scope: "PATH-I-design-only"
source_hld: "docs/design/HLD-TRIAL-RETURN-DEPLOYMENT-CONTRACTS.md"
source_discussion: "process/discussions/CP3-CR172-PATH-I-HLD-DISCUSSION-LOG.md"
confirmed_by: "user"
confirmed_at: "2026-07-18T10:10:02+08:00"
---

# 架构决策：Trial-Return 与跨机部署合同

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-17 | meta-se-critical | 初始 ADR 集；冻结 PATH-I 九项互相一致的架构决策、备选、切换条件、回退边界和不授权声明。 |
| 1.1 | 2026-07-18 | meta-se-critical | CP3 review correction R1：扩充 ADR-005，区分独立授权记录与动作执行资格 DAG；新增 ADR-010 冻结 `ReturnDefinitionV1` 的 architecture owner、v1 恰好两列和 versioned evolution；将 producer 具体位置收敛为 CP5 `DO-CR172-CP5-001`。 |
| 1.2 | 2026-07-18 | meta-se-critical | CP5 LLD review R1：ADR-001/010 将当前切片收窄为 fixture producer port，现有 runner diff=`0`，并禁止 `forward_label_proxy@v1` 进入 trial-return/empirical-R；ADR-004 固化唯一 seal digest/verifier；ADR-005 固化 decision origin/target binding；ADR-009 固化 append-only partial lineage BLOCKED audit lane；新增 ADR-011 登记真实 producer 与原子 lineage 独立前置 CR。 |
| 1.3 | 2026-07-18 | meta-se-critical | CP5 R2 最小整改：ADR-005 将 current-v1 `approved_ledger` 固定双 false；ADR-004 允许 S04 verifier-library dependency 但禁止绕过 S03 selection；ADR-005 删除 `evidence_kind` 第二真相；ADR-007 将 REQ-013 定为 contract-ready/runtime-enforcement-deferred，并加入 future native-producer path-enforcement 子前置。 |
| 1.4 | 2026-07-18 | meta-se-critical | CP5 R3 上层合同最小整改：ADR-004 删除 S03 verifier facade 备选，唯一冻结 S04 从 S03 current selected-replica read contract 取数并直接调用 S02 唯一 verifier-library；其余 schema、hash domain、Story DAG、file owner、测试计数、授权与 claim ceiling 不变。 |

## 1. 决策上下文

CR-172 PATH-I 需要为未来 empirical C1 建立可信 trial-return 上游和跨机器 artifact 链，但当前没有 native multi-trial producer、独立 trial-return series、真实数据/NAS/runtime 授权。CP2 已批准 DQ-CR172-009～015；本 ADR 只把批准的产品方向精确化为可评审的架构合同，不授权实现或真实操作。

## 2. 决策状态与适用范围

| 项 | 值 |
|---|---|
| 当前状态 | `accepted-cp3`（review correction R1 后） |
| 推荐架构 | `ARCH-A Native Sealed Artifact Pipeline` |
| 适用对象 | future per-trial portfolio return artifact、manifest/seal、NAS replica、execution materialization、authorization、empirical disposition |
| 不适用对象 | CR-163 metadata/ref 本体、CR-173 v2 方法实现、SignalBatch exchange/intraday、migration、trading/deploy |
| 当前真实动作 | 授权 `0/6`；执行 `0/6`；其他 lake/NAS/runtime/R/signal/trading/Git remote 操作均 `0` |

## 3. ADR-CR172-I-001：选择 native sealed artifact pipeline

### 决策

目标态采用 future native multi-trial instrumentation；当前 CR 只交付同一合同的 repository fixture producer port、seal/replica/materialization contract 与 zero-op guard，不修改现有 mature runner。external import-first 是 future native producer 不可行时的条件备选；真实 source absent 时回退 `typed_unavailable`。

### 备选与取舍

| 选项 | 优势 | 代价 | 选择 / 切换条件 |
|---|---|---|---|
| Native-first | 同源 lineage 最完整、验证链明确 | 后续需插桩和独立 runtime/generation 授权 | 推荐；producer 不可接入时切 import |
| Import-first quarantine | 可接外部预计算 series | provenance/security 风险更高 | 仅完整 import contract 获独立批准后启用 |
| Source absent | 零越权、诚实降级 | 无 positive empirical 输出 | 任一 owner/source/provenance/auth 缺失时使用 |

### 后果

- CR-163 metadata/ref、`layered_returns.csv`、scalar metrics 和 CR-173 declared R 都不具备 source kind。
- 当前能力必须写为 contract-only，不得写成 producer implemented。
- 当前仓库可识别的 native multi-trial portfolio-period-return producer=`0`；`DO-CR172-CP5-001` 关闭为现有 runner hook/diff=`0/0`。未来 producer/instrumentation 必须独立走 runtime-high-risk CR 并重新证明唯一 integration point、完整 owner inventory、授权与失败合同。

## 4. ADR-CR172-I-002：identity 使用 logical URI + content hash

### 决策

artifact canonical identity 是 stable logical URI 与 payload `content_sha256` 的组合；research/NAS/execution 的绝对路径只保存于 deployment mapping，不参与 lineage identity 或 hash domain。

### 理由

这样同一 sealed content 可在不同挂载布局下保持同一身份，同时允许同一 logical URI 的不可变历史版本由不同 content hash/release 明确区分。pointer 可切换，sealed version 不覆盖。

### 禁止

- mount path、主机名、当前 symlink target 进入 identity。
- 以“同名文件”或“路径更新日期”替代 expected release/hash。

### 回退

若实现不能稳定复算 canonical manifest/hash，保持 contract-only/BLOCKED，不得改用 absolute path identity。

## 5. ADR-CR172-I-003：四组件与三段数据链

### 决策

研究机本地目录是唯一 active canonical；NAS 只持有 versioned/hash-verified replica、backup、distribution；执行机只消费 pull+verify+atomic materialize 后的本地 immutable cache；GitHub 只持有 code/schema/docs/manifest/hash/pointer/release metadata。

### 依赖方向

```text
research-local canonical -> NAS verified replica -> execution local immutable cache -> future local runtime
                                    metadata refs -> GitHub metadata ceiling
```

reverse write、execution direct-NAS/research runtime read、NAS canonical promotion、GitHub real payload 均禁止。

### 备选

NAS/materializer 不可用时采用 research-local only、distribution blocked；不得采用 shared-drive runtime 作为降级。

## 6. ADR-CR172-I-004：严格 manifest / seal / replica / materialization 顺序

### 决策

顺序固定为：

1. future payload 写入 non-canonical staging；
2. 校验 object/schema/identity/time/value/basis/lineage；
3. 计算 payload hash，生成 canonical manifest body 与 manifest hash；
4. 生成引用 payload+manifest hash 的 immutable seal；
5. 复验后原子推进 research canonical selection；
6. authorized NAS staging → verify → replica receipt → distribution pointer；
7. authorized execution staging → verify expected release/manifest/seal/content → atomic cache → materialization receipt。

### 关键约束

- seal 在任何 replica 之前；未 seal artifact 永不分发。
- replica/materialization 复用并验证原 seal，不重新 seal、不改变 identity。
- 唯一 seal digest 是 `sha256:` + 同一 `canonical_artifact_seal_bytes` 的 lowercase SHA-256；`verify_sealed_trial_return_bundle` 返回 `VerifiedTrialReturnBundleV1.original_seal_sha256`，S03 不得计算第二套 digest。
- S04 仍只能从 S03 current selected-replica read contract 取数，execution staging port 必须返回 sealed bundle + selection；S04 必须直接依赖并调用 S02 唯一 verifier-library 做 bytes-level 复验。S03 verifier facade 允许项=`0`；S03 新增 digest/verifier facade/data bypass=`0/0/0`；S04 direct S02 verifier=`1`。禁止绕过 S03 selection、receipt-only seal 比对和第二 digest。
- pointer 只在本层全部验证通过后推进；partial object 永不 distributable/runtime。

### 回滚

采用 immutable versions + pointer-only rollback。回滚不修改、删除或重写旧 sealed payload/manifest/seal；清理 partial staging 必须由新的有效动作授权覆盖。

## 7. ADR-CR172-I-005：六类真实动作独立授权与判定

### 决策

以下 `6/6` action 必须使用独立 `ActionAuthorizationRecordV1`，在操作前分别判定，不做权限并集：

| Action | Enforcement point |
|---|---|
| `data_lake_read` | 真实 release/dataset dereference 前 |
| `multi_trial_runtime_and_workspace_write` | runner launch/workspace create-write 前 |
| `trial_return_generation` | artifact candidate 首次写入前 |
| `empirical_R_computation` | sealed trial refs 进入 estimator 前 |
| `nas_replica_sync` | NAS staging/write/pointer 前 |
| `execution_pull_verify_materialize` | NAS read/pull、execution staging/pointer 前 |

每个 record 独立携带 owner、scope revision/hash、allow/deny logical paths、有效期/撤销、approval/evidence refs。缺失、过期、撤销、scope/hash/path mismatch 一律 deny。

12-field approval record 不变。派生 `ActionDecisionV1.decision_origin` 只允许 `repository_fixture|approved_ledger`，`ActionScopeContextV1.target_kind` 只允许 `repository_fixture|real_operation`；fixture origin 与 non-fixture target 的接受数=`0`。current-v1 对任何 `approved_ledger` 输入固定 `authorized=false`、`eligible_to_execute=false`、reason=`APPROVED_LEDGER_ADAPTER_UNAVAILABLE`；caller 自报 enum 不构成 provenance。未来可信 issuer/envelope/adapter 只允许独立 runtime-high-risk CR 引入。`evidence_kind` 别名/字段/第二真相=`0`。

### 独立审批不等于独立执行

六份 record 可以独立审批、撤销和到期，但动作执行资格必须满足以下直接前置 DAG；箭头是 artifact/eligibility 依赖，不是合并审批：

```text
data_lake_read
  -> multi_trial_runtime_and_workspace_write
    -> trial_return_generation
      -> empirical_R_computation
      -> nas_replica_sync
        -> execution_pull_verify_materialize
```

| 动作 | 执行资格的直接前置 | 必须同一的上下文 / provenance | 前置不足时 |
|---|---|---|---|
| `data_lake_read` | 无上游 action | approved release/dataset/path 与本 record scope revision/hash | 不 dereference |
| `multi_trial_runtime_and_workspace_write` | 有效 `data_lake_read` | scope revision/hash、release/run/family context 全部相同 | `eligible_to_execute=false`；runner/workspace/pointer=`0/0/0` |
| `trial_return_generation` | eligible runtime | generation 与 runtime/read chain 全部相同 | 不写第一个 artifact candidate byte |
| `empirical_R_computation` | sealed generation provenance | ordered sealed refs、same family/run/window 和历史 eligibility evidence；无 lake dereference 时不要求 read record 同时活动 | typed_unavailable/BLOCKED，不进入 estimator |
| `nas_replica_sync` | sealed generation artifact | 原 seal/manifest/logical URI/content hash/release 与 sync scope 一致 | 不创建 NAS staging、不推进 pointer |
| `execution_pull_verify_materialize` | verified replica receipt | expected release/manifest/seal/hash 与 pull scope 一致 | 不读 NAS payload、不创建 staging、不推进 cache pointer |

因此，`multi_trial_runtime_and_workspace_write=approved` 而相同 scope/release/run/family 的 `data_lake_read` 缺失、deny、过期或撤销时，runtime 的批准记录仍可保留，但 runner 不得启动、workspace 不得创建/写入、任何 pointer 不得推进。partial authorization 的权限并集始终为 `0`。

### 操作中撤销

撤销或到期后不得启动下一步或推进 pointer；当前 partial staging 保持 non-distributable/non-runtime。授权撤销不删除 immutable historical evidence。

### 当前状态

架构长期采用六动作模型；当前 CP3 运行状态为六类全部 deny，授权/执行=`0/6`。

## 8. ADR-CR172-I-006：empirical R 四态与 FU 条件式前置

### 决策

R disposition 必须显式为：

| 状态 | 使用条件 | Claim ceiling |
|---|---|---|
| `declared_exact` | fixture/golden matrix，未声明真实估计 provenance | 只能验证方法合同，不能证明 empirical evidence |
| `empirical` | sealed same-family/run trial refs、ordered IDs、common window、alignment/missing policy、method v2+hash、valid domain/PSD repair policy、独立验证和 compute auth 全部满足 | future gate 才可评估 available count/C1；本 CR 不计算 |
| `typed_unavailable` | source、授权、v2 或其他前置未提供，且无完整性冲突 | 可继续 DQ-003 降级设计；`c1_computable=false` |
| `BLOCKED` | hash/identity/window/alignment 冲突、tamper、未授权 repair/compute | 不得自动修复、重标或降级为 declared_exact |

`FU-CR173-001` 不是 PATH-C/A 降级设计的绝对前置；它是 available effective count 或 `c1_computable=true` 的硬前置。未来 PATH-C/A CP2 必须记录“完成 v2 / split future activation / DQ-003 downgrade”三选一。

## 9. ADR-CR172-I-007：新路径与 legacy 只读

### 决策

新单次研究目标合同使用 `${QUANT_LAB_RESEARCH_RUNS_ROOT}/multifactor-strategy-research/{run_id}/`，但本 CR 的 REQ-013 结论仅为 `contract_ready/runtime_enforcement_deferred`。现有 runner diff/default switch/runtime enforcement=`0/0/0`。future native-producer CR 必须在 launch/workspace first side effect 前消费 `RunPathDecisionV1`，证明 new default=`1`、legacy write=`0`；在此之前不得声明 runtime delivered。历史路径仅用于只读审计，不 move、rename、copy-recanonicalize 或 rewrite。

### 回退

若新路径兼容性有阻断，停止创建新 run 并回设计澄清；不得恢复双默认或写回 legacy。迁移只能由独立 CR 完成。

## 10. ADR-CR172-I-008：SignalBatch 只冻结 exact 8-slot boundary

### 决策

默认信号由执行机本地生成，跨机信号传输=`0`。PATH-I 只冻结 mandatory slots：

1. `schema_version`
2. `batch_id`
3. `strategy_id`
4. `strategy_package_hash`
5. `content_sha256`
6. `signature/key_id`
7. `valid_from/valid_until`
8. `sequence_no`

slots=`8/8`，extra mandatory=`0`。第 6/7 是批准的复合 slot；物理 nesting/flattening 延后。credential、account secret、broker order command 禁止。

### Deferred

mailbox physical path、七级状态机、ack、idempotency/replay、exchange/sync/transport/consumer 和 intraday 不进入当前模块、Story 或实现：数量全部为 `0`。分别衔接 `DF-CR172-SIGNAL-BATCH-EXCHANGE` 与 `DF-CR172-INTRADAY-REALTIME-SIGNAL`。

## 11. ADR-CR172-I-009：失败、恢复与声明边界

### 决策

- source/wrong-kind/unsealed：unavailable 或 BLOCKED，不进入 artifact chain。
- replica/materialization mismatch：不推进 pointer，保留上一 verified selection。
- interrupted sync/pull：partial staging non-distributable/non-runtime；重试必须重新授权并完整复验。
- empirical integrity conflict：BLOCKED；前置不足：typed_unavailable。
- PATH-I CP8 最高只允许 `path_i_design_ready=true`。
- append-only lineage 若在 finish/finalize/selection 中部分成功，已追加事件不可擦除，不得用 `fail()` 伪装回滚；输出 machine-readable `partial_lineage_blocked_audit`，trial-return canonical selection advance=`0`。

### 明确不授权 / 不声明

`stage3_entry_ready`、`c1_computable`、`real_data_authorized`、`multi_trial_runtime_authorized`、`signal_transport_authorized` 均保持 `false`。HLD/ADR/CP3 PASS 不构成 source/test implementation、目录创建、真实 lake/NAS、runtime、trial-return/R、signal、migration、trading、publish/deploy 或 Git remote write 授权。

## 12. ADR-CR172-I-010：ReturnDefinitionV1 归属与 schema 演进

### 决策

CP3 architecture owner 对 `ReturnDefinitionV1` 的业务语义、canonical schema identity 和演进规则负责。`trial_portfolio_return_series@v1` canonical payload 固定为恰好两个必填列：

1. `timestamp`
2. `simple_return`

`net_return`、`gross_return`、`nav` 都不是 v1 canonical 字段。`simple_return` 的 net/gross 含义只允许通过 manifest `return_basis` 显式声明；不得根据列名、目录、producer 默认值或历史 artifact 推断。

### CP5 权限边界

CP5 只能选择 validation/serialization library、Parquet 编码、压缩参数与 repository-local fixture 实现；不得静默增加、删除、重命名 canonical 字段，也不得改变 `simple_return` 语义。增加 `net_return`、`gross_return`、`nav` 或重定义任一 return 字段，必须先完成 versioned return-definition ADR 和 schema version bump。

若字段演进改变 owner、真实运行授权、跨 trial 可比性或 empirical-R 统计语义，则不得作为 CP5 实现细节处理，必须拆独立后续 CR。

### 备选、回退与后果

| 选项 | 优势 | 代价 | 结论 / 切换条件 |
|---|---|---|---|
| v1 恰好两列 + manifest return_basis | identity 最小、跨 trial schema 一致、现有 CP2 合同不扩张 | 无法同时存多种 return view | 推荐；PATH-I v1 固定使用 |
| v1 增加 net/gross/nav | 表面信息更丰富 | 造成 schema identity 漂移、双真相和下游误读 | 禁止在 CP5 静默采用 |
| versioned v2/new object kind | 可承载未来多 return view | 需要迁移、兼容、owner 与统计语义重新评审 | 仅有 versioned ADR + schema bump；命中高风险影响时另起 CR |

return basis 未知或跨 trial 不一致时回退 unavailable；字段/manifest 语义冲突时 BLOCKED，不允许 alias 或自动修复。

现有 `turnover.next_rebalance_date/net_forward_return` 定类为 `forward_label_proxy@v1`，不是 portfolio period return；进入 trial-return、empirical-R、effective-count 的接受数=`0/0/0`。未来真实 producer 必须显式提供区间端点、持仓/权重、交易成本和 non-overlap/alignment 语义。

## 13. ADR-CR172-I-011：真实 producer 与原子 lineage 独立前置

本 CR 不修改 `engine/mature_multifactor_research.py`，也不扩张 CR-163 lineage owner。真实 native multi-trial period-return producer/instrumentation 必须由独立 runtime-high-risk CR 承接；append-only lineage 的批次原子性、outbox 或 correction/supersession 必须由独立 lineage-owner CR 承接。在两项前置完成前，真实 trial-return source 保持 `typed_unavailable`，partial lineage 保持 `BLOCKED`。

## 14. ADR / HLD / Risk / NFR 一致性矩阵

| 决策 | HLD | Risk | NFR | 一致性 |
|---|---|---|---|---|
| ADR-001 native/import/absent | §3/§5 | source absent | 可验证性/诚实降级 | PASS |
| ADR-002 logical URI+hash | §10.3 | identity drift | identity/path `100%/0` | PASS |
| ADR-003 三段链 | §8/§9 | NAS canonical conflict | fault isolation/direct-NAS `0` | PASS |
| ADR-004 strict seal sequence | §10.2/§11 | stale/partial | integrity/atomicity | PASS |
| ADR-005 six-action auth | §10.4/§11.1 | auth gap | unauthorized `0/6` | PASS |
| ADR-005 trusted origin | §10.4 | caller self-assertion | current approved-ledger accepted/eligible=`0/0`；evidence_kind truth=`0` | PASS |
| ADR-006 empirical four-state | §11.5 | R domain mismatch | positive overclaim `0` | PASS |
| ADR-007 path/legacy | §11.6 | legacy rewrite | legacy mutation `0` | PASS |
| ADR-008 signal boundary | §10.5/§11.7 | signal ambiguity | 8/8, detailed implementation `0` | PASS |
| ADR-009 rollback/claim | §11.8/§12 | all high risks | previous selection preservation `100%` | PASS |
| ADR-010 return definition | §10.1 | return-definition drift | canonical v1 columns=`2/2`；silent field addition=`0` | PASS |
| ADR-011 current-slice narrowing | §9.4/§11.2 | source/lineage semantic escape | current runner diff=`0`；forward proxy accepted=`0`；partial lineage selection=`0` | PASS |

## 15. HLD 拆分决定

保持单份 HLD：trial-return、manifest/seal、replica/materialization、authorization 与 empirical disposition 共享同一 artifact identity 和 failure boundary。SignalBatch 只有 value-object contract boundary，独立 exchange module/Story/成功路径均为 `0`，不足以形成 companion HLD。未来 signal exchange、intraday、FU-CR173-001 或 external import 激活时必须各自新建 CR/HLD。

## 16. CP3 Decision Brief 输入

| Decision ID | 对应 ADR | 推荐 | 备选 / 回退 |
|---|---|---|---|
| CP3-DQ-CR172-I-01 | ADR-001 | ARCH-A native sealed pipeline | import-first / typed_unavailable |
| CP3-DQ-CR172-I-02 | ADR-002/003/004/009 | logical identity + strict seal/replica/materialization + pointer rollback | research-local only、distribution blocked |
| CP3-DQ-CR172-I-03 | ADR-005 | 六动作六判定点；当前全 deny | permanent fixture-only；不采用粗粒度合并 |
| CP3-DQ-CR172-I-04 | ADR-006/008 | empirical 四态/v2 条件前置 + signal 8-slot ceiling | declared-exact-only / local-signal-only |

## 17. Gotchas

- pointer 是可变选择，不是 artifact identity；回滚 pointer 不得改写 sealed bytes。
- replica receipt 不能替代 original seal；execution materialization 也不能生成新的 canonical seal。
- authorization schema 完整不等于 action 已获批；当前 6 个 action 全部 deny。
- action record 自身获批不等于动作具备执行资格；runtime 缺少同 scope data-lake read 前置时必须保持 `eligible_to_execute=false`。
- current-v1 `approved_ledger` 一律双 false；只有未来独立可信 adapter CR 才能改变。
- verifier library dependency 不等于 data-source dependency；S04 数据必须来自 S03 selection。
- REQ-013 contract ready 不等于 runtime default switch 已交付。
- `simple_return` 是 v1 唯一 canonical return 列；CP5 不得用 net/gross/nav 扩列形成未版本化双真相。
- `forward_label_proxy@v1` 即使列名为 return 也不是 trial portfolio period return；不得 seal 后改名提升。
- append-only partial lineage 是审计事实，不是可由 exception handler 回滚的事务；只能 BLOCKED 并移交 lineage owner。
- `typed_unavailable` 不能吞掉 tamper/alignment/hash 冲突；这些必须 BLOCKED。
- SignalBatch 8-slot boundary 不能被解释为 transport schema 已实现或 replay safety 已证明。

## 18. 确认记录

| 字段 | 值 |
|---|---|
| CP3 自动预检 | `process/checks/CP3-CR172-PATH-I-HLD-CONSISTENCY.result.json` |
| CP3 人工状态 | approved；用户于 2026-07-18 在三项整改 `3/3 PASS` 后批准 |
| 当前授权变化 | 无；`0/6` |
