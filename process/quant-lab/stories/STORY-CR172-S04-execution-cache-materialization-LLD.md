---
story_id: "CR172-S04-execution-cache-materialization"
title: "Execution-local immutable cache verification and atomic materialization"
story_slug: "execution-cache-materialization"
lld_version: "1.3"
tier: "L"
status: "confirmed"
confirmed: true
created_by: "meta-dev"
created_at: "2026-07-18T11:08:00+08:00"
confirmed_by: "user"
confirmed_at: "2026-07-18T13:49:27+08:00"
implementation_allowed: true
shared_fragments: []
feature_design_refs:
  - "docs/features/trial-return-artifact-pipeline/DESIGN.md"
  - "docs/features/trial-return-artifact-pipeline/TEST-PLAN.md"
  - "docs/features/trial-return-artifact-pipeline/TASKS.md"
  - "docs/features/research-artifact-replica-materialization/DESIGN.md"
  - "docs/features/research-artifact-replica-materialization/TEST-PLAN.md"
  - "docs/features/research-artifact-replica-materialization/TASKS.md"
  - "docs/features/path-i-authorization-claim-governance/DESIGN.md"
  - "docs/features/path-i-authorization-claim-governance/TEST-PLAN.md"
  - "docs/features/path-i-authorization-claim-governance/TASKS.md"
lld_policy:
  required_level: "full-lld"
  trigger_reasons:
    - "execution-machine boundary"
    - "pull verification"
    - "atomic immutable cache"
    - "direct-NAS deny"
  rationale: "执行机数据边界、verified replica pull、non-runtime staging、4/4 复验和 cache pointer rollback 必须在实现前冻结。"
open_items: 0
---

# LLD: CR172-S04 — Execution-local immutable cache verification and atomic materialization

> 本文仅设计 repository-local contract、注入式 materialization port 和 fixture 验证面。它不提供真实 NAS pull/执行机 adapter，不实现 future runtime，不批准或执行 `execution_pull_verify_materialize`。

### 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-18 | meta-dev | 初始 S04 full LLD。 |
| 1.1 | 2026-07-18 | meta-dev | CP5 R2：精确消费 S03 `ReplicaVerificationReceiptV1`/`DistributionSelectionV1`，贯穿 S01 fixture origin/target 绑定，并将测试表固化为 16 个唯一 ID、duplicate=`0`。 |
| 1.2 | 2026-07-18 | meta-dev | CP5 R3：增加 S02 verifier-library contract dependency；S04 只从 S03 current selection 获取 typed sealed bundle+selection+receipt，staging 后调用唯一 verifier 再计算 `4/4`；bypass/receipt-only/secondary digest=`0/0/0`，补 tampered seal bytes 负例。 |
| 1.3 | 2026-07-18 | host-orchestrator | CP5 批准前 optional 整改：authority pointer-only 刷新至 HLD/ADR v1.4；normative delta=`0`。 |

## 0. 工程依据（上游设计依据）

| 来源 | 路径 / ID | 被本 LLD 消费的内容 |
|---|---|---|
| HLD v1.4 | `docs/design/HLD-TRIAL-RETURN-DEPLOYMENT-CONTRACTS.md` §9.1～§12 | S03-selected replica→execution local immutable fixture cache、S02 verifier-library、FLOW-I03、bytes-level 4/4、direct-NAS deny |
| ADR v1.4 | `docs/design/ARCHITECTURE-DECISION-TRIAL-RETURN-DEPLOYMENT-CONTRACTS.md` / ADR-CR172-I-002～005、009、011 | logical identity、fixture/real typed binding、S03 数据选择 + S02 verifier 分离、独立 pull 授权、失败保留上一 cache |
| Feature Matrix v1.30 | `docs/design/FEATURE-DESIGN-MATRIX.md` / CR172 PATH-I CP4 增量 | S04=`full-lld`、Wave 4、S01 governance/S02 verifier-library/S03 runtime-selection 三前置、owner、禁止 direct-NAS/runtime |
| Trial-return Feature v1.2 | `docs/features/trial-return-artifact-pipeline/DESIGN.md` | S02 exact public types、唯一 `verify_sealed_trial_return_bundle`、canonical seal truth；S04 仅 verifier-library dependency |
| Replica Feature v1.2 | `docs/features/research-artifact-replica-materialization/DESIGN.md` | S03 selected replica port→typed staging bundle/selection→S02 verifier→4/4→atomic selection |
| Replica TEST-PLAN v1.2 | `docs/features/research-artifact-replica-materialization/TEST-PLAN.md` | I02-P02/N02/N03/F02/F03、tampered-seal negative、bypass/receipt-only/secondary digest zero |
| Governance Feature | `docs/features/path-i-authorization-claim-governance/DESIGN.md` | `ActionDecisionV1`、`nas_replica_sync -> execution_pull_verify_materialize`、verified receipt prerequisite、commit recheck |
| CP4 result | `process/checks/CP4-CR172-PATH-I-STORY-DAG-PARALLEL-SAFETY.result.json` | DAG/coverage/owner PASS；六动作 authorized/executed=`0/6`,`0/6` |
| Story | `process/stories/STORY-CR172-S04-execution-cache-materialization.md` | AC、primary/forbidden files、S01/S03 dev gate、失败路由 |
| CP5 capsule | `process/context/CP5-CR172-PATH-I-LLD-CONTEXT.yaml` | claim ceiling、clarification queue=`0`、真实 NAS/执行机/runtime 全部未授权 |

