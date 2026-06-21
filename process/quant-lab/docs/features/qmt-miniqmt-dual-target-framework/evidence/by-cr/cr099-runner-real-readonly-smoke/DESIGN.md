---
feature_id: "cr099-runner-real-readonly-smoke"
cr_id: "CR-099"
status: "verified-with-risk"
owner: "host-orchestrator"
created_at: "2026-06-19T16:04:52+08:00"
source_hld: "docs/qmt/CR099-QMT-RUNNER-REAL-READONLY-SMOKE-HLD.md"
---

# CR099 Runner Real Readonly Smoke Feature Design

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 0.1 | 2026-06-19 | host-orchestrator | 初版 Feature Design，冻结 run contract、授权门禁、preflight、evidence schema、失败路径和不授权边界。 |
| 0.2 | 2026-06-19 | host-orchestrator | 用户回复“同意”批准 CP5；CP6 离线 checker / tests 已实现并通过。 |

## 1. Feature 边界

CR099 Feature 只覆盖 runner 真实消费 Windows gateway readonly path 的单次 smoke 设计。它以 CR098 runner readonly adapter 为消费方，以 CR097 Windows gateway readonly path 为候选运行环境，并新增 CR099 run contract、authorization gate、redacted evidence schema 和 blocked-path rules。

本 Feature 不改写策略 runner 核心执行逻辑，不引入交易写，不访问 NAS，不做 provider / lake / catalog publish。

## 2. 触发原因

| 触发条件 | 是否命中 | 说明 |
|---|---|---|
| 外部接口 | true | Windows QMT gateway readonly HTTP path |
| 安全 / 权限 | true | HMAC client env、账户只读查询、脱敏 evidence |
| 运行授权 | true | 必须逐 run 授权 |
| 失败路径 | true | 缺授权、缺 env、gateway 不可用、redaction fail 均需 blocked |
| 数据模型 | true | redacted evidence schema 和 forbidden counters |
| 迁移 / 不可逆变更 | false | 本轮不改生产数据或远端状态 |

## 3. Run Contract

CP5 只批准 contract 形态，不填入真实 secret 或执行参数。真实运行前必须由用户另行提供逐 run 授权文本。

```yaml
schema_version: 1
contract_id: CR099-RUNNER-REAL-READONLY-SMOKE-RUN-CONTRACT
cr_id: CR-099
run_id: "<user-authorized-run-id>"
authorization_ref: "<user-visible authorization reference>"
authorized_by: "<user>"
authorized_at: "<ISO-8601>"
expires_at: "<ISO-8601 or single-use>"
gateway:
  host: "<explicit host>"
  port: "<explicit port>"
  scheme: "http"
  allowlist:
    - health
    - capabilities
    - query_positions_readonly
client_env:
  handling: "explicit-user-authorized-reference-only"
  env_ref: "<redacted ref or path allowed by user for this run>"
  secret_values_allowed_in_contract: false
evidence:
  output_dir: ".quant-lab/evidence/qmt/cr099/redacted/<run_id>/"
  raw_payload_allowed: false
abort_conditions:
  - missing_cp5_approval
  - missing_runtime_authorization
  - missing_or_ambiguous_env_ref
  - endpoint_not_in_allowlist
  - redaction_gate_unavailable
  - forbidden_counter_non_zero
  - raw_payload_detected
```

## 4. Authorization Gate

| Gate | 条件 | 失败行为 |
|---|---|---|
| CP5 approved | CP5 checkpoint status = approved | 不读取 env，不运行 |
| per-run authorization | 用户明确授权本次 `authorization_ref` / `run_id` / host / port / allowlist / evidence dir | `blocked-pending-runtime-authorization` |
| env handling | 用户明确允许读取的 client env ref 或用户手工执行路径 | `blocked-missing-env-authorization` |
| allowlist | 仅 `health`、`capabilities`、`query_positions_readonly` | 非 allowlist endpoint 立即 blocked |
| redaction gate | evidence schema 和 forbidden keys checker ready | 不运行或不保存结果 |

