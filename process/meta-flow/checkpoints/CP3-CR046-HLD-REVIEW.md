---
checkpoint_id: "CP3-CR046-HLD-REVIEW"
checkpoint_name: "CR-046 Evidence Integrity Architecture Review"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-07-12T01:55:00Z"
reviewed_by: "user"
reviewed_at: "2026-07-12T03:17:27Z"
auto_check_result: "process/checks/CP3-CR046-HLD-CONSISTENCY-R3.result.json"
target:
  phase: "solution-design"
  story_id: ""
  artifacts: ["docs/design/CR046-EVIDENCE-INTEGRITY-BLUEPRINT.md", "docs/design/CR046-EVIDENCE-INTEGRITY-DOMAIN-MAP.md", "docs/design/CR046-EVIDENCE-INTEGRITY-DEPENDENCY-MAP.md", "docs/design/CR046-EVIDENCE-INTEGRITY-HLD.md", "docs/design/CR046-EVIDENCE-INTEGRITY-ARCHITECTURE-DECISION.md", "docs/design/CR046-CODEX-SUBAGENT-PLATFORM-CONTRACT.md"]
---

# CP3 CR-046 Evidence Integrity Architecture Review

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP3-CR046-HLD-CONSISTENCY-R3.result.json` | PASS | 0 | check attempt 3、9 个 input hashes、checker provenance、6 项 R3 架构检查通过；supersedes R2 |
| `process/checks/CP3-CR046-DISCUSSION-CHECKPOINT.json` + R3 design delta | ready-for-host-broker | 0 | 6 个 gray areas 收敛为 5 个 CP3 决策 |
| `process/context/CP3-CR046-DESIGN-CONTEXT.yaml` | design-draft-ready | 0 | compact capsule；read expansion 已按 deep_review 校正并通过 policy check |

## Decision Brief

### 审批者摘要

| 字段 | 内容 |
|---|---|
| 本次确认服务的整体目标 | 冻结 CR-046 的 typed evidence、routing、phase/gate、Codex 平台契约/discovery/thread reuse、dogfooding、compaction、correction、audit/replay/telemetry 架构，作为 CP4/CP5 设计输入 |
| 推荐动作 | `approve` 五项推荐方案；它们共同保持现有 ledgers/results/state 为事实源，同时消除 identity、route、custom-agent 自证和 post-close split-brain，并采用 A-baseline + Conditional-B 自举策略 |
| approve 后会发生什么 | meta-se 自动进入 CP4，创建 FEATURE-DESIGN-MATRIX、七 Story 技术拆分、DAG、文件 owner、Wave、TASK-ID 和 DEVELOPMENT-PLAN；完成自动 CP4 后准备全量 CP5 设计证据，不直接实现 |
| approve 不授权什么 | 代码实现、LLD 省略、runtime、credentials、production write、publish、交易、commit/push、quant-lab lineage 修改或真实 CR-163 pilot |
| 不确认会阻塞什么 | 阻塞 CP4、Feature design、Story 拆分、LLD 和后续实现 |

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP3-CR046-DESIGN-CONTEXT.yaml` |
| capsule 状态 | design-draft-ready-for-host-gate-preparation |
| read_profile | compact |
| 默认读取策略 | capsule-first；只在架构 traceability/deep review 时展开产品和设计全文 |
| 全文档读取扩展 | `RE-20260712T015333Z0000-aa4ca88d`，reason=`deep_review`，policy check PASS |
| 缺失 / waived 理由 | blocker=0；waiver=0 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `STATE.current.json` / gate ledger | scanned | 1 | 0 | CP3 尚未打开，Host 正在准备本门 |
| 委托 Agent 交还摘要 | R1/R2 summary + `CR046-CP3-R3-CODEX-PLATFORM-CONTRACT-META-SE-RETURN-SUMMARY.md` | scanned | 5 | 5 | DQ-01..05；R3 generic spawn 仅 session-observed，不冒充 meta-se-critical attestation |
| 自动预检结果 | CP3 result | scanned | 0 | 0 | PASS，blocker=0 |
| discussion log/checkpoint | CP3 discussion artifacts | scanned | 6 | 5 | AGA-01/03 合并到 DQ-01；AGA-02→DQ-02；AGA-04→DQ-03；AGA-05→DQ-04；AGA-06→DQ-05 |
| 下游正式产物 | 六份 CR046 design truth + delivery platform contract | scanned | 11 | 5 | 十一项 ADR 中五组需要本门确认；其余为推荐方案的派生 contract |
| 用户显式输入 | CP2 approval condition | scanned | 1 | 1 | routing compatibility 必须在 CP3 决策，不得延后 |

