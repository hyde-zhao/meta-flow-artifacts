---
checkpoint_id: "CP2-CR085-REMOTE-CLEANUP-BASELINE"
checkpoint_name: "CR085 Remote Cleanup Baseline"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-06-17T13:42:00+08:00"
reviewed_by: "user"
reviewed_at: "2026-06-17T13:56:36+08:00"
auto_check_result: "process/checks/CP2-CR085-REMOTE-CLEANUP-BASELINE.md"
context_capsule: "process/context/CP2-CR085-REMOTE-CLEANUP-CONTEXT.yaml"
auto_final_authorization: false
target:
  phase: "documentation"
  story_id: ""
  artifacts:
    - "process/context/CP2-CR085-REMOTE-CLEANUP-CONTEXT.yaml"
---

# CP2 CR085 Remote Cleanup Baseline 人工审查

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP2-CR085-REMOTE-CLEANUP-BASELINE.md` | PASS | 0 | `.env.example` 已改为允许保留，`.env` 明确禁止上传；远端当前无必须删除项。 |

## Decision Brief

推荐决策：批准 CR085 CP2 baseline，确认当前远端清理边界为：`.env.example` 可以保留，`.env` 不得上传；远端 master 当前没有 `.env`、`reports/`、`runs/`、`data/`、`process/` 命中，因此当前删除清单为空。直接 push `d4d2881` 仍被阻断，因为本地候选包含 `runs/RUN-EXEC-20260613-001.md` 且会扩大远端文件面。

备选方案：继续删除 `.env.example`；直接 push `d4d2881`；停止 CR085。推荐方案优先，因为它符合用户最新边界，同时保持远端写入 fail-closed。

影响维度：远端同步范围、模板文件保留、`.env` 凭据边界、clean publication 候选、CR084 普通 push 暂停。

风险与回退：若 reject，CR085 停止且不执行远端动作；若修改，需重写扫描结论和待决策表；若后续发现 `.env` 或其他禁止路径进入远端，再另行进入删除 / 历史清理门禁。回退条件为保持远端 master `3ade165e8b1edad031a911490cf6c1cee942616e` 不变。

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP2-CR085-REMOTE-CLEANUP-CONTEXT.yaml` |
| capsule 状态 | ready |
| read_profile | compact |
| 默认读取策略 | 先读 capsule；仅在缺失、冲突、字段不足、人工审计或深度评审时读取完整文档 |
| 全文档读取扩展 | 0 次 |
| 缺失 / waived 理由 | Story LLD N/A：CR085 是 Git remote cleanup / clean publication gate。 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `process/STATE.md.human_gate_decisions.pending_human_decisions[]` | scanned | 2 | 2 | CR085 非授权边界纳入。 |
| CR085 formal CR | `process/changes/CR-085-REMOTE-SYNCHRONIZATION-CLEANUP-GATE-2026-06-17.md` | scanned | 5 | 5 | scope、env 边界、clean publication 纳入。 |
| 自动预检结果 | `process/checks/CP2-CR085-REMOTE-CLEANUP-BASELINE.md` | scanned | 5 | 5 | scan 修正与 direct push blocker 纳入。 |
| 只读远端扫描 | `process/checks/CR085-REMOTE-READONLY-SCAN-2026-06-17.md` | scanned | 4 | 4 | 远端删除候选为空，local `runs/**` 命中纳入。 |
| 用户显式输入 | 当前对话 `.env.example可以保留。.env不要上传` | scanned | 2 | 2 | 作为优先级最高边界。 |
| discussion log / checkpoint | `process/discussions/*` | n/a | 0 | 0 | Git cleanup gate 不需要产品场景 discussion log。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP2-CR085-01 | scope | 是否暂停 CR084 直接 push，继续由 CR085 收敛远端同步 / 清理？ | 暂停 CR084 写动作，CR085 先收敛 clean publication 边界。 | 继续 CR084 普通 push；停止 CR085。 | 推荐避免把本地 forbidden hit 推到远端；继续 push 风险高。 | 远端暂不变。 | reject 时 CR085 停止；修改时重写门禁。 |
| DQ-CP2-CR085-02 | security | `.env.example` 与 `.env` 如何分类？ | `.env.example` 保留；`.env` 禁止上传、读取、打印或保存。 | 删除 `.env.example`；允许提交 `.env`。 | 推荐符合用户修正；备选会误删模板或泄露凭据。 | 影响 clean publication allowlist。 | 若 `.env.example` 后续含真实密钥迹象，停止并另起安全门。 |
| DQ-CP2-CR085-03 | scope | 当前远端是否需要删除文件？ | 当前不删除；远端扫描未见 `.env` / reports / runs / data / process。 | 删除 `.env.example`；直接历史清理。 | 推荐最小化远端写；备选与用户边界冲突。 | 当前没有远端删除动作。 | 后续发现真实禁止路径时另起删除门禁。 |
| DQ-CP2-CR085-04 | risk_acceptance | 是否接受 `d4d2881` 不能直接 push？ | 接受；必须先走 clean publication 候选。 | 强行 push；停止发布。 | 推荐可控；强推会扩大远端文件面。 | 发布被延后。 | clean candidate 通过后再评估 push。 |
| DQ-CP2-CR085-05 | follow_up_tracking | CR046、data/reports、external process 是否纳入？ | 不纳入；保持 CR046 paused，数据和 external process 独立治理。 | 同步处理 CR046；同步治理 data/reports/process。 | 推荐降低 blast radius；备选扩大范围。 | 后续治理仍未完成。 | 需要时另起独立 CR。 |

