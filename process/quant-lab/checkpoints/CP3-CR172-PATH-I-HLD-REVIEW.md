---
checkpoint: CP3
checkpoint_id: CP3-CR172-PATH-I-HLD-REVIEW
cr_id: CR-172
title: CR-172 PATH-I Trial-Return 与跨机部署合同 — HLD Review
status: approved
gate_profile: runtime-high-risk
opened_at: "2026-07-17T17:31:00+08:00"
reviewed_by: "user"
reviewed_at: "2026-07-18T10:10:02+08:00"
auto_check_result: process/checks/CP3-CR172-PATH-I-HLD-CONSISTENCY.result.json
correction_result_ref: process/checks/CP3-CR172-PATH-I-HLD-REVIEW-CORRECTION-R1.result.json
context_ref: process/context/CP3-CR172-PATH-I-DESIGN-CONTEXT.yaml
route_plan_ref: process/checks/CP0-CR172.route-plan.json
target:
  phase: solution-design
  artifacts:
    - docs/design/BLUEPRINT.md
    - docs/design/DOMAIN-MAP.md
    - docs/design/DEPENDENCY-MAP.md
    - docs/design/HLD-TRIAL-RETURN-DEPLOYMENT-CONTRACTS.md
    - docs/design/ARCHITECTURE-DECISION-TRIAL-RETURN-DEPLOYMENT-CONTRACTS.md
---

# CP3 CR-172 PATH-I Trial-Return 与跨机部署合同 — HLD Review

## 自动预检摘要

| 预检文件 | 结论 | 通过项 | 阻断项 | Waiver | 说明 |
|---|---:|---:|---:|---:|---|
| `process/checks/CP3-CR172-PATH-I-HLD-CONSISTENCY.result.json` | PASS | `18/18` | `0` | `0` | 4 个 Gray Areas、3 套候选、REQ `7/7`、UC `1/1`、simulation `5/5`、蓝图三件套 `3/3`、ADR `9/9` 均通过。 |
| `process/checks/CP3-CR172-PATH-I-HLD-REVIEW-CORRECTION-R1.result.json` | PASS | `3/3` | `0` | `0` | return-definition ADR 归属、CP5 native producer 插桩义务、六类授权执行依赖 DAG 全部关闭；原 4 个 DQ 和范围不变。 |
| Host correlation | PASS | `4/4` | `0` | `0` | route plan、真实 dispatch、discussion evidence、read-expansion event 均已关联；`meta-flow cp result-check --check-consistency` 通过。 |

## Decision Brief

### 审批者摘要

| 字段 | 内容 |
|---|---|
| 本次确认服务的整体目标 | 确认 PATH-I 是否采用一条可审计、fail-closed、逐动作授权的 trial-return sealed artifact chain，并把研究机、NAS、执行机、GitHub 四组件边界冻结为后续 repository-local fixture 实现的设计输入。 |
| 推荐动作 | `approve` 下方 4 项推荐：ARCH-A native sealed pipeline；严格 manifest/seal/replica/materialization 与 pointer-only rollback；六动作六判定点；empirical 四态/FU 条件前置与 SignalBatch 8-slot ceiling。 |
| approve 后会发生什么 | Host 自动进入 story-planning；meta-se 生成 FEATURE-DESIGN-MATRIX、必要 Feature 实现设计与 Story/DAG，CP4 自动预检后停在 CP5 全量设计证据人工门禁。 |
| approve 不授权什么 | 不授权 Story 实现、不创建真实运行目录、不修改 runner 默认值、不读写真实 lake/NAS、不运行 multi-trial、不生成 trial-return/R/SignalBatch、不迁移、不交易、不发布/部署、不执行 Git remote write；六类真实动作继续 `0/6`。 |
| 本轮最高正向声明 | `path_i_hld_review_ready=true`；`path_i_design_ready=false`，后者只可能在未来 CP8 有完整 evidence 后成立。 |
| 不确认会阻塞什么 | 阻塞 PATH-I Feature/Story 设计、CP4 和 CP5；CR-172 不进入实现，PATH-C/A 也不会自动恢复。 |

### Context Capsule Summary

