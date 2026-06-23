---
cr_id: "CR-117"
cr_type: "process"
cr_kind: "requirement-change"
lifecycle_status: "closed"
readiness_status: "ready"
gate_status: "closed"
gate_profile: "compact"
status: "closed-current-delivery"
impact_level: "low"
workflow_mode_before: "standard"
workflow_mode_after_change: "fast-lane"
fast_lane_upgrade_reason: ""
rollback_to: "CR116 closed READY with no active formal CR"
approval_result: "cp8-approved-user-20260622-ready"
created_at: "2026-06-22T17:53:44+08:00"
created_by: "host-orchestrator"
approved_by: "user"
approved_at: "2026-06-22T18:21:08+08:00"
source: "cp8-follow-up"
linked_issue: ""
parent_cr: "CR-116"
source_checkpoint: "process/checkpoints/CP8-CR116-DELIVERY-READINESS.md"
source_decision_id: "DQ-CP8-CR116-02"
follow_up_type: "context-path-alias-normalization-notes"
risk_class: "low-scope-no-runtime"
owner: "host-orchestrator"
revisit_condition: "If the notes require changing directories, symlinks, install paths, source code, tests, checker implementation, runtime access, NAS access, credential read, trading write or publish, stop CR117 and create a separate scoped CR."
acceptance_criteria: "CR117 records a checkable path alias display convention for context / design references, classifies docs/design and process/docs/design references, and keeps directory layout, symlinks, source code, tests and runtime out of scope."
close_condition: "CP8 approved by user at 2026-06-22T18:21:08+08:00; CR117 closed READY without directory, symlink, source, tests, checker or runtime changes."
cr_index_path: "process/changes/CR-INDEX.yaml"
current_requirement_baseline_path: "process/baseline/CURRENT-REQUIREMENT-BASELINE.yaml"
historical_baseline_status: "active"
reframed_by: []
reframe_summary: ""
---

# CR-117 Context Path Alias Normalization Notes

## 变更描述

用户明确要求：

> 启动 FU-CR114-002 Context Path Alias Normalization Notes

本 CR 从 `process/changes/CR-114-FOLLOW-UP-TRACKING-2026-06-22.md#FU-CR114-002` 转入正式 CR。目标是在 no-code / no-runtime 范围内，整理 context capsule、checkpoint、design notes 和对话提示中 `docs/design/*` 与 `process/docs/design/*` 的展示关系，形成可审计、可检查的路径别名说明。

本 CR 不改目录、不改 symlink、不迁移文件、不修改源码、不修改 tests、不修改 checker implementation，不启动 CR105，不恢复 CR089，不连接 QMT / MiniQMT / XtQuant / gateway runtime，不访问 NAS，不读取凭据 / 账户 / raw log，不执行 submit / cancel / buy / sell、simulation / live、provider fetch、lake write 或 catalog publish。

## CP8 Follow-up 来源

| 字段 | 内容 |
|---|---|
| 父级 CR | `CR-116` |
| 来源检查点 | `process/checkpoints/CP8-CR116-DELIVERY-READINESS.md` |
| 来源决策 ID | `DQ-CP8-CR116-02` |
| follow-up 类型 | `context-path-alias-normalization-notes` |
| 来源候选 | `FU-CR114-002` |
| 原始来源 | `CR-114` / `DQ-CP8-CR114-01` |
| 风险等级 | `low-scope-no-runtime` |
| owner | `host-orchestrator` |
| 重访条件 | 若需要改变目录、symlink、源码、tests、checker、runtime、NAS、凭据、交易写或 publish，另起独立 CR。 |
| 验收标准 | 输出路径别名展示约定 notes、适用范围、不授权边界和后续分流；不改目录、不改 symlink、不启动 runtime。 |
| 关闭条件 | CP2 确认范围，CP8 确认 notes 结论和 follow-up tracking。 |

## CR 类型与门禁策略

| 字段 | 内容 |
|---|---|
| CR 类型 | `process` |
| Legacy CR kind | `requirement-change` |
| 生命周期状态 | `closed` |
| 就绪状态 | `ready` |
| 门禁状态 | `closed` |
| 门禁模板 | `compact` |

## 结构化权限策略

