---
checkpoint_id: "CP2-CR166-WALK-FORWARD-OOS-EVIDENCE-SCOPE"
checkpoint_name: "CR166 Requirement, Scenario and Scope Baseline"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-07-13T10:40:00+08:00"
reviewed_by: "user"
reviewed_at: "2026-07-13T10:46:00+08:00"
auto_check_result: "process/checks/CP2-CR166-WALK-FORWARD-OOS-EVIDENCE-SCOPE.result.json"
decision_brief_profile: "compact"
target:
  phase: "requirement-clarification"
  story_id: ""
  artifacts: ["docs/product/USE-CASES.md", "docs/product/REQUIREMENTS.md", "docs/product/SCENARIOS.yaml", "docs/product/TEST-MATRIX.md", "docs/product/STORY-MAP.md", "docs/product/MVP-SCOPE.md", "docs/product/RELEASE-SLICES.md", "docs/product/BACKLOG.md"]
---

# CP2 CR166 Walk-forward / OOS Evidence Scope 人工审查

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP0-CR-166-BOOTSTRAP.result.json` | PASS | 0 | CR032 stale、候选映射、分支、Stage/权限边界完成。 |
| `process/checks/CP1-CR166-WALK-FORWARD-OOS-USE-CASE-COMPLETENESS.result.json` | PASS | 0 | 8/8 产品基线增量更新，10/10 P0 场景有覆盖。 |
| `process/checks/CP2-CR166-WALK-FORWARD-OOS-EVIDENCE-SCOPE.result.json` | PASS | 0 | 范围、QAC、兼容、授权和 CP7 commitments 可进入人工确认。 |

## Decision Brief

### 审批者摘要

| 字段 | 内容 |
|---|---|
| 本次确认服务的整体目标 | 在不连接生产数据存储、不执行真实研究运行的前提下，批准 CR166 fixture/static C2 producer foundation 的产品范围、fail-closed、兼容和 claim ceiling。 |
| 推荐动作 | `approve`：接受 DQ-CP2-CR166-001..004 推荐方案，进入 CP3 HLD/ADR；继续使用用户要求的 host inline/no-subagent 执行方式。 |
| approve 后会发生什么 | 主编排器进入 CP3，冻结 C2 envelope、fold/leakage policy、component registry、event applicability 与三 consumer 集成契约；CP3 完成后停在下一人工门禁。 |
| approve 不授权什么 | 不授权源码/测试实现（仍需 CP5）、生产数据/凭据/NAS/provider、真实 folds/OOS、runtime/external framework、broker/trading、publish/deploy 或 Git remote write。 |
| 不确认会阻塞什么 | CP3/HLD、正式 Story/DAG、CP5 设计证据及后续实现全部保持阻断。 |

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP2-CR166-WALK-FORWARD-OOS-EVIDENCE-CONTEXT.yaml` |
| capsule 状态 | ready |
| read_profile | compact |
| 默认读取策略 | capsule + CR summary + route plan first；只在缺失、冲突、人工审计或深度设计时扩展。 |
| 全文档读取扩展 | 4 次：CR166 正式范围、CR032 stale、CR161 mapping 与本轮 discussion audit；均登记 `READ-EXPANSION-LEDGER.ndjson`。 |
| 默认禁止读取 | archive、历史 Story/LLD、完整 transcript；CP2 前不读 CR166 DEVELOPMENT-PLAN。 |
| 缺失 / waiver | 0 / 0 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| 用户两轮审核 | 当前对话 + discussion log | scanned | 6 | 0 | 6 个 SGQ 已 confirmed；不替代 formal CP2。 |
| STATE pending queue | `process/state/STATE.current.json` | scanned | 0 | 0 | 启动前无其他 active gate；CR166 gate 将在本轮打开。 |
| CP0/CP1/CP2 result | `process/checks/CP*-CR166*` | scanned | 0 | 0 | 自动结果均 PASS。 |
| 产品基线 | `docs/product/*` 8 files | scanned | 4 | 4 | scope、QAC、compatibility、authorization 需要整体 gate 批准。 |
| CP3/HLD/ADR/Story/LLD | 尚未生成 | n/a | 0 | 0 | CP2 前按 route 正常阻断。 |

