---
status: "approved"
version: "1.1"
cr_id: "CR-157"
title: "Stage 2 Multifactor Research Framework Upgrade HLD"
complexity: "standard"
selected_option: "contract-first Stage 2 mature admission and Stage 3 handoff hardening"
confirmed: true
confirmed_by: "user"
confirmed_at: "2026-07-05T12:48:20+08:00"
source_cp2: "process/checkpoints/CP2-CR157-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE-SCOPE.md"
discussion_log: "process/discussions/CP3-CR157-HLD-DISCUSSION-LOG.md"
discussion_checkpoint: "process/checks/CP3-CR157-DISCUSSION-CHECKPOINT.json"
authorization_boundary: "design-only/static-fixture-only; no implementation, real lake write, NAS/provider/credential/QMT/runtime/simulation/live/trading/broker/external framework/Git remote/catalog/store/registry operation"
---

# HLD: CR157 Stage 2 Multifactor Research Framework Upgrade

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-05 | host-orchestrator | 初版：基于 CP2 approved first slice，定义成熟准入包 builder、research evidence traceability、Stage 2/3 handoff hardening 和 no-runtime guard 的 CP3 架构方案。 |
| 1.1 | 2026-07-05 | host-orchestrator | CP3 人工确认通过：接受 contract-first Stage 2 exit hardening、refs-only evidence、fail-closed handoff、event/ML adapter deferred 和 no-runtime boundary。 |

## 1. 问题定义

### 问题陈述

CR150 已完成多因子基础合同，CR151 已补齐统计准入门控，CR154 已补齐跨策略可靠性门控，CR155 提供了 daily baseline strategy artifact 的第一条可审计策略样例。当前 Stage 2 仍有一个结构性缺口：`engine/mature_multifactor_framework.py` 已有 `MatureAdmissionSupport`、`ResearchEvidenceIndex`、`Stage3ResearchMachineHandoff` 和 `Stage3MatureResearchPackage` 等对象，但还需要把它们提升为一个明确的 Stage 2 出口合同，保证后续 Stage 3 研究机能按 refs-only evidence、fail-closed handoff 和 no-runtime guard 接收工作，而不把 Stage 2 误读为真实数据、运行时或交易授权。

CR157 的 CP3 目标是确认这套 Stage 2 出口架构。它不新增真实数据接入，不运行策略，不做交易，不发布 catalog，也不实现 event / ML adapter。

### 核心价值

CR157 让多因子研究框架从“对象已存在”推进到“出口可准入、证据可追踪、交接可失败关闭”。后续开发可以围绕明确的 builder、evidence index、handoff 和 guardrail 切 Story，避免把 Stage 2 fixture/static 合同扩展成未经授权的 Stage 3 数据或 runtime 行为。

### 目标

| 优先级 | 目标 | 度量方式 |
|---|---|---|
| P0 | Mature admission package builder 合同明确 | 1 个 builder 入口能产出 `Stage2MatureFrameworkBundle`，包含 `MatureAdmissionSupport`、`StrategyCandidate`、`Stage3ResearchMachineHandoff` 和 CR030/CR039 refs。 |
| P0 | Research evidence traceability 收紧 | `ResearchEvidenceIndex` 至少覆盖 `data_release_ref`、`run_manifest_ref`、`metric_refs`、`lineage_refs`、`limitations`、`typed_unavailable` 6 类字段，并保持 refs-only。 |
| P0 | Stage 2/Stage 3 handoff fail-closed | Stage 3 handoff 明确 `required_inputs=12`、`required_evidence=13`、blocked_until 条件和 no-runtime execution boundary。 |
| P0 | No-runtime guard 可测试 | 禁止操作计数中 lake write、provider fetch、catalog publish、QMT operation、simulation/live、credential read 均为 0；任一非 0 必须 BLOCKED。 |
| P1 | BLUEPRINT Stage 2 退出条件可追溯 | CP3 文档列出 CR150/151 已满足项和 CR157 关闭项，且没有声明 event/ML adapter 已实现。 |
| P2 | Deferred adapter 决策稳定 | DF-CR157-001 event adapter 与 DF-CR157-002 ML adapter 均记录为 backlog，不进入本 CR CP4/CP5。 |