| 项 | 内容 |
|---|---|
| capsule | `process/context/CP3-CR172-PATH-I-DESIGN-CONTEXT.yaml`，状态 `ready_for_human_gate`。 |
| read_profile | `compact` + `deep_review`；capsule-first。 |
| 默认读取策略 | 先读 capsule 的 `must_read`，只在架构冻结、冲突核验或人工深审时展开 `allowed_reads/read_if_needed`。 |
| 全文档读取 | 只展开 CP2 approved 基线、CR-172 产品段落、三份长期设计索引和本轮 discussion；扩读事件为 `RE-20260717T092611Z0000-99b4ad72`。 |
| 实际调度 | canonical role=`meta-se`、agent=`meta-se-critical`、reasoning=`critical`；dispatch `AD-CR172-CP3-META-SE-CRITICAL-20260717T170547+0800` 已完成。 |
| 设计产物 | 长期蓝图增量 `3/3`；CR-specific HLD/ADR `1/1`；discussion/result/return 齐备。 |
| 实现与真实操作 | Story/LLD/source/test/fixture=`0/0/0/0/0`；真实六动作 authorized/executed=`0/6`、`0/6`。 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| CP2 已批准产品基线 | DQ/REQ-CR172-009～015 | scanned | `7` | `0` | 冻结输入，不重开 CP2。 |
| Architecture Gray Areas | AGA-CR172-I-01～04 | scanned | `4` | `4` | source/拓扑/授权/empirical disposition 均需人工确认。 |
| 候选架构 | ARCH-A/B/C | scanned | `3` | `1` bundle | 目标态、条件备选和安全回退合并到 DQ-I-01。 |
| HLD / ADR | HLD 20 节、ADR 9 项 | scanned | `9` ADR | `4` bundle | identity/拓扑/顺序/rollback 合并；empirical 与 signal ceiling 合并为边界确认。 |
| 自动预检 | CP3 result 18 items | scanned | `4` | `4` | `18/18 PASS`；等待人工接受推荐与风险。 |
| Story/LLD/实现 | 下游阶段 | N/A | `0` | `0` | CP3 批准前禁止创建。 |
| Deferred candidates | signal exchange、intraday、FU-CR173-001、external import、migration | recorded | `5` | `0` | 不在本轮激活或实现。 |

### 决策分层

| 层级 | 数量 | 项目 |
|---|---:|---|
| 必须用户决策 | `4` | `CP3-DQ-CR172-I-01`～`04`。 |
| 高风险策略确认 | `3` | NAS 不能成为 runtime canonical；六动作不做权限并集；pre-v2 不得产生 available effective count/C1 true。 |
| agent 默认处理 | `5` | 文档命名、typed receipt 命名、immutable-version/pointer 分层、refs-only evidence、CP5 再选序列化实现库。 |
| 仅审计记录 | `5` | deferred signal exchange、intraday、empirical v2、external import、legacy migration。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| `CP3-DQ-CR172-I-01` | architecture | 是否确认 native sealed artifact pipeline 为 PATH-I 目标态，并保留 import/absent 两级回退？ | `ARCH-A`：future native multi-trial producer 输出 per-trial payload，经 research-local validate/manifest/seal 成为 active canonical。 | `ARCH-B`：external import 先进入 quarantine，需独立 provenance contract；`ARCH-C`：source absent，保持 `typed_unavailable`。 | ARCH-A lineage 最完整、owner/identity 最清晰，但后续插桩和 fixture 设计量最大；B 可绕过 native producer 缺口但增加外部 provenance/security 面；C 风险最低但没有 positive empirical 价值。 | 影响 source owner、future Feature/Story、schema、lineage 与 C1 上游。主要风险是把合同误写成 producer 已实现，或让错误对象冒充 trial-return。 | CP4/CP5 证明 native producer 不可插桩时切 B；外部 provenance、owner 或授权不足时切 C。 |
| `CP3-DQ-CR172-I-02` | architecture | 是否确认 logical URI+hash、严格 payload→manifest→seal→local selection→replica verify→materialize verify 顺序和 pointer-only rollback？ | stable logical URI + payload SHA-256；research-local 唯一 canonical；NAS verified replica；execution local immutable cache；所有 pointer 仅在本层验证完成后原子推进。 | research-local only、distribution/materialization blocked。NAS runtime canonical 或 direct-NAS read 不是备选。 | 推荐方案提供跨主机稳定身份、故障隔离和可回滚性，代价是 manifest/seal/receipt/staging 合同较多；保守备选减少跨机价值但不扩大共享盘故障域。 | 主要风险是 stale/partial/hash mismatch 被消费、绝对路径进入 identity、NAS 被提升为 canonical 或回滚改写 sealed bytes。 | manifest/seal/hash/freshness/atomic pointer 任一能力不能证明时回退 distribution blocked；不得切为 direct-NAS runtime。 |
| `CP3-DQ-CR172-I-03` | security | 是否确认六类真实动作必须使用六个独立 authorization envelope/判定点，partial approval 不产生权限并集，mid-operation revoke 不推进 pointer？ | `data_lake_read`、`multi_trial_runtime_and_workspace_write`、`trial_return_generation`、`empirical_R_computation`、`nas_replica_sync`、`execution_pull_verify_materialize` 各自 owner/scope/hash/path/expiry/revoke/evidence；当前全 deny。 | 永久 fixture-only；不采用 runtime+generation 或 sync+pull 粗粒度合并。 | 推荐 blast radius、撤销和事故归因最小，但门禁与测试数量更多；全 deny 最安全但无法进入真实链路。 | 主要风险是一次批准隐含其他动作、撤销后继续 commit pointer、partial staging 被分发/运行。 | 授权系统不能安全表达 `6/6` 时维持 `0/6`，回到设计澄清；不能以粗粒度合并绕开。 |
| `CP3-DQ-CR172-I-04` | risk_acceptance | 是否确认 empirical 四态/FU-CR173-001 条件式前置，同时把 SignalBatch 严格限制为 8-slot boundary？ | R 显式为 `declared_exact/empirical/typed_unavailable/BLOCKED`；DQ-003 降级可设计，positive available count/C1 true 必须先完成 FU-CR173-001；signal 默认 execution-local，只冻结 exact `8/8` slots。 | declared-exact-only / permanent typed_unavailable；或完全 local-signal-only、不保留 batch boundary。 | 推荐保留演进路线且防 sampling-error overclaim，代价是 claim guard/provenance 字段多；备选更保守但降低后续价值。 | 主要风险是 declared-exact 静默重标 empirical、完整性冲突被吞成 unavailable、8-slot boundary 膨胀为 mailbox/ack/replay/intraday 实现。 | source/auth/v2 缺失且无冲突时 `typed_unavailable`；hash/identity/window/alignment/tamper/unauthorized repair 为 `BLOCKED`；详细 signal 或 intraday 必须独立 CR/HLD。 |

