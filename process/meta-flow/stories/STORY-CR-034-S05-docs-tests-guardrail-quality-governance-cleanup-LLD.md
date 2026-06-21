---
story_id: "STORY-CR-034-S05"
title: "docs, tests, guardrail, and quality governance cleanup"
story_slug: "docs-tests-guardrail-quality-governance-cleanup"
lld_version: "1.0"
tier: "M"
status: "confirmed"
confirmed: true
created_by: "host-orchestrator"
created_at: "2026-06-21T18:30:00+08:00"
confirmed_by: "user"
confirmed_at: "2026-06-21T18:36:21+08:00"
shared_fragments: []
feature_design_refs: ["process/changes/CR-034.md"]
lld_policy:
  required_level: "full-lld"
  trigger_reasons: ["quality-governance-cleanup", "docs", "tests", "guardrail"]
  rationale: "当前工作区已有 quality governance / doctor 相关源码和文档变更未被 active CR 持有，本 Story 负责把这些变更纳入 CR-034 并完成验证闭环。"
open_items: 0
---

# LLD: STORY-CR-034-S05 - docs, tests, guardrail, and quality governance cleanup

## 0. 上游设计依据

| 来源 | 路径 / ID | 被本 LLD 消费的内容 |
|---|---|---|
| CR | `process/changes/CR-034.md` | 当前未登记 quality governance 变更需纳入 CR-034 或前置清理 |
| Git status | 当前工作区 | `meta_flow/checks/quality_governance.py`、quality 模板、CLI/docs 已修改 |
| Quality governance | `meta_flow/checks/quality_governance.py` | `quality init/model-check/eval-check`、doctor quality/workflow |
| Guardrail | `scripts/check_delivery_guardrails.py` | 无 `__pycache__`、规则一致性检查 |

## 1. Goal

把已有 quality governance / doctor 相关变更纳入 CR-034 的正式范围，补齐文档、测试、guardrail 和发布说明，确保 CR-034 最终交付时没有未归属变更和运行副产物。

## 2. Requirements（Functional / Non-Functional）

### 2.1 Functional

- 明确当前已有 quality governance 变更属于 CR-034 的 S05 范围。
- 保留并验证 `meta-flow quality init/model-check/eval-check`、`doctor quality`、`doctor workflow`。
- 补齐 delivery 模板、README、USER-MANUAL、RELEASE-NOTES 的一致说明。
- 运行或记录无法运行的测试命令；当前环境 `pytest` 缺失需通过 `uv sync` 或项目推荐方式处理。
- 保证 `scripts/check_delivery_guardrails.py` PASS，清理 `__pycache__` 等运行副产物。

### 2.2 Non-Functional

- 不新增手工 `WORKFLOW-METRICS` 真相源；workflow metrics 只从 CP result、event ledger、read-expansion ledger 派生。
- 不引入额外运行时依赖；YAML 子集解析继续复用保守 parser。
- 文档和代码命令名必须一致。

## 3. 模块拆分与职责

| 模块 / 文件组 | 职责 | 说明 |
|---|---|---|
| `meta_flow/checks/quality_governance.py` | 质量模型 / eval 矩阵 / workflow doctor | 当前已有未提交变更 |
| `meta_flow/cli.py` | CLI 路由 | 当前已有 quality / doctor 路由变更 |
| `delivery/skills/context-manifest-builder/templates/*` | 模板交付 | 当前已有 QUALITY / EVAL 模板 |
| `tests/test_quality_governance.py` | 回归测试 | 当前已有新增测试文件 |
| `delivery/README.md` / `USER-MANUAL.md` / `RELEASE-NOTES.md` | 文档说明 | 当前已有修改 |

## 4. 代码结构与文件影响范围

| 动作 | 文件路径 | 变更内容 |
|---|---|---|
| 保留 / 修正 | `meta_flow/checks/quality_governance.py` | 检查质量模型、eval 矩阵、workflow metrics 派生来源 |
| 保留 / 修正 | `meta_flow/cli.py` | quality 命令和 doctor 子命令 |
| 保留 / 修正 | `delivery/skills/context-manifest-builder/SKILL.md` | 模板路由说明 |
| 创建 | `delivery/skills/context-manifest-builder/templates/QUALITY-MODEL-TEMPLATE.yaml` | 质量模型模板 |
| 创建 | `delivery/skills/context-manifest-builder/templates/EVAL-MATRIX-TEMPLATE.yaml` | eval 矩阵模板 |
| 保留 / 修正 | `tests/test_quality_governance.py` | 质量治理测试 |
| 修改 | 文档 | 用户可见说明 |

## 5. 数据模型与持久化设计

| 对象 / 字段 | 类型 | 约束 | 说明 |
|---|---|---|---|
| `process/policies/QUALITY-MODEL.yaml` | YAML 子集 | `source_policy=derived-only` | 质量模型 |
| `process/policies/EVAL-MATRIX.yaml` | YAML 子集 | 引用 quality model | eval 矩阵 |
| workflow metrics | 派生输出 | 不落独立真相源 | 来自 CP result / event ledger / read expansion |

## 6. API / Interface 设计

