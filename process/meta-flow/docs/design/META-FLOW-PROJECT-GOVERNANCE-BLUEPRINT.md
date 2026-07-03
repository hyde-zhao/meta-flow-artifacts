---
status: baseline
version: "1.0"
cr_ref: "CR-037"
source_plan: "process/docs/design/META-FLOW-PROJECT-GOVERNANCE-STATE-ENFORCEMENT-IMPLEMENTATION-PLAN-2026-07-02.md"
source_state:
  - "process/state/STATE.current.json"
  - "process/STATE.md"
source_cr_index: "process/changes/CR-INDEX.json"
confirmed_by: ""
confirmed_at: ""
---

# Meta Flow Project Governance Blueprint

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-02 | meta-se | 从已批准实施计划抽取项目治理能力地图、Feature 边界、数据归属和跨 Feature 流程 |

## 1. 蓝图定位

本文档是 `STATE.current.json v2 enforcement`、项目级治理、roadmap refresh、impact surface 归一和 quant-lab 迁移准备的长期蓝图输入。它不替代实施计划，不作为代码实现证据；后续正式 CR、Story、LLD 和验证计划应引用本文档中的 Feature / Capability / Flow / Decision ID。

读取扩展说明：本轮按用户明确要求读取 `process/STATE.md`，原因分类为 `human_audit`；因本轮编辑范围限制为 `process/docs/design/*` / `process/docs/features/*`，未写入 `process/state/READ-EXPANSION-LEDGER.ndjson`。

## 2. 能力地图

| Capability ID | 能力域 | 用户价值 | 覆盖计划范围 | Owner Feature |
|---|---|---|---|---|
| CAP-PG-001 | Current State Enforcement | 保证 `STATE.current.json` 保持 refs-only、allowlist、预算受控，防止 agent 写入巨型状态或自造字段 | P0 `STATE.current.json` enforcement、writer/update API、agent 写契约 | FEAT-PG-001 |
| CAP-PG-002 | Ledger Compaction | 控制 ledger 长期增长，保留可恢复索引与审计窗口 | P0 后段 ledger compact / archive / index | FEAT-PG-002 |
| CAP-PG-003 | Project State Governance | 为长期项目提供轻量项目状态、scale、roadmap、milestone 入口，避免污染 current state | P1 `PROJECT.current.json`、`PROJECT-SCALE.yaml`、`ROADMAP.yaml`、`MILESTONES.yaml` | FEAT-PG-003 |
| CAP-PG-004 | Registry-backed Capability / Feature Refs | 防止 `capability_refs` / `feature_refs` 退化为自由字符串，提升迁移和冲突检测可信度 | P1.2a capability source normalization | FEAT-PG-004 |
| CAP-PG-005 | Impact Surface Normalization | 把治理影响面、路径、Feature、Capability 分开，修复冲突检测弱化问题 | P1.3 impact_surface normalization | FEAT-PG-005 |
| CAP-PG-006 | Roadmap Refresh Governance | 在过程归档库内自动刷新机器状态，同时只对发布库输出 stale / follow-up | P1.4 / P1.5 roadmap refresh result、checker、cascade | FEAT-PG-006 |
| CAP-PG-007 | Roadmap Follow-up Tracking | 让 roadmap refresh 产出的后续事项进入 CR tracking，而不混入 release context | P1.6 FU-RF candidate support | FEAT-PG-007 |
| CAP-PG-008 | Project Stale Detection | 检测 roadmap、HLD、测试策略、项目阶段之间的跨对象语义陈旧 | P1.7 stale check | FEAT-PG-008 |
| CAP-PG-009 | Quant-lab Migration Readiness | 用真实样本验证治理模型，不自动修改 quant-lab 发布库 | P2 quant-lab migration | FEAT-PG-009 |

## 3. Feature / Epic 边界

