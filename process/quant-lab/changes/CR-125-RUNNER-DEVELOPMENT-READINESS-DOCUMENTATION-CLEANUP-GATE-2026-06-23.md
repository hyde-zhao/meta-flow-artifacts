---
cr_id: "CR-125"
title: "Runner Development Readiness Documentation Cleanup Gate"
cr_type: "process"
cr_kind: "requirement-change"
lifecycle_status: "closed"
readiness_status: "ready_with_risk"
gate_status: "closed"
gate_profile: "standard"
status: "closed-current-delivery"
impact_level: "medium"
workflow_mode_before: "production"
workflow_mode_after_change: "standard"
rollback_to: "pre-CR125 process state; no docs/source/runtime changes are authorized before CP5 approve"
source: "user-request"
parent_cr: "CR-124"
source_checkpoint: "process/checkpoints/CP8-CR124-DELIVERY-READINESS.md"
source_follow_up_id: "FU-CR124-001"
follow_up_type: "runner-development-readiness-documentation-cleanup-gate"
risk_class: "medium-documentation-routing-and-runner-readiness-alignment"
owner: "host-orchestrator"
created_at: "2026-06-23T12:06:50+08:00"
updated_at: "2026-06-23T13:34:51+08:00"
acceptance_criteria: "CP2 confirms CR125 only aligns CR121 remaining directory routing/index cleanup decisions and the five retained CR102/CR103/CR104 feature docs with the next runner development entry. CP5 confirms the actual remediation subset before any deletion, movement, document rewrite, .gitignore/source/checker/tests change, runtime, push or external access."
close_condition: "CP8 approved after the minimal runner development readiness cleanup scope is either executed and verified or explicitly deferred; then stop historical docs cleanup and return to quant-lab runner development."
---

# CR-125 Runner Development Readiness Documentation Cleanup Gate

## 变更来源

用户批准 CR124 CP8 后要求：关闭 CR124 为 READY_WITH_RISK，并启动一个小范围 runner development readiness / cleanup CR，把 `FU-CR124-001` 中的 CR121、CR102/103/104 文档治理与后续 runner 开发入口对齐；完成后停止继续清理历史文档，转入 quant-lab 后续开发。

## 当前目标

本 CR 的目标不是继续做大规模历史文档清理，而是建立进入 runner development 前的最小文档治理闸门：

1. 对齐 CR121 的剩余目录 routing / index cleanup 结论，避免分支合并后再次产生 docs 目录归属混乱。
2. 对 CR102/CR103/CR104 三个保留 feature 目录做重点复盘，只判断哪些文件必须保留、迁移、压缩或可删除。
3. 明确清理停止线：完成本 CR 后，不再继续横向清理历史文档，除非后续 runner 开发直接需要。
4. 保护开发入口：不把 artifact / process 治理与 runner 代码实现、runtime 验证、NAS/QMT/凭据访问混在同一个门禁里。

## 当前只读发现

当前 `docs/features` 中 CR102/103/104 保留文件共 5 个：

| 目录 | 文件 | 初步归类 |
|---|---|---|
| `docs/features/cr102-nas-real-package-exchange-authorization` | `DESIGN.md` | 可能为授权设计上下文；需判断是否迁移/压缩到 runner dev 文档。 |
| `docs/features/cr102-nas-real-package-exchange-authorization` | `TASKS.md` | 可能为执行任务残余；需判断是否删除或转任务摘要。 |
| `docs/features/cr102-nas-real-package-exchange-authorization` | `TEST-PLAN.md` | 可能为验证计划；需判断是否保留摘要或迁移。 |
| `docs/features/cr103-qmt-miniqmt-non-trading-day-validation` | `VERIFICATION.md` | 验证证据候选；默认保留，除非已有等价归档。 |
| `docs/features/cr104-qmt-miniqmt-trading-day-validation` | `INPUT-CHECKLIST.md` | 运行前输入检查证据候选；默认保留或迁移为 runner dev checklist。 |

## 当前授权边界