### 需要知情接受的设计边界

| 边界 | 当前结论 | 后续影响 |
|---|---|---|
| PATH-I 解决什么 | 解决“如何真实地产生、封存、复制、物化、授权和分类 trial-return artifact”的设计/fixture 合同。 | 不等于真实 source、真实 R 或 C1 已可用。 |
| FU-CR173-001 | 只硬阻断 positive empirical effective count 和 `c1_computable=true`。 | DQ-003 `typed_unavailable` 降级设计仍可继续。 |
| Signal | 默认由执行机本地生成；当前只保留低频/EOD immutable batch 的 8-slot 边界。 | mailbox path、7 级状态、ack、idempotency/replay、exchange、consumer、intraday 均另起 CR。 |
| 新旧路径 | 新 run 合同使用 `multifactor-strategy-research/{run_id}`；legacy 只读。 | 本轮不改 runner，不迁移历史；未来兼容失败只能停新 run 或独立 migration CR。 |
| PATH-I 完成后的 CR-172 | 不自动选择或恢复 PATH-C/A。 | 仍需 five fields `5/5`、owner、授权、fresh precheck 和重开 CP2。 |

### 评审整改 R1（已关闭）

| 整改项 | 批准后的合同 | 结果 |
|---|---|---|
| return-definition ADR 归属 | `ADR-CR172-I-010` 由 CP3 architecture owner 冻结；v1 canonical payload 恰好为 `timestamp`、`simple_return` 两列，字段演进必须 versioned ADR + schema bump。 | PASS |
| native producer 插桩位置 | 具体文件不在 CP3 猜测；`DO-CR172-CP5-001` 要求 CP5 证明唯一 integration point、source/file owner、调用/授权/失败/测试/merge/rollback 合同。 | PASS |
| 六类授权依赖顺序 | 授权记录继续独立审批/撤销；执行资格按 5 条 DAG 边校验，本动作授权不替代直接前置。runtime 自身获批但无同 scope `data_lake_read` 时 runner/workspace/pointer=`0/0/0`。 | PASS |

### 用户需决策事项

