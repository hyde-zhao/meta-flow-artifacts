---
checkpoint_id: "CP2"
checkpoint_name: "CR099 Runner Real Readonly Smoke Scope Review"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-06-19T15:24:50+08:00"
reviewed_by: "user"
reviewed_at: "2026-06-19T15:54:07+08:00"
auto_check_result: "process/checks/CP2-CR099-RUNNER-REAL-READONLY-SMOKE-SCOPE-PRECHECK.md"
target:
  phase: "requirement-clarification"
  story_id: "CR099-RUNNER-REAL-READONLY-SMOKE"
  artifacts:
    - "process/changes/CR-099-QMT-RUNNER-REAL-READONLY-SMOKE-PER-RUN-AUTHORIZATION-2026-06-19.md"
    - "process/context/CP2-CR099-RUNNER-REAL-READONLY-SMOKE-CONTEXT.yaml"
---

# CP2 CR099 Runner Real Readonly Smoke Scope Review 人工审查

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP2-CR099-RUNNER-REAL-READONLY-SMOKE-SCOPE-PRECHECK.md` | PASS | 0 | 推荐进入 CP3 HLD；当前不授权 runtime 执行 |

## Decision Brief

CR099 的用户意图是推进 `CR098-FU-01`：在 CR098 已完成离线 runner readonly integration 的基础上，让 runner 真实消费 Windows gateway readonly path，并输出脱敏 evidence。CP2 approve 只确认范围、风险和后续设计方向；它不授权读取 HMAC secret、不读取 Windows `.env`、不启动 gateway、不执行 runner runtime、不查询账户。

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP2-CR099-RUNNER-REAL-READONLY-SMOKE-CONTEXT.yaml` |
| capsule 状态 | `ready` |
| read_profile | `minimal` |
| 默认读取策略 | 先读 capsule；仅在字段缺失、冲突、人工审计或深度评审时读取完整文档 |
| 全文档读取扩展 | 0 次 |
| 缺失 / waived 理由 | N/A |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `process/STATE.md` | scanned | 1 | 1 | next_action 推荐 `CR098-FU-01`，用户本轮明确推进 |
| Handoff | `process/context/CR098-CLOSURE-CONTEXT-RESET-HANDOFF-2026-06-19.md` | scanned | 3 | 3 | 推荐下一 CR、风险、不授权边界纳入 |
| Follow-up tracking | `process/changes/CR-098-FOLLOW-UP-TRACKING-2026-06-19.md` | scanned | 4 | 4 | `CR098-FU-01` 和备选候选纳入 |
| CR099 | `process/changes/CR-099-QMT-RUNNER-REAL-READONLY-SMOKE-PER-RUN-AUTHORIZATION-2026-06-19.md` | scanned | 5 | 5 | scope / runtime / security / follow-up 纳入 |
| 上游 CR097 / CR098 | CR097 / CR098 checkpoints | scanned | 2 | 2 | gateway baseline 和 runner baseline 纳入 |
| discussion log / checkpoint | N/A | n/a | 0 | 0 | 本轮用户直接选择明确 follow-up，暂无多轮灰区讨论 |

### 待人工决策清单

如果你回复 `approve`，表示你接受以下 5 项推荐方案；不表示授权后面的禁止操作。

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP2-CR099-01 | scope | 本轮 CR099 是否只覆盖 runner real readonly smoke？ | 是，仅覆盖 runner 调用 Windows gateway health / capabilities / query_positions_readonly 和 redacted evidence。 | A: 只做离线 reverify；B: 合并非空 / 交易日复测。 | 推荐方案能关闭 CR098 的真实 runner runtime 风险；离线 reverify 覆盖不足；合并复测可能等待运行条件。 | 将进入 CP3/CP5 设计和后续逐 run 授权。 | 若暂不做真实 run，选择 A；若要求更强覆盖，选择 B 并接受延期。 |
| DQ-CP2-CR099-02 | runtime_authorization | CP2 approve 是否授权读取 HMAC env、启动 gateway 或执行 runner？ | 不授权；CP2 只授权进入设计门禁。真实 run 必须在 CP5 后逐 run 授权。 | A: 立即授权一次 runner smoke；B: 永久禁止 runtime。 | 推荐方案避免设计批准被误读成运行授权；立即授权会越过 CP3/CP5。 | 当前只写设计和门禁，不触碰 secret / env / account。 | 需要执行时再发 runtime authorization。 |
| DQ-CP2-CR099-03 | security | evidence 和敏感文件如何处理？ | 只允许 `.quant-lab/evidence/qmt/cr099/redacted/` 脱敏输出；禁止 raw payload、账号、证券代码、数量、资金、委托、成交和日志原文。 | A: 允许保存完整 CLI 输出；B: 允许读取 Windows `.env` 自动配置。 | 推荐方案最小化敏感面；备选会带来 secret / 账户泄露风险。 | 需要 redaction gate 和 forbidden counters。 | 如发现原文泄露，立即 BLOCKED 并停止。 |
| DQ-CP2-CR099-04 | architecture | 后续 CP3/CP5 应采用哪条执行路径？ | 使用 CR098 runner adapter + CR097 gateway 模式，新增 CR099 run contract / authorization_ref / evidence schema。 | A: 用户手工执行后回传 evidence；B: 新写独立脚本绕过 runner。 | 推荐方案证明 runner integration；手工回传可作为 fallback；绕过 runner不能关闭目标风险。 | 影响 CP3 HLD、CP5 run contract 和 CP6/CP7 证据。 | 若 Codex 无法安全执行，切换为用户手工执行后回传 redacted evidence。 |
| DQ-CP2-CR099-05 | follow_up_tracking | NAS、order-write、非空持仓复测如何处理？ | 全部保持独立候选，不并入 CR099。 | A: 合并 NAS；B: 合并 order-write。 | 推荐方案保持单一目标；合并会扩大权限和风险。 | CR099 不解决包交换、下单或更广账户覆盖。 | 用户明确选择时另起独立 CR。 |

