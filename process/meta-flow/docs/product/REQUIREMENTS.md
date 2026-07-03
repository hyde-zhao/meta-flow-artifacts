---
status: baseline
version: "1.0"
created_at: "2026-07-02"
owner: "meta-pm"
cr_ref: "CR-037"
ready_for_design: true
approval_context: "source implementation plan accepted; user authorized pausing unfinished CR-036 and activating CR-037 on 2026-07-02; CR-037 remains CP2 pending and does not authorize implementation, CP5, runtime, production write, or quant-lab release repo changes"
source_use_cases:
  - UC-PG-001
  - UC-PG-002
  - UC-PG-003
  - UC-PG-004
  - UC-PG-005
  - UC-PG-006
  - UC-PG-007
source_plan: "process/docs/design/META-FLOW-PROJECT-GOVERNANCE-STATE-ENFORCEMENT-IMPLEMENTATION-PLAN-2026-07-02.md"
---

# Meta Flow 项目治理与状态强制需求

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 | 文档处理方式 |
|---|---|---|---|---|
| 1.1 | 2026-07-02 | host-orchestrator | 同步 CR-037 已激活、CR-036 暂停未完成和 CP2 pending 不授权实现的状态语义 | 小范围状态语义同步 |
| 1.0 | 2026-07-02 | meta-pm | 基于已批准实施计划提取产品需求基线 | 初始化长期可追踪需求基线 |

## 功能需求

