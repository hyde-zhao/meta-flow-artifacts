---
story_id: "STORY-CR-034-S01"
title: "adoption readiness preflight and doctor"
story_slug: "adoption-readiness-preflight-and-doctor"
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
  trigger_reasons: ["target-project-bootstrap", "process-health", "multi-check-aggregation", "human-gate-readiness"]
  rationale: "目标项目 adoption 前置检查会聚合 workspace、state、CR、identity、quality、workflow 和 human gate 状态，影响后续是否允许进入 quant-lab 整改，因此必须完整 LLD。"
open_items: 0
---

# LLD: STORY-CR-034-S01 - adoption readiness preflight and doctor

## 0. 上游设计依据

| 来源 | 路径 / ID | 被本 LLD 消费的内容 |
|---|---|---|
| CR | `process/changes/CR-034.md` | adoption readiness 范围、禁止 quant-lab 写入、CR-033 deferred 边界、五维影响分析 |
| State | `process/STATE.md` | 当前 active_change=`CR-034`，legacy state 仍是人类摘要 / fallback |
| CR Index | `process/changes/CR-INDEX.yaml` | CR-034 active、CR-033 candidate、当前无 blocked CR |
| Workspace routing | `meta_flow/workspace/routing.py` | `workspace check/link` 和 process route health 的现有实现 |
| Existing doctors | `meta_flow/cli.py` / `meta_flow/checks/*` | `doctor workspace/context/artifacts/quality/workflow` 命令面和 checker 入口 |

## 1. Goal

为 meta-flow 增加目标项目 adoption readiness preflight / doctor 聚合入口，在进入 quant-lab 前能用本地、非凭据、非 runtime 的方式判断安装、初始化、状态、CR、identity、quality、workflow 和 human gate 是否具备启动整改的条件。

## 2. Requirements（Functional / Non-Functional）

### 2.1 Functional

- 提供一个只读聚合检查入口，建议命令名为 `meta-flow doctor adoption --project-root <target-root>` 或 `meta-flow adoption check --project-root <target-root>`，最终名称在实现前可二选一但必须在文档中统一。
- 聚合执行或复用以下检查：workspace route health、state v2 readiness、CR tracking readiness、package identity readiness、quality policy readiness、workflow ledger readiness、human gate launch prerequisites。
- 输出稳定的 PASS / WARN / FAIL 摘要，包含检查项、证据路径、影响、下一步动作。
- 默认不得写目标项目业务文件；若需要 scaffold，必须交由 S02/S04 的显式 bootstrap 命令处理。
- 明确不授权凭据读取、runtime、SaaS、production write、真实交易和 CR-033。

### 2.2 Non-Functional

- 本地确定性：不访问网络、不读凭据、不依赖外部服务。
- 可脚本化：退出码 `0` 表示无 FAIL，`1` 表示存在阻断，warning 不阻断。
- 可审计：每个 FAIL / WARN 必须给出证据路径和修复建议。
- 可迁移：路径输出优先相对项目根；不固化 `/home/...`。

## 3. 模块拆分与职责

| 模块 / 文件组 | 职责 | 说明 |
|---|---|---|
| `meta_flow/checks/adoption_readiness.py` | 新增 adoption readiness 聚合检查器 | 只读检查，汇总现有 checker 结果，不执行 bootstrap 写入 |
| `meta_flow/cli.py` | 暴露命令入口 | 增加 `doctor adoption` 或 `adoption check` 路由和 help 文案 |
| `tests/test_adoption_readiness.py` | 覆盖 PASS / WARN / FAIL 场景 | 用 tmp fixture 构造目标项目，不进入真实 quant-lab |
| `delivery/README.md` / `delivery/doc/USER-MANUAL.md` | 用户说明 | 说明 adoption 前置检查命令链和不授权边界 |

## 4. 代码结构与文件影响范围

