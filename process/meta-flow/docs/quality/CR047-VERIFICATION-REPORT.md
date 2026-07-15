---
cr_id: "CR-047"
checkpoint: "CP7"
decision: "PASS_WITH_RISK"
validation_mode: "mixed/local-deterministic"
verified_at: "2026-07-15"
verified_by: "host-orchestrator-inline/meta-qa"
independent_qa_attestation: "unavailable"
---

# CR-047 Verification Report

## 验证对象与追踪

| Story | 主要契约 | CP7 证据 | 结论 |
|---|---|---|---|
| ST-WT-001 | State/CR/CURRENT relation、closed active fail-closed | truth/lifecycle fixtures、CR tracking | PASS_WITH_RISK |
| ST-WT-002 | relative/idempotent routing、单一 internal docs canonical | routing/relocation/regular-path fixtures | PASS_WITH_RISK |
| ST-WT-003 | lifecycle/read budget、append-only legacy correction | quality fixtures、Doctor B0_cp7 | PASS_WITH_RISK |
| ST-WT-004 | tracked canonical wrapper、cache precedence | guardrail fixture、clean-staging proof | PASS_WITH_RISK |
| ST-WT-005 | Ruff=0、full regression | Ruff 与 400+70 tests | PASS_WITH_RISK |
| ST-WT-006 | 3 平台非交互 dry-run、五门 preflight | docs fixture、3/3 dry-run | PASS_WITH_RISK |
| ST-WT-007 | object-identity firewall、CR-046 current projection | 57-object CP7 hash recheck | PASS_WITH_RISK |

## 分层结果

- Unit/contract：400 tests 全通过，另有 70 subtests；CR-047 新增 truth、routing、quality、guardrail、operator 负向 fixture。
- Static：Ruff 0 error；`git diff --check` 0 error。
- Integration：workspace route health=ok；CR tracking active=`CR-047`、candidate=`CR-033`、stale conflict=0。
- Delivery：clean-clone root wrapper 不再是隐式依赖；当前 guardrail exit 0，仅 ignored local cache warning。
- Doctor：总体 exit 0；`B0_cp7=21`、blocking active/default-required=0、unclassified=0。相对 `B0_pre=21` 最终 delta=0；CP3 临时 +1 来自 CR-047 summary，compact writer 后回落，解释链完整。
- Installer：Codex/Claude/Qoder project/full/explicit-project-dir dry-run 3/3，真实目标写入=0。
- Firewall：CP6 pre 57 objects；CP6 completion=0 findings；CP7 recheck=0 findings；未触发 child CR。

## 风险与阶段决策

无 blocker、无 NEEDS_REWORK、无设计澄清。以下风险仍 OPEN 且相互叠加：本次无独立 meta-qa 子 Agent；平台 receipt 不可得；token telemetry 不可得；真实 pilot 未授权；结果仅证明 working tree；ignored local cache 与历史 reference-only 超预算仍告警。因此 CP7 结论严格限制为 `PASS_WITH_RISK`，不得声称 independent-QA-verified、platform-attested 或 production-ready。
