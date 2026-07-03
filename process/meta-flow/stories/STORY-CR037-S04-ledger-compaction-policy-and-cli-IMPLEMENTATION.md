---
story_id: "CR037-S04"
cr_ref: "CR-037"
title: "ledger compaction policy and CLI"
stage: "CP6"
status: "implemented"
owner: "meta-dev"
created_at: "2026-07-03T10:42:44+08:00"
context_ref: "process/context/stories/CR037-S04.CP6.work-packet.json"
---

# CR037-S04 Implementation

## 实现前置检查

| 条目 | 结论 | 证据 |
|---|---|---|
| capsule-first 输入 | PASS | `process/context/stories/CR037-S04.CP6.work-packet.json` |
| Story 状态与依赖 | PASS | Story 为 `implementing`；S01 CP7 为 `PASS_WITH_RISK`，下游携带 `R-CR037-SECOND-MECHANISM`、`R-CR037-REGISTRY-DRIFT` |
| 文件所有权 | PASS | 仅写 work packet 允许路径；未写 `process/quant-lab/**`、`docs/**`、`delivery/**` |
| 授权边界 | PASS | 未执行 runtime、production write、publish、live、外部调用或凭据读取；apply 行为仅在 pytest 临时 fixture 中验证 |

## 实现对象清单

| 对象 | 路径 | 动作 | 验证 |
|---|---|---|---|
| ledger compact 核心模块 | `meta_flow/state/ledger_compaction.py` | 新增 retention policy、dry-run plan、apply、backup、archive summary/index、path guard | `tests/test_cr037_ledger_compaction.py` |
| event checker 兼容 | `meta_flow/state/event_ledger.py` | `ledger_compacted` marker 使用专用 required fields，不破坏普通 ledger required fields | compacted checkpoint fixture event check |
| CLI 路由 | `meta_flow/cli.py` | 新增顶层 `meta-flow ledger compact`，独立于 `state compact` | help 与 CLI fixture |
| state compact 文案 | `meta_flow/state/current.py` | 仅澄清 `state compact` 不压缩 NDJSON ledger；不改变执行语义 | `meta-flow state compact --help` |
| retention policy | `process/policies/LEDGER-RETENTION.yaml` | 新增独立默认 policy：90d / 500 events / 20 CR | policy loader 测试 |
| 回归测试 | `tests/test_cr037_ledger_compaction.py` | 覆盖 CLI 分离、policy、dry-run、apply、marker、hash mismatch、path guard | pytest |

## 设计契约映射

| 契约 | 实现位置 | 验证 |
|---|---|---|
| 不复用 `state compact` | `meta_flow/cli.py` 新增 `_run_ledger()`；`current.py` 只改 help | `meta-flow state compact --help && meta-flow ledger compact --help` |
| 默认 dry-run 不写文件 | `ledger_compaction.main()` 未传 `--apply` 只打印 plan | `test_dry_run_writes_nothing` |
| `--apply` 写 backup/archive/index/marker | `apply_compaction()` | `test_apply_writes_archive_index_backup_marker_and_event_check_passes` |
| retention 与 context/current budgets 分离 | `RetentionPolicy` 与 `process/policies/LEDGER-RETENTION.yaml` | `test_retention_policy_defaults_and_invalid_policy` |
| archive summary 不复制完整长日志 | `_build_summary()` 只写统计、hash、range、sample ids、refs | summary 内容断言 |
| event checker 兼容 marker | `event_ledger.COMPACT_MARKER_REQUIRED_FIELDS` | compacted ledger `validate_event_ledger(..., checkpoint)` |
| path guard 拒绝 quant-lab 和越界路径 | `guard_ledger_path()` | `test_path_guard_rejects_quant_lab_and_outside_project` |
| hash mismatch 不改原 ledger | `apply_compaction()` apply 前乐观锁 | `test_hash_mismatch_aborts_without_changing_current_payload` |

## 单元测试与 Fixture 计划