## 1. Goal

创建 `engine/research_artifact_materialization.py` 及其 repository-local 测试，精确消费 S01 `ActionDecisionV1`/`ActionScopeContextV1`、S03 current `DistributionSelectionV1`/`ReplicaVerificationReceiptV1`/selected-replica read contract，以及 S02 `SealedTrialReturnBundleV1`/`ResearchCanonicalSelectionV1`/`VerifiedTrialReturnBundleV1`/`verify_sealed_trial_return_bundle` verifier-library，完成“fixture preflight → 只从 S03 selection 拉取 → typed non-runtime staging → S02 bytes-level verifier → release/manifest/seal/content `4/4` → immutable fixture cache → CAS local selection → receipt”的可实施合同。future runtime 只能消费本地 selected cache；bypass-S03/receipt-only-seal-trust/secondary-digest=`0/0/0`，tampered seal bytes pointer advance=`0`，真实 pull/执行/runtime=`0`。

## 2. 需求（Functional / Non-Functional Requirements）

### 2.1 Functional

- F-S04-01：在读取任何 replica fixture byte 或创建 execution staging 前调用 S01 `require_action_eligible(decision, expected_kind=execution_pull_verify_materialize, expected_context=action_context, expected_origin=repository_fixture)`，并精确校验 `ActionDecisionV1.target_kind == ActionScopeContextV1.target_kind == repository_fixture`；fixture+real accepted=`0`，不得借用 sync 或其他动作授权。
- F-S04-02：数据来源必须是 S03 current `DistributionSelectionV1` 唯一选定的 replica；S04 同时提交该 selection 与其 `ReplicaVerificationReceiptV1`，通过 S03 selected-replica read contract 取得对应 immutable `SealedTrialReturnBundleV1` + 原 `ResearchCanonicalSelectionV1`。receipt/selection/version/bundle refs 任一不一致即 BLOCKED；unselected、staging、path/latest 或 receipt-only source accepted=`0`。
- F-S04-03：只允许 `source_authority=nas_verified_replica` 与 `target_authority=execution_local_cache`；direct-research source、runtime-NAS source、shared-drive runtime 和“latest NAS file”全部拒绝。
- F-S04-04：authorized `MaterializationStoragePort.pull_to_staging(...)` 必须从 S03 selected-replica read contract 拉取并返回 typed tuple[`ExecutionStagingTokenV1`, `SealedTrialReturnBundleV1`, `ResearchCanonicalSelectionV1`]；staging 状态固定 `non_runtime`。pull interruption/partial 或返回类型/selection 漂移时 cache version/selection/receipt advance=`0/0/0`。
- F-S04-05：对 staging 返回的 typed bundle+selection 精确调用 S02 `verify_sealed_trial_return_bundle(bundle, selection) -> VerifiedTrialReturnBundleV1`，再计算 `expected release / manifest / seal / content` `4/4`；任一 mismatch 或 verifier integrity failure 均 BLOCKED，不依赖或导入 S02 私有异常类型。receipt 只作 predecessor/correlation，不替代 bytes；S04 不重新 seal、不调用 seal canonicalizer/digest helper。
- F-S04-06：通过 4/4 的 bytes 物化到 content-addressed immutable cache version；同 identity 已存在时只允许 hash 完全相同的幂等复用，覆盖/重写=`0`。
- F-S04-07：在 cache selection commit 前消费调用方提供的 fresh S01 decision；撤销、过期、context drift 或 replica selection drift 均使 pointer advance=`0`。
- F-S04-08：以 compare-and-swap 原子推进 `ExecutionCacheSelectionV1`，并生成 immutable `MaterializationReceiptV1`；CAS conflict/commit interruption 保留 previous cache selection。
- F-S04-09：输出给 future runtime 的 handle 必须标记 `source_kind=execution_local_immutable_cache` 且只引用 selected cache；模块不提供打开 NAS/research/staging 给 runtime 的 API。
- F-S04-10：稳定 identity 使用 logical URI + content hash；NAS/execution/research physical path、hostname、mount target 进入 canonical receipt/hash=`0`。
- F-S04-11：S04 不创建 seal、不修改 research/NAS pointer、不回写 S03 receipt、不实现 runtime consumer；legacy 不迁移、不重写、不成为第二默认。
- F-S04-12：当前可执行 adapter 只允许 `decision_origin=repository_fixture` + `target_kind=repository_fixture` + `fixture://` URI + repository-owned fixture/in-memory port；真实 NAS reader、execution host writer、network/mount/credential adapter 数量=`0`。
- F-S04-13：S02 依赖只允许 verifier-library；S04 不得用 S02 选择数据，也不得绕过 S03 current distribution selection。bypass-S03-selection/receipt-only-seal-trust/secondary-seal-digest=`0/0/0`；每个完整 staging candidate 的 S02 verifier call=`1`。

### 2.2 Non-Functional

