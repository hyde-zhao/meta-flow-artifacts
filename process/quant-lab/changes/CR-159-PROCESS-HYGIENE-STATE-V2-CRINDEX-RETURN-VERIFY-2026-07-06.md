---
cr_id: "CR-159"
cr_type: "process"
cr_kind: "requirement-change"
title: "Process Hygiene: STATE v2 Slim, CR-INDEX Legacy (scoped), Return/Verify Field Validation"
lifecycle_status: "closed"
readiness_status: "ready_with_risk"
gate_status: "closed"
status: "closed"
gate_profile: "compact"
conflict_keys:
  - "state_v2_slim"
  - "crindex_legacy_hygiene"
  - "return_verify_field_validation"
impact_surface:
  - "process-state-v2"
  - "cr-index-legacy"
  - "story-return-verify-tooling"
product_baseline_refresh_required: false
product_baseline_refresh_status: "not_required_process_hygiene_only"
required_phase: "requirement-clarification"
required_agent: "host-orchestrator"
required_gate: "CP2"
block_story_decomposition_until: "N/A - no story decomposition planned"
affected_product_docs: []
affected_use_cases: []
authz_policy_refs:
  - "NO_CREDENTIAL_READ"
  - "NO_RUNTIME"
  - "NO_PRODUCTION_WRITE"
  - "NO_TRADING"
  - "NO_REAL_LAKE_WRITE"
  - "NO_NAS_SYNC_OR_WRITE"
  - "NO_PROVIDER_FETCH"
  - "NO_BROKER_WRITE"
  - "NO_EXTERNAL_FRAMEWORK_RUN"
  - "NO_CATALOG_POINTER_WRITE"
risk_refs:
  - "R-CR159-STATE-SLIM-ARCHIVE-READABILITY"
  - "R-CR159-STATE-RESIDUAL-WARN"
  - "R-CR159-CRINDEX-MUTATION-MISREAD"
  - "R-CR159-AUTHZ-L2-DATA-FIX-VS-CAPABILITY-CHANGE"
  - "R-CR159-CP8-SCOPED-NOT-GLOBAL"
  - "R-CR159-CP6-CP7-CONDITIONAL"
  - "R-CR159-ROADMAP-REWRITE-SCOPE"
created_at: "2026-07-06T22:05:54+08:00"
created_by: "host-orchestrator"
source: "user"
workflow_mode_before: "standard"
workflow_mode_after_change: "standard"
impact_level: "low"
rollback_to: "delivered / no active formal CR baseline after CR158 closure"
approval_result: "approved"
closed_at: "2026-07-07T00:30:00+08:00"
release_decision: "READY_WITH_RISK"
cr_index_path: "process/changes/CR-INDEX.json"
summary_ref: "process/changes/summaries/CR-159.summary.json"
cp0_result_ref: "process/checks/CP0-CR159-REQUEST-INTAKE.result.json"
cp0_context_ref: "process/context/CP0-CR159.context.json"
cp1_result_ref: "process/checks/CP1-CR159-PROCESS-HYGIENE-STATE-V2-CRINDEX-RETURN-VERIFY-SCOPE-COMPLETENESS.result.json"
cp2_result_ref: "process/checks/CP2-CR159-PROCESS-HYGIENE-STATE-V2-CRINDEX-RETURN-VERIFY-SCOPE.result.json"
cp2_context_ref: "process/context/CP2-CR159-PROCESS-HYGIENE-STATE-V2-CRINDEX-RETURN-VERIFY-SCOPE-CONTEXT.yaml"
cp2_checkpoint_ref: "process/checkpoints/CP2-CR159-PROCESS-HYGIENE-STATE-V2-CRINDEX-RETURN-VERIFY-SCOPE.md"
cp3_result_ref: ""
cp4_result_ref: ""
cp5_result_ref: ""
cp6_result_ref: ""
cp7_result_ref: ""
cp8_result_ref: ""
release_context_ref: ""
closure_summary_ref: ""
routing_design_ref: ""
revision_notes: "v0.3 (2026-07-06): per 2nd CP2 review — subitem A no longer deletes current_agent/last_action/cr_tracking_ref/cr_ledger_ref (state-router/SKILL.md:104,114,290 + STATE-TEMPLATE.md contract dependency); keeps 4 residual WARN documented + FU-CR159-002 revisit; only deletes stale phase + archives source_refs/workflow_health. Added CP8 scoped verifier (executable). Fixed Checkpoint Index 6→7. v0.2 (2026-07-06): split subitem B into B1 + B2 (deferred); CP8 scoped; CP6/CP7 conditional; mapping table; cr-tracking split."
---