```yaml
authorization_policy:
  path_layout:
    change_directories: false
    change_symlinks: false
    move_files: false
    rename_files: false
  source_code:
    modify: false
    refactor: false
    test_change: false
    checker_implementation_change: false
  process_artifacts:
    read: true
    write_audit_evidence: true
    update_cr_index: true
    update_state_summary: true
    update_path_alias_notes: true
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

允许动作仅限：

- 读取 CR114 / CR116 follow-up tracking、CR-INDEX、STATE 顶层摘要、相关 context capsule、checkpoint 和 design notes。
- 生成 CR117 notes、context capsule、CP2 预检和人工审查稿。
- 记录 `docs/design/*` 与 `process/docs/design/*` 的展示关系、推荐引用方式和例外说明。
- 运行本地只读状态检查或 git status；不启动服务、不连接外部 runtime。

## 文档处理决策

| 受影响文档 | 处理方式 | 旧基线保留方式 | 修订记录位置 | 批准状态 |
|---|---|---|---|---|
| `process/docs/design/CONTEXT-PATH-ALIAS-NORMALIZATION-NOTES-CR117.md` | 新增 | N/A | 文件头部修订记录 | cp2-approved |
| `process/context/CP2-CR117-CONTEXT-PATH-ALIAS-NORMALIZATION-CONTEXT.yaml` | 新增 | N/A | 不适用 | approved |
| `process/checks/CP2-CR117-CONTEXT-PATH-ALIAS-NORMALIZATION-PRECHECK.md` | 新增 | N/A | 不适用 | approved |
| `process/checkpoints/CP2-CR117-CONTEXT-PATH-ALIAS-NORMALIZATION-REVIEW.md` | 新增 | N/A | 不适用 | approved |
| `process/context/CP8-CR117-DELIVERY-CONTEXT.yaml` | 新增 | N/A | 不适用 | approved |
| `process/release/RELEASE-CONTEXT-CR117.yaml` | 新增 | N/A | 不适用 | approved |
| `process/checks/CP8-CR117-DELIVERY-READINESS.md` | 新增 | N/A | 不适用 | approved |
| `process/checkpoints/CP8-CR117-DELIVERY-READINESS.md` | 新增 | N/A | 不适用 | approved |
| `process/changes/CR-114-FOLLOW-UP-TRACKING-2026-06-22.md` | 原文档更新 | 保留 FU-CR114-002 历史来源并标记 active | 不适用 | cp2-approved |
| `process/changes/CR-116-FOLLOW-UP-TRACKING-2026-06-22.md` | 原文档更新 | 保留 CR116 后续分流并标记 related active CR | 不适用 | cp2-approved |
| `process/changes/CR-INDEX.yaml` | 原文档更新 | 追加 CR117 active 项，不删除历史项 | 不适用 | cp2-approved |
| `process/STATE.md` | 原文档更新 | 只刷新顶层人类摘要和当前门禁提示；不批量重写历史长表 | 不适用 | cp2-approved |
| 源码 / tests / checker implementation / 目录 / symlink | 不变 | 不授权修改 | 不适用 | approved |

## 五维度影响分析

| 维度 | 评估问题 | 受影响对象 | 结论 | 处理动作 |
|------|----------|-----------|------|---------|
| 需求层 | 是否新增、删除或重定义 REQ-* | `REQUIREMENTS.md` / current baseline | false | 不改需求基线；只做过程路径展示说明。 |
| 场景层 | 是否改变测试矩阵覆盖范围 | `SCENARIOS.yaml` / `TEST-MATRIX.md` | false | 不改测试矩阵；未来若要机器化 enforcement，再由独立实现 CR 定义验证范围。 |
| 计划层 | 是否改变 Phase、Wave、Story / 任务依赖 | context / checkpoint / docs path references | true | 输出路径别名 notes 和引用约定，不创建 Story 实现批次。 |
| 安全层 | 是否引入新的高风险动作或权限要求 | runtime / NAS / credentials / trading / publish boundary | false | 继续禁止目录 / symlink 修改、源码、checker implementation、runtime、NAS、凭据、交易写和 publish。 |
| 交付层 | 是否需要重新生成交付物或回归子集 | `process/docs/design` / `process/checks` / `process/checkpoints` | true | 生成 CR117 scope gate 和后续 CP8 收口证据。 |

## 冲突预检结论

| 检查维度 | 结果 | 证据 | 说明 |
|---|---|---|---|
| active formal CR | PASS | `meta-flow check cr-tracking --project-root .` | 启动前 active formal CR 为 none。 |
| blocked formal CR | PASS_WITH_NOTE | `CR-089` | CR089 仍 blocked；CR117 不恢复 CR089、不触碰 runtime / NAS / interface。 |
| 影响面重叠 | PASS | `FU-CR114-002` impact surface | 仅涉及 context path alias、docs design alias、process docs design alias 和 no runtime boundary。 |
| 高风险授权 | PASS | 本 CR authorization policy | 不授权 runtime、凭据、NAS、交易写、publish。 |
| 目录 / symlink / 源码 / tests / checker implementation | PASS | 本 CR authorization policy | 不授权目录、symlink、源码、tests 或 checker implementation 变更。 |

## fast-lane 判定

| 条件 | 是否命中 | 说明 |
|---|---|---|
| 仅低风险轻量文档 / 规则 / 排序修改 | true | 只做 notes、别名说明和台账同步。 |
| 修改架构、权限、安全边界或平台安装路径 | false | 不修改架构契约、权限边界或安装路径。 |
| 修改外部接口契约、文件所有权或多 Story 依赖 | false | 不修改接口和文件所有权，不创建实现 Story。 |
| 需要 HLD / LLD 才能解释影响 | false | 本轮只输出治理 notes；实现候选未来单独启动。 |
| 是否保持 fast-lane | true | 保留 CP2 和后续 CP8 审查证据。 |

## LLD 设计批次门禁

- 是否需要 LLD 设计批次：false。
- batch_id：N/A。
- 批次范围来源：N/A。
- 批次内 Story：N/A。
- 批次人工确认稿：N/A。
- 开发启动条件：
  - [ ] 不适用；CR117 不启动代码实现。

## CP2 待确认决策

| 决策 ID | 类型 | 推荐方案 | 备选方案 | 回退条件 |
|---|---|---|---|---|
| DQ-CP2-CR117-01 | scope | 接受 CR117 只做 no-code/no-runtime context path alias notes，不改目录、不改 symlink、不改源码、不启动 runtime | A. 暂停 CR117；B. 改为目录 / symlink 迁移设计 CR；C. 改为 checker implementation CR | 若用户要求目录、symlink、源码、runtime 或 checker enforcement，另起独立 CR |
| DQ-CP2-CR117-02 | implementation | 接受当前展示原则：正式过程路径以 `process/...` 为可点击入口；外置 artifact 真实路径只作为路由事实；`docs/design/*` 可作为历史/语义别名但不得替代 process symlink 入口 | A. 所有文档统一只写 `process/docs/design/*`；B. 所有文档统一只写 `docs/design/*`；C. 先保留混用只记录风险 | 推荐方案兼顾本地可点、外置路由和历史可读性；A/B 会丢失一侧语义；C 继续产生歧义 |
| DQ-CP2-CR117-03 | runtime_authorization | 继续禁止本 CR 隐式授权目录变更、symlink 变更、源码、tests、checker implementation、CR105、CR089、runtime、NAS、凭据、交易写、publish | A. 另起目录迁移 CR；B. 另起 checker implementation CR；C. 另起 runtime/NAS/trading authorization gate | 若用户明确要求高风险验证或实际路径变更，停止 CR117 默认路线并创建独立 gate |

## 当前状态

| 字段 | 内容 |
|---|---|
| 当前门禁 | closed |
| 自动预检 | `process/checks/CP2-CR117-CONTEXT-PATH-ALIAS-NORMALIZATION-PRECHECK.md` |
| 人工审查稿 | `process/checkpoints/CP2-CR117-CONTEXT-PATH-ALIAS-NORMALIZATION-REVIEW.md` |
| 下一步 | CR117 已关闭 READY；用户已选择后续 `FU-CR117-001`，已启动 `CR-118` CP2 门禁。 |

## CP2 审批结果

| 字段 | 内容 |
|---|---|
| 审批结论 | `approved` |
| 审批人 | user |
| 审批时间 | 2026-06-22T18:02:18+08:00 |
| 接受决策 | `DQ-CP2-CR117-01`、`DQ-CP2-CR117-02`、`DQ-CP2-CR117-03` |
| 下一门禁 | CP8 delivery readiness |
| 不授权项 | 目录变更、symlink 变更、文件移动或重命名、源码修改、tests 修改、checker implementation change、CR105、CR089 恢复、runtime、交易写、NAS、凭据、provider/lake/catalog publish |

## CP8 审批结果

| 字段 | 内容 |
|---|---|
| 审批结论 | `approved` |
| 审批人 | user |
| 审批时间 | 2026-06-22T18:21:08+08:00 |
| 接受决策 | `DQ-CP8-CR117-01`、`DQ-CP8-CR117-02`、`DQ-CP8-CR117-03` |
| 关闭状态 | `closed-current-delivery / READY` |
| 接受风险 | `R-CR117-001`，历史 `docs/design/*` 路径表达不批量重写 |
| 后续选择 | `FU-CR117-001` 转入 `CR-118` CP2 门禁 |
| 不授权项 | 目录变更、symlink 变更、文件移动或重命名、源码修改、tests 修改、checker implementation change、CR105、CR089 恢复、runtime、交易写、NAS、凭据、provider/lake/catalog publish |

## 处理结论

- 审批结论：`cp8-approved-user-20260622-ready`
- [ ] 自动批准（低风险）
- [x] 待 CP8 人工确认（低风险但涉及路径展示规则）
- [ ] 待人工审批（高风险）

## 关联对象

| 类型 | 标识 | 说明 |
|---|---|---|
| follow-up candidate | `FU-CR114-002` | CR114 / CR116 后续候选，已由用户明确选择启动。 |
| checkpoint | `process/checkpoints/CP8-CR114-DELIVERY-READINESS.md` | 来源 CP8 后续分流。 |
| checkpoint | `process/checkpoints/CP8-CR116-DELIVERY-READINESS.md` | CR116 关闭时保留该候选。 |
| notes | `process/docs/design/CONTEXT-PATH-ALIAS-NORMALIZATION-NOTES-CR117.md` | 本 CR 的 notes 草案。 |
