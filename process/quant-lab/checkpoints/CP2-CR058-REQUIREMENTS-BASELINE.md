---
checkpoint_id: "CP2"
checkpoint_name: "CR058 Requirements Baseline Human Gate"
type: "manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-06-14T13:59:00+08:00"
auto_check: "process/checks/CP2-CR058-REQUIREMENTS-BASELINE.md"
launch_message: "process/checks/CP2-CR058-HUMAN-GATE-LAUNCH-MESSAGE.md"
auto_final_authorization: false
---

# CP2 CR058 Requirements Baseline 人工检查点

## 自动预检摘要

| 项目 | 结论 |
|---|---|
| 自动预检 | PASS |
| 变更来源 | `FU-CR053-01` from CR053 follow-up tracking |
| 推荐决策 | approve CR058 requirements baseline；不授权真实执行。 |
| 自动终验授权 | false |

## Entry Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| CR053 close gate 已通过 | PASS | `process/checkpoints/CP8-CR053-DELIVERY-READINESS.md` | CR053 只关闭静态 inventory / dry-run。 |
| CR058 formal CR 已创建 | PASS | `process/changes/CR-058-REPO-LOCAL-MECHANICAL-MIGRATION-RELAYOUT-GATE-2026-06-14.md` | 来源、影响分析和不授权项已写入。 |
| CR046 paused 边界已识别 | PASS | `process/STATE.md` | 不恢复 CR046 CP7。 |

## Checklist

| # | 检查项 | 推荐结果 | 用户审查 |
|---|---|---|---|
| 1 | 是否确认从 `FU-CR053-01` 创建正式 CR058 | accept | 通过 |
| 2 | 是否确认 CR058 当前只做 repo-local relayout gate 设计 | accept | 通过 |
| 3 | 是否确认候选范围采用 allowlist-first | accept | 通过 |
| 4 | 是否确认历史审计引用默认 preserve-audit | accept | 通过 |
| 5 | 是否确认 CR058 不授权真实运行 / 迁移 / 凭据 / git remote | accept | 通过 |
| 6 | 是否确认 CR058 不恢复 CR046 | accept | 通过 |

## Exit Criteria

| 条目 | 通过条件 | 状态 |
|---|---|---|
| 用户回复 `approve` | 接受 Decision Brief 内全部推荐方案，且不授权列明禁止操作。 | PASS |
| 用户回复 `修改: <具体修改点>` | host-orchestrator 按修改点更新相关 DQ / 文档并重新发起门禁。 | pending |
| 用户回复 `reject` | CR058 CP2 不通过，回到 candidate 或 blocked。 | pending |
| 人工审查结果回填 | 本文件“人工审查结果”填入用户选择、时间和处理结论。 | PASS |

## Deliverables

| 交付物 | 路径 | 状态 |
|---|---|---|
| CR058 formal CR | `process/changes/CR-058-REPO-LOCAL-MECHANICAL-MIGRATION-RELAYOUT-GATE-2026-06-14.md` | ready |
| CP2 Auto Check | `process/checks/CP2-CR058-REQUIREMENTS-BASELINE.md` | ready |
| Human Gate Launch Message | `process/checks/CP2-CR058-HUMAN-GATE-LAUNCH-MESSAGE.md` | ready |

## Decision Brief

### 推荐决策

推荐用户回复 `approve`，接受 CR058 requirements baseline：从 `FU-CR053-01` 创建正式 CR058，范围限定为 repo-local mechanical migration / relayout gate 设计，采用 allowlist-first、preserve-audit-by-default 和 no-op-by-default。此确认不授权真实文件移动、重命名、删除、批量路径替换、NAS、data lake、QMT / MiniQMT runtime、provider fetch、lake write、catalog publish、git push / tag / remote rename / history rewrite 或凭据读取。

### 备选方案

| 方案 | 内容 | 优点 | 代价 / 风险 | 切换条件 |
|---|---|---|---|---|
| 推荐方案 A | 启动 CR058 formal CR，仅做设计 gate。 | 可推进候选清单和门禁设计；不扩大授权。 | 后续真实执行仍需单独批准。 | 默认推荐。 |
| 备选方案 B | 保持 `FU-CR053-01` 为 candidate。 | 最保守。 | 无法推进 CR058 gate。 | 用户暂不想推进 relayout gate。 |
| 备选方案 C | 扩展到真实 move / rewrite。 | 能更快执行。 | 与用户当前禁止项冲突，高风险。 | 仅在用户另行明确授权真实动作和 rollback_ref 后讨论。 |

### Decision Collection Coverage

