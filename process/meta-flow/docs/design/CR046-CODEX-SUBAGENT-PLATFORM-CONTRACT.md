---
status: confirmed
version: "1.1"
active_change_ref: "CR-046"
contract_id: "CR046-CODEX-SUBAGENT-PLATFORM-CONTRACT"
contract_kind: "current-facts-and-required-extension"
confirmed: true
confirmed_by: "user"
confirmed_at: "2026-07-12T03:17:27Z"
---

# CR-046 Codex Subagent Platform Contract

> 本附录是 CR-046 的平台边界契约。它将 Codex 当前已验证能力与 Meta Flow 所需的目标能力明确分开；`REQUIRED` 字段和 API 是交付目标，不是对当前平台的事实陈述。CP3 已批准本契约架构；平台实现仍须通过后续 conformance 验收。

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-12 | meta-se | 首版显式平台契约：冻结 current-vs-required 能力、spawn/follow-up schema、线程身份、发现层级、dogfooding A 基线与条件 B、fallback、验收 fixture、所有权和 rollout/rollback。 |
| 1.1 | 2026-07-12 | host-orchestrator | 记录 CP3 批准；将 discovery freshness、followup 无 reuse receipt 和 legacy `codex_agent_name` D3 replay 作为 CP4/CP5 强制细化项。 |

## 1. 目的、适用范围与规范词

本契约为两个团队提供同一验收边界：Codex 平台团队据此提供可证明的 custom-agent 选择与线程复用能力；Meta Flow 只在收到符合本契约的平台事实后生成 `platform-attested` 结论。它适用于初始 `spawn_agent`、后续 `followup_task`/等价 thread reuse、capability discovery、profile/model/effort attestation，以及 required/preferred profile 的降级决策。

规范词含义：

- `CURRENT`：已由官方平台目录契约或本会话 callable tool schema 证明的事实。
- `REQUIRED`：CR-046 要求的平台扩展及其验收标准；在实现并通过 fixture 前不得写成现有能力。
- `MUST` / `MUST NOT`：严格验收条件。
- `SHOULD`：默认要求；偏离时必须有版本化理由和等价证据。
- `unavailable`：没有平台事实；不是失败后的待补值，也不能被 Host 自报替代。

### 1.1 事实来源与证据上限

| 来源 | 可证明事实 | 不能证明的事实 |
|---|---|---|
| `delivery/doc/PLATFORM-CONTRACTS.yaml`（官方来源链接已记录） | Codex project/user custom agent 目录分别为 `.codex/agents`、`~/.codex/agents`；Codex Skill 目录分别为 `.agents/skills`、`~/.agents/skills`，Agent 与 Skill 路径不可类比混用 | 某 profile 已在当前会话被发现、某次 dispatch 选择了该 profile、实际模型身份 |
| `.codex/agents/*.toml` / `~/.codex/agents/*.toml` | 配置已安装；TOML `name`、声明 model/effort 和计算出的 config hash | 平台已加载配置、当前会话可选择它、运行时 resolved model/effort |
| 当前 callable `spawn_agent` schema | 可提交 `task_name`、`message`、`fork_turns`，并获得 agent/task identity | `agent_type/profile/model` selector、profile discovery、resolved model receipt |
| 当前 callable `followup_task` schema | 可按 `target` 和 `message` 复用既有目标 | 动态切换 profile/model/effort、返回 reuse receipt 或重新解析证明 |
| task name、prompt、handoff、dispatch ledger 自报 | Host 的路由意图、任务标签或声明 | 平台 custom-agent selection、resolved profile/model/effort |

当前证据 ceiling 为 `session-observed/repository-unverifiable`。当前 CR-046 执行不得追溯补造 platform receipt。

## 2. Current vs Required 能力矩阵

