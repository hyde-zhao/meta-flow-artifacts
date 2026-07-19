---
status: "cp4-design-ready"
version: "1.2"
feature_id: "FEAT-CR172-I02"
feature_name: "Research Artifact Replica and Materialization"
source_blueprint: "docs/design/BLUEPRINT.md"
source_hld: "docs/design/HLD-TRIAL-RETURN-DEPLOYMENT-CONTRACTS.md"
source_adr: "docs/design/ARCHITECTURE-DECISION-TRIAL-RETURN-DEPLOYMENT-CONTRACTS.md"
source_matrix: "docs/design/FEATURE-DESIGN-MATRIX.md"
related_stories:
  - "CR172-S03-nas-replica-verification"
  - "CR172-S04-execution-cache-materialization"
  - "CR172-S05-path-i-integration-claim-zero-operation-verification"
lld_policy_summary: "3/3 full-lld; CP4 不创建 LLD"
confirmed_by: ""
confirmed_at: ""
---

# FEAT-CR172-I02 Research Artifact Replica and Materialization — DESIGN

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-18 | meta-se-critical | 初始 CP4 Feature 设计；冻结 research-local canonical→NAS verified replica→execution immutable cache 的合同、receipt、原子 pointer 与恢复边界。 |
| 1.1 | 2026-07-18 | meta-se-critical | CP5 R1：S03 输入精确绑定 S02 public types/verifier；`original_seal_sha256` 只取 verified result，禁止二次 digest；S04 测试口径固定为 16 个唯一 ID。 |
| 1.2 | 2026-07-18 | meta-se-critical | CP5 R2：S04 仍只从 S03 selected replica 取数，但允许 S02 verifier-library dependency；staging port 返回 sealed bundle+selection，bytes-level 复验不得由 receipt 或第二 digest替代。 |

## 1. 摘要与边界

| 项目 | 内容 |
|---|---|
| Feature 目标 | 在不改变 research canonical authority 的前提下，以原 seal 和 expected release 为依据验证 NAS 副本，并在执行机原子物化本地 immutable cache。 |
| 推荐方案 | 两个独立模块/Story：S03 只负责 replica；S04 只负责 materialization；S05 只读验证。 |
| 关键取舍 | NAS 只作 replica/distribution，不作 runtime working canonical；执行机不直读 NAS/研究机。 |
| LLD 策略 | S03/S04/S05 `3/3` full-lld；真实 sync/pull 执行不属于当前实现授权。 |

成功标准：三段 owner=`3/3` unique；stable logical identity=`100%`；partial/stale/hash/release mismatch pointer advance=`0`；direct-NAS runtime read=`0`；前一 verified selection 保留=`100%`；真实 NAS/pull actions authorized/executed=`0/2`,`0/2`。

## 2. 模块、对象与文件 owner

| Story / Module | 主要文件 | 输入 | 输出 | 非职责 |
|---|---|---|---|---|
| S03 `research_artifact_replica` | `engine/research_artifact_replica.py`、`tests/research/test_cr172_nas_replica_verification.py` | `SealedTrialReturnBundleV1`、`ResearchCanonicalSelectionV1`、`ActionDecisionV1`、`ActionScopeContextV1` 与 expected release | `VerifiedTrialReturnBundleV1` 驱动的 `ReplicaVerificationReceiptV1` + optional fixture distribution selection | 不 reopen path、不重 seal、不计算第二套 digest、不做真实 NAS |
| S04 `research_artifact_materialization` | `engine/research_artifact_materialization.py`、`tests/research/test_cr172_execution_materialization.py` | S03 selected replica staging port 返回的 `SealedTrialReturnBundleV1 + ResearchCanonicalSelectionV1`、receipt、expected release、eligible pull decision | S02 唯一 verifier bytes-level 复验后的 `MaterializationReceiptV1` + immutable local cache selection | 不绕过 S03 selection、不信任 receipt 替代 bytes、不建第二 digest |
| S05 QAC | 独立 fixtures/QAC tests | S02-S04 contracts | 全链 failure/recovery/zero-op evidence | 不修改 S03/S04 production modules |