- NFR-S04-01（完整性）：expected release/manifest/original seal/content 必须 `4/4`；seal bit 只可由 S02 verifier 对 staging seal bytes 成功验证后为 true。tampered seal bytes 的 seal/pointer=`false/0`；任一 mismatch 的 cache selection advance=`0`。
- NFR-S04-02（原子性）：失败路径 before/after selected receipt/content hashes 完全一致；每次 commit 最多推进一个 local pointer。
- NFR-S04-03（不可变性）：selected cache version 覆盖/修改/重新 seal=`0/0/0`；幂等复用必须复算 hash。
- NFR-S04-04（确定性）：相同 verified replica、expected release、4/4 vector 和 authorization evidence 产生相同 canonical materialization receipt hash；物理根变化不影响 identity/hash。
- NFR-S04-05（内存）：pull/hash 采用默认 `1 MiB` chunk，额外 payload 内存不超过 `2 MiB + bounded metadata`，禁止整文件 `read_bytes()`。
- NFR-S04-06（复杂度）：replica pull/hash 和 staging reverify 各 `O(n)`；cache pointer CAS=`O(1)`；不宣称真实 NAS/执行机 SLA。
- NFR-S04-07（安全）：authorization-before-read=`100%`；direct-NAS/direct-research runtime API/accepted=`0/0`；path traversal/symlink escape accepted=`0`；credential/network/backend read=`0`。
- NFR-S04-08（审计）：replica receipt、preflight/commit decisions、4/4 vector、cache version/previous-new selection 和 receipt refs coverage=`100%`；只存 refs，不存 credential。
- NFR-S04-09（边界）：real pull/materialize/runtime、研究机/NAS pointer mutation、信号/交易/publish/deploy/Git remote write=`0`；Stage3/C1/real-data claim 全不变。
- NFR-S04-10（依赖分离）：数据选择调用 S03=`1`，S02 数据选择调用=`0`；seal verifier truth count=`1`，S04 seal canonicalizer/digest implementation=`0/0`。

## 3. 模块拆分与职责

| 模块 / 文件组 | 职责 | 说明 |
|---|---|---|
| `engine/research_artifact_materialization.py` / input contract | 定义 request、expected replica、mapping、staging/candidate、receipt、selection/result/reason | 消费 S01 decision/context、S03 selected-replica/receipt 与 S02 verifier-library；不复制 authorization、replica selection 或 seal 真相 |
| `engine/research_artifact_materialization.py` / preflight guard | 验证 pull eligibility、verified distribution selection、receipt/expected release/context 与 source/target authority | 任何 NAS read/staging 前失败即零 port call |
| `engine/research_artifact_materialization.py` / materializer | 通过 `MaterializationStoragePort` 从 S03 current selection 拉取 typed sealed bundle+selection，到 non-runtime staging后调用 S02 verifier并执行 4/4，物化 immutable cache | production 文件不提供真实 NAS/执行机 adapter；不从 S02 选数 |
| `engine/research_artifact_materialization.py` / commit/runtime-boundary guard | fresh decision + replica drift recheck + CAS local selection；只生成 local selected handle | 不实现 future runtime，不返回 NAS/research/staging runtime handle |
| `tests/research/test_cr172_execution_materialization.py` | in-memory/`tmp_path` port、positive/mismatch/interruption/revoke/CAS/direct-read/path/zero-op 测试 | fixture action decision 不是实际授权；真实 host/path=`0` |

依赖方向固定为：governance=`S01 ActionDecisionV1 + ActionScopeContextV1 -> S04`；verifier-library=`S02 SealedTrialReturnBundleV1 + ResearchCanonicalSelectionV1 + VerifiedTrialReturnBundleV1 + verify_sealed_trial_return_bundle -> S04`；data selection=`S03 DistributionSelectionV1 + ReplicaVerificationReceiptV1 + read_selected_replica -> S04`。S02 不提供数据、S03 不提供 verifier facade；S04 不写上游、不导入 runner/lineage/estimator、不拥有 replica distribution pointer。

## 4. 代码结构与文件影响范围

| 动作 | 文件路径 | 变更内容 |
|---|---|---|
| 创建 | `engine/research_artifact_materialization.py` | typed materialization contracts、preflight/direct-read guard、streaming pull+4/4 verifier、immutable cache/CAS、receipt hash 与 local-only runtime handle |
| 创建 | `tests/research/test_cr172_execution_materialization.py` | valid 4/4、direct source deny、mismatch、interruption/revoke/CAS、previous cache/path identity/no-real-op 测试及 injected fixture port |

明确不修改：`engine/path_i_governance.py`、`engine/research_artifact_replica.py`、`engine/trial_return_artifact.py`、`engine/mature_multifactor_research.py`、`engine/experiment_family_lineage.py`、任何真实 NAS/执行机/runtime/部署配置。

## 5. 数据模型与持久化设计

