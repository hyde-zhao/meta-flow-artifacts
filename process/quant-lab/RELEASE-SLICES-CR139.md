---
status: draft
version: "1.0"
cr_id: "CR-139"
---

# Release Slices — CR-139 Strategy Data Foundation

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-06-28 | meta-pm | 初始 Wave1/2/3 发布切片，对齐 handoff §5 |

> CP2 阶段为切片规划，不拆 Story。切片顺序反映 handoff v0.7 §5 评审重排。每切片前置 CP3/CP5 门控。

## Wave 1 — 冻结基线 + 信任基础（P0，解锁回测）

> 评审重排：先冻结现状基线和重复画像，再实现 published pointer/read selector，再做读层去重，最后做候选压缩/迁移。**不先改物理分区**。

| Slice ID | 顺序 | 内容 | 覆盖 REQ | Story 候选 | 前置 |
|---|---|---|---|---|---|
| SLICE-CR139-1 | 1 | T8 整改对象自动化清册 | REQ-213 | STC-01 | — |
| SLICE-CR139-2 | 2 | T7 整改回归基线 + 黄金值快照 | REQ-212 | STC-02 | SLICE-1 |
| SLICE-CR139-3 | 3 | C2a 重复画像 | REQ-205(a) | STC-03 | SLICE-2 |
| SLICE-CR139-4 | 4 | V1 published pointer / read selector | REQ-232 | STC-04 | SLICE-3 |
| SLICE-CR139-5 | 5 | C1 PIT as-of reader | REQ-204 | STC-05 | SLICE-4 |
| SLICE-CR139-6 | 6 | R1 统一 published/as-of panel reader | REQ-214 | STC-06 | SLICE-5 |
| SLICE-CR139-7 | 7 | C2b 读层去重 | REQ-205(b) | STC-07 | SLICE-6 |
| SLICE-CR139-8 | 8（后置） | N1 候选压缩/物理分区迁移 | REQ-201,247 | STC-08 | SLICE-2 黄金值归因后 |

**Wave1 出口**：四道 P0 防线就绪，回测信任基础建立，物理迁移后置。

## Wave 2 — 接入与规模（P1，解锁 ML + 模拟盘）

> 评审 HIGH3：V4 schema 契约冻结必须在模拟盘前完成，从 Wave3 移入 Wave2。

| Slice ID | 顺序 | 内容 | 覆盖 REQ | Story 候选 | 前置 |
|---|---|---|---|---|---|
| SLICE-CR139-9 | 9 | V4 schema 演进 + 实盘契约冻结（模拟盘前必过） | REQ-220 | STC-16 | Wave1 |
| SLICE-CR139-10 | 10 | R2 ML 接入 lake 废除旁路 | REQ-215 | STC-09 | SLICE-9 |
| SLICE-CR139-11 | 11 | V3 feature/label/artifact 层 + E1/E2/E5 | REQ-219,221,222,225 | STC-10,11 | SLICE-10 |
| SLICE-CR139-12 | 12 | R3 DuckDB 只读 adapter | REQ-216 | STC-13 | CP3/CP5 批准 |
| SLICE-CR139-13 | 13 | L1/L2 日级增量 + pointer 前移 | REQ-243,244 | STC-14 | SLICE-9 |
| SLICE-CR139-14 | 14 | E3/E4 label 泄漏 + 离线/在线一致 | REQ-223,224 | STC-12 | SLICE-11 |
| SLICE-CR139-15 | 15 | F1/F2/F3 配置类事实源层 | REQ-236,237,238 | STC-19 | SLICE-9 |
| SLICE-CR139-16 | 16 | T4/T5/T6 broker facts + 成本前置 + run-id 贯通 | REQ-240,241,242 | STC-18 | SLICE-9 |
| SLICE-CR139-17 | 17 | X1/X2/X3/X4 跨源与时点一致性 | REQ-229,230,231,235 | STC-20 | SLICE-9 |
| SLICE-CR139-18 | 18 | L3/L4 读审计 + 读前质量门禁 | REQ-233,234 | STC-15,17 | SLICE-9 |
| SLICE-CR139-19 | 19 | T2/T3 PIT/去重正确性测试 | REQ-226,227 | STC-22 | SLICE-5,7 |
| SLICE-CR139-20 | 20 | N1/C3/M1 收尾 | REQ-201,206,208 | STC-21 | SLICE-8 |

**Wave2 出口**：ML 接入 + 模拟盘门禁就绪（V4 冻结 + 读前门禁 + 配置事实源 + 审计链）。

## Wave 3 — 实盘与运维（P2）

| Slice ID | 内容 | 覆盖 REQ | 前置 |
|---|---|---|---|
| SLICE-CR139-21 | L5 replay | REQ-245 | Wave2 |
| SLICE-CR139-22 | R4 延迟优化 | REQ-217 | SLICE-12 |
| SLICE-CR139-23 | T1 DuckDB e2e 测试 | REQ-228 | SLICE-12 |
| SLICE-CR139-24 | M3/M4 quality 整理 + CR 审计链 | REQ-210,211 | Wave2 |
| SLICE-CR139-25 | F4 政策周期/shortability 版本化 | REQ-239 | SLICE-15 |

**Wave3 出口**：实盘与运维能力补齐（P2 deferred）。

## 阻塞前置

- 所有切片前置 CP3 通过（BLUEPRINT/DOMAIN-MAP/DEPENDENCY-MAP 刷新 + companion HLD + FEATURE-DESIGN-MATRIX + 四组门禁集落为 CP3 门禁）。
- Wave1 物理迁移（SLICE-8）前置 T7 黄金值归因。
- DuckDB 切片（SLICE-12/22/23）前置 CP3/CP5 依赖批准。
- 真实执行类（L1/L2/T4 实盘写/L3 读审计写）前置独立 runtime_authorization。
