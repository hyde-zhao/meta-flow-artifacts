---
discussion_id: "CP2-CR052-VNEXT-R2"
change_id: "CR-052"
checkpoint: "CP2"
stage: "requirement-clarification"
status: "completed-for-scenario-baseline"
owner: "meta-pm"
created_at: "2026-07-19T09:04:51Z"
canonical_use_cases: "process/docs/product/USE-CASES.md"
checkpoint_ref: "process/checks/CP2-CR052-VNEXT-R2-DISCUSSION-CHECKPOINT.json"
context_ref: "process/context/CP2-CR052-VNEXT-REFRAME-CONTEXT.yaml"
---

# CP2 CR-052 vNext R2 Scenario Discussion Log

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-19 | meta-pm | 记录 vNext R2 的四个 Scenario Gray Areas、SGQ-VNEXT-001 用户可见确认、五个待人工决策项和 deferred ideas；不复用旧 migration-readiness CP2 结论。 |

## 讨论目标

确认 CR-052 的真实场景主体是否已经从“共享 artifact worktree migration-readiness”变为“每项目双库隔离 + Project/Roadmap/Phase/Work + G0/G1/G2 + Work-scoped read/write/check/token + 当前快照迁移”，并把仍需 CP2 正式批准的产品默认从场景理解中分离出来。

## Canonical References

- `process/REQUEST.md`：CR-052 vNext 重解释请求，明确双库、四层治理、风险流程和 token/scope 控制。
- `process/changes/summaries/CR-052.vnext-reframe.summary.json`：当前唯一有效的简化整改摘要、推荐默认和不授权边界。
- `process/context/CP2-CR052-VNEXT-REFRAME-CONTEXT.yaml`：CP2 capsule-first 上下文和单写约束。
- `process/docs/product/USE-CASES.md`：UC-VNEXT-001..006 当前候选场景基线。

## Scenario Gray Areas

| Gray Area ID | 问题 | Why it matters | Impact Surface | 推荐讨论顺序 | Canonical Refs | 处理状态 |
|---|---|---|---|---:|---|---|
| SGA-VNEXT-01 | 发布库复用现有源码/交付仓，还是另建专用发布仓？ | 决定每项目是严格双库还是再次引入第三仓，以及迁移/同步成本。 | scope / complexity / delivery / gate | 1 | REQUEST vNext #1；summary recommended_defaults.release_repo | scenario-direction-confirmed；DQ-VNEXT-01 pending |
| SGA-VNEXT-02 | 过程库采用 main-only 单写 publisher + expected-OID CAS，还是 per-Work branch / 多写者 merge？ | 决定能否真正消除分支/working-tree联动和并发覆盖。 | architecture / reliability / maintenance / validation | 2 | REQUEST vNext #2/#3；summary recommended_defaults.process_repo/process_writer | scenario-direction-confirmed；DQ-VNEXT-02 pending |
| SGA-VNEXT-03 | G0/G1 默认 read/write/check/token 上限取多少，超限如何升级？ | 决定轻量流程是否真的减负，也决定复杂 Work 是否 underfit。 | scope / cost / validation / gate | 3 | REQUEST vNext #4；summary risk_profiles | scenario-direction-confirmed；DQ-VNEXT-04 pending |
| SGA-VNEXT-04 | 首版迁历史还是只迁当前快照，旧共享仓保留多久只读？ | 决定迁移风险、审计语义、回滚窗口和兼容成本。 | migration / compatibility / rollback / maintenance | 4 | REQUEST vNext #5；summary recommended_defaults.migration/legacy_conversion/pilot | scenario-direction-confirmed；DQ-VNEXT-03/05 pending |

## 用户可见场景确认交互

