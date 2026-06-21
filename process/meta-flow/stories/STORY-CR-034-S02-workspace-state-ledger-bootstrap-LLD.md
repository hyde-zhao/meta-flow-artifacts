---
story_id: "STORY-CR-034-S02"
title: "workspace/state/ledger bootstrap"
story_slug: "workspace-state-ledger-bootstrap"
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
  trigger_reasons: ["state-v2", "ledger-bootstrap", "artifact-routing", "new-project-initialization"]
  rationale: "首次初始化会写入目标项目 process 路由、STATE.current.json、legacy STATE.md 摘要和基础 ledger，是目标项目整改的状态真相源，必须完整 LLD。"
open_items: 0
---

# LLD: STORY-CR-034-S02 - workspace/state/ledger bootstrap

## 0. 上游设计依据

| 来源 | 路径 / ID | 被本 LLD 消费的内容 |
|---|---|---|
| CR | `process/changes/CR-034.md` | state/ledger bootstrap 目标、不授权边界 |
| Workspace routing | `meta_flow/workspace/routing.py` | `link_process_workspace`、`PROCESS_SCAFFOLD_DIRS`、route metadata |
| State v2 | `meta_flow/state/current.py` | `default_current_state`、`write_current_state`、基础 ledger 列表 |
| README / AGENTS | `README.md` / `AGENTS.md` | 外置 process 软链接、portable-relative-v1、STATE.current 默认入口 |

## 1. Goal

补齐目标项目首次初始化的安全 bootstrap 路径：在 `workspace link` 后生成 `STATE.current.json`、legacy `STATE.md` 人类摘要和基础 ledgers，确保目标项目进入整改前不再只有半初始化的 `state_missing`。

## 2. Requirements（Functional / Non-Functional）

### 2.1 Functional

- 提供显式 bootstrap 命令，建议为 `meta-flow workspace bootstrap --project-root . --artifact-root <rel> --project-name <name>`，或在 `workspace link` 后提示并支持 `state migrate-v2/init`。
- 创建或校验基础目录：`process/checks`、`process/checkpoints`、`process/context`、`process/changes`、`process/state`、`process/policies`。
- 写入 `process/state/STATE.current.json`，且字段保持轻量，不塞入 history / cr_tracking 长字段。
- 创建基础 ledgers：`CR-LEDGER.ndjson`、`STORY-LEDGER.ndjson`、`CHECKPOINT-LEDGER.ndjson`、`HANDOFF-LEDGER.ndjson`、`AGENT-DISPATCH-LEDGER.ndjson`、`GATE-LEDGER.ndjson`、`RUN-LEDGER.ndjson`、`READ-EXPANSION-LEDGER.ndjson`。
- 渲染 legacy `process/STATE.md` 人类摘要，或在已有 legacy STATE 时保留兼容路径。
- 不覆盖已有非生成态 `STATE.md`，除非用户显式 `--force`。

### 2.2 Non-Functional

- 路由记录必须使用相对路径和 anchor，不写设备相关绝对路径。
- 命令幂等：重复运行不会破坏已有状态；覆盖必须显式 `--force`。
- 无网络、无凭据、无 runtime。

## 3. 模块拆分与职责

| 模块 / 文件组 | 职责 | 说明 |
|---|---|---|
| `meta_flow/workspace/routing.py` | 扩展 scaffold 目录和 bootstrap 组合流程 | 当前 scaffold 缺 `state`、`policies` |
| `meta_flow/state/current.py` | 提供初始化 state v2 的 API / CLI | 当前支持 migrate-v2 和 write_current_state，但缺新项目 init 入口 |
| `meta_flow/cli.py` | 暴露 workspace bootstrap 或 state init | 命令命名需与文档一致 |
| `tests/test_workspace_bootstrap.py` / `tests/test_state_v2.py` | 覆盖新项目 bootstrap | 用 tmp 目录，不触碰真实 quant-lab |

## 4. 代码结构与文件影响范围

| 动作 | 文件路径 | 变更内容 |
|---|---|---|
| 修改 | `meta_flow/workspace/routing.py` | `PROCESS_SCAFFOLD_DIRS` 增加 `state`、`policies`、`returns`、`evidence`、`handoffs` 等必要目录；或由 bootstrap 显式创建 |
| 修改 | `meta_flow/state/current.py` | 增加新项目 `init` / `bootstrap` 支持，创建基础 ledgers |
| 修改 | `meta_flow/cli.py` | 增加命令入口和 help |
| 创建 / 修改 | `tests/test_workspace_routing.py` / `tests/test_state_v2.py` | 覆盖 link 后 bootstrap、幂等、force 保护 |
| 修改 | `delivery/README.md` / `USER-MANUAL.md` | 追加目标项目初始化命令链 |

## 5. 数据模型与持久化设计

| 对象 / 字段 | 类型 | 约束 | 说明 |
|---|---|---|---|
| `STATE.current.json.schema_version` | int | 固定 2 | 机器状态版本 |
| `STATE.current.json.project_id` | str | 默认 project_name | 目标项目身份 |
| `STATE.current.json.active_change` | null / str | 初始 null | bootstrap CR 创建前为空 |
| `STATE.current.json.routing_ref` | str | `process/.meta-flow-process.yaml` | 路由元数据 |
| 基础 ledgers | NDJSON | 可为空文件 | 后续事件追加 |

## 6. API / Interface 设计

