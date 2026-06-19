---
checkpoint_id: "CP3"
checkpoint_name: "CR099 Runner Real Readonly Smoke HLD Review"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-06-19T15:54:07+08:00"
reviewed_by: "user"
reviewed_at: "2026-06-19T16:04:52+08:00"
auto_check_result: "process/checks/CP3-CR099-RUNNER-REAL-READONLY-SMOKE-HLD-PRECHECK.md"
target:
  phase: "solution-design"
  story_id: "CR099-RUNNER-REAL-READONLY-SMOKE"
  artifacts:
    - "docs/qmt/CR099-QMT-RUNNER-REAL-READONLY-SMOKE-HLD.md"
    - "process/context/CP3-CR099-RUNNER-REAL-READONLY-SMOKE-CONTEXT.yaml"
---

# CP3 CR099 Runner Real Readonly Smoke HLD Review 人工审查

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP3-CR099-RUNNER-REAL-READONLY-SMOKE-HLD-PRECHECK.md` | PASS | 0 | 推荐进入 CP3 人工审查；当前不授权 runtime 执行 |

## Decision Brief

CP2 已由用户回复“同意”批准 CR099 范围与不授权边界。CP3 的目标是确认 HLD：采用 CR098 runner adapter + CR097 gateway pattern，并新增 CR099 run contract / `authorization_ref` / redacted evidence schema。CP3 approve 只表示接受架构方向，不授权读取 secret/env、启动 gateway、执行 runner runtime 或查询账户。

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP3-CR099-RUNNER-REAL-READONLY-SMOKE-CONTEXT.yaml` |
| capsule 状态 | `ready` |
| read_profile | `minimal` |
| must_read | HLD + 本 CP3 checkpoint |
| 默认读取策略 | 先读 capsule；仅在字段缺失、冲突、人工审计或深度评审时读取完整文档 |
| 默认禁止读取 | HMAC secret、Windows `.env`、账户原文、NAS、provider/lake/publish |
| 全文档读取扩展 | 0 次 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| CP2 checkpoint | `process/checkpoints/CP2-CR099-RUNNER-REAL-READONLY-SMOKE-SCOPE-REVIEW.md` | scanned | 5 | 5 | CP2 决策转为 CP3 架构决策 |
| HLD | `docs/qmt/CR099-QMT-RUNNER-REAL-READONLY-SMOKE-HLD.md` | scanned | 5 | 5 | architecture / runtime / security / implementation / follow-up |
| AGA discussion | `process/discussions/CP3-CR099-HLD-DISCUSSION-LOG.md` | scanned | 4 | 4 | 4 个灰区全部映射 |
| CP3 precheck | `process/checks/CP3-CR099-RUNNER-REAL-READONLY-SMOKE-HLD-PRECHECK.md` | scanned | 1 | 0 | 自动预检 PASS，无新增阻断项 |
| runtime authorization | N/A | n/a | 1 | 1 | 必须保留为不授权项，CP5 后另行逐 run 授权 |

### 待人工决策清单

如果你回复 `approve`，表示你接受以下 5 项推荐方案；不表示授权后面的禁止操作。

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP3-CR099-01 | architecture | 是否接受 CR099 推荐架构？ | 接受 CR098 runner adapter + CR097 gateway pattern + CR099 run contract / `authorization_ref` / evidence schema。 | A: 用户手工 Windows-side 执行并回传脱敏 evidence；B: 只做离线 reverify。 | 推荐方案最能证明 runner integration；手工方案安全但自动化低；离线方案不能关闭真实 runtime 风险。 | 影响 CP5 run contract 和 CP6/CP7 验证对象。 | 若 Codex 无法安全代理运行，切换 A；若用户暂不授权 runtime，切换 B。 |
| DQ-CP3-CR099-02 | runtime_authorization | CP3 approve 是否授权真实 run？ | 不授权；真实 run 必须 CP5 后逐 run 授权。 | A: CP3 后立即运行；B: 永久禁止 runtime。 | 推荐方案保持设计批准和运行授权分离；立即运行越过 CP5。 | 当前只允许写设计，不触碰 secret/env/gateway/account。 | 用户要运行时，先完成 CP5，再给单次授权。 |
| DQ-CP3-CR099-03 | security | env / HMAC / evidence 如何控制？ | run contract 只保存授权引用；evidence 只保存 redacted summary，forbidden counters 为 0。 | A: 保存完整 CLI 输出；B: 隐式读取默认 env。 | 推荐方案最小化泄露面；备选会泄露 secret 或账户原文。 | 影响 CP5 schema 和 redaction gate。 | 发现 raw leak 或 forbidden counter 非 0 时立即 blocked。 |
| DQ-CP3-CR099-04 | implementation | CP5 应实现 / 固化什么？ | CP5 只固化 run manifest、evidence schema、preflight checklist、blocked-path checks 和测试计划；不执行 runtime。 | A: 直接执行 runner smoke；B: 只写文档不做验证设计。 | 推荐方案可验证且不越权；直接执行风险高；只写文档缺可执行门禁。 | 影响后续 CP5 / CP6 / CP7 工作量。 | 若用户不接受 CP5，CR099 保持 blocked-pending-design。 |
| DQ-CP3-CR099-05 | follow_up_tracking | 非空 / 交易日 / NAS / order-write 是否并入？ | 不并入 CR099，保持独立后续候选。 | A: 合并非空 / 交易日复测；B: 合并 NAS / order-write。 | 推荐方案保持目标单一；合并会扩大权限和等待条件。 | CR099 关闭时可能仍保留 READY_WITH_RISK。 | 用户明确要求扩展范围时回到 CP2 或另起 CR。 |

