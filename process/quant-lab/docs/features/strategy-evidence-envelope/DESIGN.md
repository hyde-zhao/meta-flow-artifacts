---
status: "ready-for-cp5-review"
version: "0.1"
cr_id: "CR-166"
feature_id: "FEAT-166-01"
owner: "meta-se-inline"
source_hld: "process/archive/design-cr-docs/HLD-WALK-FORWARD-OOS-EVIDENCE.md"
source_adr: "process/archive/design-cr-docs/ARCHITECTURE-DECISION-WALK-FORWARD-OOS-EVIDENCE.md"
---

# FEAT-166-01 Strategy Evidence Envelope 设计

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 0.1 | 2026-07-13 | host-orchestrator inline meta-se | 冻结 method-neutral canonical primitive、稳定 envelope、静态 component catalog 与 C1 兼容迁移。 |

## 1. 目标与边界

建立不属于 C1 statistical method 的公共证据基础层，使 C2 以及未来 C3/C4 在稳定 header 下独立演进。只提供 immutable value、canonical bytes/hash、availability 与 catalog 判断；不计算统计方法、walk-forward outcome、admission policy，不做 I/O、动态发现或持久化。

## 2. 模块与文件

| 文件 | 动作 | 职责 |
|---|---|---|
| `engine/strategy_evidence.py` | 新建 | neutral canonical、availability、descriptor、envelope、静态 catalog、自校验。 |
| `engine/statistical_evidence.py` | 修改 | 从 neutral 层 import/re-export canonical API；保留 C1 默认 domain 与返回字节/hash 兼容。 |
| `engine/walk_forward_oos_evidence.py` | 新建 | 仅在 S01 写入 C2 input/evidence value contracts；后续 S02/S03 串行扩展。 |

## 3. 公共数据模型

| 对象 | 必需字段 | 规则 |
|---|---|---|
| `EvidenceAvailability` | `present/typed_unavailable/not_applicable_with_reason/blocked` | 不与 outcome 混用；N/A 不能满足 mandatory C2。 |
| `ComponentDescriptor` | type、schema_version、required、ref、hash、availability | canonical inventory 按 type/version/ref 稳定排序。 |
| `StrategyEvidenceEnvelope` | schema/version、kind、subject_ref、components、provenance、authorization、limitations、reason_codes、hash | 不含路径存在性、mtime、current clock、凭据或 runtime state。 |
| `WalkForwardEvidenceInput` | 7 个字段族 | 只声明 shape；S02 负责充分性与泄漏校验。 |

## 4. Canonical 与兼容契约

- neutral `canonical_hash(value, *, domain)` 强制调用者提供 domain；输出 `sha256:<64 hex>`。
- UTF-8、递归 map key 排序、JSON-safe finite value、稳定 sequence；拒绝 NaN/Inf、set、非字符串 map key 与不支持对象。
- C1 `statistical_evidence.canonical_json_bytes` 名称、签名、默认 `SUMMARY_HASH_DOMAIN`、serialized bytes 与 golden hash 保持 100% 不变；compatibility wrapper 是唯一允许的默认 domain。
- neutral domain：`quant-lab.strategy-evidence-envelope.v1`；C2 component/input/config 使用独立显式 domain，不能复用 C1 domain。

## 5. 静态 Component Catalog

| type/version | 状态 | mandatory | optional |
|---|---|---|---|
| `walk_forward_oos@v1` | active | 正常校验 | 正常校验 |
| `economic_cost@reserved` | reserved，calculator=0 | typed_unavailable | audit-only，无 PASS 贡献 |
| `capacity_liquidity@reserved` | reserved，calculator=0 | typed_unavailable | audit-only，无 PASS 贡献 |
| unknown | unknown | blocked | round-trip audit-only，无 PASS 贡献 |

catalog 是源码常量，不提供注册 API、entry point、filesystem scan 或远程 discovery。

## 6. 调用与失败路径

S01 contracts → S02 validator → S03 producer → S04 projection。schema/version/hash 不一致、非有限值、mandatory unknown 直接 blocked；缺少 mandatory reserved component 为 typed_unavailable。任何 C1 golden 差异停止 neutral migration，保留旧 C1 wrapper，不能用更新 golden 掩盖回归。

## 7. NFR、安全与 Gotchas

- 10 次相同 normalized payload 产生 1 个 hash；外部操作数 0；生产依赖环 0。
- Gotcha：`component_ref` 是逻辑 content identity，不证明文件或 store 存在。
- Gotcha：optional unknown 可保留不等于可消费，更不等于 PASS。
- Gotcha：不要把 C1 `EvidenceStatus` 扩成 C2 availability；二者通过 compatibility wrapper 隔离。

## 8. 回退与重访

C1 bytes/hash 任一变化时停用 neutral re-export；envelope/C2 新模块可保留未消费状态。出现第三方 component、跨包发现或 artifact service 的已批准需求时重访动态 registry；C3/C4 calculator 不在本 Feature。
