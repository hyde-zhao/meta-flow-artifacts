---
story_id: "CR166-S01-evidence-envelope-contracts"
title: "Strategy evidence envelope and C2 contracts"
story_slug: "evidence-envelope-contracts"
lld_version: "0.1"
tier: "L"
status: "confirmed"
confirmed: true
created_by: "host-orchestrator inline meta-dev"
created_at: "2026-07-13T12:30:00+08:00"
confirmed_by: "user"
confirmed_at: "2026-07-13T13:43:35+08:00"
feature_design_refs: ["docs/features/strategy-evidence-envelope/DESIGN.md", "docs/features/strategy-evidence-envelope/TEST-PLAN.md", "docs/features/strategy-evidence-envelope/TASKS.md"]
lld_policy: {required_level: "full-lld", trigger_reasons: ["public-schema", "cross-module-canonical-contract", "C1-hash-compatibility"], rationale: "Shared neutral contract and compatibility migration are claim-sensitive."}
open_items: 0
---

# LLD: CR166-S01 — Strategy Evidence Envelope and C2 Contracts

## 0. 上游设计依据（工程依据）

CR166 HLD §5.1/5.2、ADR-001/002/009、Blueprint FEAT-166-01、Domain OBJ-166-01..03、FEAT-166-01 三件套。CP3、CP5 均已批准。

## 1. Goal

新增 method-neutral `engine/strategy_evidence.py` 与 C2 contract skeleton，抽取 canonical primitives，同时通过 `engine/statistical_evidence.py` wrapper/re-export 保持 C1 public name/signature/default domain/bytes/hash 100% 兼容。

## 2. 需求（Requirements）

- Functional：availability、descriptor、stable envelope、static catalog、C2 7-field input shape、unknown decision table。
- Compatibility：C1 `canonical_json_bytes` / `canonical_hash` 的 import 与 golden output 差异=0。
- NFR：finite-only、10 runs→1 hash、operation=0、production cycle=0。

## 3. 模块拆分与职责

| 模块 | 职责 | 禁止 |
|---|---|---|
| `strategy_evidence` | canonical、domain hash、availability、descriptor、envelope、catalog/self-validation | statistical method、walk-forward calculation、I/O |
| `walk_forward_oos_evidence` S01 slice | C2 input/evidence dataclass shape、schema constants | validator/producer/projection |
| `statistical_evidence` wrapper | C1 default domain 和 API 兼容 | C2 import C1 method types |

## 4. 代码结构与文件影响

| 动作 | 文件 | 精确内容 |
|---|---|---|
| 新建 | `engine/strategy_evidence.py` | `EvidenceAvailability`、`ComponentDescriptor`、`StrategyEvidenceEnvelope`、`canonical_json_bytes`、required-domain `canonical_hash`、catalog/self-validation。 |
| 新建 | `engine/walk_forward_oos_evidence.py` | C2 schema constants 与 immutable input/output value types。 |
| 修改 | `engine/statistical_evidence.py` | import neutral canonical；用 wrapper 保留 `SUMMARY_HASH_DOMAIN` 默认值与 exports。 |
| 新建 | `tests/research/test_walk_forward_oos_contracts.py` | C2/envelope/catalog/canonical fixtures。 |
| 修改 | `tests/research/test_statistical_evidence_contracts.py` | C1 public API/bytes/hash golden compatibility。 |

## 5. 数据模型与持久化

`StrategyEvidenceEnvelope` 字段：schema/version、evidence_kind、subject_ref、ordered component descriptors、logical provenance、authorization summary、limitations/reasons、inventory/envelope hash。`ComponentDescriptor` 含 type/version/required/ref/hash/availability。无持久化：ref 是 logical content identity，不检查文件/store。

## 6. API / Interface

| API | 输入 | 输出 | 错误 |
|---|---|---|---|
| neutral `canonical_json_bytes` | supported JSON-safe value | UTF-8 stable bytes | unsupported/non-finite→`ValueError/TypeError` |
| neutral `canonical_hash` | value + required domain | `sha256:*` | blank domain→error |
| `validate_strategy_evidence_envelope` | envelope | typed issues/status | hash/schema/catalog mismatch→blocked |
| C1 wrapper `canonical_hash` | value + optional legacy domain | existing C1 hash | 语义不变 |

## 7. 核心流程

构造 immutable value → normalize finite JSON → catalog classify → canonical descriptor inventory → component/inventory/envelope hash → self-validation。C1 path 仅 wrapper 传入旧默认 domain；C2 path 必须显式 domain。

## 8. 技术细节与失败策略

map key 排序、UTF-8、compact separators；tuple/list 按 schema 保持序列；set 与非字符串 key 拒绝。mandatory unknown=blocked；reserved mandatory=typed_unavailable；optional unknown preserve-only/zero contribution。任何 C1 golden 差异阻断迁移，禁止“更新 golden”。

## 9. 安全与性能

stdlib only；不读取 path/env/credential/network；对 payload 单遍 normalization，目标 `O(n)`。不提供 plugin entry point、runtime registry 或 store writer。

## 10. 测试设计

contract round-trip；status separation；C1 import/signature/default/golden；10-run hash；component reorder；tamper old hash；NaN/Inf/set/key rejection；active/reserved/unknown table；C3/C4 calculator=0。

## 11. 实施步骤

| TASK | 动作 | 验证 |
|---|---|---|
| CR166-S01-T01 | neutral enums/canonical | finite/golden tests |
| CR166-S01-T02 | descriptor/envelope/catalog | catalog/unknown tests |
| CR166-S01-T03 | C2 value contracts | 7/7 schema round-trip |
| CR166-S01-T04 | C1 wrapper/re-export | compatibility difference=0 |
| CR166-S01-T05 | full contract regression | 10→1、tamper blocked、operation=0 |

## 12. 风险与 Gotchas

无 OPEN/LCQ。Gotchas：logical ref≠persistence；optional unknown≠supported；availability≠outcome；C1 status 不扩为 C2 availability；domain string 属于公共兼容契约。

## 13. 回滚与发布

C1 compatibility 不通过时恢复旧 C1 local canonical 实现，保留未消费 neutral/C2 module；不迁移数据、无 release/runtime 操作。

## 14. DoD / Definition of Done

- [x] schema/catalog/unknown table 完整，7/7 字段可表达
- [x] C1 API/default domain/bytes/hash 差异=0
- [x] 10 runs→1 hash，tamper/mandatory unknown false PASS=0
- [x] CP5 已批准，`confirmed=true`；允许进入 repository-local 实现
