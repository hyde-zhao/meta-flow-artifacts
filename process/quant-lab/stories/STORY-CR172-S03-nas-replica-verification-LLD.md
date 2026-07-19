---
story_id: "CR172-S03-nas-replica-verification"
title: "NAS sealed-artifact replica verification and distribution pointer"
story_slug: "nas-replica-verification"
lld_version: "1.3"
tier: "L"
status: "confirmed"
confirmed: true
created_by: "meta-dev"
created_at: "2026-07-18T11:05:00+08:00"
confirmed_by: "user"
confirmed_at: "2026-07-18T13:49:27+08:00"
implementation_allowed: true
shared_fragments: []
feature_design_refs:
  - "docs/features/research-artifact-replica-materialization/DESIGN.md"
  - "docs/features/research-artifact-replica-materialization/TEST-PLAN.md"
  - "docs/features/research-artifact-replica-materialization/TASKS.md"
  - "docs/features/path-i-authorization-claim-governance/DESIGN.md"
  - "docs/features/path-i-authorization-claim-governance/TEST-PLAN.md"
  - "docs/features/path-i-authorization-claim-governance/TASKS.md"
lld_policy:
  required_level: "full-lld"
  trigger_reasons:
    - "NAS boundary"
    - "immutable replica receipt"
    - "atomic pointer and recovery"
    - "independent sync authorization"
  rationale: "跨机副本、staging、原 seal 复验、freshness/hash/release 校验及 pointer rollback 属于高风险持久化合同。"
open_items: 0
---

# LLD: CR172-S03 — NAS sealed-artifact replica verification and distribution pointer

> 本文仅设计 repository-local contract、注入式 storage port 和 fixture 验证面。它不提供真实 NAS adapter，不创建或访问真实 NAS 路径，不批准或执行 `nas_replica_sync`。

### 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-18 | meta-dev | 初始 S03 full LLD。 |
| 1.1 | 2026-07-18 | meta-dev | CP5 R2：精确绑定 S01/S02 public types 与 `verify_sealed_trial_return_bundle`；删除旧的非 public context 别名；固化 repository fixture origin/target；`original_seal_sha256` 只透传 `VerifiedTrialReturnBundleV1`，S03 secondary seal canonicalizer/digest=`0/0`。 |
| 1.2 | 2026-07-18 | meta-dev | CP5 R3：冻结 S03 selected-replica read contract；只按当前 `DistributionSelectionV1` 返回 immutable `SealedTrialReturnBundleV1`、原 `ResearchCanonicalSelectionV1` 和 `ReplicaVerificationReceiptV1`，供 S04 staging 使用；新增 digest/verifier facade/data bypass=`0/0/0`。 |
| 1.3 | 2026-07-18 | host-orchestrator | CP5 批准前 optional 整改：authority pointer-only 刷新至 HLD/ADR v1.4；normative delta=`0`。 |

## 0. 工程依据（上游设计依据）

| 来源 | 路径 / ID | 被本 LLD 消费的内容 |
|---|---|---|
| HLD v1.4 | `docs/design/HLD-TRIAL-RETURN-DEPLOYMENT-CONTRACTS.md` §9.1～§12 | repository fixture sealed bundle、NAS replica 边界、selected-replica read contract、唯一 seal verifier/digest、FLOW-I02、pointer-only rollback |
| ADR v1.4 | `docs/design/ARCHITECTURE-DECISION-TRIAL-RETURN-DEPLOYMENT-CONTRACTS.md` / ADR-CR172-I-002～005、009、011 | path 不进 identity、original seal 复用、fixture/real typed binding、S04 verifier-library dependency、失败保留上一 selection |
| Feature Matrix v1.30 | `docs/design/FEATURE-DESIGN-MATRIX.md` / CR172 PATH-I CP4 增量 | S03=`full-lld`、Wave 3、owner、S01/S02 前置、S03→S04 runtime-selection edge、禁止范围 |
| Feature DESIGN v1.2 | `docs/features/research-artifact-replica-materialization/DESIGN.md` | `ReplicaVerificationReceiptV1`、selected replica sealed-bundle/selection read contract、NAS staging→verify→receipt→atomic pointer |
| Feature TEST-PLAN v1.2 | `docs/features/research-artifact-replica-materialization/TEST-PLAN.md` | I02-P01/N01/F01/F03、路径身份边界、previous selection preservation、S04 bytes-level seal negative |
| Governance Feature | `docs/features/path-i-authorization-claim-governance/DESIGN.md` | `ActionDecisionV1`、`trial_return_generation -> nas_replica_sync`、同 context/provenance、mid-operation recheck |
| CP4 result | `process/checks/CP4-CR172-PATH-I-STORY-DAG-PARALLEL-SAFETY.result.json` | Story/DAG/owner/coverage PASS；真实动作 authorized/executed=`0/6`,`0/6` |
| Story | `process/stories/STORY-CR172-S03-nas-replica-verification.md` | AC、文件 owner、失败路由、S01/S02 dev gate |
| CP5 capsule | `process/context/CP5-CR172-PATH-I-LLD-CONTEXT.yaml` | claim ceiling、allowed reads、clarification queue=`0`、non-authorized scope |

