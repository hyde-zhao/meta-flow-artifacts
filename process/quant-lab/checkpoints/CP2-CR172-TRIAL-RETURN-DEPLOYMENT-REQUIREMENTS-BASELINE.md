---
checkpoint_id: "CP2-CR172-TRIAL-RETURN-DEPLOYMENT"
checkpoint_name: "CR172 PATH-I / trial-return / deployment scope delta"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-07-17T14:38:00+08:00"
reviewed_by: "user"
reviewed_at: "2026-07-17T16:54:09+08:00"
auto_check_result: "process/checks/CP2-CR172-TRIAL-RETURN-DEPLOYMENT-REQUIREMENTS-BASELINE.result.json"
target: {phase: "requirement-clarification", story_id: "", artifacts: ["docs/product/USE-CASES.md", "docs/product/REQUIREMENTS.md", "docs/product/SCENARIOS.yaml", "docs/product/TEST-MATRIX.md", "docs/product/STORY-MAP.md", "docs/product/MVP-SCOPE.md", "docs/product/RELEASE-SLICES.md", "docs/product/BACKLOG.md"]}
---

# CP2 CR172 PATH-I / trial-return / deployment 人工审查

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP1-CR172-TRIAL-RETURN-DEPLOYMENT-USE-CASE-COMPLETENESS.result.json` | PASS | 0 | CR172 场景 27/27。 |
| `process/checks/CP2-CR172-TRIAL-RETURN-DEPLOYMENT-REQUIREMENTS-BASELINE.result.json` | PASS | 0 | 15/15；0 waiver。 |

## Decision Brief

### 审批者摘要

| 字段 | 内容 |
|---|---|
| 本次确认服务的整体目标 | 在 prior-approved PATH-B 历史上，建立 PATH-I trial-return、长期 identity、“研究机本地 active canonical → NAS verified replica/backup/distribution → 执行机按需 pull+verify+本地 immutable cache”的部署合同；信号仅冻结默认本地生成、低频 8 字段边界与 intraday 独立 CR，不进入详细传输设计。 |
| 推荐动作 | `approve` 新增 DQ-009~015 的推荐方案及三项评审边界：CP3 只设计；DQ-015 详细 path/state/ack/idempotency deferred；`FU-CR173-001` 只阻断 positive empirical result，不阻止 DQ-003 降级设计。 |
| approve 后会发生什么 | 进入 CP3，由 meta-se 冻结 PATH-I 蓝图/HLD/ADR；不创建目录、不改 runner、不写 NAS、不跑 multi-trial、不传信号。 |
| approve 不授权什么 | 真实 lake/NAS 读写或同步、package pull、multi-trial runtime、trial-return/R generation、signal generation/transport、migration、trading、publish/deploy 或 Git remote write。 |
| 不确认会阻塞什么 | PATH-I 设计、trial-return source/deployment/signal contract 与所有未来 PATH-C/A activation。 |

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP2-CR172-TRIAL-RETURN-DEPLOYMENT-CONTEXT.yaml` |
| capsule 状态 / read_profile | `ready` / `compact` |
| 默认读取策略 | capsule-first；只读 allowed_reads。 |
| 全文档读取扩展 | meta-pm 本轮 `0`；使用 capsule 允许的产品文档。 |
| 缺失 / waived | 无。 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `process/state/STATE.current.json` | scanned | 7 | 7 | scope/runtime/deployment/signal boundary |
| 委托 Agent 交还摘要 | correction R1 + `process/handoffs/CR172-PATH-I-REVIEW-CORRECTION-R2-RETURN-SUMMARY.md` | scanned | 7 | 7 | R2 已收缩 DQ-015 并强化 DQ-014 follow-up 语义，不新增 DQ |
| 自动预检结果 | CP1/CP2 result JSON | scanned | 7 | 7 | PASS，交人工确认 |
| discussion log / checkpoint | N/A | n/a | 0 | 0 | correction R1 已将部署主权与信号频率意图写入 SGQ-CR172-004/005；正式 gate 由本文件承接 |
| 下游正式产物 | HLD/LLD/REVIEW | n/a | 0 | 0 | CP2 前禁止生成 |
| 用户显式输入 | correction handoff / delta summary / PATH-I 评审意见 | scanned | 10 | 7 | 7 个 DQ 保留；三项评审建议作为 DQ-014/015 与阶段守卫修订，不另造 DQ |

