---
story_id: "CR164-S05-independent-verification"
title: "Independent QAC permission and CR155 verification"
story_slug: "independent-verification"
lld_version: "0.1"
tier: "L"
status: "confirmed"
confirmed: true
created_by: "host-orchestrator inline meta-dev"
created_at: "2026-07-12T20:25:00+08:00"
confirmed_by: "user"
confirmed_at: "2026-07-12T20:50:00+08:00"
feature_design_refs: ["docs/features/statistical-evidence-contract/TEST-PLAN.md", "docs/features/multiple-testing-calculators/TEST-PLAN.md", "docs/features/overfit-deflation-calculators/TEST-PLAN.md", "docs/features/statistical-evidence-projection/TEST-PLAN.md"]
lld_policy: {required_level: "full-lld", trigger_reasons: ["cross-story-verification", "authorization", "negative-regression"], rationale: "Fresh independent evidence across four feature contracts."}
open_items: 0
---

# LLD: CR164-S05 — Independent verification

## 0. 上游设计依据（工程依据）

CR164 10 QAC、13 scenarios、HLD simulations、FEAT-24..27 TEST-PLAN、FEAT-28 standalone waiver。

## 1. 目标（Goal）

创建独立 fixture/static verification suite，证明 contracts、methods、aggregation、permissions、CR155 和 overclaim ceiling。

## 2. 需求（Requirements）

QAC 10/10；scenarios 13/13；binding=100%；count difference=0；negative hit=100%；10 runs→1 hash；orphan=0；projection=3/3；CR155=1/1 blocked；each forbidden counter=0；overclaim=0。

## 3. 模块拆分

QAC trace/golden/determinism tests、authorization static scan、CR155 negative regression。S05 不拥有 production code，不在测试内 monkeypatch 修复实现。

## 4. 代码结构与文件影响

创建 `tests/research/test_statistical_evidence_qac.py`、`tests/research/test_statistical_evidence_authorization.py`、`tests/research/test_statistical_evidence_cr155_regression.py`。禁止修改 `engine/**`。

## 5. 数据模型

仅 synthetic fixtures 与 expected values；fixture 含 explicit schema/family refs/candidate ids/returns/moments/splits/config。无真实数据或持久化。

## 6. API

消费 S01-S04 public APIs。Test evidence index/return packet 在 CP7 阶段生成；本 Story 不创建新的 runtime API。

## 7. 流程

inventory 9 REQ/13 SCN/10 QAC → run unit/contract/integration/golden/determinism/negative/static → aggregate evidence index → independent CP7 decision。任一 false PASS、missing denominator、permission nonzero 阻断。

## 8. 技术细节

参数化 matrix 显式列 denominator；hash test 重建对象 10 次而非重复读取同一对象；authorization test 扫描 call/fixture paths 和 operation counters；CR155 使用已封存 negative fixture，不重建历史 lineage。

## 9. 安全与性能

No network/env credentials/NAS/provider/broker/trading/real lake。所有临时路径由 test temp dir。性能仅记录 test duration，不声明统计生产容量。

## 10. 测试设计

13 scenario ids 与 10 QAC 一一映射；method golden values由 S02/S03 primary-source fixtures提供；consumer combinations由 S04提供；authorization/CR155独立文件覆盖。

## 11. 实施步骤（TASKS）

S05-T01 QAC/scenario/golden/determinism；T02 authorization/overclaim；T03 CR155/projection/regression and evidence mapping。

## 12. 风险与灰区

无 LCQ。风险为测试与实现同源误差；CP7 必须 fresh independent review。若 no-subagent 指令延续，由 Host inline QA 执行并显式记录非独立-agent限制，不得伪造子 Agent。

## 13. 回滚

测试文件可独立回滚；不影响 source/runtime。任一 test failure 路由回相应 Story，不允许 waiver 高风险 false PASS。

## 14. DoD / Definition of Done

- [ ] QAC=10/10；SCN=13/13；denominators明确
- [ ] deterministic/negative/permission/CR155 全满足
- [ ] tests 不修改 source、不访问真实系统；CP5 前不实现
