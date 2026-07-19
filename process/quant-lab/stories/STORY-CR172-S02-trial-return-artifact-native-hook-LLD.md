---
story_id: "CR172-S02-trial-return-artifact-native-hook"
title: "Trial-return artifact contract, verified seal, and fixture producer port"
story_slug: "trial-return-artifact-native-hook"
lld_version: "1.3"
tier: "L"
status: "confirmed"
confirmed: true
created_by: "meta-dev"
created_at: "2026-07-18T10:58:00+08:00"
confirmed_by: "user"
confirmed_at: "2026-07-18T13:49:27+08:00"
implementation_allowed: true
shared_fragments: []
feature_design_refs:
  - "docs/features/trial-return-artifact-pipeline/DESIGN.md"
  - "docs/features/trial-return-artifact-pipeline/TEST-PLAN.md"
  - "docs/features/trial-return-artifact-pipeline/TASKS.md"
  - "docs/features/path-i-authorization-claim-governance/DESIGN.md"
  - "docs/features/path-i-authorization-claim-governance/TEST-PLAN.md"
  - "docs/features/path-i-authorization-claim-governance/TASKS.md"
lld_policy:
  required_level: "full-lld"
  trigger_reasons:
    - "new immutable data/schema"
    - "fixture/real authorization binding"
    - "verified seal handoff"
    - "partial-lineage blocked audit"
  rationale: "现有 runner 不具备真实 portfolio-period-return 语义且 append-only lineage 无原子回滚能力；本 Story 必须收窄为 pure contract、repository fixture producer port 和唯一 verified-seal handoff。"
open_items: 0
---

# LLD: CR172-S02 — Trial-return artifact contract, verified seal, and fixture producer port

> 本 LLD 只设计 `engine/trial_return_artifact.py` 和本 Story 独占测试。历史 Story ID 中的 `native-hook` 仅保留审计稳定性：现有 mature runner hook/diff=`0/0`，lineage store/test diff=`0/0`，真实 trial-return generation/canonical advance=`0/0`。

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-18 | meta-dev | 初始 full LLD；曾规划现有 mature runner hook、forward-label 映射和 lineage 补偿。 |
| 1.1 | 2026-07-18 | meta-dev | CP5 R1 F-001～004 整改：删除 runner/turnover/lineage 写入承诺；收窄为 pure schema + repository fixture producer port；冻结 forward proxy 禁入、fixture/real binding、唯一 seal bytes/digest/verifier 与 append-only partial lineage BLOCKED audit。 |
| 1.2 | 2026-07-18 | meta-dev | CP5 R2 F-R2-002 指针/依赖整改：同步 HLD/ADR v1.3、Feature I01/I03 v1.2；保持 normative seal contract 不变，明确 S04 可消费 S02 唯一 verifier library，但 payload/selection 仍只能来自 S03 selected replica。 |
| 1.3 | 2026-07-18 | host-orchestrator | CP5 批准前 optional 整改：当前 authority pointer-only 刷新至 HLD/ADR v1.4；historical v1.2 修订记录保留当时基线，normative delta=`0`。 |

## 0. 上游设计依据（工程依据）

| 来源 | 路径 / ID | 被本 LLD 消费的内容 |
|---|---|---|
| CP5 context / R3 handoff | `process/context/CP5-CR172-PATH-I-LLD-CONTEXT.yaml`；`process/handoffs/CR172-CP5-LLD-R3-BATCH-A-META-DEV-HANDOFF-2026-07-18.md` | runtime-high-risk-design、R3 allowed writes、六类真实动作 `0/6`、只更新指针与 S04 verifier-library 边界 |
| HLD v1.4 | `docs/design/HLD-TRIAL-RETURN-DEPLOYMENT-CONTRACTS.md` §1.6、§3、§9.1/9.4、§10.1～10.4、§11.2～11.4/11.8 | contract/fixture-only、唯一 seal verifier、current-v1 origin guard、S04 verifier-library / S03 data-source boundary |
| ADR v1.4 | `docs/design/ARCHITECTURE-DECISION-TRIAL-RETURN-DEPLOYMENT-CONTRACTS.md` ADR-001、004、005、009～011 | immutable seal顺序、S04 verifier-library许可、12-field record、真实 producer/原子 lineage 独立前置 |
| Feature I01 v1.2 | `docs/features/trial-return-artifact-pipeline/DESIGN.md`、`TEST-PLAN.md`、`TASKS.md` | pure contract、fixture producer port、S02→S03 exact public contract、normative seal truth、primary paths=`2` |
| Feature I03 v1.2 | `docs/features/path-i-authorization-claim-governance/DESIGN.md`、`TEST-PLAN.md` | current-v1 approved-ledger双拒绝、`decision_origin`/`target_kind`、fixture first-side-effect deny |
| Story | `process/stories/STORY-CR172-S02-trial-return-artifact-native-hook.md` | R3 目标、AC、文件 owner、forbidden paths、dev/verification gate |
| R2 review / minimal correction | `process/checks/CP5-CR172-LLD-REVIEW-R2-FINDINGS.md` F-R2-002；`process/handoffs/CR172-CP5-LLD-R3-MINIMAL-CORRECTION-META-SE-CRITICAL-RETURN-SUMMARY.md` | S04 bytes-level verifier-library 缺口与已冻结的最小、不扩权修正 |

