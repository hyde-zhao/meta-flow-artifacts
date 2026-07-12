---
status: baseline
version: "1.3"
created_at: "2026-07-02"
owner: "meta-pm"
cr_ref: "CR-037"
active_change_ref: "CR-046"
source_use_cases:
  - UC-PG-001
  - UC-PG-002
  - UC-PG-003
  - UC-PG-004
  - UC-PG-005
  - UC-PG-006
  - UC-PG-007
  - UC-EI-001
  - UC-EI-002
  - UC-EI-003
  - UC-EI-004
  - UC-EI-005
source_requirements: "process/docs/product/REQUIREMENTS.md"
---

# Meta Flow 项目治理与状态强制 Story Map

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 | 文档处理方式 |
|---|---|---|---|---|
| 1.0 | 2026-07-02 | meta-pm | 基于产品场景和需求建立用户故事地图 | 初始化长期可追踪产品规划基线 |
| 1.2 | 2026-07-11 | meta-pm | 增量追加 CR-046 的 5 个活动、7 个 outcome Story 与 release slice；保留 ST-PG-001..013 | 原文档增量更新 |
| 1.3 | 2026-07-12 | meta-pm | CR-046 CP2 scope rework R2：不新增或重编号 Story，扩展 ST-EI-002/004/006/007 的 requirement refs 与验收语义 | 原文档增量更新 |

## 用户活动与任务

| Activity ID | 用户活动 | 目标用户 | 用户任务 | 来源场景 |
|---|---|---|---|---|
| ACT-PG-01 | 保护轻量运行态入口 | P-01, P-02 | 在写入 current state 前识别非法字段、超预算字段和绕过 writer 的路径 | UC-PG-001 |
| ACT-PG-02 | 建立项目级治理状态 | P-01, P-04 | 用 refs-only 项目状态表达项目定位、规模、milestone 和 roadmap | UC-PG-002, UC-PG-003 |
| ACT-PG-03 | 归一治理引用语义 | P-01, P-03 | 将 capability / feature / impact surface 从自由文本归一到可检查对象 | UC-PG-004, UC-PG-005 |
| ACT-PG-04 | 刷新 roadmap 且不越权写发布库 | P-01, P-04 | 生成 roadmap refresh result、ledger event、stale finding 和 follow-up 候选 | UC-PG-006 |
| ACT-PG-05 | 迁移真实长期项目 | P-03, P-04 | 使用新机制迁移 quant-lab 并输出可复现验证证据 | UC-PG-007 |
| ACT-EI-01 | 证明门禁时序 | P-05, P-01 | 重建 checkpoint/gate/result/state/ledger 的有效因果链并拒绝非法顺序 | UC-EI-001 |
| ACT-EI-02 | 证明真实平台调度 | P-05, P-06 | 追踪 attempt、receipt、retry、supersession 与 terminal closure | UC-EI-002 |
| ACT-EI-03 | 重放历史证据 | P-05, P-03 | 用已识别 checker 输出 as-executed/current-replay 双口径 | UC-EI-003 |
| ACT-EI-04 | 度量 workflow 成本 | P-06, P-04 | 区分 measured、estimated 和 unavailable token usage | UC-EI-004 |
| ACT-EI-05 | 验收 append-only pilot | P-03, P-05 | 在不改业务实现的前提下迁移并重放 CR-163 证据 | UC-EI-005 |

## Story 列表

