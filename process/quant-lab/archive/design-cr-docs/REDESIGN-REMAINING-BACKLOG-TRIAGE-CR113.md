---
title: "CR113 Redesign Remaining Backlog Triage"
cr_id: "CR-113"
status: "approved-closed-ready"
created_at: "2026-06-22T16:13:35+08:00"
owner: "host-orchestrator"
source_cr: "CR-113"
scope_policy: "no-code-no-runtime"
---

# CR113 Redesign Remaining Backlog Triage

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| v0.1 | 2026-06-22 | host-orchestrator | 初稿：承接 CR112 READY，复盘剩余 redesign backlog 候选并给出轻量排序。 |
| v0.2 | 2026-06-22 | host-orchestrator | 用户 approve CR113 CP2，确认 no-code/no-runtime triage 范围、排序原则和高风险不授权边界。 |
| v1.0 | 2026-06-22 | host-orchestrator | 用户 approve CR113 CP8，关闭为 READY；后续候选仅登记，不自动启动。 |

## 1. 当前基线

| 维度 | 当前结论 | 证据 |
|---|---|---|
| 最近关闭 CR | `CR112 closed-current-delivery / READY` | `process/checkpoints/CP8-CR112-DELIVERY-READINESS.md` |
| 当前 active formal CR | 启动 CR113 前为 none；CR113 启动后为 active / CP2 pending | `process/changes/CR-INDEX.yaml` |
| 代码 / tests | 本轮不修改 | 用户明确要求不改代码；CR113 authorization policy |
| runtime | 本轮不启动 | 用户明确要求不启动 runtime；CR113 authorization policy |
| 高风险边界 | CR105 / CR089 / runtime / NAS / credentials / publish 均未授权 | CR107-CR112 关闭边界 |

## 2. Triage 目标

| 目标 ID | 目标 | 可验证标准 | 当前 CR113 处理 |
|---|---|---|---|
| TRIAGE-GOAL-01 | 复盘 CR107-CR112 之后仍未关闭的 redesign 候选 | 候选来源、状态、阻塞和下一步清楚列出 | In Scope |
| TRIAGE-GOAL-02 | 给出下一步可启动候选排序 | 至少输出推荐启动、保留、延后、不启动四类 | In Scope |
| TRIAGE-GOAL-03 | 防止 triage 被误解为实现授权 | 源码、tests、checker implementation、runtime 均列为 Not Authorized | In Scope |
| TRIAGE-GOAL-04 | 把高风险 runtime / trading / NAS 路线隔离 | CR105、CR089、RA-CR097-001 等不进入默认低风险队列 | In Scope |

## 3. 分类原则

| 类别 | 定义 | 默认处理 |
|---|---|---|
| A. Low-risk governance / decision | 只影响文档、台账、门禁体验或候选排序 | 可作为下一步候选 |
| B. Architecture-only | 只做 HLD / ADR / feature boundary，不做实现 | 可启动，但需要独立 CP3/HLD gate |
| C. Implementation planning / source change | 需要源码、tests、checker implementation 或 Story 实现计划 | 保留候选，必须另起实现 CR |
| D. Research / data Spike | 需要数据源、依赖隔离、provider 或实验计划 | 延后，只有触发条件明确时启动 |
| E. Runtime / trading / NAS high-risk | 涉及真实 runtime、账户、交易写、NAS、publish 或凭据 | 默认不启动，必须独立高风险授权 gate |

## 4. 候选复盘与排序

| 排名 | 候选项 | 类别 | 当前状态 | 推荐动作 | 理由 | 阻塞 / 前置 |
|---:|---|---|---|---|---|---|
| 1 | `FU-CR113-001 Context Capsule / Human Gate Consistency Review`（legacy: `FU-CR109-002`） | A | candidate | 推荐作为下一条低风险治理候选 | 继续 no-code/no-runtime，能减少 CP2/CP3/CP5/CP8 发起消息、capsule 和 pending decision queue 的一致性风险 | CR113 已确认；启动仍需用户明确选择并另起正式 CR |
| 2 | `FU-CR113-002 Architecture-only Redesign HLD Gate`（legacy: `FU-CR108-001` / `FU-CR107-002`） | B | candidate | 推荐保留为第二优先级 | 治理清理后可进入架构视角，梳理模块边界、状态机、ledger/context/delivery routing；不涉及实现 | 需用户选择 architecture-first，并另起 CP3/HLD gate |
| 3 | `FU-CR113-003 CP Result JSON / Ledger Gap Analysis`（legacy: `FU-CR109-003`） | A | candidate | 保留为治理候选 | `STATE.current.json` 缺失、ledger 机器真相源不完整是可见缺口；但需要先决定 context/human gate 是否也要收敛 | 需明确只做 gap analysis，不补实现 |
| 4 | `FU-CR108-003 Redesign Implementation Planning Gate` | C | candidate | 延后 | 实现规划应等待 HLD 或治理 gap 收敛，否则容易把排序讨论扩大成实现授权 | 需 HLD 或明确实现边界后启动 |
| 5 | `FU-CR112-001 Checker Implementation Alignment Candidate` | C | candidate | 保留但不启动 | CR112 只写 expectation notes；实现对齐会修改源码 / tests / checker implementation，风险等级更高 | 需要独立 implementation CR 和源码授权 |
| 6 | `CR-026 Qlib isolated runner optional Spike` | D | candidate | 延后 / 可后续人工取消 | CR030-039 已覆盖自有多因子主框架；Qlib runner 只在仍需要对照运行时有价值 | 需依赖隔离、provider boundary 和运行授权 |
| 7 | `CR-027 Minute data feasibility Spike` | D | spike_candidate | 延后 | 当前没有日频假设失效证据，也没有 minute source/schema/storage/quality plan | 需明确触发证据和数据方案 |
| 8 | `CR-028 Level2 rights and microstructure Spike` | D/E | spike_candidate | 延后，高风险预备 | 涉及 Level2 权利、盘口、queue、impact-cost、replay fixture，成本和权限风险高 | 需数据权利、schema、质量审计和成本评估 |
| 9 | `CR-089 QMT Interface Validation Gate` | E | blocked | 不启动 | 与 NAS/QMT/interface runtime 强相关，仍 blocked-readiness-approved | 需用户明确恢复并重做 runtime/NAS authorization |
| 10 | `FU-CR101-001 / CR105 Order-write / simulation / live design authorization gate` | E | not_started | 不启动 | 交易写和 simulation/live 风险最高，不属于 redesign remaining backlog 默认路线 | 只有用户明确要求交易写验证时启动 |
| 11 | `RA-CR097-001 Non-empty / trading-day readonly retest` | E | closed/deferred by CR104 scope | 不启动 | readonly runtime 路线已经被 CR104 限定关闭；额外 runtime 需 per-run authorization | 需独立 runtime authorization gate |