### 决策分层

| 分类 | 数量 | 处理方式 |
|---|---:|---|
| 必须用户决策 | 5 | CP3-DQ-01..05 |
| 高风险策略确认 | 3 | DQ-02 routing metadata/migration boundary；DQ-04 platform attestation/thread reuse；DQ-05 dogfooding evidence ceiling |
| agent 默认处理 | 6 | 已形成 proposed ADR，由五项核心决策派生 |
| 仅审计记录 | 5 | 六项模拟、hash/provenance、receipt/telemetry degradation、pilot authz boundary |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| CP3-DQ-01 | architecture | 是否采用 existing canonical truth + shared typed evidence contract，并区分 phase-in-progress 与 opened gate？ | 保留 ledgers/results/state 为事实源；增加 typed identity graph、shared checker pipeline、RouteTruth；合法运行中间态不提前打开 gate | A. checker-local patches；B. 新 evidence DB | 推荐不复制事实且统一语义，代价是跨模块 contract 改动；A 快但继续 split-brain；B 查询强但形成第二真相源和迁移负担 | shared checker blast radius、legacy strict adoption | enforce 可回 audit；只有性能 SLO 实证失败才另立只读索引 CR；禁止直接切换为写入型 evidence DB |
| CP3-DQ-02 | architecture | `routing_ref` local-directory compatibility 应采用哪种机器契约？ | local-directory 也必须有真实、portable-relative metadata；非空 routing_ref 必须可解析；workspace/state/doctor/CP 共用 RouteTruth resolver；dangling 在 strict/compat 均 FAIL/BLOCKED | A. `routing_ref=null` + explicit legacy-local-directory + migration deadline；B. artifact root 确认后迁移标准 symlink | 主选保持统一契约并最小化迁移；A 可承载暂时不可描述布局但降低能力且必须有期限；B 最终最整洁但迁移/回滚成本最高 | 当前正存在 non-null dangling ref 与 workspace PASS 的 split-brain；错误迁移可能影响过程路径 | candidate metadata dry-run 后原子切换；失败不应用；冲突回旧 state hash并进入 explicit legacy-null，绝不恢复 dangling PASS；可准确描述用主选，不可描述用 A，artifact root/迁移检查就绪用 B |
| CP3-DQ-03 | architecture | 是否采用通用 versioned append-only post-close correction lifecycle？ | allowlist correction scope + author/reason/evidence/supersedes + acyclic audit trail；原始事件不可修改；CR-163 pilot 后续消费通用 contract | A. 真实 rerun result 使用新 result supersedes，仅限 result 对象；B. 禁止任何关闭后修正 | 推荐覆盖非运行事实修正且保留历史；A 对真实重新执行最准确但不能覆盖 checkpoint/gate/dispatch/state correction；B 简单但无法治理已知历史错误 | correction abuse、循环 supersession、错误扩大 CP8 后修改范围 | 非法字段/环/缺 evidence 直接拒绝；合法错误 correction 通过新 superseding correction 追加，不删除历史；真实 rerun 优先用 A |
| CP3-DQ-04 | architecture | 如何形式化并证明 Meta Flow 真正调用了指定 Codex custom agent及其模型？ | 采用规范性 `CR046-CODEX-SUBAGENT-PLATFORM-CONTRACT.md`：区分 CURRENT/REQUIRED；D0 平台 session discovery 是唯一 `PROFILE_DISCOVERED` 来源；显式 selector 和绑定 dispatch/attempt 的 spawn/reuse receipt；不可变 ThreadRuntimeIdentity；profile mismatch 必须新 spawn | A. preferred/default profile 经用户明确批准 generic fallback，标 `degraded-unattested`；B. required critical/debugger 阻塞直到 D0/selector/receipt 可用 | 主选给平台团队明确接口与 Meta Flow 验收标准；D2 TOML scan 只证明安装；reuse receipt 防止 followup/账本改名伪升级 | 当前工具只暴露 task_name/message/fork_turns；公开契约没有 runtime discovery/selector/receipt；required extension 的交付时序不在 Meta Flow 控制内 | 会话 reload、config hash变化或 receipt mismatch 撤销旧 discovery；reuse identity mismatch 新 spawn；critical/debugger 不得走 generic fallback |
| CP3-DQ-05 | architecture | CR-046 自己在平台扩展尚未提供时，CP6/CP7 应采用哪种狗粮策略？ | A-baseline + Conditional-B：仓库 contract/checker/PC-01..17 不依赖平台扩展；若 CP7 前 D0+selector+spawn/reuse receipt 全可用且 17/17 PASS，则新 spawn 已验证 `meta-qa-critical` 做 runtime conformance | A. pure A：全生命周期 session-observed；B. pure B：无平台扩展就阻塞 CR | 推荐方案诚实且可交付，并在能力到位时完成理想自举；pure A 留下 runtime proof 风险；pure B 证明最强但把治理 CR 绑定外部平台时序 | A-baseline 时 CP7 最高 PASS_WITH_RISK、CP8 最高 READY_WITH_RISK；不得复用未证明 profile 的既有 QA thread | 任一 capability/receipt/hash 退化即回 A；保留 repository evidence，不改写旧 dispatch；只有全部 switch 条件满足才启用 B |