| Story ID | 用户故事 | 优先级 | 来源需求 | 关键验收 |
|---|---|---|---|---|
| ST-PG-001 | 作为维护者，我要用 allowlist 和字段预算约束 `STATE.current.json`，以防止轻量入口继续膨胀。 | P0 | REQ-PG-001, REQ-PG-002, REQ-PG-NF003 | unknown field 在 audit WARN、enforce ERROR；超预算字段被定位并拒绝。 |
| ST-PG-002 | 作为维护者，我要所有 current state 写入走受控更新入口，以便写入前后都可校验。 | P0 | REQ-PG-003, REQ-PG-004 | 内部绕过 writer 的路径改为受控 API；直接写入路径被测试覆盖。 |
| ST-PG-003 | 作为 Agent / Skill 作者，我要清楚知道 current state 写契约，以避免把重型状态写入轻量状态。 | P0 | REQ-PG-005, REQ-PG-NF001 | 规则、state-router、README 同步写明禁止直编和替代落盘位置。 |
| ST-PG-004 | 作为维护者，我要用 `PROJECT.current.json` 表达 refs-only 项目状态，以承载长期项目治理。 | P1 | REQ-PG-006, REQ-PG-007, REQ-PG-008 | bootstrap 创建 `process/project/`；project state ref 可从 current state 访问；大小预算通过。 |
| ST-PG-005 | 作为审批者，我要看到项目规模和 gate profile bias 的原因，以便理解后续门禁默认倾向。 | P1 | REQ-PG-009, REQ-PG-C003 | `PROJECT-SCALE.yaml` 记录三档规模、bias 和 reason；不修改 gate profile 源文件。 |
| ST-PG-006 | 作为迁移执行者，我要 capability / feature refs 只引用标准 registry，以避免能力名称漂移。 | P1 | REQ-PG-010, REQ-PG-011, REQ-PG-C004 | 未注册 ID 进入 blocked finding 或 FU-RF；不自动创造 ID。 |
| ST-PG-007 | 作为维护者，我要把 impact surface、路径、feature、capability 分开，以恢复冲突检测语义。 | P1 | REQ-PG-012, REQ-PG-013 | 新 CR 使用治理面枚举；历史漂移有 migration report；普通 cr check 不刷屏。 |
| ST-PG-008 | 作为维护者，我要 roadmap refresh 生成独立 result 和 checker，以区别于 CP gate。 | P1 | REQ-PG-014, REQ-PG-015 | result decision 使用固定枚举；checker 校验 refresh 专属 schema。 |
| ST-PG-009 | 作为审批者，我要 refresh cascade 只自动更新过程归档库，以避免误改发布库。 | P1 | REQ-PG-016, REQ-PG-017, REQ-PG-C002 | 发布库陈旧项只进入 must_check / stale_items / follow_up_candidates；ledger 记录 refresh event。 |
| ST-PG-010 | 作为维护者，我要 RF 前缀 follow-up 和 stale check，以持续追踪 roadmap 影响。 | P1 | REQ-PG-018, REQ-PG-019 | FU-RF / SP-RF / RA-RF 被 tracking 接受；stale check 输出跨对象陈旧 finding。 |
| ST-PG-011 | 作为迁移执行者，我要用新治理机制迁移 quant-lab，以验证机制能服务真实长期项目。 | P2 | REQ-PG-020, REQ-PG-021, REQ-PG-022, REQ-PG-NF004 | quant-lab state / capability / feature / stale 检查可复跑；发布库未被自动改写。 |
| ST-PG-012 | 作为维护者，我要 ledger compaction 与 state compact 在命令和职责上明确分离，以避免长期审计数据治理和 current state 渲染语义混淆。 | P0.5 | REQ-PG-C005 | ledger compaction 使用 `ledger compact` 或 `event compact`；不复用 `state compact`；帮助文档说明二者职责差异。 |
| ST-PG-013 | 作为维护者，我要所有 CR-037 子能力复用现有 state、context、result、ledger 和 registry 体系，以避免为整改再造第二套治理机制。 | P0 | REQ-PG-C001 | 设计和实现审查能识别 hot/warm/cold、roadmap_impact、PROJECT-LEDGER、自由 capability 命名空间等平行机制，并阻断。 |
| ST-EI-001 | 作为工作流审计者，我要机器校验 gate 时序和条件式批准，以便拒绝提前推进或回填式历史证明。 | P0 | REQ-EI-001, REQ-EI-002, REQ-EI-NF001 | chronology negative fixtures 100% 被拒绝；合法条件式批准可重放。 |
| ST-EI-002 | 作为证据生产者，我要每个 dispatch attempt 都有可验证 provenance、retry/supersession 和 terminal closure，以便证明平台调用真实发生。 | P0 | REQ-EI-003..005, REQ-EI-019, REQ-EI-023, REQ-EI-C001 | receipt 可用时验证；仅 session 可观察而仓库无 receipt 时披露 `session-observed/repository-unverifiable`；所有 attempts 有终态；compaction/restore 不混淆 event/dispatch/attempt/run 或改变 terminal selection。 |
| ST-EI-003 | 作为工作流审计者，我要 CP result 与最终 attempt、输入 hash 和其他真相源一致，以便不再只依赖 schema PASS。 | P0 | REQ-EI-006..008 | input/check attempt 覆盖 100%；final correlation 合法；跨真相源冲突被阻断。 |
| ST-EI-004 | 作为维护者，我要 delivered finalization、workflow health 与 read expansion 授权可检查，以便关闭残留 active refs 和隐式全文读取。 | P0 | REQ-EI-009, REQ-EI-010, REQ-EI-019, REQ-EI-021, REQ-EI-C003 | delivered active refs 为 0；health 目标存在且归属当前 CR；compaction 保留 health refs；未授权 expansion 为 0；机器 audit report 分离 event/attempt/thread/outcome/token 指标并携带 provenance/input hashes。 |
| ST-EI-005 | 作为成本分析者，我要诚实区分 measured、estimated 和 unavailable token usage，以便用可信数据优化上下文和调度。 | P0 | REQ-EI-011..013, REQ-EI-C001 | measurement status 100% 覆盖；估算不冒充实测；成本可归属聚合。 |
| ST-EI-006 | 作为审计者，我要记录 checker identity 并输出双口径 replay，以便解释 checker 演进而不改写历史。 | P0 | REQ-EI-014, REQ-EI-015, REQ-EI-021, REQ-EI-022, REQ-EI-C002, REQ-EI-NF002 | checker/version/hash 可追踪；as-executed/current-replay 均保留；机器 audit report 的计数维度与 provenance 可复核；CR-046 R1 null-provenance results 保留且 strict profile 不得报 fully replayable。 |
| ST-EI-007 | 作为迁移执行者，我要以 append-only 方式迁移 quant-lab CR-163 证据，以便用 23/23 replay 验收机制且业务源码不变。 | P1 | REQ-EI-016..020, REQ-EI-NF002 | pilot 消费通用 versioned post-close correction lifecycle；correction 限定允许字段/范围并带 author/reason/evidence/supersedes/audit trail；原历史不变；23/23 PASS；lineage 业务源码 diff 为 0。 |

