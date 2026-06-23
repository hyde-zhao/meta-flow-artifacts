---
title: "CR107 Redesign Scope Intake"
cr_id: "CR-107"
status: "draft-for-cp2-review"
created_at: "2026-06-22T13:38:58+08:00"
owner: "host-orchestrator"
source_cr: "CR-107"
scope_policy: "no-code-no-runtime"
---

# CR107 Redesign Scope Intake

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| v0.1 | 2026-06-22 | host-orchestrator | 初稿：承接 CR106 READY，定义 redesign 范围、非范围和候选项排序。 |

## 1. 当前基线

| 维度 | 当前结论 | 证据 |
|---|---|---|
| 分支状态 | `precheck/ql-rd-000-redesign-baseline` 已完成 baseline 清洁检查 | `process/checks/PRECHECK-QL-RD-000-REDESIGN-BASELINE-2026-06-22.md` |
| 代码健康 | 当前证据不要求源码整改 | CR106 CP7 / CP8 结论 |
| 治理事实源 | module boundary 与 package identity 已补齐 | `docs/design/MODULE-BOUNDARIES.yaml`、`docs/design/PACKAGE-IDENTITY.yaml` |
| CR 状态 | 启动 CR107 前 active formal CR 为 none | `meta-flow check cr-tracking --project-root .` |
| 高风险边界 | CR105 / runtime / 交易写 / NAS / 凭据 / publish 未授权 | CR106 和 CR107 authorization policy |

## 2. Redesign 目标

本轮 redesign 的目标限定为“先定范围，再排队执行”，不把范围讨论混入代码实现。

| 目标 ID | 目标 | 可验证标准 | 当前 CR107 处理 |
|---|---|---|---|
| RD-GOAL-01 | 明确后续 redesign 要解决的问题类别 | 至少形成模块边界、候选 CR、非范围和下一步建议 | In Scope |
| RD-GOAL-02 | 把高风险 runtime 和交易写从默认路线隔离出去 | CR105 / runtime / NAS / credentials / publish 均列为不授权项 | In Scope |
| RD-GOAL-03 | 对现有候选项排序 | CR-026、CR-027、CR-028、RA-CR097-001、FU-CR101-001、CR089 均有推荐路由 | In Scope |
| RD-GOAL-04 | 避免无依据源码整改 | 没有具体失败证据时不创建代码修复 Story | In Scope |
| RD-GOAL-05 | 为下一条低风险 CR 提供入口 | CP8 输出一个明确推荐候选 | In Scope |

## 3. In Scope

| 范围项 | 说明 | 输出 |
|---|---|---|
| Scope intake | 说明 redesign 的目标、边界、输入证据和非目标 | 本文件 |
| Candidate prioritization | 对现有候选 CR / Spike / blocked CR 做排序和路由 | §6 候选项排序 |
| Non-authorization boundary | 明确 CR107 不授权事项 | §4 非范围 |
| Next CR recommendation | 给出 CR107 之后最稳妥的下一 CR 类型 | §7 下一步建议 |
| Governance state sync | 同步 CR-INDEX、STATE、CP2/CP8 检查证据 | `process/changes` / `process/checks` / `process/checkpoints` |

## 4. Out of Scope / Not Authorized

| 项目 | 状态 | 说明 |
|---|---|---|
| 源码实现 / 重构 | Not Authorized | CR107 不修改 Python 代码、测试代码、包结构或运行时逻辑。 |
| CR105 / order-write | Not Authorized | 不做 submit / cancel / buy / sell，不做 simulation / live。 |
| QMT / MiniQMT / XtQuant / gateway runtime | Not Authorized | 不连接、不登录、不查询、不读取账户、仓位、订单、成交或 raw log。 |
| NAS | Not Authorized | 不 mount、不 list、不 read、不 write、不 publish、不 delete。 |
| Credentials / secrets | Not Authorized | 不读取 `.env`、token、账号、密码、cookie、session。 |
| Provider / lake / catalog | Not Authorized | 不抓取 provider、不写 lake、不 publish catalog。 |
| CR089 恢复 | Not Authorized | CR089 保持 blocked，不由 CR107 恢复。 |

## 5. Redesign 分类框架

CR107 使用四类候选池，避免把不同风险等级混在一个执行队列里。

| 类别 | 定义 | 默认处理 |
|---|---|---|
| A. Governance / baseline | 只影响台账、文档、边界说明、checker evidence | 可优先启动 |
| B. Architecture-only | 只做 HLD / ADR / Feature boundary，不做实现 | 需要 CP3/HLD gate |
| C. Research / data Spike | 探索性数据或研究路线，不承诺交付 | 仅在明确触发条件满足后启动 |
| D. Runtime / trading high-risk | 真实 runtime、账户、交易写、NAS、publish | 默认不启动，必须独立高风险门禁 |