| 范围 | 当前授权 | 说明 |
|---|---:|---|
| process artifacts 读取 / 写入 / 更新 | true | 仅用于 CR125 变更单、context、precheck、checkpoint、CR index 和 STATE。 |
| docs/features CR102/103/104 read-only audit | true | 可列文件和读取必要摘要；默认不改内容。 |
| CR121 状态 / 结论 read-only audit | true | 可读取 CR121 与相关 checkpoint 结论。 |
| docs deletion / movement / rewrite | false | CP5 approve 前不执行。 |
| `.gitignore` / source / checker / tests 修改 | false | 本 CR 启动阶段不修改。 |
| git add / commit / push / remote write | false | 本轮不授权。 |
| runtime / NAS / credentials / trading / provider / lake / catalog / publish | false | 本 CR 完全不授权。 |

## 五维度影响分析

| 维度 | 影响 | 结论 |
|---|---|---|
| 需求层 | true | 将文档治理从历史清理转为 runner development readiness 的最小入口条件。 |
| 场景层 | true | 覆盖 CR121 剩余 routing/index cleanup 与 CR102/103/104 五个保留文件的处理决策。 |
| 计划层 | true | 当前只生成 CP2/CP5 门禁；实际整改必须等 CP5 approve 后按最小子集执行。 |
| 安全 / 权限层 | true | 明确不授权 runtime、NAS、凭据、交易、push、source/checker/tests 修改。 |
| 交付层 | true | 输出 runner 开发前文档 stop line 和可执行 cleanup 子集，避免继续清理历史文档。 |

## 文档处理决策

| 对象 | 当前处理决策 | 旧基线保留方式 | 后续要求 |
|---|---|---|---|
| CR121 routing / index cleanup 结论 | 原文档不变，作为输入复核 | CR121 正式 CR 与 checkpoints 保留 | CP5 冻结是否需要最小 index/routing follow-up；本 CR 不自动执行。 |
| `docs/features/cr102-*` | 待 CP5 决策 | 当前文件保持不变 | 判断设计/任务/测试计划是否迁移为 runner dev 摘要或删除。 |
| `docs/features/cr103-*` | 待 CP5 决策 | 当前文件保持不变 | `VERIFICATION.md` 默认作为证据候选保留，除非已有等价归档。 |
| `docs/features/cr104-*` | 待 CP5 决策 | 当前文件保持不变 | `INPUT-CHECKLIST.md` 默认迁移或保留为 runner dev checklist 候选。 |
| 其他历史 docs/features | 不变 | CR123/CR124 已处理范围保持 | 不再横向清理。 |

## 冲突预检

| 冲突对象 | 状态 | 处理结论 |
|---|---|---|
| CR124 | closed-current-delivery / READY_WITH_RISK | 已关闭；CR125 从 `FU-CR124-001` 启动。 |
| CR121 | blocked-awaiting-user-next-step | 本 CR 只消费其剩余结论并决定是否合并最小 cleanup；不直接执行 CR121 历史方案。 |
| CR102/103/104 runtime / NAS / QMT | closed / deferred scopes | 本 CR 只处理文档入口，不授权任何 runtime 或外部访问。 |
| 当前主仓库 dirty source/tests | present | 不纳入本 CR；不回滚、不暂存、不提交。 |

