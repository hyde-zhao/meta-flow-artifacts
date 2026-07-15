---
story_id: "ST-WT-003"
title: "实现 lifecycle-aware Doctor 与历史治理"
story_slug: "doctor-history-governance"
lld_version: "1.0"
tier: "L"
status: "approved"
confirmed: true
created_by: "host-orchestrator-inline/meta-dev"
created_at: "2026-07-14"
confirmed_by: "user"
confirmed_at: "2026-07-15T12:05:17Z"
feature_design_refs: ["process/docs/features/cr047-quality/DESIGN.md", "process/docs/features/cr047-quality/TEST-PLAN.md", "process/docs/features/cr047-quality/TASKS.md"]
lld_policy: {required_level: "full-lld", trigger_reasons: ["policy-schema", "history-integrity", "cross-module"]}
open_items: 0
---

# LLD: ST-WT-003 — Doctor History Governance

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-14 | host-orchestrator-inline/meta-dev | 冻结 B0 双锚、lifecycle/read class 与 append-only 历史治理。 |

## 0. 工程依据与上游设计依据

工程依据为 CP3-DQ-03、ADR-WT-003、Feature quality design、当前 token/artifact/quality/workflow Doctor。固定“21”已从历史观测漂移到当前 22，故成功判定必须基于对象分类、active/default-required blocker 与可解释 delta，而非固定总数。

## 1. 目标

让 Doctor 对每个超预算对象输出 lifecycle/read class、blocker/warning、remediation ref，并以 `B0_pre` 历史回归锚和 `B0_cp7` 动态分类锚证明所有 delta 可解释；同时用 append-only correction 处理历史 provenance 缺口。

## 2. 需求（Functional / Non-Functional）

- `B0_pre` 保存 CR-047 产物生成前的历史观测（21）用于回归解释，不作为最终固定阈值。
- `B0_cp7` 在 CP7 开始实测 observed/classified/unclassified/blocking-active/warning。
- active 或 default-required 超预算始终 BLOCKER；unclassified=BLOCKER；closed/archive/reference-only 可 warning但必须有 remediation/ref。
- 每个新增/删除/重分类 delta 都含 object identity、before/after class、reason、evidence ref。
- 历史 CP/read expansion/run 缺口只追加 correction/recovery；不伪造 actor/time/receipt，不覆盖原 hash。

## 3. 模块拆分与职责

| 模块 | 职责 |
|---|---|
| `meta_flow/checks/token_budget.py` | 发现超预算对象、计算 lifecycle/read class 与 severity |
| `meta_flow/checks/quality_governance.py` | 验证 QUALITY-MODEL allowed_sources/correction refs |
| `meta_flow/context_pack/{builder,read_expansion}.py` | 复用 read profile/ledger schema，不回写旧事件 |
| `process/policies/QUALITY-MODEL.yaml` | 修正 canonical read-expansion source 路径 |
| `tests/test_cr047_quality_governance.py` | B0/delta/classification/correction fixtures |

## 4. 代码结构与文件影响范围

修改 token budget、quality governance、必要的 context read helpers 和 QUALITY-MODEL；创建 CR-047 quality tests。ledger 只允许追加新 recovery/correction event，不修改旧行；历史 CP result 不原位重写。

## 5. 数据模型与持久化设计

`BudgetObservation`：path、size、limit、lifecycle_class(`active|closed|archive|unknown`)、read_class(`default-required|reference-only|cold`)、remediation_ref、severity。`BudgetBaseline`：kind(`B0_pre|B0_cp7`)、observed/classified/unclassified/blocking_active/warning、captured_at、evidence_ref。`BudgetDelta`：identity、operation、before/after、reason/ref。新增证据走 append-only result/ledger，不建第二 Doctor truth。

## 6. API / Interface 设计

`classify_budget_observation(path, policy, lifecycle) -> observation`；`reconcile_baselines(pre, current) -> deltas+findings`；Doctor 现有入口渲染计数与明细。缺 classification/remediation 或 active/default-required 超限时非零退出；warning 不得被渲染为 PASS-without-risk。

## 7. 核心处理流程

1. 扫描当前 policy 范围并记录 `B0_cp7` 原始观测。
2. 以 CR/state/archive/read-policy 推导两类 class；无法推导则 unknown/unclassified。
3. 根据 severity 规则生成 blocker/warning。
4. 与 `B0_pre` 历史快照按对象 identity 对账 delta。
5. 验证 remediation/correction refs 与 allowed_sources。
6. 输出 deterministic report；不截断、不搬迁、不覆盖历史对象。

## 8. 技术细节与设计细节

identity 使用 project-relative path+content hash+object type，不以计数代替对象对账。CR-047 自己新增 HLD/LLD/summary 导致 `B0_cp7` 上升是允许的，但必须分类；`blocking_active_over_budget=0` 才成功。QUALITY-MODEL canonical ledger 修正为 `process/state/READ-EXPANSION-LEDGER.ndjson`。

## 9. 安全与性能设计

扫描范围由 policy 明确列出，默认跳过 archive 内容读取但可读元数据；禁止跟随到 backup/quant-lab。线性扫描文件和 ledger；避免加载完整大型 artifact。correction 必须带原始 hash 且不得声称不可恢复的授权已存在。

## 10. 测试设计

| 场景 | 预期 |
|---|---|
| active/default-required 超限 | BLOCKER |
| closed/reference-only 有 remediation | warning、可披露 |
| unknown/unclassified | BLOCKER |
| 21→22 已解释新增 | delta=1、非固定计数失败 |
| 新增 active LLD 未分类 | blocker |
| QUALITY-MODEL canonical source | 路径检查通过 |
| append-only correction | 原 hash 不变、新事件可追踪 |
| 伪造 actor/time/receipt | fixture 拒绝 |
| Doctor 重复运行 | 排序/计数确定 |

## 11. 实施步骤

1. `TASK-WT-003-01`：实现 lifecycle/read class 与 severity schema。
2. `TASK-WT-003-02`：实现 B0_pre/B0_cp7 对账和 delta finding。
3. `TASK-WT-003-03`：修正 QUALITY-MODEL source，增加 append-only correction 校验。
4. `TASK-WT-003-04`：补正负 fixture并运行 Doctor/quality/read-expansion 回归。

## 12. 风险、难点与预研建议

风险是把 active 产物误降为 warning、用 archive 隐藏 required evidence、或美化遗失 provenance。unknown 一律 blocker；archive 不改变 required class；legacy unavailable 显式保留。若需移动/改写历史对象，另开 CR，不在本 Story 执行。

## 13. 回滚与发布策略

分类逻辑先以报告形式落地，fixture 完整后再成为 release blocker。误分类时回滚 severity 接线，保留原始扫描和 delta 证据；不还原/覆盖历史 ledger。发布结论仍受 DQ-06 风险上限约束。

## 14. DoD（Definition of Done）

- [ ] 0–14 章节完整、`open_items=0`。
- [ ] `unclassified_over_budget=0`、`blocking_active_over_budget=0`。
- [ ] B0_pre/B0_cp7 所有新增、删除、重分类 delta 100% 有 reason/ref。
- [ ] QUALITY-MODEL canonical source 正确，历史原 hash 变化=0。
- [ ] 正负 fixture 和 Doctor/quality 回归通过。
- [ ] CP5 未批准前 `confirmed=false`，不开始实现。

## 人工确认区

- 结论：`approved`
- 审查人：user
- 审查时间：2026-07-15T12:05:17Z
- 修改意见：
- 风险接受项：inline fallback；CP7/CP8 风险上限。
