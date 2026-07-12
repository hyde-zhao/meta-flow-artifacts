---
status: confirmed
version: "1.3"
active_change_ref: "CR-046"
confirmed: true
confirmed_by: "user"
confirmed_at: "2026-07-12T03:17:27Z"
---

# CR-046 Evidence Integrity Architecture Decisions

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-12 | meta-se-critical | 初始 CP3 ADR 集：typed graph、identity、phase/gate、routing、compaction、correction、audit/replay、telemetry、pilot。 |
| 1.1 | 2026-07-12 | meta-se-critical / host-orchestrator | CP3 用户修订：新增 ADR-010，冻结 custom-agent config/discovery/显式 selector/platform receipt 与 requested/resolved profile/model attestation 契约。 |
| 1.2 | 2026-07-12 | meta-se / host-orchestrator | CP3-R3 整改：将 Codex 子 Agent 平台行为形式化为规范附录，增加 D0-D3 discovery、不可变 ThreadRuntimeIdentity/followup reuse，并新增 ADR-011 的 A-baseline + Conditional-B 狗粮策略。 |
| 1.3 | 2026-07-12 | host-orchestrator | 记录 CP3-R3 批准；三个 minor 不改变 ADR，转为 CP4/CP5 的 fixture、freshness 与 legacy replay 细化要求。 |

> 所有 ADR 已由用户在 CP3-R3 于 2026-07-12 批准；三个不阻塞 minor 进入 CP4/CP5 设计细化和验收。

## CR046-EI-ADR-001：在现有真相源上建立 typed evidence contract

- 状态：accepted；Decision Brief：CP3-DQ-01。
- 主选：canonical ledgers/results/state/route metadata 保持事实源；运行时构建 typed graph，统一 normalization、correlation、strict finding。
- 备选 A：各 checker 局部关联（交付快但 split-brain 高）。备选 B：新 evidence DB（查询强但第二真相源和迁移成本不可接受）。
- 后果：所有 producer/checker/reporter 使用一个 identity/evidence enum contract；report 只派生。
- 切换：只有实测显示文件派生图无法满足明确性能 SLO，才以新 CR 评估只读索引；不改变 canonical truth。
- 回滚：enforce -> audit，保留新 append-only 字段和 fixtures。

## CR046-EI-ADR-002：分离 event、dispatch、attempt、thread、receipt、run 与 check-attempt identity

- 状态：accepted。
- 主选：每个 namespace 强类型、唯一；retry/supersedes 形成无环链；terminal closure `100%`；checkpoint event identity 与 check attempt identity 分开。
- 备选：复合字符串/跨域 fallback。拒绝原因：R2 duplicate checkpoint event 与 compaction display fallback 已证明会混淆。
- 失败：collision、dangling、cycle、unclosed attempt、final ref 非 terminal 均 BLOCKED。
- Evidence level 独立维度：`session_observed`、`repository_verifiable`、`platform_attested`、`receipt_status`，不得由一维推导另一维。

## CR046-EI-ADR-003：phase work in progress 与 opened human gate 是不同事实

- 状态：accepted。
- 主选：执行中阶段允许 `pending_gate=null`；只有审查产物、自动预检、Decision Brief/checklist 已形成且 host 发起门禁后，才设置 pending gate + checklist + await_user。
- 备选 A：批准上一 CP 后立即预置下一 pending_gate。拒绝：伪造未来门且与 active delegation 冲突。备选 B：用 stop_reason 表达正常 phase execution。拒绝：stop_reason 是终止/中断原因，不是运行态。
- Checker contract：post-transition 必须接受中间 phase-progress event/state；“抵达下一 gate”才要求 gate-open。执行完成而门未开、门开而 checklist 缺失、执行中提前开门均拒绝。
- Acceptance fixture：CP2 approved -> solution-design active/pending_gate null 应 PASS；meta-se returned + artifacts complete/pending_gate null 应 FAIL until host opens CP3；gate open + checklist/await_user 应 PASS。

## CR046-EI-ADR-004：local-directory 也必须有可解析 routing metadata（CP3 强制决策）

