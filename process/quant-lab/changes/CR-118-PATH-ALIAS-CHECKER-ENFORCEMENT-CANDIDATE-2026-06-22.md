---
cr_id: "CR-118"
cr_type: "process"
cr_kind: "requirement-change"
lifecycle_status: "closed"
readiness_status: "ready"
gate_status: "closed"
gate_profile: "standard"
status: "closed-current-delivery"
impact_level: "medium"
workflow_mode_before: "fast-lane"
workflow_mode_after_change: "standard"
fast_lane_upgrade_reason: "Potential source/tests/checker implementation requires explicit staged gates."
rollback_to: "CR117 closed READY with FU-CR117-001 unstarted"
approval_result: "cp8-approved-user-20260622-ready"
created_at: "2026-06-22T18:21:08+08:00"
created_by: "host-orchestrator"
approved_by: "user"
approved_at: "2026-06-22T19:08:32+08:00"
source: "cp8-follow-up"
linked_issue: ""
parent_cr: "CR-117"
source_checkpoint: "process/checkpoints/CP8-CR117-DELIVERY-READINESS.md"
source_decision_id: "DQ-CP8-CR117-02"
follow_up_type: "path-alias-checker-enforcement-candidate"
risk_class: "medium-source-tests-checker-implementation-pending-authorization"
owner: "host-orchestrator"
revisit_condition: "If CP2 is not approved, keep FU-CR117-001 as candidate and do not modify source/tests/checker. If implementation is approved later, require explicit CP5/implementation evidence before source/tests/checker edits."
acceptance_criteria: "CP5 approved and implemented minimal path alias checker enforcement in scripts/check_human_gate_decision_brief.py with fixture tests."
close_condition: "CP8 approved by user at 2026-06-22T19:08:32+08:00; CR118 closed READY. Runtime/NAS/credential/trading/publish remain unauthorized."
cr_index_path: "process/changes/CR-INDEX.yaml"
current_requirement_baseline_path: "process/baseline/CURRENT-REQUIREMENT-BASELINE.yaml"
historical_baseline_status: "active"
reframed_by: []
reframe_summary: ""
---

# CR-118 Path Alias Checker Enforcement Candidate

## 变更描述

用户在 CR117 CP8 后回复：

> 批准，继续推进下一个建议的cr

该回复关闭 CR117，并选择 `FU-CR117-001 Path Alias Checker Enforcement Candidate` 作为下一个正式 CR。CR118 的当前目标是进入 CP2 决策：确认是否要把 CR117 的路径别名展示约定进一步转为可检查的 checker enforcement 路线。

当前只生成 CP2 范围、授权边界、上下文胶囊、自动预检和人工审查稿；不直接修改源码、tests 或 checker implementation。

## 来源

| 字段 | 内容 |
|---|---|
| 父级 CR | `CR-117` |
| 来源检查点 | `process/checkpoints/CP8-CR117-DELIVERY-READINESS.md` |
| 来源决策 ID | `DQ-CP8-CR117-02` |
| follow-up 类型 | `path-alias-checker-enforcement-candidate` |
| 来源候选 | `FU-CR117-001` |
| 风险等级 | `medium-source-tests-checker-implementation-pending-authorization` |
| owner | `host-orchestrator` |

## 结构化权限策略

```yaml
authorization_policy:
  process_artifacts:
    read: true
    write_audit_evidence: true
    update_cr_index: true
    update_state_summary: true
    create_cp2_gate: true
  source_code:
    discover_owner: true
    modify: false
    refactor: false
    test_change: false
    checker_implementation_change: false
  runtime:
    qmt: false
    miniqmt: false
    xtquant: false
    gateway: false
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
  trading:
    submit: false
    cancel: false
    buy: false
    sell: false
    simulation: false
    live: false
  provider_lake_catalog:
    provider_fetch: false
    lake_write: false
    catalog_publish: false
```

## 五维度影响分析

| 维度 | 评估问题 | 受影响对象 | 结论 | 处理动作 |
|------|----------|-----------|------|---------|
| 需求层 | 是否新增、删除或重定义 REQ-* | 当前需求基线 | false | 不改需求基线；只确认 checker enforcement 候选是否进入实现路线。 |
| 场景层 | 是否改变测试矩阵覆盖范围 | checker / test fixtures | potential | CP2 后若进入实现，必须在 CP5/implementation gate 定义最小测试范围。 |
| 计划层 | 是否改变 Phase、Wave、Story / 任务依赖 | CR118 gate | true | 由 CR117 follow-up 转为 CR118 CP2。 |
| 安全层 | 是否引入新的高风险动作或权限要求 | source / tests / checker | true | 当前不授权实现；后续实现需显式门禁。 |
| 交付层 | 是否需要重新生成交付物或回归子集 | process/checks、process/checkpoints、process/context | true | 生成 CP2 门禁和上下文证据。 |

## CP2 待确认决策

