---
checkpoint_id: "CP2-CR046-REQUIREMENTS-BASELINE"
checkpoint_name: "CR-046 Requirements, Scenarios and Scope Baseline"
type: "auto_then_manual"
status: "changes_requested"
owner: "host-orchestrator"
created_at: "2026-07-11T14:36:00Z"
reviewed_by: "user"
reviewed_at: "2026-07-12T01:19:18Z"
auto_check_result: "process/checks/CP2-CR046-REQUIREMENTS-BASELINE.result.json"
target:
  phase: "requirement-clarification"
  story_id: ""
  artifacts: ["docs/product/USE-CASES.md", "docs/product/REQUIREMENTS.md", "docs/product/SCENARIOS.yaml", "docs/product/TEST-MATRIX.md", "docs/product/STORY-MAP.md", "docs/product/MVP-SCOPE.md", "docs/product/RELEASE-SLICES.md", "docs/product/BACKLOG.md"]
---

# CP2 CR-046 Requirements, Scenarios and Scope Baseline

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP1-CR046-USE-CASE-COMPLETENESS.result.json` | PASS | 0 | UC 12；受影响 persona 和异常边界完整 |
| `process/checks/CP2-CR046-REQUIREMENTS-BASELINE.result.json` | PASS | 0 | REQ 54、TC 31、ST 20；traceability PASS |
| `process/checks/CP2-CR046-DISCUSSION-CHECKPOINT.json` | completed | 0 | SGA-05..08 resolved；SGQ-CR046-001 confirmed |

## Decision Brief

### 审批者摘要

| 字段 | 内容 |
|---|---|
| 本次确认服务的整体目标 | 确认 CR-046 的产品/场景/MVP 基线，使 Meta Flow 能证明可信时序、平台调度来源、checker 重放和成本度量 |
| 推荐动作 | `approve`：批准统一治理范围和降级语义，进入 CP3 架构设计 |
| approve 后会发生什么 | 自动切换到 solution-design，真实调度 meta-se-critical 设计公共 evidence contract、checker/state/ledger 边界和 7 Story 计划；CP3 仍需单独人工批准 |
| approve 不授权什么 | credentials、runtime、production write、publish、交易、commit/push、quant-lab lineage 业务代码修改、真实 CR-163 pilot 执行 |
| 不确认会阻塞什么 | 阻塞 CP3、Story 拆解、LLD 和实现；当前 Meta Flow 仍只能部分证明语义级过程合规 |

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP2-CR046-REQUIREMENT-CONTEXT.yaml` |
| capsule 状态 | ready |
| read_profile | compact |
| 默认读取策略 | 先读 capsule；仅在缺失、冲突、字段不足、人工审计或深度评审时读取完整产品文档 |
| 全文档读取扩展 | 2 次：CR full document（field conflict）和 artifact product baseline（incremental baseline update） |
| 缺失 / waived 理由 | 无 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `process/state/STATE.current.json` / `process/state/GATE-LEDGER.ndjson` | scanned | 1 | 1 | active CR046；CP2 pending |
| 委托 Agent 交还摘要 | `process/handoffs/CR046-CP1-CP2-META-PM-RETURN-SUMMARY.md` | scanned | 1 | 1 | CP2-DQ-01 |
| 自动预检结果 | CP1/CP2 result JSON | scanned | 0 | 0 | 两项 PASS，blocker 0 |
| discussion log / checkpoint | `process/discussions/CP2-CR046-SCENARIO-DISCUSSION-LOG.md` / discussion checkpoint | scanned | 4 | 0 | SGA-05..08 resolved by originating request |
| 下游正式产物 | `docs/product/*` artifact truth source | scanned | 2 | 0 | DEF-EI-001..002 deferred；非本门阻断 |
| 用户显式选择题 | 当前 CR-046 启动指令 | scanned | 1 | 1 | 批准本产品/场景/范围基线 |

### 决策分层

| 分类 | 数量 | 处理方式 |
|---|---:|---|
| 必须用户决策 | 1 | CP2-DQ-01 |
| 高风险策略确认 | 0 | 无 runtime/credential/publish 授权请求 |
| agent 默认处理 | 4 | 缺 receipt/telemetry 的 unavailable、append-only correction、双口径 replay |
| 仅审计记录 | 2 | DEF-EI-001..002 deferred |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| CP2-DQ-01 | scope | 是否批准 CR-046 产品/场景/MVP 基线，并接受 receipt/telemetry 可能 unavailable、CR-163 append-only pilot 和 lineage 业务源码 immutable 边界？ | approve：批准完整基线并进入 CP3 | A. `修改: CP2-DQ-01 <具体范围>` 后重提；B. reject，保留现有 schema-level 治理 | 推荐方案提供完整机器审计链，但增加公共 schema/checker/fixture 工作；修改可减范围但可能切断端到端证明；reject 无新增变更风险但保留已知缺口 | 主要风险是 legacy compatibility、平台 receipt/telemetry 不可用和历史证据不可变 | 修改时回 requirement-clarification；checker compatibility 或 immutable-prefix 失败时回 CP5/CP6；未单独授权时不执行 pilot |

