---
status: draft
version: "1.2"
active_change_ref: "CR-046"
source_story_map: "docs/product/STORY-MAP.md@1.3"
source_mvp_scope: "docs/product/MVP-SCOPE.md@1.3"
confirmed: false
confirmed_by: ""
confirmed_at: ""
---

# CR-046 Evidence Integrity Blueprint

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-12 | meta-se-critical | 初始 CR-046 蓝图；冻结 evidence graph、routing compatibility、compaction、correction、audit/replay/telemetry 与 CR-163 pilot 的能力边界。 |
| 1.1 | 2026-07-12 | meta-se-critical | CP3 changes-requested 增量：加入 Codex custom-agent selector/config/discovery/receipt 四段式证明链、fail-closed、Story 2/6 owner 与 CP0/CP6/CP7/CP8 消费边界。 |
| 1.2 | 2026-07-12 | meta-se / host-orchestrator | CP3 R3：接入规范性 Codex Subagent Platform Contract、D0-D3 discovery、不可变 ThreadRuntimeIdentity、followup reuse/new-spawn escalation、required/preferred 和 A-baseline/Conditional-B dogfooding。 |

## 1. 蓝图适用性

判定为 `required`。七个 Story 共享 event/result/state/routing 的身份与真相源契约，且涉及跨模块写入所有权、append-only correction、compaction 语义和跨仓 pilot 边界；若不先冻结蓝图，Story 级实现会产生第二套 ledger、身份混淆或越权写入。

## 2. 能力地图

| Capability ID | 能力域 | 用户价值 | 覆盖 Story | Owner Feature |
|---|---|---|---|---|
| CAP-EI-01 | Chronology and gate truth | 拒绝提前 gate、回填式批准和未满足条件的推进 | ST-EI-001 | FEAT-EI-CORE |
| CAP-EI-02 | Dispatch attestation and attempt lifecycle | 区分真实平台证明、session observation 与仓库可验证性 | ST-EI-002 | FEAT-EI-CORE |
| CAP-EI-03 | CP attempt correlation and cross-truth consistency | 将 result、checkpoint、gate、state、ledger 关联到同一最终 attempt | ST-EI-003 | FEAT-EI-CORE |
| CAP-EI-04 | State/routing/finalization integrity | 让中间态、人工门、delivered 和 route metadata 使用共享真相 | ST-EI-004 | FEAT-EI-GOVERNANCE |
| CAP-EI-05 | Token telemetry | 诚实区分 measured、proxy/estimated、unavailable | ST-EI-005 | FEAT-EI-OBSERVABILITY |
| CAP-EI-06 | Checker replay and generated audit | 同时保留 as-executed/current-replay，并机器生成可核对报告 | ST-EI-006 | FEAT-EI-OBSERVABILITY |
| CAP-EI-07 | Post-close correction and acceptance migration | 不改历史地修正 closed workflow，并以 CR-163 验收 | ST-EI-007 | FEAT-EI-CORRECTION |
| CAP-EI-08 | Custom-agent dispatch attestation | 以显式平台契约证明 discovery/spawn/reuse/runtime identity；平台扩展不可用时诚实输出 contract-ready/unverified | ST-EI-002, ST-EI-006 | FEAT-EI-CORE |

## 3. Feature / Epic 边界

| Feature ID | 名称 | 负责事项 | 不负责事项 | 拥有数据 | 只读数据 | 禁止依赖 |
|---|---|---|---|---|---|---|
| FEAT-EI-CORE | Evidence Integrity Core | typed identities、gate/dispatch/CP attempt state machine、Codex platform contract consumer、D0 discovery、selector/spawn/reuse receipt、不可变 thread identity、correlation、strict validation | 实现 Codex 平台扩展、report 展示、token 采集、post-close mutation、业务运行 | event/attempt/profile/thread correlation rules、strict finding model | canonical ledgers/results/checkpoints/state snapshot、platform discovery/receipt | quant-lab business code；hand-written summaries、filesystem config scan、task_name/prompt/ledger 自报作为 discovery/selector/attestation |
| FEAT-EI-GOVERNANCE | Workspace and Lifecycle Governance | `routing_ref` 解析、local-directory compatibility、phase-work vs opened-gate、delivered/health/read-expansion consistency、compaction semantic manifest | 平台 receipt 生成、audit 聚合、pilot 执行 | route contract、state transition contract、compaction semantic manifest | FEAT-EI-CORE typed graph、workspace filesystem facts | 缺失 routing metadata 时静默 PASS；用 pending_gate 伪装执行中阶段 |
| FEAT-EI-OBSERVABILITY | Replay, Audit and Cost Observability | checker provenance、as-executed/current replay、generated audit report、token measurement dimensions | 修改原结果、推断 receipt/token/checker identity | replay/audit/usage derived artifacts | validated evidence graph、platform telemetry when available | 直接写 canonical ledgers；从 event-row 数推断 attempt/thread/token share |
| FEAT-EI-CORRECTION | Correction Lifecycle and Pilot Adapter | versioned append-only correction、allowed-scope policy、supersession、audit trail、CR-163 adapter contract | 运行真实 pilot、修改 quant-lab lineage 业务代码、原位改写历史 | correction events/manifests and adapter output | closed evidence、FEAT-EI-CORE checker contract | 绕过 FEAT-EI-CORE；跨仓原子事务；对原历史就地修复 |