## 1. Goal

创建一个 repository-local、可复现、fail-closed 的 `trial_portfolio_return_series@v1` pure contract：由显式 period-return fixture observations 生成 exact two-column payload、manifest、immutable seal、fixture-only selection 和 verified result；向 S03 导出唯一 seal bytes/digest/verifier 公共合同。不得读取现有 runner output，不得把 `forward_label_proxy@v1` 重标为 trial-return，不得写真实 canonical 或 lineage store。

量化出口：payload columns=`2/2`；public seal bytes/digest/verifier=`1/1/1`；fixture+real accepted=`0`；forward proxy 进入 trial-return/empirical-R/effective-count=`0/0/0`；partial lineage erase/fake rollback/canonical advance=`0/0/0`。

## 2. Requirements（需求；Functional / Non-Functional）

### 2.1 Functional

- F-01：只接受 `object_kind=trial_portfolio_return_series@v1`；`layered_returns`、metadata/ref、scalar metric、dependency matrix 和 `forward_label_proxy@v1` accepted=`0`。
- F-02：canonical Parquet payload 必须恰好包含 `timestamp,simple_return` 两列；`net_return/gross_return/nav` canonical occurrence=`0/0/0`。`return_basis` 只在 manifest 显式声明，不从列名推断。
- F-03：每个 fixture observation 显式提供 `interval_start`、UTC `timestamp`（区间结束）和 `simple_return`；区间必须正向、按结束时点严格递增且不重叠，return finite、non-null、非 NaN/Inf、`>=-1.0`。missing/overlap/ambiguous alignment 不得 fill、repair、clamp 或推断。
- F-04：fixture identity 必须使用 `fixture://` logical URI，并绑定 `ActionDecisionV1.decision_origin=repository_fixture`、`ActionScopeContextV1.target_kind=repository_fixture` 和 repository-owned fixture/in-memory port；任一 `real_operation` 或非 fixture target 在 first port side effect 前 accepted/side-effect=`0/0`。
- F-05：`ActionAuthorizationRecordV1` 保持 S01 的精确 12 fields；S02 只消费 `ActionDecisionV1`、`ActionScopeContextV1`、`require_action_eligible`，不得读取/合并 record 或构造 approved-ledger adapter。
- F-06：manifest 精确承载 HLD 的 `15/15` 语义；seal 精确承载 `8/8` 语义。payload content hash、manifest body hash、seal digest 的 domain 分离且 versioned；host path occurrence in identity/hash=`0`。
- F-07：必须导出 `SealedTrialReturnBundleV1`、`ResearchCanonicalSelectionV1`、`VerifiedTrialReturnBundleV1`、`canonical_artifact_seal_bytes`、`canonical_artifact_seal_sha256`、`verify_sealed_trial_return_bundle`；S03 不得另算 digest 或 re-seal。
- F-08：`canonical_artifact_seal_sha256(seal)` 精确等于 `"sha256:" + lowercase_hex(SHA256(canonical_artifact_seal_bytes(seal)))`；格式必须匹配 `^sha256:[0-9a-f]{64}$`，其他 prefix/case/length accepted=`0`。
- F-09：`verify_sealed_trial_return_bundle(bundle, selection)` 复算 payload、manifest 和同一 seal bytes；只在 identity/release/hash/origin/target 全一致时返回 `VerifiedTrialReturnBundleV1`，且 `original_seal_sha256` 只能来自上述唯一 digest 函数。
- F-10：repository fixture port 只有在 verifier PASS 后才原子接受 `VerifiedTrialReturnBundleV1`；partial payload/manifest/seal、tamper、port exception 的 fixture selection advance=`0/4`。
- F-11：本 Story 不调用 lineage API。未来 append-only lineage 出现 finish/finalize/selection partial success 时只输出 `partial_lineage_blocked_audit`；已追加事件 erase=`0`、调用 `fail()` 伪装 rollback=`0`、trial-return canonical selection advance=`0`。
- F-12：现有 `engine/mature_multifactor_research.py`、lineage modules/stores 和既有 lineage tests 的 hook/diff 必须保持 `0/0`、`0`、`0`；真实 producer 与原子 lineage 分别由独立 runtime-high-risk / lineage-owner CR 承接。
- F-13：S04 可以把 `SealedTrialReturnBundleV1`、`ResearchCanonicalSelectionV1`、`VerifiedTrialReturnBundleV1` 和 `verify_sealed_trial_return_bundle` 作为 verifier-library contract dependency 使用，以完成 execution-staging bytes-level 复验；但 S04 的 bundle/selection 数据只能由 S03 selected replica staging port 提供，绕过 S03 selection/receipt-only trust/secondary digest/re-seal=`0/0/0/0`。