- 状态：accepted；Decision Brief：CP3-DQ-02。
- 已验证问题：当前 `STATE.current.json.routing_ref=process/.meta-flow-process.yaml`，目标不存在；workspace checker 仍以 local-directory compatibility 报 OK，形成 shared-truth 冲突。
- 主选：local-directory 继续作为兼容布局，但必须写真实 metadata；`routing_ref` 必须指向存在目标。最小有效字段：`schema_version`、`project_name`、`routing_mode=local-directory`、`path_format=portable-relative-v1`、`project_root` 及 anchor、`project_process_root` 及 anchor、`link_path` 及 anchor；若通用 schema 需要 artifact_root，也必须为真实 portable relative path。
- executable alternative A：`routing_ref=null` + `routing_status=legacy-local-directory`（或等价 versioned state），workspace/state 同时 WARN/BLOCKED，并设迁移期限；禁止非空 dangling ref。
- executable alternative B：迁移为标准 symlink：通过 workspace bootstrap/link 生成 portable metadata，再验证 state/target/project_name 一致；适用于 artifact root 已确认。
- 拒绝方案：checker 静默 synthetic metadata；它不可审计且让 workspace/state 继续分裂。
- Shared truth：workspace check、state check、doctor、CP consistency 必须调用同一个 RouteTruth resolver；不可各自解析。
- dangling-ref：strict profile 一律 FAIL/BLOCKED，错误含 ref、expected target、filesystem mode、safe route；compat profile 也不得 PASS，只能显式 legacy warning/block。
- migration：inventory -> 生成 candidate metadata -> resolver dry-run -> 原子写 metadata -> 更新/确认 routing_ref -> state/workspace/cross-truth checks；不改 process 业务内容。
- rollback：保存旧 state hash；candidate 不通过则不应用。应用后冲突则恢复旧 ref 并进入 explicit legacy-null mode；不得回到 non-null dangling PASS。
- switch conditions：当前 local dir 可被 portable fields 准确描述时用主选；无法描述时切 A；artifact root/跨仓治理确认且迁移检查通过时切 B。

## CR046-EI-ADR-005：compaction 以 typed semantic digest 作为 apply gate

- 状态：accepted。
- 主选：source hash + typed nodes/edges + terminal selections + correction chains + workflow-health refs 组成 semantic manifest；restore/replay 完全相等才可 apply。
- 备选：只比较 event count/hash range。拒绝：无法发现 identity fallback conflation 或关系丢失。
- 失败/回滚：mismatch 时源 ledger 不变，candidate 隔离，finding 定位 node/edge/rule；不得 best-effort restore。

## CR046-EI-ADR-006：post-close correction 使用通用 versioned append-only lifecycle

- 状态：accepted；Decision Brief：CP3-DQ-03。
- 主选：schema version、target、allowlisted fields/scope、author、reason、evidence、supersedes；链无环；原历史 hash 不变；独立 audit trail。
- 备选 A：真实重新执行时创建新 result 并 supersede。适用于 result，不替代通用 correction。备选 B：原位修改，永久拒绝。
- 回滚：错误 correction 由新的 superseding correction 撤销/更正；不删除已追加事件。

## CR046-EI-ADR-007：replay 与 generated audit 使用 provenance-bearing input manifest

- 状态：accepted。
- 主选：每次 run 记录 checker name/version or commit、schema/policy/input hashes；同时输出 as-executed/current-replay 和 diff classification；audit 分离 event/attempt/thread/outcome/token dimensions。
- 备选：手工报告或只输出 current result。拒绝：既有手工统计已出现计数错误，且会抹去历史语义。
- Dogfooding：CR-046 R1 CP1/CP2 null provenance 原文件/hash 保持，strict FAIL 或 legacy/unavailable；R2/新结果携带实际 provenance。

## CR046-EI-ADR-008：token telemetry 三态且 measured 只来自平台

- 状态：accepted。
- 主选：每条适用 usage record 有 measured/proxy/unavailable；归属 CR/phase/agent-attempt/checker-run；字段互斥。
- 备选：只有单一 token 数或从文本估算填 measured。拒绝：虚假精度。
- 失败：来源缺失但 status=measured 为 strict FAIL；无 telemetry 合法输出 unavailable。

## CR046-EI-ADR-009：CR-163 是独立授权的 acceptance adapter，不是通用架构 owner