## 1. Goal

创建 `engine/research_artifact_replica.py` 及其 repository-local 测试，精确消费 S02 `SealedTrialReturnBundleV1`、`ResearchCanonicalSelectionV1`、`VerifiedTrialReturnBundleV1`、`verify_sealed_trial_return_bundle` 与 S01 `ActionDecisionV1`、`ActionScopeContextV1`，完成“preflight 校验 → non-distributable fixture staging → 原 seal 复验 → immutable receipt → CAS 原子 fixture distribution selection”的可实施合同；CAS 后只允许按当前 `DistributionSelectionV1` 向 S04 返回对应的 immutable sealed bundle、原 source selection 和 receipt。任何授权、origin/target、完整性、freshness、copy、read-selection 或 commit 失败都保持 previous verified selection，且研究机 canonical mutation=`0`、新增 seal digest/verifier facade/data bypass=`0/0/0`、真实 NAS 操作=`0`。

## 2. 需求（Functional / Non-Functional Requirements）

### 2.1 Functional

- F-S03-01：在任何 staging 写入前调用 S01 `require_action_eligible(decision, expected_kind=nas_replica_sync, expected_context=action_context, expected_origin=repository_fixture)`，并精确校验 `ActionDecisionV1.target_kind == ActionScopeContextV1.target_kind == repository_fixture`、scope/context 一致和历史 sealed generation provenance；fixture decision + real target 接受=`0`，六类授权不得合并。
- F-S03-02：在 copy 前将 `SealedTrialReturnBundleV1` 与 `ResearchCanonicalSelectionV1` 原样交给 S02 `verify_sealed_trial_return_bundle(bundle, selection) -> VerifiedTrialReturnBundleV1`；只有成功返回才继续，并比较 expected `release_id/logical_uri/content_sha256/manifest_sha256` 以及 selection 的 `decision_origin/target_kind=repository_fixture/repository_fixture`。wrong-kind、unsealed、unversioned 或 verifier failure 均 fail-closed，不接受 partial result。
- F-S03-03：把通过 preflight 的 bundle 写入由注入式 port 管理的唯一 staging token；staging 状态固定为 `non_distributable`，不得作为 distribution selection。
- F-S03-04：对 port 返回的 staged `SealedTrialReturnBundleV1` 与原始 `ResearchCanonicalSelectionV1` 再次调用同一 S02 verifier，并完成 `original seal / manifest / content / release / freshness` 五项验证；`5/5` 才生成 immutable `ReplicaVerificationReceiptV1`，其 `original_seal_sha256` 必须直接取 staged `VerifiedTrialReturnBundleV1.original_seal_sha256`。
- F-S03-05：freshness 由 approved `expected_release_id + expected_source_selection_sha256` 的确定性相等判定，不读取 mtime、不选择“最新目录”、不扫描通配符。
- F-S03-06：在 pointer commit 前消费调用方提供的 fresh S01 decision，再校验 action/context；撤销、过期或 context drift 时 pointer advance=`0`。
- F-S03-07：以 compare-and-swap 原子推进 `DistributionSelectionV1`；CAS 失败、进程中断或 receipt 持久化失败保留 previous selection=`100%`。
- F-S03-08：identity 只由 logical URI、content/manifest/seal/receipt hashes 与 release 组成；research/NAS absolute path、hostname、mount point 进入 identity/hash=`0`。
- F-S03-09：S03 只验证 S02 原 seal，不重新 seal，不改 research pointer，不删除 immutable source/receipt/version；partial cleanup 需未来独立授权，本 Story 不实现。
- F-S03-10：当前可执行 adapter 只允许 `decision_origin=repository_fixture` + `target_kind=repository_fixture` + `fixture://` logical URI + repository-owned fixture/in-memory port；真实 NAS adapter、mount、rsync、network client 和环境凭据读取数量=`0`。
- F-S03-11：S04 只能以当前 `DistributionSelectionV1` 调用 S03 selected-replica read contract；返回值精确为对应 immutable `SealedTrialReturnBundleV1`、原 `ResearchCanonicalSelectionV1`、`ReplicaVerificationReceiptV1`。selection/receipt/version 任一不一致即 BLOCKED；unselected/staging/absolute-path lookup accepted=`0`，不提供 verifier facade或新 digest。

### 2.2 Non-Functional