| 动作 | 文件路径 | 变更内容 |
|---|---|---|
| 创建 | `meta_flow/checks/adoption_readiness.py` | 实现聚合检查模型、检查项、CLI main |
| 修改 | `meta_flow/cli.py` | 增加 doctor/adoption 命令入口和 help |
| 创建 | `tests/test_adoption_readiness.py` | 新增 adoption readiness 回归测试 |
| 修改 | `delivery/README.md` | 追加目标项目 adoption readiness 说明 |
| 修改 | `delivery/doc/USER-MANUAL.md` | 追加操作步骤和失败处理 |
| 修改 | `docs/release/RELEASE-NOTES.md` | 追加 CR-034 变更说明 |

## 5. 数据模型与持久化设计

无新增持久化数据模型。本 Story 输出运行时检查结果到 stdout，不默认写入 `process/*`。

检查结果内部结构建议：

| 对象 / 字段 | 类型 | 约束 | 说明 |
|---|---|---|---|
| `ReadinessItem.id` | `str` | 稳定 ID | 如 `workspace-route`、`state-v2` |
| `ReadinessItem.status` | `PASS/WARN/FAIL` | 枚举 | 决定汇总结论和退出码 |
| `ReadinessItem.evidence` | `list[str]` | 相对路径优先 | 证据路径或命令输出摘要 |
| `ReadinessItem.next_action` | `str` | 非空 | 失败时给出可执行下一步 |

## 6. API / Interface 设计

| 接口 / 入口 | 输入 | 输出 | 调用方 | 说明 |
|---|---|---|---|---|
| `run_adoption_doctor(project_root: Path) -> int` | 项目根 | stdout + exit code | CLI / tests | 聚合检查，不写文件 |
| `collect_adoption_readiness(project_root: Path) -> list[ReadinessItem]` | 项目根 | 检查项列表 | 单测 / CLI | 纯函数式收集，便于测试 |
| CLI `meta-flow doctor adoption --project-root .` | `--project-root` | 人类可读报告 | Host Orchestrator / 用户 | 推荐入口；若最终实现选择 `meta-flow adoption check`，需在 docs 中统一 |

## 7. 核心处理流程

1. 解析 `project_root`，不改变当前工作目录。
2. 调用 workspace route 检查；阻断时记录 FAIL。
3. 检查 `process/state/STATE.current.json`、基础 ledgers 和 legacy `process/STATE.md` fallback。
4. 检查 CR tracking 是否可用；若存在 active / blocked CR，列出状态。
5. 检查 package identity 文件和 pyproject/package 关系。
6. 检查 quality policy、workflow ledger、human gate prerequisites。
7. 汇总 PASS / WARN / FAIL，打印不授权项，按 FAIL 决定退出码。

## 8. 技术设计细节

- 关键规则：adoption doctor 只判断 readiness，不创建或修复文件。
- 依赖复用：优先复用 `check_process_route`、state checker、CR tracking checker、product governance checker、quality governance checker。
- 兼容性处理：若目标项目尚未初始化 `process`，workspace 缺失应为 FAIL，并提示先执行 `workspace link`，不得静默创建。
- 图示类型选择：当前流程少于 3 个跨异步模块，不需要 Mermaid 图。

## 9. 安全与性能设计

| 维度 | 设计措施 | 验证方式 |
|---|---|---|
| 安全 | 不读 `.env`、不访问凭据、不触发 runtime、不进入 quant-lab 业务路径写入 | 单测检查无写入；文档列出不授权项 |
| 性能 | 只扫描少量固定路径和小型索引文件 | tmp fixture 测试，避免递归全仓库扫描 |

## 10. 测试设计

| 测试场景 | 前置条件 | 操作 | 预期结果 | 验证方式 |
|---|---|---|---|---|
| healthy initialized project | tmp 项目有 process symlink、STATE.current、基础 ledgers、identity、quality policy | 运行 adoption doctor | exit 0，全部 PASS 或非阻断 WARN | `tests/test_adoption_readiness.py` |
| missing process route | tmp 项目无 process | 运行 adoption doctor | exit 1，workspace-route FAIL | 单测 |
| missing state v2 | 只有 legacy STATE.md | 运行 adoption doctor | WARN 或 FAIL，提示 bootstrap/migrate-v2 | 单测 |
| missing package identity | 缺 `PACKAGE-IDENTITY.yaml` | 运行 adoption doctor | FAIL，提示 identity init/check | 单测 |
| no credential access | fixture 含 `.env` | 运行 adoption doctor | 不读取 `.env` 内容 | 单测可用 sentinel 文件权限或 monkeypatch |

