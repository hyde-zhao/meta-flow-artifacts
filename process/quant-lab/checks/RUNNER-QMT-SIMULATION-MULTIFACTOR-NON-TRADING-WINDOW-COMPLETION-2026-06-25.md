---
check_id: "RUNNER-QMT-SIMULATION-MULTIFACTOR-NON-TRADING-WINDOW-COMPLETION-2026-06-25"
workflow_id: "RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-2026-06-25"
created_at: "2026-06-25T16:45:00+08:00"
owner: "host-orchestrator"
status: "PASS_WITH_TRADING_WINDOW_TASKS_DEFERRED"
runtime_touched: false
runtime_authorization_granted: false
small_live_or_live_authorized: false
---

# Runner QMT Simulation Multifactor Non-Trading Window Completion

## 1. 结论

本轮完成交接文件第 4 节中所有非交易窗口期可完成任务。执行范围限定为本地文件、离线 fixture、文档与证据索引；未读取 `.env`，未启动或重启 QMT gateway，未连接真实账户，未执行 signed positions readonly，未执行 runtime operator，未执行 simulation submit/cancel，未触达 `small_live` / `live`。

正式结论仍保持范围限制：

```text
runtime 链路已通过临时 spec 证明可通；正式 StrategyAdmissionPackage、正式 operator spec 已冻结并通过非交易窗口离线验证；真实交易窗口 runtime、真实 current positions、正式 buy/sell 路径、post positions 对账和稳定性窗口累计仍待授权后执行。
```

## 2. 正式输入

| 对象 | 路径 | 状态 |
|---|---|---|
| StrategyAdmissionPackage | `process/context/RUNNER-QMT-SIMULATION-MULTIFACTOR-FORMAL-STRATEGY-ADMISSION-PACKAGE-2026-06-25.json` | 固定；`not_authorization=true`，所有 forbidden operation counts 为 0 |
| operator spec | `process/context/RUNNER-QMT-SIMULATION-MULTIFACTOR-FORMAL-OPERATOR-SPEC-2026-06-25.json` | 固定；不含凭据、不含账号、不含 broker ref；`authorization_ref=""` 表示非交易窗口未授权 runtime |

输入说明：

- 新增输入仅用于非交易窗口合同验证，不代表日常运行验收。
- package/spec 使用离线 fixture instrument 标识，不保存真实账号、真实证券代码、原始订单回执、资金明细或 raw broker payload。
- 正式交易窗口运行前，必须用逐次授权窗口内的真实 current positions 脱敏摘要替换 fixture 输入，并重新生成 runtime evidence。

## 3. 离线验证结果

| Mode | Evidence | Index | 结果 |
|---|---|---|---|
| `preflight-only` | `process/evidence/runner-simulation-formal-2026-06-25/preflight-only/runner-qmt-simulation-multifactor-formal-non-trading-2026-06-25/operator-evidence.json` | `process/evidence/runner-simulation-formal-2026-06-25/preflight-only/runner-qmt-simulation-multifactor-formal-non-trading-2026-06-25/operator-evidence.index.json` | `pass` |
| `plan-only` | `process/evidence/runner-simulation-formal-2026-06-25/plan-only/runner-qmt-simulation-multifactor-formal-non-trading-2026-06-25/operator-evidence.json` | `process/evidence/runner-simulation-formal-2026-06-25/plan-only/runner-qmt-simulation-multifactor-formal-non-trading-2026-06-25/operator-evidence.index.json` | `pass` |
| `fixture` | `process/evidence/runner-simulation-formal-2026-06-25/fixture/runner-qmt-simulation-multifactor-formal-non-trading-2026-06-25/operator-evidence.json` | `process/evidence/runner-simulation-formal-2026-06-25/fixture/runner-qmt-simulation-multifactor-formal-non-trading-2026-06-25/operator-evidence.index.json` | `pass` |
| `reconcile-only` | `process/evidence/runner-simulation-formal-2026-06-25/reconcile-only/runner-qmt-simulation-multifactor-formal-non-trading-2026-06-25/operator-evidence.json` | `process/evidence/runner-simulation-formal-2026-06-25/reconcile-only/runner-qmt-simulation-multifactor-formal-non-trading-2026-06-25/operator-evidence.index.json` | `pass` |

关键断言：

| 字段 | 值 |
|---|---|
| `runtime_touched` | `false` |
| `runtime_authorization_granted` | `false` |
| `small_live_or_live_authorized` | `false` |
| `submitted_count` | `0` |
| `cancelled_count` | `0` |
| `unknown_count` | `0` |
| evidence index validation | `pass` |
| redaction summary | raw payload / account / symbol / broker ref / secret / fund detail 均为 `false` |

## 4. 稳定性窗口标准

| 字段 | 冻结值 |
|---|---|
| `window_id` | `simulation-daily-readiness-window` |
| `required_runs` | `5` |
| `required_trading_days` | `3` |
| pass criteria | 每次都有有效 runtime authorization；P1-P4 evidence schema 通过；`unknown_count=0`；manual takeover 记录关闭；gateway stop confirmed |
| fail criteria | 授权缺失或过期；evidence 出现 raw payload / secret；unknown order 未关闭；reconciliation diff 未关闭；gateway stop 未确认 |
| 当前状态 | `collecting`；本轮非交易窗口 fixture 不计入真实 runtime 稳定性累计 |

## 5. Manual Takeover Checklist

