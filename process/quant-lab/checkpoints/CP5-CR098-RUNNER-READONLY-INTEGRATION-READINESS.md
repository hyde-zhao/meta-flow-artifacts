---
checkpoint_id: "CP5"
checkpoint_name: "CR098 Runner Readonly Integration Readiness"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-06-19T12:07:32+08:00"
reviewed_by: "user"
reviewed_at: "2026-06-19T12:18:02+08:00"
auto_check_result: "process/checks/CP5-CR098-RUNNER-READONLY-INTEGRATION-READINESS.md"
target:
  phase: "story-planning"
  story_id: "CR098-RUNNER-READONLY-INTEGRATION"
  artifacts:
    - "docs/features/cr098-runner-readonly-integration/DESIGN.md"
    - "docs/features/cr098-runner-readonly-integration/TEST-PLAN.md"
    - "docs/features/cr098-runner-readonly-integration/TASKS.md"
    - "process/stories/CR098-RUNNER-READONLY-INTEGRATION-LLD.md"
    - "process/context/CP5-CR098-RUNNER-READONLY-INTEGRATION-CONTEXT.yaml"
---

# CP5 CR098 Runner Readonly Integration Readiness 人工审查

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP5-CR098-RUNNER-READONLY-INTEGRATION-READINESS.md` | PASS | 0 | 推荐批准 LLD / Feature Design / TEST-PLAN / TASKS，进入 CP6 offline implementation；不授权 runtime。 |

## Decision Brief

CR098 CP5 推荐批准 runner readonly integration 的实现设计。批准后只允许修改 runner adapter、evidence 和离线 / fixture tests；仍不授权读取 HMAC secret、Windows `.env`、启动 gateway、执行 runner runtime、读取账户原文、访问 NAS、交易写、simulation/live 或 provider/lake/publish。

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP5-CR098-RUNNER-READONLY-INTEGRATION-CONTEXT.yaml` |
| capsule 状态 | `ready-for-human-review` |
| read_profile | `compact` |
| 默认读取策略 | 先读 capsule；仅在缺失、冲突、字段不足、人工审计或深度评审时读取完整文档 |
| 全文档读取扩展 | 本轮读取 HLD、当前 runner / qmt client 代码、Feature Design、LLD 和 TEST-PLAN |
| 缺失 / waived 理由 | N/A |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| CP3 checkpoint | `process/checkpoints/CP3-CR098-QMT-RUNNER-READONLY-INTEGRATION-HLD-REVIEW.md` | scanned | 5 | 0 | 已 approved，作为 CP5 输入 |
| HLD | `docs/qmt/CR098-QMT-RUNNER-READONLY-GATEWAY-INTEGRATION-HLD.md` | scanned | 5 | 5 | 文件 owner、env、evidence、runtime、follow-up 纳入 |
| Feature Design | `docs/features/cr098-runner-readonly-integration/DESIGN.md` | scanned | 6 | 5 | adapter / evidence / failure path 纳入 |
| LLD | `process/stories/CR098-RUNNER-READONLY-INTEGRATION-LLD.md` | scanned | 8 | 5 | 接口、数据映射、测试和 gotchas 纳入 |
| TEST-PLAN | `docs/features/cr098-runner-readonly-integration/TEST-PLAN.md` | scanned | 6 | 5 | 必测场景和拒收规则纳入 |
| STATE pending queue | `process/STATE.md` | scanned | 0 | 0 | 无其他未回答阻断项 |

### 待人工决策清单

