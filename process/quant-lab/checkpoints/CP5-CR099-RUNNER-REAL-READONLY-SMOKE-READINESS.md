---
checkpoint_id: "CP5"
checkpoint_name: "CR099 Runner Real Readonly Smoke Readiness"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-06-19T16:04:52+08:00"
reviewed_by: "user"
reviewed_at: "2026-06-19T16:13:23+08:00"
auto_check_result: "process/checks/CP5-CR099-RUNNER-REAL-READONLY-SMOKE-READINESS.md"
target:
  phase: "story-planning"
  story_id: "CR099-RUNNER-REAL-READONLY-SMOKE"
  artifacts:
    - "docs/features/cr099-runner-real-readonly-smoke/DESIGN.md"
    - "docs/features/cr099-runner-real-readonly-smoke/TEST-PLAN.md"
    - "docs/features/cr099-runner-real-readonly-smoke/TASKS.md"
    - "process/stories/CR099-RUNNER-REAL-READONLY-SMOKE-LLD.md"
    - "process/context/CP5-CR099-RUNNER-REAL-READONLY-SMOKE-CONTEXT.yaml"
---

# CP5 CR099 Runner Real Readonly Smoke Readiness 人工审查

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP5-CR099-RUNNER-REAL-READONLY-SMOKE-READINESS.md` | PASS | 0 | 推荐批准 run contract / redacted evidence schema / preflight blocked paths / TEST-PLAN，进入 CP6 offline contract implementation；不授权 runtime。 |

## Decision Brief

CR099 CP5 推荐批准 runner real readonly smoke 的设计证据：run contract、authorization gate、redacted evidence schema、blocked preflight rules、TEST-PLAN、TASKS 和 full LLD。批准后只允许进入 CP6 的离线 contract / schema checker / fixture tests；仍不授权读取 HMAC secret、Windows `.env`、启动 gateway、执行 runner runtime、读取账户原文、访问 NAS、交易写、simulation/live 或 provider/lake/publish。

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP5-CR099-RUNNER-REAL-READONLY-SMOKE-CONTEXT.yaml` |
| capsule 状态 | `ready-for-human-review` |
| read_profile | `compact` |
| 默认读取策略 | 先读 capsule；仅在缺失、冲突、字段不足、人工审计或深度评审时读取完整文档 |
| 全文档读取扩展 | 本轮读取 HLD、Feature Design、LLD、TEST-PLAN 和 TASKS |
| 缺失 / waived 理由 | N/A |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| CP3 checkpoint | `process/checkpoints/CP3-CR099-RUNNER-REAL-READONLY-SMOKE-HLD-REVIEW.md` | scanned | 5 | 0 | 已 approved，作为 CP5 输入 |
| HLD | `docs/qmt/CR099-QMT-RUNNER-REAL-READONLY-SMOKE-HLD.md` | scanned | 5 | 5 | architecture / runtime / security / implementation / follow-up 纳入 |
| Feature Design | `docs/features/cr099-runner-real-readonly-smoke/DESIGN.md` | scanned | 7 | 5 | run contract、authorization gate、schema、failure path 纳入 |
| LLD | `process/stories/CR099-RUNNER-REAL-READONLY-SMOKE-LLD.md` | scanned | 8 | 5 | 接口、数据结构、控制流、异常、测试纳入 |
| TEST-PLAN | `docs/features/cr099-runner-real-readonly-smoke/TEST-PLAN.md` | scanned | 8 | 5 | 必测场景和拒收规则纳入 |
| STATE pending queue | `process/STATE.md` | scanned | 0 | 0 | 无其他未回答阻断项 |

### 待人工决策清单

如果你回复 `approve`，表示你接受以下 5 项推荐方案；不表示授权后面的禁止操作。

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP5-CR099-01 | implementation | 是否批准当前 Feature Design / TEST-PLAN / TASKS / full LLD？ | 批准，进入 CP6 offline contract / schema checker implementation。 | 修改设计后重提；reject 回 CP3 approved。 | 推荐方案可执行且不越权；修改可更精细但延后；reject 暂停真实 smoke。 | 影响 run contract、evidence schema、tests 和后续 runtime 授权准备。 | 发现 contract / schema 缺口时重提 CP5。 |
| DQ-CP5-CR099-02 | architecture | run contract 是否作为真实 smoke 唯一入口？ | 是；真实 run 只能从 CP5 approved run contract + 用户逐 run 授权进入。 | 用户手工 Windows-side 执行后回传 evidence；只做离线 reverify。 | 推荐方案可审计；手工方案更保守；离线方案不能证明真实 runner runtime。 | 影响 CP6 / CP7 证据形态。 | Codex 无法安全代理 runtime 时切换手工方案。 |
| DQ-CP5-CR099-03 | security | env / HMAC / evidence 是否按当前 schema 控制？ | 确认；contract 只保存 env ref，不保存 secret；evidence 只保存 redacted summary。 | 保存完整 CLI 输出；隐式读取默认 env。 | 推荐方案最小泄露面；备选会扩大 secret / 账户原文风险。 | 安全风险 HIGH。 | 发现 raw leak 或 forbidden counter 非 0 时 blocked。 |
| DQ-CP5-CR099-04 | runtime_authorization | CP5 approve 是否授权真实 runner smoke？ | 不授权；只授权 CP6 离线 contract / schema implementation。 | 同时授权一次真实 runner smoke；永久禁止 runtime。 | 推荐方案符合门禁；直接授权会越过逐 run授权；永久禁止无法最终验证。 | 防止 CP5 被误读为运行许可。 | CP6 后需要真实运行时再逐 run 授权。 |
| DQ-CP5-CR099-05 | follow_up_tracking | 非空 / 交易日 / NAS / order-write 是否独立？ | 独立；CR099 当前只做 runner real readonly smoke gate。 | 合并非空复测；合并 NAS；合并 order-write。 | 推荐方案保持单一目标；合并会扩大权限和风险。 | 当前可能仍保留空持仓 / 非交易日剩余风险。 | 用户明确选择时另起 follow-up CR。 |

