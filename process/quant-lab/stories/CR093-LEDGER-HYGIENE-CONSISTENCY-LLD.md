---
story_id: "CR093-LEDGER-HYGIENE-CONSISTENCY"
title: "Ledger Hygiene Consistency"
story_slug: "ledger-hygiene-consistency"
lld_version: "1.0"
tier: "M"
status: "ready-for-review"
confirmed: false
created_by: "host-orchestrator"
created_at: "2026-06-18T17:55:00+08:00"
source_cr: "CR-093"
feature_design_refs:
  - "docs/design/HLD-CR093-LEDGER-HYGIENE.md"
  - "docs/design/FEATURE-DESIGN-MATRIX-CR093.md"
  - "docs/features/CR093-ledger-hygiene/DESIGN.md"
  - "docs/features/CR093-ledger-hygiene/TEST-PLAN.md"
  - "docs/features/CR093-ledger-hygiene/TASKS.md"
lld_policy:
  required_level: "full-lld"
  trigger_reasons:
    - "state-machine-checker"
    - "historical-ledger-normalization"
    - "cross-CR status equivalence"
    - "current-vs-audit-history boundary"
open_items: 0
---

# LLD: CR093 — Ledger Hygiene Consistency

本文档是 CR093 CP5 的 Story 级 full-lld 设计证据。它只定义 checker / 账本修复的实施合同；CP5 未 approved 前不得修改 checker 或旧账本正文。

## 0. 上游设计依据

| 来源 | 路径 / ID | 被本 LLD 消费的内容 |
|---|---|---|
| CR | `process/changes/CR-093-LEDGER-HYGIENE-CR019-CR025-TRACKING-CLEANUP-2026-06-18.md` | 范围、目标缺陷、不授权边界 |
| CP2 | `process/checkpoints/CP2-CR093-LEDGER-HYGIENE-SCOPE-REVIEW.md` | 用户批准 ledger-only 范围 |
| CP3 HLD | `docs/design/HLD-CR093-LEDGER-HYGIENE.md` | 三层状态模型、状态等价表、warning-only 范围 |
| Feature Design | `docs/features/CR093-ledger-hygiene/DESIGN.md` | 文件 owner、接口合同、失败路径 |

## 1. Goal

实现前冻结 CR093 的文件范围、状态模型、测试计划和回退策略，让后续实现能安全修复 CR019 / CR025 旧账阻断。

## 2. Requirements

- FR-01：checker 必须从 current 层判断当前 active / blocked / closed 冲突。
- FR-02：history / discussion / backup 中的 nested active_change 默认只能产生 audit warning。
- FR-03：closed / cancelled 状态等价表必须覆盖 CR020、CR030、CR040..045、CR021..024。
- FR-04：真实 current conflict 仍必须 exit 1。
- NFR-01：不访问 NAS、runtime、凭据、账户、data/reports 或 provider/lake/publish。
- NFR-02：不删除历史审计文本。

## 3. 模块拆分

| 模块 | 责任 |
|---|---|
| CurrentStateReader | 读取 STATE 顶层字段、`cr_tracking.active_crs[]`、formal CR frontmatter |
| TrackingRowReader | 读取 follow-up tracking 当前表格行 |
| AuditHistoryScanner | 扫描 history / discussion，仅输出 warning |
| StatusNormalizer | 把 formal / tracking 状态归一为 active / blocked / closed / cancelled / candidate / spike |
| ReportWriter | 输出固定五类 summary 和 exit code |

## 4. 文件影响范围

| 文件 | 动作 | 说明 |
|---|---|---|
| `scripts/check_cr_tracking_consistency.py` | 修改 | 分层读取、状态归一、summary / exit code 调整 |
| `tests/test_cr093_cr_tracking_consistency.py` | 新增 | 单元测试 / fixture |
| `process/changes/CR-019-FOLLOW-UP-TRACKING-2026-05-31.md` | 最小修改 | 规范化必要行或补充状态等价说明 |
| `process/STATE.md` / `process/changes/CR-INDEX.yaml` | 更新 | CP6/CP7/CP8 状态同步 |

