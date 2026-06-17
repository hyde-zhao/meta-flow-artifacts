---
check_id: "CR089-STARTUP-CONFLICT-PRECHECK-2026-06-17"
status: "BLOCKED_PENDING_CP2_CP3_CP5_REVIEW"
checked_at: "2026-06-17T21:20:00+08:00"
owner: "host-orchestrator"
source_request: "用户要求继续规划 QMT 接口验证，并确认 QMT 运行级可从 NAS 获取可运行策略"
---

# CR089 启动冲突预检

## 结论

CR089 可以作为 QMT 接口验证的独立运行授权门禁草案记录，但当前不得静默转为 active。用户已接受 CR089 独立 blocked start 的 startup 冲突决策；当前阻断转为 CP2/CP3/CP5 formal review pending。

阻断原因：

1. `process/STATE.md.active_change` 仍为 `CR-046`。
2. CR046 仍是 `active-cp6-pass-ready-for-verification`，并且暂停在 CP7 前。
3. CR089 的 `strategy_package_contract`、`qmt_terminal_target`、`miniqmt_runner_target`、`trading_runtime_boundary` 和 `per_run_authorization` 与 CR046 冲突键重叠。
4. CR020 已是 `deleted-by-user`，不得作为当前 QMT 验证入口恢复。

## 路由与索引检查

| 检查 | 结果 | 说明 |
|---|---|---|
| process route health | PASS | `meta-flow workspace check --project-root /home/hyde/workspace/quant-lab` 返回 `process_link_health: ok` |
| CR tracking consistency | PASS | `scripts/check_cr_tracking_consistency.py --project-root /home/hyde/workspace/quant-lab` 返回 PASS |
| 当前 active_change | PASS_WITH_RISK | `CR-046`，保持不变 |
| CR089 active lock | PASS | 未设置为 active |

## 离线验证结果

| 验证 | 结果 | 说明 |
|---|---|---|
| `py_compile` | PASS | `trading/qmt_client.py`、`trading/qmt_endpoint_matrix.py`、`trading/qmt_runtime_cli.py`、`scripts/check_cr_tracking_consistency.py` 可编译 |
| QMT no-authorization / readonly / client CLI 回归子集 | PASS | 40 个测试通过：CR020 docs、query_positions readonly、Linux REST transport、CR019 run gates、CR019 C side client CLI |
| 宽 endpoint matrix 子集 | PASS_WITH_RISK | `tests/test_cr019_qmt_endpoint_matrix.py::test_client_methods_consume_matrix_and_default_later_gated_endpoints_blocked` 失败：测试期望 `query_positions` 默认先返回 `per_run_authorization_missing`，而 CR020 REST 特例要求无 base_url/transport 时返回 `transport_unavailable`。该冲突需在 CR089 CP3/CP5 明确 positions endpoint 的优先阻断语义后再修复，不在本启动预检内改代码。 |

## 冲突矩阵

| 影响面 | CR046 | CR089 | 冲突 |
|---|---|---|---|
| strategy package contract | 已定义框架 | 扩展 NAS exchange 与 runtime intake | 是 |
| QMT terminal target | 已定义人工导入 / shadow plan | 验证取包后只读 smoke | 是 |
| MiniQMT runner target | 已定义安装设计 | 可选接口 smoke 依赖 XtQuant/MiniQMT 权限 | 是 |
| NAS package exchange | CR051 定义 blocked flow | CR089 计划转为待授权 flow | 是 |
| runtime authorization | CR046 明确不授权 | CR089 需要逐 run 只读授权 | 是 |
| CR020 gateway | deleted-by-user | 只复用代码 / runbook 安全边界 | 否，不恢复 |

## 允许继续的只读工作

- 记录 CR089 草案。
- 记录策略包 / NAS 交付规划。
- 运行静态测试或离线合同测试。
- 生成待人工决策清单。

## 继续禁止的工作

- 不访问 NAS 内容、不挂载、不复制真实策略包。
- 不读取 `.env`、NAS 凭据、QMT 账号、HMAC secret。
- 不启动 QMT / MiniQMT / XtQuant / gateway。
- 不查询真实账户、持仓、资金、委托、成交。
- 不 submit/cancel，不 simulation/live。

## 待人工决策

Startup 冲突决策已由用户 2026-06-17 回复“同意”接受。当前待审查对象是：

- `process/checkpoints/CP2-CR089-QMT-INTERFACE-VALIDATION-BASELINE.md`
- `process/checkpoints/CP3-CR089-QMT-INTERFACE-VALIDATION-HLD-REVIEW.md`
- `process/checkpoints/CP5-CR089-QMT-INTERFACE-VALIDATION-READINESS.md`

该审查仍不授权 NAS / QMT / 凭据 / 账户 / submit / cancel / simulation / live。

| 决策 ID | 问题 | 推荐 |
|---|---|---|
| DQ-CR089-01 | CR089 与 CR046 重叠如何处理 | 先保持 CR089 blocked formal CR，用户确认后再独立启动 |
| DQ-CR089-02 | QMT 运行级是否 NAS 原地执行 | 默认从 NAS 拉取 approved package，校验后复制到交易主机本地缓存 |
| DQ-CR089-03 | 本次接口验证级别 | 只读 `query_positions` smoke，scope=`qmt:positions:read` |
| DQ-CR089-04 | agent 是否可读凭据或代跑 runtime | 不允许 |