### 成功标准

- [ ] 100% CR157 Story 切片必须映射到 FEAT-03、FEAT-13 或 FEAT-14，不新建无 owner Feature。
- [ ] 100% CP6/CP7 验证使用 static/fixture/local tests，不读取 `.env`、真实 lake、NAS、provider、QMT、broker 或外部框架。
- [ ] 100% Stage 3 package/handoff 必须保留 `not_runtime_authorization=true`、`not_simulation_authorization=true`、`not_live_authorization=true`。
- [ ] 13 个 `STAGE3_REQUIRED_EVIDENCE` 缺任一真实 ref 时，Stage 3 mature research package 必须 fail-closed。
- [ ] 12 个 `STAGE2_DATA_REQUIREMENTS` 缺任一真实 ref 时，Stage 2 evidence index 必须输出 typed unavailable，不得 silent pass。

### 非目标

- 不实现 event-driven strategy adapter；记录为 `DF-CR157-001`。
- 不实现 machine-learning strategy adapter；记录为 `DF-CR157-002`。
- 不连接真实数据湖，不写 lake，不 publish catalog，不读取 provider/NAS/credential。
- 不启动 QMT/MiniQMT/xtquant/gateway，不进行 simulation、paper、live 或 trading。
- 不把 fixture PASS 表述为真实收益可靠性、生产准入或 runtime readiness。
- 不关闭后续 Stage 3 真实数据研究、Stage 4 simulation 或 live switch gate。

### 关键约束

| 类型 | 约束内容 |
|---|---|
| Stage boundary | Stage 2 只冻结合同和 fixture/static guard；Stage 3 才能在研究机补齐真实 data release、PIT universe、factor panel、label window、evaluation 和 portfolio/risk evidence。 |
| 技术 | 复用 `engine/mature_multifactor_framework.py`、`engine/multifactor_contracts.py` 和现有 pytest/uv stack；不引入新运行时框架。 |
| 权限 | CP3 为 design-only；CP3 approve 只进入 CP4 Story planning。 |
| 兼容 | 保持 CR030/CR039 package schema refs、CR151 statistical gate 和 CR154 reliability gate 的上下游关系，不重写历史语义。 |

## 2. BLUEPRINT Stage 2 退出条件追踪

| Stage 2 退出条件 | 现状 | CR157 处理 |
|---|---|---|
| FactorSpec / FactorRunSpec | CR150 已覆盖，`FactorSpec` 已被 `build_stage2_mature_admission_support` 校验。 | 引用，不重做。 |
| factor panel / label window | CR150/CR151 作为研究链路与统计门控输入，Stage 2 缺真实数据时 typed unavailable。 | 在 evidence/handoff 中要求 refs。 |
| evaluation | CR151 统计准入门控已覆盖 Wave A；CR154 可靠性门控覆盖生产可靠性 first wave。 | 只追踪 evidence refs，不新增统计计算。 |
| portfolio / risk | `PortfolioRiskPolicy` 已存在，真实 capacity/fee/slippage 仍为 Stage 3 required。 | 强化 refs 和 fail-closed。 |
| admission package | CR030/CR039 历史包 + CR151/154 gate refs 可作为来源。 | CR157 关闭 mature admission package builder 出口。 |
| evidence index | 对象已存在但需要 refs-only 语义和缺口闭环。 | CR157 关闭 evidence traceability。 |
| Stage 2/3 handoff | 对象已存在但需要作为稳定出口审查。 | CR157 关闭 handoff hardening。 |

## 3. Architecture Gray Areas

**讨论日志**：`process/discussions/CP3-CR157-HLD-DISCUSSION-LOG.md`

**讨论恢复点**：`process/checks/CP3-CR157-DISCUSSION-CHECKPOINT.json`