| 接口 / 入口 | 输入 | 输出 | 调用方 | 说明 |
|---|---|---|---|---|
| `bootstrap_workspace_state(project_root, artifact_root, project_name, force=False)` | 路径和项目名 | 写入 process scaffold/state/ledgers | CLI | 组合 link + state init |
| `init_current_state(project_root, project_id, force=False)` | 项目根 | `STATE.current.json` 和 ledgers | workspace bootstrap | 新项目使用，不依赖 legacy STATE |
| CLI `meta-flow workspace bootstrap ...` | 参数 | 人类摘要 + exit code | 用户 / Host Orchestrator | 推荐入口 |

## 7. 核心处理流程

1. 校验或建立 process symlink，写 `.meta-flow-process.yaml`。
2. 创建基础目录和空 ledger 文件。
3. 若 `STATE.current.json` 不存在，写入 default current state。
4. 若 `STATE.md` 不存在或是 generated summary，渲染人类摘要。
5. 运行 `state check` 和 `workspace check`。
6. 输出下一步：运行 adoption doctor 或创建 bootstrap CR。

## 8. 技术设计细节

- 关键规则：`workspace link` 的 `state_missing` 是正常中间态，但 adoption 前必须通过 bootstrap 收敛。
- 依赖复用：`write_current_state` 已创建部分 ledgers，需补齐 CR / STORY / READ-EXPANSION ledgers。
- 兼容性处理：已有 legacy `STATE.md` 时不强制覆盖；可通过 `state migrate-v2` 迁移。
- 图示类型选择：不需要 Mermaid 图，流程线性且无异步补偿。

## 9. 安全与性能设计

| 维度 | 设计措施 | 验证方式 |
|---|---|---|
| 安全 | 仅写目标项目 process 路由和运行态 scaffold；不写业务源码 | tmp fixture 验证写入路径集合 |
| 性能 | 只创建小文件和目录 | 单测检查命令快速返回 |

## 10. 测试设计

| 测试场景 | 前置条件 | 操作 | 预期结果 | 验证方式 |
|---|---|---|---|---|
| bootstrap new project | tmp 项目无 process | `workspace bootstrap` | symlink、metadata、STATE.current、STATE.md、ledgers 全部存在 | 单测 |
| idempotent rerun | 已 bootstrap | 重跑命令 | 不报错，不覆盖非生成文件 | 单测 |
| existing non-generated STATE | 已有人工 `STATE.md` | bootstrap | 阻断或提示 `--force` / migrate-v2 | 单测 |
| portable paths | tmp 路径 | bootstrap | metadata 不含绝对 tmp 路径 | 单测 |

## 11. 实施步骤

| TASK-ID | 动作 | 目标文件 | 详细描述 | 对应测试 |
|---|---|---|---|---|
| TASK-CR034-S02-01 | 修改 | `meta_flow/state/current.py` | 增加 init API 并补齐基础 ledgers | `test_state_init_*` |
| TASK-CR034-S02-02 | 修改 | `meta_flow/workspace/routing.py` | 扩展 bootstrap scaffold 或新增组合函数 | `test_workspace_bootstrap_*` |
| TASK-CR034-S02-03 | 修改 | `meta_flow/cli.py` | 增加 `workspace bootstrap` 或等价命令 | CLI help 测试 |
| TASK-CR034-S02-04 | 修改 | 文档 | 更新初始化命令链 | guardrail |

## 12. 风险、难点与预研建议

### 12.1 实现灰区与取舍记录

| Clarification ID | 问题 | 选项与推荐 | 决策 / 答案 | 影响面 | 证据 | 重访条件 |
|---|---|---|---|---|---|---|
| LCQ-STORY-CR-034-S02-01 | bootstrap 是新增 `workspace bootstrap`，还是组合 `workspace link` + `state init` | 推荐新增 `workspace bootstrap` 作为用户入口，同时保留底层 `state init`；备选只补文档命令链 | 待 CP5 确认；非阻断 | CLI / docs / tests | `workspace link` 当前只到 `state_missing` | 若实现复杂度过高，可先落地 `state init` + 文档命令链 |

| 风险 / 难点 | 影响 | 缓解措施 / 预研建议 |
|---|---|---|
| 覆盖已有 STATE.md | 破坏 legacy 状态 | 默认不覆盖非 generated 文件，必须 `--force` |
| 生成 ledgers 不完整 | 后续 CP / CR 检查 warning | 明确基础 ledger 清单并测试 |

### OPEN / Spike 跟踪

| ID | 类型（OPEN / Spike） | 问题 | 下一动作 | 责任方 |
|---|---|---|---|---|
| O-S02-01 | OPEN | 命令形态需 CP5 确认 | 在 CP5 Decision Brief 中采用推荐方案 | host-orchestrator |

## 13. 回滚与发布策略

- 发布方式：随 CLI 和安装文档发布。
- 回滚触发条件：bootstrap 命令覆盖现有状态或路由错误。
- 回滚动作：撤销 CLI 入口，保留底层 `workspace link` / `state migrate-v2`，文档回退为手工命令链。

## 14. Definition of Done

- [ ] 新项目可一条命令完成 process symlink、metadata、STATE.current、STATE.md 摘要和基础 ledgers。
- [ ] 现有 legacy 项目可迁移或被安全阻断。
- [ ] 路径记录不含设备相关绝对路径。
- [ ] `workspace check`、`state check`、adoption doctor 能识别 bootstrap 状态。

## 人工确认区

**人工审查结果回填**：

- 结论：`approved`
- 审查人：user
- 审查时间：2026-06-21T18:36:21+08:00
- 修改意见：无
- 风险接受项：不适用；不授权 runtime / quant-lab 写入 / CR-033