### 2.2 Non-Functional

- NFR-01（确定性）：相同显式 fixture、serializer profile 和时间输入重复 `3/3`，payload bytes、content hash、manifest bytes/hash、seal bytes/digest distinct count 均为 `1`。
- NFR-02（完整性）：所有 digest 为 `sha256:` + lowercase 64 hex；canonical JSON key order/number/string/UTC 规则固定；selection/pointer 不进入任何 hash domain。
- NFR-03（fail-closed）：wrong kind、origin/target mismatch、interval/schema/hash/seal/port error 均产生 typed reason，verified result/fixture selection=`0`。
- NFR-04（安全）：production module 无 filesystem/network/env/credential/subprocess/NAS/lake/runtime/lineage/R/signal/trading/deploy/Git remote adapter；真实 operation=`0`。
- NFR-05（性能）：observation 校验/排序为 `O(n log n)`，serialization/hash 为 `O(n)`，内存 `O(n)`；只声明 repository fixture 规模，不声明 production SLA。
- NFR-06（兼容）：v1 schema 不容忍 unknown/extra canonical column；增加 net/gross/nav 或改变 simple-return 语义必须 versioned ADR + schema bump，必要时独立 CR。

## 3. 模块拆分与职责

| 模块 / 文件组 | 职责 | 明确不负责 |
|---|---|---|
| `engine/trial_return_artifact.py` — schema/serialization | fixture observation/identity/definition 校验；exact two-column Parquet 与 canonical JSON bytes/hash | 现有 runner mapping、真实 producer、数据湖/NAS/path IO |
| `engine/trial_return_artifact.py` — manifest/seal/verifier | 构造 immutable manifest/seal/bundle/selection；唯一 seal digest；返回 verified result | re-seal、second digest、真实 canonical pointer |
| `engine/trial_return_artifact.py` — fixture port/audit facet | 定义 repository-owned in-memory port 协议与 `partial_lineage_blocked_audit` 纯值合同 | lineage 写入、erase、`fail()`、原子批次/outbox/correction |
| `tests/research/test_cr172_trial_return_artifact.py` | schema/hash/binding/failure/zero-diff deterministic fixtures | 既有 lineage tests、真实 root/runtime/NAS |

## 4. 代码结构与文件影响范围

| 动作 | 文件路径 | 变更内容 / owner |
|---|---|---|
| 创建 | `engine/trial_return_artifact.py` | S02 primary；pure contract、fixture port、manifest/seal/verifier、partial-lineage audit value |
| 创建 | `tests/research/test_cr172_trial_return_artifact.py` | S02 primary；10 个 Feature R2 cases + public-contract/static zero-diff tests |

Primary paths=`2/2`，duplicate owner=`0`。下列路径修改数必须分别为 `0`：`engine/mature_multifactor_research.py`、`tests/research/test_trial_lineage_producer_adapters.py`、`engine/path_i_governance.py`、`engine/experiment_family_lineage.py`、`engine/experiment_family_lineage_store.py`、public C1/admission/effective-estimator paths。

## 5. 数据模型与持久化设计

本 Story 没有真实持久化；所有对象采用 frozen/slots value object，唯一写面是测试注入的 repository-owned in-memory fixture port。

### 5.1 输入与 artifact value objects

| 对象 | 精确字段 | 约束 / 说明 |
|---|---|---|
| `TrialReturnSourceKindV1` | enum values `trial_portfolio_return_series@v1`, `forward_label_proxy@v1` | 仅第一值可进入 candidate；第二值永远 BLOCKED |
| `TrialReturnObservationV1` | `interval_start, timestamp, simple_return` | aware UTC；start < end；ordered/non-overlap；finite return >= -1 |
| `ReturnDefinitionV1` | `object_kind, schema_version, return_basis, endpoint_semantics, non_overlap_required, alignment_policy` | object/schema 固定 v1；basis 非空且跨 observations 一致；本 Story不扩列 |
| `TrialReturnIdentityV1` | `family_id, run_id, trial_id, release_id, logical_uri` | fixture port 只接受 `fixture://`；host absolute path occurrence=0 |
| `TrialReturnPayloadV1` | `schema_version, payload_bytes, row_count, observation_window, content_sha256` | decoded schema exact `timestamp,simple_return`；payload bytes 是 content hash 的唯一 domain |
| `TrialReturnManifestV1` | `object_kind, schema_version, family_id, run_id, trial_id, logical_uri, return_basis, source_lineage_refs, row_count, observation_window, content_sha256, producer_contract_version, release_id, created_at, seal_status` | semantics=`15/15`；canonical body 不含自身 hash、selection 或 host path |
| `ArtifactSealV1` | `seal_version, logical_uri, content_sha256, manifest_sha256, release_id, sealed_at, producer_contract_version, authorization_evidence_refs` | semantics=`8/8`；immutable；refs-only，不含凭据 |