| 能力 | CURRENT | CR-046 REQUIRED | 未满足时 Meta Flow 行为 |
|---|---|---|---|
| Custom-agent 配置位置 | `.codex/agents` / `~/.codex/agents` | 保持；提供可关联的规范化 profile identity 与 config hash | 配置不存在/无效：BLOCKED |
| Profile 配置内容 | TOML 可声明 name/model/effort | 平台 discovery 与 receipt 必须返回可和已验证配置关联的 profile/config hash/model/effort | 只能记录 installed/configured，不能 verified |
| Spawn 请求 | `task_name/message/fork_turns` | 新增显式 `agent_type`（或语义等价、schema 可探测的 selector）及 correlation 字段 | required：BLOCKED；preferred：按 fallback 表处理 |
| Spawn 响应 | agent/task identity；无 resolved model identity | 返回绑定 dispatch/attempt 的 platform receipt 与不可变 ThreadRuntimeIdentity | execution 可单列，attestation=unavailable |
| Capability discovery | 当前无 callable API | 会话级、带 freshness 的 capability/profile discovery | 不得用磁盘扫描推断 discoverable |
| Follow-up | `target/message` | 返回 reuse receipt；证明继承同一 thread runtime identity；禁止无 receipt 的身份升级 | mismatch 时 NEW_SPAWN_REQUIRED 或 BLOCKED |
| Profile escalation | 当前无显式平台保证 | default→debugger/critical 等升级必须 new spawn + 新 receipt + lineage | 改 handoff/ledger 不得升级 |
| Model/effort proof | 当前无 | 必须由平台 receipt 报告 resolved 值及来源 | 保持 unavailable，不得从 TOML 反推实际值 |
| Reload/config change | 当前无 freshness contract | discovery 有 session/epoch/expiry；config hash 变化使旧 discovery 失效，新 spawn 才可采用新配置 | stale proof BLOCKED |

## 3. Capability discovery contract

### 3.1 发现来源层级

Meta Flow MUST 按下列优先级判定，低层来源不得提升高层结论：

| 层级 | 来源 | 允许结论 | 禁止结论 |
|---|---|---|---|
| D0 | 平台会话级 discovery API/response | selector 是否支持、当前 session/epoch 可选 profiles、每个 profile 的 resolved config identity/hash、能力版本和有效期 | 无 |
| D1 | 平台 spawn schema introspection | 是否存在语义明确的 profile selector 和 correlation/receipt 字段 | 目标 profile 已被当前会话加载 |
| D2 | 官方目录契约 + TOML 校验/hash | installed/configured/validated | discovered/selectable/resolved |
| D3 | task name、prompt、handoff、ledger、agent/thread label | requested/declared/session-observed | selector applied/platform-attested |

只有 D0 可产生 `PROFILE_DISCOVERED`。D1 可证明 selector capability，但不能证明某个 profile 可选。D2 是进入 discovery 前的 config input。D3 永远不能升级 attestation。

### 3.2 REQUIRED discovery response

字段名可由平台采用等价命名，但语义和可机器读取性 MUST 保持：

```json
{
  "schema_version": "1.0",
  "capability_id": "cap-...",
  "session_id": "session-...",
  "session_epoch": "epoch-...",
  "observed_at": "2026-07-12T00:00:00Z",
  "expires_at": "2026-07-12T01:00:00Z",
  "spawn_selector": {
    "supported": true,
    "field": "agent_type"
  },
  "followup_reuse_receipt_supported": true,
  "profiles": [
    {
      "agent_type": "meta-qa-critical",
      "config_scope": "project",
      "config_ref": ".codex/agents/meta-qa-critical.toml",
      "config_sha256": "sha256:...",
      "configured_model": "gpt-5.6-sol",
      "configured_reasoning_effort": "xhigh",
      "selectable": true
    }
  ]
}
```

Discovery MUST 绑定当前 `session_id/session_epoch`，并有 `observed_at` 及 `expires_at` 或等价 freshness 机制。会话 reload、epoch 变化、selector schema 变化、配置 hash 变化或 expiry 到期后，旧 discovery MUST 被视为 stale。若平台不能披露 config path，可返回稳定的 `config_identity`，但 config hash/model/effort 的核对能力仍是 required attestation 前提。

