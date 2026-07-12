---
status: baseline
version: "1.3"
created_at: "2026-07-02"
owner: "meta-pm"
cr_ref: "CR-037"
active_change_ref: "CR-046"
ready_for_design: true
approval_context: "source implementation plan accepted; user authorized pausing unfinished CR-036 and activating CR-037 on 2026-07-02; CR-037 remains CP2 pending and does not authorize implementation, CP5, runtime, production write, or quant-lab release repo changes"
source_use_cases:
  - UC-PG-001
  - UC-PG-002
  - UC-PG-003
  - UC-PG-004
  - UC-PG-005
  - UC-PG-006
  - UC-PG-007
  - UC-EI-001
  - UC-EI-002
  - UC-EI-003
  - UC-EI-004
  - UC-EI-005
source_plan: "process/docs/design/META-FLOW-PROJECT-GOVERNANCE-STATE-ENFORCEMENT-IMPLEMENTATION-PLAN-2026-07-02.md"
---

# Meta Flow 项目治理与状态强制需求

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 | 文档处理方式 |
|---|---|---|---|---|
| 1.1 | 2026-07-02 | host-orchestrator | 同步 CR-037 已激活、CR-036 暂停未完成和 CP2 pending 不授权实现的状态语义 | 小范围状态语义同步 |
| 1.2 | 2026-07-11 | meta-pm | 为 CR-046 增量增加 evidence-integrity、replayability、telemetry 和 CR-163 append-only pilot 需求；保留全部 REQ-PG ID | 原文档增量更新 |
| 1.3 | 2026-07-12 | meta-pm | CR-046 CP2 scope rework R2：新增 REQ-EI-019..023，覆盖 compaction 语义保持、通用 post-close correction、机器 audit report、null-provenance dogfooding 与 dispatch 证据限制；保留全部既有 REQ ID | 原文档增量更新 |
| 1.0 | 2026-07-02 | meta-pm | 基于已批准实施计划提取产品需求基线 | 初始化长期可追踪需求基线 |

## 功能需求