### 决策分层

| 分类 | 数量 | 处理方式 |
|---|---:|---|
| 必须用户决策 | 4 | 进入下方待人工决策清单。 |
| 高风险策略确认 | 2 | DQ-001 Stage claim；DQ-004 deny-default/gate sequence。 |
| agent 默认处理 | 3 | canonical field ordering、reason-code 命名、内部模块布局由 CP3 提案，但不得改变 CP2 ceiling。 |
| 仅审计记录 | 6 | SGQ-CR166-001..006 已由用户审核意见确认。 |

### 已解决 Scenario Gray Areas

| SGQ | 结论 | 基线化结果 |
|---|---|---|
| SGQ-CR166-001 | Stage bridge | Stage2 complete；Stage3 not-started/not-authorized。 |
| SGQ-CR166-002 | Future extension | versioned typed components；C3/C4 calculators=0。 |
| SGQ-CR166-003 | Fixture priority | daily + ML=P0；event=P1 applicability/N/A。 |
| SGQ-CR166-004 | Boundary split | zero-dereference + determinism=P0；deep runtime resolver=P1。 |
| SGQ-CR166-005 | Regression | CR166-introduced failures=0；CR165 rebaseline touched paths itemized if any。 |
| SGQ-CR166-006 | CP8 claims | Stage2 complete=true；Stage3 started=false；real evidence available=false。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| `DQ-CP2-CR166-001` | scope | 是否批准 CR166 为 fixture/static C2 foundation，并冻结 Stage2 complete / Stage3 not-started / real-evidence unavailable ceiling？ | 批准；只接收显式 fixture/static folds，不连接生产数据，不把 producer foundation 解释为真实 evidence availability。 | A. 暂停 C2，保持 typed_unavailable；B. 扩为真实数据 Stage3（需拒绝本 gate 并另起授权 CR）。 | 推荐方案先补齐真实架构缺口且不扩大数据风险；暂停无当前价值；混入 Stage3 会改变权限、数据、验证和回滚边界。 | 决定 HLD 的输入/输出、CP8 声明与是否存在过度准入 claim。 | scope 或 Stage flag 改变回退 CP2；任何真实数据需求立即 BLOCKED。 |
| `DQ-CP2-CR166-002` | risk_acceptance | 是否批准 8 类 P0 fail-closed、daily+ML 2 个 P0 fixture 族和 12 项 QAC？ | 批准；缺 fold、时间逆序、purge、embargo、metric、lineage、unauthorized ref、hash 8/8；10 reruns→1 hash；consumer=3/3。 | A. 把 unauthorized ref/hash 降 P1；B. 缩为 daily-only；C. 增加 event 为 P0。 | 推荐方案把零解引用与确定性保留为 foundation 基础，同时不要求深度 runtime resolver；降级会削弱审计性，扩大 event 会引入未冻结语义。 | 决定 CP3 test contract、CP7 required evidence 和完成定义。 | 数量/优先级变化回退 CP2；算法细节可在不降低 QAC 时由 CP3 收敛。 |
| `DQ-CP2-CR166-003` | architecture | 是否批准 versioned typed component 扩展与 event P1 applicability 边界？ | 批准 stable header + registered components；C3/C4 仅预留、calculator=0；event 未冻结时显式 N/A。 | A. C3+C4 同 CR 实现；B. 不预留扩展；C. 交付 event 空壳 producer。 | 推荐方案避免 C3/C4 未来破坏兼容，又不制造虚假 event 覆盖；扩大实现重复数据合同，取消扩展会导致后续迁移。 | 决定 envelope 演进、unknown-component fail-closed 和未来 CR 耦合。 | component 兼容方案变化回 CP3；把 C3/C4/event 计算纳入则回 CP2。 |
| `DQ-CP2-CR166-004` | security | 是否批准 architecture-major route 与 deny-default：CP2 后只进入 CP3，CP5 前不实现，继续 no-subagent inline？ | 批准 CP3→CP4→CP5→CP6→CP7→CP8；当前只授权本地设计，所有外部/生产数据/runtime/write 操作为 0。 | A. 暂停在 CP2；B. 允许 CP3 后直接实现；C. 启用子 agent。 | 推荐方案保持关键门禁和用户既有 no-subagent 要求；绕过 CP5 会缺少全量设计证据；暂停最保守但阻断价值。 | 决定下一步能否生成 HLD/ADR；不授权代码实现或任何高风险操作。 | 需实现则等待 CP5；需外部数据/runtime 则独立授权；用户解除 no-subagent 前保持 inline。 |