## 11. 实施步骤

| TASK-ID | 动作 | 目标文件 | 详细描述 | 对应测试 |
|---|---|---|---|---|
| TASK-CR034-S01-01 | 创建 | `meta_flow/checks/adoption_readiness.py` | 新增检查项模型和聚合函数 | `test_adoption_doctor_*` |
| TASK-CR034-S01-02 | 修改 | `meta_flow/cli.py` | 增加 CLI 路由与 help | CLI 单测或 stdout 断言 |
| TASK-CR034-S01-03 | 创建 | `tests/test_adoption_readiness.py` | 构造 tmp fixture 覆盖 PASS / FAIL / WARN | pytest |
| TASK-CR034-S01-04 | 修改 | `delivery/README.md` / `USER-MANUAL.md` | 追加 adoption doctor 用法和边界 | 文档 guardrail |

## 12. 风险、难点与预研建议

### 12.1 实现灰区与取舍记录

| Clarification ID | 问题 | 选项与推荐 | 决策 / 答案 | 影响面 | 证据 | 重访条件 |
|---|---|---|---|---|---|---|
| LCQ-STORY-CR-034-S01-01 | adoption 入口命名使用 `doctor adoption` 还是独立 `adoption check` | 推荐 `doctor adoption`，备选 `adoption check`；推荐理由是复用现有 doctor 语义 | 待 CP5 确认；非阻断 | CLI / docs | `meta_flow/cli.py` 当前已有 doctor 子命令 | 若实现时发现 doctor 参数解析冲突，可切换独立命令 |

| 风险 / 难点 | 影响 | 缓解措施 / 预研建议 |
|---|---|---|
| 聚合检查误写目标项目 | 破坏 adoption 的只读边界 | 聚合函数不得调用 scaffold 写入 API；写入能力放到 S02/S04 |
| checker 之间输出格式不一致 | 报告难以机器消费 | 内部统一 `ReadinessItem`，stdout 只作为展示 |

### OPEN / Spike 跟踪

| ID | 类型（OPEN / Spike） | 问题 | 下一动作 | 责任方 |
|---|---|---|---|---|
| O-S01-01 | OPEN | 最终 CLI 名称需在 CP5 确认 | CP5 Decision Brief 收敛推荐 | host-orchestrator |

## 13. 回滚与发布策略

- 发布方式：随 meta-flow CLI 和 delivery 文档发布。
- 回滚触发条件：adoption doctor 引入误报阻断或错误写入行为。
- 回滚动作：移除 CLI 路由和 `adoption_readiness.py`，保留 CR-034 文档记录，恢复文档中的命令说明。

## 14. Definition of Done

- [ ] adoption readiness 入口可在 tmp 目标项目运行。
- [ ] 缺 process、缺 state v2、缺 identity、缺 ledger 均有明确 FAIL/WARN 和下一步。
- [ ] 不读取凭据、不执行 runtime、不写目标项目业务文件。
- [ ] 文档说明 adoption doctor 是 preflight，不替代 bootstrap。
- [ ] CP6 / CP7 证据引用测试和 guardrail。

## 人工确认区

**CP5 checklist 摘要**：

| # | 检查项 | 状态 | 证据 |
|---|---|---|---|
| 1 | LLD 覆盖 AC | 待检查 | 第 2 / 10 / 14 节 |
| 2 | 与 CR-034 一致 | 待检查 | 第 0 / 12 节 |
| 3 | 文件影响范围明确 | 待检查 | 第 4 / 11 节 |
| 4 | 接口契约完整 | 待检查 | 第 6 节 |
| 5 | 测试与 dev_gate 可计算 | 待检查 | 第 10 / 14 节 |

**人工审查结果回填**：

- 结论：`approved`
- 审查人：user
- 审查时间：2026-06-21T18:36:21+08:00
- 修改意见：无
- 风险接受项：不适用；不授权 runtime / quant-lab 写入 / CR-033