| ID | 需求描述 | 优先级 | 验收条件 | 来源场景 |
|---|---|---|---|---|
| REQ-PG-001 | 系统必须为 `STATE.current.json` 建立 allowlist schema，仅允许声明字段写入。 | P0 | Given 待写入 patch 包含未声明顶层 key When 执行 current state 写入 Then audit 模式输出 WARN，enforce 模式拒绝写入并返回可追踪错误。 | UC-PG-001 |
| REQ-PG-002 | 系统必须为 `STATE.current.json` 合法字段执行字段级预算校验。 | P0 | Given `next_action`、`source_refs` 或 `open_risks` 超过预算 When 执行写入 Then 写入失败，错误指出字段和预算原因。 | UC-PG-001 |
| REQ-PG-003 | 系统必须提供受控 current state 更新入口，支持 deep-merge patch、actor 和 reason。 | P0 | Given 调用 `update_current_state(project_root, patch, actor, reason)` When patch 合法 Then 完整 state 写入前后均通过 schema 和预算校验。 | UC-PG-001 |
| REQ-PG-004 | 内部 current state 更新路径不得绕过受控 writer。 | P0 | Given CR lifecycle 或 state-router 需要更新 current state When 执行更新 Then 不存在直接 `write_text()` 写 `STATE.current.json` 的路径。 | UC-PG-001 |
| REQ-PG-005 | Agent / Skill 写契约必须禁止直接编辑 `STATE.current.json`。 | P0 | Given 交付规则和 state-router 文档更新完成 When 审查 agent / skill contract Then 明确声明合法更新入口和重型状态落盘位置。 | UC-PG-001 |
| REQ-PG-006 | `STATE.current.json` allowlist 必须包含 `project_state_ref`，用于引用项目级状态。 | P0 | Given P1 项目状态启用 When 写入 `project_state_ref` Then current state schema 校验通过且字段预算生效。 | UC-PG-002 |
| REQ-PG-007 | 系统必须创建 `process/project/` scaffold。 | P1 | Given 初始化或迁移 workspace When 执行 bootstrap Then `process/project/` 被创建且测试覆盖该行为。 | UC-PG-002 |
| REQ-PG-008 | 系统必须提供 refs-only 的 `PROJECT.current.json`，并限制大小。 | P1 | Given 项目状态包含 roadmap / milestone / health / deferred refs When 执行 project state check Then 文件不超过 16KB 且不包含 roadmap 全文或 deferred 详情。 | UC-PG-002 |
| REQ-PG-009 | 系统必须通过 `PROJECT-SCALE.yaml` 表达 `lite / standard / full` 三档项目规模和 gate profile bias。 | P1 | Given 项目设为 `full` When 读取 project scale Then 可看到默认 gate profile bias 和 reason，且未修改 `GATE-PROFILES.json`。 | UC-PG-003 |
| REQ-PG-010 | `feature_refs` 必须引用标准 `FEATURE-REGISTRY.yaml` 中已注册 feature ID。 | P1 | Given 新 CR 或 migration 声明 `feature_refs` When 运行 feature check Then 未注册 ID 被报告为 blocked finding 或 follow-up。 | UC-PG-004 |
| REQ-PG-011 | `capability_refs` 必须引用标准 `CAPABILITY-STATUS.yaml` 中已注册 capability ID。 | P1 | Given 新 CR 或 migration 声明 `capability_refs` When 运行 capability check Then 未注册 ID 被报告为 blocked finding 或 follow-up，不被自动创建。 | UC-PG-004 |
| REQ-PG-012 | 系统必须将 `impact_surface` 限定为治理面枚举，并将路径写入 `affected_paths`。 | P1 | Given 输入包含 `process/checks` 或 `trading/strategy_runner` When 运行 migration Then 路径进入 `affected_paths`，可映射治理面进入 `impact_surface`。 | UC-PG-005 |
| REQ-PG-013 | 系统必须为历史 impact surface 漂移生成 migration report，避免普通 `cr check` 对历史数据刷屏。 | P1 | Given 历史 CR 含路径类 impact surface When 运行常规 `cr check` Then 不刷屏；When 运行 migration report Then 输出可追踪 finding。 | UC-PG-005 |
| REQ-PG-014 | Roadmap refresh 必须使用独立 result schema 和 checker，不复用 CP result checker。 | P1 | Given 生成 `ROADMAP-REFRESH-<id>.result.json` When 运行 roadmap-refresh checker Then 校验 refresh schema、decision、source、machine_updates、follow-up 字段。 | UC-PG-006 |
| REQ-PG-015 | Roadmap refresh decision 必须使用固定枚举。 | P1 | Given refresh 完成 When 写入 result Then decision 只能为 `NO_CHANGE`、`UPDATED`、`UPDATED_WITH_DOC_IMPACTS`、`BLOCKED` 或 `FAILED`。 | UC-PG-006 |
| REQ-PG-016 | Roadmap refresh cascade 自动写入范围必须限制在过程归档库。 | P1 | Given refresh 发现 quant-lab 发布库文档陈旧 When 执行 cascade Then 发布库文件不被自动修改，陈旧项进入 `must_check`、`stale_items` 或 `follow_up_candidates`。 | UC-PG-006 |
| REQ-PG-017 | Roadmap refresh 事件必须写入 `GATE-LEDGER` 或等价 CR 相关 ledger。 | P1 | Given refresh result 写入成功 When 追加 ledger Then ledger event 包含 refresh gate、status 和 result ref。 | UC-PG-006 |
| REQ-PG-018 | FU-RF、SP-RF、RA-RF 候选必须被跟踪模板和检查器接受。 | P1 | Given roadmap refresh 生成 follow-up When 运行 cr-tracking Then `FU-RF001-001`、`SP-RF001-001`、`RA-RF001-001` 被识别且不写入 `RELEASE-CONTEXT`。 | UC-PG-006 |
| REQ-PG-019 | 系统必须提供 project stale check，识别跨对象语义陈旧。 | P1 | Given project state 进入 runtime 或 paper/live 阶段但 HLD / TEST-STRATEGY 仍停留 backtest-only When 运行 stale check Then 输出 stale finding 和 follow-up 候选。 | UC-PG-006 |
| REQ-PG-020 | quant-lab 迁移必须清理污染的 `STATE.current.json` 并通过 state check。 | P2 | Given quant-lab current state 含 forbidden / unknown / oversized 字段 When 执行迁移 Then 清理后通过 state check，并保留迁移报告。 | UC-PG-007 |
| REQ-PG-021 | quant-lab capability 来源必须归一到标准 registry。 | P2 | Given quant-lab capability 线索分散在 CR、代码和研究注册文档中 When 执行迁移 Then 真实 capability 被核查并进入标准 registry 或 blocked follow-up。 | UC-PG-007 |
| REQ-PG-022 | quant-lab 迁移必须输出 roadmap stale report 和 FU-RF 候选，且不得自动改发布库。 | P2 | Given quant-lab 发布库存在陈旧 HLD / TEST / release 文档 When 执行迁移 Then 只输出 stale report 和 FU-RF 候选，不修改发布库文件。 | UC-PG-007 |
| REQ-EI-001 | 系统必须校验 checkpoint、human gate、state 与 ledger 的时序不变量。 | P0 | Given gate/result/state 事件违反前置或因果顺序 When 运行 chronology checker Then 非法 fixture 100% 被拒绝，并定位冲突事件与规则。 | UC-EI-001 |
| REQ-EI-002 | 条件式批准必须保留条件、满足证据和生效事件，未满足条件时不得视为最终批准。 | P0 | Given human approval 含未满足条件 When state transition 尝试进入后续阶段 Then transition 被拒绝；When 条件证据满足并追加生效事件 Then 可重放为有效批准。 | UC-EI-001 |
| REQ-EI-003 | 每个 execution/dispatch attempt 必须具有唯一 ID、生命周期状态和 terminal closure。 | P0 | Given spawn/resume/send_input attempt 已开始 When 工作结束、失败或被替代 Then attempt 必须进入 terminal status，未关闭 attempt 被 checker 阻断。 | UC-EI-002 |
| REQ-EI-004 | retry 与 supersession 必须显式引用被替代 attempt，并保持完整链路。 | P0 | Given attempt 失败后重试 When 写入 retry event Then 新 attempt 引用旧 attempt，旧 attempt terminal，且 final chain 无循环和悬空引用。 | UC-EI-002 |
| REQ-EI-005 | Dispatch evidence 必须表达 platform-attested、session-observed、repository-verifiable 或 unavailable 的证明层级。 | P0 | Given 平台提供可验证 receipt When 检查 dispatch Then receipt 与 attempt correlation 可验证；Given 平台不提供 receipt Then attestation status 为 unavailable 且不被提升为 verified。 | UC-EI-002 |
| REQ-EI-006 | CP result 必须记录 check attempt、适用输入 artifact hash 和 supersession 关系。 | P0 | Given 适用 CP result 被生成 When 运行 result checker Then check attempt 与 input hash 覆盖率为 100%，重复检查通过 supersedes 形成无环链。 | UC-EI-001 |
| REQ-EI-007 | 最终 CP result 必须引用最终 terminal dispatch/check attempt，且 terminal outcome 与 CP decision 兼容。 | P0 | Given 多次 attempt 后产生 final result When 运行 correlation checker Then final ref 只指向最终 terminal attempt；不兼容 outcome/decision 被拒绝。 | UC-EI-001, UC-EI-002 |
| REQ-EI-008 | 系统必须校验 result、checkpoint、gate、state 与 ledger 的跨真相源一致性。 | P0 | Given 任一对象在 decision、CR、checkpoint、attempt 或 ref 上冲突 When 运行 consistency check Then 输出冲突对象和字段，且不得以 summary 覆盖机器真相。 | UC-EI-001 |
| REQ-EI-009 | delivered state 必须清空 active refs，并引用存在且属于当前 CR 的 workflow health 生成物。 | P0 | Given workflow 尝试 finalized/delivered When 运行 state check Then active change/story/delegation/gate refs 为 0，health ref 目标存在且 CR attribution 匹配。 | UC-EI-001 |
| REQ-EI-010 | deny-default read expansion 必须记录授权、原因、CR attribution 和 scoped context slicing。 | P0 | Given agent 展开 deny-default 文档 When 运行 read-expansion check Then 授权引用、允许原因、CR ID 与 scope 均可验证；未授权事件数为 0。 | UC-EI-001 |
| REQ-EI-011 | token usage 必须为每条适用记录声明 `measurement_status`。 | P0 | Given agent/checker/context run 被计量 When 生成 usage record Then status 只能表达 measured、estimated/proxy 或 unavailable，适用记录覆盖率为 100%。 | UC-EI-004 |
| REQ-EI-012 | 只有平台报告的 token usage 可标记为 measured；估算与不可用必须独立表达。 | P0 | Given 仅有文本长度估算或无 telemetry When 生成成本报告 Then measured 字段为空，estimate 仅进入 proxy 字段或 status 为 unavailable。 | UC-EI-004 |
| REQ-EI-013 | 成本记录必须可按 CR、阶段、agent/attempt 和 checker run 归属与聚合。 | P1 | Given 多阶段 workflow usage records When 生成 cost report Then 可按 CR/phase/agent/attempt 聚合，且无法归属项显式列出。 | UC-EI-004 |
| REQ-EI-014 | replay report 必须记录 checker version、commit、schema hash、policy hash 和输入 artifact hash。 | P0 | Given evidence 被重放 When 生成 report Then checker provenance 与所有适用 hash 非空；确实不可取得的字段明确标记 unavailable。 | UC-EI-003 |
| REQ-EI-015 | replay report 必须同时保留 as-executed 与 current-replay 结论及差异分类。 | P0 | Given 当前 checker 与历史 checker 结论不同 When 重放 Then 两个结论均保留，差异被分类且不改写历史结论。 | UC-EI-003 |
| REQ-EI-016 | quant-lab CR-163 迁移必须 append-only，仅追加 manifest、correction、supersession 和 replay evidence。 | P1 | Given 迁移前后历史事件集合 When 校验 immutable prefix/hash Then 原事件逐字节不变，新事件仅追加且回链原对象。 | UC-EI-005 |
| REQ-EI-017 | quant-lab CR-163 current-checker replay fixture 的目标必须为 23/23 PASS。 | P1 | Given migration fixture 完成 When 使用已识别 current checker 重放 Then 23 个目标证据全部 PASS，否则逐项输出 blocker。 | UC-EI-005 |
| REQ-EI-018 | CR-046 不得修改 quant-lab lineage 业务实现。 | P0 | Given CR-163 pilot diff When 按受保护业务路径检查 Then lineage contract、recorder、producer、consumer、admission 源码 diff 为 0。 | UC-EI-005 |
| REQ-EI-019 | Ledger compaction/restore 必须保持 event、dispatch、attempt、run 的类型与关系语义，并保留 terminal attempt selection、correction chain 和 workflow-health refs。 | P0 | Given 含重试、修正和 health ref 的 ledger fixture When compact 后 restore/replay Then 标识类型不混淆、关系图和最终 terminal selection 与 compact 前语义等价；任一丢失或 fallback conflation 100% 被拒绝。 | UC-EI-001, UC-EI-002 |
| REQ-EI-020 | 系统必须提供可复用的 CP8 后 append-only correction lifecycle，而不是仅为 CR-163 定制迁移事件。 | P0 | Given closed workflow 需要修正允许字段 When 追加 correction Then 使用 versioned schema，声明允许字段/范围、author、reason、evidence、supersedes，并形成可独立检查的无环 audit trail；非法原位改写 100% 被拒绝。 | UC-EI-003, UC-EI-005 |
| REQ-EI-021 | 系统必须从 ledgers/results 机器生成 provenance-bearing audit report，并分离 event-row、attempt、thread、terminal outcome 与 token measurement metrics。 | P0 | Given 含 retry、多 event rows 和混合 measured/proxy/unavailable usage 的已知 fixture When 生成 audit report Then 各维计数与 fixture 100% 一致，checker provenance 与输入 hashes 非空，且不得从 dispatch 行数推断 attempt/thread 或 token 占比。 | UC-EI-001, UC-EI-004 |
| REQ-EI-022 | CR-046 CP1/CP2 原始 null-provenance results 必须保留为 first-class dogfooding fixtures，并按新 strict profile 失败或报告 legacy/unavailable。 | P0 | Given 原始 R1 CP1/CP2 result 未含 checker_provenance When strict replay/check 执行 Then 原文件 hash 不变，结果不得标 fully replayable；迁移/current result 只能通过 append-only correction 或新结果记录 measured provenance。 | UC-EI-003 |
| REQ-EI-023 | Dispatch 报告必须披露 session-observed 与 repository-verifiable 的独立状态，以及平台 receipt 不可用时的证明上限。 | P0 | Given agent/thread/tool 仅在当前 Codex session 可观察且仓库无平台 receipt When 生成 dispatch evidence/audit report Then 状态为 `session-observed/repository-unverifiable`，receipt 为 unavailable，且不得声称 repository-verified 或 platform-attested。 | UC-EI-002 |

