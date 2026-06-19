---
checkpoint_id: "CP5"
checkpoint_name: "CR093 Ledger Hygiene Readiness"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-06-18T17:55:00+08:00"
reviewed_by: "user"
reviewed_at: "2026-06-18T17:58:38+08:00"
auto_check_result: "process/checks/CP5-CR093-LEDGER-HYGIENE-READINESS.md"
target:
  phase: "story-planning"
  story_id: "CR093-LEDGER-HYGIENE-CONSISTENCY"
  artifacts:
    - "process/stories/CR093-LEDGER-HYGIENE-CONSISTENCY-LLD.md"
    - "docs/features/CR093-ledger-hygiene/DESIGN.md"
    - "docs/features/CR093-ledger-hygiene/TEST-PLAN.md"
    - "docs/features/CR093-ledger-hygiene/TASKS.md"
    - "process/context/CP5-CR093-LEDGER-HYGIENE-CONTEXT.yaml"
---

# CP5 CR093 Ledger Hygiene Readiness 人工审查

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP5-CR093-LEDGER-HYGIENE-READINESS.md` | PASS | 0 | 推荐批准 LLD / TEST-PLAN / TASKS，进入受限实施；不授权外部动作。 |

## Decision Brief

CR093 CP5 推荐批准单 Story full-lld、Feature DESIGN、TEST-PLAN 和 TASKS。批准后仅允许实现 checker 语义修正、测试和 CR019 最小台账规范化；仍不授权任何 runtime、NAS、凭据、账户、交易或数据发布动作。

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP5-CR093-LEDGER-HYGIENE-CONTEXT.yaml` |
| capsule 状态 | `ready-for-human-review` |
| read_profile | `compact` |
| 默认读取策略 | 先读 capsule；仅在缺失、冲突、字段不足、人工审计或深度评审时读取完整文档 |
| 全文档读取扩展 | 0 次 |
| 缺失 / waived 理由 | N/A |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `STATE.md.human_gate_decisions.pending_human_decisions[]` | scanned | 4 | 0 | CP3 队列已由用户同意，本轮新增 CP5 决策。 |
| CP5 auto check | `process/checks/CP5-CR093-LEDGER-HYGIENE-READINESS.md` | scanned | 5 | 5 | readiness / owner / test / security 纳入 DQ。 |
| LLD | `process/stories/CR093-LEDGER-HYGIENE-CONSISTENCY-LLD.md` | scanned | 6 | 5 | 文件范围、测试、回退纳入 DQ。 |
| Feature TEST-PLAN | `docs/features/CR093-ledger-hygiene/TEST-PLAN.md` | scanned | 4 | 2 | 验证命令与拒收条件纳入 DQ。 |
| TASKS | `docs/features/CR093-ledger-hygiene/TASKS.md` | scanned | 4 | 2 | 实施顺序与文件 owner 纳入 DQ。 |
| 用户显式选择题 | 当前会话 | scanned | 1 | 0 | 用户已同意 CP3。 |

### 待人工决策清单

如果你回复 `approve`，表示你接受以下 5 项推荐方案；不表示授权后面的禁止操作。

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP5-CR093-01 | implementation | 是否批准 CR093 单 Story full-lld 作为实现设计证据？ | 批准当前 LLD，允许进入受限实施。 | 修改 LLD 后重提；reject 回 CP3。 | 推荐方案已覆盖文件、接口、测试、回退；修改会延后但可提高精度。 | CP5 通过后可改 checker / tests / CR019 tracking。 | 发现文件 owner 或测试缺口时重提 CP5。 |
| DQ-CP5-CR093-02 | implementation | 是否按 TASKS 的串行顺序实施？ | 先写测试，再改 checker，再最小规范化 CR019 tracking。 | 先改 checker；先改旧台账。 | 推荐方案降低回归风险；先改实现容易缺少失败基线。 | 有利于证明目标缺陷被修复。 | 若测试构造成本过高，可先 py_compile + cr-tracking 验证。 |
| DQ-CP5-CR093-03 | security | CP5 approve 是否继续禁止外部动作？ | 继续禁止；只允许读写列明的仓库文本文件。 | 授权 NAS / runtime / 账户证据；授权清理 broader warnings。 | 推荐方案符合 ledger-only；备选扩大风险。 | 无外部副作用。 | 用户要求外部动作时另起 CR。 |
| DQ-CP5-CR093-04 | risk_acceptance | broader warning-only 项是否仍不作为本轮 DoD？ | 接受；DoD 只要求 CR019 / CR025 阻断项收敛。 | 本轮清所有 warnings；全部 suppress。 | 推荐方案聚焦目标；清全部会扩大范围。 | CP7 可能仍有 warning，但不阻断当前目标。 | 用户要求 warning 清零时另起后续 CR。 |
| DQ-CP5-CR093-05 | implementation | CP5 approve 后允许的实施范围是什么？ | 只允许修改 `scripts/check_cr_tracking_consistency.py`、新增 `tests/test_cr093_cr_tracking_consistency.py`、最小修改 CR019 tracking 和必要 process 证据。 | 仅改 checker；仅改文档；全量重写 checker。 | 推荐方案覆盖根因且可验证；单边改动容易残留问题。 | 进入 CP6 后以该范围为文件 owner。 | 超出文件范围必须回 CP5 或新 CR。 |

