---
checkpoint_id: "CP2-CR047"
checkpoint_name: "CR-047 Requirements and Scope Baseline"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-07-13T03:40:39Z"
reviewed_by: "user"
reviewed_at: "2026-07-13T03:59:04Z"
auto_check_result: "process/checks/CP2-CR047-REQUIREMENTS-BASELINE.result.json"
context_ref: "process/context/CP2-CR047-REQUIREMENT-CONTEXT.yaml"
decision_brief_profile: "compact"
target:
  phase: "requirement-clarification"
  artifacts: ["docs/product/USE-CASES.md", "docs/product/REQUIREMENTS.md", "docs/product/SCENARIOS.yaml", "docs/product/TEST-MATRIX.md", "docs/product/STORY-MAP.md", "docs/product/MVP-SCOPE.md", "docs/product/RELEASE-SLICES.md", "docs/product/BACKLOG.md"]
---

# CP2 CR-047 Requirements and Scope Baseline 人工审查

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP1-CR047-USE-CASE-COMPLETENESS-R2.result.json` | PASS | 0 | R1 BLOCKED 保留；R2 经用户批准 inline fallback 完成 |
| `process/checks/CP2-CR047-REQUIREMENTS-BASELINE.result.json` | PASS | 0 | 四项策略决策待人工选择 |

## Decision Brief

### 审批者摘要

| 字段 | 内容 |
|---|---|
| 本次确认服务的整体目标 | 让 clean clone 与已链接工作区共享一个 workflow truth，并使 CR tracking、Doctor、guardrail、Ruff、安装说明和 CR-046 当前状态可确定验证。 |
| 推荐动作 | `approve`：接受 CP2-DQ-01..04 推荐方案，进入 CP3 架构设计。 |
| approve 后会发生什么 | 主进程继续采用 user-approved inline fallback，生成 HLD/ADR 候选并在 CP3 再次请求人工确认；不会立即实现代码。 |
| approve 不授权什么 | credentials、runtime、SaaS、production write、publish、trading、repository commit/push、prelink backup 修改。 |
| 不确认会阻塞什么 | CR-047 停留在 requirement-clarification；不得进入 CP3、Story 拆解、LLD 或实现。 |

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP2-CR047-REQUIREMENT-CONTEXT.yaml` |
| capsule 状态 | ready |
| read_profile | compact |
| 默认读取策略 | capsule first；正式产品文档只在核验 DQ/traceability 时展开 |
| 全文档读取扩展 | 已由 meta-pm 尝试和 Host inline fallback 写入 `READ-EXPANSION-LEDGER.ndjson` |
| 缺失 / waived 理由 | 无 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `process/state/STATE.current.json`、`GATE-LEDGER.ndjson` | scanned | 1 | 0 | 子 Agent 卡顿已由用户批准 inline fallback 解决 |
| 委托 Agent 交还摘要 | `process/handoffs/CR047-CP1-CP2-META-PM.md` | scanned | 1 | 0 | 无完整 agent return；中断事实保留，Host fallback 已授权 |
| 自动预检结果 | CP1/CP2 result | scanned | 0 | 0 | 均 PASS |
| discussion log / checkpoint | `process/discussions/CP2-CR047-SCENARIO-DISCUSSION-LOG.md`、discussion checkpoint | scanned | 4 | 4 | SGA-WT-01..04 进入 DQ |
| 下游正式产物 | 8 个产品基线文件、修复前命令输出摘要 | scanned | 4 | 4 | canonical rule、Doctor、budget、cache |
| 用户显式选择题 | 当前对话 | scanned | 2 | 0 | 已确认启动 CR 和禁用子 Agent；未替代 CP2 决策 |

### 决策分层

