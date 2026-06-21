---
story_id: "STORY-CR-034-S04"
title: "bootstrap CR / CP0 / context / human gate readiness"
story_slug: "bootstrap-cr-cp0-context-human-gate-readiness"
lld_version: "1.0"
tier: "L"
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
  trigger_reasons: ["bootstrap-cr", "cp0", "context-capsule", "human-gate", "cr-ledger"]
  rationale: "目标项目启动整改时需要首个 CR、CP0/context/ledger/human gate 的可审计闭环，影响正式 workflow 入口，必须完整 LLD。"
open_items: 0
---

# LLD: STORY-CR-034-S04 - bootstrap CR / CP0 / context / human gate readiness

## 0. 上游设计依据

| 来源 | 路径 / ID | 被本 LLD 消费的内容 |
|---|---|---|
| CR | `process/changes/CR-034.md` | bootstrap CR、CP0/context/human gate 支撑目标 |
| Checkpoint manager | `.agents/skills/checkpoint-manager/SKILL.md` | CP0 / CP5 / human gate 文件结构和发起协议 |
| Context builder | `.agents/skills/context-manifest-builder/SKILL.md` | context capsule 默认读取入口和 token 策略 |
| CR lifecycle | `meta_flow/workflow/cr_lifecycle.py` | CR ledger / summary / index v2 能力 |
| Human gate checker | `meta_flow/checks/human_gate.py` | Decision Brief 和 launch message 校验入口 |

## 1. Goal

补齐目标项目启动整改时的 bootstrap CR、CP0 result、context capsule、event ledger、human gate launch readiness，确保目标项目第一轮正式工作流以 `CR-xxx` 命名并可审计进入人工决策。

## 2. Requirements（Functional / Non-Functional）

### 2.1 Functional

- 提供创建 bootstrap CR 的推荐命令或流程，统一使用 `CR-xxx`，不再新建 `MF-xxx`。
- bootstrap CR 创建后同步 CR index、CR summary、CR ledger、STATE.current.active_change 或 legacy fallback。
- 生成 CP0 request intake result / summary 或最小 Markdown 检查结果，记录 workspace、artifact routing、project identity、delivery routing、禁止项。
- 生成目标项目初始 context capsule，包含 allowed_reads、must_read、read_if_needed、do_not_read_by_default、不授权项和 human decisions。
- 生成 human gate checkpoint / launch message，并可通过 `meta-flow check human-gate`。

### 2.2 Non-Functional

- 全部输出路径默认在目标项目 `process` 路由内，不写目标项目业务源码。
- human gate 消息必须包含 checklist 路径、自动预检结论、决策收集覆盖、待决策项数量、待决策表和三个 exact 回复。
- 不授权 runtime、凭据、SaaS、production write、交易。

## 3. 模块拆分与职责

| 模块 / 文件组 | 职责 | 说明 |
|---|---|---|
| `meta_flow/workflow/cr_lifecycle.py` | bootstrap CR 创建 / summary / ledger / index | 现有 close/index 能力需扩展 active CR 创建 |
| `meta_flow/checks/cp_result.py` | CP0 result 校验复用 | 需要确认 CP0 result schema 支持无 story_id |
| `meta_flow/context_pack/builder.py` | 初始 context capsule / Story packet 支撑 | 需支持 adoption / bootstrap stage |
| `meta_flow/checks/human_gate.py` | 校验 checkpoint 和 launch message | 复用现有 checker |
| `tests/test_cr_lifecycle.py` / `test_cp_result_event_ledger.py` / `test_ask_user.py` | 回归覆盖 | 验证完整 bootstrap 门禁链 |

## 4. 代码结构与文件影响范围

| 动作 | 文件路径 | 变更内容 |
|---|---|---|
| 修改 | `meta_flow/workflow/cr_lifecycle.py` | 增加 active bootstrap CR 创建 / conflict keys / ledger append |
| 修改 | `meta_flow/context_pack/builder.py` | 增加 adoption / CP0 context profile 或复用 CP2/CP5 profile |
| 修改 | `meta_flow/ask_user.py` / `human_gate.py` | 必要时调整 launch message 校验支持 CR bootstrap 场景 |
| 修改 | `meta_flow/cli.py` | 暴露 bootstrap CR / context / gate 命令 |
| 创建 / 修改 | 测试文件 | 覆盖 CR bootstrap 到 human gate |
| 修改 | delivery 文档 | 说明目标项目启动 CR 流程 |

## 5. 数据模型与持久化设计

| 对象 / 字段 | 类型 | 约束 | 说明 |
|---|---|---|---|
| `CR-xxx.md` frontmatter | YAML-like | `cr_id`、`lifecycle_status=active`、`gate_status=cp2_pending/cp5_pending` | 正式 CR |
| `CR-INDEX.yaml/json` | index | active/blocked/candidate 可机器读 | legacy 和 v2 兼容 |
| `CR-LEDGER.ndjson` | append-only | 事件 JSON per line | active/closed 等生命周期事件 |
| `CP0-*.result.json` | CP result | schema_version=1 | 机器真相源 |
| `CP0/CP2 context capsule` | YAML/JSON | allowed_reads / must_read 完整 | 默认读取入口 |
| `CP*-LAUNCH-MESSAGE.md` | Markdown | human-gate checker 通过 | 人工门禁发起消息 |

## 6. API / Interface 设计

