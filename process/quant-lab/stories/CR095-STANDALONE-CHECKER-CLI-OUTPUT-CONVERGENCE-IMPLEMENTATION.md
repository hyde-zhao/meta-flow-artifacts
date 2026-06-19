---
story_id: "CR095-STANDALONE-CHECKER-CLI-OUTPUT-CONVERGENCE"
status: "implemented"
owner: "host-orchestrator"
created_at: "2026-06-19T09:25:32+08:00"
source_cr: "CR-095"
validation_mode: "static-only"
---

# CR095 实现说明

## 实现前置检查

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| 用户明确启动 | PASS | 当前对话“启动CR093-FU-02” | 仅启动 standalone checker 输出收敛 |
| process 路由健康 | PASS | `meta-flow workspace check` | `process_link_health: ok` |
| active CR 冲突 | PASS | `meta-flow check cr-tracking --strict-warnings` | 启动前无 active formal CR |
| 风险边界 | PASS | CR095 / CR094 CP8 | 不授权 runtime、NAS、凭据、账户、交易、provider/lake/publish |

## 实现对象清单

| 对象 | 路径 | 动作 | 验证 |
|---|---|---|---|
| standalone checker | `scripts/check_cr_tracking_consistency.py` | 新增四类 CR tracking summary 输出；保留原深度一致性检查和 PASS/FAIL 结果 | pytest / standalone 命令 |
| 测试 | `tests/test_cr093_cr_tracking_consistency.py` | 增加 CR095 active fixture、summary lines 和 main 输出断言 | pytest |
| CR 台账 | `process/changes/CR-093-FOLLOW-UP-TRACKING-2026-06-19.md` | 将 CR093-FU-02 转为正式 CR095 active row | main CLI / standalone checker |
| 状态索引 | `process/STATE.md`、`process/changes/CR-INDEX.yaml` | 切换 active_change / active_crs 到 CR095 | main CLI / standalone checker |
| 正式 CR | `process/changes/CR-095-STANDALONE-CHECKER-CLI-OUTPUT-CONVERGENCE-2026-06-19.md` | 记录影响分析、执行链路和不授权边界 | CR tracking checker |

## 设计契约映射

| 契约 | 实现位置 | 结果 |
|---|---|---|
| standalone 输出应与主 CLI 同形态展示四类摘要 | `tracking_summary_lines()` / `main()` | PASS |
| 不引入 `meta_flow` Python 依赖到 quant-lab 项目 | 使用标准库解析 formal CR 和 follow-up rows | PASS |
| 保留原有深度一致性检查 | `check_project()` 未移除；summary 在 PASS/FAIL 前输出 | PASS |
| 支持 CR095 active formal CR | `FORMAL_CR_FILES` / tests fixture | PASS |
| 不触碰 runtime / NAS / 凭据 / 交易 / publish | 本轮仅静态文件与测试 | PASS |

## 最小实现切片

| Slice | 内容 | 验证 |
|---|---|---|
| S1 | standalone checker 新增 summary 解析与输出 | `pytest tests/test_cr093_cr_tracking_consistency.py` |
| S2 | CR095 active 状态同步到 formal CR / STATE / CR-INDEX / follow-up tracking | `meta-flow check cr-tracking --strict-warnings` |
| S3 | release feedback 从 candidate-not-started 更新为 active-formal-cr | 人工审查 + checker |

## 本地验证

| 命令 | 结果 |
|---|---|
| `uv run --python 3.11 meta-flow check cr-tracking --project-root . --strict-warnings` | PASS |
| `uv run --python 3.11 python scripts/check_cr_tracking_consistency.py --project-root .` | PASS；输出 summary + `CR tracking consistency: PASS` |
| `PYTHONDONTWRITEBYTECODE=1 PYTEST_ADDOPTS='-p no:cacheprovider' uv run --python 3.11 pytest -q tests/test_cr093_cr_tracking_consistency.py` | PASS，9 passed |

## 交接摘要

- 行为变化：standalone checker 现在先输出 `CR tracking summary` 四类摘要，再输出原 PASS/FAIL 结果。
- 兼容性：命令参数不变；退出码语义不变。
- 剩余风险：未启动独立 subagent，按低风险 static-only inline fallback 记录到 CP8 风险接受。
- QA 关注点：确认主 CLI 和 standalone 在当前项目上 active / blocked / candidate / spike 四类摘要一致。
