---
story_id: "CR093-LEDGER-HYGIENE-CONSISTENCY"
feature_id: "F-CR093-01"
source_cr: "CR-093"
status: "verified-with-risk"
validation_mode: "static-only"
checked_at: "2026-06-18T18:18:18+08:00"
owner: "meta-qa-critical"
---

# Verification: CR093 Ledger Hygiene Consistency

## 1. 结论

| 项目 | 内容 |
|---|---|
| 阶段决策 | PASS_WITH_RISK |
| validation_mode | static-only；feature TEST-PLAN 标注为 `static-ledger-and-unit-tests-only` |
| 路由 | host-orchestrator；进入 CP8 前保留风险接受 / follow-up tracking 输入 |
| 阻断项 | 0 |
| P0 / P1 缺陷 | 0 |
| 风险摘要 | `meta-flow check cr-tracking` 已 exit 0，但仍保留 warning-only 项；本轮 DoD 允许这些 broader warning 不阻断 |

CR093 的核心 DoD 已达成：CR019 / CR025 旧账不再导致 `meta-flow check cr-tracking --project-root .` 非零退出，当前 CR093 能被识别为 active formal CR，嵌套历史 `active_change` 被降级为 warning-only，且顶层 `active_change` 指向 closed formal CR 的真实冲突仍由单元测试覆盖为失败路径。

## 2. 验证范围

| 类别 | 范围 | 说明 |
|---|---|---|
| 设计证据 | `process/stories/CR093-LEDGER-HYGIENE-CONSISTENCY-LLD.md` | full-lld，CP5 人工审查已 approved |
| 测试计划 | `docs/features/CR093-ledger-hygiene/TEST-PLAN.md` | TP-CR093-01..08 |
| CP6 证据 | `process/checks/CP6-CR093-LEDGER-HYGIENE-CONSISTENCY-CODING-DONE.md`、`process/stories/CR093-LEDGER-HYGIENE-CONSISTENCY-IMPLEMENTATION.md` | CP6 结论 PASS |
| 本仓库 checker | `scripts/check_cr_tracking_consistency.py` | 静态文本 checker，不启动 runtime |
| CR093 测试 | `tests/test_cr093_cr_tracking_consistency.py` | 临时 project fixture 单元测试 |
| 外部 CLI checker | `/home/hyde/workspace/meta-flow/meta_flow/checks/cr_tracking.py` | `meta-flow check cr-tracking` 实际消费的 checker |
| 过程路由 | `uv run --python 3.11 meta-flow workspace check --project-root /home/hyde/workspace/quant-lab` | process symlink health |

非范围：未读取 `.env`、凭据、账号 / 账户、资金、持仓、委托、成交或日志原文；未启动 QMT / MiniQMT / XtQuant / gateway / runner runtime；未访问 NAS；未执行 provider / lake / publish、submit / cancel、simulation / live。

## 3. 验证对象清单

| 对象 | 验证方式 | 结果 |
|---|---|---|
| `normalize_status()` 状态归一 | 单元测试 + 代码审查 | PASS |
| current 层顶层 `active_change` 失败能力 | `test_current_active_change_to_closed_formal_cr_still_fails` | PASS |
| history / discussion nested `active_change` warning-only | 外部 checker warning + `test_audit_history_active_change_is_warning_only` | PASS_WITH_RISK |
| 本仓库 checker CLI | `uv run --python 3.11 python scripts/check_cr_tracking_consistency.py --project-root .` | PASS |
| 外部 `meta-flow check cr-tracking` CLI | `uv run --python 3.11 meta-flow check cr-tracking --project-root .` | PASS_WITH_RISK，exit 0 with warnings |
| 语法完整性 | 两组 `py_compile` | PASS |
| process 路由 | `meta-flow workspace check` | PASS；`process_link_health: ok` |
| dangerous-command-scan 窄范围扫描 | `rg` 扫 CR093 代码 / 测试 / 外部 checker 的 shell、网络、runtime、交易动作入口 | PASS；命中项为状态枚举或 fixture 文本，不是危险命令 |

## 4. 验证追踪矩阵

| TEST-PLAN ID | LLD / DoD 来源 | 实现 / 证据 | 验证结果 | 风险 |
|---|---|---|---|---|
| TP-CR093-01 | FR-01；DoD: 当前 CR093 active | 外部 checker summary: `active formal CRs: CR-093` | PASS | 无 |
| TP-CR093-02 | FR-02；history nested active_change warning-only | 外部 checker 对 CR025 nested active_change 输出 warning；单测 `test_audit_history_active_change_is_warning_only` | PASS_WITH_RISK | warning 保留 |
| TP-CR093-03 | FR-03；closed 状态等价 | `test_normalize_status_groups_formal_and_tracking_equivalents` 覆盖 `closed-current-delivery`、`closed-spike-complete`、`closed-cp8-approved` | PASS | 无 |
| TP-CR093-04 | FR-03；cancelled-user-deleted 等价 | 同一单测覆盖 `cancelled-user-deleted` | PASS | 无 |
| TP-CR093-05 | FR-04；真实 current conflict exit 1 | `test_current_active_change_to_closed_formal_cr_still_fails` | PASS | 无 |
| TP-CR093-06 | warning-only 范围 | 主 CLI exit 0，同时输出 cp8-follow-up missing tracking row warnings | PASS_WITH_RISK | broader warning 不清零 |
| TP-CR093-07 | py_compile | 本仓库脚本 / 测试和外部 checker 均编译通过 | PASS | 无 |
| TP-CR093-08 | workspace health | `process_link_health: ok` | PASS | artifact repo dirty 为既有状态，不阻断 |