| 对象 / 字段 | 类型 | 约束 | 说明 |
|---|---|---|---|
| `ExecutionMaterializationRequestV1.request_id` | `str` | non-empty、调用域内唯一 | 审计关联，不参与 artifact identity |
| `.expected_release_id` | `str` | 必填，与 S03 receipt/selection、manifest/seal 同值 | 不允许 latest/wildcard |
| `.expected_logical_uri` | `str` | 与 S03 selected artifact 相同 | stable identity |
| `.expected_content_sha256` / `.expected_manifest_sha256` | `str` | lowercase 64-hex | pull 前比 metadata，staging 后复验 bytes |
| `.expected_replica_receipt_sha256` | `str` | 必须指向当前 distribution selection | 阻止消费 unselected/stale receipt |
| `.action_context` | `ActionScopeContextV1`（S01 public type） | `target_kind=repository_fixture`；scope revision/hash、release/run/family 全匹配 | S04 不推断、继承或别名化 |
| `.source_authority` / `.target_authority` | enum | 仅 `nas_verified_replica` / `execution_local_cache` | direct research/runtime NAS/shared-drive 直接 BLOCKED |
| `ExecutionDeploymentMappingV1` | value object | 只接受与 `ActionScopeContextV1.target_kind=repository_fixture` 一致的 `fixture://` replica/staging/cache/version/pointer handles，并受 repository fixture root confinement | absolute paths 只属 deployment mapping，不进 identity/hash；real target accepted=`0` |
| `ExecutionStagingTokenV1` | opaque value | `state=non_runtime`，绑定 request、S03 distribution selection/receipt、expected hashes | 不能被 future runtime 选择；不承载或替代 sealed bundle |
| `MaterializationVerificationVectorV1` | four booleans | `release/manifest/seal/content` 必须全 true | `4/4` 才形成 verified candidate |
| `VerifiedCacheCandidateV1` | immutable candidate ref | staging token、S02 `VerifiedTrialReturnBundleV1`、S03 receipt/distribution refs、4/4、request/context | verified result 必须由 staging bundle+selection 调 S02 verifier得到；尚未 CAS 前不是 runtime-ready |
| `MaterializationReceiptV1` | immutable record | schema/version、release/logical URI/content/manifest/`original_seal_sha256`/replica-receipt hashes、`decision_origin`、`target_kind`、4/4 vector、preflight+commit evidence refs、cache version ref、`receipt_sha256` | digest/origin/target 必须等于输入 S03 receipt/selection；canonical body排除 self-hash、展示时间和所有 physical path；不含 credential |
| `ExecutionCacheSelectionV1` | immutable pointer value | release/logical URI/content hash/materialization receipt hash/cache version ref/revision/`decision_origin`/`target_kind`/`source_kind=execution_local_immutable_cache` | 当前 origin/target 固定 `repository_fixture/repository_fixture`；future runtime 只消费该 selected value |
| `ExecutionLocalCacheHandleV1` | host-local mapping result | selected identity + execution-local path handle | handle path 不参与 canonical identity/hash；不提供 NAS/research handle |
| `MaterializationResultV1` | tagged result | `MATERIALIZED` 或 `BLOCKED`；BLOCKED 必有 reason，receipt/selection/handle 为空 | pointer 状态必须显式可审计 |
| `MaterializationBlockReasonV1` | enum | authorization、receipt/selection stale、direct-source、partial、release/manifest/seal/content mismatch、immutable conflict、path escape、interrupted、CAS conflict | 未识别 reason fail-closed |

当前新增 persistence 仅是 repository-local fixture storage abstraction；无真实 execution directory、NAS mount、数据库/catalog。canonical receipt 使用版本化 `materialization-receipt@v1` hash domain、UTF-8 deterministic JSON、sorted keys/compact separators/NaN 禁止。cache version 由 content hash 寻址，存在时必须复验 bytes，绝不覆盖。

## 6. API / Interface 设计

| 接口 / 入口 | 输入 | 输出 | 调用方 | 说明 |
|---|---|---|---|---|
| `validate_materialization_preflight(request, replica_receipt: ReplicaVerificationReceiptV1, distribution_selection: DistributionSelectionV1, decision: ActionDecisionV1, action_context: ActionScopeContextV1)` | S04 request；S03 两个精确 public types；S01 两个精确 public types | `MaterializationPreflightV1` 或 BLOCKED result | future authorized execution materializer；当前 tests | first side effect 前校验 `repository_fixture/repository_fixture`、action/context/direct predecessor/source-target authority |
| `pull_and_verify_execution_staging(preflight, mapping, storage_port)` | verified preflight；repository-owned fixture mapping；port | `VerifiedCacheCandidateV1` 或 BLOCKED result | materialization orchestration | 调 port 获取 typed staging token+sealed bundle+selection；调用 S02 verifier exactly once；以 verified result + receipt/expected refs计算 4/4；无真实 NAS adapter |
| `commit_execution_cache(candidate, commit_decision: ActionDecisionV1, commit_context: ActionScopeContextV1, current_replica_selection: DistributionSelectionV1, storage_port)` | verified candidate；fresh S01 两个 public types；当前 S03 exact selection；port | `MaterializationResultV1` | materialization orchestration | 防 replica drift/revoke/origin-target mismatch；immutable version 后 CAS local selection；失败保留 previous cache |
| `canonical_materialization_receipt_bytes(receipt_body)` | 不含 self-hash/path 的 receipt body | canonical UTF-8 bytes | S04/tests | deterministic receipt hash；拒绝额外 mandatory 字段 |
| `resolve_execution_local_handle(selection, mapping)` | 已 selected `ExecutionCacheSelectionV1` + execution mapping | `ExecutionLocalCacheHandleV1` | future runtime adapter；当前 tests | 只接受 `source_kind=execution_local_immutable_cache`；没有打开/读取 runtime API |
| `MaterializationStoragePort.pull_to_staging(distribution_selection, replica_receipt, staging_token)` | S03 current `DistributionSelectionV1` + matching `ReplicaVerificationReceiptV1` + non-runtime token | tuple[`ExecutionStagingTokenV1`, `SealedTrialReturnBundleV1`, `ResearchCanonicalSelectionV1`] 或 typed failure | injected adapter | fixture adapter 必须委托 S03 `read_selected_replica`；禁止 path/latest/receipt-only/S02-data lookup；当前无真实 adapter |
| `MaterializationStoragePort.compare_and_swap_cache_selection(...)` | previous revision/hash + new local selection | committed bool/current ref | commit guard | 唯一 local pointer mutation 点；禁止 silent overwrite |