| 灰区 ID | 关键问题 | 推荐结论 | 状态 |
|---|---|---|---|
| AGA-CR157-01 | 本轮应做新 builder 还是只补文档？ | 做 contract-first builder hardening；文档-only 不能关闭 Stage 2 出口缺口。 | selected |
| AGA-CR157-02 | Evidence index 是否复制证据正文？ | refs-only，不复制正文；缺真实 ref 输出 typed unavailable 或 blocked。 | selected |
| AGA-CR157-03 | Stage 3 handoff 是否改名或重塑 schema？ | 保持现有 schema 名，增强字段和验证，不破坏历史引用。 | selected |
| AGA-CR157-04 | event / ML adapter 是否顺手纳入？ | 不纳入，分别 deferred。 | selected |
| AGA-CR157-05 | no-runtime guard 放在哪里？ | 放在 core contract validation + tests + CP release wording 三层。 | selected |

### Advisor Table

| 议题 | 方案 A 推荐 | 方案 B | 方案 C | 推荐理由 |
|---|---|---|---|---|
| Builder scope | 在 `mature_multifactor_framework` 现有对象上收紧 builder 和 validator。 | 只写 HLD/ADR。 | 直接实现跨策略通用 adapter 层。 | A 最小闭合 Stage 2 出口；B 留下代码缺口；C 超出 CP2。 |
| Evidence semantics | refs-only index + typed unavailable + limitations。 | 内联复制 metric/report 内容。 | 允许 Stage 2 读 lake 补真实 evidence。 | A 可审计且不越权；B 增加漂移；C 违反 Stage 2 边界。 |
| Stage 3 transition | Handoff declares required inputs/evidence and blocked_until. | Stage 2 package 直接声明 Stage 4 ready。 | Stage 3 自动运行研究机。 | A 保持 stage gate；B/C 越权。 |
| Runtime safety | Validation fail-closed plus forbidden import/counter tests。 | 只靠文档声明。 | 运行时黑名单扫描覆盖真实操作。 | A 可测试；B 证据弱；C 可能引入不必要 runtime surface。 |

### Deferred Architecture Ideas

| ID | 内容 | 延后原因 | 触发条件 |
|---|---|---|---|
| DF-CR157-001 | Event-driven strategy adapter | FEAT-13 跨策略 adapter 需要事件 feed、event-time semantics 和 event evidence contract；本 CR 只关闭 multifactor Stage 2 出口。 | CR153 follow-up 或用户要求 event adapter Stage 2 package。 |
| DF-CR157-002 | ML strategy adapter | ML adapter 涉及 training snapshot、feature store/model registry、leakage controls 和 model evidence，不应塞入 multifactor first slice。 | CR152 follow-up 或用户要求 ML adapter Stage 2 package。 |
| DF-CR157-003 | Real data evidence fill | Stage 2 禁止真实 lake/provider 访问；真实 ref 在 Stage 3 研究机补齐。 | Stage 3 research-machine authorization。 |
| DF-CR157-004 | Runtime/simulation readiness | 需要独立 runtime_authorization gate 和 QMT/gateway evidence。 | Stage 4 simulation or later runtime CR。 |

## 4. 候选架构方案对比

### 方案 A：Contract-First Stage 2 Exit Hardening

在现有 `engine/mature_multifactor_framework.py` 上强化 builder、refs-only evidence、Stage 3 handoff/package validation 和 no-runtime guard。CP5 再拆 Story 做代码与测试。

| 维度 | 评估 |
|---|---|
| 优点 | 复用现有对象；变更集中；能关闭 Stage 2 出口；和 CR150/151/154/155 连续。 |
| 缺点 | 需要仔细维护 schema 兼容和历史命名。 |
| 复杂度 | standard |
| 风险 | 若 builder 语义过宽，可能被误读为生产准入；通过 blocked claims 和 release wording 限制。 |

### 方案 B：Documentation-Only Handoff

只更新文档说明 Stage 2/3 边界，不修改 builder/validator/tests。