## 约束需求

| ID | 需求描述 | 优先级 | 验收条件 | 来源 |
|---|---|---|---|---|
| REQ-PG-C001 | 不得新增第二套上下文治理、影响分析、capability 命名、result 或 ledger 体系。 | P0 | Given 设计或实现 diff When 审查新增对象 Then 不出现与现有体系平行的新治理体系。 | 明确排除 |
| REQ-PG-C002 | 不得声明或实现跨仓原子事务。 | P1 | Given roadmap refresh 涉及过程归档库和 quant-lab 发布库 When 审查实现 Then 仅过程归档库自动写入，发布库仅生成 follow-up。 | UC-PG-006 |
| REQ-PG-C003 | 不得直接修改 `process/policies/GATE-PROFILES.json` 表达 project scale。 | P1 | Given project scale 实现完成 When 审查文件变更 Then gate profile bias 位于 `PROJECT-SCALE.yaml`。 | UC-PG-003 |
| REQ-PG-C004 | migration 不得凭空创造 capability 或 feature ID。 | P1 | Given registry 缺失目标 ID When 运行 migration Then 输出 blocked finding 或 FU-RF，不创建未经确认 ID。 | UC-PG-004 |
| REQ-PG-C005 | Ledger compaction 不得复用 `meta-flow state compact` 名称。 | P0 | Given 新增 ledger compaction 命令 When 检查 CLI Then 命令为 `ledger compact` 或 `event compact` 等独立名称。 | 实施计划 P0 后段 |
| REQ-EI-C001 | 不得伪造或推断不存在的 platform receipt、签名、token telemetry 或 checker identity。 | P0 | Given 来源字段缺失 When 生成 evidence Then 使用 unavailable/unknown 的合法表达，不生成看似 verified/measured 的值。 | UC-EI-002, UC-EI-003, UC-EI-004 |
| REQ-EI-C002 | canonical checker 必须以 canonical JSON CR index 为输入，不得被 legacy YAML 状态干扰。 | P1 | Given JSON 与 legacy YAML 并存或冲突 When 运行 canonical checker Then 只消费 canonical JSON，并对 legacy 干扰给出迁移提示。 | UC-EI-003 |
| REQ-EI-C003 | CP2 approval 不授权 credentials、runtime、production write、publish、交易、repository publication 或 quant-lab business-code changes。 | P0 | Given 用户批准 CP2 When 检查授权边界 Then 上述动作仍为未授权，必须独立获得授权。 | UC-EI-005 |

