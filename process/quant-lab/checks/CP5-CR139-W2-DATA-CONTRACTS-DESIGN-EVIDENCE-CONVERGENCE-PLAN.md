---
checkpoint: "CP5"
cr_id: "CR-139"
batch_id: "CR139-W2-DATA-CONTRACTS"
title: "CR139 W2 Data Contracts Design Evidence Convergence Plan"
type: "cp5-batch-preparation"
status: "ready-for-design-evidence-convergence"
created_at: "2026-06-28T23:14:15+08:00"
created_by: "host-orchestrator"
context_capsule: "process/context/CP5-CR139-W2-DATA-CONTRACTS-LLD-CONTEXT.yaml"
formal_cp5_checkpoint: "process/checkpoints/CP5-CR139-W2-DATA-CONTRACTS-LLD-BATCH-REVIEW.md"
implementation_allowed: false
runtime_allowed: false
---

# CR139-W2-DATA-CONTRACTS 设计证据收敛计划

## 目标

本计划启动 CR139 后续 CP5 批次准备，默认选择 `CR139-W2-DATA-CONTRACTS`。目标是在 Wave1 S01-S08 verified 后，先收敛最容易影响后续 ML、读门禁、审计链和配置事实源的底层数据契约，再进入任何 Wave2/W3 实现。

本计划不是正式 CP5 通过结论。它不把任何 W2/W3 Story 标记为 `dev-ready`，也不授权实现、runtime、真实 lake 写入、provider catalog 写入、published pointer advance execution 或物理分区迁移执行。

## Entry Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| process 路由健康 | PASS | `process/.meta-flow-process.yaml`；`process/STATE.md` 可读 | `process` 为外置 artifact symlink |
| Active CR | PASS | `process/state/STATE.current.json` | `active_change=CR-139`，`blocked=false` |
| Wave1 完成 | PASS | `process/STORY-STATUS-CR139.md` | S01-S08 全部 `verified` |
| 后续批次推荐存在 | PASS | `process/context/CR139-WAVE1-COMPLETE-FOLLOWUP-CONTEXT.yaml` | 推荐首批为 `CR139-W2-DATA-CONTRACTS` |
| 禁止项明确 | PASS | `process/state/STATE.current.json`；本计划 | runtime、真实 lake 写入、provider catalog 写入、pointer advance execution、物理迁移均未授权 |

## 批次范围

| Story | 设计证据类型 | 当前状态 | 依赖 | 文件 / 契约焦点 | 收敛输出 |
|---|---|---|---|---|---|
| S09 | full-lld | lld-pending | S01/S02/S03/S07 | `engine/contracts.py`、`market_data/readers.py` | 14 章 LLD，冻结 schema 演进与 reader 兼容回退 |
| S14 | technical-note | lld-pending | S04/S08 | `incremental.py`、`publish.py`、`normalization.py`、`cli.py` | 技术说明，强调 pointer advance 只设计不执行 |
| S22 | technical-note | lld-pending | S04/S14 | `publish.py`、`manifest.py`、`evidence_index.py` | 技术说明，定义 FEAT-02 owner 的 run-id 贯通 |
| S30 | technical-note | lld-pending | S08 | `normalization.py`、`lake_layout.py` | 技术说明，新 run_id 规约、旧路径保留 |
| S31 | technical-note | lld-pending | S01/S02/S03 | `normalization.py` events 分支 | 技术说明，events schema 类型修复，不执行真实重跑 |
| S32 | technical-note | lld-pending | S07 | `normalization.py`、`validation.py` | 技术说明，写入去重或校验失败阻断 |
| S33 | technical-note | lld-pending | S04 | `catalog.py`、`manifest.py` | 技术说明，catalog 为真相源、manifest 派生、CR014/CR018 命名清理 |
| S34 | technical-note | lld-pending | S33 | `manifest.py`、`normalization.py` | 技术说明，lineage_checksum 回填 |
| S39 | technical-note | lld-pending | S08/S33 | `catalog.py` | 技术说明，`triggered_by_cr`、`run_lineage` 审计链 |

设计证据分布：`full-lld=1`（S09），`technical-note=8`（S14/S22/S30/S31/S32/S33/S34/S39）。

## 收敛顺序

| 顺序 | 对象 | 原因 | 阻塞关系 |
|---:|---|---|---|
| 1 | S09 | schema 演进和 reader 兼容回退是后续模拟盘、PIT universe 和 DuckDB/read-path 工作的前置契约 | 依赖 Wave1 verified；不依赖 S14/S33 |
| 2 | S30/S31/S32 | 都消费 Wave1 S08/S07/基线门，且主要在 `normalization.py` 写侧清理；先冻结命名、events schema、写入去重的边界 | `normalization.py` 顺序 S31 -> S32 -> S34 -> S14 |
| 3 | S33 | `catalog.py` 的 source-of-truth 和 CR014/CR018 命名清理必须先于 S34/S39 | catalog.py 顺序 S04 -> S33 -> S39 |
| 4 | S34 | lineage_checksum 回填消费 S33 的 catalog/manifest 定主 | S33 完成后 |
| 5 | S14 | 增量 append 与 pointer plan 消费 S08 路径治理和 S33/S34 元数据约束；只设计，不执行 pointer advance | S04/S08 已 verified；建议在 S31/S32/S34 后冻结 normalization 切片 |
| 6 | S22 | run-id 贯通消费 S14 的增量/publish run-id 输出 | S14 完成后 |
| 7 | S39 | CR -> data audit chain 消费 S33 source-of-truth 与 S08/S30 命名/路径约束 | S33 完成后，可与 S22 设计并行但不能并行改 `catalog.py` |