planned production/test paths=`4`，duplicate primary owner=`0`。S03 与 S04 文件完全分离；依赖是 runtime/artifact 依赖，不是共享写入。

## 3. 数据与接口合同

| Object | Owner | 必须验证 | pointer 规则 |
|---|---|---|---|
| `ReplicaVerificationReceiptV1` | S03 | 先调用 S02 `verify_sealed_trial_return_bundle`；expected release、logical URI、content/manifest hash、`VerifiedTrialReturnBundleV1.original_seal_sha256`、freshness、source selection | 全部 PASS 才推进 fixture distribution selection；`original_seal_sha256` 不得自行复算 |
| `MaterializationReceiptV1` | S04 | expected release + manifest/seal/content `4/4`、replica receipt、execution mapping | 全部 PASS 才原子推进 local cache pointer |
| deployment mapping | I02 | logical URI→research/NAS/execution absolute path | path 不进 identity/hash；不同 host 同 content identity 仍为 1 |

数据流固定：S02 sealed bundle → S03 verify/select → S04 从 S03 selected replica 拉取 sealed bundle+selection → 复用 S02 唯一 verifier library → materialize。S04 的 S02 依赖仅是 verifier-library contract，不是数据选择旁路；绕过 S03 selection、execution direct-NAS/research read、NAS canonical promotion=`0`。

## 4. 前置、失败与恢复

| Flow | 前置 | Failure | Fail-closed / 恢复 |
|---|---|---|---|
| NAS replica | S02 original seal + S01 eligible `nas_replica_sync` | partial/stale/unversioned/hash/release mismatch/revoke | 不创建或不发布 staging；pointer 不推进；上一 verified replica 保留 |
| execution materialize | verified replica + S01 eligible `execution_pull_verify_materialize` | direct-NAS runtime request、pull interruption、4/4 mismatch/revoke | staging non-runtime；cache pointer 不推进；上一 cache 保留 |
| cleanup/retry | 新的独立 action authorization | 复用旧过期授权或跳过复验 | deny；不删除 immutable evidence；重新完整验证 |

回滚只切 pointer，不覆盖或重写 sealed/receipt bytes。S03 失败不得改变 research canonical；S04 失败不得改变 research/NAS selection。

## 5. Story / Wave 与实现顺序

| Wave | Story | depends_on | 理由 |
|---|---|---|---|
| W3 | S03 NAS replica verification | S01 contract + S02 sealed artifact | 必须先有原 seal 和独立 sync eligibility |
| W4 | S04 execution cache materialization | S01 contract + S02 verifier-library contract + S03 selected replica/receipt | 必须由 S03 selection 供数；允许复用 S02 verifier，禁止绕过 S03 取数 |
| W5 | S05 integrated verification | S01-S04 | 全链 recovery/claim/zero-op 收口 |

merge order=`S01→S02→S03→S04→S05`；same-wave conflict=`0`。

## 6. 测试与验收

- S03：normal fixture、partial/stale/unversioned/content/manifest/seal/release mismatch、mid-operation revoke、research pointer unchanged。
- S04：valid `4/4`、direct-NAS runtime deny、pull interruption、cache atomicity、previous cache preservation。
- S05：三 host mapping identity、两个 authorization action no-union、真实 NAS reads/writes/pointer advances=`0`。

## 7. 风险与 Gotchas

| Risk | 缓解 | 回退 |
|---|---|---|
| NAS 被误作 canonical | receipt 不拥有 canonical authority；forbidden dependency static guard | research-local only / distribution blocked |
| “最新文件”替代 expected release | request 必须携带 expected release/hash | 缺失即 deny |
| pull 成功被误作 runtime-ready | 只有 materialization receipt + atomic selected cache 可消费 | 保留上一 cache |

Gotchas：复制字节成功不等于 verified replica；materialize 成功也不能生成新 seal；repository-local fake mapping 不构成真实 NAS/执行机授权。
