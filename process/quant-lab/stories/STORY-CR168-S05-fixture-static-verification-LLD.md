---
story_id: "CR168-S05-fixture-static-verification"
title: "Fixture 静态验证、授权与 Claim Regression"
story_slug: "fixture-static-verification"
lld_version: "1.1"
tier: "L"
status: "confirmed"
confirmed: true
created_by: "host-orchestrator-inline-meta-dev"
created_at: "2026-07-14T11:20:00+08:00"
confirmed_by: "user"
confirmed_at: "2026-07-14T12:17:56+08:00"
shared_fragments: []
feature_design_refs: ["docs/features/economic-cost-verification/DESIGN.md", "docs/features/economic-cost-evidence/TEST-PLAN.md", "docs/features/strategy-evidence-envelope/TEST-PLAN.md", "docs/features/economic-cost-gate4-projection/TEST-PLAN.md"]
lld_policy: {required_level: "full-lld", trigger_reasons: ["cross_feature", "security", "claim_sensitive", "regression"], rationale: "跨 Feature verification and claim ceiling。"}
open_items: 0
---

# LLD: CR168-S05 — Fixture 静态验证、授权与 Claim Regression

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-14 | host-orchestrator inline meta-dev | 初始 full LLD。 |
| 1.1 | 2026-07-14 | host-orchestrator inline meta-dev | CP5 评审整改：将 capability registry missing 的 N/A-with-reason 验证明确归属给 authorization suite，保持 registry 创建/写入为 0。 |

## 0. 上游工程依据

| 来源 | 消费内容 |
|---|---|
| S01-S04 | C3, envelope, Gate4 contracts。 |
| FEAT-168-03 | 17 scenarios/15 QAC/2 fixtures/CP7 attribution。 |
| HLD §9/§11/§12 | claim ceiling/CR155/authorization。 |

## 1. 目标

建立两组 synthetic fixtures 和 cross-feature verification，证明静态 C3 foundation 安全而不宣称 real TCA、C4/aggregate PASS 或 CR155 promotion。

## 2. 需求（Functional / Non-Functional）

### 2.1 Functional

- daily multifactor fixture + daily/ML compatibility fixture=2/2。
- REQ=9/9、SC=17/17、QAC=15/15、P0=16、P1=1；negative=10/10；10 runs→1。
- B01/B02=2/2；CR155 remains BLOCKED/paper_candidate=false。

### 2.2 Non-Functional

- forbidden ops/source overreach/wrong quality refs=0；CP7 full-suite unknown attribution=0。

## 3. 模块拆分与职责

| 组 | 职责 |
|---|---|
| fixtures | static daily/multi-strategy inputs only。 |
| QAC tests | scenario/method/hash/Gate4 assertions。 |
| authorization tests | no data/runtime/trading/registry/Git operations。 |
| regression tests | C1/C2 and CR155 claim boundary。 |

## 4. 代码结构与文件影响范围

| 动作 | 文件 | 内容 |
|---|---|---|
| 创建 | `tests/fixtures/economic_cost/` | 2 fixture families。 |
| 创建 | `test_economic_cost_cr168_qac.py` | 17/15/10/10/10→1。 |
| 创建 | `test_economic_cost_authorization.py` | forbidden/doc/source guards；capability registry missing 的 N/A-with-reason + existing Feature/module refs 验证。 |
| 创建 | `test_economic_cost_cr155_regression.py` | blocked/paper claim checks。 |

## 5. 数据模型与持久化设计

Fixture contains only `fixture://` refs and synthetic Decimal/string values. No real data, persistence, credentials or provider pointers.

## 6. API / Interface 设计

| 入口 | 输入 | 输出 | 调用方 |
|---|---|---|---|
| fixture builders | static values | typed C3 input | tests |
| QAC suite | S01-S04 public APIs | assertions/evidence | CP7 |
| attribution record | test result list | introduced_by map | CP7 quality artifact |

## 7. 核心处理流程

fixture→S01 validate→S02 produce→S03 attach→S04 project→assert C4 BLOCKED→assert claims/operations/CR155 ceiling. Full suite failures are classified; unknown stops CP7.

## 8. 技术细节

Event fixture is intentionally absent. `cost_underestimation_status=PASS` is contract completeness only. If no independent verifier exists, preserve risk and disclose at CP8; do not emulate independent dispatch.

capability registry 不是本 Story 的交付物：authorization suite 必须在 registry 缺失时断言使用既有 Feature/module refs 的 N/A-with-reason，且 `persistent_registry_write=0`、parallel registry=0。不得把此 N/A 路径解释成 capability registry 已存在或已满足。

## 9. 安全与性能设计

| 维度 | 措施 | 验证 |
|---|---|---|
| privacy/auth | synthetic only/no dereference | authorization suite |
| claim | negative checks/CR155 blocked | QAC/regression |
| reproducibility | fixture hashes/10 reruns | QAC |

## 10. 测试设计

| 场景 | 预期 | 验证 |
|---|---|---|
| two fixture families | 2/2 complete | EV-T02 |
| all requirements/scenarios/QAC | 9/9/17/17/15/15 | EV-T01 |
| B01/B02 | non-PASS/escape=0 calls | EV-T04 |
| authorization | all prohibited=0 | EV-T06 |
| capability registry missing | existing Feature/module refs 的 N/A-with-reason；persistent/parallel registry=0 | EV-T06 |
| CR155 | BLOCKED, paper false, promotion=0 | EV-T05 |

## 11. 实施步骤

| TASK-ID | 动作 | 文件 | 对应测试 |
|---|---|---|---|
| CR168-S05-T01 | 创建 | fixture directory | EV-T02 |
| CR168-S05-T02 | 创建 | QAC suite | EV-T01/03/04 |
| CR168-S05-T03 | 创建 | authorization suite（含 capability-registry N/A 路径） | EV-T06/07 |
| CR168-S05-T04 | 创建 | CR155 suite | EV-T05 |

## 12. 风险、难点与预研建议

无 OPEN。`R-CR168-VERIFIER-INDEPENDENCE` 非阻断但 CP8 mandatory disclosure。full suite failure attribution is a CP7 blocker when unknown.

## 13. 回滚与发布策略

无发布。任何 fixture/claim drift stop CP7 and route to owning Story; no waiver for false readiness.

## 14. DoD

- [ ] 2/2 fixtures；9/9/17/17/15/15 trace。
- [ ] negative=10/10；10 runs→1；B01/B02=2/2。
- [ ] C4/capacity/aggregate PASS=0；CR155 promotion=0。
- [ ] forbidden operations/wrong quality ref=0。