| 字段 | 内容 |
|---|---|
| 本轮待人工决策项 | `4`：`CP3-DQ-CR172-I-01`、`CP3-DQ-CR172-I-02`、`CP3-DQ-CR172-I-03`、`CP3-DQ-CR172-I-04`。 |
| 推荐回复 | `approve` 或 `批准`，表示同时接受 4 项推荐及上述设计边界。 |
| approve 后下一人工门禁 | CP5；Host 会自动完成 story-planning、CP4 自动预检和全量设计证据准备，然后停在 CP5。 |
| 修改 | `修改: CP3-DQ-CR172-I-01=<具体修改>`；可一次列多项。 |
| reject | CR-172 保持 CP3，不创建 Story、不实现；PATH-I 与 PATH-C/A 均不继续。 |

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| CP2 已人工批准 | PASS | CP2 checkpoint + Gate Ledger | DQ/REQ-009～015 `7/7` 已冻结；本轮不重开。 |
| CP3 context capsule 完整 | PASS | `process/context/CP3-CR172-PATH-I-DESIGN-CONTEXT.yaml` | capsule-first；设计输出和 result 已回填。 |
| meta-se-critical 真实调度完成 | PASS | handoff + dispatch/handoff ledgers + return | canonical role/model profile/agent ID 可追溯。 |
| Architecture Gray Areas / advisor discussion 完成 | PASS | discussion log/checkpoint | `4/4` Gray Areas、3 candidates、4 DQ；无伪造 reviewer。 |
| HLD/ADR/蓝图三件套齐备 | PASS | Deliverables | `5/5` 设计入口齐备。 |
| CP3 自动预检无 blocker | PASS | CP3 result | `18/18 PASS`、blocker/waiver=`0/0`。 |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---:|---|---|---|---|
| 1 | 目标与成功标准量化 | PASS | HLD §1.3～1.4 | REQ 7/7、fields 9/9、components 4/4、actions 6/6、signal 8/8、forbidden=0。 |
| 2 | 至少两套真实候选与切换条件 | PASS | HLD §3、discussion §3/§5 | ARCH-A/B 是真实候选；C 是安全回退。 |
| 3 | HLD 拆分原则已应用 | PASS | discussion §6、ADR §13 | 单 HLD；future signal/v2/import/migration 分别独立 CR/HLD。 |
| 4 | Trial-return source 与相邻对象边界清楚 | PASS | HLD §1.6/§10.1 | CR-163 metadata/ref、layered returns、scalar 和 declared R 均不能冒充 source。 |
| 5 | Payload/manifest/seal/identity 合同完整 | PASS | HLD §10.1～10.3 | `timestamp/simple_return`、logical URI/hash、seal 顺序和复验边界齐备。 |
| 6 | 四组件 owner 与单向数据流 | PASS | HLD §8/§9.3；Blueprint/Dependency Map | research canonical 唯一；NAS replica；execution local cache；GitHub metadata-only。 |
| 7 | 六类授权与撤销边界可操作 | PASS | HLD §10.4/§11.1；ADR-005 | action/owner/scope/hash/path/expiry/revoke/evidence 和 enforcement points `6/6`。 |
| 8 | Empirical classification 与 FU 前置无矛盾 | PASS | HLD §11.5；ADR-006 | DQ-003 可降级；positive claim 才硬依赖 v2。 |
| 9 | Signal scope 被约束到 8-slot boundary | PASS | HLD §10.5/§11.7；ADR-008 | detailed module/Story/implementation=`0/0/0`。 |
| 10 | 新旧路径和迁移边界明确 | PASS | HLD §11.6；ADR-007 | 新 default 合同唯一；legacy mutation=`0`。 |
| 11 | 失败、恢复与回滚可操作 | PASS | HLD §11.8；ADR-009 | immutable versions + pointer-only rollback；partial staging 非 runtime/distributable。 |
| 12 | NFR/Risk/ADR/HLD 内部一致 | PASS | HLD §12～§14；ADR §12 | ADR `9/9`，contradiction=`0`。 |
| 13 | Use Case/Requirement trace 完整 | PASS | HLD §6 | UC `1/1`、REQ `7/7`。 |
| 14 | 关键场景模拟覆盖正常/异常/授权/降级/信号 | PASS | HLD §7 | `5/5 PASS`。 |
| 15 | 初审整改闭环 | PASS | CP3 result item 16 | policy input、payload columns、Mermaid DENY `3/3` 已修复。 |
| 16 | Claim ceiling 与真实操作守卫 | PASS | HLD §1.5/§20；CP3 result | path_i_design_ready 与五个高阶 flag 均 false；真实操作全 0。 |
| 17 | Story/LLD/实现未提前发生 | PASS | result real_operation_counts | Story/Wave/LLD/source/test/fixture=`0`。 |
| 18 | 人工确认 4 项推荐 | PASS | 本 Decision Brief + Gate Ledger | 用户以“整改完成后继续推进项目”接受原 4 项推荐。 |
| 19 | return-definition ADR 归属已闭环 | PASS | HLD v1.1；ADR-CR172-I-010；correction result | v1 恰好两列；CP5 静默扩列=`0`。 |
| 20 | native producer CP5 设计义务已闭环 | PASS | HLD `DO-CR172-CP5-001`；correction result | CP5 不能证明唯一安全插桩点时不得实现。 |
| 21 | 六类授权执行依赖 DAG 已闭环 | PASS | HLD/ADR v1.1；SIM-CR172-I-06；correction result | DAG `5/5`；permission union=`0`。 |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| CP3 auto precheck PASS | PASS | CP3 result | `18/18`、0 blocker、0 waiver。 |
| 4 个 CP3 决策完整呈现 | PASS | Decision Brief | 推荐、备选、优劣、风险、回退齐备。 |
| HLD 可馈入 story-planning | PASS | HLD/ADR/Blueprint | 仅待人工接受设计。 |
| Deferred scope 不会进入当前 Story | PASS | HLD §1.6/§16/§19 | signal exchange/intraday/v2/import/migration 保持独立。 |
| Unauthorized scope 明示 | PASS | 本 checkpoint + HLD claim ceiling | 六动作仍 0/6；无真实运行授权。 |
| 人工批准 | PASS | 人工审查结果 | 仅解锁 Story/Feature/LLD 与 CP4/CP5 设计准备；CP5 前不得实现。 |