### 用户需决策事项

| 字段 | 内容 |
|---|---|
| 推荐决策 | `approve` 接受 `DQ-CP2-CR099-01..05`，进入 CP3 HLD |
| 备选方案 | `修改: <具体修改点>` 后重出 CP2；或 `reject` 取消 / 暂停 CR099 |
| 影响维度 | runner real readonly smoke 范围、runtime 授权边界、evidence 安全、执行路径、后续 CR 分流 |
| 风险与回退 | 风险为 HIGH 但当前只做设计门禁；真实运行风险在 CP5 后逐 run 授权 |
| 用户需决策事项 | 是否接受 CR099 的推荐范围和不授权边界 |

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
| 用户启动 CR099 | approved | 当前对话 / 用户回复“同意” | 用户明确要求推进 `CR098-FU-01`，并接受 CP2 推荐范围 |
| CR099 已创建 | approved | `process/changes/CR-099-QMT-RUNNER-REAL-READONLY-SMOKE-PER-RUN-AUTHORIZATION-2026-06-19.md` | 范围已确认 |
| CP2 context ready | approved | `process/context/CP2-CR099-RUNNER-REAL-READONLY-SMOKE-CONTEXT.yaml` | 最小上下文已生成 |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | 范围是否清晰 | approved | Decision Brief / 用户回复“同意” | 仅 runner real readonly smoke |
| 2 | runtime 不授权是否清楚 | approved | `DQ-CP2-CR099-02` / 用户回复“同意” | CP2 不授权 secret / gateway / runner run |
| 3 | evidence / secret 边界是否清楚 | approved | `DQ-CP2-CR099-03` / 用户回复“同意” | `.quant-lab` redacted evidence |
| 4 | 执行路径是否可接受 | approved | `DQ-CP2-CR099-04` / 用户回复“同意” | CP3/CP5 冻结 run contract |
| 5 | 后续分流是否清楚 | approved | `DQ-CP2-CR099-05` / 用户回复“同意” | NAS/order-write/非空复测独立 |

## Exit Criteria

| 用户回复 | 处理 |
|---|---|
| `approve` | 接受 CP2 范围，进入 CP3 HLD；仍不授权 runtime。 |
| `修改: <具体修改点>` | 按修改点修订 CP2 材料并重新发起确认。 |
| `reject` | 取消 / 暂停 CR099，回到无 active CR 或重新选择后续候选。 |

## Deliverables

- `process/changes/CR-099-QMT-RUNNER-REAL-READONLY-SMOKE-PER-RUN-AUTHORIZATION-2026-06-19.md`
- `process/changes/CR-098-FOLLOW-UP-TRACKING-2026-06-19.md`
- `process/context/CP2-CR099-RUNNER-REAL-READONLY-SMOKE-CONTEXT.yaml`
- `process/checks/CP2-CR099-RUNNER-REAL-READONLY-SMOKE-SCOPE-PRECHECK.md`
- `process/checkpoints/CP2-CR099-RUNNER-REAL-READONLY-SMOKE-SCOPE-REVIEW.md`

## 人工审查结果

- 结论：`approved`
- 审查人：`user`
- 审查时间：`2026-06-19T15:54:07+08:00`
- 用户回复：`同意`
- 接受的决策项：`DQ-CP2-CR099-01`、`DQ-CP2-CR099-02`、`DQ-CP2-CR099-03`、`DQ-CP2-CR099-04`、`DQ-CP2-CR099-05`
- 修改意见：无
- 不授权边界：
  - 不授权读取 HMAC secret、`/home/hyde/.quant-lab/runtime/qmt/cr097/client.env.template` 或 Windows `.env`。
  - 不授权启动 Windows gateway、执行 runner runtime 或查询账户。
  - 不授权读取 QMT 账号原文、资金、持仓明细原文、证券代码、委托、成交或原始日志。
  - 不授权 NAS package exchange / pull / publish。
  - 不授权 submit / cancel、buy / sell、simulation / live。
  - 不授权 provider fetch、lake write、catalog publish。
  - 不授权真实 release execution / publish。
  - 不自动启动 CR089 或恢复 CR020 gateway route。