## 5. Evidence Schema

```yaml
schema_version: 1
cr_id: CR-099
run_id: "<run_id>"
authorization_ref: "<authorization_ref>"
generated_at: "<ISO-8601>"
execution_mode: "codex-runner-authorized | user-manual-windows-side | blocked-preflight"
endpoints:
  health:
    attempted: true
    status: "ok | blocked | error"
    raw_payload_emitted: false
  capabilities:
    attempted: true
    status: "ok | blocked | error"
    readonly_supported: true
    raw_payload_emitted: false
  query_positions_readonly:
    attempted: true
    status: "ok | empty | blocked | error"
    position_count_bucket: "zero | one_to_ten | gt_ten | unknown"
    positions_digest: "<sha256 over redacted normalized shape or empty>"
    items_redacted_count: 0
    raw_payload_emitted: false
forbidden_counters:
  account_id_values: 0
  security_code_values: 0
  raw_quantity_values: 0
  raw_cash_values: 0
  order_fields: 0
  fill_fields: 0
  submit_cancel_calls: 0
  buy_sell_calls: 0
  nas_operations: 0
  provider_lake_publish: 0
```

## 6. 失败路径

| 条件 | 输出 | 是否可继续 |
|---|---|---|
| CP5 未批准 | 无 runtime 输出；记录门禁 blocked | 否 |
| 用户未给逐 run 授权 | `blocked-pending-runtime-authorization` | 否 |
| env ref 缺失或不明确 | `blocked-missing-env-authorization` | 否 |
| gateway 不可达 | `blocked-runtime-preflight` redacted summary | 否，除非用户重新授权 |
| redaction gate 失败 | `blocked-redaction`，不得落 raw payload | 否 |
| forbidden counter 非 0 | `blocked-security` | 否 |
| 用户选择手工 Windows-side 执行 | 只接收用户提供的 redacted evidence 文件 | 是，进入 CP6/CP7 evidence validation |

## 7. 文件影响范围

| 文件 / 目录 | 处理 | 说明 |
|---|---|---|
| `docs/features/cr099-runner-real-readonly-smoke/DESIGN.md` | 新增 | Feature Design |
| `docs/features/cr099-runner-real-readonly-smoke/TEST-PLAN.md` | 新增 | 测试和验证计划 |
| `docs/features/cr099-runner-real-readonly-smoke/TASKS.md` | 新增 | 任务拆分 |
| `process/stories/CR099-RUNNER-REAL-READONLY-SMOKE-LLD.md` | 新增 | Story LLD / run contract 细节 |
| `.quant-lab/evidence/qmt/cr099/redacted/<run_id>/` | 仅未来逐 run 授权后写入 | 当前 CP5 不创建、不写入 |
| runtime env / HMAC secret | 禁止读取 | 当前 CP5 不触碰 |

## 8. 下游 Story Contract

| Story | feature_design_refs | lld_policy.required_level | 理由 |
|---|---|---|---|
| `CR099-RUNNER-REAL-READONLY-SMOKE` | 本文件、TEST-PLAN、TASKS、LLD | `full-lld` | 命中真实 runtime、HMAC env、账户只读、脱敏 evidence 和安全门禁 |

## 9. Gotchas

- CP5 approve 不等于 runtime authorization；真实运行必须另有逐 run 授权。
- `authorization_ref` 只能证明用户授权链路，不得包含 secret 值。
- redacted evidence 不能保存账号、证券代码、数量、资金、委托、成交或日志原文。
- 空持仓不是失败，但只能证明 readonly path 和脱敏 schema，不证明非空持仓路径。
- 用户手工 Windows-side 执行是 fallback，不得把手工 evidence 当作 Codex 已读取 secret 或执行 runner 的证据。