依赖公共名精确为：S01 `PathIActionKind`、`ActionScopeContextV1`、`ActionDecisionV1`、`require_action_eligible(...)`；S02 `SealedTrialReturnBundleV1`、`ResearchCanonicalSelectionV1`、`VerifiedTrialReturnBundleV1`、`verify_sealed_trial_return_bundle`；S03 `ReplicaVerificationReceiptV1`、`DistributionSelectionV1`、`ReplicaStoragePort.read_selected_replica(...)`。S04 不调用 S01 ledger/backend，不从 S02 选择数据，不重建/别名化/改写 S03 selection/receipt，也不 import/call `canonical_artifact_seal_bytes` 或 `canonical_artifact_seal_sha256`。全部接口只映射到 §10 的 16 个唯一测试 ID。

## 7. 核心处理流程

```mermaid
sequenceDiagram
  participant C as Caller
  participant G as S01 decision
  participant M as S04 materializer
  participant R as S03 selected replica
  participant P as MaterializationStoragePort
  participant V as S02 verifier library
  C->>M: request + ReplicaVerificationReceiptV1 + DistributionSelectionV1 + S01 decision/context
  M->>G: require repository_fixture origin/target + pull eligible
  alt auth/predecessor/source-target invalid
    M-->>C: BLOCKED; NAS read=0; staging=0; cache pointer unchanged
  else preflight PASS
    M->>P: pull_to_staging(current selection + receipt)
    P->>R: read_selected_replica(current DistributionSelectionV1)
    R-->>P: immutable bundle + source selection + receipt correlation
    P-->>M: non_runtime token + SealedTrialReturnBundleV1 + ResearchCanonicalSelectionV1
    M->>V: verify_sealed_trial_return_bundle(staged bundle, selection)
    V-->>M: VerifiedTrialReturnBundleV1 / integrity failure
    M->>M: compare expected + S03 receipt; compute release/manifest/seal/content 4/4
    alt interruption or mismatch
      M-->>C: BLOCKED; runtime handle=0; cache pointer unchanged
    else candidate verified
      C->>M: fresh commit decision + current replica selection
      M->>G: recheck eligibility/context/revoke and replica drift
      alt commit guard fails
        M-->>C: BLOCKED; previous cache retained
      else commit eligible
        M->>P: persist/reuse verified immutable cache + receipt
        M->>P: CAS local cache selection
        P-->>M: committed / conflict
        M-->>C: local selected handle or BLOCKED with previous retained
      end
    end
  end
```

处理步骤：

1. 拒绝 missing expected release/hash/receipt、`decision_origin!=repository_fixture`、`target_kind!=repository_fixture`、非 `fixture://` URI、非 repository-owned fixture/in-memory port，以及 direct research/NAS-runtime/shared-drive source 请求。
2. 调用 S01 `require_action_eligible(...expected_kind=execution_pull_verify_materialize, expected_context=action_context, expected_origin=repository_fixture)`；确认 predecessor 是当前 S03 `DistributionSelectionV1` 选中的 `ReplicaVerificationReceiptV1`，所有 context/ref 与 decision/context target 一致，fixture+real 在 first side effect 前 deny。
3. 仅在 preflight PASS 后调用 `pull_to_staging(current_distribution_selection, replica_receipt, token)`；其 fixture adapter 必须委托 S03 `read_selected_replica`，并返回 non-runtime token + typed sealed bundle + source selection。中断、返回类型错误或 refs drift 即 BLOCKED。
4. 对 staging 返回的 `SealedTrialReturnBundleV1`、`ResearchCanonicalSelectionV1` 调用 `verify_sealed_trial_return_bundle` exactly once；成功得到 `VerifiedTrialReturnBundleV1` 后，才与 expected/S03 receipt 比较并设置 release/manifest/seal/content 四 bit。tampered seal bytes 触发 verifier integrity failure，seal=false、pointer=0。
5. receipt validity 只作为前置/correlation，不能把 seal bit 置 true；S04 不调用 seal canonicalizer/digest、不提供 fallback verifier。bypass-S03/receipt-only/secondary-digest=`0/0/0`。
6. 生成 verified candidate；commit 前重新校验 fresh `ActionDecisionV1` + `ActionScopeContextV1` 和 current `DistributionSelectionV1`，阻断 revoke/expiry/context/selection/origin-target drift。
7. 把 4/4 bytes 写入或幂等复用 content-addressed immutable cache，构造 path-free receipt。
8. 用 expected previous cache revision/hash 做 CAS；成功后才生成 `ExecutionLocalCacheHandleV1`，source kind 固定 local immutable cache。
9. 任何失败返回 machine reason/previous selection；不自动删除 staging/immutable cache，不改 research/NAS pointer，不暴露 staging 给 runtime。

## 8. 技术细节（Technical Design Details）