| 维度 | 评估 |
|---|---|
| 优点 | 最快。 |
| 缺点 | 不能用自动测试证明 evidence/handoff fail-closed；后续 Stage 3 消费方仍需猜字段。 |
| 适用前提 | 仅做流程审计，不推进 Stage 2 framework。 |

### 方案 C：Cross-Strategy Adapter Platform Now

一次性定义 event / ML / multifactor 通用 adapter 层，并统一输出 Stage 2 package。

| 维度 | 评估 |
|---|---|
| 优点 | 长期抽象更统一。 |
| 缺点 | 过早耦合 FEAT-13，多出 event-time、training snapshot、model registry 等未确认合同。 |
| 适用前提 | CP2 明确改为跨策略 adapter platform。 |

### 对比矩阵

| 维度 | 方案 A | 方案 B | 方案 C |
|---|---|---|---|
| 关闭 CP2 first slice | high | low | medium |
| Stage 2 权限边界 | high | medium | medium-low |
| 可测试性 | high | low | medium |
| 变更范围 | medium | low | high |
| schema 兼容 | high | high | low-medium |
| 推荐度 | selected | rejected | deferred |

**推荐方案**：方案 A。它是最小可验证闭环：不扩大授权，不提前做 event/ML adapter，又能让 Stage 2 mature admission package、evidence index 和 Stage 3 handoff 成为可消费出口。

## 5. 推荐架构

### 5.1 模块职责

| 模块 / 对象 | 责任 | 输入契约 | 输出契约 | 失败 / 降级 |
|---|---|---|---|---|
| `build_mature_admission_support_from_cr030_cr039_outputs` | 组装 Stage 2 mature framework bundle。 | strategy_id、FactorSpec refs、CR039 candidate、SignalSet、ResearchEvidenceIndex、PortfolioRiskPolicy、CR030/CR039 package refs、permission counters。 | `Stage2MatureFrameworkBundle`。 | 任何输入或权限违规 -> bundle blocked。 |
| `MatureAdmissionSupport` | 表达 Stage 2 准入支撑状态，不等于生产准入。 | adapter、signal、evidence、risk、factor spec validation。 | package_id、refs、stage3_data_requirements、typed_unavailable、blocked_reasons。 | 缺 mandatory field 或 factor spec invalid -> blocked。 |
| `ResearchEvidenceIndex` | 记录研究证据索引，保持 refs-only。 | data_release_ref、run_manifest_ref、metric_refs、lineage_refs。 | index_id、limitations、typed_unavailable、not_catalog_publish、not_data_lake_write。 | 缺 Stage 3 真实 evidence -> typed_unavailable，不做 lake read。 |
| `Stage3ResearchMachineHandoff` | 把 Stage 2 出口转换为 Stage 3 研究机待办。 | support、strategy candidate、evidence index、risk policy。 | required_inputs、required_evidence、execution_boundary、blocked_until、validation_plan。 | 缺 support/candidate -> blocked。 |
| `Stage3MatureResearchPackage` | Stage 3 补齐真实 refs 后的成熟研究包。 | run_manifest、input_refs、evidence_refs、signal/candidate/evidence/risk refs。 | package_id、blocked_claims、unlock_conditions、status。 | 缺 12 类 input 或 13 类 evidence -> blocked。 |
| no-runtime guard | 阻断未授权操作。 | `PermissionCounters`、source import scan、blocked claims。 | `Stage2ValidationResult` / test evidence。 | 任一 forbidden counter 非 0 -> blocked。 |

### 5.2 主要数据流

```text
CR030/CR039 candidate refs
        |
        v
FactorSpec + SignalSet + ResearchEvidenceIndex + PortfolioRiskPolicy
        |
        v
MatureAdmissionSupport + StrategyCandidate
        |
        v
Stage2MatureFrameworkBundle
        |
        v
Stage3ResearchMachineHandoff
        |
        v
Stage3 research machine fills 12 inputs + 13 evidence refs
        |
        v
Stage3MatureResearchPackage
        |
        v
Future Stage 4 review / simulation authorization gate
```