## 6. 候选项排序

| 排名 | 候选项 | 类别 | 推荐动作 | 理由 | 阻塞 / 前置 |
|---:|---|---|---|---|---|
| 1 | `FU-CR107-001 Redesign Backlog Decision Gate` | A | 推荐作为 CR107 后的默认下一步 | 低风险，只把 redesign backlog 固化为可执行队列，不改代码 | 需要 CR107 CP8 确认 |
| 2 | `FU-CR107-002 Architecture-only Redesign HLD Gate` | B | 仅当用户要进入架构设计时启动 | 可把模块边界、状态机、ledger / context / delivery routing 做 HLD，不涉及实现 | 需要先确认目标模块和成功标准 |
| 3 | `CR-026 Qlib isolated runner optional Spike` | C | 延后 / 可人工取消 | 自有多因子主框架已由 CR030-039 覆盖；只有仍需要 Qlib 对照 runner 时才有价值 | 依赖隔离、provider boundary、运行授权需单独门禁 |
| 4 | `CR-027 Minute data feasibility Spike` | C | 延后 | 当前没有日频假设失效证据；minute source/schema/storage/quality 未定义 | 数据来源、schema、质量计划明确后再启动 |
| 5 | `CR-028 Level2 rights and microstructure Spike` | C/D | 延后，高风险预备 | 涉及 Level2 权利、盘口、queue、impact-cost、replay fixture，成本和权限风险高 | 需要数据权利、质量审计和成本评估 |
| 6 | `RA-CR097-001 Non-empty / trading-day readonly retest` | D | 暂不启动 | 真实 runtime / readonly retest 属于运行授权，不是 redesign 默认下一步 | 需要独立 per-run runtime authorization |
| 7 | `CR-089 QMT Interface Validation Gate` | D | 保持 blocked | 与 QMT/NAS/interface runtime 强相关；当前不恢复 | 需要用户明确恢复并重做 runtime authorization |
| 8 | `FU-CR101-001 / CR105 Order-write / simulation / live design authorization gate` | D | 明确不启动 | 交易写和 simulation/live 风险最高，不属于 baseline redesign 默认路线 | 只有用户明确要求交易写验证时启动 |

## 7. 下一步建议

默认推荐在 CR107 CP8 后启动：

| 推荐 | 候选 | 目标 | 为什么 |
|---|---|---|---|
| 首选 | `FU-CR107-001 Redesign Backlog Decision Gate` | 把 redesign backlog 固化为 2-4 个低风险后续 CR | 延续当前 no-code/no-runtime 基线，先把工作队列变清楚 |
| 备选 | `FU-CR107-002 Architecture-only Redesign HLD Gate` | 进入模块边界 / 状态机 / context / ledger 的 HLD | 适合用户决定要做结构性设计时 |
| 不推荐 | `CR105 / FU-CR101-001` | 交易写 / simulation / live | 高风险，必须独立授权，不作为默认路线 |
| 不推荐 | `CR089` | QMT/NAS/interface validation | blocked 状态，恢复成本和授权风险高 |

## 8. CP2 待确认决策

| 决策 ID | 类型 | 推荐方案 | 备选方案 | 回退条件 |
|---|---|---|---|---|
| DQ-CP2-CR107-01 | scope | 接受 CR107 只做 no-code/no-runtime scope intake 和候选排序 | A. 暂停 redesign；B. 直接进入 architecture-only HLD | 若用户要求代码实现，另起实现 CR |
| DQ-CP2-CR107-02 | follow_up_tracking | 接受候选排序，并把 FU-CR107-001 作为默认下一步 | A. 优先 architecture-only HLD；B. 优先 CR-026 Spike | 若候选排序不符合用户目标，重排并重新发起 CP2 |
| DQ-CP2-CR107-03 | runtime_authorization | 继续禁止 CR105、CR089 恢复、runtime、交易写、NAS、凭据、publish | A. 另起 CR105；B. 另起 runtime/NAS gate | 若用户明确要求高风险验证，停止 CR107 默认路线 |

## 9. 当前结论

CR107 的推荐路线是：先批准 CP2 scope intake，再把候选排序固化为 CP8 关闭结论。当前不需要代码整改，不应默认进入 CR105、CR089、runtime retest 或数据高风险 Spike。