如果你回复 `approve`，表示你接受以下 5 项推荐方案；不表示授权后面的禁止操作。

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP5-CR098-01 | implementation | 是否批准当前 LLD / Feature Design / TEST-PLAN / TASKS？ | 批准，进入 CP6 offline implementation。 | 修改设计后重提；reject 回 CP3 approved。 | 推荐方案范围小且可测试；修改可更精细但延后；reject 暂停集成。 | 影响 runner adapter、evidence、tests。 | 发现接口缺口时重提 CP5。 |
| DQ-CP5-CR098-02 | architecture | real adapter 是否只通过 injected `QmtClient` / transport 接入？ | 是；runner 不 import XtQuant，不启动 gateway。 | runner 直接 import XtQuant；独立 CLI 绕过 runner。 | 推荐方案保持 runner 集成和平台隔离；备选风险高或证明弱。 | 影响 `readonly_gateway.py` owner。 | 如果 injection 无法满足，另起 HLD 修订。 |
| DQ-CP5-CR098-03 | security | 是否确认 env / secret 仍按 explicit per-run authorization？ | 确认；CP6 代码不得隐式读取 env，CP7 运行另行授权。 | 固定读取 `.quant-lab` env；读取 Windows `.env`。 | 推荐方案最小暴露；备选会扩大 secret 面。 | secret 泄露风险 HIGH。 | 自动 secret 管理需另起安全 CR。 |
| DQ-CP5-CR098-04 | runtime_authorization | CP5 approve 是否授权真实 runner runtime smoke？ | 不授权；只授权离线实现和 fixture tests。 | 同时授权一次 runner smoke；永久禁止 runtime。 | 推荐方案符合门禁；直接授权过早；永久禁止无法最终验证。 | 防止 CP5 被误读为运行许可。 | CP7 需要真实运行时再逐 run 授权。 |
| DQ-CP5-CR098-05 | follow_up_tracking | 非空/交易日复测、NAS、order-write 是否独立？ | 独立；CR098 当前只做 runner readonly integration。 | 合并非空复测；合并 NAS；合并 order-write。 | 推荐方案闭环小；合并会扩大权限和风险。 | 当前模拟新账户空持仓、非交易日仍是剩余风险。 | 用户明确选择时另起 follow-up CR。 |

### 用户需决策事项

| 字段 | 内容 |
|---|---|
| 推荐决策 | `approve` 接受 DQ-CP5-CR098-01..05，进入 CP6 offline implementation |
| 备选方案 | `修改: <具体修改点>` 后重出 CP5；或 `reject` 回退到 CP3 approved / 暂停 |
| 影响维度 | 实现范围、架构依赖、secret 边界、runtime 授权、follow-up 分流 |
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
| CP3 approved | approved | `process/checkpoints/CP3-CR098-QMT-RUNNER-READONLY-INTEGRATION-HLD-REVIEW.md` | 用户已同意 |
| Feature Design ready | approved | `docs/features/cr098-runner-readonly-integration/DESIGN.md` | 用户已同意 |
| LLD ready | approved | `process/stories/CR098-RUNNER-READONLY-INTEGRATION-LLD.md` | 用户已同意 |
| TEST-PLAN ready | approved | `docs/features/cr098-runner-readonly-integration/TEST-PLAN.md` | 用户已同意 |
| CP5 auto precheck ready | approved | `process/checks/CP5-CR098-RUNNER-READONLY-INTEGRATION-READINESS.md` | 用户已同意 |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | 实现范围是否可接受 | approved | `DQ-CP5-CR098-01` | 接受进入 CP6 offline implementation |
| 2 | injected QmtClient / transport 路线是否可接受 | approved | `DQ-CP5-CR098-02` | 接受 runner 不直接 import runtime |
| 3 | env / secret 显式授权边界是否可接受 | approved | `DQ-CP5-CR098-03` | 接受 per-run authorization |
| 4 | CP5 不授权 runtime 是否清楚 | approved | `DQ-CP5-CR098-04` | CP5 不授权真实 runner runtime |
| 5 | follow-up 分流是否清楚 | approved | `DQ-CP5-CR098-05` | 非空复测 / NAS / order-write 独立 |

## Exit Criteria

| 用户回复 | 处理 |
|---|---|
| `approve` | 接受 5 项推荐方案，进入 CP6 offline implementation；仍不授权 runtime。 |
| `修改: <具体修改点>` | 按修改点修订 LLD / TEST-PLAN / CP5 材料并重新发起确认。 |
| `reject` | 回退到 CP3 approved / CP5 redesign 或暂停 CR098。 |

## Deliverables

- `docs/features/cr098-runner-readonly-integration/DESIGN.md`
- `docs/features/cr098-runner-readonly-integration/TEST-PLAN.md`
- `docs/features/cr098-runner-readonly-integration/TASKS.md`
- `process/stories/CR098-RUNNER-READONLY-INTEGRATION-LLD.md`
- `process/context/CP5-CR098-RUNNER-READONLY-INTEGRATION-CONTEXT.yaml`
- `process/checks/CP5-CR098-RUNNER-READONLY-INTEGRATION-READINESS.md`

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-06-19T12:18:02+08:00
- 用户回复：同意
- 接受的决策项：`DQ-CP5-CR098-01..05`
- 修改意见：无
- 不授权边界：CP5 approve 只允许进入 CP6 offline implementation；不授权读取 HMAC secret、Windows `.env`、启动 gateway、执行 runner runtime、读取账户原文、访问 NAS、交易写、simulation/live、provider/lake/publish、CR089 auto-start 或 CR020 gateway route restore。