## 推荐发布切片

| Slice ID | 切片名称 | 包含 Story | 用户价值 | 前置依赖 |
|---|---|---|---|---|
| SL-PG-00 | Second-system Guardrail | ST-PG-013 | 先冻结“不造第二套机制”的跨切面边界，后续所有整改切片默认复用既有 state/context/result/ledger/registry 体系 | 无 |
| SL-PG-01 | P0 State Enforcement | ST-PG-001, ST-PG-002, ST-PG-003 | 先关闭 current state 污染入口，避免后续治理对象继续被错误状态拖累 | SL-PG-00 |
| SL-PG-01A | P0.5 Ledger Hygiene Boundary | ST-PG-012 | 先冻结 ledger compact 与 state compact 的命令边界，避免后续实现混淆审计数据压缩与 current state 渲染 | SL-PG-01 |
| SL-PG-02 | P1 Project State Foundation | ST-PG-004, ST-PG-005 | 建立长期项目治理的最小机器状态和规模偏好 | SL-PG-01 |
| SL-PG-03 | P1 Reference Normalization | ST-PG-006, ST-PG-007 | 恢复 capability、feature、impact surface 的可检查语义 | SL-PG-02 |
| SL-PG-04 | P1 Roadmap Refresh | ST-PG-008, ST-PG-009, ST-PG-010 | 让 roadmap 变更可自动刷新过程状态，同时控制跨仓风险 | SL-PG-03 |
| SL-PG-05 | P2 quant-lab Migration | ST-PG-011 | 用真实长期项目验证治理机制，并输出迁移证据 | SL-PG-04 |
| SL-EI-01 | Evidence Integrity Core | ST-EI-001, ST-EI-002, ST-EI-003, ST-EI-004 | 先让 chronology、dispatch、CP correlation、state/read 边界成为可执行治理 contract | CR-046 CP2/CP3/CP5 |
| SL-EI-02 | Replay and Cost Observability | ST-EI-005, ST-EI-006 | 在可信核心上补齐 token telemetry 与 checker 双口径重放 | SL-EI-01 |
| SL-EI-03 | quant-lab CR-163 Acceptance Pilot | ST-EI-007 | 用 append-only 真实样本证明当前 checker 23/23 可重放 | SL-EI-02；独立 pilot 授权 |

## 规划边界

- Story 只表达用户 outcome，不规定具体代码模块拆分。
- 每个 Story 必须回链至少一个 UC 和 REQ。
- P2 迁移必须等待 P0 / P1 机制可用后再执行。
- 任何发布库写入都不由 roadmap refresh 自动授权。
- CR-046 CP2 只确认产品/场景/范围基线，不授权实现、runtime、credentials、publish、commit/push 或 quant-lab business-code 修改。
- ST-EI-007 是 append-only process-evidence pilot，不得承担 quant-lab lineage 业务功能重做。