| 测试 ID | 状态 | 覆盖 |
|---|---|---|
| TP-LC-01 | PASS | `state compact` 与 `ledger compact` help 分离 |
| TP-LC-02 | PASS | 默认 policy 与非法 policy |
| TP-LC-03 | PASS | dry-run no-write |
| TP-LC-04 | PASS | apply 生成 summary/index/backup/marker |
| TP-LC-05 | PASS | compacted ledger event check 兼容 |
| TP-LC-06 | PASS | hash mismatch abort，不写 archive |
| SEC-LC-TC-03 | PASS | `process/quant-lab/**` 与项目外路径拒绝 |

## 最小实现切片

| Slice | 内容 | 结果 |
|---|---|---|
| S04-A | 新增 policy 与 compaction planner/apply 模块 | PASS |
| S04-B | 新增 CLI 路由与 `state compact` help 边界 | PASS |
| S04-C | 更新 event checker marker 兼容 | PASS |
| S04-D | 增加临时 fixture 测试与边界验证 | PASS |
| S04-E | 生成 CP6 return/evidence/result/summary | PASS |

## 平台差异处理

N/A。本 Story 修改 Python CLI 与 process policy，不涉及 Claude / Codex / OpenClaw 平台安装结构或 agent frontmatter。

## 验证结果

| 命令 | 结果 | 摘要 |
|---|---|---|
| `PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 pytest -q tests/test_cr037_ledger_compaction.py` | PASS | 6 passed |
| `PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 meta-flow state compact --help && PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 meta-flow ledger compact --help` | PASS | state help 明确不压缩 NDJSON event ledgers；ledger help 覆盖 retention/archive 与 `--apply` |
| `PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 python -m py_compile meta_flow/state/ledger_compaction.py meta_flow/state/event_ledger.py meta_flow/cli.py meta_flow/state/current.py` | PASS | 语法检查通过 |
| `git diff -- process/quant-lab docs delivery` | PASS_WITH_WARNING | 命令显示 `delivery/rules/AGENT-SKILL-CONTRACT.md` 存在外部/并行 diff；S04 touched files 不包含 `process/quant-lab/**`、`docs/**` 或 `delivery/**` |
| `git diff --check -- meta_flow/state/ledger_compaction.py meta_flow/state/event_ledger.py meta_flow/cli.py meta_flow/state/current.py tests/test_cr037_ledger_compaction.py` | PASS | 无 whitespace 错误 |

## 未覆盖项

| 项 | 原因 | 后续入口 |
|---|---|---|
| 真实项目长期 ledger apply | 未授权 runtime / production write；apply 仅允许临时 fixture | CP7 可继续 fixture 验证；真实 apply 需维护者单独授权 |
| archive 写入失败的文件系统级异常 | 当前自动化覆盖 hash mismatch abort；未强制模拟权限错误 | 可在 CP7 增加只读目录 fixture，如环境支持 |
| 并发锁 | LLD 仅要求 hash 乐观锁；文件锁属于未来 CR 候选 | 若出现并发写 ledger，再发 CR |

## 设计缺口反馈

无需要回写长期 Feature DESIGN / ADR / HLD 的设计变更。实现采用 CP5 确认推荐值 `window_days=90`、`keep_latest_n_events=500`、`keep_latest_n_cr=20`。

## 后续交接

交给 meta-qa 时重点验证：

- `meta-flow ledger compact` 是独立命令，未改变 `event append/check/list` 和 `state compact` 执行语义。
- `--apply` 只在临时 fixture 中执行；真实 `process/state/*.ndjson` 未被压缩。
- archive summary/index 只保存统计、hash、range、refs 和 restore hint，不复制完整长日志。
- `current.py` 中除 help 边界外存在 S01 既有改动，本 Story 未回滚或重写。
- 工作区当前存在非 S04 的 `delivery/rules/AGENT-SKILL-CONTRACT.md` diff；S04 未写 delivery 路径，CP7 边界验证应以 touched files 和 diff 归属区分。
