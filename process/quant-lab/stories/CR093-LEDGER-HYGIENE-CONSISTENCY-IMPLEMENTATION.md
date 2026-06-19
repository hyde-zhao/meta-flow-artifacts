---
story_id: "CR093-LEDGER-HYGIENE-CONSISTENCY"
story_slug: "ledger-hygiene-consistency"
source_cr: "CR-093"
status: "implemented"
owner: "meta-dev"
created_at: "2026-06-18T18:07:40+08:00"
---

# CR093 Ledger Hygiene Consistency Implementation

## 实现前置检查

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| CP5 用户同意 | PASS | 当前任务上下文 | 用户明确说明 CR093 CP5 已获“同意”。 |
| 授权边界 | PASS | 当前任务上下文 | 仅修改 checker、CR093 测试、CR019 最小状态规范化和实现证据。 |
| 不授权项 | PASS | 命令记录 | 未读取 `.env`、凭据、账号、账户、资金、持仓、委托、成交或日志原文；未启动 runtime；未访问 NAS；未执行 provider/lake/publish、submit/cancel、simulation/live。 |
| 外部 CLI 一致性 | PASS | `uv run --python 3.11 meta-flow check cr-tracking --project-root .` | 用户批准扩大 owner 后，已同步外部 CLI checker；目标命令通过。 |

## 实现对象清单

| 对象 | 路径 | 动作 | 验证 |
|---|---|---|---|
| cr-tracking checker | `scripts/check_cr_tracking_consistency.py` | 新增状态归一、tracking 行状态解析、当前 active formal CR 校验、audit history warning-only 解析 | py_compile；CR093 pytest；脚本自检 PASS |
| CR093 单元测试 | `tests/test_cr093_cr_tracking_consistency.py` | 新增临时 project fixture，覆盖状态等价、history active_change warning-only、真实 current conflict 失败 | 4 passed |
| CR019 台账 | `process/changes/CR-019-FOLLOW-UP-TRACKING-2026-05-31.md` | 增加状态规范化补充说明 | diff review；不删除历史 |
| 外部 meta-flow CLI checker | `/home/hyde/workspace/meta-flow/meta_flow/checks/cr_tracking.py` | 用户批准扩大 owner 后，同步状态归一与 history warning-only 语义 | py_compile；目标 CLI PASS |
| CP6 输入 | `process/checks/CP6-CR093-LEDGER-HYGIENE-CONSISTENCY-CODING-DONE.md` | 记录实现与阻断状态 | 本文件 |

## 设计契约映射

| LLD 契约 | 实现位置 | 验证结果 |
|---|---|---|
| FR-01 current 层判断 active / closed 冲突 | `check_project()` 顶层 `active_change` + formal status 归一校验 | `test_current_active_change_to_closed_formal_cr_still_fails` |
| FR-02 history / discussion nested active_change 默认 warning-only | `audit_history_active_changes()` | `test_audit_history_active_change_is_warning_only` |
| FR-03 closed / cancelled 状态等价 | `normalize_status()`、`require_tracking_status()` | `test_normalize_status_groups_formal_and_tracking_equivalents` |
| FR-04 真实 current conflict 仍 exit 1 | `check_project()` failures | pytest 失败路径覆盖 |
| NFR-01 静态文本、无 runtime / NAS / 凭据 / 交易副作用 | 实现仅用 pathlib/re/sys 标准库 | 命令记录；无外部运行命令 |

## 单元测试与 Fixture 计划

| 测试 ID | 覆盖 | 状态 |
|---|---|---|
| TP-CR093-01 | 当前 CR093 active formal CR 可被当前层识别 | PASS |
| TP-CR093-02 | CR025 nested history active_change 不阻断 | PASS |
| TP-CR093-03 | CR020/030/040..045 closed 等价 | PASS |
| TP-CR093-04 | CR021..024 cancelled-user-deleted 等价 | PASS |
| TP-CR093-05 | 顶层 active_change 指向 closed CR 仍失败 | PASS |
| TP-CR093-06 | broader cp8-follow-up warning-only | PASS | 外部 CLI 保留 warning-only，但 exit 0。 |

## 最小实现切片

| Slice | TASK-ID | 文件 | 结果 |
|---|---|---|---|
| S1 | TASK-CR093-01 | `tests/test_cr093_cr_tracking_consistency.py` | 新增 4 个单测，覆盖目标语义。 |
| S2 | TASK-CR093-02 / 03 | `scripts/check_cr_tracking_consistency.py` | 状态归一与 current/history 分层实现完成。 |
| S3 | TASK-CR093-04 | `process/changes/CR-019-FOLLOW-UP-TRACKING-2026-05-31.md` | 增加最小状态规范化说明。 |
| S4 | TASK-CR093-05 | `process/stories/*IMPLEMENTATION.md`、`process/checks/CP6-*` | 记录验证结果。 |
| S5 | DQ-CP6-CR093-01 | `/home/hyde/workspace/meta-flow/meta_flow/checks/cr_tracking.py` | 用户批准扩大 owner 后，同步正式 CLI checker；目标命令 PASS。 |

## 平台差异处理

N/A。本 Story 不涉及 Claude / Codex / OpenClaw agent 安装结构、平台路径或运行时适配。

## 验证结果

| 命令 | 结果 | 说明 |
|---|---|---|
| `PYTHONDONTWRITEBYTECODE=1 PYTEST_ADDOPTS='-p no:cacheprovider' uv run --python 3.11 pytest -q tests/test_cr093_cr_tracking_consistency.py` | PASS | 4 passed |
| `uv run --python 3.11 python -m py_compile scripts/check_cr_tracking_consistency.py tests/test_cr093_cr_tracking_consistency.py` | PASS | 无输出，编译通过 |
| `uv run --python 3.11 python scripts/check_cr_tracking_consistency.py --project-root .` | PASS | `CR tracking consistency: PASS` |
| `git diff --check -- scripts/check_cr_tracking_consistency.py tests/test_cr093_cr_tracking_consistency.py process/changes/CR-019-FOLLOW-UP-TRACKING-2026-05-31.md` | PASS | 无 whitespace error |
| `/home/hyde/.local/share/uv/tools/meta-flow/bin/python3 -m py_compile /home/hyde/workspace/meta-flow/meta_flow/checks/cr_tracking.py` | PASS | 外部 CLI checker 编译通过 |
| `uv run --python 3.11 meta-flow check cr-tracking --project-root .` | PASS | `OK`，仅保留 warning-only 项 |

## 未覆盖项

- 未处理 broader cp8-follow-up 缺 follow-up tracking row warning；LLD 允许 warning-only broader 项保留。

## 设计缺口反馈

CR093 的 LLD 最初将 `scripts/check_cr_tracking_consistency.py` 视为目标 checker，但目标验证命令 `meta-flow check cr-tracking` 实际消费外部包实现 `/home/hyde/workspace/meta-flow/meta_flow/checks/cr_tracking.py`。CP6 中已由用户批准扩大 owner，并将同等 current/history 分层与状态归一语义落到外部 CLI 实现。

## 后续交接

给 meta-qa / host-orchestrator 的建议：

1. CP7 应以 `meta-flow check cr-tracking --project-root .` 为主验收命令。
2. warning-only 的 `source=cp8-follow-up` missing tracking row 不纳入本轮 DoD。
3. 不应新增 runtime / NAS / 凭据 / 账户 / 交易 / provider / lake / publish 验证。