# CR-159 Process Hygiene: STATE v2 Slim, CR-INDEX Legacy (scoped), Return/Verify Field Validation

## 变更描述

用户请求：“启动 process hygiene CR（STATE v2 slim + CR-INDEX legacy 治理 + return/verify 字段校验探测），从 CP0 受理；CP2 Decision Brief 须含 STATE unknown field 处置、CR-151/152/153/154 authz L2 语义判定、CR-158 status 修正三项决策，可完成后需要回写路标文件。”

本 CR 来源是 `docs/design/QUANT-RESEARCH-PRODUCTION-ROADMAP.md` §7 近期建议第 3 条“小型治理候选”。路标 §7 第 1、2 条（FU-CR154-001 packaging、FU-CR152-001 test taxonomy）已由 CR-156 于 2026-07-05 关闭，路标 v1.2 未回写此事实；本 CR 收尾时一并回写路标 v1.3。

CP0 受理前已完成上下文恢复与冲突预检：`process` 路由健康（`routing_mode=symlink`），`STATE.current.json.active_change=null`，`CR-INDEX.json.active_crs=None`，`meta-flow check cr-tracking` 报告 active formal CRs none / blocked formal CRs none（但命令整体退出码 1，因历史 invalid lifecycle_status 和 follow-up missing warnings——这些属本 CR 子项 B 范围）。CR-159 编号可用。

CP2 评审（用户 findings）后修订：真实 `cr check` 失败面为 **187 ERROR**（63 cr_ledger invalid status + 59 cr_index invalid status + 57 missing summary_ref + 4 invalid cr_type + 4 authz L2），远超初版子项 B 范围。invalid status 涉及 44 种历史自定义值，全局归一化需逐 CR 语义判断，不适合本 hygiene CR。故 CP8 改为 **scoped check**，子项 B 拆 B1（本 CR 数据修正）/ B2（deferred FU-CR159-001）。

## 目标影响摘要

| 字段 | 内容 |
|---|---|
| 目标引用 | `CR159-PROCESS-HYGIENE` |
| 整体目标 | 收束三项本地 process hygiene：A) STATE v2 slim（source_refs/phase resolved + 4 residual WARN documented）；B1) CR-INDEX scoped 数据修正（CR-158 status + required_evidence 留痕）；C) return/verify 字段校验探测。B2 全局 legacy + authz L2 infer 误判 deferred 到 FU-CR159-001。CP8 后回写路标 v1.3。 |
| 用户目标影响 | 降低 STATE / CR-INDEX / story tooling 噪声（scoped，非全局 0 ERROR）；并回写路标 v1.3。 |
| approve 后会发生什么 | 进入实施：STATE slim apply + 删 stale phase（保留 4 运行态字段，residual WARN documented）、CR-INDEX B1 数据修正（含备份 + mapping table）、story 工具探测；条件 CP6/CP7 或 N/A；随后 CP8 scoped closure + 路标 v1.3。 |
| reject / 不确认会阻塞什么 | 不确认会保留现有 STATE WARN、CR-INDEX ERROR 和 cr-tracking WARNING；三项治理候选保持 open。 |
| 决策负担 | `low`：范围已由用户指定，主要确认 scoped 边界、字段处置、不授权边界和路标回写范围。 |

## 三子项范围

