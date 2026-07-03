---
status: draft
version: "1.0"
feature_id: "FEAT-PG-002"
feature_name: "Ledger Compaction"
source_blueprint: "process/docs/design/META-FLOW-PROJECT-GOVERNANCE-BLUEPRINT.md"
source_hld: "process/docs/design/META-FLOW-PROJECT-GOVERNANCE-HLD.md"
source_adr: "process/docs/design/META-FLOW-PROJECT-GOVERNANCE-ARCHITECTURE-DECISION.md"
source_matrix: "process/docs/design/META-FLOW-PROJECT-GOVERNANCE-FEATURE-DESIGN-MATRIX.md"
related_stories:
  - "CR037-S04-ledger-compaction-policy-and-cli"
lld_policy_summary: "retention policy, archive/index, CLI and rollback require full-lld"
confirmed_by: ""
confirmed_at: ""
---

# Feature Design: Ledger Compaction

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-02 | host-orchestrator | CP3 approved 后生成 ledger compaction 的 per-feature 设计基线 |

## 摘要

| 项目 | 内容 |
|---|---|
| Feature 目标 | 为线性增长的 NDJSON ledger 提供可审计 rolling / compaction 执行器，避免历史事件长期堆积为默认读取和维护负担。 |
| 推荐方案 | 新增 `meta-flow ledger compact`，定义独立 retention / archive policy；保留近期窗口全量，旧事件压缩为 summary + archive index。 |
| 关键取舍 | 不复用 `meta-flow state compact`，不把读取策略 `default_context` 当成压缩策略。 |
| 下游 Story | `CR037-S04-ledger-compaction-policy-and-cli` |
| LLD 策略 | full-lld；涉及归档、回滚、CLI、ledger 兼容和审计。 |

## 背景与问题

| 问题 ID | 背景 | 触发场景 | 影响 | 若不设计的风险 |
|---|---|---|---|---|
| P-LC-01 | `CR-LEDGER`、`GATE-LEDGER` 等 NDJSON 线性增长 | 长期项目多 CR / 多 gate | 检索慢、默认审计成本高 | ledger 成为下一个膨胀源 |
| P-LC-02 | `state compact` 已存在但语义是 render + check | 用户寻找压缩命令 | 命令名混淆 | 误以为 state compact 会压缩 ledger |
| P-LC-03 | `default_context` 是读取策略，不是 archive 策略 | 直接套用 retention | 压缩规则不可审计 | 丢失恢复窗口或归档不一致 |

## 上游依据与输入

| 来源 | 路径 / ID | 被本设计消费的内容 |
|---|---|---|
| HLD | `process/docs/design/META-FLOW-PROJECT-GOVERNANCE-HLD.md` | P0.5 ledger hygiene，命令边界 |
| Feature Matrix | `process/docs/design/META-FLOW-PROJECT-GOVERNANCE-FEATURE-DESIGN-MATRIX.md` | FEAT-PG-002 required，full-lld，TC-PG-017 |
| Product Baseline | `process/docs/product/REQUIREMENTS.md` / `TEST-MATRIX.md` | REQ-PG-C005、TC-PG-017 |

## 目标与非目标

| 类型 | 内容 | 来源 |
|---|---|---|
| Goal | 定义 `keep_latest_n_cr`、`window_days`、`archive_rule`、`dry_run`、`restore_ref` | HLD P0.5 |
| Goal | compact 后仍能通过 event checker 或 legacy compatibility checker | CP2 / CP3 |
| Goal | 输出 archive index，记录压缩前后计数、hash 和恢复路径 | 审计要求 |
| Non-Goal | 不改变 `STATE.current.json` compact / render 语义 | REQ-PG-C005 |
| Non-Goal | 不新建 `PROJECT-LEDGER` | CP2 / CP3 评审结论 |

## Feature 边界与相邻对象