## 5. 数据模型

| 对象 | 字段 | 说明 |
|---|---|---|
| NormalizedStatus | `raw_status` / `normalized` / `source` / `is_current_failure` | 状态归一结果 |
| LedgerFinding | `severity` / `category` / `message` / `source_path` / `line` | checker 输出项 |
| TrackingSummary | `active_formal_crs` / `blocked_formal_crs` / `follow_up_candidates` / `spike_candidates` / `stale_status_conflicts` | 固定 summary |

## 6. API / Interface

| 函数候选 | 输入 | 输出 |
|---|---|---|
| `normalize_status(raw_status: str) -> str` | formal / tracking raw status | normalized category |
| `scan_current_state(project_root: Path) -> list[LedgerFinding]` | project root | current findings |
| `scan_audit_history(text: str) -> list[LedgerFinding]` | state / discussion text | warning findings |
| `check_project(project_root: Path) -> list[str]` | project root | failure messages，兼容现有入口 |

## 7. 核心流程

1. 读取 formal CR frontmatter 和 CR-INDEX。
2. 读取 CR019 follow-up tracking 当前表格。
3. 对 formal / tracking 状态执行 `normalize_status`。
4. 检查 current 层冲突，保留 exit 1。
5. 扫描 audit-history，输出 warning，不计入 failures。
6. 打印五类 summary。

## 8. 技术细节

- 优先保持现有 CLI 入口和 `check_project()` 返回类型，降低调用面改动。
- 可新增内部 dataclass，但不引入第三方依赖。
- 状态等价表集中定义，避免散落 `require("| CR-020 | closed |")` 这类硬编码。
- CR019 tracking 的最小规范化不得删除旧决策，只允许在当前表格行或说明中澄清 normalized 状态。

## 9. 安全与性能

| 维度 | 设计 |
|---|---|
| 安全 | 只读 `process/`、`scripts/`、`tests/` 内的文本；不读 `.env`、data、reports、NAS |
| 性能 | 文件数固定，文本扫描 O(n)，无需优化 |
| 可恢复 | 所有改动可通过 git diff 回滚 |

## 10. 测试设计

测试按 `docs/features/CR093-ledger-hygiene/TEST-PLAN.md` 执行。最低必须覆盖 TP-CR093-01..05。

## 11. 实施步骤

| TASK-ID | 动作 | 文件 | 验证 |
|---|---|---|---|
| TASK-CR093-01 | 新增测试 | `tests/test_cr093_cr_tracking_consistency.py` | pytest 失败证明目标缺陷 |
| TASK-CR093-02 | 修改 checker | `scripts/check_cr_tracking_consistency.py` | pytest / py_compile |
| TASK-CR093-03 | 最小规范化 tracking | CR019 follow-up | cr-tracking |
| TASK-CR093-04 | 回写状态证据 | process checks / STATE | workspace / diff |

## 12. 风险、难点与预研建议

| 风险 | 等级 | 缓解 |
|---|---|---|
| 真实 current conflict 被错降级 | high | fixture 覆盖顶层 active 指向 closed CR |
| warning-only 被误当成功 | medium | CP7 明确 warning 剩余范围 |
| 旧台账改动过大 | medium | 最小 diff，保留历史说明 |

## 13. 回滚与发布策略

不做 runtime 发布。若实现失败，回滚 checker 和 CR019 tracking 改动，CR093 回到 CP5 redesign 或 CP6 rework。

## 14. Definition of Done

- [ ] CP5 approved 后才能实施
- [ ] 新测试覆盖目标缺陷和真实 current conflict
- [ ] `meta-flow check cr-tracking --project-root .` 不再被 CR019 / CR025 旧账阻断
- [ ] warning-only 项清晰保留
- [ ] 不触碰外部运行 / NAS / 凭据 / 账户 / 数据发布