### 5.2 S02→S03 exact public contract

| Public type | 精确字段 | 不变量 |
|---|---|---|
| `SealedTrialReturnBundleV1` | `payload, manifest, manifest_sha256, seal` | pure immutable bundle；不含 host path、mutable pointer 或 lineage store state |
| `ResearchCanonicalSelectionV1` | `selection_version, release_id, logical_uri, content_sha256, manifest_sha256, original_seal_sha256, decision_origin, target_kind, selected_at` | 当前只允许 `repository_fixture/repository_fixture`；selection 不参与 payload/manifest/seal hash |
| `VerifiedTrialReturnBundleV1` | `bundle, selection, original_seal_sha256` | `original_seal_sha256 == selection.original_seal_sha256 == canonical_artifact_seal_sha256(bundle.seal)`；只能由 verifier 构造 |
| `AppendOnlyLineageAuditV1` | `state, observed_event_refs, erase_events, fake_rollback, canonical_selection_advance` | partial 时 state=`partial_lineage_blocked_audit`；后三 bool 全 false |

### 5.3 Canonical encodings and hash domains

- Parquet schema 顺序固定为 `timestamp: timestamp[us, tz=UTC]`、`simple_return: float64`，nullable=`false,false`；writer profile 名固定 `trial-return-parquet-v1`。具体实现使用锁文件已有的 pyarrow 16.1.0；压缩、row-group、statistics、dictionary、timezone/coercion 参数在实现中以一个 versioned constant 集中冻结。
- canonical JSON 使用 UTF-8、NFC string、lexicographic keys、无 insignificant whitespace、UTC `YYYY-MM-DDTHH:MM:SS.ffffffZ`、禁止 NaN/Inf；tuple 保序。manifest hash domain 是不含 `manifest_sha256` 的 manifest body bytes。
- `canonical_artifact_seal_bytes(seal)` 是 `artifact-seal-json-v1` 的唯一 canonical encoding；`canonical_artifact_seal_sha256` 只 hash 这组 bytes。S03 secondary canonicalization/digest/re-seal=`0/0/0`。
- fixture selection 只存在于 injected in-memory port；selection advance 是验证后的原子 replace 语义，不写研究机目录、不创建真实 `current.json`。

## 6. API / Interface 设计

| 接口 / 入口 | 输入 | 输出 | 调用方 | 失败行为 / side-effect 边界 |
|---|---|---|---|---|
| `prepare_repository_fixture_candidate(identity, observations, definition, *, created_at, source_lineage_refs)` | typed fixture identity、显式 period observations、definition、UTC 时间、opaque refs | `TrialReturnPayloadV1` + `TrialReturnManifestV1` | S02 fixture publisher/tests | wrong-kind/proxy/interval/schema/basis 立即 typed BLOCKED；无 port call |
| `canonical_artifact_seal_bytes(seal)` | `ArtifactSealV1` | canonical immutable `bytes` | S02 verifier、S03 只允许通过 verifier间接消费 | 不读取 path；unknown/missing field BLOCKED |
| `canonical_artifact_seal_sha256(seal)` | `ArtifactSealV1` | `sha256:` + lowercase 64 hex | S02 publisher/verifier | hash domain 只能是同一次 `canonical_artifact_seal_bytes(seal)` 返回值 |
| `verify_sealed_trial_return_bundle(bundle, selection)` | `SealedTrialReturnBundleV1`、`ResearchCanonicalSelectionV1` | `VerifiedTrialReturnBundleV1` | S02 fixture port、S03 replica | 复算 payload/manifest/seal；任一 mismatch/origin-target error 抛 `TrialReturnIntegrityError`，不返回 partial result |
| `publish_repository_fixture_trial_return_artifact(identity, observations, definition, generation_decision, generation_context, fixture_port, *, created_at, sealed_at, source_lineage_refs)` | fixture typed inputs；S01 `ActionDecisionV1`/`ActionScopeContextV1`；`RepositoryFixtureTrialReturnPortV1` | `VerifiedTrialReturnBundleV1` | repository fixture/tests | 最先 `require_action_eligible(...expected_origin=repository_fixture)`；构造/verify 后才调用一次 `fixture_port.commit_verified`；bad binding/port exception selection=0 |
| `classify_append_only_lineage_partial_success(observed_event_refs)` | 非空 opaque append-only refs | `AppendOnlyLineageAuditV1` | future integration contract test/audit | 只返回 BLOCKED audit value；不访问 lineage，不 erase，不调用 `fail()`，不推进 selection |