| 对象 | 本 Feature 负责 | 不负责 | 相邻 Feature / 模块 | 边界判定依据 |
|---|---|---|---|---|
| NDJSON ledgers | rolling、archive、index、dry-run、restore metadata | 改变事件业务语义 | event_ledger | ledger hygiene |
| `state compact` | 在帮助文案中澄清不压缩 ledger | 重命名或改变 state compact | state current | 命名边界 |
| ROADMAP-REFRESH ledger event | 可被通用 compact 处理 | 定义 refresh result schema | FEAT-PG-006 | 不造 PROJECT-LEDGER |

## 现有代码位置

| 区域 | 路径 | 当前职责 | 变更方式 |
|---|---|---|---|
| Event ledger definitions | `meta_flow/state/event_ledger.py` | ledger path 与 event check | modify |
| CLI | `meta_flow/cli.py` | 命令注册 | modify |
| State compact | `meta_flow/state/current.py` | state render + check | doc/help clarify |
| Tests | `tests/` | 需要新增 compact fixture | create |
| Archive output | `process/archive/ledger/` | compact 目标 | create at runtime |

## 推荐方案

| 设计点 | 推荐做法 | 理由 | 代价 |
|---|---|---|---|
| CLI | `meta-flow ledger compact --ledger <path|all> --dry-run` | 对象清晰，避免 state compact 混淆 | 新增命令 |
| policy | `process/policies/LEDGER-RETENTION.yaml` | 与读取策略分离 | 新配置和 checker |
| archive | 压缩旧事件到 `process/archive/ledger/<ledger>/<period>.summary.json` | 保留审计和恢复 | 需要 hash / index |
| safety | 默认 dry-run，显式 `--apply` 才写入 | 避免误删历史 | UX 多一步 |

## 数据模型与状态

| Object | Owner | 新增 / 修改字段 | 状态变化 | 兼容性 |
|---|---|---|---|---|
| `LEDGER-RETENTION.yaml` | ledger compact | `keep_latest_n_events`、`keep_latest_n_cr`、`window_days`、`archive_rule` | 新增 | 不影响 default_context |
| `ledger-archive-index.json` | ledger compact | `source_ledger`、`range`、`event_count`、`hash_before`、`summary_ref`、`restore_hint` | 新增 | 旧 ledger 可继续读取 |
| NDJSON ledger | event ledger | compact 后只保留窗口内全量行和 archive marker | 压缩 | marker 必须被 checker 接受或明确 legacy-compatible |

## API / 接口设计

| Interface ID | 调用方 | 被调用方 | 输入契约 | 输出契约 | 错误模型 |
|---|---|---|---|---|---|
| IF-LC-01 | user / CI | `meta-flow ledger compact` | ledger path、policy、dry-run/apply | plan / summary / archive index | invalid policy / unsafe ledger |
| IF-LC-02 | checker | `meta-flow event check` | compacted ledger | OK 或明确 marker 兼容 | invalid marker |
| IF-LC-03 | roadmap refresh | Gate Ledger event | normal event row | 可被 compact | missing event_id/gate/status |

## 关键流程

| Flow ID | 触发条件 | 主流程 | 异常流程 | 输出 / 状态变化 | 观测点 |
|---|---|---|---|---|---|
| FLOW-LC-01 | dry-run | 读取 ledger -> 应用 policy -> 输出 plan | policy invalid 时 FAIL | no write | dry-run report |
| FLOW-LC-02 | apply | 生成 archive summary/index -> 写 compacted ledger | hash mismatch 时 abort | ledger + archive index | event check |
| FLOW-LC-03 | restore audit | 用户需要历史事件 | 按 archive index 定位 summary / raw archive | 缺 archive 时 BLOCKED | archive index |

## 人机协作与确认点

| 确认点 | 触发条件 | 需要谁确认 | 推荐方案 | 备选方案 | 不授权项 |
|---|---|---|---|---|---|
| DQ-LC-01 | FEAT-PG-002 LLD 前 | user / host-orchestrator | 命令名 `meta-flow ledger compact` | `meta-flow event compact` | 不复用 `state compact` |
| DQ-LC-02 | apply 压缩历史 ledger 前 | maintainer | 先 dry-run，再显式 apply | 只报告不 apply | 不静默删除历史 |

