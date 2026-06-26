---
check_id: "RUNNER-QMT-SIMULATION-MULTIFACTOR-FORMAL-RUNTIME-INPUT-FIX-2026-06-26"
workflow_id: "RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-2026-06-25"
created_at: "2026-06-26T10:35:00+08:00"
owner: "host-orchestrator"
status: "IMPLEMENTED_STATIC_VERIFIED"
runtime_touched: false
gateway_required: false
small_live_or_live_authorized: false
---

# Runner QMT Simulation Multifactor Formal Runtime Input Fix

## 1. 结论

已修复 formal runtime 输入生成缺口的本地工程部分。本轮未打开 gateway，未执行 signed readonly，未执行 runtime operator，未 submit/cancel。

本次修复将交易窗口输入分为两层：

1. 仓库内 formal fixture 包继续用于非交易窗口 readiness。
2. 交易窗口 runtime 必须由私有 overlay 生成私有 runtime operator spec 和私有 runtime admission package，不得直接使用 `INSTRUMENT_*` fixture 包。

## 2. 实现内容

| 对象 | 路径 | 说明 |
|---|---|---|
| runtime fail-closed guard | `trading/strategy_runner/simulation_operator.py` | runtime mode 在触达 gateway 前检查 signal/current/risk symbols，发现 `INSTRUMENT_*`、`instrument:`、`symbol:` 或 fixture ref 时返回 `runtime_symbol_contract_invalid` |
| 私有输入构建模块 | `trading/strategy_runner/runtime_inputs.py` | 基于 base spec、formal admission 和私有 overlay 生成私有 runtime spec / admission package |
| 私有输入构建 CLI | `scripts/build_qmt_multifactor_runtime_inputs.py` | 输出 `<run-id>-runtime-spec.json` 和 `<run-id>-runtime-admission-package.json` |
| 测试 | `tests/test_runner_runtime_inputs.py` | 验证 fixture 包可通过 private symbol_map 映射为 runtime 输入，且残留 fixture symbol 会 fail closed |
| 回归测试 | `tests/test_runner_multifactor_simulation_operator.py` | 验证 runtime operator 遇到 fixture symbol contract 会在 gateway transport 前阻断 |

## 3. 使用方式

交易窗口前先准备私有 overlay，例如放在 `/home/hyde/.quant-lab/runtime/qmt/cr138-simulation/<run-id>-runtime-overlay.json`。overlay 必须提供私有 symbol map、current positions 和 risk snapshot；不要写入仓库。

示例命令形态：

```bash
uv run --python 3.11 python scripts/build_qmt_multifactor_runtime_inputs.py \
  --base-spec-json process/context/RUNNER-QMT-SIMULATION-MULTIFACTOR-FORMAL-OPERATOR-SPEC-2026-06-25.json \
  --strategy-admission-json process/context/RUNNER-QMT-SIMULATION-MULTIFACTOR-FORMAL-STRATEGY-ADMISSION-PACKAGE-2026-06-25.json \
  --runtime-overlay-json /home/hyde/.quant-lab/runtime/qmt/cr138-simulation/<run-id>-runtime-overlay.json \
  --readonly-evidence-ref process/evidence/<readonly-evidence>.json \
  --run-id <run-id> \
  --runtime-authorization-ref <runtime-authorization-ref> \
  --expected-runtime-profile cr138-simulation \
  --output-dir /home/hyde/.quant-lab/runtime/qmt/cr138-simulation
```

后续 runtime operator 应使用生成出的私有 runtime spec 和私有 runtime admission package，而不是仓库里的 formal fixture admission package。

## 4. 验证

| 命令 | 结果 |
|---|---|
| `PYTHONDONTWRITEBYTECODE=1 PYTEST_ADDOPTS='-p no:cacheprovider' uv run --python 3.11 pytest -q tests/test_runner_runtime_inputs.py tests/test_runner_multifactor_simulation_operator.py tests/test_cr020_runtime_manual_validation.py -q` | PASS，18 passed |
| `PYTHONPYCACHEPREFIX=/tmp/quant-lab-runtime-inputs-pycompile uv run --python 3.11 python -m py_compile scripts/build_qmt_multifactor_runtime_inputs.py trading/strategy_runner/runtime_inputs.py trading/strategy_runner/simulation_operator.py tests/test_runner_runtime_inputs.py tests/test_runner_multifactor_simulation_operator.py` | PASS |
| `uv run --python 3.11 python scripts/build_qmt_multifactor_runtime_inputs.py ... --runtime-overlay-json /home/hyde/.quant-lab/runtime/qmt/cr138-simulation/runner-qmt-simulation-multifactor-trading-window-2026-06-26-r3-runtime-overlay.json ...` | PASS，已生成私有 r3 runtime spec / admission |
| `PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 python scripts/run_qmt_multifactor_simulation_operator.py --mode preflight-only ... r3 private inputs ...` | PASS，`runtime_touched=false` |
| `PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 python scripts/run_qmt_multifactor_simulation_operator.py --mode plan-only ... r3 private inputs ...` | PASS，`runtime_touched=false`，`order_count=1` |
| `rg -n "510300|510500|INSTRUMENT_" process/evidence/runner-simulation-runtime-input-fix-2026-06-26/.../operator-evidence.json` | PASS，无命中 |

## 5. 剩余风险

| 风险 | 状态 | 处理 |
|---|---|---|
| 真实可提交 symbol / risk snapshot 尚未提供 | OPEN | 需要用户或上游私有 runtime overlay 提供；本轮不猜测、不从脱敏 evidence 反推 raw symbol |
| gateway 是否需要打开 | NOT_REQUIRED_THIS_FIX | 本轮只做本地输入生成和静态/单元验证；下一次 runtime 前再逐次授权并按 P0 开始 |
| 稳定性窗口 | NOT_COUNTED | 本轮不是 runtime pass，不计入 `1/5` |

## 6. R3 私有输入准备

用户授权选择两个 ETF 作为本次 runtime overlay 的候选标的：

- 沪深300ETF华泰柏瑞：上交所基金代码 `510300`，私有 overlay 使用 QMT 格式 `510300.SH`。
- 中证500ETF南方：上交所基金代码 `510500`，私有 overlay 使用 QMT 格式 `510500.SH`。

来源核对：

- 上海证券交易所 ETF 信息页确认 `510300` 为沪深300ETF华泰柏瑞。
- 上海证券交易所 ETF 信息页确认 `510500` 为中证500ETF南方。

私有文件路径：

| 文件 | 路径 |
|---|---|
| runtime overlay | `/home/hyde/.quant-lab/runtime/qmt/cr138-simulation/runner-qmt-simulation-multifactor-trading-window-2026-06-26-r3-runtime-overlay.json` |
| runtime spec | `/home/hyde/.quant-lab/runtime/qmt/cr138-simulation/runner-qmt-simulation-multifactor-trading-window-2026-06-26-r3-runtime-spec.json` |
| runtime admission package | `/home/hyde/.quant-lab/runtime/qmt/cr138-simulation/runner-qmt-simulation-multifactor-trading-window-2026-06-26-r3-runtime-admission-package.json` |

这些私有文件不得提交到仓库。下一次 runtime 前应重新确认价格、资金、持仓和授权窗口；当前本地 plan-only 通过不等于 runtime 授权。
