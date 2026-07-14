---
status: "ready-for-cp5-review"
version: "1.1"
cr_id: "CR-168"
feature_id: "FEAT-168-03"
feature_name: "Economic Cost Fixture Verification"
source_hld: "process/archive/design-cr-docs/HLD-ECONOMIC-COST-IMPACT-EVIDENCE.md"
related_stories: ["CR168-S05"]
---

# FEAT-168-03 Economic Cost Fixture Verification 设计

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-14 | host-orchestrator inline meta-se | 冻结 17 场景、15 QAC、两 fixture、Gate4、CR155、授权与 claim ceiling 的静态验证编排。 |
| 1.1 | 2026-07-14 | host-orchestrator inline meta-dev | CP5 评审整改：把 capability registry 缺失的 N/A-with-reason 路径显式归属给 S05 authorization suite；不创建或持久化 registry。 |

## 1. 验证目标与对象

本 Feature 不拥有生产 API；它拥有跨 FEAT-168-01、FEAT-166-01、FEAT-168-02 的 fixture/QAC/authorization/claim regression 设计，防止各 Story 单测通过但整体产生虚假 readiness。

| 验证面 | 精确值 |
|---|---:|
| requirements / scenarios / QAC | 9/9；17/17；15/15 |
| P0/P1 | 16/16；1/1 |
| input fail-closed | 10/10 |
| fixture families | 2/2 |
| deterministic identity | 10 runs→1 component hash |
| Gate4 B01/B02 | 2/2 |
| forbidden operation / source overreach | 0 |
| C4/capacity/aggregate PASS / CR155 promotion | 0/0/0/0 |

## 2. Fixture 合同

| Fixture | 内容 | 证明 | 不证明 |
|---|---|---|---|
| daily multifactor synthetic | CNY、daily calendar、gross/trade/notional、fee/tax/spread/slippage、static square-root | itemized arithmetic、reconciliation、lineage、hash | 真实 TCA/ADV/capacity |
| multi-strategy-type | daily 与 ML 使用相同 families 2-9 成本语义、不同 strategy/package subject | same component hash + distinct envelope hash；attach 不依赖 strategy type | event semantics/真实 ML training |

fixture 文件只保存 synthetic/static values，不含客户/生产/credential 数据。event-specific 保持 deferred=1、producer=0。

## 3. 验证分层

| Layer | 文件 | 目标 |
|---|---|---|
| contract/unit | `test_economic_cost_contracts.py` | 9-family/10-class/hash/input identity。 |
| calculation | `test_economic_cost_producer.py` | Decimal/formula/rounding/reconciliation。 |
| compatibility | `test_economic_cost_envelope_compatibility.py` | catalog、daily/ML、C1/C2 regression。 |
| security integration | `test_economic_cost_gate4_projection.py` | 4/8 keys、candidate profile、postconditions。 |
| QAC/auth/claim | `test_economic_cost_cr168_qac.py`、`test_economic_cost_authorization.py`、`test_economic_cost_cr155_regression.py` | 17/15 coverage、zero ops、claim ceiling、CR155 blocked。 |

## 4. Failure attribution 与门禁

- targeted suite 必须 0 fail；full repository suite 若失败逐项记录 `introduced_by_cr168=true|false|unknown`，unknown 不得通过 CP7。
- canonical/aggregate source diff、错误质量文档路由、parallel gate/envelope/registry、C4 calculator/event producer 均要求计数 0。
- capability registry 缺失必须在 `test_economic_cost_authorization.py` 验证：以既有 Feature/module refs 形成 N/A-with-reason 审计路径，`persistent_registry_write=0`、平行 registry=0；不得把缺失误判为 capability 已满足。
- CR155 regression 必须断言 admission status=`BLOCKED` 且 `paper_candidate=false`；不得写 lifecycle still blocked 的错误表述。
- 若 CP7 仍由 inline fallback 验证，CP8 必须披露 `R-CR168-VERIFIER-INDEPENDENCE`；本 Feature 不伪造 independent agent。

## 5. 回退与 Gotchas

- 任一 P0/QAC 不可验证：回 meta-dev 修复或 meta-se clarification，不以 waiver 放行 false readiness。
- fixture 算法期待与 HLD 冲突：回设计，不更新 expected value 掩盖。
- Gotcha：Gate4 BLOCKED 是正确安全结果，不是测试失败。
- Gotcha：Stage2 complete 是前置事实；CR168 不启动 Stage3，也不把 C4 unavailable 说成 C4 producer complete。