| 字段 | 内容 |
|---|---|
| 推荐决策 | `approve`：接受 DQ-CP2-CR166-001..004 推荐方案，进入 CP3。 |
| 备选方案 | `修改: <决策 ID + 修改点>` 回到 requirement-clarification；或 `reject` 暂停/退回 CR166。 |
| 影响维度 | 用户价值、泄漏/过拟合风险、可审计性、兼容性、维护成本、安全权限和 Stage claim。 |
| 风险与回退 | 产品范围/claim 变化回 CP2；架构细节回 CP3；实现设计回 CP5；外部数据/runtime 立即 BLOCKED。 |
| 用户需决策事项 | `DQ-CP2-CR166-001`、`DQ-CP2-CR166-002`、`DQ-CP2-CR166-003`、`DQ-CP2-CR166-004`。 |

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| Workspace/process routing healthy | 通过 | CP0 result | symlink route healthy。 |
| CR conflict/governance precheck complete | 通过 | CR166、CR-INDEX、CR161 tracking | CR032 与 candidate mapping 已处理。 |
| CP1 use case/requirement/scenario complete | 通过 | CP1 result | 8 docs、10 P0、12 QAC。 |
| CP2 capsule ready | 通过 | CP2 context | compact capsule-first。 |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---:|---|---|---|---|
| 1 | 批准 fixture/static C2 foundation 与 Stage claim ceiling | 通过 | DQ-CP2-CR166-001 | 接受推荐方案。 |
| 2 | 批准 8 P0、2 fixture families 与 12 QAC | 通过 | DQ-CP2-CR166-002 | 接受推荐方案。 |
| 3 | 批准 C3/C4 extension 与 event P1 applicability | 通过 | DQ-CP2-CR166-003 | 接受推荐方案。 |
| 4 | 批准 deny-default route 与 no-subagent inline | 通过 | DQ-CP2-CR166-004 | 接受推荐方案；继续 inline，不拉起子 agent。 |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| 产品基线可作为 CP3 输入 | 通过 | 本 checkpoint | 允许进入 CP3 HLD/ADR。 |
| 用户明确 approve / 修改 / reject | 通过 | 人工审查结果 | 用户回复 `approve，继续推进到下一个人工门禁`。 |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| 产品基线 | `docs/product/*` CR166 revisions | 通过 | 8/8 已确认。 |
| CP2 result | `process/checks/CP2-CR166-WALK-FORWARD-OOS-EVIDENCE-SCOPE.result.json` | 通过 | 自动 PASS。 |
| CP2 capsule | `process/context/CP2-CR166-WALK-FORWARD-OOS-EVIDENCE-CONTEXT.yaml` | 通过 | ready。 |

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-07-13T10:46:00+08:00
- 修改意见：接受 `DQ-CP2-CR166-001..004` 推荐方案，并继续推进到 CP3 人工门禁；保持 no-subagent inline。
- 风险接受项：无；本 gate 不接受 external/runtime/data/write 风险。