## 4. Spawn request/response contract

### 4.1 CURRENT request

```json
{
  "task_name": "qa_cp7",
  "message": "...",
  "fork_turns": "all"
}
```

这里 `task_name` 只是协作任务标签。无论它是否包含 `critical`、`debugger` 或 profile 名称，都不得视作 selector。

### 4.2 REQUIRED request

```json
{
  "task_name": "cr046_cp7",
  "message": "...",
  "fork_turns": "all",
  "agent_type": "meta-qa-critical",
  "dispatch_id": "dispatch-...",
  "attempt_id": "attempt-...",
  "requested_config_sha256": "sha256:...",
  "profile_requirement": "required",
  "capability_id": "cap-...",
  "session_epoch": "epoch-..."
}
```

约束：

- `agent_type` 是建议的规范字段名；平台可采用语义等价名称，但 schema discovery MUST 明确标识它是 custom-agent selector。
- `dispatch_id` 和 `attempt_id` MUST 原样进入 receipt；平台若生成自己的 request id，必须同时保留调用方 correlation。
- `requested_config_sha256` 防止 discovery 后配置漂移；平台不能接受/核对时，不得满足 strict attestation。
- `profile_requirement` 是 Meta Flow 策略输入；平台不负责替 Meta Flow 决定 fallback。
- 不允许用 `model` 参数绕过 custom-agent identity；实际 model/effort 必须由所选配置解析并在 receipt 中报告。

### 4.3 REQUIRED spawn response/receipt

```json
{
  "schema_version": "1.0",
  "receipt_id": "receipt-...",
  "receipt_kind": "spawn",
  "receipt_time": "2026-07-12T00:00:01Z",
  "platform_request_id": "platform-...",
  "dispatch_id": "dispatch-...",
  "attempt_id": "attempt-...",
  "requested_agent_type": "meta-qa-critical",
  "resolved_agent_type": "meta-qa-critical",
  "resolved_config_sha256": "sha256:...",
  "resolved_model": "gpt-5.6-sol",
  "resolved_reasoning_effort": "xhigh",
  "agent_id": "agent-...",
  "thread_id": "thread-...",
  "session_id": "session-...",
  "session_epoch": "epoch-...",
  "resolution_status": "applied"
}
```

`platform-attested/custom_agent_verified=true` 仅在以下条件全部成立时允许：requested profile=resolved profile；validated config hash=resolved config hash；配置声明 model/effort=resolved model/effort；receipt 的 dispatch/attempt/session epoch 与本次请求一致；`resolution_status=applied`。缺字段、`unknown`、错绑、mismatch 或 stale 均不得通过。

### 4.4 ThreadRuntimeIdentity

成功 spawn receipt MUST 冻结以下线程身份：

```json
{
  "thread_id": "thread-...",
  "agent_id": "agent-...",
  "spawn_receipt_id": "receipt-...",
  "spawn_dispatch_id": "dispatch-...",
  "spawn_attempt_id": "attempt-...",
  "resolved_agent_type": "meta-qa-critical",
  "resolved_config_sha256": "sha256:...",
  "resolved_model": "gpt-5.6-sol",
  "resolved_reasoning_effort": "xhigh",
  "session_epoch": "epoch-...",
  "immutable": true
}
```

线程生命周期内 `resolved_agent_type/config_sha256/model/reasoning_effort` MUST 不可由 follow-up request、handoff 或 ledger 修改。若平台支持运行时切换，它 MUST 创建新的 execution/thread identity 和 transition receipt；Meta Flow 默认按 new spawn 处理，不推断原线程已升级。

## 5. Follow-up and reuse contract

### 5.1 CURRENT behavior ceiling

当前请求只有：

```json
{
  "target": "/root/qa_cp7",
  "message": "..."
}
```