| 接口 / 入口 | 输入 | 输出 | 调用方 | 说明 |
|---|---|---|---|---|
| `meta-flow quality init --project-root .` | 项目根 | policy 文件 | 用户 / Host | 显式写入 |
| `meta-flow quality model-check --project-root .` | 项目根 | OK/FAIL | CI / Host | 校验模型 |
| `meta-flow quality eval-check --project-root .` | 项目根 | OK/FAIL | CI / Host | 校验矩阵 |
| `meta-flow doctor quality --project-root .` | 项目根 | 摘要 | adoption doctor | 复用 |
| `meta-flow doctor workflow --project-root .` | 项目根 | 派生指标 | adoption doctor | 不写 metrics 真相源 |

## 7. 核心处理流程

1. 确认已有 quality governance 变更纳入 CR-034 S05。
2. 校验模板与 runtime 默认内容一致。
3. 校验 CLI help、README、USER-MANUAL、RELEASE-NOTES 命令名一致。
4. 运行 quality / workflow doctor。
5. 处理测试环境：优先 `uv sync` 后 `uv run pytest`，或记录不可运行原因。
6. 清理 `__pycache__`，运行 delivery guardrail。

## 8. 技术设计细节

- 关键规则：workflow metrics 是派生报告，不创建 `WORKFLOW-METRICS` 文件。
- 依赖选择：不新增第三方依赖，继续使用现有 YAML 子集 parser。
- 兼容性处理：缺 ledger 时 `doctor workflow` 输出 WARN，不静默创建统计文件。
- 图示类型选择：不需要 Mermaid。

## 9. 安全与性能设计

| 维度 | 设计措施 | 验证方式 |
|---|---|---|
| 安全 | 不读取凭据，不访问外部服务 | 测试和代码审查 |
| 性能 | 只读取 policy、CP result、ledger 小文件 | 单测和 doctor 输出 |

## 10. 测试设计

| 测试场景 | 前置条件 | 操作 | 预期结果 | 验证方式 |
|---|---|---|---|---|
| quality init writes policies | tmp 项目 | `quality init` | 写入两份 policy | `test_quality_governance.py` |
| model-check rejects manual truth | policy 含 manual metrics | check | FAIL | 单测 |
| eval-check maps dimensions | eval matrix 引用未知 dimension | check | FAIL | 单测 |
| workflow doctor derives metrics | 有 CP result / ledgers | doctor | 输出派生指标 | 单测 |
| guardrail no cache | 运行后清理缓存 | guardrail | OK | 手工 / CI |

## 11. 实施步骤

| TASK-ID | 动作 | 目标文件 | 详细描述 | 对应测试 |
|---|---|---|---|---|
| TASK-CR034-S05-01 | 审核 | 当前 quality governance diff | 确认纳入 CR-034 范围，无越界 | review |
| TASK-CR034-S05-02 | 修正 | `quality_governance.py` / templates | 对齐模板、checker、doctor | `test_quality_governance.py` |
| TASK-CR034-S05-03 | 修正 | `cli.py` / docs | 保持命令名一致 | CLI help / guardrail |
| TASK-CR034-S05-04 | 验证 | tests / guardrail | 运行测试并清缓存 | pytest / guardrail |

## 12. 风险、难点与预研建议

### 12.1 实现灰区与取舍记录

| Clarification ID | 问题 | 选项与推荐 | 决策 / 答案 | 影响面 | 证据 | 重访条件 |
|---|---|---|---|---|---|---|
| LCQ-STORY-CR-034-S05-01 | 当前未提交 quality governance 变更是否全部纳入 CR-034 | 推荐全部纳入 S05；备选拆独立 CR，但会延迟 adoption readiness | 待 CP5 确认；非阻断 | 源码 / 文档 / 测试 | `git status --short` 显示相关变更 | 若发现与 adoption 无关内容，可拆分后续 CR |

| 风险 / 难点 | 影响 | 缓解措施 / 预研建议 |
|---|---|---|
| 测试依赖未同步 | 无法证明回归 | 先用 `uv sync`，若用户不授权则记录未运行风险 |
| 运行 Python 生成缓存 | guardrail 失败 | 测试后清理 `__pycache__` 或设置 `PYTHONDONTWRITEBYTECODE=1` |

### OPEN / Spike 跟踪

| ID | 类型（OPEN / Spike） | 问题 | 下一动作 | 责任方 |
|---|---|---|---|---|
| O-S05-01 | OPEN | 是否允许执行 `uv sync` 补测试依赖 | 实现 / 验证阶段按需请求授权或使用已同步环境 | host-orchestrator |

## 13. 回滚与发布策略

- 发布方式：随 CR-034 统一发布。
- 回滚触发条件：quality governance 命令破坏现有 CLI 或误报阻断。
- 回滚动作：移除 quality 命令路由，保留模板作为后续候选；文档回退。

## 14. Definition of Done

- [ ] quality governance 相关 diff 明确归属 CR-034。
- [ ] quality / workflow doctor 可运行并有测试覆盖。
- [ ] 文档、模板、CLI help 一致。
- [ ] guardrail PASS，且无 `__pycache__`。
- [ ] 测试命令已运行或记录未运行原因。

## 人工确认区

**人工审查结果回填**：

- 结论：`approved`
- 审查人：user
- 审查时间：2026-06-21T18:36:21+08:00
- 修改意见：无
- 风险接受项：不适用；不授权 runtime / quant-lab 写入 / CR-033