## Deliverables

| 交付物 | 路径 | 审查结果 | 说明 |
|---|---|---|---|
| CP3 context | `process/context/CP3-CR172-PATH-I-DESIGN-CONTEXT.yaml` | PASS | compact/deep-review capsule，已关联最终设计。 |
| Blueprint | `docs/design/BLUEPRINT.md#cr-172-增量path-i-trial-return-与跨机部署合同` | PASS | 5 capabilities、3 Feature、6 flows、5 shared contracts。 |
| Domain Map | `docs/design/DOMAIN-MAP.md#cr-172-增量path-i-trial-return-与部署领域模型` | PASS | 12 objects、6 state machines、12 rules。 |
| Dependency Map | `docs/design/DEPENDENCY-MAP.md#cr-172-增量path-i-trial-return-与部署依赖边界` | PASS | 单向依赖、11 forbidden edges、5 cycles eliminated。 |
| HLD | `docs/design/HLD-TRIAL-RETURN-DEPLOYMENT-CONTRACTS.md` | PASS | 20 节、3 candidates、7/7 trace、5 simulations。 |
| ADR | `docs/design/ARCHITECTURE-DECISION-TRIAL-RETURN-DEPLOYMENT-CONTRACTS.md` | PASS | 9 proposed decisions 与一致性矩阵。 |
| Discussion | `process/discussions/CP3-CR172-PATH-I-HLD-DISCUSSION-LOG.md` | PASS | 4 Gray Areas、table-first advisor、4 DQ。 |
| Discussion checkpoint | `process/checks/CP3-CR172-PATH-I-DISCUSSION-CHECKPOINT.json` | PASS | 可恢复结构化决策。 |
| CP3 result | `process/checks/CP3-CR172-PATH-I-HLD-CONSISTENCY.result.json` | PASS | 18/18、decision PASS。 |
| CP3 correction result | `process/checks/CP3-CR172-PATH-I-HLD-REVIEW-CORRECTION-R1.result.json` | PASS | 3/3、decision PASS；scope expansion=0。 |
| meta-se return | `process/handoffs/CR172-CP3-META-SE-RETURN-SUMMARY.md` | PASS | 调度完成与零操作回执。 |
| Human gate launch | `process/checks/CP3-CR172-PATH-I-HUMAN-GATE-LAUNCH-MESSAGE.md` | PENDING | 由 Host 生成并校验。 |

## 人工审查结果

- 状态：`approved`
- 审查人：user
- 审查时间：2026-07-18T10:10:02+08:00
- 决定：批准原 `CP3-DQ-CR172-I-01～04` 推荐，并接受三项评审整改后的合同。
- 风险接受：接受 PATH-I 只形成 repository-local fixture 设计/实现前置、真实六动作保持 `0/6`、C1/Stage 3 继续 false 的边界。
- 用户指令：`这个三个点整改完成后继续推进项目`。