它证明消息被发送到既有协作目标，不证明 profile/model/effort 被重新选择，也没有可用的 reuse receipt。当前 follow-up 只能继承“未知/未证明”的原始证据上限，不能因新的 ledger 声明而提升。

### 5.2 REQUIRED follow-up request and receipt

```json
{
  "target": "thread-...",
  "message": "...",
  "dispatch_id": "dispatch-followup-...",
  "attempt_id": "attempt-followup-...",
  "expected_spawn_receipt_id": "receipt-...",
  "expected_agent_type": "meta-qa-critical",
  "expected_config_sha256": "sha256:..."
}
```

```json
{
  "schema_version": "1.0",
  "receipt_id": "reuse-receipt-...",
  "receipt_kind": "followup-reuse",
  "receipt_time": "2026-07-12T00:10:00Z",
  "dispatch_id": "dispatch-followup-...",
  "attempt_id": "attempt-followup-...",
  "thread_id": "thread-...",
  "agent_id": "agent-...",
  "inherited_spawn_receipt_id": "receipt-...",
  "resolved_agent_type": "meta-qa-critical",
  "resolved_config_sha256": "sha256:...",
  "resolved_model": "gpt-5.6-sol",
  "resolved_reasoning_effort": "xhigh",
  "reuse_status": "inherited-unchanged"
}
```

平台 MUST 保证：

1. follow-up 只继承初始 `ThreadRuntimeIdentity`，不得静默重新解析为另一 profile/model/effort。
2. reuse receipt 必须关联原 spawn receipt 和本次 dispatch/attempt。
3. requested expected identity 与线程绑定身份不一致时，平台必须拒绝并返回机器可识别 mismatch；不得接受后静默沿用旧模型。
4. closed/failed/interrupted thread 是否可恢复必须由 capability contract 明示；若支持恢复，需独立 resume receipt。未声明则 Meta Flow 视为不可恢复。
5. config 文件后来变化不改变既有线程身份；若任务要求新 hash/profile，必须 new spawn。

### 5.3 Reuse admission and escalation

| 请求 | 线程绑定身份 | 结果 |
|---|---|---|
| profile/hash 完全一致且 receipt 有效 | 同一 identity | `ALLOW_REUSE`，继承原证明并保存 reuse receipt |
| default 请求 debugger/critical | default thread | `NEW_SPAWN_REQUIRED` |
| debugger 请求 critical | debugger thread | `NEW_SPAWN_REQUIRED` |
| profile 相同但要求新 config hash | 旧 hash thread | `NEW_SPAWN_REQUIRED` |
| 原 spawn receipt 不可用 | 任意 required profile | `BLOCKED_UNATTESTED_THREAD` |
| profile mismatch | 任意 | `REUSE_DENIED_PROFILE_MISMATCH` |
| terminal thread 且无 resume capability/receipt | 任意 | `REUSE_DENIED_TERMINAL_THREAD` |

Profile/model escalation 是新的 execution lineage：新 `dispatch_id`、新 `attempt_id`、显式 selector、新 spawn receipt、新 `ThreadRuntimeIdentity`，并可通过 `escalates_from_thread_id` 关联旧线程。它绝不是 metadata mutation。

## 6. Dogfooding strategy for CR-046

CP3 MUST 采用以下明确策略，避免 CP6/CP7 临时决策：

### 6.1 A-baseline（无条件基线）

CR-046 整个生命周期的既有和后续 dispatch，默认按当前平台能力记录为 `session-observed/repository-unverifiable/platform-attestation-unavailable`。若 CP6/CP7 时平台仍未提供并验证 discovery + selector + receipt：