全局 `docs/product/TEST-MATRIX.md` / `docs/product/SCENARIOS.yaml` 在当前仓库路径不存在；本 CP7 使用 CR093 feature-scoped `TEST-PLAN.md` 作为覆盖矩阵等价输入，并记录 N/A 原因：CR093 是单 CR ledger hygiene 修复，已在 CP5 人工确认 feature-scoped 测试矩阵。

## 5. 设计契约验证清单

| 契约 | 来源 | 验证方式 | 结果 |
|---|---|---|---|
| checker 必须从 current 层判断 active / closed 冲突 | LLD FR-01 | 外部 checker `collect_errors_and_warnings()`；单元失败路径 | PASS |
| history / discussion / backup 中 nested active_change 默认只能产生 audit warning | LLD FR-02 | 外部 checker warning-only；单测 helper 行为 | PASS_WITH_RISK |
| closed / cancelled 状态等价必须覆盖 CR020、CR030、CR040..045、CR021..024 | LLD FR-03 | 状态归一单测 + 代码审查 | PASS |
| 真实 current conflict 仍必须 exit 1 | LLD FR-04 | 单元测试断言失败信息 | PASS |
| 不访问 NAS、runtime、凭据、账户、data/reports 或 provider/lake/publish | LLD NFR-01 / CP5 不授权范围 | 命令边界 + dangerous-command-scan | PASS |
| 不删除历史审计文本 | LLD NFR-02 | 本轮验证未修改实现；CP6 说明 CR019 tracking 是最小说明补充 | PASS |

## 6. 分层验证计划

| 层级 | 执行项 | 结果 | N/A / 风险说明 |
|---|---|---|---|
| 静态语法 | py_compile 两组目标 | PASS | 无 |
| 单元 / fixture | CR093 pytest | PASS | 4 passed |
| 契约 / CLI | `meta-flow check cr-tracking`、本仓库脚本自检 | PASS_WITH_RISK | warning-only 仍存在 |
| 过程路由 | workspace check | PASS | artifact repo dirty 是过程仓库状态提示 |
| 安全扫描 | dangerous-command-scan 窄范围扫描 | PASS | QMT / simulation 等命中均为 fixture 路径或说明文本 |
| Prompt / Skill fixture | N/A | N/A | 本 Story 是 code-project / checker 修复，不涉及 Prompt / Skill / workflow eval |
| 平台安装 dry-run | N/A | N/A | 不涉及 Claude / Codex / OpenClaw 安装器 |
| runtime / integration | N/A | N/A | 用户明确禁止 runtime、NAS、provider/lake/publish 和交易动作 |

## 7. 自动化验证结果

| # | 命令 | 结果 | 关键输出 / 说明 |
|---|---|---|---|
| 1 | `uv run --python 3.11 meta-flow check cr-tracking --project-root .` | PASS_WITH_RISK | exit 0；summary: active formal CRs = CR-093；OK；保留 12 条 warning-only |
| 2 | `PYTHONDONTWRITEBYTECODE=1 PYTEST_ADDOPTS='-p no:cacheprovider' uv run --python 3.11 pytest -q tests/test_cr093_cr_tracking_consistency.py` | PASS | `4 passed in 0.03s` |
| 3 | `uv run --python 3.11 python -m py_compile scripts/check_cr_tracking_consistency.py tests/test_cr093_cr_tracking_consistency.py` | PASS | 无输出，编译通过 |
| 4 | `/home/hyde/.local/share/uv/tools/meta-flow/bin/python3 -m py_compile /home/hyde/workspace/meta-flow/meta_flow/checks/cr_tracking.py` | PASS | 无输出，编译通过 |
| 5 | `uv run --python 3.11 python scripts/check_cr_tracking_consistency.py --project-root .` | PASS | `CR tracking consistency: PASS` |
| 6 | `uv run --python 3.11 meta-flow workspace check --project-root /home/hyde/workspace/quant-lab` | PASS | `process_link_health: ok`；`artifact_git_dirty: dirty` |
| 7a | `git diff --check -- scripts/check_cr_tracking_consistency.py tests/test_cr093_cr_tracking_consistency.py` | PASS | 无 whitespace error |
| 7b | `git -C /home/hyde/workspace/meta-flow diff --check -- meta_flow/checks/cr_tracking.py` | PASS | 无 whitespace error |
| 7c | `git -C /home/hyde/workspace/meta-flow-artifacts diff --check -- process/quant-lab/changes/CR-019-FOLLOW-UP-TRACKING-2026-05-31.md process/quant-lab/checks/CP6-CR093-LEDGER-HYGIENE-CONSISTENCY-CODING-DONE.md process/quant-lab/stories/CR093-LEDGER-HYGIENE-CONSISTENCY-IMPLEMENTATION.md process/quant-lab/stories/CR093-LEDGER-HYGIENE-CONSISTENCY-LLD.md` | PASS | 无 whitespace error |