## 异常、失败与降级策略

| Failure ID | 失败条件 | 系统行为 | 用户可见影响 | 恢复 / 回退 | 测试入口 |
|---|---|---|---|---|---|
| F-LC-01 | ledger 包含不合法行 | dry-run FAIL | 需先修复或 legacy 迁移 | 保持原 ledger | malformed fixture |
| F-LC-02 | archive 写入失败 | abort，不改原 ledger | compact 未完成 | 重试或换 archive path | filesystem fixture |
| F-LC-03 | compacted ledger event check 失败 | 回滚到备份 | 不进入后续门禁 | restore backup | integration |

## 权限与安全

| Rule ID | 规则 | 触发条件 | 失败行为 | 测试入口 |
|---|---|---|---|---|
| SEC-LC-01 | 默认 `--dry-run`，`--apply` 才修改 ledger | CLI 调用 | 无 apply 不写文件 | CLI test |
| SEC-LC-02 | 不处理 `process/quant-lab/**`，除非 P2 migration 授权 | 当前 CR story-planning | 跳过并报告 | path guard test |
| SEC-LC-03 | archive index 不包含凭据或原始长日志正文 | summary 生成 | 检查失败 | content guard |

## 测试与验收策略

| 验收对象 | 测试层级 | 覆盖场景 | 自动化方式 | 未自动化原因 / 手工入口 |
|---|---|---|---|---|
| CLI 命令边界 | CLI | TC-PG-017 | `meta-flow ledger compact --dry-run` | N/A |
| archive index | unit | ledger archive | pytest fixture | N/A |
| event checker compatibility | integration | compacted Gate/CR ledger | event check | N/A |
| rollback | integration | hash mismatch / write failure | tmp path fixture | N/A |

## 实现顺序

| Step | 内容 | 前置条件 | 输出 | 验证入口 |
|---|---|---|---|---|
| 1 | 定义 retention policy schema | CP3 approved | policy schema / examples | unit |
| 2 | 实现 dry-run planner | Step 1 | compact plan | pytest |
| 3 | 实现 archive index 和 apply | Step 2 | archive summary/index | integration |
| 4 | 接入 CLI 和 help 文案 | Step 2 | CLI | CLI test |
| 5 | 接入 event checker compatibility | Step 3 | checker update | event check |

## Story 拆分建议与 LLD 策略

| Story ID | feature_design_refs | lld_policy.required_level | 触发原因 | 必须进一步设计的问题 | 可用设计证据 |
|---|---|---|---|---|---|
| CR037-S04-ledger-compaction-policy-and-cli | `process/docs/features/ledger-compaction/DESIGN.md` | full-lld | migration / rollback / audit | policy fields、archive marker、restore behavior | Story LLD |

## 下游消费契约

| 消费方 | 消费时机 | 输入契约 | 输出 / 状态要求 | 降级策略 |
|---|---|---|---|---|
| story-manager | CP4 前 | Story ID、lld_policy、file ownership | Story 卡片 | 缺失则 CP4 FAIL |
| lld-designer | CP5 前 | CLI、policy、archive model | full-lld | 命令名未决则 clarification queue |
| meta-qa | CP7 | TEST-PLAN、fixtures | verification report | 无 restore 测试则 PASS_WITH_RISK 不可默认 |

## 风险与回退

| Risk ID | 风险 | 影响 | 缓解 | 回退 |
|---|---|---|---|---|
| R-LC-01 | 压缩破坏审计链 | 高 | dry-run、hash、archive index、backup | restore backup |
| R-LC-02 | 命令名与 state compact 混淆 | 中 | help 文案和 TC-PG-017 | 改 docs，不复用 state compact |
| R-LC-03 | compact policy 被误认为读取策略 | 中 | 独立 `LEDGER-RETENTION` | 文档修正和 checker |
