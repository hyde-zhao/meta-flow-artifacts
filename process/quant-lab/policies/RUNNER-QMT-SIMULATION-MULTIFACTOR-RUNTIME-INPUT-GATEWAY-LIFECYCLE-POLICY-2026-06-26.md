---
policy_id: "RUNNER-QMT-SIMULATION-MULTIFACTOR-RUNTIME-INPUT-GATEWAY-LIFECYCLE-POLICY-2026-06-26"
workflow_id: "RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-2026-06-25"
created_at: "2026-06-26T11:37:00+08:00"
owner: "host-orchestrator"
status: "active"
covers:
  - "SIM-OPS-GATE-02"
  - "SIM-OPS-GATE-03"
runtime_touched: false
small_live_or_live_authorized: false
---

# Runner QMT Simulation Multifactor Runtime Input And Gateway Lifecycle Policy

## 1. 目的

本策略固化 Phase B 后续两个 operational gates：

| Gate | 结论 | 本策略覆盖 |
|---|---|---|
| `SIM-OPS-GATE-02` Runtime input policy | `POLICY_DEFINED` | private overlay 来源、刷新时机、字段边界、输出位置和脱敏要求 |
| `SIM-OPS-GATE-03` Gateway lifecycle policy | `POLICY_DEFINED` | Windows gateway 启动、重启、停止、WSL/Windows 同步和每轮前置检查 |

本策略不是 runtime authorization，不授权新的 simulation submit/cancel，不授权 `small_live` / `live`，不授权读取凭据、raw account/order/fund detail、NAS/provider/lake/catalog/publish 或 remote Git。

## 2. Runtime Input Policy

### 2.1 Source Of Truth

| 输入类型 | 允许来源 | 禁止来源 |
|---|---|---|
| base operator spec | 已审核的 formal operator spec / admission package 路径引用 | 手工拼接 JSON 字符串、未审计临时 spec |
| runtime overlay | 私有 runtime 目录下的 overlay 文件 | `process/` 正式 evidence、Git tracked 文件、聊天记录 |
| runtime authorization ref | 本次逐次授权文本中的 ref | 上一次 runtime ref、smoke ref、live ref |
| current positions / risk snapshot | 授权窗口内 signed readonly 的脱敏摘要或私有输入 | raw account、raw position payload、fund detail |
| stability evidence refs | 已通过且脱敏的 evidence index / summary refs | raw operator payload、raw broker ref、失败运行 refs |

默认私有 runtime 目录：

`/home/hyde/.quant-lab/runtime/qmt/cr138-simulation/`

该目录允许保存授权窗口内的私有 runtime overlay、runtime spec 和 admission package；不得把这些文件复制到 `process/` 或 Git tracked 路径。

### 2.2 Overlay 字段边界

允许字段：

| 字段 | 规则 |
|---|---|
| `symbol_map` | 必填；只用于把 fixture symbol 映射到授权窗口内选定的真实 runtime symbol。正式 evidence 不保存 raw symbol。 |
| `current_positions` | 可选；来自授权窗口内 readonly 结果或空仓假设，保存在私有 runtime 输入中。 |
| `risk_snapshot` | 可选；只保存在私有 runtime 输入中。 |
| `risk_profile` | 可选；必须与 simulation profile 一致。 |
| `capital_base` | 可选；不得泄露真实资金细节。 |
| `max_turnover_notional` | 可选；用于限制订单计划。 |
| `stability_evidence_refs` | 可选；只包含历史成功 evidence refs，不包含当前 readonly ref。 |

禁止字段：

| 字段类别 | 规则 |
|---|---|
| credentials / secrets / tokens / session | 不得进入 overlay、spec、admission、正式 evidence 或 checkpoint。 |
| raw account / raw positions / fund detail | 不得进入正式 evidence；若 runtime 私有输入需要 current positions，只能留在私有 runtime 目录。 |
| raw broker order ref / raw order payload | 不得写入 overlay 或正式 evidence。 |
| `small_live` / `live` profile | 本策略只覆盖 simulation；live 输入必须另起 CR。 |

### 2.3 Build And Output Rules

Runtime input 只能通过受控 builder 生成：

```bash
uv run --python 3.11 python scripts/build_qmt_multifactor_runtime_inputs.py \
  --base-spec-json <formal-operator-spec.json> \
  --strategy-admission-json <formal-admission-package.json> \
  --runtime-overlay-json <private-runtime-overlay.json> \
  --readonly-evidence-ref <redacted-readonly-evidence-ref> \
  --run-id <authorized-run-id> \
  --runtime-authorization-ref <per-run-authorization-ref> \
  --expected-runtime-profile cr138-simulation \
  --output-dir /home/hyde/.quant-lab/runtime/qmt/cr138-simulation
```

Builder 输出规则：

