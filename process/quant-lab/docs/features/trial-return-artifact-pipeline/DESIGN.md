---
status: "cp5-r2-design-ready"
version: "1.2"
feature_id: "FEAT-CR172-I01"
feature_name: "Trial Return Artifact Pipeline"
source_hld: "docs/design/HLD-TRIAL-RETURN-DEPLOYMENT-CONTRACTS.md"
source_adr: "docs/design/ARCHITECTURE-DECISION-TRIAL-RETURN-DEPLOYMENT-CONTRACTS.md"
source_matrix: "docs/design/FEATURE-DESIGN-MATRIX.md"
related_stories:
  - "CR172-S02-trial-return-artifact-native-hook"
  - "CR172-S05-path-i-integration-claim-zero-operation-verification"
lld_policy_summary: "2/2 full-lld; R2 required"
---

# FEAT-CR172-I01 Trial Return Artifact Pipeline — DESIGN

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-18 | meta-se-critical | 初始 CP4 Feature 设计。 |
| 1.1 | 2026-07-18 | meta-se-critical | CP5 LLD review R1 安全收窄：删除现有 mature runner hook 与 `net_forward_return` 映射；改为 pure contract + repository fixture producer port；冻结 forward-label proxy 禁入、partial lineage BLOCKED audit、fixture/real binding 与 S02→S03 verified seal digest。 |
| 1.2 | 2026-07-18 | meta-se-critical | CP5 R2：current-v1 approved-ledger 固定双 false；future native-producer 前置增加 RunPathDecisionV1 runtime path-enforcement；当前 runner/default switch 仍为 0。 |

## 1. 摘要与量化成功标准

本 Feature 只实现 `trial_portfolio_return_series@v1` 的 pure schema/manifest/seal/verifier、repository fixture producer port 和 fixture selection。现有 runner/source/lineage store 修改=`0/0/0`，真实生成/canonical advance=`0/0`。

- payload exact columns=`2/2`；`forward_label_proxy@v1` 进入 payload/empirical-R/effective-count=`0/0/0`。
- `ActionDecisionV1.decision_origin` 与 `ActionScopeContextV1.target_kind` binding=`2/2`；fixture+real accepted=`0`。
- canonical seal bytes/digest/verifier truth=`1/1/1`；S03 secondary digest implementation=`0`。
- wrong-kind、origin/target mismatch、partial bytes、seal mismatch、producer-port exception 的 fixture selection advance=`0/5`。
- partial lineage canonical advance=`0`；machine-readable `partial_lineage_blocked_audit`=`1/1`。
- 六类真实动作 authorized/executed=`0/6`,`0/6`。

## 2. 范围与相邻对象边界

| 对象 | 本 Feature 负责 | 不负责 / 禁止 |
|---|---|---|
| ReturnDefinitionV1 | exact `timestamp,simple_return`、return basis、interval metadata contract | 从现有 runner/turnover 推断真实 period return |
| fixture producer port | 接受显式 fixture observations/identity，纯内存生成候选 | 现有 runner hook、multi-trial sweep、真实 workspace/data read |
| manifest/seal | canonical bytes/hash、sealed bundle、verified result、fixture selection | 真实 research canonical、lineage store 写入 |
| lineage boundary | 输出 refs 与 `partial_lineage_blocked_audit` contract | 擦除 append-only event、调用 `fail()` 伪装回滚、原子批次实现 |
| future producer | 记录 runtime-high-risk 前置条件 | 在本 CR 选择文件/函数/插桩点 |

现有 `turnover.next_rebalance_date/net_forward_return` 是 `forward_label_proxy@v1`：label horizon 与 rebalance step 可不同、窗口可重叠，不能作为 `trial_portfolio_return_series@v1`。未来真实 producer 必须提供 observation start/end、持仓/权重、交易成本、return basis、non-overlap/alignment 与原子 lineage/correction 证据。

## 3. Source-touch 与文件 owner