## 文件所有权与 merge order

| 文件 | 本批 Story | merge order / 设计顺序 | CP5 要求 |
|---|---|---|---|
| `market_data/readers.py` | S09 | S05 -> S06 -> S07 已 verified；本批 S09 下一位 | S09 LLD 必须只声明兼容回退切片，不抢占 S25/S27/S28/S13/S16/S36 |
| `market_data/publish.py` | S14/S22 | S04 已 verified；S14 -> S22 | S14 设计 pointer plan；S22 消费 run-id lineage；均不得执行 pointer advance |
| `market_data/catalog.py` | S33/S39 | S04 已 verified；S33 -> S39 | S33 是 CR014/CR018 命名清理 owner；S39 只加审计链字段 |
| `market_data/normalization.py` | S31/S32/S34/S14 | S08 已 verified；S31 -> S32 -> S34 -> S14 | 每个技术说明必须写出切片边界和 forbidden writes |
| `market_data/manifest.py` | S33/S34/S22 | S33 -> S34 -> S22 | S33 定主，S34 lineage checksum，S22 lineage 消费 |

## S33 验收补充：catalog.py CR014/CR018 命名清理

`market_data/catalog.py` 当前仍包含历史 CR 命名 API 和 helper，例如 `CR014_CATALOG_POINTER_REQUIRED_FIELDS`、`CR018_*` 常量、`build_cr018_*()` 与 `_cr018_*()` helper。S33 的技术说明必须新增验收补充：

| 补充验收 | 要求 |
|---|---|
| 领域命名 API | 设计新的领域命名 public API / constants，用 catalog、release、current pointer、publish evidence 等语义命名，不再把新 API 绑定到 CR014/CR018 |
| 兼容 alias | 保留 CR014/CR018 旧命名 alias，避免破坏既有测试和历史合同 |
| 测试覆盖 | 现有 CR014/CR018 测试继续通过；新增领域命名测试覆盖同一行为 |
| 弃用路径 | 明确旧 CR 命名 alias 的 deprecation 说明和迁移顺序 |
| S39 边界 | S39 只负责 `triggered_by_cr`、`run_lineage`，不得承担旧 API 改名主体 |
| 禁止项 | 不得执行 provider catalog 写入、published pointer advance、真实 lake 写入或物理迁移 |

## Clarification Queue 预检查

| 队列项 | 分类 | 状态 | 处理 |
|---|---|---|---|
| S33 领域命名 API 精确命名 | non-blocking-open | planned | meta-dev 写 S33 技术说明时给出推荐名称、兼容 alias、测试入口；如发现破坏性 public API 风险，转 CP5 DQ |
| S14 pointer advance 执行边界 | resolved-by-user | fixed | 本批只设计，不执行；`approve` 不授权 pointer advance execution |
| S08 物理迁移边界 | resolved-by-user | fixed | S08 verified 不等于物理迁移授权；本批不执行物理迁移 |
| runtime / provider / real lake | resolved-by-user | fixed | 本批全部禁止；需要时另走 runtime_authorization |

当前无 `blocks_lld=true` 的已知未回答项。正式 CP5 发起前必须重新扫描实际 S09 LLD 与八个 technical-note 的 OPEN / LCQ / O-* 项。

## Exit Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| 批次范围冻结 | PASS | 本文件；CP5 Decision Brief | 默认选择 `CR139-W2-DATA-CONTRACTS` |
| 设计证据待产出 | PENDING | Story 卡片 / S09 LLD | S09 LLD 和八个技术说明尚需生成/刷新 |
| S33 验收补充已登记 | PASS | 本文件；S33 Story 卡片补充 | 后续 S33 技术说明必须消费 |
| 正式 CP5 可发起 | BLOCKED | 无完整设计证据 | 等设计证据、per-story precheck、clarification queue 收敛后再发起正式 CP5 approval |

## 不授权项

本批次准备和后续 `approve` 均不授权：

- runtime
- NAS / QMT / MiniQMT / gateway runtime
- trading / small_live / live
- credential / secret read
- provider catalog 写入 / provider-lake-catalog 写入
- 真实 lake 写入
- published pointer advance execution
- physical partition migration execution
- Git remote write

任何真实运行、真实写入、pointer 前移或物理迁移必须另走 `runtime_authorization`，写明 action scope、运行窗口、脱敏、回滚、审计范围和停止条件。

## 下一步

1. meta-dev 或 inline-fallback 按本计划生成 S09 full LLD 与 S14/S22/S30/S31/S32/S33/S34/S39 technical-note。
2. 生成 per-story `CP5-CR139-Sxx-*-LLD-IMPLEMENTABILITY.md` 预检。
3. 重新生成正式 `CP5-CR139-W2-DATA-CONTRACTS-LLD-BATCH.result.json` 与 Decision Brief，只有无阻断项时才发起人工确认。
