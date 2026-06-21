# CR104 Trading-Day Runtime Validation Input Checklist

> 适用日期：2026-06-22，按用户确认该日为交易日。若交易所日历临时变化，以用户现场确认和 QMT 可用状态为准。

## 用户可提前填写

```yaml
run_id: cr104-trading-day-runtime-20260622-USER_FILL
execution_host_label: USER_FILL_WINDOWS_QMT_HOST_LABEL
account_mode: simulated_or_readonly_real
qmt_terminal_status: user-opened-or-not-opened
gateway_mode: user-manual-windows-side
authorization_ref: user-approved-cr104-USER_FILL
credential_source: user-held-not-shared
evidence_output_mode: redacted-summary-only
allowed_endpoints:
  - health
  - capabilities
  - query_positions_readonly
forbidden_endpoints:
  - submit_order
  - cancel_order
  - simulation_submit
  - simulation_cancel
  - live_submit
  - live_cancel
raw_payload_allowed: false
credential_values_allowed: false
account_raw_values_allowed: false
```

## 回传 evidence 建议格式

```text
CR104 trading-day runtime validation result:
run_id: <脱敏 run id>
execution_host_label: <脱敏>
account_mode: simulated | paper | readonly-real
health_status: PASS/FAIL/SKIPPED
capabilities_status: PASS/FAIL/SKIPPED
query_positions_readonly_status: PASS/FAIL/SKIPPED
position_count_bucket: zero | one_to_ten | ten_plus | unavailable
positions_digest: sha256:<脱敏摘要或 unavailable>
raw_payload_included: false
credential_included: false
submit_cancel_attempted: false
simulation_live_attempted: false
checked_by: execution-host-operator
checked_at: <yyyy-MM-ddTHH:mm:ss+08:00>
notes: <脱敏说明>
```

## 明确不要回传

- 账号、资金、持仓明细、委托、成交、证券代码、数量、市值、交易密码。
- `.env` 内容、token、secret、cookie、session、private key、HMAC secret。
- QMT / MiniQMT / gateway 原始日志。
- 截图原图中包含账户或持仓原文的内容。

## 今日试运行脚本

今天只运行默认 dry-run / preflight 模式，不启动 QMT/MiniQMT/XtQuant/gateway，不读取 `.env`、凭据、账户或真实持仓：

```powershell
$ErrorActionPreference = "Stop"
cd <quant-lab项目根目录>
.\scripts\run_cr104_qmt_miniqmt_validation.ps1
```

预期输出包含：

```text
cr_id=CR-104
mode=DryRunPreflight
runtime_started=false
env_or_credential_read=false
account_or_position_raw_read=false
order_or_trade_action=false
evidence_path=<...>\cr104-dry-run-preflight-evidence.json
```

交易日真实只读 runtime 模式必须等 CR104 CP2 明确批准后再使用，并且需要单独传入 `-IApproveCR104CP2RuntimeAuthorization`、`-AuthorizationRef` 和用户侧授权的 `-EnvFile` / gateway 参数。未批准前不要运行 `ReadonlyRuntime` 模式。