### 用户需决策事项

| 字段 | 内容 |
|---|---|
| 推荐决策 | `approve` 接受 `DQ-CP5-CR093-01`、`DQ-CP5-CR093-02`、`DQ-CP5-CR093-03`、`DQ-CP5-CR093-04`、`DQ-CP5-CR093-05`，允许 CR093 进入受限实施 |
| 备选方案 | `修改: <具体修改点>` 后重出 CP5；或 `reject` 回退到 CP3 approved / CP5 redesign |
| 影响维度 | 实施范围、文件 owner、测试策略、安全边界、warning 剩余风险 |
| 优劣分析 | 推荐方案修根因且可验证；备选要么保留根因，要么扩大范围 |
| 风险与回退 | CP5 通过后会进入代码 / 旧台账修复；如失败回退到 CP5 redesign 或 CP6 rework |
| 用户需决策事项 | `DQ-CP5-CR093-01` 是否批准 LLD；`DQ-CP5-CR093-02` 是否批准任务顺序；`DQ-CP5-CR093-03` 是否批准安全边界；`DQ-CP5-CR093-04` 是否接受 warning-only 风险；`DQ-CP5-CR093-05` 是否批准文件范围 |

## 不授权范围

本 CP5 即使通过，也不授权：

- QMT / MiniQMT / XtQuant / gateway / runner runtime 启动、连接、安装或运行。
- NAS 访问、列取、读取、复制、拉取、写入、发布或删除。
- 读取 `.env`、凭据、账号、账户、资金、持仓、委托、成交或日志原文。
- submit / cancel、buy / sell。
- simulation / live。
- provider fetch、lake write、catalog publish。
- 自动启动 CR089、CR091-FU-02、CR091-FU-03 或恢复 CR020 用户删除的 gateway 路线。
- 清理 broader `source=cp8-follow-up` warning-only 项。

## Entry Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| CP3 approved | 通过 | `process/checkpoints/CP3-CR093-LEDGER-HYGIENE-HLD-REVIEW.md` | 用户回复“好的，同意，继续推进”，按 CP5 approve 处理 |
| CP4 PASS | 通过 | `process/checks/CP4-CR093-STORY-DAG-PARALLEL-SAFETY.md` | 用户接受自动预检结论 |
| LLD ready | 通过 | `process/stories/CR093-LEDGER-HYGIENE-CONSISTENCY-LLD.md` | 用户接受 full-lld 设计证据 |
| TEST-PLAN / TASKS ready | 通过 | `docs/features/CR093-ledger-hygiene/` | 用户接受测试计划、任务顺序与文件 owner |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | full-lld 是否覆盖实现所需章节 | 通过 | LLD §0..§14 | 接受 |
| 2 | TEST-PLAN 是否覆盖目标缺陷和真实 current conflict | 通过 | TEST-PLAN TP-CR093-01..08 | 接受 |
| 3 | TASKS 顺序是否接受 | 通过 | TASKS | 接受先测试、再 checker、再最小台账规范化 |
| 4 | 文件 owner 是否清楚 | 通过 | TASKS / CP4 | 接受授权文件范围 |
| 5 | 不授权项是否完整 | 通过 | 本文件“不授权范围” | 接受；不授权外部动作 |

## Exit Criteria

| 用户回复 | 处理 |
|---|---|
| `approve` | 接受 5 项推荐方案，CR093 可进入受限实施；仍不授权外部动作。 |
| `修改: <具体修改点>` | 按修改点修订 LLD / TEST-PLAN / TASKS / CP5 材料并重新发起确认。 |
| `reject` | 回退到 CP3 approved / CP5 redesign。 |

## Deliverables

- `docs/design/FEATURE-DESIGN-MATRIX-CR093.md`
- `docs/features/CR093-ledger-hygiene/DESIGN.md`
- `docs/features/CR093-ledger-hygiene/TEST-PLAN.md`
- `docs/features/CR093-ledger-hygiene/TASKS.md`
- `process/stories/CR093-LEDGER-HYGIENE-CONSISTENCY-LLD.md`
- `process/context/CP5-CR093-LEDGER-HYGIENE-CONTEXT.yaml`
- `process/checks/CP4-CR093-STORY-DAG-PARALLEL-SAFETY.md`
- `process/checks/CP5-CR093-LEDGER-HYGIENE-READINESS.md`

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：`2026-06-18T17:58:38+08:00`
- 用户回复：好的，同意，继续推进
- 接受的决策项：`DQ-CP5-CR093-01`、`DQ-CP5-CR093-02`、`DQ-CP5-CR093-03`、`DQ-CP5-CR093-04`、`DQ-CP5-CR093-05`
- 修改意见：无
- 风险接受项：接受 broader warning-only 项不作为本轮 DoD；DoD 只要求 CR019 / CR025 阻断项收敛
- 不授权边界：不授权 QMT / MiniQMT / XtQuant / gateway / runner runtime、NAS、凭据、账户、交易、simulation/live、provider/lake/publish、CR089 自动启动、CR091-FU-02、CR091-FU-03 或 CR020 gateway 路线恢复
