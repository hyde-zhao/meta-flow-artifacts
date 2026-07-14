---
story_id: "CR168-S03-envelope-multi-strategy-compatibility"
title: "Neutral Envelope 激活与多策略兼容"
story_slug: "envelope-multi-strategy-compatibility"
lld_version: "1.1"
tier: "L"
status: "confirmed"
confirmed: true
created_by: "host-orchestrator-inline-meta-dev"
created_at: "2026-07-14T11:20:00+08:00"
confirmed_by: "user"
confirmed_at: "2026-07-14T12:17:56+08:00"
shared_fragments: []
feature_design_refs: ["docs/features/strategy-evidence-envelope/DESIGN.md", "docs/features/economic-cost-evidence/DESIGN.md"]
lld_policy: {required_level: "full-lld", trigger_reasons: ["shared_contract", "backward_compatibility", "hash_identity"], rationale: "catalog activation affects neutral public contract。"}
open_items: 0
---

# LLD: CR168-S03 — Neutral Envelope 激活与多策略兼容

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-14 | host-orchestrator inline meta-dev | 初始 full LLD。 |
| 1.1 | 2026-07-14 | host-orchestrator inline meta-dev | CP5 评审整改：限制既有 Feature 文档任务为 revision/traceability，禁止 S03 在实现阶段改写既有设计语义。 |

## 0. 上游工程依据

| 来源 | 消费内容 |
|---|---|
| S01/S02 | present C3 component and subject-neutral hash。 |
| FEAT-166-01 v0.2 | catalog/attachment identity/C1-C2 regression。 |
| HLD §6/SIM-02 | component same + envelope distinct assertion。 |

## 1. 目标

激活唯一 `economic_cost@v1` catalog descriptor、attach present C3 component，并对 daily/ML fixture 验证两个 hash 域的不同职责。

## 2. 需求（Functional / Non-Functional）

### 2.1 Functional

- `economic_cost@reserved`→active `economic_cost@v1`，active schema=1。
- same families 2-9 + different subject → component hash same=1；envelope hash distinct=2。
- identity/auth tamper with stale envelope hash must blocked。

### 2.2 Non-Functional

- C1/C2 bytes/hash differences=0；parallel envelope/registry=0；C4 active=0。

## 3. 模块拆分与职责

| 模块 | 职责 |
|---|---|
| `strategy_evidence.py` | static catalog/descriptor/envelope public API。 |
| compatibility test | catalog + hash split + C1/C2 regression。 |

## 4. 代码结构与文件影响范围

| 动作 | 文件 | 内容 |
|---|---|---|
| 修改 | `engine/strategy_evidence.py` | active descriptor and attachment-only identity binding。 |
| 创建 | `tests/research/test_economic_cost_envelope_compatibility.py` | daily/ML/catalog/tamper/golden assertions。 |
| 修改 | existing Feature three docs | 仅追加 CR168 revision record、实现证据链接与 traceability；不得改需求、合同、测试范围、验收、架构或原有设计语义。 |

## 5. 数据模型与持久化设计

| 对象 | 约束 |
|---|---|
| descriptor | `economic_cost`, `v1`, availability/ref/hash。 |
| component | no attachment identity in semantic body/hash。 |
| envelope | subject plus package/run provenance/auth canonical binding。 |

无 dynamic registry/persistence。

## 6. API / Interface 设计

| 接口 | 输入 | 输出 | 调用方 |
|---|---|---|---|
| existing envelope builder | component descriptor + subject | envelope hash | fixture adapters |
| catalog lookup | type/version | active descriptor | S05 |

## 7. 核心处理流程

daily/ML 输入先由 S02 produce C3；相同 semantic component attach 到不同 subject envelope；component hash 比较为相同，envelope hash 比较为不同。unknown/mandatory mismatch 或 stale hash 均 blocked。

## 8. 技术细节

只使用 existing public API；neutral module cannot import C3 producer. C1 wrapper/default domain and C2 catalog behavior retain exact compatibility. C4 capacity remains reserved/unavailable.

## 9. 安全与性能设计

| 维度 | 措施 | 验证 |
|---|---|---|
| tamper | envelope canonical self-validation | identity stale hash blocked |
| compatibility | existing goldens never updated | C1/C2 diff=0 |

## 10. 测试设计

| 场景 | 预期 | 验证 |
|---|---|---|
| catalog activation | active=1/C4=0 | F166-01-T08 |
| daily/ML | component=1, envelope=2 | F166-01-T07 |
| identity tamper | blocked | F166-01-T09 |
| C1/C2 suite | no regression | existing targeted suites |

## 11. 实施步骤

| TASK-ID | 动作 | 文件 | 对应测试 |
|---|---|---|---|
| CR168-S03-T01 | 修改 | strategy evidence catalog | T08 |
| CR168-S03-T02 | 创建 | compatibility test | T07/T09 |
| CR168-S03-T03 | 修改 | Feature docs | 仅 revision/traceability；若发现必须改设计语义，停止并回 CP3/新 CR，不在本 Story 直接处理。 |

## 12. 风险、难点与预研建议

无 OPEN。若 C1/C2 golden drift，stop activation and keep C3 unattached; do not patch goldens.

## 13. 回滚与发布策略

无发布。rollback only descriptor state to reserved; retain leaf C3 values for diagnosis.

## 14. DoD

- [ ] catalog active=1、component/envelope split test passes。
- [ ] C1/C2 regression=0、C4 activation=0。
- [ ] no reverse import/dynamic registry。