### 5.3 Stage 2/Stage 3 边界

| 能力 | Stage 2 CR157 | Stage 3 future |
|---|---|---|
| FactorSpec validation | yes, static contract | yes, with real lineage refs |
| PIT universe | typed unavailable / required input | real ref required |
| factor panel / label window | evidence ref required, no real build | real artifact required |
| IC/RankIC/layered returns/turnover/exposure | evidence refs only | real evidence required |
| mature admission package | support package and handoff | mature package with real refs |
| QMT/runtime/simulation/live | explicitly blocked | still blocked until separate runtime gate |

### 5.4 集成契约

| 调用方 | 调用时机 | 输入 | 输出 | 后续衔接 |
|---|---|---|---|---|
| CP6 implementation Story | CP5 批准后 | CP5 LLD 中的 builder/evidence/handoff task refs | source/test changes | CP6 result/evidence index |
| CP7 verification | CP6 PASS 后 | Story return、evidence index、targeted pytest | verification report/result | CP8 release readiness |
| Stage 3 research machine | CR157 release 后且另有 Stage 3 授权 | `Stage3ResearchMachineHandoff` | real refs and Stage3 mature package | future Stage 4 review |
| Runtime/simulation gate | future explicit authorization | Stage3 package + runner preflight + user per-run approval | runtime-specific evidence | not part of CR157 |

## 6. 前置校验与失败路径

| 阶段 | 前置条件 | 失败行为 |
|---|---|---|
| CP4 Story planning | CP3 approved；HLD/ADR 决策已接受。 | 停止 Story 拆解，回 CP3 修订。 |
| CP5 LLD | CP4 DAG/check PASS；Story owner/file boundary 清楚。 | 阻断 CP5，拆分 Story 或改 LLD policy。 |
| CP6 implementation | CP5 approved；禁止操作清单仍为 0。 | 若需要真实数据/runtime，停止并发起新 CR/authorization gate。 |
| CP7 verification | CP6 result/evidence index PASS；targeted tests 可运行。 | FAIL -> 回 CP6；NEEDS_DESIGN_CLARIFICATION -> 回 CP5/CP3。 |
| Stage 3 consumption | CR157 released；Stage 3 另行授权；真实 refs 可用。 | 缺任一 required input/evidence -> Stage 3 package blocked。 |

## 7. 场景模拟

| 场景 | 输入状态 | 期望结果 |
|---|---|---|
| S1 Stage 2 fixture bundle | FactorSpec valid，fixture signal/risk/evidence refs，permission counters 全 0。 | bundle `stage2_to_stage3_handoff_ready`，typed unavailable 指向 Stage 3。 |
| S2 evidence refs 缺真实 data release | Stage 2 evidence index 无 `data_release_ref`。 | `typed_unavailable.stage3_real_data_lineage_required`，不访问 lake。 |
| S3 forbidden operation counter 非 0 | `provider_fetch=1` 或 `lake_write=1`。 | Stage 2 validation `blocked`，blocked reason field 指向 counter。 |
| S4 Stage 3 package 缺 factor panel | Stage 3 evidence_refs 少 `factor_panel_ref`。 | package status `blocked`，不能进入 Stage 4 review。 |
| S5 event adapter 请求 | strategy_family=event 或 event feed refs。 | CR157 不处理；路由到 DF-CR157-001/backlog。 |

## 8. 安全与授权边界

| 项 | CP3/CR157 授权 |
|---|---|
| source implementation | not authorized before CP5 approval |
| tests implementation | not authorized before CP5 approval |
| real lake read | not authorized |
| real lake write | not authorized |
| provider fetch | not authorized |
| NAS operation | not authorized |
| credential / `.env` read | not authorized |
| QMT / gateway / xtquant | not authorized |
| simulation / paper / live / trading | not authorized |
| broker operation | not authorized |
| external framework clone/install/run | not authorized |
| Git remote write | not authorized |
| catalog/store/registry publish | not authorized |