- 可继续完成 Meta Flow 自身代码、fixture 和 checker 的实现/验证，但不得声称 `meta-dev-debugger`、`meta-qa-critical` 或相应 model/effort 实际生效。
- CP7 必须把平台集成验收拆成两类：Meta Flow contract/fixture 可 `PASS`；真实 Codex runtime conformance 为 `BLOCKED_EXTERNAL_CAPABILITY` 或 `NOT_RUN_UNAVAILABLE`，不得合并成完全 PASS。
- CR-046 的功能交付结论与模型路由 attestation 结论正交。CP8 只有在 CP3/CP5 批准的 release policy 允许“平台适配待实现”时才可 `READY_WITH_RISK`；若目标定义要求端到端 platform-attested，则保持 BLOCKED。
- 所有历史记录保持原样，只追加诚实的 availability/limitation evidence。

### 6.2 Conditional B（满足条件才启用的自举闭环）

仅当 CP6 期间平台已独立提供 REQUIRED discovery、selector、spawn receipt 和 follow-up reuse receipt，并且本契约的 platform conformance fixtures 在新调度前 `100% PASS`，才允许用 Story 2 producer 调度 CP7 的 `meta-qa-critical`。Conditional B 还要求：

- 新机制不能只由被测 producer 自报；平台 receipt 必须来自平台边界。
- CP7 使用全新的 spawn，不复用此前 unattested thread。
- CP7 记录 bootstrap boundary、capability/session epoch、config hash 和 receipt refs。
- 任一 capability 退化、fixture 失败、receipt mismatch 或 reload 立即回 A-baseline；不将 B 的部分证据追溯应用到旧 dispatch。

A 是默认且诚实的可执行策略；B 是条件升级，不是 CR-046 对外部平台交付时间的假设。

## 7. Profile requirement and fallback rules

每次路由 MUST 记录：`requested_agent_profile`、`profile_requirement=required|preferred`、`requirement_source=user|risk-policy|checkpoint|route-plan`、`fallback_policy=block|user-approved-degrade`。

| 条件 | required | preferred |
|---|---|---|
| 配置缺失/无效 | BLOCKED | 请求用户批准替代 profile 或 generic；未批准 BLOCKED |
| D0 discovery 不可用或目标未发现 | BLOCKED | 可请求用户批准 generic fallback |
| selector 不可用 | BLOCKED | 可请求用户批准 generic fallback |
| receipt 缺失、错绑、mismatch、stale | BLOCKED；该次 execution 可单列但不得满足 route | 不得标 verified；继续与否须用户明确批准并记录风险 |
| follow-up 身份不匹配 | 禁止复用，new spawn；new spawn 仍不可用则 BLOCKED | new spawn 或用户批准 degraded generic |
| 用户批准 generic fallback | 不改变 required 要求；必须由用户显式修改 route/risk decision 才可降级 | `degraded-unattested`，`custom_agent_verified=false` |
| 会话 reload/config hash 变化 | 重做 discovery，并 new spawn | 重做 discovery；兼容路径也需重新批准或绑定现有批准范围 |

Generic fallback 记录 MUST 包含 approval ref、批准主体/时间、原因、适用 dispatch/attempt、到期/撤回条件；它不能生成 resolved custom-agent/model 值。critical/debugger 若由安全、权限、生产写、PIT/泄漏风险、最终高风险验证或用户明确模型要求触发，默认 `required`。

## 8. Acceptance fixtures and measurable exit criteria

### 8.1 Platform conformance fixtures