| 接口 / 入口 | 输入 | 输出 | 调用方 | 说明 |
|---|---|---|---|---|
| `create_bootstrap_cr(project_root, cr_id, title, scope)` | 项目根、CR 信息 | CR file、summary、ledger、index | CLI / Host | 创建 active CR |
| `build_bootstrap_context(project_root, cr_id)` | 项目根、CR | context capsule | Host / CLI | 默认读取入口 |
| `meta-flow cr bootstrap ...` | CLI | active CR + index/ledger | 用户 / Host | 命名待 CP5 确认 |
| `meta-flow ask-user human-gate ...` | checkpoint | launch message | 已存在 | 复用 |

## 7. 核心处理流程

1. adoption doctor 确认 workspace / state / identity 不存在阻断，或列出需用户接受的 WARN。
2. 创建 bootstrap CR 文件，写入禁止项和目标范围。
3. 更新 CR index / CR ledger / STATE current。
4. 生成 CP0 result / summary，记录 request intake、artifact routing、identity、delivery routing。
5. 生成 context capsule，约束 allowed_reads 和 do_not_read_by_default。
6. 生成 human gate checkpoint 和 launch message。
7. 运行 `meta-flow check human-gate`，通过后发起用户确认。

## 8. 技术设计细节

- 关键规则：bootstrap CR 创建不等于授权目标项目业务整改；它只是正式工作流入口。
- 依赖复用：复用 `cr_lifecycle` 的 summary/index/ledger 能力，但需补 active 创建路径。
- 兼容性处理：当前仓库同时有 legacy `CR-INDEX.yaml` 与 v2 `CR-INDEX.json` 可能性，CR-034 实现必须明确主写对象。
- 图示类型选择：涉及 CR、CP0、context、human gate 多模块，建议实现文档或 README 中可选给出流程图；LLD 不强制图示。

## 9. 安全与性能设计

| 维度 | 设计措施 | 验证方式 |
|---|---|---|
| 安全 | human gate 明确 approve 不授权 runtime / credentials / production write / trading | human-gate 测试检查不授权项 |
| 性能 | 只写少量 process 文件 | tmp fixture 测试 |

## 10. 测试设计

| 测试场景 | 前置条件 | 操作 | 预期结果 | 验证方式 |
|---|---|---|---|---|
| bootstrap CR creates active CR | initialized tmp project | `cr bootstrap` | CR 文件、index、ledger、STATE active_change 存在 | 单测 |
| CP0 context generated | active bootstrap CR | build context | capsule 包含 allowed_reads / no credentials | 单测 |
| human gate launch valid | checkpoint + launch message | `check human-gate` | PASS | 单测 |
| CR-xxx naming only | title includes old alias | create CR | formal file 仍为 CR-xxx | 单测 |

## 11. 实施步骤

| TASK-ID | 动作 | 目标文件 | 详细描述 | 对应测试 |
|---|---|---|---|---|
| TASK-CR034-S04-01 | 修改 | `cr_lifecycle.py` | 增加 active bootstrap CR 创建 | `test_cr_bootstrap_*` |
| TASK-CR034-S04-02 | 修改 | `context_pack/builder.py` | 增加 bootstrap context profile | `test_bootstrap_context_*` |
| TASK-CR034-S04-03 | 修改 | `cli.py` | 增加 CLI 路由 | CLI help 测试 |
| TASK-CR034-S04-04 | 修改 | `human_gate` 相关测试 | 校验 launch message | `test_ask_user.py` |
| TASK-CR034-S04-05 | 修改 | docs | 写明 bootstrap CR / CP0 / human gate 步骤 | guardrail |

## 12. 风险、难点与预研建议

### 12.1 实现灰区与取舍记录

| Clarification ID | 问题 | 选项与推荐 | 决策 / 答案 | 影响面 | 证据 | 重访条件 |
|---|---|---|---|---|---|---|
| LCQ-STORY-CR-034-S04-01 | CR index 主写 `YAML` 还是 v2 `JSON` | 推荐当前 legacy 项目继续同步 `CR-INDEX.yaml`，新 bootstrap 支持 v2 JSON；备选只支持 YAML | 待 CP5 确认；非阻断 | CR lifecycle / state | 当前 meta-flow 使用 `CR-INDEX.yaml` 且 cr-tracking PASS | state v2 默认项目可逐步切 JSON |

| 风险 / 难点 | 影响 | 缓解措施 / 预研建议 |
|---|---|---|
| human gate 校验严格导致发起失败 | 无法进入人工确认 | 生成 checkpoint 和 launch message 后必须运行 checker |
| active CR 创建与 legacy index 冲突 | 状态漂移 | 创建后立即运行 cr-tracking |

### OPEN / Spike 跟踪

| ID | 类型（OPEN / Spike） | 问题 | 下一动作 | 责任方 |
|---|---|---|---|---|
| O-S04-01 | OPEN | v2 JSON index 与 legacy YAML 同步策略 | CP5 接受推荐后实现兼容写入 | host-orchestrator |

## 13. 回滚与发布策略

- 发布方式：随 CLI 和 workflow docs 发布。
- 回滚触发条件：bootstrap CR 创建造成 active_change / index 漂移。
- 回滚动作：移除 bootstrap 命令，恢复手工 CR 创建流程；保留 checker 能力。

## 14. Definition of Done

- [ ] 可在 tmp 目标项目创建首个 `CR-xxx` active CR。
- [ ] CP0 result / context / human gate launch message 可生成并校验。
- [ ] CR index / summary / ledger / STATE 同步无漂移。
- [ ] approve 不授权项在 checkpoint 和 launch message 中明确列出。

## 人工确认区

**人工审查结果回填**：

- 结论：`approved`
- 审查人：user
- 审查时间：2026-06-21T18:36:21+08:00
- 修改意见：无
- 风险接受项：不适用；不授权 runtime / quant-lab 写入 / CR-033