| 来源 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---|
| `process/changes/CR-053-FOLLOW-UP-TRACKING-2026-06-14.md` | scanned | 1 | 1 | `FU-CR053-01` 转正式 CR。 |
| `process/release/RELEASE-CONTEXT-CR053.yaml` | scanned | 3 | 2 | READY_WITH_RISK 和 not_authorized 作为 CR058 输入。 |
| `process/checkpoints/CP8-CR053-DELIVERY-READINESS.md` | scanned | 3 | 2 | 不授权项和后续候选进入决策。 |
| 当前用户指令 | scanned | 6 | 6 | 明确禁止真实动作、CR046 不恢复。 |
| 合计 | scanned | 13 | 6 | 收敛为 CP2 六项 DQ。 |

## 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP2-CR058-01 | scope | 是否确认从 `FU-CR053-01` 创建正式 CR058？ | 创建正式 CR058，来源追溯到 CR053 follow-up tracking；CR053 保持 closed-current-delivery。 | 保持 `FU-CR053-01` 为 candidate，不启动正式 CR。 | 推荐方案可进入门禁和设计；备选更保守但无法推进 relayout gate。 | 创建 CR058 不代表授权执行迁移。 | 用户只想保留候选时切换备选。 |
| DQ-CP2-CR058-02 | scope | CR058 当前范围是否限制为 repo-local mechanical migration / relayout gate 设计？ | 限定为候选清单、门禁、回滚、preserve-audit 设计；不执行真实迁移。 | 扩展到真实 move / rename / delete；不推荐。 | 推荐方案符合用户边界；扩展方案会越权。 | CP2/CP3/CP5 通过也只确认设计。 | 用户另行授权真实 move 且给出 rollback_ref 后再切换。 |
| DQ-CP2-CR058-03 | architecture | 候选范围如何定义？ | allowlist-first，仅 future-facing docs 和明确项目身份说明进入 candidate。 | repo-wide rewrite；不推荐。 | allowlist 降低破坏审计风险；repo-wide 覆盖大但风险高。 | 影响 README、USER-MANUAL、future-facing docs，不影响历史 `process/**`。 | 候选不足时新增人工候选并重过 CP5。 |
| DQ-CP2-CR058-04 | risk_acceptance | 是否接受历史审计引用默认 preserve-audit？ | 接受：`process/**`、checks、checkpoints、handoffs、历史 Story / LLD / DEV-LOG 默认保留。 | 对历史证据逐项重写；不推荐。 | 推荐方案保持审计链完整；逐项重写成本高。 | 历史文档仍保留 `local_backtest` 作为历史事实。 | 单项外发文档可列豁免并重过 CP5。 |
| DQ-CP2-CR058-05 | runtime_authorization | 是否确认 CR058 不授权 NAS、lake、runtime、provider、git remote 等真实操作？ | 确认不授权。 | 授权单项真实操作；必须独立 runtime gate。 | 推荐方案保持安全边界；备选会混淆设计和运行授权。 | 防止误触发 NAS / lake / QMT / git remote 风险。 | 用户指定操作、路径、窗口、回滚和安全条件后另起 CR。 |
| DQ-CP2-CR058-06 | implementation | CR058 是否恢复或推进 CR046？ | 不恢复 CR046；CR046 继续保持用户暂停的 CP6 PASS / ready-for-verification。 | 合并恢复 CR046；不推荐且与用户指令冲突。 | 推荐方案避免交易 runtime 误启动。 | CR058 不碰 QMT/MiniQMT、策略 runtime、账户查询或交易。 | 只有用户另行明确“恢复 CR046”时才单独恢复。 |

## 不授权项

如果你回复 approve，表示接受上述 6 项推荐方案；不表示授权以下操作：

| Item ID | 不授权操作 |
|---|---|
| NA-CP2-CR058-01 | 真实文件 move / rename / delete / bulk rewrite |
| NA-CP2-CR058-02 | NAS mount / scan / mkdir / copy / delete / migration |
| NA-CP2-CR058-03 | `MARKET_DATA_LAKE_ROOT` replacement、lake write、catalog publish |
| NA-CP2-CR058-04 | git push / tag / remote rename / history rewrite |
| NA-CP2-CR058-05 | 凭据、`.env`、token、password、cookie、session、private key 读取 |
| NA-CP2-CR058-06 | QMT / MiniQMT runtime、账户查询或交易动作 |
| NA-CP2-CR058-07 | provider fetch、untracked data bulk scan |
| NA-CP2-CR058-08 | 恢复或推进 CR046 CP7 |

## 人工审查结果

| 字段 | 值 |
|---|---|
| 用户回复 | 接受，继续推进 |
| 审查人 | user |
| 审查时间 | 2026-06-14T14:59:00+08:00 |
| 决策结果 | approved；接受 `DQ-CP2-CR058-01..06` 推荐方案；确认 CR058 正式启动但只做 repo-local relayout gate 设计，不授权真实迁移或 CR046 恢复。 |
