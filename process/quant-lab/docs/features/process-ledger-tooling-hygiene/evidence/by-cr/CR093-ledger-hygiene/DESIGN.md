---
status: draft
version: "1.0"
feature_id: "F-CR093-01"
source_cr: "CR-093"
created_at: "2026-06-18T17:55:00+08:00"
owner: "host-orchestrator"
---

# CR093 Ledger Hygiene Feature Design

## 目标

冻结 CR093 的实现设计：让 `meta-flow check cr-tracking --project-root .` 能区分当前状态、当前 tracking 表格和历史审计文本，消除 CR019 / CR025 旧账导致的阻断，同时保留真实 current conflict 的失败能力。

## 设计范围

| 范围 | 包含 | 不包含 |
|---|---|---|
| checker 语义 | current / tracking-current / audit-history 三层读取；状态等价表；summary 输出 | 全量 checker 重写 |
| CR019 tracking | 最小状态规范化和说明补充 | 删除历史决策或重写旧台账 |
| 测试 | 新增 CR093 checker fixture / 单元测试；保留真实 conflict 失败用例 | runtime / NAS / 凭据测试 |
| 状态索引 | 同步 CR093 CP5 pending 和后续 CP6 输入 | 自动启动其他 CR |

## 接口 / 数据合同

| 合同 | 输入 | 输出 | 失败路径 |
|---|---|---|---|
| `parse_current_state` | `process/STATE.md` 顶层字段、`cr_tracking`、`orchestrator_session` | current active / blocked / pending gate | 顶层 active 指向 closed formal CR 时 FAIL |
| `parse_tracking_rows` | CR follow-up tracking 当前表格 | candidate / active / closed / cancelled / spike rows | tracking row 与 formal CR 无法映射时 FAIL 或 warning |
| `classify_audit_history` | `STATE.md.history`、discussion、backup 文本 | audit warnings | 默认不影响 exit code，除非配置为强审计 |
| `status_equivalence` | formal CR status + tracking status | normalized active / closed / cancelled / blocked | 未知状态 FAIL |

## 文件影响

| 文件 | 动作 | 说明 |
|---|---|---|
| `scripts/check_cr_tracking_consistency.py` | 修改 | 拆分当前状态读取和 audit-history 扫描；引入状态等价工具 |
| `tests/test_cr093_cr_tracking_consistency.py` | 新增 | 覆盖 CR019 状态等价、CR025 history 误扫、真实 current conflict |
| `process/changes/CR-019-FOLLOW-UP-TRACKING-2026-05-31.md` | 最小修改 | 规范化必要状态或补充说明，不删除历史 |
| `process/checks/*CR093*` / `process/checkpoints/*CR093*` | 更新 | 写入 CP6/CP7 证据 |

## 失败与回退

| 失败 | 处理 | 回退 |
|---|---|---|
| 真实 current conflict 被错降级 | 测试阻断，修正 current 层规则 | revert checker parser |
| CR019 文本改动过大 | 停止并转人工审查 | revert tracking 文档改动 |
| warning-only 项被误纳入本轮 | 从实现中移除，保留 follow-up | 另起 CR |

## 下游 Story

| Story | lld_policy | 设计证据 |
|---|---|---|
| CR093-LEDGER-HYGIENE-CONSISTENCY | full-lld | `process/stories/CR093-LEDGER-HYGIENE-CONSISTENCY-LLD.md` |