| 决策 ID | 类型 | 推荐方案 | 备选方案 | 回退条件 |
|---|---|---|---|---|
| DQ-CP2-CR118-01 | scope | 接受 CR118 作为 path alias checker enforcement 候选门禁，先确认最小 enforcement 范围，不改源码 / tests / checker | A. 暂停 CR118，保留候选；B. 只保留 CR117 notes，不做机器 enforcement；C. 改为目录 / symlink 迁移 CR | 若用户不需要机器 enforcement，则关闭 / 取消 CR118 并保留 CR117 notes |
| DQ-CP2-CR118-02 | implementation | 采用分阶段路线：CP2 只授权设计、owner discovery 和检查目标定义；源码 / tests / checker implementation 需后续 CP5/implementation gate 单独授权 | A. CP2 直接授权最小源码 / tests / checker 改动；B. 只做设计 notes；C. 改为人工流程约定，不进入 checker | 推荐方案避免把“继续下一个 CR”误读为立即改代码；A 更快但权限边界更宽 |
| DQ-CP2-CR118-03 | runtime_authorization | 不授权 runtime、NAS、凭据、交易写、provider fetch、lake write 或 catalog publish；checker 后续也只允许本地静态 / fixture 验证 | A. 另起 runtime gate；B. 另起 NAS gate；C. 另起 publish gate | 任何 runtime/NAS/publish 需求必须停止 CR118 默认路线并新建独立授权门禁 |

## 当前状态

| 字段 | 内容 |
|---|---|
| 当前门禁 | closed |
| 自动预检 | `process/checks/CP2-CR118-PATH-ALIAS-CHECKER-ENFORCEMENT-PRECHECK.md` |
| 人工审查稿 | `process/checkpoints/CP2-CR118-PATH-ALIAS-CHECKER-ENFORCEMENT-REVIEW.md` |
| 下一步 | CR118 已关闭 READY；当前不新增 follow-up。 |

## CP2 审批结果

| 字段 | 内容 |
|---|---|
| 审批结论 | `approved` |
| 审批人 | user |
| 审批时间 | 2026-06-22T18:54:16+08:00 |
| 接受决策 | `DQ-CP2-CR118-01`、`DQ-CP2-CR118-02`、`DQ-CP2-CR118-03` |
| 授权范围 | 设计、owner discovery、检查目标定义和 CP5 implementation gate 草案 |
| 不授权项 | CP2 不直接授权源码修改、tests 修改、checker implementation change、目录 / symlink 变更、runtime、NAS、凭据、交易写、provider fetch、lake write 或 catalog publish |

## CP5 审批与实现结果

| 字段 | 内容 |
|---|---|
| 审批结论 | `approved` |
| 审批人 | user |
| 审批时间 | 2026-06-22T18:59:13+08:00 |
| 接受决策 | `DQ-CP5-CR118-01`、`DQ-CP5-CR118-02`、`DQ-CP5-CR118-03`、`DQ-CP5-CR118-04` |
| 实现文件 | `scripts/check_human_gate_decision_brief.py` |
| 测试文件 | `tests/test_cr118_human_gate_path_alias_checker.py` |
| 实现摘要 | 人工门禁 checker 现在会拒绝仅以 `docs/design/*` 作为设计证据入口、且缺少对应 `process/docs/design/*` 入口的 checkpoint。 |
| 验证命令 | `PYTHONDONTWRITEBYTECODE=1 PYTEST_ADDOPTS='-p no:cacheprovider' uv run --python 3.11 pytest -q tests/test_cr118_human_gate_path_alias_checker.py`；`uv run --python 3.11 python scripts/check_human_gate_decision_brief.py --checkpoint-file process/checkpoints/CP5-CR118-PATH-ALIAS-CHECKER-ENFORCEMENT-IMPLEMENTATION-REVIEW.md` |
| 不授权项 | 目录 / symlink 变更、runtime、NAS、凭据、交易写、provider fetch、lake write、catalog publish |

## CP8 审批结果

| 字段 | 内容 |
|---|---|
| 审批结论 | `approved` |
| 审批人 | user |
| 审批时间 | 2026-06-22T19:08:32+08:00 |
| 接受决策 | `DQ-CP8-CR118-01`、`DQ-CP8-CR118-02`、`DQ-CP8-CR118-03` |
| 关闭状态 | `closed-current-delivery / READY` |
| 接受风险 | `R-CR118-001`，历史 `docs/design/*` alias 不批量重写 |
| 后续选择 | 当前不新增 follow-up |
| 不授权项 | 目录 / symlink 变更、runtime、NAS、凭据、交易写、provider fetch、lake write、catalog publish |

## Owner Discovery

| 对象 | 候选路径 | 角色 | 结论 |
|---|---|---|---|
| human gate checker | `scripts/check_human_gate_decision_brief.py` | primary owner | 最小 enforcement 可扩展此 checker，使 CP2/CP3/CP5/CP8 人工门禁检查 path alias 证据引用。 |
| CR tracking checker | `scripts/check_cr_tracking_consistency.py` | supporting validation | 适合继续检查 active / closed / candidate 状态一致性，不作为 path alias enforcement 主 owner。 |
| fixture tests | `tests/` 下新增或扩展 human gate checker 测试 | validation owner | CP5 若批准，需添加最小 fixture 覆盖：`process/docs/design/*` 推荐入口通过、仅 `docs/design/*` 作为唯一证据路径时失败或提示。 |

## 处理结论

- 审批结论：`cp8-approved-user-20260622-ready`
- [ ] 自动批准
- [x] CP2 人工确认已通过
- [x] CP5 人工确认已通过
- [x] 实现和本地 fixture 验证已完成
- [x] CP8 人工确认已通过
