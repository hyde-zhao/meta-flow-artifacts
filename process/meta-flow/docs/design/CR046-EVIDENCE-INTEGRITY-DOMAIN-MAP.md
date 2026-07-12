---
status: draft
version: "1.2"
active_change_ref: "CR-046"
confirmed: false
---

# CR-046 Evidence Integrity Domain Map

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-12 | meta-se-critical | 定义 evidence identity、gate/attempt/correction/replay/route/telemetry 领域对象与状态不变量。 |
| 1.1 | 2026-07-12 | meta-se-critical | 增加 custom-agent profile/config/capability/selector/receipt 领域对象、八态状态机和 completion/attestation 正交语义。 |
| 1.2 | 2026-07-12 | meta-se / host-orchestrator | 增加 PlatformContract、D0-D3 discovery、ThreadRuntimeIdentity、ReuseReceipt、profile requirement 与 A/B dogfooding 状态。 |

## 1. 术语表

| Term | 定义 | 来源 | 备注 |
|---|---|---|---|
| event row | append-only ledger 中的一行事实 | REQ-EI-001, 021 | 不等于 attempt/thread |
| dispatch | 一次平台调用意图与生命周期容器 | REQ-EI-003..005 | 由多个事件描述 |
| attempt | dispatch 的一次具体尝试 | REQ-EI-003, 004 | 必须唯一且最终 closed |
| thread | 平台会话/线程身份（若平台提供） | REQ-EI-021, 023 | 缺失合法为 unavailable |
| receipt | 平台可验证回执 | REQ-EI-005, 023 | 不可由 session observation 合成 |
| check attempt | 某 checkpoint checker 的一次执行 | REQ-EI-006, 007 | 与 checkpoint identity 分离 |
| phase work | 为某 CP 形成产物的执行中阶段 | CP2 approved observation | 不代表 human gate 已打开 |
| opened gate | 审查稿已存在且状态明确等待用户的人工门 | REQ-EI-001 | 必须同时有 pending checklist |
| semantic snapshot | typed evidence graph 在某输入 hash 下的规范化语义 | REQ-EI-019 | 用于 compact/restore 等价性 |
| correction | closed workflow 上追加的受限修正事实 | REQ-EI-020 | 不改原对象 |
| as-executed | 历史执行时 checker 所给结论 | REQ-EI-015 | provenance 不可得须披露 |
| current-replay | 当前 checker 对同一输入的结论 | REQ-EI-015 | 不覆盖 as-executed |
| measurement status | measured / proxy(estimated) / unavailable | REQ-EI-011..013 | 状态互斥 |
| RouteTruth | filesystem、metadata 与 state ref 共同解析出的路由判定 | CP2 condition | workspace/state checker 必须共享 |
| canonical_role | Meta Flow 状态机使用的功能角色，如 meta-dev/meta-qa | CR-046 increment | 不等于 Codex custom-agent profile |
| task_name | 一次子任务的调度标签/路径 | current collaboration API | 不能充当 agent selector 或 attestation |
| requested_agent_profile | 调用方要求平台选择的 custom-agent profile/type | CR-046 increment | 必须经 capability/discovery 和显式 selector |
| resolved_agent_profile | 平台 receipt 声明实际解析/运行的 profile/type | CR-046 increment | 必须严格等于 requested 才可 verified |
| agent_id / thread_id | 平台返回的执行实体/线程标识 | REQ-EI-003..005 | 关联 attempt，但不能单独证明 profile |
| config hash | 已安装 custom-agent 配置内容的 sha256 | CR-046 increment | selector 与 receipt 必须绑定同一 hash |
| capability/discovery probe | 当前会话向平台查询 selector 支持和已发现 profile 的证明 | CR-046 increment | “磁盘存在文件”不足以证明会话已发现 |
| ThreadRuntimeIdentity | 初始 spawn receipt 冻结的 thread/profile/config/model/effort 身份 | Platform Contract §4.4 | followup/handoff/ledger 不得修改 |
| profile requirement | `required|preferred` 的可计算路由强度 | Platform Contract §7 | required 阻断；preferred 可请求用户批准降级 |
| A-baseline / Conditional B | CR-046 平台能力不可用时的诚实基线 / 能力与17/17 fixture满足后的条件自举 | Platform Contract §6 | B 不是平台交付时间承诺 |

