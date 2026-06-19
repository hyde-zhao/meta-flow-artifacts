---
checkpoint_id: "CP2"
checkpoint_name: "CR098 QMT Runner Readonly Integration Scope Review"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-06-19T11:55:26+08:00"
reviewed_by: "user"
reviewed_at: "2026-06-19T12:00:19+08:00"
auto_check_result: "process/checks/CP2-CR098-QMT-RUNNER-READONLY-INTEGRATION-SCOPE-PRECHECK.md"
target:
  phase: "requirement-clarification"
  story_id: "CR098-QMT-RUNNER-READONLY-INTEGRATION"
  artifacts:
    - "process/changes/CR-098-QMT-RUNNER-READONLY-GATEWAY-INTEGRATION-SMOKE-2026-06-19.md"
    - "process/context/CP2-CR098-RUNNER-READONLY-INTEGRATION-CONTEXT.yaml"
---

# CP2 CR098 QMT Runner Readonly Integration Scope Review 人工审查

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP2-CR098-QMT-RUNNER-READONLY-INTEGRATION-SCOPE-PRECHECK.md` | PASS | 0 | 推荐进入 CP3 HLD，不授权 runtime 执行 |

## Decision Brief

CR098 的用户意图是把 CR091 离线 runner 与 CR097 已验证的 Windows gateway 只读能力连接起来。推荐首轮只做 runner readonly integration：runner 能在后续受控授权下调用 gateway `health`、`capabilities` 和 `query_positions_readonly`，并输出 `.quant-lab` redacted evidence。CP2 approve 只确认范围和设计方向，不授权读取 HMAC secret、不启动 gateway、不执行 runner runtime。

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP2-CR098-RUNNER-READONLY-INTEGRATION-CONTEXT.yaml` |
| capsule 状态 | `ready` |
| read_profile | `minimal` |
| 默认读取策略 | 先读 capsule；仅在字段缺失、冲突、人工审计或深度评审时读取完整文档 |
| 全文档读取扩展 | 0 次 |
| 缺失 / waived 理由 | N/A |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `process/STATE.md` | scanned | 1 | 1 | next_action 推荐启动 runner readonly integration |
| 用户显式选择题 | 当前对话 | scanned | 1 | 1 | 用户明确启动 CR098 |
| CR098 | `process/changes/CR-098-QMT-RUNNER-READONLY-GATEWAY-INTEGRATION-SMOKE-2026-06-19.md` | scanned | 4 | 4 | scope / architecture / runtime / security 纳入 |
| 上游 CR091 / CR097 | CR091 / CR097 checkpoints | scanned | 2 | 2 | runner baseline 和 gateway evidence 纳入 |
| discussion log / checkpoint | N/A | n/a | 0 | 0 | 本轮用户直接选择明确 gate，暂无多轮灰区讨论 |

### 待人工决策清单

如果你回复 `approve`，表示你接受以下 5 项推荐方案；不表示授权后面的禁止操作。

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP2-CR098-01 | scope | 本轮 CR098 是否只覆盖 runner readonly integration？ | 是，仅覆盖 runner 调用 gateway health / capabilities / query_positions_readonly 和 redacted evidence。 | A: 继续只做手工 gateway smoke；B: 合并 NAS package exchange。 | 推荐方案能证明 runner integration；A 覆盖不足；B 范围膨胀。 | 将进入 CP3/CP5 设计和后续实现。 | 若不需要 runner，选择 A；若要 NAS，另起 CR091-FU-02。 |
| DQ-CP2-CR098-02 | architecture | runner 如何消费真实 gateway？ | 新增 runner-owned readonly gateway client adapter，默认仍 fake/offline，真实 adapter 需显式启用。 | A: 手工脚本调用 gateway；B: runner 直接 import XtQuant。 | 推荐方案边界清楚；手工脚本不能证明 runner；直接 import XtQuant 风险高。 | 需要设计 adapter、env contract、tests。 | 若 CP3 发现复杂度过高，可回退为手工脚本方案。 |
| DQ-CP2-CR098-03 | runtime_authorization | CP2 approve 是否授权读取 HMAC env、启动 gateway 或执行 runner？ | 不授权；真实执行必须在 CP5 后逐 run 另行授权。 | A: 立即授权一次 runner smoke；B: 完全取消真实 runtime。 | 推荐方案避免设计批准被误读成运行授权；立即授权会越过设计门禁。 | 当前只写设计和门禁，不触碰 secret。 | 需要执行时再发 runtime authorization。 |
| DQ-CP2-CR098-04 | security | evidence 和敏感文件如何处理？ | 只允许 `.quant-lab/evidence/qmt/cr098/redacted/` 脱敏输出；最小 client env 只能在逐 run 授权时读取。 | A: 允许读取 Windows `.env`；B: 保存完整 CLI 输出。 | 推荐方案最小化敏感面；备选会泄露账号/secret/原始持仓风险。 | 需要 checker 或验证规则保证 raw 输出为 0。 | 如发现原文泄露，立即 BLOCKED 并停止。 |
| DQ-CP2-CR098-05 | follow_up_tracking | NAS、order-write、非空持仓复测如何处理？ | 全部保持独立候选，不并入 CR098。 | A: 合并 NAS；B: 合并 order-write。 | 推荐方案保持单一目标；合并会扩大权限和风险。 | CR098 不解决包交换或下单。 | 用户明确选择时另起独立 CR。 |