## 5. 推荐队列

| 序号 | 推荐候选 | 建议 | 预期产出 |
|---:|---|---|---|
| 1 | `FU-CR113-001 Context Capsule / Human Gate Consistency Review` | CR113 CP8 后首选 | 一份 no-code/no-runtime consistency review，聚焦 capsule、Decision Brief、pending decision queue 和 launch message 的一致性。 |
| 2 | `FU-CR113-002 Architecture-only Redesign HLD Gate` | 用户想进入结构设计时启动 | HLD / ADR / feature boundary，不改实现。 |
| 3 | `FU-CR113-003 CP Result JSON / Ledger Gap Analysis` | 治理缺口分析候选 | 只盘点 result JSON / ledger 缺口，是否补实现另起 CR。 |
| 4 | `FU-CR112-001 Checker Implementation Alignment Candidate` | 仅登记，不启动 | 需要代码 / tests / checker implementation 授权，不能被 CR113 approve 隐式启动。 |

## 6. 不启动 / Deferred 边界

| 项目 | 状态 | 原因 | 可重启条件 |
|---|---|---|---|
| CR105 / order-write / simulation / live | Not Authorized | 交易写和真实运行风险高 | 用户明确要求并通过独立高风险门禁 |
| CR089 恢复 | Not Authorized | 当前 blocked，涉及 QMT/NAS/interface runtime | 用户明确恢复并通过 runtime/NAS authorization |
| NAS compare / provider rebuild | Not Authorized | 涉及 NAS 或 provider/lake/catalog | 用户明确发起独立数据 / NAS / provider gate |
| Checker implementation alignment | Deferred | 需要源码 / tests / checker implementation change | 用户明确启动独立 implementation CR |
| Historical STATE long-table bulk rewrite | Deferred | 易造成审计链混乱 | 用户明确启动审计清理 CR 并接受追溯风险 |

## 7. CP2 待确认决策

| 决策 ID | 类型 | 推荐方案 | 备选方案 | 回退条件 |
|---|---|---|---|---|
| DQ-CP2-CR113-01 | scope | 接受 CR113 只做 no-code/no-runtime remaining backlog triage | A. 暂停 triage；B. 直接选择单个候选启动；C. 改为实现计划 CR | 若用户要求源码或 runtime，另起独立 CR |
| DQ-CP2-CR113-02 | follow_up_tracking | 接受推荐排序：context/human gate consistency 优先，architecture HLD 第二，CP result / ledger gap 第三，implementation/high-risk 延后 | A. architecture-only HLD 优先；B. CP result / ledger gap 优先；C. checker implementation alignment 优先 | 若用户调整排序，更新本文件并重发 CP2 |
| DQ-CP2-CR113-03 | runtime_authorization | 继续禁止源码修改、tests 修改、checker implementation、CR105、CR089、runtime、NAS、凭据、交易写和 publish 被 CR113 隐式启动 | A. 另起实现 CR；B. 另起 checker implementation CR；C. 另起 runtime/NAS/trading gate | 若用户明确授权高风险工作，停止 CR113 默认路线 |

## 8. 当前结论

CR113 的推荐结论是：本轮只确认剩余 backlog 排序，不启动任何下游候选。CP2 和 CP8 均已确认，默认下一条可启动候选是 `FU-CR113-001 Context Capsule / Human Gate Consistency Review`；`FU-CR113-002 Architecture-only Redesign HLD Gate` 作为结构设计备选；所有源码、checker implementation、runtime、NAS、交易写和 publish 路线保持未授权。

## 9. CP2 审批结果

| 字段 | 内容 |
|---|---|
| 审批结论 | `approved` |
| 审批人 | user |
| 审批时间 | 2026-06-22T16:21:59+08:00 |
| 接受决策 | `DQ-CP2-CR113-01`、`DQ-CP2-CR113-02`、`DQ-CP2-CR113-03` |
| 当前门禁 | closed |
| 不授权项 | 源码修改、tests 修改、checker implementation、CR105、CR089、runtime、NAS、凭据、交易写、publish |

## 10. CP8 审批结果

| 字段 | 内容 |
|---|---|
| 审批结论 | `approved` |
| 审批人 | user |
| 审批时间 | 2026-06-22T16:39:47+08:00 |
| 接受决策 | `DQ-CP8-CR113-01`、`DQ-CP8-CR113-02` |
| 关闭状态 | `closed-current-delivery / READY` |
| 默认下一候选 | `FU-CR113-001 Context Capsule / Human Gate Consistency Review`，仅登记为 candidate、不启动 |
| 不授权项 | 源码修改、tests 修改、checker implementation、CR105、CR089、runtime、NAS、凭据、交易写、publish |