- 状态：accepted。
- 主选：Meta Flow 先交付通用 correction/replay/audit；pilot adapter 后续读取 CR-163 process evidence，append-only 追加 manifest/correction/supersession/replay；目标 23/23，protected business diff=0。
- 备选：为 CR-163 定制 correction schema。拒绝：不可复用且反转依赖。备选：本 CR 直接执行 pilot。拒绝：当前未授权。
- 执行前置：独立 authorization ref、immutable prefix/hash manifest、protected paths、rollback append set、current checker identity 全部存在。

## CR046-EI-ADR-010：指定 custom agent 必须经 live discovery、显式 selector 与平台 receipt 证明

- 状态：accepted；Decision Brief：CP3-DQ-04。
- 主选：安装配置先校验 schema 并计算 config sha256；当前会话 capability probe 必须发现目标 profile 和平台 `agent_type/profile` 等显式 selector；dispatch request 绑定 dispatch/attempt/requested profile/config hash；平台 receipt 返回并绑定 requested/resolved profile、config hash、model、reasoning effort、agent/thread 与 receipt time/id。所有字段严格匹配后才可 `custom_agent_verified=true` / `platform-attested`。
- 身份边界：`canonical_role` 是工作流角色，`task_name` 是任务标签，`agent_id/thread_id` 是执行实例；它们与 prompt/ledger 自报都不能替代 requested/resolved profile 或平台 receipt。
- 状态机：`PROFILE_REQUESTED -> CONFIG_VALIDATED -> PROFILE_DISCOVERED -> DISPATCH_SUBMITTED -> RECEIPT_RECEIVED -> PROFILE_RESOLVED -> RUNNING -> terminal`；`execution_completed` 与 profile/model attestation 正交。
- executable alternative A：普通 profile 在用户显式批准后使用 generic fallback，记录 approval/ref/reason 与 `degraded-unattested`，且 `custom_agent_verified=false`。
- executable alternative B：阻塞直到平台安装重载、发现面、selector 与 receipt 可用。required critical/debugger profile 固定采用 B，不得静默回默认/generic Agent。
- strict failure：profile/config 不存在、当前会话未发现、selector 缺失、receipt 缺失或错绑 dispatch/attempt、requested/resolved/profile/config/model/effort mismatch、reload 后 stale hash 均 BLOCKED；不得追溯补造 receipt。
- 当前事实：本会话 collaboration `spawn_agent` schema 未暴露 `agent_type/profile` 或 resolved model receipt；CR-046 现有 dispatch 保留 `session-observed/repository-unverifiable`，仅作为 dogfooding fixture。
- 规范边界：`docs/design/CR046-CODEX-SUBAGENT-PLATFORM-CONTRACT.md` 是本 CR 的规范性平台附录，`delivery/doc/PLATFORM-CONTRACTS.yaml` 是平台路径/发现面的派生单一真相源。附录必须明确标记 CURRENT 与 REQUIRED；REQUIRED 字段不得被文档措辞冒充为当前平台能力。
- Discovery 分层：D0 是平台对当前会话返回的 profile list/resolve 响应，只有 D0 可建立 `PROFILE_DISCOVERED`；D1 tool schema introspection 只证明 selector 字段能力；D2 TOML scan 只证明 `CONFIG_VALIDATED`；D3 task/prompt/handoff/ledger 只证明 requested/declared intent。
- Thread identity：初次 spawn receipt 建立不可变 `ThreadRuntimeIdentity(thread_id,resolved_agent_profile,resolved_model,resolved_reasoning_effort,agent_config_hash,receipt_id)`。`followup_task` 只能继承并返回绑定原 spawn 的 reuse receipt，不允许动态升级 identity；profile mismatch 的 required route 必须新 spawn。
- required/preferred：required critical/debugger profile 缺 D0/selector/receipt 时 BLOCKED；preferred/default profile 只有用户显式 waiver 才可 generic fallback，并保持 `degraded-unattested`。
- Story/CP：ST-EI-002 owner capability/config/dispatch producer/receipt lifecycle；ST-EI-006 owner checker/replay/correlation。CP0 消费 capability/discovery，CP6 消费 producer evidence，CP7 独立验证负例，CP8 汇总 execution/profile/model 三个口径。
- 负向 fixture：磁盘有配置但未发现、task_name 冒充 selector、ledger 自报但请求无 selector、profile/model mismatch、receipt 缺失/错误 dispatch/config hash、reload 后 stale hash，strict 拒绝率 `100%`。
- 切换/回滚：selector/receipt 可用且全部 fixture 通过后从 B 或 A 切主选；会话 reload、配置 hash 变化或 receipt 能力退化立即撤销旧 discovery 证明并回 strict probe/BLOCKED；不改写历史 dispatch。

