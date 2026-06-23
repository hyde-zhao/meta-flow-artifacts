---
checkpoint_id: "CP2-CR126"
checkpoint_name: "CR126 Runner Development Readiness Scope Review"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-06-23T13:40:14+08:00"
reviewed_by: "user"
reviewed_at: "2026-06-23"
auto_check_result: "process/checks/CP2-CR126-RUNNER-DEVELOPMENT-READINESS-PRECHECK.md"
target:
  phase: "requirement-clarification"
  story_id: ""
  artifacts:
    - "process/context/CP2-CR126-RUNNER-DEVELOPMENT-READINESS-CONTEXT.yaml"
    - "process/changes/CR-126-QUANT-LAB-RUNNER-DEVELOPMENT-READINESS-RUNNER-ENTRY-DESIGN-2026-06-23.md"
---

# CP2 CR126 Runner Development Readiness Scope Review

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP2-CR126-RUNNER-DEVELOPMENT-READINESS-PRECHECK.md` | PASS | 0 | 可发起 runner readiness 范围确认。 |

## Decision Brief

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP2-CR126-RUNNER-DEVELOPMENT-READINESS-CONTEXT.yaml` |
| capsule 状态 | ready |
| read_profile | minimal |
| 默认读取策略 | 先读 capsule；仅在缺失、冲突、字段不足、人工审计或用户要求时读取完整来源。 |
| 全文档读取扩展 | 1 次，只读 runner 入口、adapter、evidence、相关测试和 runtime collector 边界。 |
| 缺失 / waived 理由 | N/A |

### 压缩规划摘要

| 项 | 当前口径 |
|---|---|
| CR126 | Runner Entry Design / Scope Gate；仅冻结 runner MVP 范围、模块边界、禁止项和 CR127 实现范围。 |
| CR127 | Runner Core MVP Implementation；集中完成 offline runner core，不再把 `run_spec`、runner entry、result、evidence、安全测试拆成多个正式 CR。 |
| CR128+ | 仅当用户明确要求触碰 readonly runtime、NAS exchange、provider/lake/catalog、simulation/live 等外部权限域时才启动。 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| CR126 context | `process/context/CP2-CR126-RUNNER-DEVELOPMENT-READINESS-CONTEXT.yaml` | scanned | 3 | 3 | scope、architecture、runtime_authorization。 |
| runner code entry | `trading/strategy_runner/*` | scanned-readonly | 1 | 1 | architecture 决策并入 DQ-CP2-CR126-02。 |
| runtime / external boundary | QMT/NAS/provider/lake/catalog | scanned-as-forbidden | 1 | 1 | runtime_authorization 决策并入 DQ-CP2-CR126-03。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP2-CR126-01 | scope | 是否确认 CR126 从历史文档清理切换到 runner development readiness / runner entry design？ | 确认切换；历史 docs cleanup 只在直接阻塞 runner 开发时另起小 CR，runner 本体后续集中到 CR127。 | A. 继续清理历史文档后再进入 runner；B. 跳过入口设计直接进入源码实现。 | 推荐方案是最短路径；A 会继续消耗在低收益清理上；B 容易把 runtime/source 权限混在一起。 | 影响后续工作是否聚焦 runner 开发入口，并把 runner core MVP 收敛到 CR127。 | 若后续发现缺失文档直接阻塞 runner，可另起小范围文档 CR。 |
| DQ-CP2-CR126-02 | architecture | runner 入口是否以现有 `trading/strategy_runner` 离线 package/adapter/evidence 合同为基线？ | 以现有离线合同为基线；CR127 集中实现 offline runner core，真实 runtime 保持 CR128+ 独立 gate。 | A. 以 QMT runtime collector 为入口；B. 重新设计全新 runner 包结构。 | 推荐方案复用已测试边界且风险最低；A 会触发 runtime 授权问题；B 周期更长。 | 决定 CR127 的技术入口和内部 slice 顺序。 | 若后续离线合同不足，在 CR127 内通过设计 delta 扩展；外部权限域仍走 CR128+。 |
| DQ-CP2-CR126-03 | runtime_authorization | CP2 是否授权 runtime、NAS、QMT、凭据、provider/lake/catalog 或交易写操作？ | 不授权；全部冻结为 CR128+ 可选后续 gate，且默认不启动。 | A. 另起 runtime authorization gate；B. 仅由用户手动回填脱敏 runtime 摘要。 | 推荐方案权限最小；A/B 都需要独立门禁，不应混入本 CR。 | 防止入口设计阶段误触外部系统或真实交易边界。 | 任一禁止项必须另起独立 CR 并重新 CP2/CP5。 |

| 字段 | 内容 |
|---|---|
| 推荐决策 | `approve` 接受 CP2 三项推荐，确认 CR126 范围和禁止项。 |
| 备选方案 | `修改: <具体修改点>` 指明要调整的 DQ；或 `reject` 不接受本范围。 |
| 影响维度 | runner development readiness、adapter boundary、runtime authorization、historical docs cleanup stop-line |
| 风险与回退 | 若不接受范围，回到 CR125 后续建议重新排序；若误把 runtime 纳入，必须停止并另起高风险 gate。 |
| 用户需决策事项 | 是否批准 `DQ-CP2-CR126-01..03`。 |
| 自动终验授权 | `auto_final_authorization: false`；CP2 approve 不授权 CP6、源码实现、runtime 或外部访问。 |

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| CP2 precheck PASS | 待审查 | `process/checks/CP2-CR126-RUNNER-DEVELOPMENT-READINESS-PRECHECK.md` | 无 blocker。 |
| context ready | 待审查 | `process/context/CP2-CR126-RUNNER-DEVELOPMENT-READINESS-CONTEXT.yaml` | ready。 |
| CR126 formal file ready | 待审查 | `process/changes/CR-126-QUANT-LAB-RUNNER-DEVELOPMENT-READINESS-RUNNER-ENTRY-DESIGN-2026-06-23.md` | active-gate-prep。 |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | runner readiness 范围清晰 | approved | DQ-CP2-CR126-01 | 用户 approve；接受 CR126 + CR127 压缩规划。 |
| 2 | runner 入口基线清晰 | approved | DQ-CP2-CR126-02 | 用户 approve；以离线 package/adapter/evidence 合同为基线。 |
| 3 | runtime / external 禁止项清晰 | approved | DQ-CP2-CR126-03 | 用户 approve；当前不授权外部动作，未来需要时另行授权门禁。 |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| 用户确认 CP2 | approved | 本文件人工审查结果 | 用户已 approve。 |
| 可进入 CP5 | approved | DQ-CP2-CR126-01..03 | CP2 已批准，可消费 CP5 授权结论。 |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| CP2 context | `process/context/CP2-CR126-RUNNER-DEVELOPMENT-READINESS-CONTEXT.yaml` | approved | 已按压缩规划更新。 |
| CP2 precheck | `process/checks/CP2-CR126-RUNNER-DEVELOPMENT-READINESS-PRECHECK.md` | approved | PASS。 |

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-06-23
- 修改意见：用户选择 `approve`，并补充“需要外部权限是我可以授权，你需要完成既定目标”。本次批准按当前 Decision Brief 解释：接受 CR126 + CR127 压缩规划；当前不直接授权 runtime、NAS、QMT、凭据、provider/lake/catalog、commit/push、源码修改或交易写操作。若完成既定目标确需外部权限，需由 host-orchestrator 另行发起独立授权门禁。
- 风险接受项：接受当前 process/design gate 继续推进到 CP6 设计材料阶段；不接受未授权外部动作风险。