| Feature ID | 名称 | 负责事项 | 不负责事项 | 拥有数据 | 只读数据 | 禁止依赖 |
|---|---|---|---|---|---|---|
| FEAT-PG-001 | Current State Enforcement | `STATE.current.json` allowlist schema、字段预算、写前校验、`update_current_state()`、绕过 writer 路径收敛、agent 写契约 | 不承载 roadmap、deferred、checkpoint、human gate 全文；不新增第二套 state 体系 | `process/state/STATE.current.json` schema 与 writer/update API 约束 | `process/STATE.md` 人类摘要、ledgers、context refs、CP result refs | 禁止直接读取或复制 CR / Story / checkpoint 全文到 current state；禁止 agent 直接写 current state |
| FEAT-PG-002 | Ledger Compaction | 新增 ledger/event compact 命令、retention 执行策略、archive/index 输出 | 不复用 `meta-flow state compact`；不新增 `PROJECT-LEDGER.ndjson` | compaction policy、ledger compact result / archive index | 现有 checkpoint / gate / handoff / run / CR ledgers | 禁止把 compaction 伪装成 state render；禁止破坏审计窗口 |
| FEAT-PG-003 | Project State Governance | `process/project/` scaffold、`PROJECT.current.json`、`PROJECT-SCALE.yaml`、`ROADMAP.yaml`、`MILESTONES.yaml` | 不直接修改 `GATE-PROFILES.json`；不把 roadmap 全文放进 current state | `PROJECT.current.json`、`PROJECT-SCALE.yaml`、`ROADMAP.yaml`、`MILESTONES.yaml` | gate profiles、CR index、state current refs | 禁止 project state 反向写入 `STATE.current.json` 重型字段 |
| FEAT-PG-004 | Capability / Feature Registry | `FEATURE-REGISTRY.yaml`、`CAPABILITY-STATUS.yaml`、引用解析、migration blocked finding | 不从 markdown 或 Python 常量长期消费 registry；不自动创造 capability ID | 标准 Feature / Capability registry | quant-lab capability 来源、CR 记录、unsupported.py、research registry spec | 禁止 migration 将示例值或自由字符串注册为真实 capability |
| FEAT-PG-005 | Impact Surface Normalization | `impact_surface` 有限枚举、`affected_paths`、`feature_refs`、`capability_refs`、migration report、audit/enforce | 不在普通 `cr check` 对历史 CR 刷屏；不把路径继续塞回 impact_surface | impact surface schema、migration report | CR index、feature/capability registry、历史 CR | 禁止 unknown surface 在 enforce 模式继续通过 |
| FEAT-PG-006 | Roadmap Refresh Governance | ROADMAP-REFRESH result schema、checker、cascade、Gate Ledger event、过程归档库自动更新边界 | 不复用 CP result checker；不跨仓原子修改 quant-lab 发布库 | `ROADMAP-REFRESH-*.result.json` / summary、refresh event refs | PROJECT.current、ROADMAP、MILESTONES、CR index、Gate Ledger | 禁止声明跨仓原子事务；禁止同一 cascade 自动改发布库代码 / docs / tests |
| FEAT-PG-007 | FU-RF Candidate Support | `FU-RF` / `SP-RF` / `RA-RF` 编号、tracking regex、模板、状态查询 | 不写入 `RELEASE-CONTEXT` | FU-RF tracking entries and templates | roadmap refresh results、CR tracking summary | 禁止把 roadmap follow-up 混成发布上下文或普通 release 风险 |
| FEAT-PG-008 | Project Stale Check | `meta-flow project stale-check` 或 `meta-flow check project-stale`，跨对象语义陈旧检测 | 不重复 cr-tracking 结构一致性检查 | stale-check result / findings | PROJECT.current、ROADMAP、HLD、TEST-STRATEGY、release docs refs | 禁止在 stale-check 内自动修改被判定陈旧的正式文档 |
| FEAT-PG-009 | Quant-lab Migration | 清理 quant-lab current state、归一 capability、迁移 impact_surface、生成 project objects、stale report 和 FU-RF | 不改 quant-lab 发布库正式代码或正式文档；不与机制实现 CR 混在一起 | quant-lab process-side migration reports / project state objects | quant-lab 发布库只读信息、meta-flow 新治理能力 | 禁止迁移阶段绕过 registry 和 roadmap refresh 边界 |

## 4. 跨 Feature 流程

