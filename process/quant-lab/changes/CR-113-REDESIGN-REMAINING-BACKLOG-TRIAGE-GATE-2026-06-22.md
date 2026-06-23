---
cr_id: "CR-113"
cr_type: "product-scope"
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
rollback_to: "CR112 closed READY with no active formal CR"
approval_result: "cp8-approved-user-20260622-ready"
created_at: "2026-06-22T16:13:35+08:00"
created_by: "host-orchestrator"
approved_by: "user"
approved_at: "2026-06-22T16:39:47+08:00"
source: "user"
linked_issue: ""
parent_cr: "CR-112"
source_checkpoint: "process/checkpoints/CP8-CR112-DELIVERY-READINESS.md"
source_decision_id: "USER-20260622-START-REDESIGN-REMAINING-BACKLOG-TRIAGE"
follow_up_type: "redesign-remaining-backlog-triage"
risk_class: "low-scope-no-runtime"
owner: "host-orchestrator"
revisit_condition: "If the user asks to modify source code, tests, checker implementation, runtime, NAS, credentials, trading write or provider/lake/catalog publish, stop CR113 default route and create a separate scoped CR."
acceptance_criteria: "CR113 reviews and ranks remaining redesign backlog candidates, classifies start/defer/not-start recommendations, and keeps all implementation/runtime work out of scope."
close_condition: "Closed by CP8 approval at 2026-06-22T16:39:47+08:00 as READY. Ranked remaining backlog and follow-up tracking are written; no source code, tests, checker implementation, runtime, NAS, credentials, trading or publish action is authorized."
cr_index_path: "process/changes/CR-INDEX.yaml"
current_requirement_baseline_path: "process/baseline/CURRENT-REQUIREMENT-BASELINE.yaml"
historical_baseline_status: "active"
reframed_by: []
reframe_summary: ""
---

# CR-113 Redesign Remaining Backlog Triage Gate

## 变更描述

用户明确要求：

> 下一步启动一个轻量的 Redesign Remaining Backlog Triage Gate，只做候选复盘和排序，不改代码、不启动 runtime。

本 CR 承接 CR112 closed-current-delivery / READY 后的空闲状态。CR113 只做剩余 redesign backlog 候选复盘、排序、分流和下一步推荐，不创建实现 Story，不修改源码或测试，不启动任何 runtime。

本 CR **不是** checker implementation CR，不启动 CR105，不恢复 CR089，不连接 QMT / MiniQMT / XtQuant / gateway runtime，不访问 NAS，不读取凭据 / 账户 / raw log，不执行 submit / cancel / buy / sell、simulation / live、provider fetch、lake write 或 catalog publish。

## CP8 Follow-up 来源

| 字段 | 内容 |
|---|---|
| 父级 CR | `CR-112` |
| 来源检查点 | `process/checkpoints/CP8-CR112-DELIVERY-READINESS.md` |
| 来源决策 ID | `USER-20260622-START-REDESIGN-REMAINING-BACKLOG-TRIAGE` |
| follow-up 类型 | `redesign-remaining-backlog-triage` |
| 风险等级 | `low-scope-no-runtime` |
| owner | `host-orchestrator` |
| 重访条件 | 用户要求源码、checker 实现、runtime、NAS、凭据、交易写或 publish 时，另起独立 CR。 |
| 验收标准 | 输出剩余候选复盘、排序、分流和下一 CR 推荐；不改代码、不启动 runtime。 |
| 关闭条件 | CP2 确认范围，CP8 确认排序和 follow-up tracking。 |

## CR 类型与门禁策略

| 字段 | 内容 |
|---|---|
| CR 类型 | `product-scope` |
| Legacy CR kind | `requirement-change` |
| 生命周期状态 | `closed` |
| 就绪状态 | `ready` |
| 门禁状态 | `closed` |
| 门禁模板 | `compact` |

## 结构化权限策略