| ID | 需求描述 | 优先级 | 验收条件 | 来源场景 |
|---|---|---|---|---|
| REQ-PG-001 | 系统必须为 `STATE.current.json` 建立 allowlist schema，仅允许声明字段写入。 | P0 | Given 待写入 patch 包含未声明顶层 key When 执行 current state 写入 Then audit 模式输出 WARN，enforce 模式拒绝写入并返回可追踪错误。 | UC-PG-001 |
| REQ-PG-002 | 系统必须为 `STATE.current.json` 合法字段执行字段级预算校验。 | P0 | Given `next_action`、`source_refs` 或 `open_risks` 超过预算 When 执行写入 Then 写入失败，错误指出字段和预算原因。 | UC-PG-001 |
| REQ-PG-003 | 系统必须提供受控 current state 更新入口，支持 deep-merge patch、actor 和 reason。 | P0 | Given 调用 `update_current_state(project_root, patch, actor, reason)` When patch 合法 Then 完整 state 写入前后均通过 schema 和预算校验。 | UC-PG-001 |
| REQ-PG-004 | 内部 current state 更新路径不得绕过受控 writer。 | P0 | Given CR lifecycle 或 state-router 需要更新 current state When 执行更新 Then 不存在直接 `write_text()` 写 `STATE.current.json` 的路径。 | UC-PG-001 |
| REQ-PG-005 | Agent / Skill 写契约必须禁止直接编辑 `STATE.current.json`。 | P0 | Given 交付规则和 state-router 文档更新完成 When 审查 agent / skill contract Then 明确声明合法更新入口和重型状态落盘位置。 | UC-PG-001 |
| REQ-PG-006 | `STATE.current.json` allowlist 必须包含 `project_state_ref`，用于引用项目级状态。 | P0 | Given P1 项目状态启用 When 写入 `project_state_ref` Then current state schema 校验通过且字段预算生效。 | UC-PG-002 |
| REQ-PG-007 | 系统必须创建 `process/project/` scaffold。 | P1 | Given 初始化或迁移 workspace When 执行 bootstrap Then `process/project/` 被创建且测试覆盖该行为。 | UC-PG-002 |
| REQ-PG-008 | 系统必须提供 refs-only 的 `PROJECT.current.json`，并限制大小。 | P1 | Given 项目状态包含 roadmap / milestone / health / deferred refs When 执行 project state check Then 文件不超过 16KB 且不包含 roadmap 全文或 deferred 详情。 | UC-PG-002 |
| REQ-PG-009 | 系统必须通过 `PROJECT-SCALE.yaml` 表达 `lite / standard / full` 三档项目规模和 gate profile bias。 | P1 | Given 项目设为 `full` When 读取 project scale Then 可看到默认 gate profile bias 和 reason，且未修改 `GATE-PROFILES.json`。 | UC-PG-003 |
| REQ-PG-010 | `feature_refs` 必须引用标准 `FEATURE-REGISTRY.yaml` 中已注册 feature ID。 | P1 | Given 新 CR 或 migration 声明 `feature_refs` When 运行 feature check Then 未注册 ID 被报告为 blocked finding 或 follow-up。 | UC-PG-004 |
| REQ-PG-011 | `capability_refs` 必须引用标准 `CAPABILITY-STATUS.yaml` 中已注册 capability ID。 | P1 | Given 新 CR 或 migration 声明 `capability_refs` When 运行 capability check Then 未注册 ID 被报告为 blocked finding 或 follow-up，不被自动创建。 | UC-PG-004 |
| REQ-PG-012 | 系统必须将 `impact_surface` 限定为治理面枚举，并将路径写入 `affected_paths`。 | P1 | Given 输入包含 `process/checks` 或 `trading/strategy_runner` When 运行 migration Then 路径进入 `affected_paths`，可映射治理面进入 `impact_surface`。 | UC-PG-005 |
| REQ-PG-013 | 系统必须为历史 impact surface 漂移生成 migration report，避免普通 `cr check` 对历史数据刷屏。 | P1 | Given 历史 CR 含路径类 impact surface When 运行常规 `cr check` Then 不刷屏；When 运行 migration report Then 输出可追踪 finding。 | UC-PG-005 |
| REQ-PG-014 | Roadmap refresh 必须使用独立 result schema 和 checker，不复用 CP result checker。 | P1 | Given 生成 `ROADMAP-REFRESH-<id>.result.json` When 运行 roadmap-refresh checker Then 校验 refresh schema、decision、source、machine_updates、follow-up 字段。 | UC-PG-006 |
| REQ-PG-015 | Roadmap refresh decision 必须使用固定枚举。 | P1 | Given refresh 完成 When 写入 result Then decision 只能为 `NO_CHANGE`、`UPDATED`、`UPDATED_WITH_DOC_IMPACTS`、`BLOCKED` 或 `FAILED`。 | UC-PG-006 |
| REQ-PG-016 | Roadmap refresh cascade 自动写入范围必须限制在过程归档库。 | P1 | Given refresh 发现 quant-lab 发布库文档陈旧 When 执行 cascade Then 发布库文件不被自动修改，陈旧项进入 `must_check`、`stale_items` 或 `follow_up_candidates`。 | UC-PG-006 |
| REQ-PG-017 | Roadmap refresh 事件必须写入 `GATE-LEDGER` 或等价 CR 相关 ledger。 | P1 | Given refresh result 写入成功 When 追加 ledger Then ledger event 包含 refresh gate、status 和 result ref。 | UC-PG-006 |
| REQ-PG-018 | FU-RF、SP-RF、RA-RF 候选必须被跟踪模板和检查器接受。 | P1 | Given roadmap refresh 生成 follow-up When 运行 cr-tracking Then `FU-RF001-001`、`SP-RF001-001`、`RA-RF001-001` 被识别且不写入 `RELEASE-CONTEXT`。 | UC-PG-006 |
| REQ-PG-019 | 系统必须提供 project stale check，识别跨对象语义陈旧。 | P1 | Given project state 进入 runtime 或 paper/live 阶段但 HLD / TEST-STRATEGY 仍停留 backtest-only When 运行 stale check Then 输出 stale finding 和 follow-up 候选。 | UC-PG-006 |
| REQ-PG-020 | quant-lab 迁移必须清理污染的 `STATE.current.json` 并通过 state check。 | P2 | Given quant-lab current state 含 forbidden / unknown / oversized 字段 When 执行迁移 Then 清理后通过 state check，并保留迁移报告。 | UC-PG-007 |
| REQ-PG-021 | quant-lab capability 来源必须归一到标准 registry。 | P2 | Given quant-lab capability 线索分散在 CR、代码和研究注册文档中 When 执行迁移 Then 真实 capability 被核查并进入标准 registry 或 blocked follow-up。 | UC-PG-007 |
| REQ-PG-022 | quant-lab 迁移必须输出 roadmap stale report 和 FU-RF 候选，且不得自动改发布库。 | P2 | Given quant-lab 发布库存在陈旧 HLD / TEST / release 文档 When 执行迁移 Then 只输出 stale report 和 FU-RF 候选，不修改发布库文件。 | UC-PG-007 |

## 约束需求

| ID | 需求描述 | 优先级 | 验收条件 | 来源 |
|---|---|---|---|---|
| REQ-PG-C001 | 不得新增第二套上下文治理、影响分析、capability 命名、result 或 ledger 体系。 | P0 | Given 设计或实现 diff When 审查新增对象 Then 不出现与现有体系平行的新治理体系。 | 明确排除 |
| REQ-PG-C002 | 不得声明或实现跨仓原子事务。 | P1 | Given roadmap refresh 涉及过程归档库和 quant-lab 发布库 When 审查实现 Then 仅过程归档库自动写入，发布库仅生成 follow-up。 | UC-PG-006 |
| REQ-PG-C003 | 不得直接修改 `process/policies/GATE-PROFILES.json` 表达 project scale。 | P1 | Given project scale 实现完成 When 审查文件变更 Then gate profile bias 位于 `PROJECT-SCALE.yaml`。 | UC-PG-003 |
| REQ-PG-C004 | migration 不得凭空创造 capability 或 feature ID。 | P1 | Given registry 缺失目标 ID When 运行 migration Then 输出 blocked finding 或 FU-RF，不创建未经确认 ID。 | UC-PG-004 |
| REQ-PG-C005 | Ledger compaction 不得复用 `meta-flow state compact` 名称。 | P0 | Given 新增 ledger compaction 命令 When 检查 CLI Then 命令为 `ledger compact` 或 `event compact` 等独立名称。 | 实施计划 P0 后段 |