### 子项 A：STATE v2 slimming（scoped：source_refs/phase resolved + 4 residual WARN documented）

**缺口（`meta-flow state check --mode audit` 实证，9 WARN）**

| 缺口 | 计数 | 说明 |
|---|---|---|
| unknown field | 6 | `cr_ledger_ref` / `cr_tracking_ref` / `current_agent` / `last_action` / `phase` / `workflow_health` 不在 v2 schema 识别集 |
| source_refs 超 budget | 1 | 4.1 KB > 4.0 KB；37 项 > 24 项上限（CR158 全链 CP1-CP8 + evidence + release refs） |

**工具核验**：`meta-flow state slim --dry-run` 已确认 `source_size: 7.9 KB → projected_size: 5.2 KB`，可归档 `source_refs` + `workflow_health`。但 slim **仅归档 `source_refs` + `workflow_health`，不会自动清除剩余 5 个 unknown field**（`cr_ledger_ref` / `cr_tracking_ref` / `current_agent` / `last_action` / `phase`）。v0.3 决定保留 4 个运行态契约字段（见下），仅删除 stale `phase`，接受 4 residual WARN。

**处置方案（v0.3 修订：保留运行态契约字段）**：

核验发现 `state-router/SKILL.md:104,114,290` 和 `STATE-TEMPLATE.md:7,11,318,319` 依赖 `current_agent` / `last_action` / `cr_tracking_ref` / `cr_ledger_ref`——直接删除会被 state-router 下一轮补回或破坏运行态契约。故这 4 字段保留，接受 residual WARN。

| 字段 | 处置 | 理由 |
|---|---|---|
| `source_refs` | slim 归档到 archive | 超 budget；全链 ref 可从 archive 审计回溯 |
| `workflow_health` | slim 归档（已有 `workflow_health_ref`） | v2 不识别；archive 已有指针 |
| `phase` | 删除 | 与 `current_phase` 冲突且 stale（`phase=story-execution` vs `current_phase`）；state-router 不读 `phase` |
| `current_agent` | **保留**（接受 residual WARN） | `state-router/SKILL.md:104,225,290` 读取并更新；运行态契约依赖 |
| `last_action` | **保留**（接受 residual WARN） | `state-router/SKILL.md:290` 更新；运行态契约依赖 |
| `cr_tracking_ref` | **保留**（接受 residual WARN） | `state-router/SKILL.md:104,114` 读取并要求缺失时写入 `process/changes/CR-INDEX.json` |
| `cr_ledger_ref` | **保留**（接受 residual WARN） | `state-router/SKILL.md:104,114` 读取并要求缺失时写入 `process/state/CR-LEDGER.ndjson` |

**目标（量化退出条件，v0.3 修订）**：
- `source_refs` 项数 ≤ 24、体积 ≤ 4.0 KB（slim 归档解决 budget）
- `workflow_health` 归档（`workflow_health_ref` 指向 archive）
- `phase` 删除
- **4 residual WARN documented**（`current_agent` / `last_action` / `cr_tracking_ref` / `cr_ledger_ref`，因 state-router 契约保留，非数据质量问题）
- 不承诺 `state check --mode audit` 全局 0 WARN；residual WARN 重访条件 = `FU-CR159-002` candidate（STATE v2 schema/check allowlist 扩展，使这 4 字段合法化）
- 实施前备份 `STATE.current.json` 到 `process/backups/`；slim + 删 phase 后运行 `meta-flow state check --mode audit` + `meta-flow doctor context` 确认 archive 可读、无功能破坏

### 子项 B：CR-INDEX legacy 治理（B1 本 CR + B2 deferred）

**真实失败面（`meta-flow cr check` 实证，187 ERROR）**

| 缺口类别 | 计数 | 样本 |
|---|---|---|
| `cr_ledger` invalid status | 63 | `CR-138: invalid status closed-current-delivery` |
| `cr_index` invalid status | 59 | `CR-010: invalid status verified-limited-window-pass-pending-close-decision` |
| missing `summary_ref` | 57 | CR-001~CR-154 |
| invalid `cr_type` | 4 | `CR-109: invalid cr_type process-governance` |
| authz L2 `required_evidence_forbidden_by_authz` | 4 | CR-151/152/153/154 |

