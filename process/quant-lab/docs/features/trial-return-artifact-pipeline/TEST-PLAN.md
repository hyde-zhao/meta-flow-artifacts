---
status: "cp5-r2-design-ready"
version: "1.1"
feature_id: "FEAT-CR172-I01"
source_design: "docs/features/trial-return-artifact-pipeline/DESIGN.md"
---

# FEAT-CR172-I01 Trial Return Artifact Pipeline — TEST-PLAN

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-18 | meta-se-critical | 初始计划。 |
| 1.1 | 2026-07-18 | meta-se-critical | 删除 runner hook/forward-label 映射测试；新增 fixture binding、verified seal digest、partial lineage BLOCKED audit 与 runner zero-diff。 |

## 1. 用例

| ID | 输入 | 期望 |
|---|---|---|
| T-S02-P01 | fixture observations + fixture decision/target | exact 2 columns；verified bundle/selection=`1/1` |
| T-S02-P02 | same seal under 3 mappings | `original_seal_sha256` distinct count=`1` |
| T-S02-N01 | `forward_label_proxy@v1` | trial-return/empirical-R/effective-count accepted=`0/0/0` |
| T-S02-N02 | fixture decision + real target | accepted/side-effect=`0/0` |
| T-S02-N03 | wrong kind / missing interval semantics | unavailable/BLOCKED；selection=0 |
| T-S02-F01 | partial payload/manifest | seal/selection=`0/0` |
| T-S02-F02 | seal digest mismatch | BLOCKED；selection=0 |
| T-S02-F03 | producer port exception | BLOCKED；selection=0 |
| T-S02-F04 | simulated append-only partial lineage | audit state=`partial_lineage_blocked_audit`；selection=0；erase/fake rollback=0 |
| T-S02-B01 | static source inventory | mature runner/lineage store diff/hook=`0/0/0` |

## 2. Public contract correlation

S02 exports exactly `SealedTrialReturnBundleV1`、`ResearchCanonicalSelectionV1`、`VerifiedTrialReturnBundleV1`、`canonical_artifact_seal_bytes`、`canonical_artifact_seal_sha256`、`verify_sealed_trial_return_bundle`。S03 secondary canonicalization/digest/re-seal count=`0/0/0`。

## 3. 边界

测试只使用 repository-owned fixtures 与 in-memory ports；真实 lake/NAS/root/runtime/lineage/signal/trading/deploy/Git remote operation=`0`。发现需要现有 runner 或原子 lineage 修改时转独立 runtime-high-risk / lineage-owner CR，不在本 Story 扩张。

## 4. Gotchas

fixture PASS 只证明合同；不证明真实 producer、真实 canonical 或真实授权。必须分别断言 source semantics、origin/target 与 seal integrity，不能用 hash 正确替代业务语义正确。