| Flow ID | 触发 | 参与 Feature | 数据写入 Owner | 失败路径 | 验证入口 |
|---|---|---|---|---|---|
| FLOW-PG-001 | agent / host 更新 current state | FEAT-PG-001 -> ledgers/context/CP refs | FEAT-PG-001 只写 allowlist 字段；重型内容由对应 ledger / result owner 写入 | unknown field audit WARN；enforce ERROR 并阻断推进 | state check、writer unit tests、contract guardrail |
| FLOW-PG-002 | 项目级状态初始化或刷新 | FEAT-PG-003 -> FEAT-PG-001 | FEAT-PG-003 写 `PROJECT.current.json`；FEAT-PG-001 只保存 `project_state_ref` | project object 超预算或缺字段时刷新失败，不回写 current state | project state schema check、workspace bootstrap tests |
| FLOW-PG-003 | roadmap refresh cascade | FEAT-PG-006 -> FEAT-PG-003 / FEAT-PG-007 / FEAT-PG-008 | FEAT-PG-006 写 refresh result；FEAT-PG-003 写过程归档项目对象；FEAT-PG-007 写 follow-up 候选 | 发布库陈旧只进入 `must_check` / `stale_items` / `follow_up_candidates` | roadmap-refresh checker、Gate Ledger check、stale-check |
| FLOW-PG-004 | CR impact migration | FEAT-PG-005 -> FEAT-PG-004 | FEAT-PG-005 写 normalized impact fields；FEAT-PG-004 校验 refs | refs 无法解析时 blocked finding，不创造 ID | impact migration report、capability / feature check |
| FLOW-PG-005 | quant-lab 真实迁移验证 | FEAT-PG-009 -> FEAT-PG-004 / FEAT-PG-005 / FEAT-PG-006 / FEAT-PG-008 | FEAT-PG-009 写迁移报告和过程侧对象 | 发布库变更需求转 FU-RF 或正式 CR | quant-lab dry-run report、state check、stale report |

## 5. 共享能力

| Shared ID | 名称 | 使用方 | Owner | 调用方向 | 降级策略 |
|---|---|---|---|---|---|
| SH-PG-001 | Allowlist Schema Validation | state writer、state check、current-state gate、agent contract guardrail | FEAT-PG-001 | writer/checker 消费 schema；其他 Feature 只引用结果 | audit 模式 WARN；enforce 模式阻断 |
| SH-PG-002 | Registry Resolver | impact migration、roadmap refresh、quant-lab migration、capability claims check | FEAT-PG-004 | Consumer -> registry resolver -> registry files | registry 缺失或 ID 未注册时输出 blocked finding |
| SH-PG-003 | Process-only Cascade Boundary | roadmap refresh、project stale check、quant-lab migration | FEAT-PG-006 | refresh 写过程归档库；发布库只生成检查项 | 自动写入失败时 result=`BLOCKED`，不跨仓补偿 |
| SH-PG-004 | Follow-up Candidate Contract | roadmap refresh、stale check、CR tracking | FEAT-PG-007 | refresh/stale -> FU-RF candidate -> CR tracking | FU-RF 校验失败时保持 stale item，不写 release context |

## 6. 待确认边界

当前实施计划已确认主方向；以下决策项用于后续 CP3 / CR 分解时显式追踪，不表示本轮存在阻塞。

| Decision ID | 决策类型 | 问题 | 推荐方案 | 备选方案 | 推荐 / 备选优劣 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-BP-001 | architecture | 是否将 project state 独立为 `PROJECT.current.json`，而不是扩展 `STATE.current.json` | 独立对象，`STATE.current.json` 只保存 `project_state_ref` | 继续扩展 current state | 推荐方案降低 current state 膨胀风险；备选实现更快但违背瘦身目标 | 影响 P0/P1 边界和 schema 预算 | 若 project state 也膨胀，收紧 `PROJECT.current.json` allowlist 和预算 |
| DQ-BP-002 | architecture | roadmap refresh 是否自动修改 quant-lab 发布库 | 不自动修改，只输出 stale / follow-up | 尝试跨仓自动更新 | 推荐方案避免跨仓事务与误写；备选自动化程度高但风险不可控 | 影响发布库安全边界和用户授权 | 若未来获得显式发布库写授权，仍需单独 CR 和 human gate |
| DQ-BP-003 | implementation | capability / feature refs 的真相源是否固定为 YAML registry | 固定为 `CAPABILITY-STATUS.yaml` / `FEATURE-REGISTRY.yaml` | 长期消费 markdown register 或 Python 常量 | 推荐方案利于机器校验；备选迁移更省事但继续漂移 | 影响 quant-lab migration 和 impact conflict 检测 | 若 registry 无法覆盖领域差异，先扩展 registry schema，不退回自由字符串 |