invalid status 涉及 **44 种不同值**（57 个 `closed-current-delivery` + 大量历史自定义值如 `verified-limited-window-pass-pending-close-decision`、`active-cp5-pending-design` 等），多数是早期 CR 的自定义状态语义。

**B1（本 CR 实施，明确数据修正）**

| 修正项 | 计数 | 性质 |
|---|---|---|
| CR-151/152/153/154 `required_evidence`: `real_lake_validation` → `static_fixture_only_validation` | 4 | evidence 标签修正，匹配实际 local/static/fixture-only 验证模式（ROADMAP §2.1） |
| CR-158 `status`/`lifecycle_status`: `closed-current-delivery` → `closed`（implementation correction：CP2 推荐的 `closed-current-delivery-ready-with-risk` 实施验证发现不是合法 status enum，cr check 报 invalid；实际采用 `closed`，READY_WITH_RISK 语义保留在 `release_decision`/`summary`/`risk acceptance`） | 1 | CP2 DQ 推荐值修正；READY_WITH_RISK 语义不编码进 `status` 字段 |
| CR-159 自身 CR-INDEX item 合规 | 1 | 注册时已用合法 enum |

**B2（deferred candidate `FU-CR159-001`，不在本 CR 修复）**

| 缺口类别 | 计数 | deferred 理由 |
|---|---|---|
| `cr_ledger` invalid status | 63 | 历史事件 status 语义映射需逐事件判断 |
| `cr_index` invalid status（含 56 个 `closed-current-delivery` + 其他 44 种值） | 58 | 44 种自定义 status → 合法 enum 映射需逐 CR 语义判断 |
| missing `summary_ref` | 57 | 57 个历史 CR summary 生成或 N/A 标注，工作量大且非本 CR 目标 |
| invalid `cr_type` | 4 | `process-governance` / `requirement-change` 历史值映射 |

**CP8 验收（scoped，非全局 0 ERROR）**：
- B1 scoped 修正后：`cr check` 中 CR-158 index item + CR-159 index item 合规（authz L2 因 formal CR 正文 "real lake" 触发 `infer_required_evidence_from_text` 误判，cr check 用 `record_from_cr_file` 不读 CR-INDEX item `required_evidence`，deferred FU-CR159-001）
- B2 全部转入 `FU-CR159-001` follow-up candidate（`process/changes/CR-159-FOLLOW-UP-TRACKING-2026-07-06.md`），状态 `candidate`，不占执行锁
- 全局 `cr check` 仍会有 B2 legacy ERROR，本 CR **不承诺全局 0 ERROR**

**CP8 scoped verifier（可执行规则）**：

```bash
# B1 清零：CR-158 index item + CR-159 index item 相关 ERROR = 0（authz L2 deferred to FU-CR159-001）
test "$(meta-flow cr check --project-root . 2>&1 | grep -cE 'CR index item CR-158: invalid status|CR index item CR-159:')" -eq 0 && echo "B1 zero: PASS" || echo "B1 zero: FAIL"

# B2 登记验证：FU-CR159-001 台账存在且含 candidate 记录
grep -q 'FU-CR159-001' process/changes/CR-159-FOLLOW-UP-TRACKING-2026-07-06.md && echo "B2 registered: PASS" || echo "B2 registered: FAIL"

# 子项 A 验证：source_refs budget WARN + phase WARN = 0（4 residual WARN 保留，documented）
test "$(meta-flow state check --mode audit --project-root . 2>&1 | grep -cE 'source_refs exceeds|unknown field: phase')" -eq 0 && echo "A budget: PASS" || echo "A budget: FAIL"
```