## 待人工决策项

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 影响 / 风险 |
|---|---|---|---|---|---|
| DQ-CP2-CR125-01 | scope | 是否接受 CR125 为 runner development 前最后一个文档治理门禁？ | 接受，仅覆盖 CR121 剩余结论与 CR102/103/104 五个文件处理决策。 | A. 只做 CR102/103/104；B. 只做 CR121；C. 暂停，直接进入开发。 | 推荐方案防止目录治理遗留继续影响开发入口。 |
| DQ-CP2-CR125-02 | scope | 是否冻结历史 docs cleanup 停止线？ | 冻结：本 CR 后停止继续横向清理历史文档，后续只处理 runner 开发直接需要的文档。 | A. 继续全量 docs 清理；B. 完全停止不处理 CR102/103/104；C. 转 spike。 | 推荐方案最短路径回到开发。 |
| DQ-CP2-CR125-03 | runtime_authorization | 是否继续禁止 push、runtime、NAS、credentials、source/checker/tests、`.gitignore`？ | 继续禁止；当前只允许 process 门禁与 read-only audit。 | A. 另起 push gate；B. 另起 runtime gate；C. 另起 checker/source CR。 | 防止文档治理门禁扩大为实现或外部运行授权。 |
| DQ-CP5-CR125-01 | implementation | 后续 CP6 是否可执行最小文档整改？ | 仅在 CP5 approve 后允许对 CP5 列出的最小 docs/features 子集做删除/迁移/压缩；默认先不动。 | A. 只输出建议不改文件；B. 直接删除 5 个文件；C. 全部保留。 | 推荐方案保留执行能力，但避免默认误删证据。 |
| DQ-CP5-CR125-02 | follow_up_tracking | CR121 如何处理？ | 将 CR121 剩余 cleanup 纳入 CR125 的最小 readiness 判断；若无需执行则标记 superseded/deferred。 | A. 恢复 CR121 单独执行；B. 关闭 CR121 为 superseded；C. 保持 blocked。 | 推荐方案减少门禁数量。 |
| DQ-CP5-CR125-03 | runtime_authorization | CP5 approve 是否授权 runtime、push、source/checker/tests？ | 不授权；只允许后续 CP6 最小文档整改。 | A. 另起 runner implementation CR；B. 另起 runtime gate；C. 另起 push gate。 | 保持最小权限边界。 |

## 当前结论

CR125 已完成 CP2/CP5 人工批准和 CP6/CP7 最小整改验证，当前进入 CP8 关闭审查 pending。

## CP2 / CP5 人工确认

| 门禁 | 结论 | 审查时间 | 接受决策 |
|---|---|---|---|
| CP2 | approved | 2026-06-23T12:43:37+08:00 | `DQ-CP2-CR125-01..03` |
| CP5 | approved | 2026-06-23T12:43:37+08:00 | `DQ-CP5-CR125-01..03` |

## CP6 最小整改结果

| 对象 | 处理 | 理由 |
|---|---|---|
| `docs/features/cr102-nas-real-package-exchange-authorization/DESIGN.md` | deleted | 设计型授权残留；非验证证据；等价授权门禁记录已保留在 process/checkpoints 与 process/checks。 |
| `docs/features/cr102-nas-real-package-exchange-authorization/TASKS.md` | deleted | 任务拆分残留；不应继续作为 runner development 入口。 |
| `docs/features/cr102-nas-real-package-exchange-authorization/TEST-PLAN.md` | deleted | 测试计划残留；真实 NAS/runtime 后续仍需独立 gate，不作为当前入口证据。 |
| `docs/features/cr103-qmt-miniqmt-non-trading-day-validation/VERIFICATION.md` | retained | 有明确验证结论和命令摘要，保留为历史验证证据。 |
| `docs/features/cr104-qmt-miniqmt-trading-day-validation/INPUT-CHECKLIST.md` | retained | 可作为未来 runner/runtime 独立 gate 的输入检查模板；本 CR 不迁移、不改写。 |
| CR121 | superseded-by-CR125 | CR121 index-only remediation 不再恢复；进入开发前只保留本 CR 的最小 readiness 结论。 |

## CP7 验证结论

静态验证结论为 `PASS_WITH_RISK`：最小文档整改已完成，CR103/CR104 入口已保留，CR121 已从 blocked noise 收敛为 superseded-by-CR125。剩余风险是 artifact/docs 变更尚未提交或 push，且未来如需真实 runtime、NAS/QMT、Git index cleanup、`.gitignore` 或 checker/source/tests 变更，仍必须另起独立 CR。

## 下一步

用户已于 2026-06-23T13:34:51+08:00 批准 CP8，CR125 关闭为 `closed-current-delivery / READY_WITH_RISK`。本轮停止继续清理历史文档；下一步转入 quant-lab runner development readiness / runner 开发入口设计。

关闭不授权 push、commit、runtime、NAS/QMT、credentials、provider/lake/catalog、`.gitignore`、source/checker/tests 或 `git rm --cached`。如后续需要发布当前 artifact/docs 变更，必须另起 publication gate。