## 2. 领域对象

| Object ID | 对象 | Owner Feature | 关键属性 | 状态 | 规则来源 |
|---|---|---|---|---|---|
| OBJ-EI-01 | EvidenceEvent | FEAT-EI-CORE | event_id、event_type、timestamp、typed refs、payload hash | appended | REQ-EI-001, 019 |
| OBJ-EI-02 | DispatchAttempt | FEAT-EI-CORE | dispatch_id、attempt_id、tool、agent/thread、supersedes、terminal outcome | planned/started/terminal | REQ-EI-003..005, 023 |
| OBJ-EI-03 | CheckAttempt | FEAT-EI-CORE | checkpoint_id、check_attempt_id、checker provenance、input hashes、supersedes | started/terminal/superseded | REQ-EI-006, 007, 022 |
| OBJ-EI-04 | HumanGate | FEAT-EI-CORE | gate_id、checkpoint、conditions、evidence、effective event | unopened/open/conditionally-approved/effective/rejected | REQ-EI-001, 002 |
| OBJ-EI-05 | WorkflowStateSnapshot | FEAT-EI-GOVERNANCE | phase、active refs、pending_gate/checklist、next_action、routing_ref、health_ref | in-progress/awaiting-gate/blocked/delivered | REQ-EI-008..010 |
| OBJ-EI-06 | RouteMetadata | FEAT-EI-GOVERNANCE | schema_version、project_name、routing_mode、path_format、anchored relative paths | valid/legacy/dangling/conflict | CP2 condition / REQ-EI-008 |
| OBJ-EI-07 | SemanticSnapshot | FEAT-EI-GOVERNANCE | source hash、typed nodes/edges digest、terminal selection、correction/health refs | captured/verified/mismatch | REQ-EI-019 |
| OBJ-EI-08 | CorrectionEvent | FEAT-EI-CORRECTION | version、target、allowed patch、author、reason、evidence、supersedes | proposed/appended/rejected/superseded | REQ-EI-020 |
| OBJ-EI-09 | ReplayRun | FEAT-EI-OBSERVABILITY | checker identity、schema/policy/input hashes、as-executed/current outcomes、diff class | complete/legacy-unavailable/failed | REQ-EI-014, 015, 022 |
| OBJ-EI-10 | UsageRecord | FEAT-EI-OBSERVABILITY | CR/phase/agent/attempt/checker attribution、status、measured/proxy values | measured/proxy/unavailable | REQ-EI-011..013 |
| OBJ-EI-11 | AuditReport | FEAT-EI-OBSERVABILITY | input manifest/hashes、checker provenance、event/attempt/thread/outcome/token dimensions | generated/invalid | REQ-EI-021 |
| OBJ-EI-12 | PilotManifest | FEAT-EI-CORRECTION | authorization ref、immutable prefix/hash、append set、23-target replay、protected diff | planned/executed/accepted/rolled-back | REQ-EI-016..018 |
| OBJ-EI-13 | AgentProfileConfig | FEAT-EI-CORE | canonical_role、profile/type、config path、sha256、model、reasoning effort、loaded_at | installed/validated/stale/invalid | CR-046 CP3 increment |
| OBJ-EI-14 | CapabilityProbe | FEAT-EI-CORE | session id、API/schema version、selector fields、discovered profiles、observed_at | supported/unsupported/stale | CR-046 CP3 increment |
| OBJ-EI-15 | ProfileDispatchReceipt | FEAT-EI-CORE | receipt_id/time、dispatch_id、attempt_id、requested/resolved profile、config hash、model、effort、agent_id/thread_id | received/verified/mismatch/unavailable | REQ-EI-005,023 + CR-046 increment |
| OBJ-EI-16 | CodexSubagentPlatformContract | FEAT-EI-CORE | CURRENT facts、REQUIRED schemas、discovery levels、fallback、fixtures、owner、rollout | draft/approved/contract-ready/runtime-conformant | CR046 platform appendix |
| OBJ-EI-17 | ThreadRuntimeIdentity | FEAT-EI-CORE | thread/agent、spawn receipt/dispatch/attempt、resolved profile/config/model/effort、session epoch、immutable | bound/unattested/stale/closed | Platform Contract §4.4 |
| OBJ-EI-18 | ReuseReceipt | FEAT-EI-CORE | followup dispatch/attempt、thread、inherited spawn receipt、resolved identity、reuse status | inherited-unchanged/mismatch/unavailable | Platform Contract §5 |
| OBJ-EI-19 | ProfileRequirementDecision | FEAT-EI-CORE | requested profile、required/preferred、source、fallback policy、approval/expiry | required/preferred/degraded/blocked | Platform Contract §7 |