```yaml
authorization_policy:
  source_code:
    modify: false
    refactor: false
    test_change: false
    checker_implementation_change: false
  process_artifacts:
    read: true
    write_triage_evidence: true
    update_cr_index: true
    update_state_summary: true
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

- 读取 CR107-CR112 关闭证据、follow-up tracking、CR-INDEX、STATE 顶层摘要和已存在 redesign / governance 文档。
- 生成 CR113 triage 文档、context capsule、CP2 预检和人工审查稿。
- 对候选项做分类：推荐启动、保留候选、延后、需要独立实现 CR、需要高风险授权 gate、不启动。
- 运行本地只读状态检查或 git status；不启动服务、不连接外部 runtime。

## 文档处理决策

| 受影响文档 | 处理方式 | 旧基线保留方式 | 修订记录位置 | 批准状态 |
|---|---|---|---|---|
| `docs/design/REDESIGN-REMAINING-BACKLOG-TRIAGE-CR113.md` | 新增 | N/A | 文件头部修订记录 | approved-for-cp8 |
| `process/changes/CR-INDEX.yaml` | 原文档更新 | 追加 CR113 active 项，不删除历史项 | 不适用 | approved-for-cp8 |
| `process/STATE.md` | 原文档更新 | 只刷新顶层人类摘要，不重写历史长表 | 不适用 | approved-for-cp8 |
| 源码 / tests / checker implementation | 不变 | 不授权修改 | 不适用 | approved |

## 五维度影响分析

| 维度 | 评估问题 | 受影响对象 | 结论 | 处理动作 |
|------|----------|-----------|------|---------|
| 需求层 | 是否新增、删除或重定义 REQ-* | `REQUIREMENTS.md` / current baseline | false | 不改需求基线；只做 backlog triage。 |
| 场景层 | 是否改变测试矩阵覆盖范围 | `SCENARIOS.yaml` / `TEST-MATRIX.md` | false | 不改测试矩阵；未来候选启动时再定义验证范围。 |
| 计划层 | 是否改变 Phase、Wave、Story / 任务依赖 | redesign backlog / candidate queue | true | 输出剩余候选排序，不创建 Story 实现批次。 |
| 安全层 | 是否引入新的高风险动作或权限要求 | runtime / NAS / credentials / trading / publish boundary | false | 继续禁止源码、checker implementation、runtime、NAS、凭据、交易写和 publish。 |
| 交付层 | 是否需要重新生成交付物或回归子集 | `process/checks` / `process/checkpoints` / `docs/design` | true | 生成 CR113 triage、CP2 scope gate 和后续 CP8 收口证据。 |

## fast-lane 判定

| 条件 | 是否命中 | 说明 |
|---|---|---|
| 仅低风险轻量文档 / 规则 / 排序修改 | true | 只做候选复盘、排序和台账同步。 |
| 修改架构、权限、安全边界或平台安装路径 | false | 不修改架构契约；只判断是否应启动未来 HLD。 |
| 修改外部接口契约、文件所有权或多 Story 依赖 | false | 不修改接口和文件所有权，不创建实现 Story。 |
| 需要 HLD / LLD 才能解释影响 | false | 本轮只输出候选优先级；HLD 候选未来单独启动。 |
| 是否保持 fast-lane | true | 保留 CP2 和 CP8 审查证据。 |

## LLD 设计批次门禁

- 是否需要 LLD 设计批次：false。
- batch_id：N/A。
- 批次范围来源：N/A。
- 批次内 Story：N/A。
- 批次人工确认稿：N/A。
- 开发启动条件：
  - [ ] 不适用；CR113 不启动代码实现。

## 执行链路

| 顺序 | 责任角色 | 动作 | 输入 | 输出 | 门控 | 完成后下一步 |
|---|---|---|---|---|---|---|
| 1 | `host-orchestrator` | 创建 CR113 和 CP2 triage scope gate | 用户明确启动轻量 triage gate | CR113、triage draft、CP2 自动预检、CP2 人工审查稿 | CP2 pending | 等待用户确认 triage 范围和排序原则 |
| 2 | `host-orchestrator` | 收敛剩余候选排序 | CP2 approved | ranked remaining backlog / next CR recommendation | CP8 pending | 用户确认关闭或启动下一 CR |
| 3 | `host-orchestrator` | 同步 CR tracking | CR113 结论 | CR-INDEX / STATE / follow-up tracking | CP8 | active formal CR 恢复为 none |

## 后续事项台账

- 是否存在后续事项：待 CP8 判断。
- 台账路径：`process/changes/CR-113-FOLLOW-UP-TRACKING-2026-06-22.md`。
- CR 索引路径：`process/changes/CR-INDEX.yaml`。

## CP2 待确认决策

| 决策 ID | 类型 | 推荐方案 | 备选方案 | 回退条件 |
|---|---|---|---|---|
| DQ-CP2-CR113-01 | scope | 接受 CR113 只做 no-code/no-runtime remaining backlog triage，不改代码、不启动 runtime | A. 暂停 triage；B. 直接选择单个候选启动；C. 改为实现计划 CR | 若用户要求源码或 runtime，另起独立 CR |
| DQ-CP2-CR113-02 | follow_up_tracking | 接受按“低风险可决策优先、实现/高风险延后”的排序原则复盘剩余候选 | A. 优先 architecture-only HLD；B. 优先 context/human gate consistency；C. 优先 checker implementation alignment | 若用户调整排序，更新 triage 文档并重新发起 CP2 |
| DQ-CP2-CR113-03 | runtime_authorization | 继续禁止源码修改、tests 修改、checker implementation change、CR105、CR089 恢复、runtime、交易写、NAS、凭据和 publish 被 CR113 隐式启动 | A. 另起源码实现 CR；B. 另起 checker implementation CR；C. 另起 runtime/NAS/trading authorization gate | 若用户明确要求高风险验证，停止 CR113 默认路线并创建独立 gate |

## CP2 审批结果

| 字段 | 内容 |
|---|---|
| 审批结论 | `approved` |
| 审批人 | user |
| 审批时间 | 2026-06-22T16:21:59+08:00 |
| 接受决策 | `DQ-CP2-CR113-01`、`DQ-CP2-CR113-02`、`DQ-CP2-CR113-03` |
| 下一门禁 | CP8 delivery readiness |
| 不授权项 | 源码修改、tests 修改、checker implementation change、CR105、CR089 恢复、runtime、交易写、NAS、凭据、provider/lake/catalog publish |

## CP8 审批结果

| 字段 | 内容 |
|---|---|
| 审批结论 | `approved` |
| 审批人 | user |
| 审批时间 | 2026-06-22T16:39:47+08:00 |
| 接受决策 | `DQ-CP8-CR113-01`、`DQ-CP8-CR113-02` |
| 关闭状态 | `closed-current-delivery / READY` |
| 默认下一候选 | `FU-CR113-001 Context Capsule / Human Gate Consistency Review`，仅登记为 candidate、不启动 |
| 不授权项 | 源码修改、tests 修改、checker implementation change、CR105、CR089 恢复、runtime、交易写、NAS、凭据、provider/lake/catalog publish |

## 当前状态

| 字段 | 内容 |
|---|---|
| 当前门禁 | closed |
| 自动预检 | `process/checks/CP2-CR113-REDESIGN-REMAINING-BACKLOG-TRIAGE-PRECHECK.md` |
| 人工审查稿 | `process/checkpoints/CP2-CR113-REDESIGN-REMAINING-BACKLOG-TRIAGE-REVIEW.md` |
| Triage draft | `docs/design/REDESIGN-REMAINING-BACKLOG-TRIAGE-CR113.md` |
| 下一步 | CR113 已关闭为 READY；`FU-CR113-001` 仅为后续 candidate，需用户明确选择并另起正式 CR 才能启动。 |