| 项目 | 要求 |
|---|---|
| output dir | 必须是私有 runtime 目录，不得是 `process/`、`reports/`、`data/` 或 Git tracked 路径。 |
| fixture guard | `INSTRUMENT_*`、`fixture:`、`symbol:`、`instrument:` 等非 runtime symbol 必须 fail closed。 |
| evidence summary | 只允许输出 count、path、run_id、authorization_ref、profile 和 `raw_symbols_printed=false`。 |
| overwrite | 同一 run_id 不应重复生成；如需重跑，必须使用新的 run_id 和新的逐次授权。 |

### 2.4 Refresh Rules

| 触发条件 | 处理 |
|---|---|
| 新的一次 simulation runtime | 必须重新生成 overlay/spec/admission，并重新取得逐次授权。 |
| gateway 重启或 session 变化 | 必须重新执行 P0 health / identity 和 P0.5 signed readonly；不得复用旧 readonly 状态。 |
| target symbol / risk 参数变化 | 必须重新生成 overlay/spec/admission。 |
| 代码修改影响 `trading/*` | 必须同步到 `/mnt/c/quant-lab-runtime`，用户重启 gateway 后再做任何 runtime。 |
| 任一 P0/P0.5/P1/P2/P3/P4 失败 | 当前 run 不计入稳定窗口；不得沿用失败输入继续提交。 |

## 3. Gateway Lifecycle Policy

### 3.1 Windows Gateway Start

Gateway 由用户在 Windows 项目目录启动，项目目录为：

`C:\quant-lab-runtime`

推荐启动命令：

```powershell
uv run --with typer --python 3.11 python -m trading.qmt_runtime_cli serve `
  --env-file .env `
  --host 172.30.32.1 `
  --port 18765 `
  --runtime-authorization-ref <per-run-simulation-runtime-authorization-ref>
```

启动命令本身不等于 runtime authorization；`runtime-authorization-ref` 必须来自本次逐次授权。

### 3.2 WSL Preflight Sequence

每次 simulation runtime 前必须从 P0 开始：

| Step | 必须结果 | 阻断条件 |
|---|---|---|
| P0 health | gateway reachable，session ready，runtime profile 为 `cr138-simulation` / simulation | unreachable、session expired、profile mismatch |
| P0 capabilities | endpoint matrix 包含 positions、simulation submit、simulation cancel | missing positions / submit / cancel |
| P0.5 signed readonly | signed positions readonly pass，正式 evidence 脱敏 | auth fail、session fail、redaction fail |
| P1/P2 | target 和 order plan pass | fixture symbol、order_count 不符合授权 |
| P3/P4 | submit/cancel 与 reconciliation pass | unknown、unresolved、manual takeover |

任一步失败都必须停止，不进入下一步。

### 3.3 Restart And Stop Rules

| 场景 | 规则 |
|---|---|
| session expired / credential not configured / identity mismatch | 用户在 Windows 重新登录或重启 gateway；WSL 侧从 P0 重来。 |
| 修改 `/mnt/c/quant-lab-runtime/trading/*` | 用户必须重启 gateway；重启后重新验证 health/capabilities/readonly。 |
| 单次 runtime 完成 | 如果没有下一次明确授权和目标，建议用户停止 gateway。 |
| runtime 中断或 unknown order state | 不自动重试 submit/cancel；先形成 incident / reconciliation summary。 |
| 准备 `small_live` / `live` | 不复用当前 gateway profile / authorization；必须独立 CR 和 live-specific policy。 |

### 3.4 WSL / Windows Code Sync Rule

| 项目 | 要求 |
|---|---|
| source of code edits | 默认在 WSL 仓库 `/home/hyde/workspace/quant-lab` 修改。 |
| Windows runtime mirror | 影响 gateway runtime 的 `trading/*` 文件必须同步到 `/mnt/c/quant-lab-runtime`。 |
| restart trigger | 同步后必须提示用户手工重启 gateway。 |
| verification trigger | 重启后必须重新执行 P0 health / identity；旧验证不再有效。 |

本策略不要求本次执行同步或重启，因为本次只写 `process/*` policy 文档，没有修改 runtime code。

## 4. Evidence And Redaction Policy

正式 evidence 允许保存：

- run_id、authorization_ref、runtime_profile、count、bucket、status、reason_code。
- 脱敏 positions count / digest / bucket。
- evidence index path、policy path、runbook path。
- submit/cancel count 与 adapter status 的脱敏 summary。

正式 evidence 禁止保存：

- raw account、raw symbol、raw positions、raw broker order ref、raw payload。
- credential、secret、token、session。
- fund detail、真实资金明细。

## 5. Gate Closure

| Gate | Status | Closure Evidence |
|---|---|---|
| `SIM-OPS-GATE-02` | `POLICY_DEFINED` | 本策略 §2 |
| `SIM-OPS-GATE-03` | `POLICY_DEFINED` | 本策略 §3 |

如未来需要长期自动化模拟盘，下一步应把本策略转成可执行 preflight checker；在此之前，本策略作为人工 runbook 和状态恢复入口。
