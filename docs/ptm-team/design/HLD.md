# PTM Team 高层设计（HLD）

> 版本：v1.0 · 更新：2026-06-08 · 状态：引用蓝图作为权威来源

---

## 概述

本文档是 ptm-team 项目的**项目级高层设计索引**。完整架构设计见 `docs/ptm-team-blueprint.md`（权威来源）。本文档提供快速导航和各 Agent 的 HLD 引用。

## 架构总览

### 三层架构

```
规划层（Planning）
  通用大模型 + Agent → 读取测试用例 → 规划测试步骤
  调用执行层工具执行测试行为，调用感知层工具 Check 执行结果

执行层（Execution）
  单个可执行工具 → TGFW 配置 / 组网切换 / 流量生成 / 设备管理

感知层（Perception）
  多模态感知 → 文本（日志、命令行）/ 图像（截图）/ 表格（性能数据）
  对照用例 Check 点给出 Pass/Fail 判定和失败原因摘要
```

### 六 Agent 体系

| Agent | 层级 | 状态 | HLD |
|---|---|---|---|
| ptm-tm | 规划层 | ⬜ 未开始 | — |
| ptm-tse | 规划层 | ⬜ 未开始 | — |
| ptm-tde | 规划层 | ✅ 已交付 | `docs/ptm-tde/HLD.md` |
| ptm-te | 执行层 | ⬜ 未开始 | — |
| ptm-tae | 执行层/基础设施 | 🔄 进行中 | — |
| ptm-qa | 感知层 | ⬜ 未开始 | — |

## 关键架构决策

| ADR | 决策 | 状态 | 文档 |
|---|---|---|---|
| ADR-01 | 渐进落地：Skill → 工作流 → Agent | confirmed | `docs/ptm-team-blueprint.md` §1.2 |
| ADR-02 | 分层解耦：规划层/执行层/感知层独立演进 | confirmed | 同上 |
| ADR-03 | 共建共用：公共 Skill 由 ptm-tae 统一维护 | confirmed | 同上 |
| ADR-04 | ptm-tde 基于 MFQ&PPDCS 方法论 | confirmed | `docs/ptm-tde/ARCHITECTURE-DECISION.md` |
| ADR-05 | 原子操作采用 Python CLI + git 离线优先策略 | confirmed | `research/atomic-operations/sources/HLD.md` |

## 数据流

```
需求文档 → ptm-tse（分析+策略）
         → ptm-tde（12 步流程 → 测试方案+用例）
         → ptm-te（解析→配置→打流→判定→执行记录）
         → ptm-tae（脚本翻译→工厂注册→回归）
         → ptm-qa（审计→数据采集→趋势报告）
         → ptm-tm（汇总→报告→风险推送）
```

## 技术栈

| 组件 | 技术 |
|---|---|
| Agent 框架 | Claude Code / Codex / Qoder Agent / Skill |
| Python 环境 | uv（`pyproject.toml` + `uv.lock`） |
| 原子操作 CLI | Python CLI（`atomic-ops`，`uv tool install` 分发） |
| 知识库 | YAML schema（因子库、接口依赖关系） |
| 协作 | git（版本控制）、禅道（任务管理）、钉钉（通知） |

## 各 Agent HLD

- **ptm-tde**：`docs/ptm-tde/HLD.md` — 三阶段框架（KYM → MFQ → PPDCS）、12 步主流程、5 个人工检查点
- 其余 Agent HLD 待各自开发启动时创建

## 相关文档

- 权威蓝图：`docs/ptm-team-blueprint.md`
- ptm-tde 架构决策：`docs/ptm-tde/ARCHITECTURE-DECISION.md`
- 测试策略：`docs/quality/TEST-STRATEGY.md`