## 3. 身份与关系不变量

| Rule ID | 规则 | 验证方式 |
|---|---|---|
| RULE-EI-ID-01 | event/dispatch/attempt/run/thread/check-attempt 的 ID namespace 不可互作 canonical fallback | typed fixture + schema/checker negative tests |
| RULE-EI-ID-02 | 一个 attempt 属于且只属于一个 dispatch；retry 必须指向已 terminal 的前 attempt；链无环、无 dangling ref | graph validation |
| RULE-EI-ID-03 | final CP result 只可指向同 checkpoint 的最终 terminal check/dispatch attempt，且 outcome 与 decision 兼容 | correlation fixture |
| RULE-EI-ID-04 | compaction 前后 typed nodes、typed edges、terminal selection、correction chain、health refs 的 canonical digest 必须完全相等 | pre/post semantic digest |
| RULE-EI-ID-05 | report 的 event-row、attempt、thread、outcome、token dimension 分别聚合，任何维度不得由另一维度推断 | golden oracle |
| RULE-EI-ID-06 | D2 filesystem/TOML validation 最高只能产生 CONFIG_VALIDATED；只有 session-bound D0 platform discovery 可产生 PROFILE_DISCOVERED | discovery source fixture |
| RULE-EI-ID-07 | ThreadRuntimeIdentity 的 resolved profile/config/model/effort 在线程内不可变；followup 只能继承，升级必须 new spawn | reuse/new-spawn fixtures PC-08..13 |

## 4. 状态机

### SM-EI-01 DispatchAttempt

| 当前状态 | 合法转换 | 条件 | 非法转换处理 |
|---|---|---|---|
| planned | started / cancelled | 有唯一 attempt_id | 拒绝并定位 ID |
| started | succeeded / failed / interrupted / superseded | terminal evidence 非空 | 未关闭则阻断 gate |
| failed/interrupted | 新 retry attempt（旧对象不变） | 新 attempt.supersedes=旧 attempt | 悬空/循环拒绝 |
| terminal | 无原位状态改写 | 修正仅可追加 correction/superseding event | immutable violation |

### SM-EI-02 PhaseWorkAndHumanGate

| 语义状态 | 必要事实 | 合法下一步 | 禁止状态 |
|---|---|---|---|
| phase-in-progress | current_phase=solution-design；active_delegation_ref 非空；pending_gate=null；next_action.type=delegate/continue | 产物和自动预检完成后 open gate | checker 要求提前 pending_gate=CP3 |
| gate-open | pending_gate=CP3；pending_checklist_path 存在；next_action.type=await_user | approved/changes_requested/rejected | checklist 不存在或仍宣称 phase work running |
| approved-effective | gate ledger 有 approval/effective event；pending_gate 已清 | 执行 post-approval route | 只凭 Markdown 文本推进 |
| interrupted | stop_reason 为授权/健康/失败枚举 | 修复/授权后恢复 | 用假 gate 表达 interruption |