- 关键算法 / 规则：流式 SHA-256 只用于 payload/cache/本 Story receipt，不用于 seal；`hmac.compare_digest`；版本化 deterministic JSON；content-addressed immutable cache；pointer CAS；root confinement；receipt/selection drift 双重检查。seal canonical bytes/digest 只在 S02 verifier 内部，S04 helper count=`0/0`。
- 4/4 口径：先对 staging typed bundle+selection调用 S02 verifier；`release` 比 verified bundle/selection、expected 与 S03 receipt；`manifest` 取 verifier 已复算通过的 manifest hash并与 expected/receipt比；`seal` 仅在 verifier 成功且 `VerifiedTrialReturnBundleV1.original_seal_sha256 == S03 receipt.original_seal_sha256` 时为 true；`content` 取 verifier 已复算通过的 payload hash并与 expected/receipt比。receipt validity 是前置/correlation，不替代任一 bytes-level bit。
- direct-read deny：公共 request enum 不包含 runtime-NAS/research 的允许值；validation 对未知或 forbidden source 返回 BLOCKED；public output 只有 execution-local handle，不输出可供 runtime 使用的 NAS/research/staging path。
- immutable/idempotent：cache version key=`logical_uri + content_sha256 + release_id` 的 deployment mapping；已存在时重新 4/4，完全相同才复用，任何差异 `IMMUTABLE_CACHE_CONFLICT`。
- authorization：preflight/commit 均消费 S01 `ActionDecisionV1` + `ActionScopeContextV1`；本模块不调用 backend。只有 `decision_origin=repository_fixture` + `target_kind=repository_fixture` + `fixture://` URI + repository-owned fixture/in-memory port 生效，不能转换为真实 authorization evidence。
- 路径与 identity：physical paths 只存在 mapping/handle；receipt/selection canonical allowlist 排除路径、hostname、mount。legacy 只读且不得成为 source fallback。
- 依赖选择与复用点：S01 只提供 action/context guard；S03 current distribution selection 是唯一数据来源；S02 只提供唯一 verifier-library。S04 只拥有 materialization receipt/local selection/handle，不拥有 seal parser/canonicalizer/digest。
- 图示类型选择：使用时序图，因为跨 Caller、S01 decision、S04 materializer、storage port，且中断、撤销、replica drift、CAS conflict 分支需要可视化。

## 9. 安全与性能设计

| 维度 | 设计措施 | 验证方式 |
|---|---|---|
| 授权最小化 | NAS read前和pointer commit前分别校验独立 pull decision；前置是 verified S03 receipt，不合并 sync 权限 | allow-without-predecessor、deny/expiry/revoke/context mismatch，NAS read/staging/pointer=`0/0/0` |
| runtime 边界 | 仅输出 selected execution-local handle；staging 标记 non-runtime；无 NAS/research runtime API | static API inventory + direct-source negative tests，accepted=`0/2` |
| 真实操作隔离 | production module 只有 port/orchestrator；target 只接受 repository fixture；无 network/mount/credential adapter | import/static scan；real pull/host path literal=`0` |
| 路径安全 | mapping resolve 必须限制在 execution fixture root；拒绝 traversal/symlink/absolute override | tmp_path path-escape matrix |
| 完整性/不可变性 | S03 selected typed bundle/selection → S02 verifier exactly once → 4/4；content-addressed cache 重验；original seal 不重建 | tampered seal bytes + mutation + immutable conflict tests；seal/pointer=`false/0` |
| 原子性 | immutable persist + expected-previous CAS；失败只留下 unselected bytes/receipt | interruption/CAS fault injection，previous cache 100% 保留 |
| 性能 | 1 MiB chunk；replica一次pull/hash、staging一次re-read/hash；无目录全扫描 | counting stream + bounded read assertions |
| 可审计性 | result带reason/receipt/decision/replica/previous-new refs；canonical body排除 path/credential | golden schema/hash tests |
| 依赖边界 | S03 是唯一 data-source，S02 只作 verifier-library；receipt-only、canonical helper import、secondary digest均禁止 | import/call/read-spy：bypass/receipt-only/secondary=`0/0/0`，verifier call=`1` |

## 10. 测试设计