| Fixture ID | 输入 | 预期 |
|---|---|---|
| PC-01 | 磁盘 TOML 存在，D0 未发现 profile | `BLOCKED_PROFILE_NOT_DISCOVERED` |
| PC-02 | task name/prompt 写 critical，请求无 selector | `custom_agent_verified=false` |
| PC-03 | ledger 自报 profile，请求无 selector | strict reject |
| PC-04 | selector 请求 A，receipt resolved B | `BLOCKED_PROFILE_MISMATCH` |
| PC-05 | model/effort 与 validated config 不同 | `BLOCKED_RUNTIME_CONFIG_MISMATCH` |
| PC-06 | receipt 缺失或绑定错误 dispatch/attempt | `BLOCKED_RECEIPT_CORRELATION` |
| PC-07 | discovery epoch/config hash stale | `BLOCKED_STALE_CAPABILITY` |
| PC-08 | default thread follow-up 声明 debugger | `NEW_SPAWN_REQUIRED`；旧身份不变 |
| PC-09 | meta-qa thread 被复用为 meta-qa-critical | `REUSE_DENIED_PROFILE_MISMATCH` |
| PC-10 | follow-up 修改 effort | `REUSE_DENIED_IDENTITY_MUTATION` |
| PC-11 | 无初始 receipt 的 thread 声称继承 critical | `BLOCKED_UNATTESTED_THREAD` |
| PC-12 | receipt 完整且 identity 相同的 follow-up | `ALLOW_REUSE` + bound reuse receipt |
| PC-13 | default→debugger 通过全新 spawn | 旧 thread 不变；新 receipt/identity；lineage 已关联 |
| PC-14 | current schema 只有 task_name/message/fork_turns | A-baseline；不得 platform-attested |
| PC-15 | reload 后尝试复用旧 discovery 发起新 spawn | reject 并要求新 discovery |
| PC-16 | preferred generic fallback 未有用户 approval | BLOCKED |
| PC-17 | preferred generic fallback 有有效 approval | `degraded-unattested`，verified=false |

Exit criteria：非法 fixture 拒绝率=`100%`；合法 spawn/reuse fixture 接受率=`100%`；verified receipt 的 profile/hash/model/effort/correlation 匹配率=`100%`；无完整 selector/receipt 而 `custom_agent_verified=true` 的数量=`0`；thread identity mutation 接受数量=`0`。

### 8.2 CR-046 dogfooding acceptance

- A-baseline 下所有 CR-046 dispatch 的 actual resolved profile/model 空缺必须明确为 `unavailable`，误报 verified 数=`0`。
- Conditional B 只有 platform conformance fixtures `17/17 PASS` 后可启用；启用后 CP7 必须使用新 thread 且 spawn/reuse receipt coverage=`100%`。
- CP7/CP8 必须分别报告 `execution_completed`、`custom_agent_verified`、`model_attested`，不得折叠为一个 PASS。

## 9. Ownership and integration responsibilities

| Owner | 必须交付 | 不得承担/推断 |
|---|---|---|
| Codex platform | D0 discovery、显式 selector、spawn receipt、ThreadRuntimeIdentity、reuse/resume receipt、稳定错误码、schema/version/freshness | Meta Flow 风险分级、fallback approval、ledger 自报 attestation |
| Meta Flow ST-EI-002 | 配置校验/hash、capability consumer、request producer、dispatch/attempt correlation、receipt 持久化、reuse admission、new-spawn escalation | 生成或补造平台 receipt |
| Meta Flow ST-EI-006 | strict checker/replay、requested/resolved/config/model/effort/thread correlation、负向 fixture、CP 报告分维 | 从 completion/task name/prompt 推断 profile |
| Host Orchestrator | profile requirement、requirement source、用户 approval、A/B 策略切换、诚实 ledger/handoff | 把 requested/declared 字段写成 resolved |
| CP0 | capability/discovery 前置检查 | 仅扫描配置就宣称 selectable |
| CP6 | producer/receipt/reuse lifecycle 实现证据 | 把 proposed CLI 当平台已实现 |
| CP7 | 独立 conformance 与 negative fixtures、dogfooding结论 | 用被测 producer 自报替代平台边界证据 |
| CP8 | execution/profile/model 三维汇总及剩余风险 | 以功能成功覆盖 attestation 缺口 |

Codex 平台与 Meta Flow 的集成边界是 receipt：平台拥有 resolved runtime facts；Meta Flow 拥有请求意图、策略、持久化、correlation 和 gate 决策。

## 10. Rollout, switch and rollback

### 10.1 Rollout phases