| 字段 | 内容 |
|---|---|
| Question ID | `SGQ-VNEXT-001` |
| 问题 | CR-052 是否应放弃“共享 artifact worktree migration-readiness”为当前目标，改为每项目双库隔离、四层长期治理、G0/G1/G2 与 Work-scoped 资源控制，并以快照方式迁移？ |
| 选项 | A. 按 vNext 推荐方向重构产品基线；B. 保持旧 MR 主线；C. 只写概念文档、暂不进入正式流程。 |
| 推荐 | A。它直接消除跨项目联动，优先简单、可靠、低治理成本；历史转换和批量迁移后置。 |
| 用户回答 | 用户在 vNext 重解释请求中明确接受推荐方向并要求开始正式流程。 |
| 复述确认 | 当前确认只证明 vNext 应成为新的 CP2 候选场景基线；发布库复用、main-only publisher/CAS、G0/G1 数值、快照兼容期与试点边界仍由 DQ-VNEXT-01..05 正式决策。它不等于 CP2 全部批准，也不授权源码、真实迁移、commit/push、publication、runtime 或 production。 |
| 影响面 | scope / architecture / validation / migration / gate / authorization |
| 来源 | `process/REQUEST.md`“CR-052 vNext 重解释请求”；`process/changes/summaries/CR-052.vnext-reframe.summary.json` |
| 状态 | `confirmed-for-scenario-baseline` |

## 待人工决策分类

| 决策 ID | 类型 | 推荐方案摘要 | 分类 | 正式来源 |
|---|---|---|---|---|
| DQ-VNEXT-01 | architecture | 复用现有发布仓，每项目只新增一个独立过程仓 | decision-item | REQUIREMENTS.md vNext DQ 表 |
| DQ-VNEXT-02 | architecture | 过程库 main-only 单写 publisher + expected-OID CAS | decision-item | REQUIREMENTS.md vNext DQ 表 |
| DQ-VNEXT-03 | scope | 只迁当前快照，旧共享仓永久只读索引，不拆史/不无损转换 | decision-item | REQUIREMENTS.md vNext DQ 表 |
| DQ-VNEXT-04 | implementation | G0=8/8/3/32k，G1=20/24/8/96k，高风险或超限进入G2 | decision-item | REQUIREMENTS.md vNext DQ 表 |
| DQ-VNEXT-05 | scope | 2项目×每项目2周期、每项目覆盖G0/G1，旧仓观察期推荐30天 | decision-item | REQUIREMENTS.md vNext DQ 表 |

> 分类结果：阻断性未分类灰区为 0；五项都已转为 `decision-item`，但尚未获得 CP2 人工批准。Host Orchestrator 必须在正式 Decision Brief 中提供推荐、备选、优劣、风险与回退。

## Deferred Ideas

| ID | 延后项 | 原因 | 重启条件 |
|---|---|---|---|
| DEF-VNEXT-001 | 旧共享过程仓历史按项目拆分 | 首版快照可实现核心隔离，历史重写风险高 | 只读索引无法满足强制审计/恢复且新CR批准历史策略 |
| DEF-VNEXT-002 | 旧 CP/CR/Story 无损转换器 | 异构语义可能产生虚假等价 | 至少两个项目证明必须转换且有版本化 schema/差异/回滚 |
| DEF-VNEXT-003 | 多 publisher 分布式自动 merge | 单写 publisher + CAS 更简单可靠 | 连续两个周期违反已批准SLO且CAS重试为主要瓶颈 |
| DEF-VNEXT-004 | 批量迁移项目 | 试点前会扩大爆炸半径 | 2×2试点全通过且用户批准项目清单/顺序/逐项目回滚 |
| DEF-VNEXT-005 | 远端 publication / production cutover 自动化 | 当前未授权外部写入或运行 | 指定目标/OID/环境、最小权限和回滚获独立批准 |

## 场景充分性结论

- 当前 vNext 用户场景：6 个（UC-VNEXT-001..006）。
- Scenario Gray Areas：4 个；4 个均已形成场景方向并路由到 5 个正式决策项。
- 用户可见交互：1 条（SGQ-VNEXT-001），状态为 `confirmed-for-scenario-baseline`。
- 工程场景：20 个（TC-VNEXT-001..020），覆盖 positive / negative / boundary / permission / failure-recovery 五类。
- P0 场景阻断缺口：0。
- CP2 正式待人工决策：5；CP2 人工结论：尚未发起/尚未批准。

## 不授权边界

本讨论和 SGQ 确认不授权正式 Feature/Story/LLD、源码实现、真实仓创建或迁移、旧仓写入、commit/push、repository publication、credentials、runtime、production、publish、live 或 trading。
