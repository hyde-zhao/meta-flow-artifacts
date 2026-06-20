---
story_id: CR101-S03-runner-adapter-evidence-boundary
change_id: CR-101
title: Runner adapter and evidence boundary realignment LLD
status: implemented-and-verified
created_at: "2026-06-20T09:20:00+08:00"
owner: host-orchestrator
source_hld: docs/qmt/CR101-CROSS-PLATFORM-STRATEGY-DELIVERY-ADAPTER-REALIGNMENT-HLD.md
cp6_check: process/checks/CP6-CR101-S03-runner-adapter-evidence-boundary-CODING-DONE.md
cp7_check: process/checks/CP7-CR101-S03-runner-adapter-evidence-boundary-VERIFICATION-DONE.md
---

# CR101-S03 Runner adapter and evidence boundary realignment LLD

## 1. 目标

在 runner adapter、readonly gateway 和 evidence summary 中显式记录 CR101 双边界，保持未授权真实 gateway fail closed，并阻断敏感 evidence。

## 2. 文件影响范围

| 文件 | 操作 | 说明 |
|---|---|---|
| `trading/strategy_runner/adapters.py` | 修改 | adapter payload / registry 识别 `delivery_target_id` 与 `execution_adapter_id`。 |
| `trading/strategy_runner/evidence.py` | 修改 | evidence summary 增加 target / adapter 字段和 redaction guard。 |
| `trading/strategy_runner/readonly_gateway.py` | 小幅修改或不改 | 保持 allowlist；若增加 contract metadata，不能扩大 runtime 权限。 |
| `tests/test_cr091_strategy_runner_contracts.py` | 修改 / 新增用例 | adapter contract、forbidden counters、evidence redaction。 |
| `tests/test_cr098_runner_readonly_integration.py` | 回归 | readonly gateway fake transport 和授权缺失阻断。 |

禁止：SDK import、runtime start/connect、真实 gateway 调用、账户 / 持仓 / 委托 / 成交 / 原始日志读取。

## 3. 数据结构

| 字段 | 所属对象 | 规则 |
|---|---|---|
| `delivery_target_id` | adapter payload / evidence | 当前为 `qmt_terminal_direct`。 |
| `execution_adapter_id` | adapter payload / evidence | 当前为 `miniqmt_gateway_readonly` 或等价常量。 |
| `execution_adapter_capabilities` | adapter payload / evidence | 只允许 readonly 集合。 |
| `authorization_ref` | gateway runtime config / evidence | 缺失时真实 gateway 调用 blocked。 |
| `forbidden_operation_counters` | evidence | 所有值必须为 0。 |
| `sensitive_field_hits` | evidence | 必须为 0。 |

## 4. 接口契约

`AdapterRegistry`：

- 输入：adapter id。
- 输出：已注册 adapter 或明确错误。
- 不根据 `delivery_target_id` 启动 runtime。

`EvidenceSummary`：

- 输入：adapter result、package metadata、redaction scan result。
- 输出：脱敏 summary。
- 若 sensitive hit > 0 或 forbidden counter > 0，验证应失败。

`ReadonlyGatewayClient`：

- 保持 `health`、`capabilities`、`query_positions` allowlist。
- 缺 runtime authorization 时返回 blocked，不连接真实 gateway。

## 5. 控制流

1. loader 输出 CR101 adapter payload。
2. runner 根据 `execution_adapter_id` 找 adapter。
3. fake adapter 或 readonly gateway fake transport 执行离线 / fake 路径。
4. evidence 汇总 target / adapter / authorization false flags / forbidden counters。
5. redaction scan 通过后写入 summary。

## 6. 测试设计

- adapter payload 包含 target / adapter 字段。
- unknown adapter fail closed。
- readonly gateway 未授权真实 runtime blocked。
- evidence sensitive hit 阻断。
- forbidden counter 非零阻断。
- CR098 fake transport 回归通过。

## 7. 实施步骤

1. 从 S01 共享 CR101 常量。
2. 扩展 adapter payload dataclass / dict contract。
3. 扩展 evidence summary schema version 或追加兼容字段。
4. 保持 readonly gateway allowlist，不加入 submit/cancel/order-write。
5. 增加 contract / regression tests。

## 8. 风险与回退

| 风险 | 缓解 |
|---|---|
| evidence schema 变更影响历史 tests | 支持旧字段读入，新字段在 CR101 output 必填。 |
| readonly gateway 被误扩展为交易 adapter | allowlist 测试固定，不允许 order-write capability。 |
| adapter_id 与 target_id 混用 | tests 强制两个字段同时存在且值不同语义。 |

回退：若 evidence schema 变更过大，先用 `metadata.cr101` 扩展块承载新字段，CP8 标注后续 schema cleanup。

## 9. 不授权项

本 LLD 不授权真实 runtime、gateway connect、账户数据读取、交易、simulation/live、NAS 或 publish。