| 触发 | 必须动作 | 恢复条件 |
|---|---|---|
| `session_expired` | 停止 runner；不进入 P3；重启 / 重新登录 gateway 后从 health、capabilities、positions readonly 重新开始 | 新的授权窗口内 readonly pass，且无未关闭 incident |
| `order_submit_unknown` | 停止新单；记录 unknown intent digest；人工在 MiniQMT 核对状态 | unknown 转为 accepted/rejected/cancelled 明确状态，并重新跑 P4 |
| `cancel_unknown` | 停止新 submit；人工核对是否仍有未撤订单；禁止继续自动撤单循环 | cancel 状态明确，未成订单处理完成或记录为人工持仓风险 |
| `recon_diff` | 停止下一轮自动运行；记录 diff bucket 和 evidence ref；升级为 kill-switch candidate | 差异关闭，manual takeover 记录关闭，P4 重新 pass |
| `risk_blocked` | 不进入 submit/cancel；保留 P2 risk blocked 摘要 | 调整策略输入或风险参数后重新从 P1/P2 生成 |
| redaction fail | 丢弃该 evidence；不发布、不进入下一阶段 | 重跑并通过 redaction validation |

## 6. Kill-Switch 判定表

| 条件 | stop new orders | cancel unfinished | manual takeover | 备注 |
|---|---:|---:|---:|---|
| 授权缺失 / 过期 | 是 | 否，除非新授权允许 | 是 | fail closed |
| gateway health fail / capabilities missing | 是 | 否 | 是 | 不进入 P3 |
| kill switch active | 是 | 仅在授权内 | 是 | 不允许绕过 gate |
| submit unknown | 是 | 仅人工确认后 | 是 | unknown 关闭前不得恢复 |
| cancel unknown | 是 | 否，避免重复未知操作 | 是 | 需人工核对 broker 状态 |
| reconciliation diff unclosed | 是 | 视差异和授权决定 | 是 | 可升级 incident |
| raw evidence / secret leak | 是 | 否 | 是 | 丢弃 evidence，修复脱敏后重跑 |

## 7. Evidence 保留策略

| 项目 | 策略 |
|---|---|
| 保存范围 | 只保存 summary、bucket、digest、instrument ref、intent ref、validation result、evidence path |
| 禁止保存 | 真实账号、真实证券代码、原始持仓、原始订单、broker raw ref、资金明细、token、secret、raw payload |
| 保留周期 | 默认 `30` 天；长期只保留 index、check summary 和必要 digest refs |
| Git 边界 | 正式仓库只应保留脱敏 fixture、index、summary 和文档；任何本机私有 runtime 输入、env、raw broker payload 不得入库 |
| 发布边界 | 本轮未执行 publish、NAS、broker lake write、provider fetch、lake write 或 remote write |

## 8. Operator Command / Runbook Entry

非交易窗口日常入口：

```bash
uv run --python 3.11 python scripts/run_qmt_multifactor_simulation_operator.py \
  --mode preflight-only \
  --spec-json process/context/RUNNER-QMT-SIMULATION-MULTIFACTOR-FORMAL-OPERATOR-SPEC-2026-06-25.json \
  --strategy-admission-json process/context/RUNNER-QMT-SIMULATION-MULTIFACTOR-FORMAL-STRATEGY-ADMISSION-PACKAGE-2026-06-25.json \
  --output-dir process/evidence/runner-simulation-formal-2026-06-25/preflight-only
```

同一 spec/package 可切换 `--mode plan-only`、`--mode fixture`、`--mode reconcile-only`。非交易窗口不得传 `--env-file`，不得使用 `--mode runtime`。

交易窗口 runtime 入口仍未授权；未来必须由用户逐次授权后，才可传入 runtime env、base URL 和正式 current positions 脱敏输入。

## 9. 任务完成矩阵

| 交接顺序 | 任务 | 状态 | 证据 |
|---:|---|---|---|
| 1 | 更新状态文件，清除旧 `session_expired` 阻断 | 完成 | `process/STATE.md`、`process/state/STATE.current.json` |
| 2 | 固化 after-restart evidence 摘要 | 完成 | `process/checks/RUNNER-QMT-SIMULATION-MULTIFACTOR-AFTER-RESTART-VERIFICATION-2026-06-25.md` |
| 3 | 生成或选定正式 StrategyAdmissionPackage | 完成 | §2 |
| 4 | 生成正式 operator spec 模板 | 完成 | §2 |
| 5 | 跑正式输入 `preflight-only` | 完成 | §3 |
| 6 | 跑正式输入 `plan-only` | 完成 | §3 |
| 7 | 跑正式输入 `fixture` | 完成 | §3 |
| 8 | 跑正式输入 `reconcile-only` | 完成 | §3 |
| 9 | 定义稳定性窗口标准 | 完成 | §4 |
| 10 | 补 manual takeover checklist | 完成 | §5 |
| 11 | 补 kill-switch 判定表 | 完成 | §6 |
| 12 | 补证据保留策略 | 完成 | §7 |
| 13 | 设计 operator command / runbook-driven entry | 完成 | §8 |
| 14 | 刷新用户手册和 runbook | 完成 | `docs/USER-MANUAL.md`、`docs/components/RUNNER.md`、`docs/scenarios/*RUNNER*.md`、`process/runbooks/*RUNBOOK*.md` |
| 15 | 跑回归测试 | 完成 | 本轮命令输出；详见最终回复 |

## 10. 仍需交易窗口完成

以下事项只记录，不执行：

1. 用正式 StrategyAdmissionPackage 和真实 current positions 跑真实 runtime operator。
2. 验证正式 buy / sell 路径、post positions、订单回执状态和日终撤未成。
3. 累计 `5` 次 / `3` 个交易日稳定性窗口。
4. 演练 session_expired、gateway down、risk blocked、cancel unknown、recon diff。
5. 完成 gateway 停止与恢复验收。