| 路径 | 变更 | Owner |
|---|---|---|
| `engine/trial_return_artifact.py` | 新增 pure contract、fixture port、seal/verifier | S02 primary |
| `tests/research/test_cr172_trial_return_artifact.py` | 新增 schema/hash/binding/failure tests | S02 primary |
| `engine/mature_multifactor_research.py` | 不修改 | forbidden |
| `engine/experiment_family_lineage.py` / store | 不修改 | forbidden |
| `tests/research/test_trial_lineage_producer_adapters.py` | 不修改 | forbidden |

planned touch=`2`；owner unique=`2/2`；current runner hook/diff=`0/0`。

## 4. Public integration contract

| Public object / function | 精确合同 |
|---|---|
| `SealedTrialReturnBundleV1` | payload bytes + manifest + immutable `ArtifactSealV1`；不得含 host path identity |
| `ResearchCanonicalSelectionV1` | 本 CR 仅允许 `target_kind=repository_fixture` 的 fixture/in-memory selection |
| `canonical_artifact_seal_bytes` | 对 `ArtifactSealV1` 执行唯一 versioned canonical encoding |
| `canonical_artifact_seal_sha256` | `sha256:` + 上述 bytes 的 lowercase SHA-256 |
| `verify_sealed_trial_return_bundle` | 输入 bundle+selection；复算 payload/manifest/seal；返回 verified result或 typed BLOCKED |
| `VerifiedTrialReturnBundleV1` | 必含 `bundle`、`selection`、`original_seal_sha256`；digest 只能来自 verifier 的同一 canonical bytes |

S03 必须精确消费上述 3 个类型、2 个 governance 类型 `ActionDecisionV1`/`ActionScopeContextV1` 和 verifier；不得 reopen path、re-seal 或重算第二套 digest。

## 5. Fixture / real typed binding

12-field `ActionAuthorizationRecordV1` 不变。S01 派生 decision/context 合同：

- `decision_origin=repository_fixture|approved_ledger`；
- `target_kind=repository_fixture|real_operation`；
- `repository_fixture` 只绑定 `repository_fixture`、`fixture://` URI 和 repository-owned fixture/in-memory port；
- fixture decision + non-fixture target accepted=`0`；
- current-v1 对 `approved_ledger+real_operation` 也固定 authorized/eligible=`false/false`，reason=`APPROVED_LEDGER_ADAPTER_UNAVAILABLE`；可信 adapter 由独立 runtime-high-risk CR 引入，真实授权/执行=`0/6`,`0/6`。

## 6. 失败与 lineage 守卫

| Failure | 结果 |
|---|---|
| wrong kind / forward-label proxy | BLOCKED；fixture selection=0 |
| fixture decision + real target | deny before first side effect；accepted=0 |
| partial payload/manifest/seal | unverified；selection=0 |
| digest mismatch / tamper | BLOCKED；selection=0 |
| fixture producer port exception | typed BLOCKED；selection=0 |
| append-only lineage partial success | 不擦除、不伪装 rollback；输出 `partial_lineage_blocked_audit`；trial-return canonical selection=0 |

append-only atomic batch、outbox、correction/supersession 必须由独立 lineage-owner 前置 CR 承接。本 CR 不承诺“成功 lineage 痕迹回到 0”。

## 7. Story / merge / rollback

- S02（保留历史 Story ID）只拥有 pure artifact module/test；不再拥有 existing runner 或 lineage regression test。
- future native-producer CR 必须在 launch/workspace first side effect 前消费 `RunPathDecisionV1`，证明 new default=1、legacy write=0；本 CR 的 REQ-013 仅 contract-ready。
- merge order 仍为 `S01→S02→S03→S04→S05`；S03 依赖 S02 public contract，不依赖真实 producer evidence。
- rollback slice 是删除/禁用 fixture port 与新 pure module；现有 runner behavior diff=`0`。

## 8. 风险与 Gotchas

- 列名含 return 不证明业务语义；`forward_label_proxy@v1` 永不因 seal 而升级成 trial-return。
- immutable seal 不证明 source 是真实；fixture origin 必须贯穿 selection/replica/materialization receipts。
- append-only partial lineage 是审计事实，只能 BLOCKED，不可由 exception handler 撤销。
- Story ID 中的 `native-hook` 只保留审计稳定性，不表示当前 scope 或 hook 许可。