## 非功能需求

| ID | 需求描述 | 优先级 | 验收条件 | 来源 |
|---|---|---|---|---|
| REQ-PG-NF001 | 所有新增机器状态对象必须 refs-only、可校验、可预算。 | P0 | Given state / project state 写入 When 运行 checker Then 长正文、历史详情和大型列表被拒绝或转移到 ledger / result / summary。 | UC-PG-001, UC-PG-002 |
| REQ-PG-NF002 | 所有检查失败必须输出可定位的字段、路径或 ID。 | P1 | Given schema、registry、surface 或 refresh 检查失败 When 查看结果 Then 能定位失败字段、输入来源和建议处理方式。 | UC-PG-004, UC-PG-005, UC-PG-006 |
| REQ-PG-NF003 | P0 enforcement 必须支持 audit -> enforce 灰度。 | P0 | Given unknown field 出现 When 处于 audit 阶段 Then WARN 不阻断；When 处于 enforce 阶段 Then ERROR 阻断。 | UC-PG-001 |
| REQ-PG-NF004 | quant-lab 迁移必须可重复验证。 | P2 | Given 迁移完成 When 重新运行 state / capability / feature / capability-claims / stale checks Then 结果与迁移报告一致。 | UC-PG-007 |

## 风险与假设

| ID | 类型 | 内容 | 关联需求 | 缓解措施 |
|---|---|---|---|---|
| RA-PG-001 | RISK | allowlist 可能误伤存量但仍合理的 current state 字段。 | REQ-PG-001 | 使用 audit -> enforce 灰度，先 WARN 收集真实字段。 |
| RA-PG-002 | RISK | 历史 impact surface 漂移数量大，直接阻断会干扰当前开发。 | REQ-PG-013 | 历史静默，提供独立 migration report。 |
| RA-PG-003 | RISK | capability ID 归一时真实来源分散，容易误建自由字符串。 | REQ-PG-011, REQ-PG-021 | 只允许 registry 引用；缺失项输出 blocked finding / FU-RF。 |
| RA-PG-004 | RISK | Roadmap refresh 跨仓边界不清会误改发布库。 | REQ-PG-016, REQ-PG-C002 | 明确自动写入只限过程归档库。 |
| RA-PG-005 | ASSUMPTION | 已批准实施计划是本轮产品基线事实来源。 | 全部 | 本文档引用 source_plan；若计划变更，必须通过后续 CR 增量更新。 |

## 里程碑建议

| 里程碑 | 包含需求 | 交付物 | 前置里程碑 |
|---|---|---|---|
| M0：Second-system Guardrail | REQ-PG-C001 | 设计 / 实现审查阻断第二套上下文治理、影响分析、capability 命名、result 或 ledger 体系 | 无 |
| M1：Current State Enforcement | REQ-PG-001..006, REQ-PG-NF001, REQ-PG-NF003 | allowlist、field budgets、update API、写契约、audit/enforce gate | 无 |
| M1.5：Ledger Hygiene Boundary | REQ-PG-C005 | ledger compact / event compact 与 state compact 的命令边界和帮助文档职责区分 | M1 |
| M2：Project Governance Objects | REQ-PG-007..009, REQ-PG-NF001 | `process/project/`、`PROJECT.current.json`、`PROJECT-SCALE.yaml`、ROADMAP / MILESTONES 基线 | M1 |
| M3：Reference and Impact Normalization | REQ-PG-010..013, REQ-PG-C004, REQ-PG-NF002 | capability / feature registry 校验、impact surface migration report | M2 |
| M4：Roadmap Refresh and Follow-up | REQ-PG-014..019, REQ-PG-C002 | ROADMAP-REFRESH result/checker、GATE-LEDGER event、FU-RF 支持、stale check | M3 |
| M5：quant-lab Migration | REQ-PG-020..022, REQ-PG-NF004 | quant-lab state 清理、capability 归一、impact surface 迁移、stale report | M4 |

## 明确排除项

- 不做代码实现细节、模块拆分、CLI 参数设计或数据模型字段类型细化；这些属于设计 / 实现阶段。
- 不在本轮产品文档中修改已批准设计文档。
- 不把 quant-lab 迁移和 capability 新能力实现混为同一个交付范围。
- 不把发布库 stale finding 自动升级为发布库写入授权。