精确跨 Story签名：

- S03 import `SealedTrialReturnBundleV1`、`ResearchCanonicalSelectionV1`、`VerifiedTrialReturnBundleV1` 和 `verify_sealed_trial_return_bundle`；其 receipt 的 `original_seal_sha256` 必须直接来自 verified result。S03 不 import两个 canonical seal helper，不重算 digest。
- S04 可以 import同一 3 types + verifier 作为 library dependency，对 **S03 selected-replica staging port 返回的** bundle+selection 做 bytes-level复验；该 library dependency 不是 data-source dependency，不允许 S04直接从 S02/research canonical/NAS未选择对象取数。
- S04 同样不得 import两个 canonical seal helper、自建 parser/digest或以 S03 receipt metadata 替代 bytes；normative seal bytes/digest/verifier 定义仍只有本 LLD §5.3/§6 一套。

## 7. 核心处理流程

```mermaid
sequenceDiagram
  participant Caller as Repository fixture caller
  participant Guard as S01 require_action_eligible
  participant Artifact as trial_return_artifact pure contract
  participant Verify as verify_sealed_trial_return_bundle
  participant Port as Repository-owned in-memory port

  Caller->>Guard: decision + context + expected fixture origin/target
  alt origin/target/path/eligibility mismatch
    Guard-->>Caller: deny before first port side effect
  else fixture binding valid
    Guard-->>Caller: eligible for fixture contract only
    Caller->>Artifact: explicit period observations + fixture identity
    Artifact->>Artifact: validate intervals → exact Parquet → manifest → seal
    Artifact->>Artifact: compute one original_seal_sha256
    Artifact->>Verify: bundle + fixture selection
    Verify->>Verify: recompute payload/manifest/same seal bytes
    alt any mismatch
      Verify-->>Caller: typed BLOCKED; selection=0
    else verified
      Verify-->>Caller: VerifiedTrialReturnBundleV1
      Caller->>Port: commit_verified once
      Port-->>Caller: fixture selection committed
    end
  end
```

固定顺序：

1. 在调用 serializer 或 port 前校验 generation decision 的 action kind/context/origin/target；`repository_fixture+real_operation` 或非 `fixture://` 立即拒绝。
2. 校验 object kind 与全部 observation interval；`forward_label_proxy@v1`、重叠、缺端点或 unknown basis 不进入 payload。
3. 生成 exact two-column payload bytes/content hash，再生成 manifest body/hash，最后生成 immutable seal。
4. 生成 fixture-only candidate selection；只调用唯一 verifier，得到 `VerifiedTrialReturnBundleV1` 与 original seal digest。
5. 只有 verified result 才传给 injected in-memory port；port exception 返回 typed BLOCKED，selection advance=0。
6. 本流程不调用 runner/lineage/NAS/R/signal；partial-lineage classifier 是独立纯函数，永不参与成功 fixture selection。

## 8. 技术设计细节（技术细节）

- `simple_return` 表示输入者已明确声明的 portfolio period simple return；本 Story只验证语义元数据/区间，不从 prices、holdings、labels 或 turnover 计算。真实 producer 的区间、权重、成本和 alignment 证明仍是独立前置。
- `forward_label_proxy@v1` 是独立 source kind；即使数值、两列名称或 hash 偶然匹配，也必须在 candidate 前因 kind 不符而拒绝。禁止 alias/coercion/fallback。
- timestamp 是 interval end，必须 tz-aware UTC；payload 不保存 interval_start，但 manifest `observation_window` 和 source lineage refs 固化最早 start、最晚 end、non-overlap/alignment 声明。区间证据不完整即 unavailable/BLOCKED。
- serializer profile 作为 `producer_contract_version` 的一部分；任何 pyarrow/参数升级产生新 contract version，绝不重写旧 sealed bundle。
- digest helper 只接受 typed `ArtifactSealV1`，不得接受 path/raw mapping；verifier 内部调用同一 helper一次并把值复制到 verified result。S03/S04均不存在可选 digest 算法；S04 library reuse不得改变 hash domain。
- fixture port 是结构化 Protocol，但唯一合法实现面是 repository test fixture/in-memory adapter；生产模块不得提供 filesystem adapter、env root adapter 或 approved-ledger adapter。
- typed errors：`TrialReturnContractError`（kind/schema/interval/basis）、`TrialReturnAuthorizationError`（decision/context/origin/target）、`TrialReturnIntegrityError`（payload/manifest/seal/digest/selection）、`TrialReturnFixturePortError`（in-memory commit）。错误只带稳定 reason code 与 opaque IDs，不带 payload/credential。
- `partial_lineage_blocked_audit` 是未来集成失败分类，不是当前 lineage capability。原子 batch/outbox/correction/supersession 的 owner、文件与实现数量在本 Story都为 0。
- 图示类型选择：时序图；它能最清楚表达 typed guard、唯一 hash/verifier 和 port first-side-effect 三个顺序守卫。

