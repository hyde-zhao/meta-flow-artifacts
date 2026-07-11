---
status: draft-for-cp5
version: "1.0"
feature_id: "FEAT-20"
feature_name: "Experiment-family lineage core"
source_blueprint: "docs/design/BLUEPRINT-TRIAL-LINEAGE-INSTRUMENTATION.md"
source_hld: "docs/design/HLD-TRIAL-LINEAGE-INSTRUMENTATION.md"
source_adr: "docs/design/ARCHITECTURE-DECISION-TRIAL-LINEAGE-INSTRUMENTATION.md"
source_matrix: "docs/design/FEATURE-DESIGN-MATRIX.md#cr163-cp4-增量trial-lineage-instrumentation"
related_stories: ["CR163-S01-family-contract-validator", "CR163-S02-recorder-seal-supersession", "CR163-S05-integrity-recovery-permission-regression"]
lld_policy_summary: "S01/S02/S05 full-lld"
---

# Feature Design: Experiment-family Lineage Core

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-11 | meta-se-critical | 冻结 core module、六对象、session/command、storage/seal/validation、失败与回退边界。 |

## 摘要

| 项目 | 内容 |
|---|---|
| Feature 目标 | 为未来 native producer 提供 pre-search、append-only、可封存和可验证的 family raw lineage 真相源。 |
| 推荐方案 | `engine/experiment_family_lineage.py` public contract/validator + `engine/experiment_family_lineage_store.py` recorder/canonical/sealer。 |
| 关键取舍 | 优化审计、确定性和本地可验证性；接受 single-writer 文件模型，DB/registry 后置。 |
| 非目标 | effective trial/FDR/PBO/DSR、historical backfill、real runtime/data/external registry。 |

## 上游依据与边界

| 来源 | 消费内容 |
|---|---|
| HLD §5-7 | 六对象、façade、commands、states/count、canonical JSON/JSONL、seal/supersession |
| ADR-001..008 | single-run 分离、session+commands、storage/hash/availability/manual count |
| REQ-CR163-001..004,007,008 | ordering、append-only、count、seal/recovery、authorization |

Core 唯一拥有 FamilySpec、event ledger、manifest versions 与 validation result。Producer 只能提交 commands；consumer 只能读取 validation projection；`ExperimentManifest`/`BacktestRunSpec` 只提供 run identity refs。

## 现有代码与模块变更

| 区域 | 路径 | 当前职责 | 规划变更 |
|---|---|---|---|
| 新 public contract | `engine/experiment_family_lineage.py` | 不存在 | 创建六对象、enums、commands、session façade、pure validator/projection DTO |
| 新 storage/seal | `engine/experiment_family_lineage_store.py` | 不存在 | 创建 local recorder、canonical bytes、SHA-256 sealer、supersession resolver |
| 单次 run contracts | `engine/research_manifest.py`、`engine/backtest_production_contracts.py` | run-level identity | read-only dependency；CR163 不在 S01/S02 修改 |

## 公共数据与接口

| Interface | Caller → Callee | Input | Output | Failure |
|---|---|---|---|---|
| `FamilyLineageSession.open` | producer → core | spec, chain id, run/experiment refs, root | session / DeclareFamily receipt | post-hoc, invalid spec, forbidden root |
| `session.submit` | producer → recorder | typed command with event_id/family_id/sequence/payload | idempotent receipt | identity-content conflict, orphan, illegal transition |
| `session.seal` | producer → sealer/validator | family/version/prior head | manifest + validation | incomplete/count/hash/ref/chain blocked |
| `validate_family_lineage` | auditor/adapter → validator | target ref/hash + artifacts | `FamilyLineageValidationResult` | typed unavailable or machine blocked reasons |
| `project_family_evidence` | consumer adapter → core | manifest + matching validation | availability/ref/raw count/effective unavailable | target mismatch/tamper blocked |

持久化对象固定为 `ExperimentFamilySpec`、`ExperimentTrial`、`TrialAttempt`、`TrialSelection`、`ExperimentFamilyManifest`、`FamilyLineageValidationResult`；`FamilyLineageSession` 不持久化为第七对象。

## 状态、identity 与 count

- Family：absent→declared→recording→sealed；新合法 head 使旧 head superseded。
- Trial：declared→active→succeeded/failed/cancelled/excluded，或 never-started-with-reason；全部计数。
- Attempt：declared→running→terminal；retry 新 attempt ordinal。
- event id + identical canonical payload 幂等；same id + different payload blocked。
- stable trial id = family + normalized parameters + seed；`raw_trial_count=count(distinct trial_id)`。

## Canonical storage/seal

- spec JSON create-only；events JSONL append-only；manifest/validation按 version 新建。
- UTF-8、recursive key sort、no insignificant whitespace、no NaN/Infinity、schema-defined decimal normalization。
- events sort `(sequence,event_id)`；SHA-256 domain含 schema/spec/events/family/version/prior ref+hash；排除 path/mtime/current clock/hash itself。
- create-exclusive；旧 sealed path 不覆盖。supersession 必须 prior ref/hash/reason、单调 version、无断链/循环。

## 失败、权限与回退

| Failure | 行为 | 回退/恢复 | 测试 |
|---|---|---|---|
| trial before declaration | blocked_post_hoc | 不补造；新 native run | SC-N01 |
| duplicate conflict/orphan | block seal，保留 evidence | append correction + superseding version | N02/R01 |
| count/hash/chain mismatch | availability blocked | 修正 command + new version | B01/T01/R01 |
| no native instrumentation | typed_unavailable | producer adapter 后续接入 | N01/G01 |
| forbidden operation counter nonzero | blocked | stop；独立授权 | A01 |

Rollback 只停止新 command/consumer projection，已写 events/manifests 不删除；schema 修复创建新 version。

## Story 与实现顺序

| Story | 进一步 LLD 必须冻结 | 依赖 |
|---|---|---|
| S01 | dataclass/enum/command/result fields、error codes、pure validation/projection、module import boundary | none |
| S02 | exact file layout/API、canonical byte golden vector、atomic create/append、supersession resolver | S01 contract |
| S05 | core fixtures、tamper/recovery/permission negative matrix | S01/S02 + adapters |

## 下游消费契约

- story-manager：三个 Story 绑定本 DESIGN/TEST-PLAN/TASKS，均 full-lld。
- lld-designer：不得重开 CP3 选择；只冻结字段、函数、文件、fixture 和 error enum。
- meta-dev：CP5 未批准前不得实现。
- meta-qa：使用 TEST-PLAN 的 12 场景 traceability，不访问 real data/runtime。

## 风险与 Gotchas

- `sealed_at` 可记录但不得进 hash；validation 必须绑定 target ref/hash。
- atomic replace 不能覆盖旧 sealed version。
- session 不得 close-time snapshot 吞掉中间失败事件。
- `not_applicable_with_reason` 不能用于 included producer mapping。
- single-writer 假设失效时必须新 storage ADR/CR，不能静默加 DB。