主 CLI warning-only 明细按类型归纳：

| 类型 | 项目 | 处理 |
|---|---|---|
| nested history active_change 指向 closed CR | CR092、CR025 | warning-only；不阻断当前 active CR093 |
| nested active_change 与 top-level active_change 不一致 | CR092、CR025 vs CR093 | warning-only；审计历史文本 |
| `source=cp8-follow-up` 缺 tracking row | CR053、CR058、CR074、CR075、CR078、CR080、CR092、CR093 | broader warning-only；CP5 已接受不纳入本轮 DoD |

## 8. Prompt / Skill Fixture 验证

N/A。本 Story 不修改 Prompt、Skill、agent workflow、platform renderer 或安装脚本。`workflow_eval_required=false`；不读取 `WORKFLOW-EVAL.yaml`、`PROMPT-BUNDLE.yaml` 或 run trace。

## 9. 平台适配验证

N/A。本 Story 不涉及 Claude / Codex / OpenClaw 安装路径、frontmatter、agent tools 或 install dry-run。唯一平台相关验证是 process 路由健康检查，结果为 `process_link_health: ok`。

## 10. 人工 / 语义质量审查

| 检查点 | 结论 |
|---|---|
| 目标缺陷是否闭环 | PASS：CR019 / CR025 旧账不再阻断主 CLI |
| 是否隐藏真实 current conflict | PASS：顶层 active_change 指向 closed formal CR 的失败路径有单测 |
| warning-only 是否误写为清零 | PASS_WITH_RISK：报告明确 warning 仍存在，未把 warning-only 当作无风险 PASS |
| 外部 checker 与本仓库脚本是否一致 | PASS_WITH_RISK：状态归一和 current/history 分层一致；warning 输出主要由外部主 CLI 承担 |
| 安全边界 | PASS：未执行 runtime / NAS / provider / publish / 交易动作；实现文件无 shell / 网络 / SDK 调用 |

## 11. 问题清单

| ID | 等级 | 问题 | 状态 | Owner | 处理 |
|---|---|---|---|---|---|
| R-CR093-01 | LOW | 主 CLI 仍输出 broader `source=cp8-follow-up` missing tracking row warnings，其中包含 CR093 自身。 | accepted-risk | host-orchestrator | CP5 已接受 warning-only 不作为本轮 DoD；进入 CP8 risk acceptance / follow-up tracking 输入。 |
| R-CR093-02 | LOW | 本仓库 `scripts/check_cr_tracking_consistency.py` 的 standalone CLI 不打印 audit-history warning 明细，warning 可见性主要依赖外部 `meta-flow check cr-tracking`。 | accepted-risk | host-orchestrator / future CR owner | 本轮主验收入口是外部 CLI，且 DoD 达成；如要求两个 checker 输出完全一致，应另起 follow-up。 |

无 BLOCKER / HIGH / MEDIUM finding；无需路由 meta-dev 回修。

## 12. 剩余风险

| 风险 | 影响 | 接受条件 | 重访条件 |
|---|---|---|---|
| warning-only backlog 未清零 | CP8 前仍会看到 broader warning，可能影响后续 ledger hygiene 清理优先级 | CR093 DoD 只要求 CR019 / CR025 阻断收敛；warning 已明示且 exit 0 | 用户要求 warning 清零、`--strict-warnings` 纳入质量门、或 CP8 判定 warning 不可接受 |
| 两套 checker 输出形态不完全一致 | 使用本仓库 standalone 脚本时看不到与外部 CLI 同等 warning 明细 | 主入口 `meta-flow check cr-tracking` 已验证；本仓库脚本仍能 PASS/FAIL 核心阻断 | 后续决定收敛 checker 实现或废弃 standalone 脚本 |

## 13. 阶段决策与 CP8 输入

- CP7 结论：`PASS_WITH_RISK`
- 可推进条件：阻断缺陷为 0，验证命令全部 exit 0。
- CP8 输入：保留 `R-CR093-01`、`R-CR093-02` 作为 risk acceptance / follow-up tracking 候选。
- 不授权项：本结论不授权 runtime、凭据、外部接口、数据写入、publish、simulation/live、submit/cancel 或任何交易动作。