- NFR-S03-01（完整性）：五项 verification bit 必须 `5/5`；partial/stale/unversioned/hash/manifest/seal/release mismatch 的 accepted/pointer advance=`0/0`。
- NFR-S03-02（原子性）：每次 publish 最多改变一个 distribution pointer；失败前后 selected receipt hash 必须完全相同。
- NFR-S03-03（确定性）：相同 source bytes、manifest、seal、expected selection 和 authorization evidence 产生同一 canonical receipt hash；物理路径或遍历顺序变化不改变 hash。original seal 的 canonicalizer/digest 真相源只在 S02，S03 secondary seal canonicalizer/digest=`0/0`。
- NFR-S03-04（内存）：payload copy/hash 采用流式 chunk，默认 chunk size=`1 MiB`，额外 payload 内存不超过 `2 MiB + bounded metadata`；禁止整文件 `read_bytes()`。
- NFR-S03-05（复杂度）：source copy 和 staging reverify 各为 `O(n)`；pointer CAS 为 `O(1)` metadata operation；不声明真实 NAS SLA。
- NFR-S03-06（安全）：path traversal/symlink escape 接受=`0`，credential/env/network/backend read=`0`；absolute path 不写入 receipt canonical hash domain。
- NFR-S03-07（审计）：preflight/commit authorization evidence、source selection、5/5 vector、receipt hash 和 pointer previous/new refs 覆盖=`100%`；只存 refs，不存凭据。
- NFR-S03-08（边界）：真实 lake/NAS/runtime/R/signal/trading/publish/deploy/Git remote 操作=`0`；本 Story 不能提升任何 Stage 3/C1 claim。
- NFR-S03-09（单一数据选择）：S04 可读取的 replica 数据版本必须由 S03 当前 distribution selection 唯一决定；同一 selection 返回 bundle/selection/receipt tuple distinct count=`1`，绕过 selection 的 public read entry=`0`。

## 3. 模块拆分与职责

| 模块 / 文件组 | 职责 | 说明 |
|---|---|---|
| `engine/research_artifact_replica.py` / canonical contract | 定义 request、expectation、mapping、candidate、receipt、selection、result/reason code；提供 deterministic canonicalization/hash | 不复制 S01/S02 领域模型；只引用其公开 decision、manifest、seal 与 source selection |
| `engine/research_artifact_replica.py` / preflight verifier | 判定 action eligibility、sealed source 与 expected release/source-selection freshness | 失败发生在任何 storage port write 前 |
| `engine/research_artifact_replica.py` / replica orchestrator | 通过 `ReplicaStoragePort` 写 staging、流式重验、构造 receipt、请求 CAS commit | port 是注入边界；production 文件中不实现 NAS/mount/network adapter |
| `engine/research_artifact_replica.py` / commit guard | 重新消费 fresh decision，校验 receipt/candidate/current selection，执行 CAS | mid-operation revoke/expiry/context drift 均拒绝 commit |
| `engine/research_artifact_replica.py` / selected-replica reader | 以当前 `DistributionSelectionV1` 为唯一键，从 port 返回已选 immutable bundle、原 research selection 和 receipt | 不按 path/latest/receipt-only 选数，不验证或重算 seal；S04 再调用 S02 verifier-library |
| `tests/research/test_cr172_nas_replica_verification.py` | 提供内存/`tmp_path` fixture port、正负向/中断/CAS/边界测试 | fixture allow 决策不构成真实授权；不出现真实 NAS host/path |

依赖方向固定为 `S01 ActionDecisionV1 + ActionScopeContextV1 + S02 SealedTrialReturnBundleV1 + ResearchCanonicalSelectionV1 + VerifiedTrialReturnBundleV1 + verify_sealed_trial_return_bundle -> S03`。S03 不反向写 S01/S02，不导入 S04，不修改 runner 或 lineage store。

## 4. 代码结构与文件影响范围

| 动作 | 文件路径 | 变更内容 |
|---|---|---|
| 创建 | `engine/research_artifact_replica.py` | typed contracts、canonical receipt hashing、preflight/staging verification、fresh authorization commit guard、CAS publish orchestration 与 fixture-only target guard |
| 创建 | `tests/research/test_cr172_nas_replica_verification.py` | 5/5 positive、七类 mismatch、interruption/revoke/CAS、previous selection、identity/path/no-real-op 测试及内存/`tmp_path` port |

明确不修改：`engine/path_i_governance.py`、`engine/trial_return_artifact.py`、`engine/mature_multifactor_research.py`、`engine/research_artifact_materialization.py`、`engine/experiment_family_lineage.py`、任何真实 NAS/部署配置。

## 5. 数据模型与持久化设计