## 非功能需求

| ID | 需求描述 | 优先级 | 验收条件 | 来源 |
|---|---|---|---|---|
| REQ-PG-NF001 | 所有新增机器状态对象必须 refs-only、可校验、可预算。 | P0 | Given state / project state 写入 When 运行 checker Then 长正文、历史详情和大型列表被拒绝或转移到 ledger / result / summary。 | UC-PG-001, UC-PG-002 |
| REQ-PG-NF002 | 所有检查失败必须输出可定位的字段、路径或 ID。 | P1 | Given schema、registry、surface 或 refresh 检查失败 When 查看结果 Then 能定位失败字段、输入来源和建议处理方式。 | UC-PG-004, UC-PG-005, UC-PG-006 |
| REQ-PG-NF003 | P0 enforcement 必须支持 audit -> enforce 灰度。 | P0 | Given unknown field 出现 When 处于 audit 阶段 Then WARN 不阻断；When 处于 enforce 阶段 Then ERROR 阻断。 | UC-PG-001 |
| REQ-PG-NF004 | quant-lab 迁移必须可重复验证。 | P2 | Given 迁移完成 When 重新运行 state / capability / feature / capability-claims / stale checks Then 结果与迁移报告一致。 | UC-PG-007 |
| REQ-EI-NF001 | evidence-integrity checker 失败必须包含对象 ID、失败字段、规则 ID、证据 ref 和安全路由。 | P0 | Given 任一 chronology/provenance/replay/telemetry 检查失败 When 输出 finding Then 审计者可定位原对象、原因与回退/修复入口。 | UC-EI-001..005 |
| REQ-EI-NF002 | 新 evidence contract 必须具备 versioned schema 和 legacy read-only compatibility fixture。 | P0 | Given 旧证据进入新 checker When 执行兼容检查 Then 可读对象被重放，不兼容对象显式 BLOCKED/WARN，不静默改写。 | UC-EI-003, UC-EI-005 |