### 决策分层

| 分类 | 数量 | 处理方式 |
|---|---:|---|
| 必须用户决策 | 7 | DQ-009~015 |
| 高风险策略确认 | 4 | DQ-010/011/012/015；不等于运行或信号传输授权 |
| agent 默认处理 | 0 | 无静默扩大项 |
| 仅审计记录 | 8 | prior-approved DQ-001~008 / PATH-B 历史 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CR172-009 | scope | trial-return source 与对象 | future native multi-trial per-trial portfolio return；source 缺失/错型 fail-closed | 外部预计算 import contract；或保持 absent | native 方案 lineage 最完整，但需要后续 instrumentation/runtime gate；import 更快但 provenance 较弱；absent 最保守。 | 影响 trial-return schema、seal/hash/URI 和 CR-163 lineage 引用边界；source absent 会阻断 C1 正向结果。 | native 不可行转 import；provenance 不足回 absent。 |
| DQ-CR172-010 | architecture | 单一 canonical、NAS replica 与 stable identity 如何落地 | 研究机本地 active canonical；NAS 仅 verified replica/backup/distribution；执行机按需 pull 到临时目录并校验 release/manifest/hash 后原子物化本地 immutable cache；logical URI + content hash | NAS 不可用时保留研究机 canonical 并阻塞分发；或 storage blocked | 推荐方案保持研究主权、运行时隔离和可追溯性；NAS 故障只影响分发。备选更保守但不可跨机消费。 | replica 陈旧、partial 或 hash 不符时可能产生错误消费；运行时直读 NAS 会扩大故障域。 | 任一校验失败即 fail-closed；不得切换为运行时直读 NAS。 |
| DQ-CR172-011 | runtime_authorization | multi-trial 与跨机数据动作如何授权 | `data_lake_read`、`multi_trial_runtime_and_workspace_write`、`trial_return_generation`、`empirical_R_computation`、`nas_replica_sync`、`execution_pull_verify_materialize` 六类动作 6/6 分离 | 合并 runtime+generation；或全不授权 | 分离方案 blast radius 最小且可逐项撤销，但门禁较多；合并较简洁却削弱审计；全不授权只能 fixture。 | 影响研究机→NAS→执行机完整链路；任何授权合并都可能把数据读、runtime 与写入风险混为一体。 | 授权系统不能表达时先全不授权，CP3 只提出最小合并候选；缺 owner/scope/hash/撤销条件的动作 fail-closed。 |
| DQ-CR172-012 | security | 四组件 ownership/data flow | 研究机生产并持有 canonical、NAS 仅分区 replica/backup/distribution、GitHub metadata-only、执行机仅拉取 approved package 并使用本地 immutable cache | 取消 NAS 分发；或执行机离线预置已验证 package | 推荐方案兼顾最小泄漏与可分发性；离线预置更安全但运维效率更低。 | 扩大 GitHub 或执行机数据面会引入数据泄漏和 runtime 依赖；执行机直读研究机目录被禁止。 | 只有独立安全/合规 CR 证明必要，才评估扩大执行机或 GitHub 数据面。 |
| DQ-CR172-013 | scope | 新旧运行路径 | 新 run 使用 `multifactor-strategy-research`；legacy 只读不迁移 | 继续旧路径；或一次性 migration/rewrite | 新路径语义清晰且保留历史；旧路径继续误导；迁移可统一命名但身份与回滚成本高。 | 影响 runner 默认值、artifact identity、历史可读性和回滚；CP3 提前切换会形成双默认路径。 | 仅独立 migration CR 具备 inventory/hash/rollback/授权后切换。 |
| DQ-CR172-014 | risk_acceptance | empirical R 缺证据/授权时处理 | declared-exact/empirical 分类；重开 PATH-C/A 时显式三选一：完成 `FU-CR173-001`、拆 future activation CR、或 DQ-003 typed_unavailable 降级。v2 不阻止降级设计，但阻断 positive empirical effective count / `c1_computable=true` | 永久 declared-exact-only；或保持所有 empirical 输入 unavailable | 推荐方案保留方法演进与诚实降级；硬阻断所有 PATH-C/A 会违反 DQ-003，静默复用 v1 则越过有效域。 | 错把 empirical 重标为 declared_exact 会产生不可审计正向结果；未完成 v2 时不得声明 effective count available。 | v2 owner/证据未就绪时只能拆 future activation 或 DQ-003 降级；任何 empirical→declared_exact 重标记均 BLOCKED。 |
| DQ-CR172-015 | architecture | 信号在哪里生成、PATH-I 冻结到什么粒度 | 默认执行机本地生成；低频/EOD 只冻结 optional immutable batch 与精确 8 字段：schema_version、batch_id、strategy_id、strategy_package_hash、content_sha256、signature/key_id、validity window、sequence_no；intraday 独立 CR。物理路径、七级状态机、ack/idempotency/replay 全部 deferred | 全部执行机本地生成且不声明 batch；或把低频 detailed exchange 提升为独立后续 CR | 推荐方案保留最小扩展边界且防止 PATH-I 跨入 trading owner/Stage 4/5；本地-only 最安全；独立 CR 可完整设计但增加审批。 | detailed transport、ack/replay 或 intraday 混入会扩大 owner、运行授权和交易风险；PATH-I signal Story 必须为 0。 | detailed transport 进入 `DF-CR172-SIGNAL-BATCH-EXCHANGE`；intraday 进入 `DF-CR172-INTRADAY-REALTIME-SIGNAL`。 |