| 对象 / 字段 | 类型 | 约束 | 说明 |
|---|---|---|---|
| `ReplicaSyncRequestV1.request_id` | `str` | non-empty、调用域内唯一 | 审计关联，不进入 artifact identity |
| `.expected_release_id` | `str` | 必填、与 seal/manifest/source selection 相同 | 禁止“latest”或 wildcard |
| `.expected_logical_uri` | `str` | 必须匹配 research scheme 与 S02 selection | 稳定身份组成 |
| `.expected_content_sha256` / `.expected_manifest_sha256` | `str` | lowercase 64-hex | copy 前与 staging 后均复验 |
| `.expected_source_selection_sha256` | `str` | lowercase 64-hex | freshness 的确定性锚；不使用 mtime |
| `.action_context` | `ActionScopeContextV1`（S01 public type） | `target_kind=repository_fixture`；`scope_revision/scope_sha256/release_id/run_id/family_id` 全匹配 | S03 不复制、别名化或放宽 S01 上下文规则 |
| `ReplicaDeploymentMappingV1` | value object | 只接受与 `ActionScopeContextV1.target_kind=repository_fixture` 一致的 `fixture://` logical URI 和 repository-owned fixture/in-memory source/staging/version/pointer handles | absolute paths 只属 deployment mapping，不进 canonical receipt/hash；real target 绑定 accepted=`0` |
| `ReplicaStagingTokenV1` | opaque value | `state=non_distributable`；绑定 request/context/expected hashes | 不能被 runtime/distribution consumer 读取 |
| `ReplicaVerificationVectorV1` | five booleans | `seal/manifest/content/release/freshness` 必须全 true | 任何 false 返回 BLOCKED，不生成 receipt |
| `ReplicaVerificationReceiptV1` | immutable record | schema/version、release、logical URI、content/manifest/source-selection hashes、`original_seal_sha256`、`decision_origin`、`target_kind`、verification vector、preflight+commit evidence refs、replica version ref、`receipt_sha256` | seal digest、origin、target 必须逐值复制 staged `VerifiedTrialReturnBundleV1` 的 digest/selection；canonical body 排除 self-hash、时间展示值和物理路径；不含 credential |
| `DistributionSelectionV1` | immutable pointer value | release/logical URI/content hash/receipt hash/replica version ref/source-selection hash/selection revision/`decision_origin`/`target_kind` | 唯一选定一个 immutable replica version、对应 S02 source selection 与 verified receipt；当前值固定 `repository_fixture/repository_fixture`；不拥有 canonical authority |
| `ReplicaPublishResultV1` | tagged result | `VERIFIED` 或 `BLOCKED`；BLOCKED 必有 machine reason，receipt/selection 均为空 | 禁止用异常吞掉 pointer 状态 |
| `ReplicaBlockReasonV1` | enum | 至少覆盖 authorization、wrong-kind、unsealed、partial、stale-selection、release/logical/hash/manifest/seal mismatch、port escape、interrupted、CAS conflict | reason 集合领域经验可扩展，未知值 fail-closed |

当前 Story 新增的是 repository-local fixture 持久化抽象，不新增真实数据库、NAS schema、catalog 或运行目录。`ReplicaStoragePort` 的 fixture 实现可在 `tmp_path` 保存 staging/version/receipt/pointer；production 模块不持有真实 adapter。replica receipt 自身的 canonical JSON 使用 UTF-8、sorted keys、compact separators、禁止 NaN，hash domain 带显式 `replica-receipt@v1` 前缀；它不构成第二套 **seal** canonicalizer/digest，S03 seal canonicalizer/digest 实现数量固定为 `0/0`。

## 6. API / Interface 设计

| 接口 / 入口 | 输入 | 输出 | 调用方 | 说明 |
|---|---|---|---|---|
| `validate_replica_preflight(request, source_bundle: SealedTrialReturnBundleV1, source_selection: ResearchCanonicalSelectionV1, decision: ActionDecisionV1, action_context: ActionScopeContextV1)` | S03 request；S02 两个精确 public types；S01 两个精确 public types | `ReplicaPreflightV1`（持有 `VerifiedTrialReturnBundleV1`）或 BLOCKED result | future authorized replica adapter；当前 tests | first side effect 前先校验 `repository_fixture/repository_fixture` 绑定，再调用 `verify_sealed_trial_return_bundle(source_bundle, source_selection)`；不 reopen path、不重新 seal |
| `stage_and_verify_replica(preflight, mapping, storage_port)` | verified preflight；repository-owned fixture mapping；`ReplicaStoragePort` | `VerifiedReplicaCandidateV1`（持有 staged `VerifiedTrialReturnBundleV1`）或 BLOCKED result | replica orchestration | port 必须流式 copy 到 non-distributable staged bundle，再以原 `ResearchCanonicalSelectionV1` 调用同一 S02 verifier；当前无真实 NAS port |
| `commit_verified_replica(candidate, commit_decision: ActionDecisionV1, commit_context: ActionScopeContextV1, storage_port)` | verified candidate；重新判定的 S01 两个 public types；port 当前 selection | `ReplicaPublishResultV1` | replica orchestration | fresh decision + same fixture context + staging reverify 后写 receipt/version并 CAS；receipt seal digest 只能复制 candidate verified result；失败不改 pointer |
| `canonical_replica_receipt_bytes(receipt_body)` | 不含 self-hash/物理路径的 receipt body | canonical UTF-8 bytes | S03 verifier/tests | deterministic hash domain；拒绝额外/未知 mandatory 字段 |
| `ReplicaStoragePort.stage_bundle(...)` | verified source bundle + opaque staging token | staged `SealedTrialReturnBundleV1` 或 typed failure | injected adapter | 不创建第二个 research selection；中断时保持 non-distributable；本 Story 仅 repository-owned fixture/in-memory test double |
| `ReplicaStoragePort.compare_and_swap_selection(expected_previous, new_selection)` | previous selection hash/revision + verified selection | committed bool/current ref | commit guard | 唯一 pointer mutation 点；不得 silent overwrite |
| `ReplicaStoragePort.read_selected_replica(distribution_selection)` | 调用时读取的当前 `DistributionSelectionV1` | tuple[`SealedTrialReturnBundleV1`, `ResearchCanonicalSelectionV1`, `ReplicaVerificationReceiptV1`] 或 typed BLOCKED | S04 `MaterializationStoragePort.pull_to_staging` fixture adapter | 必须逐项匹配 selection 的 replica-version/receipt/source-selection refs；只返回 immutable typed values，不返回 path/裸 bytes，不调用 verifier，不新增 facade/digest |