scoped verifier 三项均输出 `PASS` 即 B1 清零 + B2 登记 + 子项 A budget resolved；4 residual WARN（`current_agent`/`last_action`/`cr_tracking_ref`/`cr_ledger_ref`）因 state-router 契约保留，CP8 显式记录重访条件 `FU-CR159-002`。

### 子项 C：return/verify packet 字段非空校验探测

**工具现状**：`meta-flow story return-check`（`--packet` + `--return`）、`meta-flow story evidence-check`（`--index`）、`meta-flow story verify-packet` 三个工具已存在，help 仅列参数、未说明校验深度。

**目标**：实施阶段探测三个工具是否已校验 return packet / evidence index 关键字段非空（`story_id` / `stage` / `evidence_path` / `design_delta_ref` 等）。

**CP6/CP7 条件路由（非 blanket WAIVED）**

| 探测结果 | CP6 | CP7 | CP8 |
|---|---|---|---|
| 工具已具备字段非空校验，无代码变更 | N/A（with reason） | 由 CP8 统一验证 | closure |
| 需增强 `meta-flow story` 工具校验逻辑 | **轻量 CP6 implementation evidence**（实现对象、契约映射、回归计划） | **轻量 CP7 verification evidence**（回归 CR156/CR157/CR158 return packet + evidence index，0 新失败） | closure |

实施阶段先做子项 C 探测；若需工具实现改动，必须生成轻量 CP6/CP7 evidence 再进 CP8。

## CR-INDEX / STATE 修正 mapping table

本 CR 所有数据修正项必须留机器可审计 mapping，证明“数据修正，非语义变更”。完整 mapping 见 CP2 context capsule `## correction_mapping`；此处为摘要。

### B1 authz L2 + CR-158 status 修正

| CR id | field | old_value | new_value | reason |
|---|---|---|---|---|
| CR-151 | required_evidence | [..., `real_lake_validation`, ...] | [..., `static_fixture_only_validation`, ...] | 实际验证模式 local/static/fixture-only（ROADMAP §2.1） |
| CR-152 | required_evidence | [..., `real_lake_validation`] | [..., `static_fixture_only_validation`] | 同上 |
| CR-153 | required_evidence | [..., `real_lake_validation`, ...] | [..., `static_fixture_only_validation`, ...] | 同上 |
| CR-154 | required_evidence | [..., `real_lake_validation`, ...] | [..., `static_fixture_only_validation`, ...] | 同上 |
| CR-158 | status / lifecycle_status | `closed-current-delivery` | `closed`（implementation correction：CP2 推荐值 `closed-current-delivery-ready-with-risk` 非合法 status enum，实际用 `closed`） | 对齐合法 enum；READY_WITH_RISK 语义在 release_decision/summary |

所有修正前备份 `CR-INDEX.json` 到 `process/backups/CR-INDEX.json.bak-20260706`；rollback = 恢复 backup。

### 子项 A STATE field 处置（v0.3：保留运行态契约字段）

| field | old_value | new_value | reason |
|---|---|---|---|
| source_refs | 37 项 / 4.1 KB | 归档到 archive | slim 超 budget |
| workflow_health | inline object | 归档（`workflow_health_ref` 指向 archive） | v2 不识别 |
| phase | `story-execution` | 删除 | 与 `current_phase` 冲突且 stale；state-router 不读 |
| current_agent | `host-orchestrator` | **保留**（residual WARN） | state-router/SKILL.md:104,225,290 契约依赖 |
| last_action | (string) | **保留**（residual WARN） | state-router/SKILL.md:290 契约依赖 |
| cr_tracking_ref | `process/changes/CR-INDEX.json` | **保留**（residual WARN） | state-router/SKILL.md:104,114 契约依赖 |
| cr_ledger_ref | `process/state/CR-LEDGER.ndjson` | **保留**（residual WARN） | state-router/SKILL.md:104,114 契约依赖 |

备份 `STATE.current.json` 到 `process/backups/STATE.current.json.bak-20260706`；rollback = 恢复 backup。

### B2 deferred（FU-CR159-001，仅分类计数）