1. `R0 CURRENT inventory`：保存当前 callable schema 与 A-baseline，历史 dispatch 不改写。
2. `R1 discovery-only`：平台暴露 D0；Meta Flow 仅 audit，对 selector/receipt 仍判 unavailable。
3. `R2 selector+spawn receipt audit`：新 dispatch 双写 requested/receipt evidence，但 gate 暂按已批准兼容策略；运行 PC-01..17。
4. `R3 enforce new spawn`：required profile 仅在 receipt verified 后执行；profile escalation 强制新 spawn。
5. `R4 reuse enforce`：follow-up/resume receipt 与 identity immutability 全量启用。
6. `R5 Conditional B`：只有全部 fixture 通过且当前 session capability 新鲜时，CR-046 后续未执行阶段可狗粮新机制。

### 10.2 Switch conditions

从 A 切 Conditional B 必须同时满足：D0 discovery、显式 selector、spawn/reuse receipt 均可用；schema/version 已固定；PC-01..17 `17/17 PASS`；当前配置 hash 与 discovery 一致；CP7 使用全新 thread；host 写入切换决策 ref。缺一项继续 A。

### 10.3 Rollback triggers and actions

| 触发 | 回滚行为 |
|---|---|
| capability/version/epoch 退化 | 立即停止新 verified dispatch，重做 discovery；CR-046 回 A |
| receipt 丢失或 mismatch | 当前 attempt 标 unattested/failed-attestation；required route BLOCKED |
| config hash 变化 | 旧 thread 身份保持不变；新任务重做 discovery + new spawn |
| reuse identity mutation | 禁用 reuse enforce path，隔离该 receipt，强制 new spawn；不改旧 thread 历史 |
| fixture 回归 | 从 enforce 回 audit/A；保留所有 append-only facts 和失败证据 |

回滚不得删除 receipt、重写历史 dispatch、把 unavailable 补成默认模型，或使已失效 discovery 重新有效。

## 11. Open platform dependencies

| ID | 问题 | 状态 | Owner | 关闭条件 |
|---|---|---|---|---|
| OPD-01 | Codex 是否采用 `agent_type` 还是等价 selector 字段 | OPEN | Codex platform | 官方 schema 可探测且语义明确 |
| OPD-02 | Discovery API 的具体 surface/版本/expiry 机制 | OPEN | Codex platform | D0 response 满足 §3.2 语义 |
| OPD-03 | Spawn/reuse/resume receipt 的签发与持久引用方式 | OPEN | Codex platform | PC-04..13 全通过 |
| OPD-04 | 平台是否可披露 config path；若不可，稳定 config identity 如何关联 hash | OPEN | Codex platform + Meta Flow | strict hash correlation 可机器验证 |

这些 OPEN 项不会授权 Meta Flow 假设能力存在；未关闭时执行 A-baseline。

## 12. Gotchas

- 不要把 `.codex/agents` 与 `.agents/skills` 混为同一发现面；前者是 Agent 配置，后者是 Skill。
- 不要把“文件存在”写成“当前会话发现”；D2 不能提升成 D0。
- 不要把 `task_name`、nickname、prompt、handoff 或 ledger 的 `codex_agent_name` 当 selector。
- 不要从 TOML 的声明模型反推运行时实际模型；只有平台 receipt 能提供 resolved fact。
- 不要把 agent/thread id 本身当 model receipt；identity 只说明实例，不说明配置解析。
- 不要在 follow-up 时改 metadata 伪造 profile 升级；升级必须 new spawn。
- 不要让新 config hash 追溯改变旧 thread；旧线程身份不可变，新配置只作用于新 spawn。
- 不要把 execution success 与 custom-agent/model attestation 合并；二者可独立成功或失败。
- 不要把 Conditional B 当时间承诺；平台能力未通过 fixture 时始终回 A。
- 不要把 `unavailable` 当临时空值并在事后补造；历史证明上限必须保持。
