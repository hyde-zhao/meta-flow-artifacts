---
handoff_id: "CR173-CP1-META-PM-2026-07-16"
workflow_id: "CR-173"
change_id: "CR-173"
stage: "requirement-clarification"
from_agent: "host-orchestrator"
to_agent: "meta-pm"
canonical_role: "meta-pm"
mode: "subagent"
status: "returned"
created_at: "2026-07-16T11:47:00+08:00"
completed_at: "2026-07-16T12:15:00+08:00"
context_ref: "process/context/CP1-CR173.context.json"
route_plan_ref: "process/checks/CP0-CR173.route-plan.json"
result_ref: "process/checks/CP1-CR173-USE-CASE-COMPLETENESS.result.json"
return_ref: "process/handoffs/CR173-CP1-META-PM-RETURN-SUMMARY.md"
dispatch:
  canonical_role: "meta-pm"
  codex_agent_name: "meta-pm"
  nickname: "pm-zheng"
  reasoning_profile: "default"
  dispatch_trigger: "CR-173 CP0 PASS; product_baseline_refresh_required=true; next required human gate is CP2"
  tool_name: "spawn_agent"
  agent_id: "/root/pm_zheng_cr173"
  thread_id: "/root/pm_zheng_cr173"
  agent_name: "pm_zheng_cr173"
---

# CR-173 CP1 Meta-PM 增量产品基线 Handoff

## 任务目标

针对 CR-173 离线 effective-trial 方法前置，增量完成用户场景发现、需求结构化、场景扩展、测试矩阵和产品规划；生成 CP1 自动结果与 CP2 自动预检输入。产物必须回答“研究负责人为什么不能直接使用 raw trial count、策略研究员如何使用 effective count、失败时为何必须 typed unavailable”，不能把方法治理流程本身当成全部用户价值。

## 必须读取与上下文策略

1. 先读取 `process/context/CP1-CR173.context.json` 及其 `must_read`。
2. 需要更新 8 个产品文档时，逐个执行 `meta-flow context read-log` 后才可全文读取；不得读取 archive、旧 discussions、HLD、Story 或 LLD。
3. 若 CR summary 与正式 CR 存在字段冲突，记录 read expansion 后只展开 `process/changes/CR-173.md`；不得默认读取全部 CR。

## 产品场景必须包含

1. 业务动机：raw trial count 会把相关试验当作相互独立，不能可靠表达 multiple-testing/data-snooping 暴露；研究负责人需要一个相关性调整后的 effective count 方法基础。
2. 用户痛点：当前 `effective_trial_count=typed_unavailable` 是诚实但不可计算；直接复制 raw count 又会制造虚假精度。
3. 使用价值：未来 consumer 可用可审计 effective count 形成 C1 方法证据，支持 multiple-testing/overfit 评估；本 CR 产出仍不是 real-data evidence、activation 或 admission PASS。
4. 用户旅程：提出方法需求 → 确认 strategy-agnostic / fixture-only 边界 → 冻结可测行为与失败语义 → CP3 选择算法/输入合同 → CP6/CP7 本地实现验证 → 只交付 offline method ready。
5. 相邻边界：与 CR-164 typed unavailable 历史、CR-172 activation、未来 strategy identity、C1 real producer、C2/C3/C4、aggregate/FU-009、FU-006、OI-005 的差异逐项明确。

## 需求与 DQ 最低要求

- 建立固定数量、可量化的 P0 requirements；建议 8 项并与正式 CR 的 SC-001~008 对齐。
- CP2 Decision Collection 建议形成 8 个 DQ，至少覆盖：
  1. effective-count 的产品 estimand 与边界（不得等同 raw）；
  2. CP3 需要冻结的输入合同类别与有效性前提；
  3. typed_unavailable 失败语义；
  4. 7/7 evidence schema 与 provenance；
  5. strategy-agnostic 边界；
  6. deterministic golden-vector 验收；
  7. C1 consumer projection 与 overclaim ceiling；
  8. CR-172 恢复关系与 no-runtime/no-real-data 授权。
- 每个 DQ 必须包含推荐、备选、优劣、风险、回退/切换条件；方法算法的具体选择可留给 CP3，但 CP2 必须冻结用户可观察的行为与可测约束。
- 场景必须覆盖 positive、negative、boundary、permission、failure-recovery、precheck 六类，数量固定且 TEST-MATRIX trace=`100%`。

## 冻结不变项

- FU-CR164-004 的正式 ID 是 CR-173，owner 是 `strategy-admission methodology owner`。
- PATH-B 是 CR-172 activation 前置而非替代；CR-173 完成不关闭或自动恢复 CR-172。
- CR-173 是 strategy-agnostic；不得要求、猜测或冻结具体 `strategy_id/strategy_name`。
- raw-to-effective alias 违规路径目标值=`0`；缺方法、无效输入、provenance 缺失或 projection mismatch 必须 typed unavailable。
- 当前与 CP2 批准均不授权真实 lake/NAS、credential、provider、真实 computation、producer binding、runtime、write、trading、publish/deploy 或 Git remote write。
- CP2 前 Story/Epic/DAG/Wave/LLD/源码/测试源码数量均为 `0`。

## 允许写入

- `docs/product/USE-CASES.md`
- `docs/product/REQUIREMENTS.md`
- `docs/product/SCENARIOS.yaml`
- `docs/product/TEST-MATRIX.md`
- `docs/product/STORY-MAP.md`
- `docs/product/MVP-SCOPE.md`
- `docs/product/RELEASE-SLICES.md`
- `docs/product/BACKLOG.md`
- `process/CLARIFICATION-LOG.md`
- `process/discussions/CP2-CR173-SCENARIO-DISCUSSION-LOG.md`
- `process/checks/CP2-CR173-DISCUSSION-CHECKPOINT.json`
- `process/checks/CP1-CR173-USE-CASE-COMPLETENESS.md`
- `process/checks/CP1-CR173-USE-CASE-COMPLETENESS.result.json`
- `process/checks/CP2-CR173-REQUIREMENTS-BASELINE.md`
- `process/checks/CP2-CR173-REQUIREMENTS-BASELINE.result.json`
- `process/handoffs/CR173-CP1-META-PM-RETURN-SUMMARY.md`

## 禁止写入

- `process/changes/CR-173.md`、`process/changes/CR-172.md`、follow-up tracking
- `process/state/STATE.current.json`、`process/STATE.md`、`process/current/CURRENT.json`
- `process/state/*-LEDGER.ndjson`
- `process/checkpoints/CP2-CR173-REQUIREMENTS-BASELINE.md`（人工门禁由 Host 创建）
- `process/context/CP2-CR173.context.json`、human-gate launch message
- HLD、ADR、Feature、Story、LLD、实现、源码、测试源码、真实数据或外部系统

## 退出条件

- CR-173 use case=`1/1`，业务动机/痛点/使用价值/业务触发=`4/4`。
- P0 requirements、scenarios、TEST-MATRIX rows、CP2 DQ 数量均固定并实现 `100%` 追溯；建议 `8/8/8/8`。
- 六类场景=`6/6`；七字段 schema=`7/7`；alias/overclaim 目标均=`0`。
- CP1=`PASS`、blocker=`0`；CP2 自动预检=`PASS`、manual gate=`pending`。
- `ready_for_design=false`；所有真实数据/runtime/write 操作计数=`0`。