| 测试场景 | 前置条件 | 操作 | 预期结果 | 验证方式 |
|---|---|---|---|---|
| T-S04-P01 正常 materialize | valid S03 `ReplicaVerificationReceiptV1`/current `DistributionSelectionV1`；eligible fixture decision/context；empty cache | S03 selected read→typed staging tuple→S02 verifier→4/4→cache→CAS | staged exact types=`3/3`；S02 verifier call=`1`；receipt/selection/local handle valid；pointer advance=`1` | unit + in-memory/`tmp_path` port |
| T-S04-P02 deterministic receipt | 同一 selected replica，不同 fixture roots | 各执行 3 次 | receipt/identity hash distinct count=`1`；path count=`0` | golden hash |
| T-S04-N01 独立授权缺失 | pull decision missing/deny/expired/revoked 或仅 sync approved | preflight | BLOCKED；replica read/staging/pointer=`0/0/0` | port call counters |
| T-S04-N02 predecessor/context mismatch | receipt 未被当前 `DistributionSelectionV1` 选中，或 scope/release/run/family 不同 | preflight | `INELIGIBLE_REPLICA_PREDECESSOR`；replica read=`0` | typed fixture |
| T-S04-N03 commit 前撤销/drift | staging 4/4 后 decision revoked 或 S03 selection 更新 | commit | local pointer advance=`0`；previous cache保留 | fault fixture |
| T-S04-N04 direct source deny | direct-NAS runtime、direct research、shared-drive runtime | validate request | accepted=`0/3`；runtime handle=`0` | enum/unknown-input tests |
| T-S04-N05 4/4 non-seal mismatch | expected release/manifest/content 任一篡改，或 verifier result 与 S03 receipt 不一致 | staging verify + 4/4 compare | cache version/receipt/pointer=`0/0/0`；精确 reason；不生成新 seal digest | 参数化 mutation |
| T-S04-N06 unselected/stale receipt | `ReplicaVerificationReceiptV1` 正确但 `DistributionSelectionV1` 指向其他 receipt | preflight | BLOCKED；不读取 replica | before-read guard |
| T-S04-N07 fixture origin/real target | `decision_origin=repository_fixture` + `target_kind=real_operation` | preflight | BLOCKED；fixture+real accepted=`0`；port calls=`0` | typed origin/target matrix |
| T-S04-N08 non-fixture port/path escape | 非 `fixture://` URI、非 repository-owned port、`..`、symlink escape 或 absolute override | preflight/pull | BLOCKED；file/network operation=`0` | static + `tmp_path` negative |
| T-S04-F01 pull 中断/partial | port 在第 k 个 chunk 中断 | pull | staging non-runtime；cache/receipt/pointer=`0/0/0` | fault-injection port |
| T-S04-F02 immutable/CAS conflict | 同 cache key 已存在不同 bytes，或 verify 后另一 writer推进 cache pointer | materialize/commit | `IMMUTABLE_CACHE_CONFLICT`/`POINTER_CONFLICT`；existing bytes/current pointer 不变 | immutable store + deterministic CAS fake |
| T-S04-F03 tampered seal bytes | S03 current selection 正常，但 `pull_to_staging` 返回的 sealed bundle 中 seal bytes 被篡改 | 调 S02 verifier | verifier integrity failure fail-closed；tampered seal bytes→seal=`false`/pointer=`0`；cache/receipt=`0/0`；secondary digest=`0` | byte-level mutation + verifier-call spy |
| T-S04-B01 path identity | 同 content 映射不同 NAS/execution roots | canonicalize receipt | identity/hash distinct count=`1`；absolute path=`0` | structural assertion |
| T-S04-B02 runtime handle boundary | valid selected cache及staging/replica/research refs | resolve handles | local selected handle=`1`；NAS/research/staging runtime handle=`0/0/0` | API inventory/assertion |
| T-S04-B03 recovery/dependency/no-real-op boundary | 已有 previous cache；执行全失败矩阵并扫描 imports/calls/read ports | 失败后对比 selection/bytes/公共 API | previous cache preservation=`100%`；bypass-S03/receipt-only/canonical-helper import/secondary digest=`0/0/0/0`；real pull/runtime=`0` | before/after equality + static/read-spy guard |

## 11. 实施步骤

| TASK-ID | 动作 | 目标文件 | 详细描述 | 对应测试 |
|---|---|---|---|---|
| CR172-S04-T01 | 创建 | `process/stories/STORY-CR172-S04-execution-cache-materialization-LLD.md` | 冻结 4/4、direct-source deny、port、双授权、immutable cache/CAS、local-only handle 和恢复合同 | CP5 `lld-check` |
| CR172-S04-T02 | 创建 | `engine/research_artifact_materialization.py` | 实现 exact S01 guard、S03 selected data source、S02 verifier-library、typed staging tuple、bytes-level 4/4、fixture guard、immutable cache/CAS/local handle；无 second digest | §10 positive/negative/boundary cases |
| CR172-S04-T03 | 创建 | `tests/research/test_cr172_execution_materialization.py` | 实现 in-memory/`tmp_path` ports、tampered seal bytes、verifier-call spy、bypass/receipt-only/secondary-digest static guard及恢复 fixtures；测试 ID 唯一=`16/16` | §10 全部 16 个 case |
| CR172-S04-T04 | 验证 | `engine/research_artifact_materialization.py`、`tests/research/test_cr172_execution_materialization.py` | 运行 scoped tests/static guard，证明 previous cache、local-only runtime edge、fixture origin/target 与 no-real-op | §10 authorization/failure/boundary cases |

实施顺序固定为 T01→T02→T03→T04；CP5 未批准时 T02～T04 均不得开始。dev gate 还要求 S01 contract、S02 verifier-library frozen+merged，S03 selected-replica implementation evidence PASS，且文件 owner 无冲突。

## 12. 风险、难点与预研建议

### 12.1 实现灰区与取舍记录

| Clarification ID | 问题 | 选项与推荐 | 决策 / 答案 | 影响面 | 证据 | 重访条件 |
|---|---|---|---|---|---|---|
| 无 | 当前没有需用户决策的实现灰区 | 推荐：S03 current selection 唯一供数 + S02 verifier-library 复验；备选只允许 distribution blocked | 采用推荐；bypass/receipt-only/secondary digest=`0/0/0`；clarification queue=`0` | 数据来源、完整性、owner、测试、跨 Story 契约 | HLD/ADR v1.4、Feature I01/I02/I03 v1.2、R2 F-002、CP5 context | 若提出绕过 S03、receipt-only trust、S03 verifier facade、S04 digest/parser、direct-NAS/runtime 或真实 adapter，则返回 `NEEDS_DESIGN_CLARIFICATION`/独立 CR |

