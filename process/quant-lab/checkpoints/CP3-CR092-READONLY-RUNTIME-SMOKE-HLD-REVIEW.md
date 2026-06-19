---
checkpoint_id: "CP3"
checkpoint_name: "CR092 Readonly Runtime Smoke HLD Review"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-06-18T16:17:35+08:00"
reviewed_by: "user"
reviewed_at: "2026-06-18T16:45:00+08:00"
auto_check_result: "process/checks/CP3-CR092-HLD-CONSISTENCY.md"
target:
  phase: "solution-design"
  story_id: "CR092"
  artifacts:
    - "docs/qmt/CR092-REAL-QMT-READONLY-RUNTIME-SMOKE-HLD.md"
    - "process/context/CP3-CR092-DESIGN-CONTEXT.yaml"
    - "process/discussions/CP3-CR092-HLD-DISCUSSION-LOG.md"
---

# CP3 CR092 Readonly Runtime Smoke HLD Review 人工审查

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP3-CR092-HLD-CONSISTENCY.md` | PASS | 0 | 推荐批准 HLD，进入 CP5 LLD / TEST-PLAN / readiness 设计。 |

## Decision Brief

CR092 CP3 HLD 推荐采用“人工逐 run 只读 smoke 设计门禁”：先冻结真实只读 smoke 的授权、manual execution、脱敏 evidence 和失败回退合同；不在 CP3 阶段实现或执行真实 runtime。

推荐 CP3 approve，允许进入 CP5 LLD / TEST-PLAN / readiness。CP3 approve 不授权 QMT / MiniQMT / XtQuant / gateway / runner startup，不授权 NAS、凭据、真实账户原文、submit/cancel、simulation/live、provider/lake/publish，也不自动启动 CR089 或恢复 CR020。用户已明确说明当前为模拟账户测试，可读取其显式提供的模拟账户测试证据；该授权不包含启动 runtime、读取凭据、读取真实账户或泛读未指定日志。

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP3-CR092-DESIGN-CONTEXT.yaml` |
| capsule 状态 | `ready-for-human-review` |
| read_profile | `compact` |
| 默认读取策略 | 先读 capsule；仅在缺失、冲突、字段不足、人工审计或深度评审时读取完整文档 |
| 全文档读取扩展 | 0 次；本轮读取 CP2、CR092、CR091 HLD 背景和 CP3 discussion |
| 缺失 / waived 理由 | N/A |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `STATE.md.human_gate_decisions.pending_human_decisions[]` | scanned | 0 | 0 | 本轮 CP3 决策在本 checkpoint 中新增。 |
| CP2 人工检查点 | `process/checkpoints/CP2-CR092-READONLY-RUNTIME-SMOKE-SCOPE-REVIEW.md` | scanned | 5 | 0 | 已由用户 approve，作为 CP3 输入。 |
| CP3 自动预检 | `process/checks/CP3-CR092-HLD-CONSISTENCY.md` | scanned | 1 | 1 | HLD 推荐方案进入 DQ。 |
| HLD | `docs/qmt/CR092-REAL-QMT-READONLY-RUNTIME-SMOKE-HLD.md` | scanned | 6 | 4 | 架构、安全、运行授权、follow-up 分流纳入 DQ。 |
| discussion log / checkpoint | `process/discussions/CP3-CR092-HLD-DISCUSSION-LOG.md` / `process/checks/CP3-CR092-DISCUSSION-CHECKPOINT.json` | scanned | 4 | 4 | AGA 与 advisor table 纳入 HLD。 |
| 用户显式选择题 | 当前会话 | scanned | 2 | 1 | 用户已批准 CP2，并修改 DQ-CP3-CR092-02：模拟账户测试信息允许读取；另询问 NAS 是否参与，已归入 DQ-CP3-CR092-04。 |

### 待人工决策清单

如果你回复 `approve`，表示你接受以下 4 项推荐方案；不表示授权后面的禁止操作。

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP3-CR092-01 | architecture | CR092 HLD 是否采用人工逐 run 只读 smoke 设计门禁？ | 采用方案 A，CP5 前只冻结授权、manual guide、evidence 合同。 | runner 内置真实 readonly transport；合并 CR089；暂停 CR092。 | 推荐方案权限最小；备选自动化更强但风险高或范围扩大。 | 真实 runtime readiness 仍未证明。 | 用户要求自动化或合并 CR089 时重做影响分析。 |
| DQ-CP3-CR092-02 | security | 是否允许 Codex 读取模拟账户测试证据？背景：用户说明当前在模拟账户上测试，并授权读取这些信息；该授权需要与凭据、真实账户、runtime 和 NAS 边界隔离。 | 接受修改后的安全边界：允许读取用户明确提供路径 / 内容的模拟账户测试证据；优先读取脱敏 summary，也可读取模拟账户的持仓 / 委托 / 成交 / 日志片段用于验证；禁止读取 `.env`、凭据、真实账户原文、真实资金 / 持仓 / 委托 / 成交、未指定原始日志或 NAS 路径。 | 仍坚持只读脱敏 summary；或扩大为允许真实账户原文和完整日志。 | 推荐方案贴合用户模拟账户测试场景，能提高可验证性；脱敏-only 更安全但可能不足以定位测试问题；真实账户 / 完整日志扩大权限过高。 | 需要用户给出明确文件路径或粘贴内容；若证据含真实账户、凭据、token、live 标识或 NAS 路径，必须停止读取并要求重新脱敏。 | 用户要求读取真实账户 / 凭据 / NAS / live 日志时回退到 security gate 或新 CR。 |
| DQ-CP3-CR092-03 | runtime_authorization | CP3 approve 是否授权真实运行？ | 不授权；只允许进入 CP5 LLD / TEST-PLAN / readiness。 | 同时授权一次只读 smoke；授权实现真实 transport。 | 推荐方案符合 CP2；备选需要独立逐 run 授权。 | 避免 CP3 设计通过被误读成 runtime approve。 | 用户明确授权 runtime 时另起 gate。 |
| DQ-CP3-CR092-04 | follow_up_tracking | NAS、order-write、ledger hygiene 如何处理？ | 继续保留 CR091-FU-02、FU-03、FU-04，CR092 不合并。 | 本轮合并 NAS；本轮合并 order-write；先暂停 CR092 修 ledger。 | 推荐方案不扩大范围；备选会引入新风险或打断主线。 | cr-tracking 旧账仍会非阻断存在。 | 用户启动对应候选时独立 CR。 |

