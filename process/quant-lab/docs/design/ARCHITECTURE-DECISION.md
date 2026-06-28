---
status: "current-index"
version: "1.5"
change: "CR-138"
legacy_source: "process/ARCHITECTURE-DECISION.md"
current_change_sources:
  - "process/archive/design-cr-docs/ARCHITECTURE-DECISION-CR046.md"
  - "process/archive/design-cr-docs/ARCHITECTURE-DECISION-CR053.md"
  - "process/archive/design-cr-docs/RUNNER-CORE-MVP-DESIGN-CR126.md"
  - "process/docs/features/strategy-runner-core/DESIGN.md"
  - "process/docs/design/ARCHITECTURE-DECISION-RUNNER-QMT-OPERATIONAL-CONTROL-PLANE.md"
---

# Architecture Decision Current Index

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-06-07 | meta-po | 新增 ADR 长期索引，按 Feature / Epic 聚合 legacy ADR 范围 |
| 1.1 | 2026-06-13 | meta-po | 按 CR-046 增补双目标策略交付框架 ADR 入口 |
| 1.2 | 2026-06-14 | host-orchestrator | 按 CR-053 增补 quant-lab migration inventory / dry-run ADR 入口 |
| 1.3 | 2026-06-23 | host-orchestrator | CR130 收口 runner architecture authority：CR046 ADR 标注为 legacy cross-target cluster，新增 runner core authority 行。 |
| 1.4 | 2026-06-23 | host-orchestrator | CR131 将 CR 命名 ADR / runner source design 移出默认 design surface，历史入口统一指向 `process/archive/design-cr-docs/`。 |
| 1.5 | 2026-06-24 | host-orchestrator | CR138 按功能域新增 Runner / QMT Gateway Operational Control Plane ADR 入口，覆盖 FEAT-11 / FEAT-12、REST-only Gateway P0、按需 runtime 授权和交易日历 / 佣金 / 收益查询服务。 |

## 定位

正式 ADR 正文仍在 `process/ARCHITECTURE-DECISION.md`。本文只提供按能力域聚合的 current index，帮助后续 CR 快速定位决策簇。

## ADR 聚合索引

| Feature | ADR 范围 | 主题 |
|---|---|---|
| FEAT-01 本地研究与轻量回测核心 | ADR-001..012、ADR-023..029 | 数据准备与回测隔离、轻量主路径、复权口径、T+1 成交、manifest、quality、research input、benchmark 字段隔离 |
| FEAT-02 生产级市场数据湖 | ADR-013..022、ADR-030..035、ADR-048..054、ADR-062..066 | Tushare 写湖、dataset readiness、生产数据湖、publish gate、全 A current truth、DuckDB readonly、复权双视图、rollback |
| FEAT-03 多因子研究闭环 | ADR-036..043、ADR-079..086 | benchmark/PIT/tradability/execution/adjustment/exposure/capacity/factor audit、FactorSpec、FactorRunSpec、LabelWindow、StrategyAdmissionPackage |
| FEAT-04 执行语义与可选后端 | ADR-074..078 | Backtrader optional semantic reference、module reference/no-copy、order intent draft、依赖隔离 |
| FEAT-05 QMT C/S Gateway 与只读准入 | ADR-067..073、ADR-087..093 | legacy strategy readiness admission、QMT C/S bridge、HMAC pairing、endpoint matrix、gateway runtime、login/session ready、query_positions readonly |
| FEAT-06 OMS / 风控 / Broker Lake / 阶段激活 | ADR-055..061 | QMT adapter、broker lake、OMS、pre-trade risk、stage gate、reconciliation、kill switch、cross-node deployment |
| FEAT-07 安全授权治理 | ADR-051、ADR-061、ADR-071..073、ADR-086、ADR-090..093 | 真实操作授权、redaction、HMAC / scope、fallback、no-real-operation、安全边界 |
| FEAT-08 文档 / Runbook | ADR 派生，不单独编号 | README、USER-MANUAL、QMT runbook 和 CP8 只作为用户操作与审计入口，不提供 runtime authorization |
| FEAT-09 QMT / MiniQMT 双目标策略交付框架 | ADR-CR046-001..006 | legacy cross-target ADR cluster；保留独立 FEAT-09、平台无关策略核心、MiniQMT runner 安装设计、验证证据分级、CR047/CR051 后置的历史决策，不授权恢复 offline runner implementation |
| FEAT-11 Runner Control Plane | ADR-CR138-001、ADR-CR138-003、ADR-CR138-005、ADR-CR138-007 | Runner 运营控制面独立于 offline runner core；Runner/Gateway 领域命令与事件契约；CP3 不自动授权 runtime，后续按需授权；当前保留单份 umbrella HLD，CP4 拆 Story |
| FEAT-12 QMT Gateway Service Layer | ADR-CR138-002、ADR-CR138-004、ADR-CR138-005、ADR-CR138-006、ADR-CR138-007 | Gateway 作为独立服务层；P0 REST-only；按需 runtime 授权；交易日历、佣金和收益查询归入 Query Service；CP4 拆 Gateway Story |
| Runner Core Authority | archived CR126 source design + CR128 / CR129 closure + `strategy-runner-core` | offline runner implementation authority 归 `process/docs/features/strategy-runner-core/DESIGN.md`；`process/archive/design-cr-docs/RUNNER-CORE-MVP-DESIGN-CR126.md` 只作为 CR128 implementation intake/source design；CR046 ADR 不授权 runtime/NAS/QMT/provider/lake/catalog/trading |
| FEAT-10 Strategy Research Lifecycle / quant-lab Migration Governance | ADR-CR053-001..005 | NAS 逻辑目录映射、manifest-first 数据传输、warm/cold 备份、真实迁移 CR058 后置、交易主机只读 package exchange |

## 使用规则

| 判断问题 | 读取方式 |
|---|---|
| 是否已有架构决策 | 先查上表，再读 `process/ARCHITECTURE-DECISION.md` 对应 ADR |
| 是否允许新增依赖 | 查对应 ADR 和 `docs/design/DEPENDENCY-MAP.md`；默认不允许外部框架 / QMT / DuckDB 依赖变更 |
| 是否允许真实操作 | ADR 只能给出设计边界；真实操作必须另有 CP / per-run authorization |
| 审查 CR046 ADR | 读取 `process/archive/design-cr-docs/ARCHITECTURE-DECISION-CR046.md` |
| 审查 CR053 ADR | 读取 `process/archive/design-cr-docs/ARCHITECTURE-DECISION-CR053.md` |
| 审查 offline runner implementation authority | 先读 `process/docs/features/strategy-runner-core/DESIGN.md`，再按需读 `process/archive/design-cr-docs/RUNNER-CORE-MVP-DESIGN-CR126.md` |
| ADR 与蓝图冲突 | 以 legacy ADR 正文和 HLD 事实为准，修订蓝图索引 |

## Runner ADR Authority Boundary

CR046 ADR 保留为 QMT / MiniQMT 双目标策略交付框架的历史决策簇，不再作为 offline runner core 的实现权威。任何从 CR046 ADR 恢复 MiniQMT runner install、真实 readonly runtime、NAS package exchange、provider/lake/catalog 或 trading write 的动作，都必须另起独立授权门禁。