### 用户需决策事项 summary

| 字段 | 内容 |
|---|---|
| 本轮待人工决策项 | 7 |
| 必须用户决策 | `DQ-CR172-009`、`DQ-CR172-010`、`DQ-CR172-011`、`DQ-CR172-012`、`DQ-CR172-013`、`DQ-CR172-014`、`DQ-CR172-015` |
| 推荐回复 | `approve` |
| 如果你回复 approve | 表示接受上表 7 项推荐方案和三个强制边界，只解锁 CP3 solution design。 |
| 不表示授权 | 不授权实现、目录创建、真实 lake/NAS、sync/pull/materialize、multi-trial runtime、trial-return/R 生成、SignalBatch 传输、交易、迁移、发布部署或 Git remote write。 |
| 修改: <具体修改点> | 可按一个或多个 DQ ID 修改；未修改项继续按推荐方案。 |
| reject | 拒绝本轮 scope delta，CR-172 保持 blocked/cp2_pending，不进入 CP3。 |

完整需求级成功标准见 `docs/product/REQUIREMENTS.md#scope-delta-cp2-待决-dq`。回复 `approve` 只接受以上 7 项推荐，不改变 prior-approved DQ-001~008，也不授权任何真实 sync/pull/signal/runtime 操作。

### 评审收敛后的三个强制边界

1. **CP3 design-only**：只冻结 BLUEPRINT/HLD/ADR/部署合同；目录创建、runner 默认值切换、NAS/执行机操作和任何代码实现均为 `0`。
2. **DQ-015 scope split**：PATH-I 只保留边界声明与 `8/8` 最小字段；mailbox 物理路径、七级状态机、ack/idempotency/replay 和日内传输不得进入当前 Story、LLD 或实现。
3. **Empirical positive-claim prerequisite**：`FU-CR173-001` 是 empirical effective count available / `c1_computable=true` 的硬前置，不是选择 DQ-003 降级后重开 PATH-C/A 设计的硬前置。