### 用户需决策事项

| 字段 | 内容 |
|---|---|
| 推荐决策 | `approve` 接受 DQ-CP3-CR092-01..04，允许进入 CP5 LLD / TEST-PLAN / readiness |
| 备选方案 | `修改: <具体修改点>` 后重出 CP3；或 `reject` 回退到 CP2 / follow-up backlog |
| 影响维度 | 架构形态、安全证据、运行授权、follow-up 分流 |
| 优劣分析 | 推荐方案权限最小且可审计；备选方案自动化更强但容易越过当前授权边界 |
| 风险与回退 | CP3 通过仍不证明 runtime ready；reject 时回退到 CP2 approved / CP3 redesign |
| 用户需决策事项 | DQ-CP3-CR092-01 是否接受方案 A；DQ-CP3-CR092-02 是否接受“可读取用户明确提供的模拟账户测试证据，但不读取凭据 / 真实账户 / NAS / 未指定日志”；DQ-CP3-CR092-03 是否继续不授权真实运行；DQ-CP3-CR092-04 是否保持 NAS / order-write / ledger hygiene 独立分流 |

## 不授权范围

本 CP3 即使通过，也不授权：

- QMT / MiniQMT / XtQuant / gateway / runner runtime 启动、连接、安装或运行。
- NAS 访问、列取、读取、复制、拉取、写入、发布或删除。
- 读取 `.env`、凭据、真实账号、真实账户、真实资金 / 持仓 / 委托 / 成交、未指定日志原文；但允许读取用户明确提供的模拟账户测试证据。
- submit / cancel、buy / sell。
- simulation / live。
- provider fetch、lake write、catalog publish。
- 自动启动 CR089。
- 恢复 CR020 用户删除的 gateway 路线。

## Entry Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| CP2 approved | 待审查 | `process/checkpoints/CP2-CR092-READONLY-RUNTIME-SMOKE-SCOPE-REVIEW.md` |  |
| HLD ready | 待审查 | `docs/qmt/CR092-REAL-QMT-READONLY-RUNTIME-SMOKE-HLD.md` |  |
| CP3 auto precheck ready | 待审查 | `process/checks/CP3-CR092-HLD-CONSISTENCY.md` |  |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | HLD 推荐方案清晰 | 待审查 | HLD §3 / §4 |  |
| 2 | Architecture Gray Areas 已前置处理 | 待审查 | HLD §2 / discussion log |  |
| 3 | Use Case Traceability 和场景模拟通过 | 待审查 | HLD §6 / §7 |  |
| 4 | 不授权项已列出 | 待审查 | 本文件“不授权范围” |  |
| 5 | 后续分流不扩大范围 | 待审查 | DQ-CP3-CR092-04 |  |

## Exit Criteria

| 用户回复 | 处理 |
|---|---|
| `approve` | 接受 4 项推荐方案，CR092 进入 CP5 LLD / TEST-PLAN / readiness。 |
| `修改: <具体修改点>` | 按修改点修订 HLD 和 CP3 材料并重新发起确认。 |
| `reject` | 回退到 CP2 approved / CP3 redesign 或 CR091 follow-up backlog。 |

## Deliverables

- `docs/qmt/CR092-REAL-QMT-READONLY-RUNTIME-SMOKE-HLD.md`
- `process/context/CP3-CR092-DESIGN-CONTEXT.yaml`
- `process/discussions/CP3-CR092-HLD-DISCUSSION-LOG.md`
- `process/checks/CP3-CR092-DISCUSSION-CHECKPOINT.json`
- `process/checks/CP3-CR092-HLD-CONSISTENCY.md`
- `process/checks/CP3-CR092-HUMAN-GATE-LAUNCH-MESSAGE.md`

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-06-18T16:45:00+08:00
- 用户回复：接受修订后的 DQ-CP3-CR092-01..04，进入 CP5 LLD / TEST-PLAN / readiness。
- 接受决策项：
  - DQ-CP3-CR092-01：采用人工逐 run 只读 smoke 设计门禁。
  - DQ-CP3-CR092-02：允许读取用户明确提供的模拟账户测试证据；不读取凭据、真实账户、NAS 或未指定日志。
  - DQ-CP3-CR092-03：CP3 approve 不授权真实运行，只允许进入 CP5 LLD / TEST-PLAN / readiness。
  - DQ-CP3-CR092-04：NAS、order-write、ledger hygiene 保持 CR091-FU-02..04 独立分流。
- 修改意见：无。
- 风险接受项：CP3 通过仍不证明 runtime ready；真实运行、NAS、凭据、真实账户、submit/cancel、simulation/live、provider/lake/publish 仍未授权。