| 字段 | 内容 |
|---|---|
| 推荐决策 | 批准 CP3-DQ-01、CP3-DQ-02、CP3-DQ-03、CP3-DQ-04、CP3-DQ-05 的推荐方案 |
| 备选方案 | 每项均有可执行备选与 switch condition，详见上表 |
| 影响维度 | 架构边界、兼容迁移、机器可证性、失败关闭、维护成本、平台可移植性、历史不可变 |
| 优劣分析 | 推荐组合不引入第二事实数据库，同时补齐 route/correction/identity 公共 contract；代价是 CP4/CP5 设计与跨模块测试面较大 |
| 风险与回退 | audit-first、candidate/dry-run、原子切换、explicit legacy-null、append-only superseding correction |
| 用户需决策事项 | `CP3-DQ-01`、`CP3-DQ-02`、`CP3-DQ-03`、`CP3-DQ-04`、`CP3-DQ-05` |

### CP3 架构摘要

- 方案适用条件：现有 ledgers/results/state 仍是权威事实；需要跨 checker 共享 identity/correlation/route semantics。
- 优化项：语义一致、历史可重放、报告可机器生成、token measurement honest。
- 牺牲项：需跨 `event/cp/state/workflow/context` 模块共同升级；legacy strict adoption 有迁移成本。
- Use Case → Architecture：UC-EI-001..005 全部映射到四个 Feature 与七 Story technical boundaries。
- 场景模拟：phase/gate、dangling route、compaction restore、null provenance、receipt ceiling、CR-163 pilot、selector absent、receipt/hash mismatch、D2≠D0、followup profile mutation、CP7 dogfood，设计推演 11/11 PASS；不是执行证据。
- Story DAG input：`001→002→003→004`；`002→005`；`003+004+005→006`；`004+006→007`，设计上无环。
- 讨论证据：`process/discussions/CP3-CR046-HLD-DISCUSSION-LOG.md` 和 discussion checkpoint。

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| CP2 R2 approved | PASS | CP2 R2 checkpoint/gate ledger |  |
| 六份架构真相源和 delivery platform contract 存在 | PASS | artifact design refs + `delivery/doc/PLATFORM-CONTRACTS.yaml` |  |
| CP3 result/checker provenance/hash 通过 | PASS | CP3 R3 result + Host 9/9 hash check | R1/R2 保留且由 R3 supersede |
| 用户要求的 routing 决策已进入 DQ | PASS | CP3-DQ-02 | 不得延后 CP8 |
| blockers/waivers 为 0 | PASS | CP3 result |  |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | typed evidence 与事实源边界 | 通过 | DQ-01/HLD/ADR-001 | 接受推荐方案 |
| 2 | phase-in-progress / gate-open 时序 | 通过 | DQ-01/ADR-003 | 接受推荐方案 |
| 3 | routing local-directory compatibility | 通过 | DQ-02/ADR-004 | 接受推荐方案 |
| 4 | compaction semantic preservation | 通过 | ADR-005 | 接受推荐方案 |
| 5 | post-close correction lifecycle | 通过 | DQ-03/ADR-006 | 接受推荐方案 |
| 6 | Codex 平台 contract 的 CURRENT/REQUIRED、D0-D3 discovery 与 selector/receipt | 通过 | DQ-04/ADR-010/platform contract | D2 config scan 不得提升为 D0 discovery；freshness 细节进入 CP4/CP5 |
| 7 | ThreadRuntimeIdentity、followup reuse receipt 与 profile mismatch new-spawn | 通过 | DQ-04/ADR-010/HLD §5.1A | 当前 CR-046 dispatch 仅 session-observed；无 reuse receipt 负例进入 CP4/CP5 |
| 8 | CR-046 A-baseline + Conditional-B 狗粮策略及证据结论上限 | 通过 | DQ-05/ADR-011/HLD §5.1B | 接受推荐方案 |
| 9 | generated audit/replay/token telemetry | 通过 | ADR-007/HLD |  |
| 10 | 七 Story technical boundary/DAG | 通过 | Blueprint/Dependency Map | legacy codex_agent_name D3 迁移规则进入 S09/CP5 |
| 11 | authz 与 CR-163 pilot 边界 | 通过 | ADR-009 |  |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| 五项核心架构决策关闭 | 通过 | DQ-01..05 | 用户批准全部推荐方案 |
| 六份设计与平台 contract 可作为 CP4 输入 | 通过 | CP3 result |  |
| CP4/CP5 前置边界明确 | 通过 | HLD/DAG input | 三个 minor 作为 CP4/CP5 强制设计细化项 |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| Blueprint | artifact `CR046-EVIDENCE-INTEGRITY-BLUEPRINT.md` | 通过 |  |
| Domain Map | artifact `CR046-EVIDENCE-INTEGRITY-DOMAIN-MAP.md` | 通过 |  |
| Dependency Map | artifact `CR046-EVIDENCE-INTEGRITY-DEPENDENCY-MAP.md` | 通过 |  |
| HLD | artifact `CR046-EVIDENCE-INTEGRITY-HLD.md` | 通过 |  |
| ADR | artifact `CR046-EVIDENCE-INTEGRITY-ARCHITECTURE-DECISION.md` | 通过 |  |
| Codex 子 Agent 平台契约 | artifact `CR046-CODEX-SUBAGENT-PLATFORM-CONTRACT.md` + `delivery/doc/PLATFORM-CONTRACTS.yaml` | 通过 | CURRENT/REQUIRED 分层、PC-01..17；CP4 扩充 edge fixtures |
| CP3 result/capsule | process refs | PASS |  |

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-07-12T03:17:27Z
- 修改意见：三个 minor 不阻塞 CP3，必须在 CP4/CP5 补齐：D0 freshness/重新发现触发；followup 无 reuse receipt 负例；legacy `codex_agent_name` 按 D3/self-declared-unverifiable 迁移。
- 风险接受项：接受 A-baseline 下 runtime custom-agent/model attestation unavailable 的明确证据上限；不接受虚报 verified。