### 用户需决策事项

| 字段 | 内容 |
|---|---|
| 推荐决策 | `approve` 接受 DQ-CP5-CR099-01..05，进入 CP6 offline contract / schema implementation |
| 备选方案 | `修改: <具体修改点>` 后重出 CP5；或 `reject` 回退到 CP3 approved / 暂停 |
| 影响维度 | 实现范围、run contract、secret 边界、runtime 授权、follow-up 分流 |
| 风险与回退 | CP5 通过仍不证明真实 runner runtime；reject 时回退到 CP3 approved |
| 用户需决策事项 | 是否批准当前设计材料并保持 runtime 不授权 |

## 不授权范围

本 CP5 即使通过，也不授权：

- 读取 `/home/hyde/.quant-lab/runtime/qmt/cr097/client.env.template` 或任何 HMAC secret。
- 读取 Windows `.env`、QMT 账号原文、资金、持仓明细原文、委托、成交或原始日志。
- 启动 Windows gateway、执行 runner runtime 或查询账户。
- NAS package exchange / pull / publish。
- submit / cancel、buy / sell、simulation / live。
- provider fetch、lake write、catalog publish。
- 自动启动 CR089 或恢复 CR020 gateway route。

## Entry Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| CP3 approved | approved | `process/checkpoints/CP3-CR099-RUNNER-REAL-READONLY-SMOKE-HLD-REVIEW.md` | 用户已同意 |
| Feature Design ready | approved | `docs/features/cr099-runner-real-readonly-smoke/DESIGN.md` | 用户已同意 |
| LLD ready | approved | `process/stories/CR099-RUNNER-REAL-READONLY-SMOKE-LLD.md` | 用户已同意 |
| TEST-PLAN ready | approved | `docs/features/cr099-runner-real-readonly-smoke/TEST-PLAN.md` | 用户已同意 |
| CP5 auto precheck ready | approved | `process/checks/CP5-CR099-RUNNER-REAL-READONLY-SMOKE-READINESS.md` | PASS |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | 实现范围是否可接受 | approved | `DQ-CP5-CR099-01` | 接受进入 CP6 offline implementation |
| 2 | run contract 唯一入口是否可接受 | approved | `DQ-CP5-CR099-02` | 真实 run 仍需逐 run 授权 |
| 3 | env / secret / evidence 边界是否可接受 | approved | `DQ-CP5-CR099-03` | redacted evidence only |
| 4 | CP5 不授权 runtime 是否清楚 | approved | `DQ-CP5-CR099-04` | CP5 不授权真实 runner smoke |
| 5 | follow-up 分流是否清楚 | approved | `DQ-CP5-CR099-05` | 非空复测 / NAS / order-write 独立 |

## Exit Criteria

| 用户回复 | 处理 |
|---|---|
| `approve` | 接受 5 项推荐方案，进入 CP6 offline contract / schema implementation；仍不授权 runtime。 |
| `修改: <具体修改点>` | 按修改点修订 LLD / TEST-PLAN / CP5 材料并重新发起确认。 |
| `reject` | 回退到 CP3 approved / CP5 redesign 或暂停 CR099。 |

## Deliverables

- `docs/features/cr099-runner-real-readonly-smoke/DESIGN.md`
- `docs/features/cr099-runner-real-readonly-smoke/TEST-PLAN.md`
- `docs/features/cr099-runner-real-readonly-smoke/TASKS.md`
- `process/stories/CR099-RUNNER-REAL-READONLY-SMOKE-LLD.md`
- `process/context/CP5-CR099-RUNNER-REAL-READONLY-SMOKE-CONTEXT.yaml`
- `process/checks/CP5-CR099-RUNNER-REAL-READONLY-SMOKE-READINESS.md`

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-06-19T16:13:23+08:00
- 用户回复：同意
- 接受的决策项：`DQ-CP5-CR099-01..05`
- 修改意见：无
- 不授权边界：CP5 approve 只允许进入 CP6 offline contract / schema checker implementation；不授权读取 HMAC secret、Windows `.env`、启动 gateway、执行 runner runtime、读取账户原文、访问 NAS、交易写、simulation/live、provider/lake/publish、CR089 auto-start 或 CR020 gateway route restore。