| 缺口类别 | 计数 | deferred 到 |
|---|---|---|
| cr_ledger invalid status | 63 | FU-CR159-001 |
| cr_index invalid status | 58 | FU-CR159-001 |
| missing summary_ref | 57 | FU-CR159-001 |
| invalid cr_type | 4 | FU-CR159-001 |

## 拆分理由

| 问题 | 结论 |
|---|---|
| 为什么不合并到 parent / active CR | CR-156 已关闭；当前无 active / blocked formal CR，CR-159 是新的 hygiene 收束外壳。 |
| 为什么不是 Story / task / follow-up | 用户显式启动 CR-159，且目标是三子项统一治理与路标回写，需要一个独立审计闭环。 |
| 触发独立 CR 的边界 | 三子项共同进入 CR-159；不包含 CR155 admission remediation、数据湖 deferred candidates、真实运行时或 release execution。 |
| 为什么 B2 deferred 而非本 CR 修 | 187 ERROR 中 182 项涉及历史 CR（CR-001~CR-154）的自定义 status/cr_type/summary 语义，逐项映射需人工判断，超出 hygiene CR 范围；B1（5 项）是明确数据修正，可在本 CR 完成。 |

## CP2 Follow-up 来源

| 字段 | 内容 |
|---|---|
| 父级来源 | 路标 `QUANT-RESEARCH-PRODUCTION-ROADMAP.md` §7 第 3 条 |
| 来源决策 ID | STATE v2 slim、CR-INDEX legacy (scoped)、return/verify 字段校验 |
| follow-up 类型 | local process / tooling hygiene |
| 风险等级 | low |
| owner | host-orchestrator |
| 重访条件 | CP2 compact scope baseline 发现需扩大到代码实现、真实 release、Git remote 或 runtime |
| 验收标准 | 子项 A `state check` 0 WARN；B1 `cr check` scoped 0 ERROR（authz L2 + CR-158 + CR-159）；不授权边界清晰；CR tracking active/blocked 冲突检查 OK |
| 关闭条件 | CR159 完成 CP8 或用户取消 / supersede |

## CR 类型与门禁策略

| 字段 | 内容 |
|---|---|
| CR 类型 | `process` |
| Legacy CR kind | `requirement-change` |
| 生命周期状态 | `active` |
| 就绪状态 | `not_ready` |
| 门禁状态 | `cp2_pending` |
| 门禁模板 | `compact` |

## Checkpoint Index

| CP | 状态 | 机器结果 ref | 人工门禁 ref | Context ref | 摘要 |
|---|---|---|---|---|---|
| CP0 | pass | `process/checks/CP0-CR159-REQUEST-INTAKE.result.json` | N/A | `process/context/CP0-CR159.context.json` | 目标已捕获：三子项 hygiene + 路标回写。 |
| CP1 | pass | `process/checks/CP1-CR159-...-SCOPE-COMPLETENESS.result.json` | N/A | `process/context/CP2-CR159-...-SCOPE-CONTEXT.yaml` | compact scope completeness 通过。 |
| CP2 | pending user approval | `process/checks/CP2-CR159-...-SCOPE.result.json` | `process/checkpoints/CP2-CR159-...-SCOPE.md` | `process/context/CP2-CR159-...-SCOPE-CONTEXT.yaml` | 7 项决策待用户确认（含 STATE field、authz L2、CR-158 status、CP8 scoped、CP6/7 条件路由、security）。 |
| CP3 | N/A (pending CP2 approval) | — | N/A | — | process hygiene CR 无 HLD / ADR / architecture artifacts required。 |
| CP4 | N/A (pending CP2 approval) | — | N/A | — | 无 Story decomposition / DAG / parallel-safety surface。 |
| CP5 | N/A (pending CP2 approval) | — | N/A | — | 无 LLD batch 或 Story design evidence required。 |
| CP6 | 条件路由 (pending CP2 approval) | — | N/A | — | 无代码改动则 N/A；子项 C 需工具实现改动则生成轻量 CP6 implementation evidence。 |
| CP7 | 条件路由 (pending CP2 approval) | — | N/A | — | 无代码改动则由 CP8 统一验证；子项 C 需工具改动则生成轻量 CP7 verification evidence + 回归。 |
| CP8 | pending | — | — | — | 发布就绪（scoped）+ 路标 v1.3 回写。 |

