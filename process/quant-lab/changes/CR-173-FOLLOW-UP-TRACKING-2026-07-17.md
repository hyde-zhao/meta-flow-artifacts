---
source_cr: "CR-173"
status: "open"
created_at: "2026-07-17T16:08:21+08:00"
created_by: "host-orchestrator"
updated_at: "2026-07-17T16:08:21+08:00"
checkpoint_source: "CP8"
cr_index_path: "process/changes/CR-INDEX.json"
---

# CR-173 后续事项跟踪台账

## 目的

本台账记录 CR-173 `closed/cp8_closed/READY_WITH_RISK` 后、由 CR-172 PATH-I 评审进一步具化的后续候选。候选项未启动前不得创建正式 CR，不授权真实数据、runtime、公共 C1 写入、NAS、交易、发布或远端写入。

## 结构化候选项

```yaml
follow_up_items:
  - id: "FU-CR173-001"
    legacy_ids: []
    title: "Empirical dependency-matrix methodology v2 and sampling-error validation"
    kind: "requirement-change"
    lifecycle_status: "candidate"
    readiness_status: "n/a"
    gate_status: "not_started"
    gate_profile: "standard-code"
    source_cr: "CR-173"
    source_decision_id: "CR-172/DQ-CR172-014"
    priority: 1
    owner: "strategy-admission methodology owner"
    formal_cr_path: ""
    blocked_by: []
    superseded_by: []
    activation_trigger: "真实 dependency matrix 被分类为 empirical，且目标要求 effective_trial_count available 或 c1_computable=true"
    required_before: "任何 empirical R 的 available effective count 或 c1_computable=true 声明；不阻止 DQ-003 typed_unavailable 降级后的 PATH-C/A 设计"
    minimum_scope:
      - "sampling-error / uncertainty evidence"
      - "method version 2 and canonical method hash"
      - "identifiable input domain, bias boundary and failure semantics"
      - "independent methodology verification"
    non_authorization: "候选登记不授权真实 R 计算、真实数据、public C1 projection、runtime、Stage 3、交易或远端写入"
    impact_surface:
      - "effective-trial methodology"
      - "empirical dependency matrix"
      - "CR-172 PATH-C/A positive C1 claim"
    conflict_keys:
      documents: ["CR-172 HLD", "future empirical-method HLD"]
      stories: []
      files: ["engine/effective_trial_estimator.py", "engine/effective_trial_evidence.py"]
      external_interfaces: []
      security_runtime: ["real-data characterization remains separately authorized"]
      risk_acceptance: ["R-CR172-REAL-R-DOMAIN-MISMATCH", "R-CR173-SECOND-ORDER-MODEL-BIAS", "R-CR173-EXACT-INTEGER-GROWTH"]
      source_decisions: ["DQ-CR172-014", "DQ-CR172-003"]
    authorization_required:
      runtime: false
      credential_read: false
      nas_access: false
      trading_write: false
    next_action: "等待 CR-172 PATH-I CP3 完成 empirical input/provenance contract；若用户决定推进，先做冲突预检并创建独立正式 CR"
```

## 分流总览

| 类别 | 数量 | 阻断当前 PATH-I | 说明 |
|---|---:|---|---|
| 后续 CR 候选项 | 1 | 否 | PATH-I 可冻结设计且 PATH-C/A 可选择 DQ-003 降级；但 positive empirical effective count 被阻断 |
| 不授权范围 | 1 | 否 | 候选登记不等于实现或真实运行授权 |

## 候选索引

| 候选编号 | 标题 | 状态 | 优先级 | 阻断对象 | 当前门控 | 下一步 | 来源 |
|---|---|---|---:|---|---|---|---|
| FU-CR173-001 | Empirical dependency-matrix methodology v2 and sampling-error validation | candidate | 1 | empirical effective count available / `c1_computable=true` | 未启动 | PATH-I CP3 冻结输入合同后等待用户选择 | CR-172 DQ-014 |

## 三路线衔接

| empirical R 路线 | 是否需要 FU-CR173-001 先完成 | 输出上限 |
|---|---:|---|
| 完成 v2 并独立验证 | 是 | 仍需真实授权、binding 和 public projection 后才能讨论 `c1_computable=true` |
| 拆 future activation CR | 由该 CR 决定 | 当前 CR-172 不产生 available result |
| DQ-003 typed-unavailable 降级 | 否 | `effective_trial_count=typed_unavailable`、`c1_computable=false` |

## 启动候选流程

用户决定推进 `FU-CR173-001` 时，host-orchestrator 必须先读取本台账、`STATE.current.json.active_change`、CR ledger、CR index、CR-172 当前 HLD/owner/授权边界和所有 open/blocked CR summary，完成文件 owner、方法 owner、公共 C1、真实数据与运行授权冲突预检后，才能创建正式 CR。

## 不授权范围

| 项目 ID | 范围 | 原因 | 未来动作 | 来源 |
|---|---|---|---|---|
| NA-CR173-001 | 真实 empirical R 生成、真实数据读取、public C1 写入、Stage 3/runtime/trading | 本台账只登记方法升级候选 | 独立 CR + 对应人工门禁 | CR-173 CP8 / CR-172 DQ-014 |