| 风险 / 难点 | 影响 | 缓解措施 / 预研建议 |
|---|---|---|
| pull 成功被误当 runtime-ready | runtime 读取未验证/partial bytes | staging 强制 non-runtime；只有 CAS selected local handle 可输出 |
| replica selection 在 pull 期间漂移 | cache 与 approved distribution 不一致 | preflight + commit 两次 selection/receipt hash；drift 即 BLOCKED |
| direct-NAS shortcut | NAS 变工作盘，故障/新鲜度边界丢失 | API/enum/static guard 均无 direct runtime edge；local-only handle |
| immutable cache key 冲突 | 覆盖审计版本 | content-addressed key存在时复验；不同 bytes 永不覆盖 |
| mid-operation revoke | 未授权 cache pointer commit | commit 前 fresh decision；partial staging non-runtime |
| path 进入 identity | 跨主机 hash漂移/双真相 | receipt allowlist 与 mapping/handle 分离；跨 root golden test |
| cleanup/重试越权 | 删除 immutable evidence或复用过期授权 | 当前不自动 cleanup；重试重新授权并完整 4/4 |
| streaming port 整文件读取 | 大 artifact 内存失控 | iterator/chunk contract + counting stream；禁止 `.read_bytes()` |
| receipt metadata 冒充 seal bytes 复验 | seal bit 假 PASS | port 必须返回 typed sealed bundle+selection；S02 verifier成功后才可置 seal=true；tampered seal bytes pointer=0 |
| S02 verifier dependency 被误作数据旁路 | S04 绕过 S03 selection读取未选对象 | S02仅 library import；所有 bundle/selection由 S03 read-spy提供；bypass static count=0 |

### OPEN / Spike 跟踪

| ID | 类型（OPEN / Spike） | 问题 | 下一动作 | 责任方 |
|---|---|---|---|---|
| 无 | N/A | 未决 OPEN/Spike=`0` | 无 | 无 |

## 13. 回滚与发布策略

- 发布方式：CP5 批准后按 Wave 4 交付 repository-local Python contract 与 tests；不部署真实 pull/materialization adapter，不启用 execution runtime，不切换任何设备 pointer。
- 发布前门控：S01 governance、S02 verifier-library、S03 selected-replica dev gate 已满足；S04 scoped tests/static/read-spy guard PASS；六动作 authorized/executed 仍=`0/6`,`0/6`。
- 回滚触发条件：S01/S02/S03 contract mismatch、任一 4/4 误接受、tampered seal bytes 被接受、bypass/receipt-only/secondary digest>0、direct source API 出现、path 进入 identity、immutable version 被覆盖、CAS 非原子或失败改变 previous cache。
- 回滚动作：回退 S04 新增模块/测试独立切片；真实环境无目录/缓存/pointer/runtime 需要回滚。fixture 仅把 local selection CAS 回 previous receipt ref，绝不修改 immutable bytes。
- 不采用的降级：direct NAS/research runtime、shared-drive working cache、重新 seal、absolute-path identity、自动 cleanup、legacy migration 均禁止。

## 14. DoD（Definition of Done）

- [x] 0～14 节全部填写，frontmatter `lld_version=1.3`、`status=confirmed`、`confirmed=true`、`tier=L`、`open_items=0`。
- [ ] F-S04-01～13 与 NFR-S04-01～10 均有接口、失败行为和测试；测试表 ID 唯一=`16/16`、duplicate=`0`。
- [ ] expected release/manifest/original seal/content verification=`4/4`；staging verifier call=`1` per complete candidate；tampered seal bytes 的 seal/cache pointer=`false/0`；任一 mismatch cache pointer advance=`0`。
- [ ] direct-NAS/direct-research runtime accepted=`0/2`，public runtime handle 只可能来自 selected execution-local immutable cache。
- [ ] preflight/commit 两次独立 pull decision；sync authorization 不可继承；permission union=`0`。
- [ ] interruption/revoke/replica drift/immutable conflict/CAS failure下 previous cache preservation=`100%`。
- [ ] materialization receipt/hash 排除 physical path/credential；digest/origin/target 与 S03 receipt/selection 同值覆盖=`3/3`；S04 seal/research/replica pointer mutation=`0/0/0`。
- [ ] S03 selection 是唯一 data-source，S02 仅 verifier-library；typed staging output=`3/3`；bypass-S03/receipt-only-trust/canonical-helper import/secondary digest=`0/0/0/0`。
- [ ] 仅创建 S04 的两个 primary implementation/test 文件；S01/S03/runner/lineage forbidden modification=`0`。
- [ ] `decision_origin=repository_fixture` + `target_kind=repository_fixture` + `fixture://` + repository-owned port 是唯一当前 adapter；fixture+real accepted=`0`；real pull/execution/runtime 与六动作 authorized/executed=`0/6`,`0/6`。
- [ ] clarification、OPEN、Spike=`0/0/0`；边界变化按第 12 节返回设计澄清。
- [ ] `meta-flow story lld-check` 与 targeted `git diff --check` PASS。
- [ ] CP5 全量人工确认前不实现；S01/S03 dev gate 未满足时不进入 Wave 4。

## 人工确认区

> **CP5 — Story 设计证据可实现性门**。本文件只是五份 full LLD 之一，必须与 CP4 摘要及其余 Story 设计证据统一确认；单独审阅不解锁实现。

**CP5 checklist 摘要**：

| # | 检查项 | 状态 | 证据 |
|---|---|---|---|
| 1 | LLD 覆盖 AC | 待检查 | §2、§10、§14 |
| 2 | 与 HLD / ADR 一致 | 待检查 | §0、§3、§7、§8、§12 |
| 3 | 文件影响范围明确 | 待检查 | §4、§11 |
| 4 | 接口契约完整 | 待检查 | §5、§6 |
| 5 | 测试与 dev_gate 可计算 | 待检查 | §10、§11、§14 |
| 6 | clarification queue 已收敛 | 待检查 | §12.1；queue=`0` |

**人工确认回复**：

```text
approve
修改: <具体修改点>
reject
```

**人工审查结果回填**：

- 结论：`approved | changes_requested | rejected`
- 审查人：
- 审查时间：
- 修改意见：
- 风险接受项：