## 结构化权限策略

```yaml
authorization_policy:
  nas:
    access: false
    list: false
    read: false
    write: false
    publish: false
    delete: false
  credentials:
    env_read: false
    secret_read: false
    account_read: false
  runtime:
    qmt: false
    miniqmt: false
    xtquant: false
    gateway: false
  trading:
    submit: false
    cancel: false
    simulation: false
    live: false
```

## 文档处理决策

| 受影响文档 | 处理方式 | 旧基线保留方式 | 修订记录位置 | 批准状态 |
|---|---|---|---|---|
| `docs/product/USE-CASES.md` | 不变 | 本 CR 不改变用户场景或产品基线 | N/A | pending |
| `docs/product/REQUIREMENTS.md` | 不变 | 本 CR 不新增 / 重定义 REQ | N/A | pending |
| `docs/product/SCENARIOS.yaml` | 不变 | 本 CR 不扩展产品场景 | N/A | pending |
| `docs/product/TEST-MATRIX.md` | 不变 | 本 CR 不重定义产品验收矩阵 | N/A | pending |
| `docs/design/QUANT-RESEARCH-PRODUCTION-ROADMAP.md` | CP8 后回写 v1.3 | 追加修订记录行，保留旧基线 | §修订记录 + §2.1 + §7 | pending CP2 |

路标回写内容（CP8 阶段执行）：补记 CR-156 已关闭 FU-CR154-001/FU-CR152-001、CR-157/CR-158 闭环、CR-159 hygiene 收束；修正 §7 第 1、2 条措辞从“建议”改为“已完成”。

## 五维度影响分析

| 维度 | 评估问题 | 受影响对象 | 结论 | 处理动作 |
|---|---|---|---|---|
| 需求层 | 是否新增、删除或重定义 REQ-* | 无 | no product-baseline impact | 不修改 USE-CASES / REQUIREMENTS / SCENARIOS / MVP scope。 |
| 场景层 | 是否改变测试矩阵覆盖范围 | 无 | no scenario expansion | 只治理 process/tooling，不新增产品测试场景。 |
| 计划层 | 是否改变 Phase、Wave、Story / 任务依赖 | CR159 compact hygiene | low impact | 无 Story 拆解；三子项可顺序实施并统一 closure。 |
| 安全层 | 是否引入新的高风险动作或权限要求 | 默认全部不授权 | controlled | 禁止 runtime、真实数据、凭据、NAS/provider、broker、trading、Git remote、publish。 |
| 交付层 | 是否需要重新生成交付物或回归子集 | STATE / CR-INDEX / story tooling | local only | 复用 `meta-flow state slim` / `cr check` / `story return-check` 既有工具。 |

## 回退决策

- 影响范围：本地 process state、CR-INDEX 数据、story tooling 校验逻辑和 CR tracking。
- 回退到阶段：`delivered / no active formal CR baseline after CR158 closure`。
- 需要重新确认的对象：如 CP2 发现需要扩大到代码实现、真实 release、Git remote 或 runtime，则必须新建授权决策或回退为 candidate。
- STATE slim 回退：`meta-flow state slim --apply` 前 archive_ref 已记录归档字段；如需回退，可从 `process/archive/state/<ts>/archived-fields.json` 恢复，或恢复 `process/backups/STATE.current.json.bak-20260706`。
- CR-INDEX 修正回退：所有数据修正前备份 `CR-INDEX.json` 到 `process/backups/`；修正逐项记录原值 → 新值（见 mapping table）；rollback = 恢复 backup。
- STATE 字段删除回退：恢复 `process/backups/STATE.current.json.bak-20260706`。