## 9. 安全与性能设计

| 维度 | 设计措施 | 验证方式 |
|---|---|---|
| 授权绑定 | 强制 `repository_fixture` origin+target+URI+port 四者一致；无 approved-ledger adapter | I03-N05 + write-spy；accepted/side-effect=`0/0` |
| 数据语义 | source kind 与 interval semantics 在 serialization 前检查；proxy accepted=0 | T-S02-N01/N03 |
| 完整性 | content/manifest/seal 三个独立 domain；seal bytes/digest/verifier 单一实现 | same input `3/3`、tamper、S03 import inventory |
| Side-effect ceiling | 只注入 in-memory fixture port；production imports/calls 不含 filesystem/network/env/runtime/lineage/NAS/R/signal | AST/import/monkeypatch zero-operation scan |
| 隐私/凭据 | manifest/seal 仅 opaque refs；credential/absolute path occurrence=0 | exact-field + negative fixtures |
| 性能 | 一次排序/校验和一次内存 serialization/hash；不扫描 runner/lineage/global ledger | fixture size/call-count test；无 production SLA |

## 10. 测试设计

| 测试 ID / 名称 | 前置 / 操作 | 预期结果 |
|---|---|---|
| `T-S02-P01 test_fixture_observations_produce_exact_v1_verified_bundle` | valid period observations + fixture decision/context/port | columns=`2/2`；verified bundle/selection=`1/1`；origin/target preserved=`100%` |
| `T-S02-P02 test_same_seal_has_one_digest_under_three_deployment_mappings` | same typed seal + 3 irrelevant host mappings | canonical seal bytes/digest distinct count=`1/1`；host path hash occurrence=0 |
| `T-S02-N01 test_forward_label_proxy_is_never_trial_return` | `forward_label_proxy@v1` even with valid-looking rows | trial-return/empirical-R/effective-count accepted=`0/0/0`；port calls=0 |
| `T-S02-N02 test_fixture_decision_with_real_target_is_denied_before_port` | fixture decision + real target/non-fixture URI + spy port | accepted/port side-effect=`0/0` |
| `T-S02-N03 test_wrong_kind_or_missing_interval_semantics_fails_closed` | layered/scalar/proxy/missing start/overlap/unknown basis | verified/selection=`0/0`；no repair/fill |
| `T-S02-F01 test_partial_payload_or_manifest_never_verifies_or_selects` | missing/truncated payload/manifest mutation | verified/seal/selection=`0/0/0` |
| `T-S02-F02 test_seal_digest_mismatch_uses_no_secondary_hash` | mutate seal/selection digest/prefix/case | BLOCKED；selection=0；canonicalization/digest implementations=`1/1` |
| `T-S02-F03 test_fixture_port_exception_keeps_selection_unadvanced` | valid verified result + commit exception | typed BLOCKED；selection advance=0；retry/real fallback=0 |
| `T-S02-F04 test_partial_lineage_is_blocked_audit_only` | simulated finish/finalize/selection partial refs | state=`partial_lineage_blocked_audit`；erase/fake rollback/canonical advance=`0/0/0` |
| `T-S02-B01 test_current_runner_and_lineage_are_untouched` | scoped git/source inventory | mature runner hook/diff=`0/0`；lineage modules/store/tests diff=`0/0/0`；forbidden producer call=0 |
| `test_public_contract_exports_and_digest_format_are_exact` | import public names + hash fixture | types=`3/3`、functions=`3/3`；digest regex match=`100%`；S03/S04 secondary digest contract=`0/0` |
| `test_s04_verifier_library_dependency_cannot_bypass_s03_selection` | cross-Story static contract inventory | S04 verifier import allowed=`1`；data source必须是 S03 selected bundle+selection；S02/research direct source、receipt-only trust、secondary digest/re-seal=`0/0/0/0` | cross-contract static |
| `test_real_operation_ceiling_is_zero` | static import/call inventory | lake/NAS/runtime/lineage/R/signal/trading/deploy/remote action=`0` |

目标命令：

```bash
PYTHONDONTWRITEBYTECODE=1 PYTEST_ADDOPTS='-p no:cacheprovider' uv run --python 3.11 pytest -q tests/research/test_cr172_trial_return_artifact.py
```

## 11. 实施步骤