### SM-EI-03 PostCloseCorrection

| 当前状态 | 合法转换 | 条件 | 非法转换处理 |
|---|---|---|---|
| closed-original | proposed correction | target 存在且 immutable hash 已捕获 | target 缺失 BLOCKED |
| proposed | appended / rejected | scope allowlist、author/reason/evidence/supersedes 完整 | 任何缺失拒绝 |
| appended | superseded by later correction | 新 correction 明确回链且无环 | 原位删除/修改拒绝 |
| appended chain | replayed/audited | checker provenance/input hashes 完整或 unavailable 明示 | 不得伪造历史 checker |

### SM-EI-04 RouteTruth

| 状态 | 判定 | checker 行为 |
|---|---|---|
| valid-symlink | metadata 存在、target 存在、project/name/anchors 一致 | PASS |
| valid-local-directory | metadata 存在且 routing_mode=local-directory，routing_ref 指向它，project/root/link 字段一致 | PASS_WITH_COMPATIBILITY_WARNING |
| legacy-explicit | 仅在迁移窗口使用明确 legacy marker/null ref policy | WARN/BLOCKED 由 profile 决定，不得称 valid metadata |
| dangling | 非空 routing_ref 目标不存在 | FAIL/BLOCKED（workspace 与 state 一致） |
| conflict | filesystem/state/metadata 任一冲突 | FAIL/BLOCKED |

### SM-EI-05 CustomAgentDispatchAttestation

| 当前状态 | 合法下一状态 | 进入/退出条件 | fail-closed 行为 |
|---|---|---|---|
| PROFILE_REQUESTED | CONFIG_VALIDATED | requested profile 独立于 canonical_role/task_name；安装配置存在、schema 合法、sha256 已计算 | profile 空/不存在：BLOCKED |
| CONFIG_VALIDATED | PROFILE_DISCOVERED | 当前会话 capability probe 明确支持 profile selector，discovery 返回同 profile 和同 config hash（若平台暴露） | 磁盘有配置但未发现、probe stale：BLOCKED |
| PROFILE_DISCOVERED | DISPATCH_SUBMITTED | platform dispatch API schema 显式提供 `agent_type/profile` selector，提交 requested profile/config hash/dispatch+attempt | 只有 task_name/prompt/ledger 字段：不得提交为 specified-profile verified |
| DISPATCH_SUBMITTED | RECEIPT_RECEIVED | 平台返回 receipt id/time 并绑定 dispatch_id/attempt_id | receipt missing：unattested/BLOCKED |
| RECEIPT_RECEIVED | PROFILE_RESOLVED | receipt 给出 requested/resolved profile、config hash、model、reasoning effort | 字段缺失/dispatch/config hash 错：mismatch |
| PROFILE_RESOLVED | RUNNING | requested==resolved，config hash/model/effort 与 validated config 严格一致 | 任一 mismatch：不得 RUNNING_VERIFIED |
| RUNNING | succeeded/failed/interrupted/superseded | agent_id/thread_id 与 attempt 绑定，terminal closure 完成 | terminal 缺失阻断执行闭环 |
| 任一 pre-running | degraded-unattested（仅 compat） | 普通 profile 且用户显式批准 generic fallback，记录 approval/fallback reason | critical/debugger required profile 禁止进入；`custom_agent_verified` 必须 false |

正交不变量：`execution_completed=true` 与 `custom_agent_verified=true` 是两个独立布尔结论。generic agent 完成任务可前者为 true、后者为 false；receipt/profile 验证成功但执行失败可前者为 false、后者为 true（同时 terminal outcome=failed）。