S03 精确消费且不得别名化的 public contract 为：S01 `ActionDecisionV1`、`ActionScopeContextV1`；S02 `SealedTrialReturnBundleV1`、`ResearchCanonicalSelectionV1`、`VerifiedTrialReturnBundleV1`、`verify_sealed_trial_return_bundle`。S03 不导入或调用 `canonical_artifact_seal_bytes`/`canonical_artifact_seal_sha256`，因此 secondary seal canonicalizer/digest=`0/0`。向 S04 只增加 selection-bound read 方法，不增加 verifier facade、digest helper或第二个数据选择入口。

## 7. 核心处理流程

```mermaid
sequenceDiagram
  participant C as Caller
  participant G as S01 decision
  participant R as S03 verifier
  participant P as ReplicaStoragePort
  participant M as S04 materializer
  C->>R: request + SealedTrialReturnBundleV1 + ResearchCanonicalSelectionV1 + S01 decision/context
  R->>G: require repository_fixture origin/target (no backend call)
  alt unauthorized, ineligible, source/expected mismatch
    R-->>C: BLOCKED; staging=0; pointer unchanged
  else preflight PASS
    R->>R: verify_sealed_trial_return_bundle -> VerifiedTrialReturnBundleV1
    R->>P: stage_bundle(non_distributable)
    P-->>R: staged SealedTrialReturnBundleV1 or interrupted
    R->>R: same S02 verifier(staged bundle, original selection) + verify 5/5
    alt interruption or any mismatch
      R-->>C: BLOCKED; receipt=0; pointer unchanged
    else candidate verified
      C->>R: fresh commit ActionDecisionV1
      R->>G: recheck action/context/expiry/revoke
      alt commit decision invalid
        R-->>C: BLOCKED; pointer unchanged
      else commit eligible
        R->>P: persist immutable version + receipt(digest from Verified result)
        R->>P: CAS distribution selection
        P-->>R: committed / conflict
        R-->>C: VERIFIED selection or BLOCKED with previous retained
        M->>P: read_selected_replica(current DistributionSelectionV1)
        P-->>M: immutable bundle + source selection + receipt
      end
    end
  end
```

处理步骤：

1. 解析 request，拒绝缺失 expected release/hash/source-selection、`decision_origin!=repository_fixture`、`target_kind!=repository_fixture`、非 `fixture://` logical URI 或非 repository-owned fixture/in-memory port。
2. 调用 `require_action_eligible(...expected_kind=nas_replica_sync, expected_context=action_context, expected_origin=repository_fixture)`；同时校验 decision/context/selection 的 target/origin 一致，fixture+real 绑定在 first side effect 前 deny。
3. 调用 `verify_sealed_trial_return_bundle(source_bundle, source_selection)`；只有 `VerifiedTrialReturnBundleV1` 可进入 preflight，比较其 bundle/selection 与 request，S03 不 reopen path、不调用 seal canonicalizer/digest。
4. 生成唯一 staging token，通过 port 流式 copy 为 staged `SealedTrialReturnBundleV1`；中断即 BLOCKED，staging 保持 non-distributable，S03 不创建第二个 `ResearchCanonicalSelectionV1`。
5. 对 staged bundle 与原始 selection 再调用同一 S02 verifier并完成五项校验；receipt `original_seal_sha256`、`decision_origin`、`target_kind` 直接取 staged `VerifiedTrialReturnBundleV1` 及其 selection，不自行复算、推断或改写。
6. commit 前消费 fresh `ActionDecisionV1` + `ActionScopeContextV1`；重新比较 candidate/request/context/current source selection，阻断 mid-operation revoke/drift/origin-target mismatch。
7. 先持久化 immutable version/receipt，再以 expected previous revision 做 CAS；只有 CAS 成功返回新 distribution selection。
8. 任何失败均返回 machine reason 与 previous selection ref；不自动删除 staging/immutable evidence，不改 research canonical。
9. S04 读取时必须提交当前 `DistributionSelectionV1`；port 只返回该 selection 引用的 immutable `SealedTrialReturnBundleV1`、原 `ResearchCanonicalSelectionV1` 与 `ReplicaVerificationReceiptV1`。receipt/version/source-selection ref 任一漂移即 BLOCKED；不允许 path/latest/staging/receipt-only 旁路。

