---
status: "cp3-approved"
version: "0.2"
change: "CR-158"
title: "Architecture Decisions: Event + ML Strategy Adapter"
created_at: "2026-07-05T17:20:00+08:00"
owner: "host-orchestrator"
confirmed_by: "user"
confirmed_at: "2026-07-05T17:35:00+08:00"
---

# Architecture Decisions: Event + ML Strategy Adapter

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 0.1 | 2026-07-05 | host-orchestrator | CR158 CP3 初稿：提出 shared core、refs-only evidence 和 no-runtime guard 三项 ADR。 |
| 0.2 | 2026-07-05 | host-orchestrator | CP3 approved：接受 ADR-CR158-001..003 的推荐方案并进入 CP4 Story planning。 |

## ADR-CR158-001 Thin Shared Core + Typed Extensions

| 字段 | 内容 |
|---|---|
| 状态 | accepted at CP3 |
| 决策 | `StrategyTypeAdapterCore` 只承载公共 refs、status、blocked reasons、authorization flags 和 handoff refs；event 与 ML 差异放入 type-specific extension。 |
| 推荐理由 | 满足 CP2 的统一 CR 决策，同时避免 event-only / ML-only 字段互相污染。 |
| 备选 A | Fat common schema，把 event 与 ML 字段放入一张大 schema。 |
| 备选 B | Two independent adapters，event 与 ML 完全分开。 |
| 影响 | HLD、CP4 Story 分组、CP5 LLD、contract tests、release wording。 |
| 回退 / 切换条件 | CP5 发现 extension 仍互相污染或 shared core 字段不足以稳定消费时，拆分子 CR 或改为 two independent adapters。 |

## ADR-CR158-002 Adapter Evidence Uses Typed Refs Only

| 字段 | 内容 |
|---|---|
| 状态 | accepted at CP3 |
| 决策 | Evidence index 只存 event/ML typed refs 和短元数据；不复制 report body、event payload body、model binary、transcript、full diff 或大型矩阵。 |
| 推荐理由 | 延续 CR157 refs-only evidence baseline，避免 process/evidence 膨胀和敏感内容泄露。 |
| 备选 A | 在 adapter evidence 中嵌入 report/event/model body。 |
| 备选 B | 不扩展 evidence index，只让 adapter validator 自行输出报告。 |
| 影响 | Evidence schema、CP7 traceability、CP8 wording。 |
| 回退 / 切换条件 | 若后续确需大型正文，必须进入外部 artifact store / archive refs，并另行设计 retention policy。 |

## ADR-CR158-003 Adapter Counter Report + FEAT-07 Guard

| 字段 | 内容 |
|---|---|
| 状态 | accepted at CP3 |
| 决策 | Adapter validator 产出 forbidden operation counters；FEAT-07 no-runtime policy 统一判定 counters 是否全 0。 |
| 推荐理由 | 单靠文档审查不足以防止 runtime overclaim；双层设计让 CP7 可用结构化证据证明 no-runtime。 |
| 备选 A | 只在 CP7 文档人工审查中确认不授权。 |
| 备选 B | CP3 直接授权只读真实数据或 runtime proof。 |
| 影响 | CP5 design contract、CP6 implementation evidence、CP7 verification report、release wording。 |
| 回退 / 切换条件 | 用户明确要求真实 feed / training / registry / runtime 验证时，另起 runtime authorization CR。 |

## ADR 自检

| 检查项 | 结果 | 证据 |
|---|---|---|
| 每项 ADR 有推荐方案和备选方案 | PASS | ADR-CR158-001..003 |
| ADR 回写到 HLD 对应章节 | PASS | HLD §4 / §6 / §12 |
| 不授权边界未被放大 | PASS | ADR-CR158-003 |
