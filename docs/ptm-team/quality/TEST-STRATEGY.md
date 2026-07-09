---
validation_mode: mixed
applies_to: ptm-team 全部 6 个 Agent
note: 语义见 §2 验证模式；本字段为规则 12 要求的机器可读声明
---

# PTM Team 测试策略

> 版本：v1.0 · 更新：2026-06-08 · 适用范围：ptm-team 全部 6 个 Agent

---

## 1. 测试分层

| 层级 | 说明 | 适用对象 | 当前状态 |
|---|---|---|---|
| **L0 静态检查** | lint、format、结构检查、guardrail | 全部 Skill/Agent/脚本 | ptm-tde 已覆盖 |
| **L1 单元/契约测试** | Skill 输入输出契约、Prompt 模板变量 | 全部 Skill | ptm-tde CR 级 CP6 已覆盖 |
| **L2 集成测试** | Agent 流程端到端、Skill 调用链 | ptm-tde 12 步流程 | CR 级 CP7 已覆盖 |
| **L3 dry-run / 人工审查** | 场景覆盖、语义质量、Gate 门控 | 人工检查点 | ptm-tde 5 个人工检查点 |
| **L4 运行时验证** | 真实防火墙环境测试 | ptm-te/ptm-tae | 不授权（依赖硬件） |

## 2. 验证模式

| 模式 | 说明 | ptm-tde 应用 |
|---|---|---|
| `static-only` | 仅静态检查，无需运行时环境 | CP6 编码完成检查 |
| `review-only` | 仅人工语义审查 | CP3/CP5/CP8 人工门控 |
| `dry-run-only` | 模拟执行，不接触真实环境 | Skill 输入输出验证 |
| `mixed` | 组合多种模式 | CP7 验证完成检查 |

> ptm-tde 当前验证模式为 `mixed`（static + dry-run + review），**不包含运行时验证**。运行时验证依赖防火墙硬件/拓扑/仪表，不在本交付范围内。

## 3. 各 Agent 测试策略

### ptm-tde（已交付）

| 维度 | 策略 |
|---|---|
| Skill 单元 | 每个 Skill 的输入输出契约通过 CP6 检查 |
| 流程集成 | 12 步主流程 + 扩展分支通过 CR 级 CP7 验证 |
| 人工审查 | 5 个检查点通过 CP3/CP5/CP8 人工门控 |
| 覆盖追溯 | SCENARIOS.yaml → TEST-MATRIX.md 待 P2 创建 |

### ptm-tm/tse/te/tae/qa（待开发）

| 维度 | 策略 |
|---|---|
| 启动标准 | 各自启动时重新执行 CP0-CP2 流程 |
| 测试标准 | 遵循本策略 L0-L3 分层 |
| 验证证据 | CP6/CP7 检查点 + Agent Dispatch Evidence |

## 4. 质量门控

| 门控 | 准出条件 | ptm-tde 状态 |
|---|---|---|
| CP3 HLD | 自动预检 PASS + 人工 approved | ✅ 4/4 CR 通过 |
| CP5 LLD | 全量 Story 设计证据 batch approved | ✅ 4/4 batch 通过 |
| CP6 编码 | 单 Story PASS，Agent Dispatch Evidence | ✅ 28/28 Story 通过 |
| CP7 验证 | PASS/PASS_WITH_RISK/WAIVED | ✅ 12/12 全局通过 |
| CP8 交付 | 自动预检 PASS + 人工 approved | ✅ 4/4 CR 通过 |

## 5. 风险与限制

| 风险 | 等级 | 缓解 |
|---|---|---|
| CP0-CP5 gate_inheritance 未在本仓库重跑 | MEDIUM | 后续发现基线问题发起 CR 回溯 |
| 低风险 Story 缺少 IMPLEMENTATION.md | LOW | CR 级 CP6 已覆盖，P1 补充 |
| 无真实运行时验证环境 | MEDIUM | 不在交付范围内，明确不授权 |
| 其余 5 个 Agent 测试策略待定 | LOW | 各自启动时执行 CP0-CP2 |

## 6. 测试工具

| 工具 | 用途 | 状态 |
|---|---|---|
| `scripts/check_delivery_guardrails.py` | 交付护栏静态检查 | ptm-tde 已集成 |
| `uv run --python 3.11 python <script>` | Python 脚本验证 | 可用 |
| `git diff --check` | 空白字符检查 | 可用 |
| atomic-ops CLI | 原子操作查询验证 | 可用（79 ops） |

---

*本策略覆盖 ptm-team 全部 6 个 Agent。ptm-tde 已按此策略完成交付，其余 Agent 启动时参考执行。*
