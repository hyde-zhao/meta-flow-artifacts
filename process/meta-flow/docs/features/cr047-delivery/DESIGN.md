---
status: ready-for-review
version: "1.0"
feature_id: "FEAT-WT-04"
feature_name: "Deterministic Delivery Gate"
source_blueprint: "process/docs/design/CR047-WORKFLOW-TRUTH-BLUEPRINT.md"
source_hld: "process/docs/design/CR047-WORKFLOW-TRUTH-HLD.md"
source_adr: "process/docs/design/CR047-WORKFLOW-TRUTH-ARCHITECTURE-DECISION.md"
source_matrix: "process/docs/design/CR047-FEATURE-DESIGN-MATRIX.md"
related_stories: ["ST-WT-004", "ST-WT-005"]
lld_policy_summary: "full-lld=2"
---

# Feature Design: Deterministic Delivery Gate

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-14 | host-orchestrator-inline / meta-se | 冻结 tracked rule source、cache classifier、Ruff/pytest 与现有组合 preflight。 |

## 摘要与边界

| 项目 | 内容 |
|---|---|
| 目标 | clean archive 与测试后本机 tree 都能得到确定、可解释的交付门结论。 |
| 推荐方案 | 扩展 `check_delivery_guardrails.py` 的 typed checks；组合现有 truth/Doctor/Ruff/pytest/install，不新建 orchestrator。 |
| 下游 Story | ST-WT-004、ST-WT-005 |
| 非目标 | 提交 ignored 根 wrapper、忽略所有 cache、以 lint 代替测试。 |

## 上游依据与现有代码

| 来源/区域 | 路径 | 消费内容 / 当前职责 |
|---|---|---|
| ADR | `ADR-WT-004` | tracked canonical、cache precedence、组合层只编排 |
| Guardrail | `scripts/check_delivery_guardrails.py` | 规则、缓存、交付资产检查 |
| Rules | `delivery/rules/AGENTS.md`, `.gitignore` | canonical source 与 generated wrapper 边界 |
| Quality config | `pyproject.toml`, `uv.lock` | Ruff/pytest 环境 |
| Source/tests | `meta_flow/**`, `tests/**` | 90 项 Ruff 修复与回归 |

## 推荐方案与接口

| 设计点 | 做法 | 失败行为 |
|---|---|---|
| root rule | guardrail 检查 tracked `delivery/rules/AGENTS.md`；generated root 只在安装目标验证 | clean archive 缺 ignored root 不失败；canonical 缺失/漂移失败 |
| cache | tracked 或 package-input cache=BLOCKER；pure ignored local cache=WARN/cleanup candidate | staging/package 命中立即升级 blocker |
| Ruff | 安全 auto-fix 分批审查，B/F 类人工处理 | Ruff 非零阻断 |
| pytest | `PYTHONDONTWRITEBYTECODE=1` + no cache provider，至少 377+70 | 任一失败或计数低于基线阻断 |
| composition | 现有 checker 各自输出；preflight 串行聚合 refs/exit | 不复制底层判断，不用手工“全绿”声明 |

## DAG、文件 owner 与流程

- ST-WT-004 primary：guardrail/rule/cache contract。
- ST-WT-005 primary：repo-wide Ruff 修复与 lint/test composition；对 ST-WT-004/003 的 shared Python 文件使用 file-conflict 依赖，最后合并。
- 流程：clean archive guardrail → local ignored-cache fixture → Ruff → pytest → typed result/Run event。

## 失败、回退与安全

自动修复前保留 diff；遇到语义变化或 flaky regression 立即停止 auto-fix。回退按 Story 文件所有权恢复局部变更，保留新的失败测试。禁止运行外部发布、credentials、runtime 或 push。

## 测试与下游契约

- clean archive 缺 root wrapper仍 PASS；tracked canonical 缺失 FAIL。
- ignored/tracked/package cache 三类 fixture。
- Ruff 0；pytest 计数及失败传播；组合门单步失败不可被后步覆盖。
- LLD 必须逐项列出 auto-fix/人工修复边界与共享文件 merge order。

## 风险与 Gotchas

- ignored 不等于安全进入 package；判定必须结合 tracked/staging/package set。
- Ruff 自动修复不能与跨模块功能实现并行写同一文件。
- clean archive PASS 与工作树 WARN 是不同输入，不应强行得到相同 warning 数。