| 字段 | 内容 |
|---|---|
| 推荐决策 | `approve` CP2-DQ-01，冻结产品/场景/MVP 基线并进入 CP3 |
| 备选方案 | 修改具体范围后重提；或 reject 并保留当前 schema-level 治理 |
| 影响维度 | 用户价值、实现复杂度、可验证性、维护成本、平台兼容、安全/权限、交付影响 |
| 优劣分析 | 完整范围形成端到端机器审计链；缩减范围降低工作量但会削弱证明闭环；reject 保持当前实现但保留 PARTIAL 缺口 |
| 风险与回退 | legacy compatibility、unavailable receipt/telemetry、历史不可变；修改回 requirement-clarification，后续契约失败回 CP5/CP6 |
| 用户需决策事项 | `CP2-DQ-01`：approve、修改或 reject 本轮产品/场景/MVP 基线 |

### CP2 场景与范围追加摘要

- 用户真实意图：Meta Flow 必须证明语义事实，不只验证 schema；不可验证事实必须显式 unavailable。
- 场景覆盖：CR046 新增 UC-EI-001..005、REQ-EI、TC-EI-001..013、ST-EI-001..007。
- Scenario Gray Areas：receipt、telemetry、historical correction、replay identity 均已收敛。
- Deferred Ideas：统一加密 receipt、估算 token 的强制计费/配额。
- Artifact routing：`/home/hyde/workspace/meta-flow-artifacts/process/meta-flow/docs/product` 是单一真相源；源码仓库无第二份 baseline。
- 回退：修改范围则回到 requirement-clarification；reject 则停止 CR046，不进入设计。

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| CP0/route plan 完成 | PASS | `process/checks/CP0-CR046.route-plan.json` | architecture-major 全路径 |
| CP1 自动门通过 | PASS | CP1 result | 无 blocker |
| CP2 自动预检通过 | PASS | CP2 result | 无 blocker |
| 场景讨论证据存在 | PASS | discussion log/checkpoint | SGA 全收敛 |
| CP2 capsule ready | PASS | CP2 context | compact capsule-first |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | 用户/审计 persona 与目标完整 | 待审查 | `docs/product/USE-CASES.md` |  |
| 2 | P0/P1 requirements 可检验 | 待审查 | `docs/product/REQUIREMENTS.md` |  |
| 3 | 正向/负向/边界/权限/恢复场景完整 | 待审查 | `docs/product/SCENARIOS.yaml` |  |
| 4 | UC/REQ/TC/ST 追溯闭环 | 待审查 | `docs/product/TEST-MATRIX.md` |  |
| 5 | MVP/out-of-scope/deferred 清晰 | 待审查 | `docs/product/MVP-SCOPE.md` |  |
| 6 | quant-lab pilot 与业务源码边界清晰 | 待审查 | `process/changes/CR-046.md` |  |
| 7 | receipt/telemetry unavailable 语义诚实 | 待审查 | CP2 result / discussion checkpoint |  |
| 8 | 不授权项独立列出 | 待审查 | Decision Brief |  |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| 产品/场景/范围基线可冻结 | 待审查 | CP1/CP2 results |  |
| 待人工决策全部关闭 | 待审查 | CP2-DQ-01 |  |
| 可进入 CP3 solution-design | 待审查 | route plan |  |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| 产品基线 | `docs/product/*`（artifact repo 单一真相源） | 待审查 |  |
| CP1 result | `process/checks/CP1-CR046-USE-CASE-COMPLETENESS.result.json` | PASS |  |
| CP2 precheck | `process/checks/CP2-CR046-REQUIREMENTS-BASELINE.result.json` | PASS |  |
| CP2 capsule | `process/context/CP2-CR046-REQUIREMENT-CONTEXT.yaml` | PASS |  |
| meta-pm return | `process/handoffs/CR046-CP1-CP2-META-PM-RETURN-SUMMARY.md` | PASS |  |

## 人工审查结果

- 结论：`changes_requested`
- 审查人：user
- 审查时间：2026-07-12T01:19:18Z
- 修改意见：补入 compaction 语义保持、post-close correction lifecycle、机器生成 audit report；把 CP1/CP2 null provenance 和当前 dispatch receipt 局限加入 dogfooding/披露。
- 风险接受项：
  - 本轮不接受缩减上述三项范围。
  - 当前历史 CP1/CP2 provenance 与 dispatch receipt 局限可保留为待整改事实，但不得被表述为 fully verified。