## 9. Story 候选切片

CP3 只确认架构，不创建正式 Story 真相源。若 CP3 批准，CP4 建议按下列候选拆分：

| 候选 Story | Feature owner | 目标 | 依赖 |
|---|---|---|---|
| CR157-S01 | FEAT-03 | Mature admission package builder contract hardening。 | CP3 approved |
| CR157-S02 | FEAT-13 | ResearchEvidenceIndex refs-only traceability and fail-closed validation。 | S01 |
| CR157-S03 | FEAT-14 | Stage2/Stage3 handoff and package validation hardening。 | S01/S02 |
| CR157-S04 | FEAT-14 | No-runtime guard tests and forbidden-operation scanner coverage。 | S01/S02/S03 |
| CR157-S05 | FEAT-13/14 | Documentation, release wording and deferred adapter backlog alignment。 | S01-S04 |

## 10. 风险

| 风险 ID | 风险 | 等级 | 缓解 |
|---|---|---|---|
| R-CR157-001 | Mature admission support 被误读为 production admission。 | high | blocked claims、not_runtime flags、release wording、CP8 risk item。 |
| R-CR157-002 | Evidence index refs 漂移或复制正文导致双真相源。 | medium | refs-only contract；不复制 metric/report body。 |
| R-CR157-003 | Stage 3 handoff 兼容性破坏历史引用。 | medium | 保持 schema/version 名称，新增字段需向后兼容。 |
| R-CR157-004 | event/ML adapter 被提前耦合。 | medium | DF-CR157-001/002 deferred；CP4 不拆对应 Story。 |
| R-CR157-005 | No-runtime guard 只停留在文档。 | high | CP5/CP6 必须加入 counter tests 和 forbidden import scan。 |

## 11. 决策表

| 用户意图关键词 | 路由 | 理由 |
|---|---|---|
| approve CP3 / 批准 CP3 | CP4 Story planning | HLD/ADR 架构确认，仍不授权实现。 |
| 修改 builder scope | CP3 revision | 影响核心 architecture decision。 |
| 加 event adapter | scope change / new CR or CR157 revision | 改变 FEAT-13 adapter 范围。 |
| 加 ML adapter | scope change / new CR or CR157 revision | 涉及 training/model evidence，超出 first slice。 |
| 授权真实数据或 runtime | runtime_authorization gate / new CR | CP3 不具备真实操作授权。 |
| reject CP3 | keep CR157 at solution-design | 不进入 Story/LLD/implementation。 |

## 12. 理论依据

本 HLD 使用 contract-first design、fail-closed safety gate、evidence traceability 和 staged authorization 方法。枚举型字段不是穷尽集合，而是基于当前项目 CR150/151/154/155 合同、Meta Flow CP gate 协议和领域经验形成的可扩展 first slice。

## 13. 开放问题状态

| 问题 ID | 问题 | 状态 | 决策引用 |
|---|---|---|---|
| SGQ-CR157-001 | first slice 是否聚焦 mature admission package + evidence + handoff + no-runtime guard。 | RESOLVED 2026-07-05 | DQ-CP2-CR157-FIRST-SLICE |
| AGQ-CR157-001 | 是否只做 docs-only。 | RESOLVED 2026-07-05 | CP3-DQ-CR157-ARCHITECTURE |
| AGQ-CR157-002 | event/ML adapter 是否进入本 CR。 | RESOLVED 2026-07-05 | CP3-DQ-CR157-DEFERRED-ADAPTERS |

## 14. Gotchas

- 不要把 `stage2_framework_ready` 解读为 `qmt_ready`、`simulation_ready` 或 `live_ready`。
- 不要在 evidence index 中复制 report 正文；应引用 stable refs。
- 不要为了补齐 evidence 在 Stage 2 读取真实 lake/provider/NAS。
- 不要把 event/ML adapter 塞进 CR157 Story；它们需要各自的 schema 和验证策略。
- 不要在 CP3 approve 后直接实现；CP4/CP5 仍是必经门禁。