## 8. 技术细节（Technical Design Details）

- 关键算法 / 规则：SHA-256 使用流式更新；hash 字符串用 `hmac.compare_digest`；receipt canonicalization 使用版本化 domain + deterministic JSON；路径映射在进入 port 前做 root confinement 和 symlink-escape 检查；pointer 使用 compare-and-swap，不使用“覆盖最新文件”。
- freshness：`fresh := source_selection.release_id == expected_release_id AND canonical_sha256(source_selection) == expected_source_selection_sha256`。wall-clock mtime、目录排序和通配符命中不参与。
- original seal：S03 只调用 S02 `verify_sealed_trial_return_bundle` 并读取 `VerifiedTrialReturnBundleV1.original_seal_sha256`；不导入/调用 `canonical_artifact_seal_bytes` 或 `canonical_artifact_seal_sha256`，不创建新的 `ArtifactSealV1`。receipt 只证明该原 seal 的某次 replica verification；secondary seal canonicalizer/digest=`0/0`。
- staging：token 和 candidate 均显式 `non_distributable`；只有 CAS 后的 `DistributionSelectionV1` 才可供 S04 读取。
- authorization：本模块不查 ledger/backend；preflight 与 commit 都必须由 caller 提供 S01 `ActionDecisionV1` + `ActionScopeContextV1`。fixture decision 的精确标记是 `decision_origin=repository_fixture`，且只允许 `target_kind=repository_fixture`、`fixture://` URI 与 repository-owned fixture/in-memory port；不存在 `fixture_only` 别名。
- 兼容性：legacy path 只读审计、不作为 mapping 默认值；没有 migration/rename/rewrite；未知 schema/object kind 返回 BLOCKED。
- 依赖选择与复用点：精确复用 S01 `ActionDecisionV1`/`ActionScopeContextV1` 与 S02 三个 public types/一个 verifier；S03 自己只拥有 replica receipt/selection hash domain，不拥有 seal hash domain。
- selected-replica read：S03 port 在 CAS 后按 current `DistributionSelectionV1` 做 exact ref lookup，并返回三个已有 immutable public values；该方法不做 seal 验证、不 canonicalize、不 hash。S04 必须继续调用 S02 verifier-library，故 verifier truth count 保持 `1`。
- 图示类型选择：使用时序图，因为跨 Caller、S01 decision、S03 verifier、storage port，且包含中断、撤销与 CAS 补偿分支。

## 9. 安全与性能设计

| 维度 | 设计措施 | 验证方式 |
|---|---|---|
| 授权最小化 | preflight/commit 两次消费 `nas_replica_sync` 独立 decision；直接前置只接受 sealed generation provenance；不做权限并集 | missing/deny/expired/revoked/context-mismatch 参数化测试；staging/pointer=`0/0` |
| 真实操作隔离 | production module 仅定义 port/orchestrator；当前 target kind 只接受 `repository_fixture`；无 NAS/network/mount/credential adapter | import/static scan + fake mapping；真实 host/path literal 与 socket/subprocess 调用=`0` |
| 路径安全 | logical URI 先校验；fixture port resolve 后必须位于声明 root；拒绝 `..`、symlink escape、absolute override | traversal/symlink negative tests |
| 完整性 | source 和 staged bytes 双验证；5/5 vector；original seal 不重建；canonical receipt hash | mutation matrix逐字段测试，accepted=`0` |
| 原子性 | immutable write + CAS pointer；receipt/版本写成功但 CAS 失败时保持 unselected | failure injection，断言 previous selected hash 100% 不变 |
| 性能 | 1 MiB 流式 chunk；source 一次 copy/hash、staging 一次 re-read/hash；无目录全扫描 | counting stream 断言 bounded reads；大 fixture 监测不调用整文件 API |
| 可审计性 | result 带 reason/evidence refs/previous-new selection；canonical receipt 排除 path/credential | receipt schema/hash golden tests |
| 单一数据选择 | 只有 current `DistributionSelectionV1` 可解析 S04 输入 tuple；unselected/staging/path/latest lookup 均无 public API | read-spy + wrong-selection/ref-drift negative；bypass=`0` |

## 10. 测试设计

