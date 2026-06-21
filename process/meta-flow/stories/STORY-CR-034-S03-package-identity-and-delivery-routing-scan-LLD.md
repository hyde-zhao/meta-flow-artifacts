---
story_id: "STORY-CR-034-S03"
title: "package identity and delivery routing scan"
story_slug: "package-identity-and-delivery-routing-scan"
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
  trigger_reasons: ["package-identity", "delivery-routing", "production-project-boundary"]
  rationale: "目标项目 adoption 前必须识别 repo/package/import/CLI/public API 和 README/docs 交付约定，避免把 meta-flow 默认目录强加到 production 项目。"
open_items: 0
---

# LLD: STORY-CR-034-S03 - package identity and delivery routing scan

## 0. 上游设计依据

| 来源 | 路径 / ID | 被本 LLD 消费的内容 |
|---|---|---|
| CR | `process/changes/CR-034.md` | package identity、delivery routing、target project profile 目标 |
| Product governance | `meta_flow/design/product_governance.py` | `PACKAGE-IDENTITY.yaml` init/check 现有实现 |
| Rules | `AGENTS.md` | production 项目必须先扫描 README/docs/交付约定，不默认写 `delivery/` |
| Tests | `tests/test_product_governance.py` | package identity 检查已有覆盖 |

## 1. Goal

让 meta-flow 能在目标项目 adoption 前盘点 package identity 和 delivery routing，明确项目名、包名、import 名、CLI 名、public API、legacy aliases、README/docs 交付约定和禁止写入路径。

## 2. Requirements（Functional / Non-Functional）

### 2.1 Functional

- 提供 identity init/check 的 adoption 视角报告，当前缺失时给出明确初始化路径。
- 扫描 `pyproject.toml`、顶层 package 目录、README、docs 目录和常见交付说明，生成 delivery routing 建议。
- 区分 meta-flow self-dev 与 production adoption；production 默认不得写当前仓库 `delivery/`。
- 输出 target project profile 建议字段：`project_kind`、`validation_target.sut_type`、`delivery_routing.mode`、`output_root`、`forbidden_roots_when_production`。
- 不读取凭据，不扫描隐藏凭据文件内容。

### 2.2 Non-Functional

- 可解释：每个 identity / routing 结论都有来源路径。
- 保守：冲突时输出 FAIL / decision item，不猜测写入路径。
- 可测试：tmp fixture 可构造 pyproject、README、包目录。

## 3. 模块拆分与职责

| 模块 / 文件组 | 职责 | 说明 |
|---|---|---|
| `meta_flow/design/product_governance.py` | 扩展 identity scan / routing suggestion | 复用现有 identity init/check |
| `meta_flow/checks/adoption_readiness.py` | 消费 identity / routing 结果 | S01 聚合显示 |
| `tests/test_product_governance.py` | 扩展 package identity / routing 测试 | 保持现有测试风格 |
| `delivery/skills/blueprint-design/templates/PACKAGE-IDENTITY-TEMPLATE.yaml` | 模板对齐 | 若模板与 init 默认值漂移，需同步 |

## 4. 代码结构与文件影响范围

| 动作 | 文件路径 | 变更内容 |
|---|---|---|
| 修改 | `meta_flow/design/product_governance.py` | 增加 delivery routing scan / target profile 建议函数 |
| 修改 | `meta_flow/cli.py` | 暴露 `identity scan` 或纳入 adoption doctor |
| 修改 | `tests/test_product_governance.py` | 增加 README/docs/pyproject routing fixture |
| 修改 | `delivery/skills/blueprint-design/templates/PACKAGE-IDENTITY-TEMPLATE.yaml` | 必要时同步字段 |
| 修改 | 文档 | 说明 package identity 和 delivery routing 的 adoption 规则 |

## 5. 数据模型与持久化设计

默认只读输出，不自动写 production 项目的正式 routing 决策。用户明确 bootstrap 时才允许写入 process 状态或 context。

| 对象 / 字段 | 类型 | 约束 | 说明 |
|---|---|---|---|
| `PACKAGE-IDENTITY.yaml.product_name` | str | 非空 | 产品名称 |
| `repo_name` | str | 可与 pyproject 比对 | 仓库名 |
| `python_import` | str | package_mode=true 时路径存在 | Python import 根 |
| `cli_name` | str | 可与 pyproject scripts 比对 | CLI 命令 |
| `delivery_routing.mode` | enum | `project-readme-contract` / `proposed-output` / `meta-flow-delivery` | 交付出口来源 |
| `forbidden_roots_when_production` | list[str] | 非空 | 禁止默认写入的路径 |

## 6. API / Interface 设计

| 接口 / 入口 | 输入 | 输出 | 调用方 | 说明 |
|---|---|---|---|---|
| `scan_delivery_routing(project_root) -> DeliveryRoutingReport` | 项目根 | routing 建议 | adoption doctor | 不写文件 |
| `identity_main(["scan", ...])` | CLI 参数 | 人类报告 | 用户 / Host Orchestrator | 可选；也可只被 adoption doctor 调用 |
| `validate_package_identity(project_root)` | 项目根 | errors/warnings | 已存在 | 继续复用 |