## 风险与假设

| ID | 类型 | 内容 | 关联需求 | 缓解措施 |
|---|---|---|---|---|
| RA-PG-001 | RISK | allowlist 可能误伤存量但仍合理的 current state 字段。 | REQ-PG-001 | 使用 audit -> enforce 灰度，先 WARN 收集真实字段。 |
| RA-PG-002 | RISK | 历史 impact surface 漂移数量大，直接阻断会干扰当前开发。 | REQ-PG-013 | 历史静默，提供独立 migration report。 |
| RA-PG-003 | RISK | capability ID 归一时真实来源分散，容易误建自由字符串。 | REQ-PG-011, REQ-PG-021 | 只允许 registry 引用；缺失项输出 blocked finding / FU-RF。 |
| RA-PG-004 | RISK | Roadmap refresh 跨仓边界不清会误改发布库。 | REQ-PG-016, REQ-PG-C002 | 明确自动写入只限过程归档库。 |
| RA-PG-005 | ASSUMPTION | 已批准实施计划是本轮产品基线事实来源。 | 全部 | 本文档引用 source_plan；若计划变更，必须通过后续 CR 增量更新。 |

## 里程碑建议

| 里程碑 | 包含需求 | 交付物 | 前置里程碑 |
|---|---|---|---|
| M0：Second-system Guardrail | REQ-PG-C001 | 设计 / 实现审查阻断第二套上下文治理、影响分析、capability 命名、result 或 ledger 体系 | 无 |
| M1：Current State Enforcement | REQ-PG-001..006, REQ-PG-NF001, REQ-PG-NF003 | allowlist、field budgets、update API、写契约、audit/enforce gate | 无 |
| M1.5：Ledger Hygiene Boundary | REQ-PG-C005 | ledger compact / event compact 与 state compact 的命令边界和帮助文档职责区分 | M1 |
| M2：Project Governance Objects | REQ-PG-007..009, REQ-PG-NF001 | `process/project/`、`PROJECT.current.json`、`PROJECT-SCALE.yaml`、ROADMAP / MILESTONES 基线 | M1 |
| M3：Reference and Impact Normalization | REQ-PG-010..013, REQ-PG-C004, REQ-PG-NF002 | capability / feature registry 校验、impact surface migration report | M2 |
| M4：Roadmap Refresh and Follow-up | REQ-PG-014..019, REQ-PG-C002 | ROADMAP-REFRESH result/checker、GATE-LEDGER event、FU-RF 支持、stale check | M3 |
| M5：quant-lab Migration | REQ-PG-020..022, REQ-PG-NF004 | quant-lab state 清理、capability 归一、impact surface 迁移、stale report | M4 |

## 明确排除项

- 不做代码实现细节、模块拆分、CLI 参数设计或数据模型字段类型细化；这些属于设计 / 实现阶段。
- 不在本轮产品文档中修改已批准设计文档。
- 不把 quant-lab 迁移和 capability 新能力实现混为同一个交付范围。
- 不把发布库 stale finding 自动升级为发布库写入授权。