### 用户需决策事项

| 字段 | 内容 |
|---|---|
| 推荐决策 | `approve` 接受 `DQ-CP3-CR099-01..05`，进入 CP5 design readiness |
| 备选方案 | `修改: <具体修改点>` 后重出 CP3；或 `reject` 暂停 / 取消 CR099 |
| 影响维度 | 架构基线、runtime 授权分离、evidence 安全、CP5 实施对象、后续事项分流 |
| 风险与回退 | 风险为 HIGH 但当前只做设计门禁；真实运行风险在 CP5 后逐 run 授权 |
| 用户需决策事项 | 是否接受 CR099 HLD 推荐架构和不授权边界 |

## 不授权范围

- 不授权读取 HMAC secret、`/home/hyde/.quant-lab/runtime/qmt/cr097/client.env.template` 或 Windows `.env`。
- 不授权启动 Windows gateway、执行 runner runtime 或查询账户。
- 不授权读取 QMT 账号原文、资金、持仓明细原文、证券代码、委托、成交或原始日志。
- 不授权 NAS package exchange / pull / publish。
- 不授权 submit / cancel、buy / sell、simulation / live。
- 不授权 provider fetch、lake write、catalog publish。
- 不授权真实 release execution / publish。
- 不自动启动 CR089 或恢复 CR020 gateway route。

## Entry Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| CP2 已批准 | approved | `process/checkpoints/CP2-CR099-RUNNER-REAL-READONLY-SMOKE-SCOPE-REVIEW.md` | 用户回复“同意” |
| HLD 已生成 | approved | `docs/qmt/CR099-QMT-RUNNER-REAL-READONLY-SMOKE-HLD.md` | 用户回复“同意”批准 HLD |
| CP3 自动预检 PASS | approved | `process/checks/CP3-CR099-RUNNER-REAL-READONLY-SMOKE-HLD-PRECHECK.md` | 无阻断项 |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | 架构方案是否可接受 | approved | `DQ-CP3-CR099-01` | 接受推荐方案 A |
| 2 | runtime 不授权是否清楚 | approved | `DQ-CP3-CR099-02` | CP3 不授权 secret / env / gateway / runner run |
| 3 | security / evidence 边界是否清楚 | approved | `DQ-CP3-CR099-03` | redacted evidence only |
| 4 | CP5 实施对象是否清楚 | approved | `DQ-CP3-CR099-04` | 只设计 run manifest / schema / preflight / tests |
| 5 | 后续分流是否清楚 | approved | `DQ-CP3-CR099-05` | 非空 / 交易日 / NAS / order-write 独立 |

## Exit Criteria

| 用户回复 | 处理 |
|---|---|
| `approve` | 接受 CP3 HLD，进入 CP5 design readiness；仍不授权 runtime。 |
| `修改: <具体修改点>` | 按修改点修订 HLD / CP3 材料并重新发起确认。 |
| `reject` | 暂停 / 取消 CR099，或回退到 CP2 重新定义范围。 |

## Deliverables

- `docs/qmt/CR099-QMT-RUNNER-REAL-READONLY-SMOKE-HLD.md`
- `process/discussions/CP3-CR099-HLD-DISCUSSION-LOG.md`
- `process/checks/CP3-CR099-DISCUSSION-CHECKPOINT.json`
- `process/context/CP3-CR099-RUNNER-REAL-READONLY-SMOKE-CONTEXT.yaml`
- `process/checks/CP3-CR099-RUNNER-REAL-READONLY-SMOKE-HLD-PRECHECK.md`
- `process/checkpoints/CP3-CR099-RUNNER-REAL-READONLY-SMOKE-HLD-REVIEW.md`

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-06-19T16:04:52+08:00
- 用户回复：同意
- 接受的决策项：`DQ-CP3-CR099-01..05`
- 修改意见：无
- 不授权边界：CP3 approve 只表示接受 CR099 HLD 架构方向；不授权读取 HMAC secret、Windows `.env`、启动 gateway、执行 runner runtime、读取账户原文、访问 NAS、交易写、simulation/live、provider/lake/publish、CR089 auto-start 或 CR020 gateway route restore。