| 测试场景 | 前置条件 | 操作 | 预期结果 | 验证方式 |
|---|---|---|---|---|
| T-S03-P01 正常 replica + selected read | valid `SealedTrialReturnBundleV1`/`ResearchCanonicalSelectionV1`；preflight+commit fixture decision/context；empty pointer | S02 verifier→stage→S02 verifier→5/5→CAS→按 current selection read | 两次均返回 `VerifiedTrialReturnBundleV1`；pointer advance=`1`；read tuple exact types=`3/3` 且 refs match=`3/3`；research pointer mutation=`0` | unit + in-memory/`tmp_path` port |
| T-S03-P02 deterministic receipt | 同一 canonical input，不同 fixture root | 各执行 3 次 | receipt hash distinct count=`1`；absolute path occurrence in hash/body=`0` | golden hash |
| T-S03-A01 独立授权/fixture binding 缺失 | sync allow 缺失/deny/expired/revoked，或 generation provenance/context 不匹配，或 `repository_fixture + real_operation` | 调 preflight | BLOCKED；port call/staging/pointer=`0/0/0`；fixture+real accepted=`0` | mock call counters |
| T-S03-A02 commit 前撤销 | preflight allow；staging 5/5；commit decision revoked/expired | commit | receipt selection=`0`；previous pointer 100% 保留 | failure fixture |
| T-S03-N01 source invalid | wrong-kind/unsealed/unversioned | preflight | BLOCKED；staging=`0` | 参数化测试 |
| T-S03-N02 mismatch matrix | content/manifest/seal/release/logical URI/source-selection 任一篡改 | preflight 或 staging reverify | accepted/pointer advance=`0/0`，machine reason 精确 | 参数化 mutation |
| T-S03-N03 stale source/selected read | request expected selection 与 current source selection hash 不同，或 read 请求使用旧/伪造 distribution selection | preflight/read | `STALE_SOURCE_SELECTION`/BLOCKED；不以 mtime/latest/receipt-only 兜底；read tuple=`0` | fixed clock-free fixture |
| T-S03-F01 copy 中断/partial | port 在第 k 个 chunk 抛中断 | stage | staging non-distributable；receipt/pointer=`0/0`；previous selection 保留 | fault-injection port |
| T-S03-F02 staged bytes 篡改 | copy 后修改 1 byte/manifest/seal | 对 staged public types 调同一 S02 verifier | typed BLOCKED 或 5/5 不成立；receipt/pointer=`0/0` | mutation test |
| T-S03-F03 CAS conflict | 验证后另一 writer 改变 pointer revision | commit | `POINTER_CONFLICT`；不得覆盖 current selection | deterministic CAS fake |
| T-S03-B01 path identity | 相同 bundle 映射不同 root/hostname | canonicalize receipt | identity/hash distinct count=`1`；path count=`0` | structural assertion |
| T-S03-B02 traversal/real target | `..`、symlink escape、`target_kind=real_operation`、非 `fixture://` URI 或非 repository-owned port | preflight/stage | BLOCKED；file/network operation=`0`；fixture+real accepted=`0` | static + tmp_path negative |
| T-S03-R01 rollback | 已有 previous verified replica；任一后续失败 | 完整失败矩阵 | previous selected receipt/content hash preservation=`100%` | before/after pointer equality |
| T-S03-Z01 no-real-op/claim | repository test suite | static scan/counters | legacy context alias=`0`；S03 seal canonicalizer/digest/verifier-facade/data-bypass=`0/0/0/0`；real sync/write/runtime/trading=`0`；Stage3/C1 flags不变 | static guard |

## 11. 实施步骤

| TASK-ID | 动作 | 目标文件 | 详细描述 | 对应测试 |
|---|---|---|---|---|
| CR172-S03-T01 | 创建 | `process/stories/STORY-CR172-S03-nas-replica-verification-LLD.md` | 冻结 receipt canonicalization、freshness、port、双授权判定、CAS、失败与回滚合同 | CP5 `lld-check` |
| CR172-S03-T02 | 创建 | `engine/research_artifact_replica.py` | 实现 typed value/result、两次调用 S02 verifier、source/staging 5/5、fixture guard、receipt/CAS 及 selection-bound read；精确消费 S01/S02 contract，S03 seal canonicalizer/digest/verifier-facade=`0/0/0` | T-S03-P01/P02/A01/N01/N02/N03/B01/B02/Z01 |
| CR172-S03-T03 | 创建 | `tests/research/test_cr172_nas_replica_verification.py` | 实现 in-memory/`tmp_path` port 和正向、mismatch、partial、revoke、CAS、path/identity fixtures | T-S03-P01～Z01 |
| CR172-S03-T04 | 验证 | `engine/research_artifact_replica.py`、`tests/research/test_cr172_nas_replica_verification.py` | 运行 scoped tests/static guard，证明 original seal、research pointer/no-real-op/previous selection 约束 | T-S03-A02/F01/F02/F03/R01/Z01 |

实施顺序固定为 T01→T02→T03→T04；CP5 未批准时 T02～T04 均不得开始。dev gate 还要求 S01 contract frozen+merged、S02 implementation evidence PASS、文件 owner 无冲突。

## 12. 风险、难点与预研建议

### 12.1 实现灰区与取舍记录

| Clarification ID | 问题 | 选项与推荐 | 决策 / 答案 | 影响面 | 证据 | 重访条件 |
|---|---|---|---|---|---|---|
| 无 | 当前没有需用户决策的实现灰区 | 推荐：S03 current selection 唯一选数并返回 immutable bundle/selection/receipt，S04 直连 S02 verifier-library；备选仅为 distribution blocked | 采用推荐；verifier facade/data bypass=`0/0`；clarification queue=`0` | 接口、owner、测试、安全、跨 Story 契约 | HLD/ADR v1.4、Feature I01/I02/I03 v1.2、R2 F-002、CP5 context | 若提出真实 NAS adapter、绕过 selection、S03 digest/verifier facade、shared-drive runtime 或迁移授权，则返回 `NEEDS_DESIGN_CLARIFICATION`/独立 CR |