| TASK-ID | 动作 | 目标文件 | 详细描述 | 对应测试 |
|---|---|---|---|---|
| CR172-S02-T01 | 修改 | `process/stories/STORY-CR172-S02-trial-return-artifact-native-hook-LLD.md` | R3 同步 v1.3/v1.2 基线与 S04 verifier-library / S03 selected-data 边界；normative seal contract不变 | CP5 LLD precheck |
| CR172-S02-T02 | 创建 | `engine/trial_return_artifact.py` | 创建 exact schema、period fixture validation、canonical serializer、manifest/seal、3 public types、3 public functions和 in-memory fixture port | P01/P02、N01～N03、F01～F04 |
| CR172-S02-T03 | 创建 | `tests/research/test_cr172_trial_return_artifact.py` | 创建 schema/hash/binding/failure/public-export/runner-lineage-zero-diff tests | Feature I01 TEST-PLAN 10/10 + static guards |
| CR172-S02-T04 | 生成证据 | S02 implementation/evidence refs | 输出 S02→S03→S04 exact verifier-library inventory、digest single-truth、S03 selected-data boundary、forbidden-path zero diff 与 zero-real-operation 计数 | contract/static evidence |

实施顺序固定 `T02 → T03 → T04`；开发前置为 CP5 全量批准且 S01 contract 已冻结并合并。CP5 前只允许 T01 设计证据。任何需要修改 runner/lineage 或真实 path 的实现发现必须停止并返回设计澄清/独立 CR。

## 12. 风险、难点与预研建议

### 12.1 实现灰区与取舍记录

| Clarification ID | 问题 | 选项与推荐 | 决策 / 答案 | 影响面 | 证据 | 重访条件 |
|---|---|---|---|---|---|---|
| DQ-S02-R2-01 | 现有 runner `net_forward_return` 是否可作为 trial return | A hook/mapping；B proxy 禁入并 pure fixture（推荐） | HLD/ADR v1.4 延续 B；runner hook/diff=`0/0` | source semantics、runtime、tests | R1 F-002；ADR-010/011 | 独立 runtime-high-risk producer CR 提供区间/权重/成本/non-overlap 证据 |
| DQ-S02-R2-02 | append-only lineage partial success 如何恢复 | A fake rollback；B BLOCKED audit + 独立 lineage CR（推荐） | HLD/ADR v1.4 延续 B；erase/fake rollback/advance=`0/0/0` | lineage owner、failure semantics | R1 F-001；ADR-009/011 | lineage owner批准原子 batch/outbox/correction contract |
| DQ-S02-R2-03 | seal digest 是否由 S03重算 | A 多层自算；B S02 verifier单一真相（推荐） | 选 B；S03 secondary digest=0 | S02→S03 contract | R1 F-004；ADR-004 | seal schema/version变化时回 CP3/CP5 |

上述决策均已收敛，clarification queue active items=`0`。

| 风险 / 难点 | 影响 | 缓解措施 / 预研建议 |
|---|---|---|
| fixture bytes/hash 被误称真实 evidence | 形成 false Stage3/activation claim | origin/target/URI/port typed binding；claim/real action保持 0；receipt 贯穿 fixture origin |
| Parquet library/profile 漂移 | 相同语义产生不同 content hash | versioned writer profile + 3/3 determinism；升级 producer contract version，不重写 sealed bytes |
| source kind 看似正确但 interval 语义缺失 | 重复/重叠 forward label进入 empirical-R | 显式 start/end/non-overlap/alignment precheck；缺失 unavailable/BLOCKED；不做 alias |
| consumer 创建第二套 seal digest | replica receipt 与原 seal无法证明同源 | exact public verifier/result；S03禁止 import hash helpers；cross-LLD static test |
| S04 把 verifier-library dependency误作 data-source dependency | execution materializer绕过 S03 selection或只信 receipt | S04只能消费 S03 selected-replica staging bundle+selection；复用 verifier仅做 bytes-level校验；bypass/receipt-only/secondary digest=0 |
| append-only partial lineage 被 exception handler掩盖 | 审计事实丢失/虚假 rollback | pure `partial_lineage_blocked_audit`；不调用 lineage；另起 owner CR |

### OPEN / Spike 跟踪

| ID | 类型（OPEN / Spike） | 问题 | 下一动作 | 责任方 |
|---|---|---|---|---|
| FU-S02-NATIVE-PRODUCER | OPEN-deferred prerequisite | 真实 multi-trial portfolio-period-return producer 与 runtime path enforcement 不存在 | 以独立 runtime-high-risk CR 冻结区间、持仓/权重、成本、basis、non-overlap/alignment、唯一插桩点；在 launch/workspace first side effect 前消费 `RunPathDecisionV1`，证明 new default=1、legacy write=0 | runtime/data owner |
| FU-S02-LINEAGE-ATOMICITY | OPEN-deferred prerequisite | append-only lineage 不支持原子 finish/finalize/selection | 独立 lineage-owner CR 选择 batch/outbox/correction-supersession | lineage owner |