当前 dogfooding fixture：本会话 collaboration `spawn_agent` schema 暴露 `task_name/message/fork_turns`，不含 `agent_type/profile` selector。因此当前 CR-046 dispatch 只能标 `session-observed/repository-unverifiable`；既有 task_name、prompt、agent id/thread id 或 ledger 自报均不得追溯升级为 receipt/profile proof。

### SM-EI-06 ThreadReuseAndEscalation

| 当前状态 | 请求 | 合法结果 | 禁止行为 |
|---|---|---|---|
| bound-attested | 同 profile/hash 的 followup/resume | `ALLOW_REUSE` + reuse receipt，继承初始 identity | 重新解析或修改 model/effort |
| bound-attested | default→debugger/critical 或 hash变化 | `NEW_SPAWN_REQUIRED` + `escalates_from_thread_id` | 改 handoff/ledger 完成升级 |
| unattested | required profile reuse | `BLOCKED_UNATTESTED_THREAD` | 自报 inherited critical |
| terminal | 无平台 resume receipt | `REUSE_DENIED_TERMINAL_THREAD` | 仅凭 thread id 恢复 |

### SM-EI-07 CR046Dogfooding

| 状态 | 条件 | CP7/CP8结论上限 | 转换 |
|---|---|---|---|
| A-baseline | 当前无 D0/selector/receipt 或17/17未通过 | contract fixture 可 PASS；runtime conformance unavailable；CP7最高 PASS_WITH_RISK、CP8最高 READY_WITH_RISK | 平台 extension + PC-01..17全部通过后可切 B |
| Conditional-B | fresh D0、selector、spawn/reuse receipt、17/17 PASS，新 QA thread | execution/profile/model 分轴验证 | 任一退化立即回 A；不升级旧历史 |

## 5. Route metadata 最小有效集合

推荐 `local-directory` metadata 至少包含：`schema_version`、`project_name`、`routing_mode=local-directory`、`path_format=portable-relative-v1`、`project_root="."` + anchor、`project_process_root="process"` + anchor、`link_path="process"` + anchor。若设计沿用通用 schema 还要求 `artifact_root`，其值必须是可移植相对路径且与实际归属一致；不得用当前设备绝对路径或虚构 artifact root。

`STATE.current.json.routing_ref` 必须是 project-root anchored 相对 ref，目标存在并能解析为同一 RouteTruth。最小集合字段缺失、目标不存在、project_name 不符或 mode 与 filesystem 不符均是 blocking finding。

## 6. 可量化规则

- 所有适用 attempt terminal closure 覆盖率 = `100%`。
- 所有 CP result 的 input hash 与 check-attempt coverage = `100%`。
- compaction restore semantic digest mismatch = `0`。
- closed original 的 byte/hash mutation count = `0`。
- usage record 的 measurement_status 覆盖率 = `100%`，measured-without-platform-source count = `0`。
- dangling routing_ref 被任何 checker 报 PASS 的次数 = `0`。
- specified custom-agent dispatch 的 config/capability/selector/receipt 四段证据覆盖率 = `100%`；任一段缺失时 `custom_agent_verified=true` 数量 = `0`。
- verified receipt requested/resolved profile、config hash、model、reasoning effort 严格匹配率 = `100%`。
- platform conformance fixtures = `17/17 PASS` 后才能进入 Conditional B；thread identity mutation 接受数 = `0`。

## 7. Gotchas

- superseded 是关系，不是删除；旧 attempt/result/correction 仍需保留。
- `unavailable` 是合法证据状态，不是空字符串或自动降级为 verified。
- RouteTruth 必须来自共享 resolver；两个 checker 各自“合理推断”会再次产生 cross-truth split-brain。
- 配置文件存在不代表当前会话发现了 profile；reload 后旧 discovery/receipt 不能继续证明新 config hash。
- 不要让 followup 改名模拟 profile 升级；existing thread identity 不变，新 profile必须 new spawn。