## 7. 核心处理流程

1. 读取 `PACKAGE-IDENTITY.yaml`；缺失则 FAIL 并提示 `meta-flow identity init/check`。
2. 读取 `pyproject.toml` 中 project name、scripts。
3. 检查 import package 路径和 public API 文件。
4. 扫描 README / docs 的交付、发布、安装、构建关键词和路径。
5. 若找到明确项目约定，输出 `project-readme-contract`；否则输出 `proposed-output` 并要求人工确认。
6. 将结论传给 adoption doctor 和 CP0/bootstrap context。

## 8. 技术设计细节

- 关键规则：scan 只提供建议，不把建议直接写入 production 目标项目正式产物。
- 依赖选择：继续使用标准库 `tomllib` 和保守文本扫描，不引入 YAML 依赖。
- 兼容性处理：`docs/design` 在 meta-flow 源码仓库可能不存在；production 项目缺失时只提示初始化，不创建。
- 图示类型选择：不需要 Mermaid。

## 9. 安全与性能设计

| 维度 | 设计措施 | 验证方式 |
|---|---|---|
| 安全 | 不读取 `.env`、credential、secret；只读 README/docs/pyproject/package 文件名和公开文档 | 单测构造 `.env` sentinel，确认未读取 |
| 性能 | 限定 README/docs 浅层扫描，避免全仓库递归 | 单测使用有限 fixture |

## 10. 测试设计

| 测试场景 | 前置条件 | 操作 | 预期结果 | 验证方式 |
|---|---|---|---|---|
| package identity missing | 缺 `PACKAGE-IDENTITY.yaml` | identity/adoption check | FAIL | 单测 |
| pyproject mismatch | pyproject name 与 repo_name 不一致 | identity check | FAIL | 已有 + 扩展测试 |
| README defines docs output | README 写明 docs/release | routing scan | `project-readme-contract` | 新单测 |
| no delivery convention | 只有 pyproject | routing scan | `proposed-output` + decision required | 新单测 |

## 11. 实施步骤

| TASK-ID | 动作 | 目标文件 | 详细描述 | 对应测试 |
|---|---|---|---|---|
| TASK-CR034-S03-01 | 修改 | `product_governance.py` | 增加 routing scan 数据结构和函数 | `test_delivery_routing_*` |
| TASK-CR034-S03-02 | 修改 | `cli.py` | 暴露 scan 或纳入 doctor | CLI stdout 测试 |
| TASK-CR034-S03-03 | 修改 | `tests/test_product_governance.py` | 增加 routing fixture | pytest |
| TASK-CR034-S03-04 | 修改 | 文档 / 模板 | 同步 package identity adoption 说明 | guardrail |

## 12. 风险、难点与预研建议

### 12.1 实现灰区与取舍记录

| Clarification ID | 问题 | 选项与推荐 | 决策 / 答案 | 影响面 | 证据 | 重访条件 |
|---|---|---|---|---|---|---|
| LCQ-STORY-CR-034-S03-01 | routing scan 结果是否自动写入 STATE | 推荐不自动写，只进入 context / Decision Brief；备选 bootstrap 时写入 STATE.current refs | 待 CP5 确认；非阻断 | 状态 / 人工门禁 | production 路由需用户确认 | 若后续目标项目 adoption 频繁重复，可新增显式 `--write` |

| 风险 / 难点 | 影响 | 缓解措施 / 预研建议 |
|---|---|---|
| README 文本扫描误判 | 错误建议输出路径 | 只作为建议，需 human gate 确认 |
| package identity 默认值过于简单 | 复杂仓库误报 | 支持 legacy_aliases 和 public_api_files 手工修正 |

### OPEN / Spike 跟踪

| ID | 类型（OPEN / Spike） | 问题 | 下一动作 | 责任方 |
|---|---|---|---|---|
| O-S03-01 | OPEN | 是否增加 `identity scan` 独立命令 | CP5 采用推荐：先纳入 adoption doctor，独立命令可后续扩展 | host-orchestrator |

## 13. 回滚与发布策略

- 发布方式：随 CLI / docs 发布。
- 回滚触发条件：routing scan 大量误报或阻断正常项目。
- 回滚动作：保留 identity check，禁用 routing scan 的 FAIL，仅降级为 WARN。

## 14. Definition of Done

- [ ] 缺失 / 冲突 package identity 能被 adoption doctor 明确报告。
- [ ] README/docs delivery routing 只形成建议，不自动写 production 目标项目。
- [ ] target project profile 字段和禁止写入路径能进入 CP0/context/human gate 输入。
- [ ] 相关 tests 和 docs 更新完成。

## 人工确认区

**人工审查结果回填**：

- 结论：`approved`
- 审查人：user
- 审查时间：2026-06-21T18:36:21+08:00
- 修改意见：无
- 风险接受项：不适用；不授权 runtime / quant-lab 写入 / CR-033