后续阶段守卫：CP5 未来若获批，也仅允许 repository-local code/test/fixture；CP7 六类真实动作计数必须为 `0/6`；CP8 最高 `path_i_design_ready=true`，同时 `stage3_entry_ready/c1_computable/real_data_authorized/multi_trial_runtime_authorized/signal_transport_authorized` 全为 `false`。

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| CP1/CP2 自动预检 PASS | 通过 | 上述 result JSON | PASS；blocker=0、waiver=0。 |
| 8 个产品文档增量基线 | 通过 | `docs/product/*` | 8/8 已完成 correction R2。 |
| DQ-009~015 结构完整 | 通过 | 本 Decision Brief | 用户接受 7/7 推荐值和三个强制边界。 |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | prior PATH-B 历史保留 | 通过 | REQUIREMENTS / SCENARIOS | 旧基线未删除。 |
| 2 | PATH-I source/deployment/URI 方案 | 通过 | DQ-009/010 | 接受推荐值。 |
| 3 | 六动作授权与四组件边界 | 通过 | DQ-011/012 | 接受 6/6 独立授权分类；本轮真实授权 0/6。 |
| 4 | 新路径与 empirical-R fail-closed | 通过 | DQ-013/014 | 接受新路径合同与 empirical 三路线。 |
| 5 | signal 本地默认、低频 immutable batch 仅边界/8 字段与 intraday 独立 CR；详细 exchange 为 deferred | 通过 | DQ-015 | 接受边界收缩；PATH-I signal Story=0。 |
| 6 | approve 不构成真实授权 | 通过 | MVP / TEST-MATRIX | 用户批准只解锁 CP3 设计。 |
| 7 | CP3 design-only、CP5 repository-local、CP7 六动作 0/6、CP8 claim ceiling | 通过 | 三个强制边界 / MVP / TEST-MATRIX | 阶段守卫生效。 |
| 8 | `FU-CR173-001` 仅阻断 positive empirical result，不破坏 DQ-003 降级 | 通过 | DQ-014 / CR-173 follow-up tracking | 接受条件式硬前置语义。 |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| 7 项 scope delta 全部决定 | 通过 | 人工审查结果 | DQ-009~015 7/7 approved。 |
| 无隐含 sync/pull/signal/runtime/migration/Git write 授权 | 通过 | 不授权项 | operation count 保持 0。 |
| 三个评审边界被接受且无 PATH-I scope inflation | 通过 | Decision Brief / DQ-014/015 | correction R2 被整体接受。 |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| 产品基线 8 文档 | `docs/product/` | 通过 | 8/8。 |
| 自动结果 | `process/checks/CP2-CR172-TRIAL-RETURN-DEPLOYMENT-REQUIREMENTS-BASELINE.result.json` | 通过 | PASS。 |
| 人工 Decision Brief | 本文件 | 通过 | 用户已批准。 |

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-07-17T16:54:09+08:00
- 修改意见：无；接受 DQ-009~015 推荐值与 correction R2 三个强制边界。
- 风险接受项：接受 PATH-I 仅设计/fixture 边界；`FU-CR173-001` 只阻断 positive empirical result；SignalBatch detailed exchange 与 intraday 均 deferred。
- 已接受决策项：`DQ-CR172-009`、`DQ-CR172-010`、`DQ-CR172-011`、`DQ-CR172-012`、`DQ-CR172-013`、`DQ-CR172-014`、`DQ-CR172-015`
- 授权上限：只解锁 CP3 solution design；实现、目录创建、真实 lake/NAS、sync/pull、runtime、trial-return/R、signal transport、migration、trading、publish/deploy 和 Git remote write 仍为未授权。
