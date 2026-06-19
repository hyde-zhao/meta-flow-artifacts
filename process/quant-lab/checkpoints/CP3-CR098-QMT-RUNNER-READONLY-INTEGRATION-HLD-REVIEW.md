---
checkpoint_id: "CP3"
checkpoint_name: "CR098 QMT Runner Readonly Integration HLD Review"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-06-19T12:00:19+08:00"
reviewed_by: "user"
reviewed_at: "2026-06-19T12:07:32+08:00"
auto_check_result: "process/checks/CP3-CR098-QMT-RUNNER-READONLY-INTEGRATION-HLD-PRECHECK.md"
target:
  phase: "solution-design"
  story_id: "CR098-QMT-RUNNER-READONLY-INTEGRATION"
  artifacts:
    - "docs/qmt/CR098-QMT-RUNNER-READONLY-GATEWAY-INTEGRATION-HLD.md"
    - "process/context/CP3-CR098-RUNNER-READONLY-INTEGRATION-CONTEXT.yaml"
---

# CP3 CR098 QMT Runner Readonly Integration HLD Review 人工审查

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP3-CR098-QMT-RUNNER-READONLY-INTEGRATION-HLD-PRECHECK.md` | PASS | 0 | 推荐接受 HLD 并进入 CP5 设计准备 |

## Decision Brief

CR098 HLD 推荐采用 `runner-owned readonly gateway adapter facade`：在 CR091 runner 的 readonly wrapper 基础上扩展 fake/real 可注入 adapter，默认 fake/offline；真实 REST gateway transport 只能在后续逐 run runtime authorization 后启用。HLD 不允许 runner 直接 import XtQuant，不合并 NAS，不引入 order-write。

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP3-CR098-RUNNER-READONLY-INTEGRATION-CONTEXT.yaml` |
| capsule 状态 | `ready` |
| read_profile | `minimal` |
| 默认读取策略 | 先读 capsule；仅在字段缺失、冲突、人工审计或深度评审时读取完整文档 |
| 全文档读取扩展 | 0 次 |
| 缺失 / waived 理由 | N/A |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| CP2 checkpoint | `process/checkpoints/CP2-CR098-QMT-RUNNER-READONLY-INTEGRATION-SCOPE-REVIEW.md` | scanned | 5 | 5 | CP2 决策已进入 HLD |
| HLD | `docs/qmt/CR098-QMT-RUNNER-READONLY-GATEWAY-INTEGRATION-HLD.md` | scanned | 5 | 5 | 架构 / 安全 / 实施 / runtime / follow-up 纳入 |
| AGA discussion | `process/discussions/CP3-CR098-HLD-DISCUSSION-LOG.md` | scanned | 4 | 4 | 灰区已收敛 |
| CP3 context | `process/context/CP3-CR098-RUNNER-READONLY-INTEGRATION-CONTEXT.yaml` | scanned | 3 | 3 | 关键风险和下游任务纳入 |
| STATE pending queue | `process/STATE.md` | scanned | 0 | 0 | 无未回答阻断项 |

### 待人工决策清单

如果你回复 `approve`，表示你接受以下 5 项推荐方案；不表示授权后面的禁止操作。

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP3-CR098-01 | architecture | 是否采用 runner-owned readonly gateway adapter facade？ | 采用，在 `trading/strategy_runner/readonly_gateway.py` 扩展 fake/real 可注入 facade。 | A: 独立 runtime CLI；B: runner 直接 import XtQuant。 | 推荐方案证明 runner integration 且安全；独立 CLI 集成弱；直接 XtQuant 风险高。 | 影响 adapter、tests、evidence。 | CP5 发现 wrapper 无法安全扩展时切独立 CLI。 |
| DQ-CP3-CR098-02 | security | 是否确认 per-run explicit env path，禁止隐式 secret read？ | 确认；真实 adapter 只接受显式 env path 且需逐 run 授权。 | A: 默认读取 `.quant-lab` 固定 env；B: 读取 Windows `.env`。 | 推荐方案最小敏感面；备选会扩大 secret 暴露。 | 需要 CP5 定义 env contract 和 redaction tests。 | 若需自动化 secret 管理，另起安全设计 CR。 |
| DQ-CP3-CR098-03 | implementation | 是否允许 CP5 设计 adapter / evidence / tests，但 CP5 前不写代码？ | 允许进入 CP5，冻结 LLD、TEST-PLAN、TASKS 后再实现。 | A: 立即实现；B: 只保留 HLD 不实施。 | 推荐方案符合标准门禁；立即实现越过 CP5；只保留 HLD 无法完成集成。 | 下一步是 CP5，而非代码。 | 若用户 reject，回 CP2/CP3 修改。 |
| DQ-CP3-CR098-04 | runtime_authorization | CP3 approve 是否授权真实 runner runtime？ | 不授权；真实 runner smoke 必须后续逐 run 授权。 | A: CP3 直接授权 runner smoke；B: 永久禁止 runtime。 | 推荐方案避免设计批准被误读为运行授权；直接授权过早；永久禁止无法验证。 | 当前不读 secret、不启动 gateway、不执行 runner。 | CP5/CP7 需要执行时另发授权。 |
| DQ-CP3-CR098-05 | follow_up_tracking | NAS/order-write/非空复测是否继续独立？ | 独立，不并入 CR098。 | A: 合并 NAS；B: 合并 order-write。 | 推荐方案保持最小闭环；合并会显著扩大权限。 | CR098 不解决包交换或下单。 | 用户明确选择时另起独立 CR。 |