### 用户需决策事项

| 字段 | 内容 |
|---|---|
| 推荐决策 | `approve` 接受 `DQ-CP2-CR098-01..05`，进入 CP3 HLD |
| 备选方案 | `修改: <具体修改点>` 后重出 CP2；或 `reject` 取消 CR098 / 回到无 active CR |
| 影响维度 | runner integration 范围、adapter 架构、runtime 授权边界、evidence 安全、后续 CR 分流 |
| 风险与回退 | 风险为 HIGH 但当前只做设计门禁；运行风险在 CP5 后逐 run 授权 |
| 用户需决策事项 | 是否接受 CR098 只读 runner integration 的推荐范围和不授权边界 |

## 不授权范围

- 不授权读取 `/home/hyde/.quant-lab/runtime/qmt/cr097/client.env.template` 或任何 HMAC secret。
- 不授权读取 Windows `.env`、QMT 账号原文、资金、持仓明细原文、委托、成交或原始日志。
- 不授权启动 Windows gateway、执行 runner runtime 或查询账户。
- 不授权 NAS package exchange / pull / publish。
- 不授权 submit / cancel、buy / sell、simulation / live。
- 不授权 provider fetch、lake write、catalog publish。
- 不自动启动 CR089 或 CR020 gateway route restore。

## Entry Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| 用户启动 CR098 | 待审查 | 当前对话 | 用户明确要求启动 gate |
| CR098 已创建 | approved | `process/changes/CR-098-QMT-RUNNER-READONLY-GATEWAY-INTEGRATION-SMOKE-2026-06-19.md` | 范围已确认 |
| CP2 context ready | approved | `process/context/CP2-CR098-RUNNER-READONLY-INTEGRATION-CONTEXT.yaml` | 最小上下文已生成 |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | 范围是否清晰 | approved | Decision Brief | 仅 runner readonly integration |
| 2 | 架构方向是否可接受 | approved | `DQ-CP2-CR098-02` | runner-owned readonly adapter |
| 3 | runtime 不授权是否清楚 | approved | `DQ-CP2-CR098-03` | CP2 不授权 secret / gateway / runner run |
| 4 | evidence / secret 边界是否清楚 | approved | `DQ-CP2-CR098-04` | `.quant-lab` redacted evidence |
| 5 | 后续分流是否清楚 | approved | `DQ-CP2-CR098-05` | NAS/order-write/非空复测独立 |

## Exit Criteria

| 用户回复 | 处理 |
|---|---|
| `approve` | 接受 CP2 范围，进入 CP3 HLD；仍不授权 runtime。 |
| `修改: <具体修改点>` | 按修改点修订 CP2 材料并重新发起确认。 |
| `reject` | 取消 / 暂停 CR098，回到无 active CR。 |

## Deliverables

- `process/changes/CR-098-QMT-RUNNER-READONLY-GATEWAY-INTEGRATION-SMOKE-2026-06-19.md`
- `process/changes/CR-097-FOLLOW-UP-TRACKING-2026-06-19.md`
- `process/context/CP2-CR098-RUNNER-READONLY-INTEGRATION-CONTEXT.yaml`
- `process/checks/CP2-CR098-QMT-RUNNER-READONLY-INTEGRATION-SCOPE-PRECHECK.md`
- `process/checkpoints/CP2-CR098-QMT-RUNNER-READONLY-INTEGRATION-SCOPE-REVIEW.md`

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-06-19T12:00:19+08:00
- 用户回复：同意
- 接受的决策项：`DQ-CP2-CR098-01`、`DQ-CP2-CR098-02`、`DQ-CP2-CR098-03`、`DQ-CP2-CR098-04`、`DQ-CP2-CR098-05`
- 修改意见：无
- 不授权边界：仍不授权读取 `/home/hyde/.quant-lab/runtime/qmt/cr097/client.env.template` 或任何 HMAC secret；不授权读取 Windows `.env`、QMT 账号原文、资金、持仓明细原文、委托、成交或原始日志；不授权启动 Windows gateway、执行 runner runtime 或查询账户；不授权 NAS、submit/cancel、simulation/live、provider/lake/publish；不自动启动 CR089 或 CR020 gateway route restore。