数据所有权唯一性规则：canonical ledgers/results/state 仍由既有 producer/controlled writer 拥有；本 CR 的 checker、replay 和 reporter 只读。Correction lifecycle 只拥有新的 correction event，不拥有被修正的原对象。

## 4. 跨 Feature 流程

| Flow ID | 触发 | 参与 Feature | 数据写入 Owner | 失败路径 | 验证入口 |
|---|---|---|---|---|---|
| FLOW-EI-01 | dispatch/check/gate 事实追加 | CORE -> GOVERNANCE | 既有 ledger/result producer | identity 缺失、悬空 ref、未闭合 attempt：strict BLOCKED，不推进 gate | TC-EI-001..009, TC-EI-018 |
| FLOW-EI-02 | ledger compact/restore | GOVERNANCE -> CORE | compaction service 仅写 archive/manifest/marker | 语义图 hash、terminal selection、correction/health ref 任一变化：拒绝替换源 ledger | TC-EI-014 |
| FLOW-EI-03 | audit/replay 请求 | CORE -> OBSERVABILITY | reporter/replay runner 写 derived report | provenance 或 input hash 缺失：strict FAIL 或 legacy/unavailable，绝不补造 | TC-EI-011, TC-EI-012, TC-EI-016, TC-EI-017 |
| FLOW-EI-04 | closed workflow correction | CORRECTION -> CORE -> OBSERVABILITY | correction producer 追加 versioned event | scope 非 allowlist、链有环、author/reason/evidence 缺失：拒绝追加 | TC-EI-015 |
| FLOW-EI-05 | CR-163 acceptance pilot（后续独立授权） | CORRECTION -> CORE -> OBSERVABILITY | pilot adapter 只追加 process evidence | 无独立授权、业务代码 diff 非 0 或 replay 非 23/23：BLOCKED/rollback append set | TC-EI-013 |

## 5. 共享集成契约

| Contract ID | 调用方 -> 被调用方 | 调用时机 / 触发 | 输入契约 | 输出契约 | 后续衔接 | 降级 / 失败 |
|---|---|---|---|---|---|---|
| IC-EI-01 | producer -> typed evidence checker | 每次 result/ledger/state 生成后及 gate 前 | canonical refs、typed IDs、timestamps、attempt/supersedes | findings 含 rule/object/field/evidence/safe-route | CP result / gate decision | 缺失或冲突为 FAIL/BLOCKED；summary 不可覆盖 |
| IC-EI-02 | workspace/state checkers -> route resolver | workspace/state/CP check | `routing_ref` + filesystem + metadata | 一个共享 `RouteTruth` 判定及 diagnostics | state-transition 与 CP3/CP8 | dangling ref 为 blocking；legacy mode 只能显式分类 |
| IC-EI-03 | compactor -> semantic snapshot checker | compact apply 前后 | typed graph + source hash + policy | pre/post semantic digest、manifest、restore evidence | 允许原子替换源 ledger | digest 不等则保留源并删除/隔离候选输出 |
| IC-EI-04 | replay/audit -> checker registry | replay/report 生成 | checker identity、schema/policy/input hashes | as-executed/current-replay + diff classification | generated audit report | identity 不可得标 unavailable；strict 不得 fully replayable |
| IC-EI-05 | usage producer -> audit aggregator | agent/checker/context run 结束 | attribution + platform telemetry/proxy source | measured/proxy/unavailable 互斥记录 | CR/phase/agent/attempt/checker aggregation | 不可得则 unavailable；不得由文本长度冒充 measured |
| IC-EI-06 | correction producer -> correction checker | closed evidence 需修正 | schema version、target、allowed patch、author/reason/evidence/supersedes | append-only correction + audit-chain result | current replay/audit | 非 allowlist 或链冲突拒绝；原对象 hash 不变 |
| IC-EI-07 | dispatch producer -> platform capability/discovery API | 指定 custom agent 前 | installed config path、config sha256、requested profile、session capability | discovered profile record 或 unavailable reason | selector admission | profile 未发现、config stale 或 capability 无 selector 时 fail-closed |
| IC-EI-08 | dispatch producer -> platform dispatch API | CONFIG_VALIDATED + PROFILE_DISCOVERED 后 | 显式 `agent_type/profile` selector、dispatch_id、attempt_id、requested profile、config hash | platform receipt | receipt resolver/checker | 仅 task_name/prompt/ledger 自报不得提交为 verified selector；API 无 selector 时不可声称指定 custom agent |
| IC-EI-09 | platform receipt -> profile resolver/checker | receipt 到达后、RUNNING 前 | receipt id/time、dispatch/attempt、requested/resolved profile、config hash、model、reasoning effort | `custom_agent_verified=true` 或 degraded/unattested finding | CP6/CP7/CP8 与 replay | requested/resolved/config/model/effort 任一不匹配、receipt 缺失均非 verified |
| IC-EI-10 | thread reuse admission -> platform followup/resume | followup_task/resume/send_input 前 | target thread、expected spawn receipt/profile/config、new dispatch/attempt | reuse receipt + inherited immutable ThreadRuntimeIdentity | continued attempt/correlation | mismatch/terminal/no receipt：deny reuse；profile 升级必须 new spawn |
| IC-EI-11 | Meta Flow -> Codex subagent platform contract | CP0 capability probe、CP6 producer、CP7 conformance | `CR046-CODEX-SUBAGENT-PLATFORM-CONTRACT.md` CURRENT/REQUIRED schema | contract-ready、runtime-attested 或 unavailable findings | A-baseline 或 Conditional B | TOML scan 最高 CONFIG_VALIDATED；只有 D0平台 discovery 可 PROFILE_DISCOVERED |