| 风险 / 难点 | 影响 | 缓解措施 / 预研建议 |
|---|---|---|
| NAS 被误升格为 canonical | 双真相、研究机回写风险 | receipt 明示 `authority=replica`；S03 无 research pointer 写接口；static forbidden-edge test |
| source 在 copy 期间变化 | staging 与 approved source selection 不一致 | immutable source selection + copy 前/后 hash + expected selection；不读 mutable latest |
| pointer lost update | 覆盖其他已验证版本 | 使用 expected previous revision/hash CAS；冲突返回 BLOCKED |
| mid-operation revoke | 未授权 pointer commit | commit 前重新消费 fresh typed decision；staging non-distributable |
| 物理路径进入审计 identity | 跨主机 hash 漂移 | receipt canonical body allowlist；mapping/path 对象独立；golden test |
| partial cleanup 诱发越权删除 | 删除审计证据或未授权 NAS 写 | 当前不自动 cleanup；保留 non-distributable token；未来 cleanup 需新授权 |
| streaming port 隐式整文件加载 | 大 artifact 内存失控 | iterator/chunk contract + counting stream；禁止 `.read_bytes()` |
| S04 以 receipt/path 旁路 selected bytes | seal bit 无法 bytes-level 复验或读到未选版本 | `read_selected_replica(current selection)` 返回 exact tuple；wrong selection/ref drift BLOCKED；S04 另用 S02 verifier |

### OPEN / Spike 跟踪

| ID | 类型（OPEN / Spike） | 问题 | 下一动作 | 责任方 |
|---|---|---|---|---|
| 无 | N/A | 未决 OPEN/Spike=`0` | 无 | 无 |

## 13. 回滚与发布策略

- 发布方式：CP5 批准后按 Wave 3 交付 repository-local Python contract 与 tests；不部署 NAS adapter，不切换任何真实 pointer，不修改 runner 默认路径。
- 发布前门控：S01、S02 dev gate 已满足；S03 scoped tests/static guard PASS；真实六动作 authorized/executed 仍=`0/6`,`0/6`。
- 回滚触发条件：S01/S02 contract mismatch、任何 5/5 误接受、path 进入 identity、CAS 非原子、previous selection 被失败路径改变或发现真实 NAS edge。
- 回滚动作：回退 S03 新增模块/测试这一独立切片；真实系统无迁移/无目录/无 pointer 需要回滚。若只回滚一次 fixture selection，则使用 pointer-only rollback 到 previous receipt ref，绝不改写 sealed bytes。
- 不采用的降级：NAS working disk、execution direct-NAS、重新 seal、absolute-path identity、自动 cleanup 或 legacy migration 均不得作为回滚方案。

## 14. DoD（Definition of Done）

- [x] 0～14 节全部填写，frontmatter `lld_version=1.3`、`status=confirmed`、`confirmed=true`、`tier=L`、`open_items=0`。
- [ ] F-S03-01～11 与 NFR-S03-01～09 均有对应接口、失败行为和测试；测试表唯一 ID=`14/14`、duplicate=`0`。
- [ ] source/staging verification=`5/5`，所有 partial/stale/unversioned/mismatch accepted/pointer=`0/0`。
- [ ] preflight 与 commit 两次授权判定明确；permission union=`0`；revoke/expiry/context drift pointer advance=`0`。
- [ ] `ReplicaVerificationReceiptV1` canonical hash 排除 absolute path/credential；digest/origin/target 与 staged `VerifiedTrialReturnBundleV1`/selection 同值覆盖=`3/3`；S03 secondary seal canonicalizer/digest=`0/0`。
- [ ] previous distribution selection 在 interruption/mismatch/revoke/CAS failure 下 preservation=`100%`。
- [ ] S04 selected read 只能由 current `DistributionSelectionV1` 触发，并精确返回 immutable bundle/source selection/receipt=`3/3`；unselected/staging/path/latest/receipt-only read accepted=`0/5`；新增 digest/verifier facade/data bypass=`0/0/0`。
- [ ] 仅创建 S03 的两个 primary implementation/test 文件；forbidden file modification=`0`。
- [ ] `decision_origin=repository_fixture` + `target_kind=repository_fixture` + `fixture://` + repository-owned port 是唯一当前 adapter；fixture+real accepted=`0`；真实 NAS/read/write/sync 与六动作 authorized/executed=`0/6`,`0/6`。
- [ ] clarification、OPEN、Spike=`0/0/0`；若边界条件变化按第 12 节返回设计澄清。
- [ ] `meta-flow story lld-check` 与 targeted `git diff --check` PASS。
- [ ] CP5 全量人工确认前不实现；S01/S02 dev gate 未满足时不进入 Wave 3。

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