## 产品基线重整门禁

- 是否需要产品基线重整：false。
- 必须回到阶段：`requirement-clarification`（compact CP1/CP2 scope baseline only）。
- 责任 Agent：`host-orchestrator`。
- 必须通过门禁：`CP1` / `CP2`。
- Story / LLD / 实现阻断条件：N/A，本 CR 不计划 Story / LLD。
- 受影响产品文档：无（路标回写为过程文档，非产品基线）。
- 受影响 use case：无。

## fast-lane 判定

| 条件 | 是否命中 | 说明 |
|---|---|---|
| 仅低风险轻量实现 / 文档 / 规则修改 | yes | 本地 process/tooling hygiene 收束。 |
| 修改架构、权限、安全边界或平台安装路径 | no | 不扩大权限，不修改平台路径。 |
| 修改外部接口契约、文件所有权或多 Story 依赖 | no | 不引入 Story / LLD。 |
| 需要 HLD / LLD 才能解释影响 | no | 既有工具核验证据已说明目标。 |
| 是否保持 fast-lane | no | 工作流仍记录为 standard；本 CR 采用 compact gate profile。 |

## LLD 设计批次门禁

- 是否需要 LLD 设计批次：false。
- batch_id：`CR-159-LLD-BATCH`。
- 批次范围来源：N/A。
- 批次内 Story：N/A。

## 风险与开放项

| ID | 状态 | 问题 | owner | 下一步 |
|---|---|---|---|---|
| R-CR159-STATE-SLIM-ARCHIVE-READABILITY | OPEN | slim 归档 source_refs / workflow_health 后，全链 CP ref 移入 archive，需确认仍可审计回溯。 | host-orchestrator | 实施后 `meta-flow state check` + `doctor context` 确认 archive_ref 可读。 |
| R-CR159-STATE-RESIDUAL-WARN | OPEN | state-router/SKILL.md:104,114,290 + STATE-TEMPLATE.md:7,11,318,319 依赖 current_agent/last_action/cr_tracking_ref/cr_ledger_ref；保留这 4 字段导致 4 residual WARN，但不删除以避免破坏运行态契约（v0.2 删除方案已撤销）。 | host-orchestrator | 接受 4 residual WARN documented；重访条件 = FU-CR159-002 candidate（STATE v2 schema/check allowlist 扩展）。 |
| R-CR159-CRINDEX-MUTATION-MISREAD | OPEN | CR-INDEX 数据修正（status / required_evidence）可能被误读为重开 closed CR 或改变能力声明。 | host-orchestrator | CP2 / CP8 显式声明为数据修正，保留原值备份 + mapping table；不重开 CR-151/152/153/154/158。 |
| R-CR159-AUTHZ-L2-DATA-FIX-VS-CAPABILITY-CHANGE | OPEN | 修正 CR-151/152/153/154 的 `required_evidence` 是 evidence 标签修正还是能力变更需用户确认。 | host-orchestrator | CP2 DQ-CP2-CR159-AUTHZ-L2-SEMANTICS 决策。 |
| R-CR159-CP8-SCOPED-NOT-GLOBAL | OPEN | CP8 为 scoped check，全局 `cr check` 仍有 B2 legacy ERROR；可能被误读为本 CR 未完成。 | host-orchestrator | CP8 显式记录 scoped 范围 + B2 deferred candidate；不承诺全局 0 ERROR。 |
| R-CR159-CP6-CP7-CONDITIONAL | OPEN | 子项 C 探测结果未知，CP6/CP7 路由待实施阶段判定。 | host-orchestrator | 实施阶段先做子项 C 探测；若需工具改动，生成轻量 CP6/CP7 evidence。 |
| R-CR159-ROADMAP-REWRITE-SCOPE | OPEN | 路标 v1.3 回写可能被误读为范围变更。 | host-orchestrator | CP8 仅追加修订记录 + 修正 §7 措辞，不改变阶段目标或退出条件。 |