调用方同步范围：`meta_flow/checks`、`meta_flow/state`、`meta_flow/workspace`、`meta_flow/workflow`、`meta_flow/context_pack`、CLI、delivery contracts 与 tests 必须消费相同 identity/evidence/routing enums；任何一处另定义同义枚举均视为架构违规。

## 6. 七 Story 技术边界与 DAG 输入（非 Story 拆解）

| Story | 技术边界 | 主要输出面 | 前置依赖 | 后续消费者 | 文件所有权提示 |
|---|---|---|---|---|---|
| ST-EI-001 | gate chronology、conditional approval、in-progress phase/opened gate distinction | chronology/state-transition checker + fixtures | 无 | 002, 003, 004 | state-transition/checker contract owner |
| ST-EI-002 | dispatch/event/attempt/thread/receipt identity、规范性平台契约 consumer、D0 capability discovery、显式 selector、不可变 ThreadRuntimeIdentity、followup reuse admission、new-spawn escalation 与 terminal chain | platform-contract schema + capability/dispatch/reuse producer + receipt/attempt lifecycle | 001 | 003, 004, 005, 006 | dispatch/platform-contract/reuse producer owner |
| ST-EI-003 | CP check attempt identity、hashes、supersession、final correlation、cross-truth | CP result/ledger correlator + fixtures | 001, 002 | 004, 006 | CP result/checkpoint checker owner |
| ST-EI-004 | routing/finalization/health/read-expansion consistency、compaction semantic manifest、audit orchestration | workspace/state/shared consistency + compaction hooks | 001, 002, 003 | 006, 007 | workspace/state/compaction owner |
| ST-EI-005 | token measurement status and attribution | usage schema/collector adapters | 002 | 006 | telemetry schema/adapter owner |
| ST-EI-006 | checker registry、double replay、generated audit、provenance dogfooding、platform conformance 17 fixtures、spawn/reuse receipt、thread identity mutation与 A/B dogfood correlation | checker/replay/platform-conformance contract and proposed CLI surface | 002, 003, 004, 005 | 007 | replay/audit/profile/thread-attestation checker owner |
| ST-EI-007 | reusable correction lifecycle and CR-163 adapter boundary | correction schema/checker/pilot manifest | 004, 006 | CP7/CP8; separately authorized pilot | correction core vs pilot fixture paths separated |

推荐 DAG：`001 -> 002 -> 003 -> 004`; `002 -> 005`; `003 + 004 + 005 -> 006`; `004 + 006 -> 007`。可并行候选仅限 005 与 003/004 的无冲突实现切片；最终文件 owner 和 Wave 必须在 CP3 批准后的 CP4 planning 冻结。

## 7. 可量化蓝图验收

