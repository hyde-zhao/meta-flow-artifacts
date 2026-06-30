---
status: draft
version: "1.0"
cr_id: "CR-139"
source_use_cases: ["UC-51", "UC-52", "UC-53", "UC-54", "UC-55", "UC-56", "UC-57"]
source_scenarios: "process/SCENARIOS-CR139.yaml"
source_requirements: "process/REQUIREMENTS.md"
confirmed_by: ""
confirmed_at: ""
---

# Story Map — CR-139 Strategy Data Foundation

> CP2 阶段只给 Story 候选边界与 Feature 归属，**不拆 Story**（规则 25）。CP4 由 host-orchestrator + meta-se 基于 BLUEPRINT / DOMAIN-MAP / DEPENDENCY-MAP / FEATURE-DESIGN-MATRIX 拆分。

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-06-28 | meta-pm | 初始 Story Map 候选边界，按 Wave1/2/3 + Feature 归属 |

## 用户活动

| Activity ID | 用户活动 | 目标用户 | 业务目标 | 来源 |
|---|---|---|---|---|
| ACT-CR139-01 | 冻结整改基线与清册 | P-04 | 整改前冻结现状，可归因 | UC-51 |
| ACT-CR139-02 | 建立回测信任基础（四道 P0 防线） | P-03, P-04 | 回测/ML 不前视、不重复、冻结、可 join | UC-52 |
| ACT-CR139-03 | ML 接入数据湖与版本化 | P-03 | ML 可复现、不旁路、feature 版本化 | UC-53 |
| ACT-CR139-04 | 模拟盘日级推进与读前门禁 | P-04, P-07 | 日级增量、pointer 前移、读前阻断 | UC-54 |
| ACT-CR139-05 | 实盘可审计与交易审计链 | P-05, P-06 | 读审计、run-id 贯通、broker facts、成本前置 | UC-55 |
| ACT-CR139-06 | 配置类事实源版本化 | P-03, P-04 | benchmark/commission/universe/policy 版本化 | UC-56 |
| ACT-CR139-07 | schema 冻结与跨源时点一致性 | P-04, P-06 | 模拟盘前契约冻结、跨源对齐 | UC-57 |

## 用户任务与 Story 候选

