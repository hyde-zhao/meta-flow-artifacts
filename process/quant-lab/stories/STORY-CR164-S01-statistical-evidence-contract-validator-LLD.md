---
story_id: "CR164-S01-statistical-evidence-contract-validator"
title: "Statistical evidence contract and validator"
story_slug: "statistical-evidence-contract-validator"
lld_version: "0.1"
tier: "L"
status: "confirmed"
confirmed: true
created_by: "host-orchestrator inline meta-dev"
created_at: "2026-07-12T20:25:00+08:00"
confirmed_by: "user"
confirmed_at: "2026-07-12T20:50:00+08:00"
feature_design_refs: ["docs/features/statistical-evidence-contract/DESIGN.md", "docs/features/statistical-evidence-contract/TEST-PLAN.md", "docs/features/statistical-evidence-contract/TASKS.md"]
lld_policy: {required_level: "full-lld", trigger_reasons: ["public-schema", "cross-story-contract", "lineage-integrity"], rationale: "Shared claim-sensitive contract."}
open_items: 0
---

# LLD: CR164-S01 — Statistical evidence contract and validator

## 0. 上游设计依据（工程依据）

CR164 HLD §5、ADR-001/002/007、FEAT-24 三件套、CR163 sealed lineage projection。

## 1. Goal

创建 `engine/statistical_evidence.py`，定义 immutable input/config/evidence/summary contracts、availability/reasons、normalizer/validator 和 canonical hash；测试位于 `tests/research/test_statistical_evidence_contracts.py`。

## 2. 需求（Requirements）

- Functional：四方法统一 envelope；binding/minima/domain validation；claim-specific mandatory set；raw/effective 分离。
- NFR：JSON-safe finite、10 runs→1 hash、orphan=0、无 I/O/runtime/network。

## 3. 模块拆分与职责

| 模块 | 职责 |
|---|---|
| contracts/enums | dataclasses、method/status/reason enums |
| normalizer | stable candidate/order/refs/config |
| validator | identity/minima/domain/orphan checks |
| canonicalizer | schema-versioned bytes/hash |

## 4. 代码结构与文件影响

| 动作 | 文件 | 内容 |
|---|---|---|
| 创建 | `engine/statistical_evidence.py` | contracts、validation、hash |
| 创建 | `tests/research/test_statistical_evidence_contracts.py` | positive/negative/golden fixtures |

禁止修改 CR163 lineage modules。

## 5. 数据模型与持久化

`StatisticalEvidenceInput` 含 family_ref/hash/raw_count/candidate_ids/membership_hash/method payloads；`MethodEvidence` 含 method/status/reasons/input_hash/config_hash/result/provenance/limitations；`Summary` 含 mandatory set/refs/final status/hash。无新增持久化；对象仅 JSON-safe value contracts。

## 6. API / Interface

| API | 输入 | 输出 | 调用方 |
|---|---|---|---|
| `normalize_statistical_evidence_input` | lineage projection + payload | input/status | S02/S03 |
| `validate_method_evidence` | evidence + expected input | validation | S04/S05 |
| `canonical_evidence_hash` | supported payload | SHA-256 | all stories |

## 7. 核心流程

lineage ref validation → membership/count check → method minima/domain → normalized immutable value → calculator → evidence validation → aggregation。absent 无冲突=unavailable；任何 mismatch/tamper/non-finite=blocked。

## 8. 技术细节

Canonical JSON：UTF-8、recursive sorted keys、stable sequence、no NaN/Inf、schema/version domain separator；排除 path/mtime/current clock。unknown fields/version blocked。Minima 直接编码为 policy constants，不允许 consumer 覆盖。

## 9. 安全与性能

No file/network/env/credential access；complexity 以 payload size 线性为目标；输入上限/详细性能不做生产声明。

## 10. 测试设计

valid all-method input；missing method payload；family hash/raw count/membership mismatch；NaN/Inf；candidate minima；DSR variance/sample；canonical key/order；10 hashes=1；orphan ref blocked。

## 11. 实施步骤

| TASK | 动作 | 文件 | 测试 |
|---|---|---|---|
| CR164-S01-T01 | 创建 enums/dataclasses | engine module | schema round-trip |
| CR164-S01-T02 | 创建 normalizer/validator | engine module | identity/minima fixtures |
| CR164-S01-T03 | 创建 canonicalizer | engine module | golden hash |
| CR164-S01-T04 | 创建 tests | test file | QAC-001..006 |

## 12. 风险与灰区

无 OPEN/LCQ。Gotchas：missing≠conflict；schema≠proof；membership 不可 shrink。若 exact statistical formula 需要新 fields，S02/S03 只能在 schema extension point 内增加，不得改 status/count ceiling。

## 13. 回滚与发布

实现后可删除/停用 CR164 producer/projection并恢复 typed_unavailable；不改 lineage。不授权发布。

## 14. DoD / Definition of Done

- [ ] public fields/status/reasons 完整；binding/minima/domain 100% covered
- [ ] 10 runs→1 hash；orphan=0；forbidden operations=0
- [ ] CP5 未确认前 `confirmed=false`、不实现