- 七个 Story 映射覆盖率 = `7/7`，REQ-EI-019..023 和 TC-EI-014..018 均至少有一个 owner Story。
- canonical data object 的写 owner 冲突数 = `0`；checker/reporter 对 canonical history 的写入路径数 = `0`。
- DAG 环数 = `0`；未声明跨 Feature 写依赖数 = `0`。
- strict failure 必须为每个 finding 提供 object ID、field/rule、evidence ref 和 safe route，覆盖率 = `100%`。

## 8. 待 CP3 确认边界

| Decision ID | 决策类型 | 问题 | 推荐方案 | 备选方案 | 推荐 / 备选优劣 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| CP3-DQ-01 | architecture | 是否采用单一 typed evidence graph + strict pipeline？ | 在现有 ledgers/results 上建立共享类型/关联层 | A. 每 checker 各自关联；B. 新建独立 evidence DB | 推荐不复制事实且可统一重放；A 快但漂移；B 查询强但形成第二真相源 | 全部 7 Story、维护与迁移风险 | 若现有文件规模无法满足性能 SLO，再以独立 CR 评估只读索引，不改变 source truth |
| CP3-DQ-02 | architecture | local-directory 的 `routing_ref` 如何合法？ | local-directory 也必须有真实、最小、可移植 metadata，checkers 共用 resolver | A. `routing_ref=null` + 明确 legacy 状态；B. checker 内置 synthetic route | 推荐消除 dangling ref；A 需 schema 迁移；B 隐式且难审计 | workspace/state/CP8 shared truth | metadata 部署不可行时切 A；禁止继续当前 dangling-ref PASS |
| CP3-DQ-03 | architecture | post-close correction 如何进入历史？ | append-only versioned correction event + allowlist policy | A. 新 result supersedes；B. 原位修改 | 推荐通用可审计；A 仅适合重新执行结果；B 违反不可变性 | closed workflows 与 pilot | schema 不兼容时 BLOCKED/新 CR；绝不切到原位修改 |
| CP3-DQ-04 | architecture | 如何证明真正调用指定 Codex custom agent？ | config hash + live discovery + explicit platform selector + bound receipt/resolved-profile strict match | A. 用户批准的 compat generic fallback，标 degraded/unattested；B. 阻塞直到平台提供 selector/receipt | 主选可机器证明；A 保持可执行但不能 verified；B 最安全但阻塞交付 | dispatch producer、CP0/6/7/8、critical/debugger 路由 | selector/receipt API 不可用时：critical/debugger 必须 B；普通 profile 可在用户显式批准后切 A；撤回批准或 config hash 变化即回 fail-closed |
| CP3-DQ-05 | architecture | CR-046 自己在平台扩展尚不可用时如何完成 CP6/CP7 dogfooding？ | A-baseline 是无条件可执行基线：真实子 Agent保持 session-observed，contract/fixture 可 PASS，runtime conformance 未运行则 CP7 `PASS_WITH_RISK`、CP8最高 `READY_WITH_RISK`；若 CP7 前平台 REQUIRED extension + 17/17 fixtures可用，条件切 B并用新 producer new-spawn `meta-qa-critical` | A. 纯 A，整个 CR 不尝试 runtime attestation；B. 纯 B，平台未交付则 CR 永久 BLOCKED | 推荐组合不依赖外部时间承诺且能在能力出现时闭环；纯 A 无 runtime验收；纯 B 超出 Meta Flow 控制 | CP6/CP7/CP8结论、平台依赖、发布声明 | 任一 capability/receipt/fixture退化立即回 A；B 不追溯升级旧 dispatch |

## 9. Gotchas

- `event_id`、`dispatch_id`、`attempt_id`、`run_id`、`thread_id` 是不同身份域，不能互作 fallback canonical ID。
- `session-observed` 不等于 `repository-verifiable`，二者也都不等于 `platform-attested`。
- `pending_gate=null` 在 phase work 正执行时是合法事实；只有门禁审查稿真正打开后才可设置 `pending_gate`。
- local-directory 是兼容布局，不是“允许 route metadata 悬空”的豁免。
- `canonical_role` 是工作流角色，`task_name` 是调度任务标签，二者都不是平台 profile selector；只有平台 API 的显式 selector 和绑定 receipt 能证明 custom agent。
- execution `completed` 只证明任务结束，不证明 requested profile/model/effort 被解析并运行；两条结论必须正交记录。
- `.codex/agents` 扫描只证明 installed/configured；只有平台会话级 D0 discovery 能产生 `PROFILE_DISCOVERED`。
- followup/resume 只能继承初始 thread receipt；通过 ledger 改名不能升级模型，default→debugger/critical 必须 new spawn。