### 用户需决策事项

用户需决策事项：是否接受 DQ-CP2-CR085-01、DQ-CP2-CR085-02、DQ-CP2-CR085-03、DQ-CP2-CR085-04、DQ-CP2-CR085-05 的推荐方案。回复 `approve` 表示接受这 5 项推荐方案；不表示授权“不授权项”中的任何操作。

### CP2 追加 Decision Brief 字段

| 字段 | 内容 |
|---|---|
| 用户真实意图 | 处理远程同步，并确保不该上传的内容不进入远端；最新修正为 `.env.example` 可保留、`.env` 不上传。 |
| 场景覆盖 | 远端最新树检查、本地候选检查、direct push 阻断、clean publication 后续准备。 |
| Scenario Gray Areas | N/A，Git cleanup gate 不涉及产品场景讨论。 |
| Deferred Ideas | 远端写入、历史清理、force push、默认分支治理、external process Git/NAS。 |
| 回退方式 | reject 即保持远端和本地 Git 状态不变。 |

### 不授权项

如果你回复 approve，表示接受上述 5 项推荐方案；不表示授权以下操作：

- 不授权 `git push`、`git push -u`、remote deletion、remote add / set-url / remove。
- 不授权 `git fetch` 到当前仓库、tag、force push、history rewrite、default branch governance。
- 不授权上传、读取、打印或保存 `.env`、token、password、private key、cookie、session。
- 不授权 dirty worktree commit、data/reports 内容读取、NAS 内容操作、provider/lake/catalog、runtime、CR046 recovery、old root retirement 或 cleanup。

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| CR085 已创建 | 待审查 | `process/changes/CR-085-REMOTE-SYNCHRONIZATION-CLEANUP-GATE-2026-06-17.md` |  |
| 只读扫描已完成 | 待审查 | `process/checks/CR085-REMOTE-READONLY-SCAN-2026-06-17.md` |  |
| CP2 context 已生成 | 待审查 | `process/context/CP2-CR085-REMOTE-CLEANUP-CONTEXT.yaml` |  |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | CR085 是否继续承接远端同步 / 清理 | 待审查 | DQ-CP2-CR085-01 |  |
| 2 | env 边界是否接受 | 待审查 | DQ-CP2-CR085-02 |  |
| 3 | 当前不删除远端文件是否接受 | 待审查 | DQ-CP2-CR085-03 |  |
| 4 | direct push 阻断是否接受 | 待审查 | DQ-CP2-CR085-04 |  |
| 5 | 不相关治理是否排除 | 待审查 | DQ-CP2-CR085-05 |  |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| 用户明确 approve / 修改 / reject | 待审查 | 本文件人工审查结果 |  |
| approve 后仍不执行远端写动作 | 待审查 | 不授权项 |  |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| CP2 context | `process/context/CP2-CR085-REMOTE-CLEANUP-CONTEXT.yaml` | 待审查 |  |
| CP2 auto check | `process/checks/CP2-CR085-REMOTE-CLEANUP-BASELINE.md` | 待审查 |  |
| CP2 launch message | `process/checks/CP2-CR085-HUMAN-GATE-LAUNCH-MESSAGE.md` | 待审查 |  |

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-06-17T13:56:36+08:00
- 修改意见：用户回复“好的继续”，按 CR085 CP2 baseline approve 处理；接受 DQ-CP2-CR085-01..05 的推荐方案。该批准确认 `.env.example` 可以保留、`.env` 禁止上传 / 读取 / 打印 / 保存、当前远端无删除项，并继续禁止直接 push `d4d2881`。
- 风险接受项：接受本轮不执行远端写入；direct push 阻断；CR046、data/reports、external process 不纳入本轮。