### 用户需决策事项

| 字段 | 内容 |
|---|---|
| 推荐决策 | `approve` 接受 `DQ-CP3-CR098-01..05`，进入 CP5 设计准备 |
| 备选方案 | `修改: <具体修改点>` 后重出 CP3；或 `reject` 回 CP2 / 暂停 CR098 |
| 影响维度 | 架构边界、secret 处理、实现门禁、runtime 授权、后续分流 |
| 风险与回退 | 风险为 HIGH 但 CP3 仍不执行 runtime；运行风险后续逐 run 授权 |
| 用户需决策事项 | 是否接受 HLD 推荐架构与不授权边界 |

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
| CP2 approved | approved | `process/checkpoints/CP2-CR098-QMT-RUNNER-READONLY-INTEGRATION-SCOPE-REVIEW.md` | 用户已同意 |
| HLD ready | approved | `docs/qmt/CR098-QMT-RUNNER-READONLY-GATEWAY-INTEGRATION-HLD.md` | 用户已同意 |
| AGA ready | approved | `process/discussions/CP3-CR098-HLD-DISCUSSION-LOG.md` | 已记录 |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | 推荐架构是否可接受 | approved | `DQ-CP3-CR098-01` | 接受 runner-owned readonly facade |
| 2 | secret/env 边界是否可接受 | approved | `DQ-CP3-CR098-02` | 接受 per-run explicit env path |
| 3 | CP5 前不实现是否可接受 | approved | `DQ-CP3-CR098-03` | 接受标准门禁 |
| 4 | runtime 不授权是否清楚 | approved | `DQ-CP3-CR098-04` | CP3 不授权执行 |
| 5 | follow-up 分流是否清楚 | approved | `DQ-CP3-CR098-05` | NAS/order-write 独立 |

## Exit Criteria

| 用户回复 | 处理 |
|---|---|
| `approve` | 接受 CP3 HLD，进入 CP5 design readiness；仍不授权 runtime。 |
| `修改: <具体修改点>` | 按修改点修订 HLD / CP3 材料并重新发起确认。 |
| `reject` | 回退到 CP2 或暂停 CR098。 |

## Deliverables

- `docs/qmt/CR098-QMT-RUNNER-READONLY-GATEWAY-INTEGRATION-HLD.md`
- `process/discussions/CP3-CR098-HLD-DISCUSSION-LOG.md`
- `process/checks/CP3-CR098-DISCUSSION-CHECKPOINT.json`
- `process/context/CP3-CR098-RUNNER-READONLY-INTEGRATION-CONTEXT.yaml`
- `process/checks/CP3-CR098-QMT-RUNNER-READONLY-INTEGRATION-HLD-PRECHECK.md`

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-06-19T12:07:32+08:00
- 用户回复：同意
- 接受的决策项：`DQ-CP3-CR098-01..05`
- 修改意见：无
- 不授权边界：CP3 approve 只允许进入 CP5 design readiness；不授权 HMAC secret 读取、Windows `.env`、gateway 启动、runner runtime、账户原文、NAS、交易写、simulation/live、provider/lake/publish、CR089 auto-start 或 CR020 gateway route restore。