## CR046-EI-ADR-011：CR-046 采用 A-baseline + Conditional-B 狗粮策略

- 状态：accepted；Decision Brief：CP3-DQ-05。
- 主选：A-baseline 保证 CR-046 不依赖外部 Codex 平台扩展才能完成。当前 generic/session-observed dispatch 可证明执行发生，但不能证明 custom profile/model；Story 2/6 的平台 contract、adapter、checker 和 PC-01..17 conformance fixtures 仍可形成完整仓库交付。
- 基线判定：若 CP7 时 D0 discovery、selector 或 platform receipt 任一不可用，contract/fixture 验证可 PASS，但 runtime attestation 必须为 unavailable；CP7 最高 `PASS_WITH_RISK`，CP8 最高 `READY_WITH_RISK`。该风险不允许通过 task label、handoff 或 ledger 自报消除。
- Conditional-B：仅当当前会话已具备 D0 discovery、显式 selector、spawn/reuse receipts，并且 PC-01..17=`17/17 PASS`，才新 spawn 经 receipt 验证的 `meta-qa-critical` 执行 CP7 runtime conformance；不得复用此前 generic/unattested QA thread。
- 备选 A（pure A）：整个 CR 均停留 session-observed；诚实、可确定交付，但 runtime profile proof 留作平台 follow-up。可接受但不优于主选。
- 备选 B（pure B）：没有平台 extension 就阻塞 CR。runtime proof 最强，但把 Meta Flow 治理交付绑定到外部平台时序，拒绝作为基线。
- 切换条件：平台 extension 成为 CURRENT、D0+selector+receipt 全部存在、contract fixture 17/17 后从 A 切 B；任何 capability regression、stale config hash、receipt mismatch 后立即回 A。
- 回滚：保留 A 路径已完成的 repository evidence，不重写旧 dispatch/receipt；B 的 runtime attempt 失败只追加 terminal/finding，不污染 contract fixture 结论。

## 决策与产物映射

| ADR | Blueprint/Domain/Dependency | HLD | 下游 Story |
|---|---|---|---|
| 001/002 | CAP-EI-01..03, identity rules | §§4-5 | 001..003 |
| 003/004 | RouteTruth/phase gate state machine | §§5.2-5.3, SIM-01/02 | 001,004 |
| 005/006 | SemanticSnapshot/CorrectionEvent | §§5.4-5.5 | 002,004,007 |
| 007/008 | ReplayRun/AuditReport/UsageRecord | §§5.5-7 | 005,006 |
| 009 | PilotManifest/forbidden dependency | §§1.3,7,9 | 007 |
| 010 | CapabilityProbe/ProfileDispatchReceipt/custom-agent state machine | §§1.1-1.4,4,5.1A,6-7,12 | 002,006 |
| 011 | PlatformContract/ThreadRuntimeIdentity/A-B dogfooding | §§1.4,2,5.1A-5.1B,7,9,12 | 002,006 |

## Gotchas

- ADR proposed 不等于用户接受；host 仍需在 CP3 Decision Brief 中逐项收集决定。
- compatibility 是显式状态和迁移策略，不是 checker 忽略缺失事实的理由。
- “新 result supersedes”不能覆盖非 result 对象的通用 post-close correction 需求。
- `task_name`、prompt、agent/thread id 或 ledger `codex_agent_name` 自报不能证明平台实际加载了指定 custom profile/model。
- `.codex/agents/*.toml` 可证明配置安装，不能替代 D0 runtime discovery；REQUIRED platform extension 不是 CURRENT capability。
- `followup_task` 的 target/thread 复用不能改变 ThreadRuntimeIdentity；需要升级 profile 时必须新 spawn。