| Story 候选 ID | Activity ID | 用户任务 | Story 候选 | 优先级 | 验收摘要 | Feature | 来源 |
|---|---|---|---|---|---|---|---|
| STC-CR139-01 | ACT-01 | 一行命令输出全 dataset 清册 | 作为 P-04，我希望一键生成整改对象清册，以便冻结基线 | P0 | T8 清册输出行数/覆盖/重复键/pit/published/lineage | FEAT-02 | REQ-213 / SCN-001 |
| STC-CR139-02 | ACT-01 | 冻结黄金值对比基准 | 作为 P-04，我希望冻结下游因子/回测黄金值，以便归因整改变化 | P0 | T7 黄金值快照可对比归因 | FEAT-02 | REQ-212 / SCN-001 |
| STC-CR139-03 | ACT-02 | 重复画像 | 作为 P-04，我希望看清 38 分区重复，以便决定去重策略 | P0 | C2a 画像输出重复键与来源 run | FEAT-02 | REQ-205 / SCN-007 |
| STC-CR139-04 | ACT-02 | 实现 published pointer / read selector | 作为 P-04，我希望 reader 只读已发布 current truth，以便冻结快照 | P0 | V1 pointer 固定，未发布阻断 | FEAT-02 | REQ-232 / SCN-004,005 |
| STC-CR139-05 | ACT-02 | PIT as-of reader | 作为 P-03，我希望读财报按 available_at 过滤，以便杜绝前视 | P0 | C1 as-of reader，未来财报读不到 | FEAT-02 | REQ-204 / SCN-002,003 |
| STC-CR139-06 | ACT-02 | 统一 panel reader | 作为 P-03，我希望多表 as-of join，以便 ML 不旁路 | P0 | R1 read_panel 输出统一 as-of 宽表 | FEAT-02/03 | REQ-214 / SCN-004 |
| STC-CR139-07 | ACT-02 | 读层去重 | 作为 P-04，我希望 (symbol,trade_date) 唯一，以便不污染标签 | P0 | C2b 按 source_run_id 取最新 | FEAT-02 | REQ-205 / SCN-006 |
| STC-CR139-08 | ACT-02 | N1 分区治理（读层优先，物理后置） | 作为 P-04，我希望新数据规范化分区，以便收敛 run_id 膨胀 | P0 | 新数据 dataset/schema_version，存量不重命名，物理迁移后置 | FEAT-02 | REQ-201,247 / SCN-022,024 |
| STC-CR139-09 | ACT-03 | ML 接入 lake 废除旁路 | 作为 P-03，我希望 ML 走 read_panel_as_of，以便数据湖对 ML 有效 | P1 | R2 废除 --data-dir/load_local_frames | FEAT-03 | REQ-215 / SCN-010 |
| STC-CR139-10 | ACT-03 | feature/label 版本化层 | 作为 P-03，我希望 feature 版本化持久化，以便可复现 | P1 | V3 features/ 子层带版本 + schema | FEAT-03 | REQ-219 / SCN-009 |
| STC-CR139-11 | ACT-03 | ExperimentManifest/model hash/split 闭环 | 作为 P-03，我希望实验 manifest 闭环 release，以便可复现 | P1 | E1/E2/E5 闭环 published release | FEAT-03/11 | REQ-221,222,225 / SCN-011 |
| STC-CR139-12 | ACT-03 | label 泄漏统一 gate + 离线在线一致 | 作为 P-03，我希望泄漏检查统一，以便不泄漏 | P1 | E3 统一 gate + E4 一致性校验 | FEAT-03 | REQ-223,224 / SCN-012 |
| STC-CR139-13 | ACT-03 | DuckDB 只读 adapter | 作为 P-03，我希望大查询用 DuckDB 只读，以便不撑爆内存 | P1 | R3 只读 adapter（CP3/CP5 批准） | FEAT-02 | REQ-216 |
| STC-CR139-14 | ACT-04 | 日级增量 + pointer 前移 | 作为 P-04，我希望日级 append + pointer 前移，以便模拟盘推进 | P1 | L1/L2 既有合同接通执行 | FEAT-02 | REQ-243,244 / SCN-013,023 |
| STC-CR139-15 | ACT-04 | 读前质量门禁 | 作为 P-04，我希望读前 readiness 阻断，以便不读坏数据 | P1 | L4 既有 readiness 前置为强制 gate | FEAT-02/14 | REQ-234 / SCN-014 |
| STC-CR139-16 | ACT-04 | V4 schema 冻结 | 作为 P-04，我希望模拟盘前 schema 冻结，以便契约稳定 | P1 | V4 演进规则 + 模拟盘前冻结 | FEAT-02/03 | REQ-220 / SCN-018 |
| STC-CR139-17 | ACT-05 | 读审计 + run-id 贯通 | 作为 P-06，我希望读审计与 run-id 贯通，以便可审计 | P1 | L3 读审计 log + T6 run-id 贯穿 | FEAT-02/11 | REQ-233,242 / SCN-015 |
| STC-CR139-18 | ACT-05 | broker facts 闭环 + 成本前置 | 作为 P-05，我希望 broker facts 闭环 + 成本前置，以便实盘可审计 | P1 | T4 闭环实盘写 + T5 成本门禁 | FEAT-06/12 | REQ-240,241 / SCN-016 |
| STC-CR139-19 | ACT-06 | 配置类事实源版本化 | 作为 P-03，我希望 benchmark/commission/universe 版本化，以便回测可复现 | P1 | F1/F2/F3 版本化 + release 闭环 | FEAT-02/12/14 | REQ-236,237,238 / SCN-017 |
| STC-CR139-20 | ACT-07 | 跨源时点一致性 | 作为 P-04，我希望跨源日历/复权 PIT/universe 一致，以便不偏差 | P1 | X1/X2/X3/X4 一致性门禁 | FEAT-02/14 | REQ-229,230,231,235 / SCN-019,020 |
| STC-CR139-21 | ACT-02 | 数据正确性/元数据收尾 | 作为 P-04，我希望 events schema/写入去重/catalog 主从/lineage 闭合 | P1 | C3/C4/M1/M2 + N2/N3 规约 | FEAT-02 | REQ-202,203,206,207,208,209 |
| STC-CR139-22 | ACT-07 | PIT/去重正确性测试 | 作为 P-03，我希望有回归测试，以便防退化 | P1 | T2/T3 正确性测试 | FEAT-02 | REQ-226,227 |

## 负向 / 边界 Story 候选

| Story 候选 ID | 触发场景 | 用户可见结果 | 验收摘要 | 来源 |
|---|---|---|---|---|
| STC-CR139-N01 | 未授权 runtime/lake write | 阻断 + 计数为 0 | 不授权范围计数全 0 | SCN-008 |
| STC-CR139-N02 | 未发布 candidate 读取 | catalog_not_published 阻断 | reader 不返回 candidate | SCN-005 |
| STC-CR139-N03 | PIT 前视 | as-of reader 读不到未来 | 未来财报断言通过 | SCN-003 |
| STC-CR139-N04 | label 泄漏 | cutoff gate 阻断 | 泄漏/不一致阻断 | SCN-012 |
| STC-CR139-N05 | decision_time lookahead | 强制阻断 | lookahead 违规阻断 | SCN-019 |
| STC-CR139-N06 | dry-run 合同未执行 | 结构化 not-executed | 不伪造执行 | SCN-023 |

## 非目标（CP2 不拆 Story）

- 本轮只给 Story 候选边界与 Feature 归属，CP4 由 host-orchestrator + meta-se 拆分。
- 不拆 Story ID、不写 LLD、不实现代码（规则 25）。
- Feature 归属以 BLUEPRINT v1.12 / FEATURE-DESIGN-MATRIX v1.11 为基线，CP3 由 meta-se 刷新。

## 追溯矩阵

| 来源 ID | 覆盖 Story 候选 | 覆盖状态 | 缺口 / 说明 |
|---|---|---|---|
| UC-51 | STC-01, 02 | covered | — |
| UC-52 | STC-03..08, 21 | covered | — |
| UC-53 | STC-09..13 | covered | — |
| UC-54 | STC-14, 15, 16 | covered | — |
| UC-55 | STC-17, 18 | covered | — |
| UC-56 | STC-19 | covered | — |
| UC-57 | STC-16, 20, 22 | covered | — |
| REQ-201..249 | STC-01..22 | covered | P2 项入 BACKLOG |
