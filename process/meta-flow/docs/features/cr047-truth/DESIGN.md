---
status: ready-for-review
version: "1.0"
feature_id: "FEAT-WT-01"
feature_name: "Truth Consistency"
source_blueprint: "process/docs/design/CR047-WORKFLOW-TRUTH-BLUEPRINT.md"
source_hld: "process/docs/design/CR047-WORKFLOW-TRUTH-HLD.md"
source_adr: "process/docs/design/CR047-WORKFLOW-TRUTH-ARCHITECTURE-DECISION.md"
source_matrix: "process/docs/design/CR047-FEATURE-DESIGN-MATRIX.md"
related_stories: ["ST-WT-001"]
lld_policy_summary: "full-lld=1"
---

# Feature Design: Truth Consistency

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-14 | host-orchestrator-inline / meta-se | 冻结 State/CR/CURRENT source-owned relation、JSON-only tracking 与 stale-state 失败路由。 |

## 摘要与边界

| 项目 | 内容 |
|---|---|
| 目标 | 让 active State、formal CR/index/ledger 与 CURRENT projection 的关系可机器判定，拒绝 closed/missing active CR。 |
| 推荐方案 | 扩展现有 resolver/checker；保持 State 与 CR 单写，CURRENT 只重建。 |
| 下游 Story | ST-WT-001 |
| 非目标 | 新总状态文件、从 Markdown 反推机器状态、自动激活 CR-033 candidate。 |

## 上游依据与现有代码

| 来源/区域 | 路径 | 消费内容 / 当前职责 |
|---|---|---|
| ADR | `ADR-WT-001` | route→controlled write/status-sync→CURRENT refresh→relation check |
| Requirements | REQ-WT-001..003 | closed/missing rejection、projection fields、JSON-only index |
| CR checker | `meta_flow/checks/cr_tracking.py` | formal CR/index/legacy conflict 检查 |
| State | `meta_flow/state/current.py` | STATE.current writer、CURRENT projection、state validation |
| Lifecycle | `meta_flow/workflow/cr_lifecycle.py` | CR status-sync 与 index/ledger 更新 |
| Tests | `tests/test_state_v2.py`, `tests/test_gate_policy.py`, `tests/test_cr_lifecycle.py` | 状态、index、lifecycle fixtures |

## 推荐方案与接口

| 设计点 | 做法 | 失败行为 |
|---|---|---|
| lifecycle resolver | 复用 canonical JSON CR row/formal CR lifecycle enum；candidate 不算 active | unknown/closed/cancelled/superseded/missing active ref 为 BLOCKER |
| relation checker | 输入 State、CR-INDEX.json、formal CR、CR ledger、CURRENT；输出字段级 typed findings | 不自动修复、不读 legacy YAML 作为 truth |
| CURRENT contract | 只由 `state current-refresh` 从 source refs 重建 | scalar/ref drift 非零退出并列出字段 |
| legacy migration | canonical 路径只保留 JSON；CR-033 写 candidate row | YAML 并存或 candidate 缺失阻断 |

接口输出至少包含 `code,severity,source_ref,field,expected,actual,remediation`。检查器只读；writer/status-sync 仍由现有 owner 执行。

## 流程、状态与兼容

1. workspace health 通过。
2. 受控 writer 或 CR status-sync 写 source-owned truth。
3. CURRENT refresh 重建投影。
4. relation checker 读取五类输入并输出 findings。
5. blocker=0 才允许下游 Doctor/preflight 消费。

无新增持久化数据库；只扩展 JSON/ledger 关系校验。旧 Markdown summary 继续作为人类视图，不再作为 active CR 机器入口。

## 安全、失败与回退

| 条件 | 行为 | 回退 |
|---|---|---|
| active CR 生命周期非法 | fail-closed，保留最后稳定 State/CURRENT | 修正 formal CR/index 后重放 refresh/check |
| source 间无法判断 owner | blocker，禁止猜测 | 转设计澄清或独立 CR |
| migration 破坏历史 ref | 停止迁移 | 恢复 tracked 版本并追加 correction |

不得读取 credentials/runtime，不得改写历史 CP/result/ledger，不得把 candidate 自动升级为 active。

## 测试、任务与下游契约

- 单元/契约：closed/cancelled/superseded/missing/candidate/valid active 表格 fixture。
- 集成：State→status-sync→refresh→check；CURRENT 五个 scalar/ref drift。
- 回归：现有 state/cr lifecycle 测试；`meta-flow check cr-tracking`。
- TASK：见 `TASKS.md`；每个接口在 `TEST-PLAN.md` 有用例。
- lld-designer 必须冻结具体文件、schema 字段、迁移顺序和测试入口；偏离 owner 边界需重开 CP5。

## 风险与 Gotchas

- “一致”是关系成立，不是把三个对象字段合并。
- 删除 legacy YAML 前必须确认 JSON row、formal CR 与 ledger 可追溯。
- `CURRENT.json` 可重建，不得反写 State 或 CR。
