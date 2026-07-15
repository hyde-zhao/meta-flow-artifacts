---
status: "ready-for-cp5-review"
version: "1.0"
cr_id: "CR-169"
feature_id: "FEAT-169-04"
feature_name: "C4 Fixture Claim and Stage2 Exit Guard"
related_stories: ["CR169-S05"]
---

# FEAT-169-04 C4 Fixture Claim and Stage2 Exit Guard 设计

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-14 | host-orchestrator inline meta-se/meta-dev | 冻结 fixture/QAC/authorization/CR155 回归及 7/7 Stage2 exit 证据边界。 |

## 1. 目标与验证所有权

本 Feature 汇总 2/2 fixture、17/17 CR169 scenarios、15/15 QAC、12/12 P0、10→1 hash、strict Gate4 joint outcome、CR168 absent-C4 回归及 claim ceiling。它不拥有 C4 计算和 adapter 生产代码，也不产生真实 evidence。

`STAGE2-EXIT-VERIFICATION.result.json` 在 CP8 由 Host Orchestrator 生成，逐项列出：FactorSpec、FactorRunSpec、factor panel、label window、evaluation、portfolio/risk、admission package/evidence index。每项必须是 PASS/FAIL/BLOCKED + evidence refs；总 PASS 仅在 7/7 PASS。历史前 6 项有缺口时路由 CR-157 owner 或新治理 CR；CR-169 不修复它们，其本地 C4 交付可独立判定，但 Stage2 completion claim 不成立。

## 2. Claim ceiling

固定 false/zero：`stage3_entry_ready`、real ADV、real liquidity、real capacity calibrated/ready、aggregate admission PASS、capacity-scalable claim、alpha-decay calculator、CR155 promotion、runtime/trading/external operations。`gate4_fixture_contract_pass` 最大为 1，不能传入 StrategyAdmissionPackage。

## 3. Fixture 与回归

- daily multifactor synthetic family：可复算三 refs 与 10→1。
- daily/ML multi-strategy family：相同 computational body 可同 component hash，不同 attachment 得不同 envelope hash；跨 subject join fail-closed。
- CR168 C3-only absent-C4 adapter 保持原行为；canonical/aggregate/source modifications=0。
- CR155 继续 `BLOCKED` 且 `paper_candidate=false`。

## 4. 风险与 Gotchas

- inline verifier independence 在 CP8 必须披露；不能用测试数量替代独立性说明。
- 7/7 result 是事实核验，不是把历史债务吸收到 CR-169。
- 即便 7/7 PASS，`stage3_entry_ready` 仍为 false，需独立授权与 follow-up。