这两个 OPEN 是真实 activation 的外部前置，不是本 LLD 实现灰区；本 Story `open_items=0`。

## 13. 回滚与发布策略

- 发布方式：CP5 批准且 S01 已合并后，只以 `engine/trial_return_artifact.py` + `tests/research/test_cr172_trial_return_artifact.py` 一个 repository-local slice 交付；不部署、不连接真实 root/backend。
- 回滚触发：payload schema≠2 列、proxy accepted>0、fixture+real accepted>0、digest implementation≠1、verifier 未返回 original digest、任何 runner/lineage diff、真实 operation>0。
- 代码回滚：删除/禁用新 pure module与其独占测试；现有 runner/lineage 从未改变，无补偿/迁移/default switch。
- 数据/指针回滚：无真实数据、目录或 pointer；fixture in-memory selection 随测试对象销毁。partial lineage 只保留 BLOCKED audit事实，不执行 erase/fake rollback。
- merge 顺序：`S01 → S02 → S03 → S04 → S05`；S03 合并后若回退 S02，必须先按逆序回退 S04/S03，不得留下 import dangling。

## 14. Definition of Done（DoD）

- [x] 0～14 节全部填写；frontmatter `lld_version=1.3`、`tier=L`、`status=confirmed`、`confirmed=true`、`open_items=0`。
- [ ] primary paths exact=`2/2`；mature runner hook/diff=`0/0`；lineage modules/store/test diff=`0/0/0`。
- [ ] payload exact columns=`2/2`；explicit interval semantics present=`100%`；`forward_label_proxy@v1` 进入 trial-return/R/count=`0/0/0`。
- [ ] S01 12-field record 不被扩充；decision-origin/context-target values=`2/2`；fixture+real accepted/side-effect=`0/0`。
- [ ] manifest/seal semantics=`15/15`,`8/8`；canonical seal bytes/digest/verifier implementation=`1/1/1`；digest format match=`100%`。
- [ ] 精确导出 `SealedTrialReturnBundleV1`、`ResearchCanonicalSelectionV1`、`VerifiedTrialReturnBundleV1`、`canonical_artifact_seal_bytes`、`canonical_artifact_seal_sha256`、`verify_sealed_trial_return_bundle`；S03 secondary canonicalization/digest/re-seal=`0/0/0`。
- [ ] S04 verifier-library dependency=`1`，但数据来源仍为 S03 selected replica=`1/1`；S04 bypass S03/receipt-only trust/secondary canonicalizer/digest/re-seal=`0/0/0/0/0`；normative seal contract delta=`0`。
- [ ] wrong kind、fixture+real、partial bytes、seal mismatch、port exception 的 verified/selection advance=`0/5`。
- [ ] partial lineage state=`partial_lineage_blocked_audit`；erase/fake rollback/canonical advance=`0/0/0`；不承诺成功 lineage痕迹回到 0。
- [ ] Feature I01 cases=`10/10`，每个接口/失败路径至少有一条测试；target command 只包含本 Story test file。
- [ ] real lake/NAS/runtime/lineage/R/signal/trading/migration/deploy/Git remote operation=`0`；六类真实动作 authorized/executed=`0/6`,`0/6`。
- [ ] clarification active/Open implementation/Spike=`0/0/0`；两个 deferred prerequisite 保留 owner/启动条件但不扩入本 Story。
- [ ] CP5 全量人工确认与 S01 merge gate满足前不得进入 T02～T04 实现。

## 人工确认区

> **CP5 — Story 设计证据可实现性门**。本 LLD 需与其余四份 CR172 LLD、CP4 自动预检和独立 R2 review 统一确认；本文件 ready-for-review 不授权实现或任何真实 operation。

**CP5 checklist 摘要**：

| # | 检查项 | 状态 | 证据 |
|---|---|---|---|
| 1 | R1 F-001～004 在本 Story边界内关闭 | 待检查 | §2、§7、§10、§14 |
| 2 | 与 HLD/ADR v1.4、Feature v1.2 一致 | 待检查 | §0、§5、§12 |
| 3 | 文件 owner / forbidden diff 明确 | 待检查 | §3、§4、§11 |
| 4 | S01→S02→S03 精确接口完整 | 待检查 | §5、§6、§8 |
| 5 | 测试、failure 与 dev_gate 可计算 | 待检查 | §10、§14 |
| 6 | clarification queue 已收敛 | 待检查 | §12.1；active items=`0` |

**人工审查结果回填**：

- 结论：`pending`
- 审查人：
- 审查时间：
- 修改意见：
- 风险接受项：