| 分类 | 数量 | 处理方式 |
|---|---:|---|
| 必须用户决策 | 4 | CP2-DQ-01..04 |
| 高风险策略确认 | 0 | 无 runtime/security 外部授权 |
| agent 默认处理 | 3 | Ruff safe-fix、README 示例、CR-046 状态回链的局部实现细节 |
| 仅审计记录 | 3 | 子 Agent 中断、prelink backup 排除、receipt unavailable |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| CP2-DQ-01 | scope | 根 `AGENTS.md` 与 tracked 规则的 canonical source 如何统一？ | `delivery/rules/AGENTS.md` 为 tracked canonical；根 `AGENTS.md` 为安装生成 wrapper；源码 guardrail 检查 tracked source + installer dry-run。 | A：跟踪一个去个人化根 `AGENTS.md`；B：维持现状但 guardrail 不要求根文件。 | 推荐方案符合现有 delivery/安装架构且避免个人配置入库；A 对直接 clone 友好但产生双 source；B 最小但失去 wrapper 漂移检查。 | 影响 clean clone、安装器、guardrail 与用户规则发现；错误选择会导致双真相源。 | 若平台正式要求 tracked 根文件，则切换到 A，并由生成器保证单向同步。 |
| CP2-DQ-02 | risk_acceptance | Doctor 何时算绿色？ | 仅 `ERROR/FAIL/BLOCKED` 导致非零；`WARN` 计数、分类和披露，但不自动阻断。 | A：所有 warning 阻断；B：Doctor 永远 0，仅看报告。 | 推荐方案兼顾历史兼容与发布诚实性；A 会让 legacy/empty ledger 永久红；B 会掩盖真实 blocker。 | 影响 CP8、CI 退出码和 `READY_WITH_RISK`。 | 新 warning 类别若可导致证据伪造或安全风险，升级为 blocker。 |
| CP2-DQ-03 | implementation | 历史超预算对象如何收敛？ | active/default-read 对象严格预算；closed 历史移入 archive/冷区或保留原件并生成 compact summary/index/hash/correction，Doctor 默认不按活动预算扫描 archive。 | A：提高全局阈值；B：原位截断/改写历史文件。 | 推荐方案保持预算价值和历史 hash；A 简单但掩盖膨胀；B 破坏审计，禁止采用。 | 影响 21 个超限、历史 CP result/LLD 和 token budget。 | 若某历史对象仍为当前 required read，则保留原件并单独生成受预算摘要，不归档唯一证据。 |
| CP2-DQ-04 | implementation | ignored Python cache 如何进入 guardrail？ | Git tracked 或进入打包输入的 cache 为 BLOCKING；仅 ignored 本机 cache 为 WARN，并提供标准 preflight 清理；CI/打包始终基于 clean tracked tree。 | A：任何 cache 都阻断；B：所有 cache 都忽略。 | 推荐方案避免测试后永久红灯，同时保护交付包；A 严格但开发体验差；B 可能污染交付。 | 影响本地发布前检查、CI 与打包卫生。 | 若 ignored cache 被纳入 staging/package manifest，立即升级为 BLOCKING。 |

| 字段 | 内容 |
|---|---|
| 推荐决策 | `approve` 接受 CP2-DQ-01..04 推荐方案 |
| 备选方案 | 可逐项使用 `修改: CP2-DQ-xx ...`；也可 reject 保持当前基线 |
| 影响维度 | 用户价值、可验证性、维护成本、平台兼容、交付质量 |
| 优劣分析 | 见各 DQ 行 |
| 风险与回退 | CP3 前仍可回退 requirement-clarification；未批准不实现 |
| 用户需决策事项 | CP2-DQ-01、CP2-DQ-02、CP2-DQ-03、CP2-DQ-04 |

### CP2 追加摘要

- 用户真实意图：解决已复现的治理与交付缺口，但不处理 prelink backup。
- 场景覆盖：UC-WT-001..007；TC-WT-001..007 覆盖 negative、positive、failure-recovery、boundary、precheck。
- 认知盲区：tracked/generated 规则边界、warning 退出码、历史预算语义、本机 cache 与交付输入区分。
- Deferred：platform receipt producer、runtime/SaaS/pilot。
- discussion log：`process/discussions/CP2-CR047-SCENARIO-DISCUSSION-LOG.md`。

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| CP0/CP1 通过 | PASS | CP0/CP1 result |  |
| 产品基线齐套 | PASS | 8 个 `docs/product/*` 文件 | 增量更新，旧 ID 保留 |
| Context ready | PASS | CP2 context | compact |
| 自动预检通过 | PASS | CP2 result | blockers=0 |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | UC/REQ/TC/ST 可追溯 | 通过 | USE-CASES、REQUIREMENTS、SCENARIOS、TEST-MATRIX、STORY-MAP | 用户批准当前基线 |
| 2 | In/Out/Deferred 明确 | 通过 | MVP-SCOPE、BACKLOG | 用户批准当前范围 |
| 3 | 四项灰区有推荐、备选和回退 | 通过 | CP2-DQ-01..04 | 接受四项推荐方案 |
| 4 | 不授权边界完整 | 通过 | CR-047、context | 批准不扩大外部/运行/Git 授权 |
| 5 | inline fallback 证据诚实 | 通过 | handoff、dispatch ledger | 不声称子 Agent 完成 |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| P0/P1 范围无阻断歧义 | 通过 | Decision Brief | CP2-DQ-01..04 均采用推荐方案 |
| 人工确认完成 | 通过 | 人工审查结果 | 用户回复“批准” |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| 产品基线 | `docs/product/*` | 通过 | 8 文件 |
| 自动结果 | `process/checks/CP2-CR047-REQUIREMENTS-BASELINE.result.json` | PASS |  |
| Context | `process/context/CP2-CR047-REQUIREMENT-CONTEXT.yaml` | PASS |  |
| Discussion | `process/discussions/CP2-CR047-SCENARIO-DISCUSSION-LOG.md` | PASS |  |

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-07-13T03:59:04Z
- 修改意见：无；接受 CP2-DQ-01..04 的推荐方案。
- 风险接受项：Doctor WARN 可以披露但不自动阻断；历史证据采用 archive/compact/hash/correction 而不改写；ignored 本机 cache 仅 WARN。本批准不授权 runtime、credentials、SaaS、production write、publish、trading、repository commit/push 或 prelink backup 变更。
