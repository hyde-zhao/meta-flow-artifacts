---
story_id: CR101-S01-target-taxonomy-manifest-contract
change_id: CR-101
title: Target taxonomy and manifest contract realignment LLD
status: implemented-and-verified
created_at: "2026-06-20T09:20:00+08:00"
owner: host-orchestrator
source_hld: docs/qmt/CR101-CROSS-PLATFORM-STRATEGY-DELIVERY-ADAPTER-REALIGNMENT-HLD.md
cp6_check: process/checks/CP6-CR101-S01-target-taxonomy-manifest-contract-CODING-DONE.md
cp7_check: process/checks/CP7-CR101-S01-target-taxonomy-manifest-contract-VERIFICATION-DONE.md
---

# CR101-S01 Target taxonomy and manifest contract realignment LLD

## 1. 目标

将策略包 manifest 与 adapter payload 从历史平台名混用改为双边界：

- `delivery_targets[]`：策略交付 target，当前唯一 implemented target 是 `qmt_terminal_direct`。
- `execution_adapters[]`：runner 到 broker / gateway 的 adapter，当前 MiniQMT 只作为 readonly gateway / broker adapter contract。
- `miniqmt_runner` 不再是 delivery target；如果历史 fixture 或 manifest 出现该值，后续 checker 必须 fail closed。

## 2. 文件影响范围

| 文件 | 操作 | 说明 |
|---|---|---|
| `trading/strategy_runner/package_loader.py` | 修改 | 新增 manifest contract 常量、target / adapter schema 校验和 adapter payload 字段。 |
| `tests/test_cr091_strategy_runner_contracts.py` | 修改 / 新增用例 | 覆盖 manifest contract、authorization false flags、legacy target blocked。 |
| `tests/test_cr100_package_exchange.py` | 协调修改 | S02 会消费 S01 contract 名称；S01 只提供常量 / fixture 约定。 |
| `docs/qmt/CR101-CROSS-PLATFORM-STRATEGY-DELIVERY-ADAPTER-REALIGNMENT-HLD.md` | 不改或小幅同步 | 若实现命名偏离 HLD，必须回写偏差。 |

禁止修改 / 读取：

- `.env`、凭据、账号、账户、持仓、委托、成交、原始日志。
- 真实 NAS 路径和 runtime launcher。
- QMT / MiniQMT / XtQuant SDK runtime import。

## 3. 数据模型

推荐 schema 字段：

| 字段 | 类型 | 规则 |
|---|---|---|
| `schema_version` | string | 建议 `cr101-strategy-runner-package-manifest-v1`。 |
| `delivery_targets` | list[object] | 至少 1 项；implemented target 只能有 `qmt_terminal_direct`。 |
| `delivery_targets[].target_id` | string | 当前允许 `qmt_terminal_direct`；future slot 可为 `goldminer_future` / `generic_python_future` 且 `implemented=false`。 |
| `delivery_targets[].implemented` | bool | `qmt_terminal_direct=true`；future slot 必须 false。 |
| `delivery_targets[].entrypoint` | string | 本地包内相对路径；不得 path escape。 |
| `execution_adapters` | list[object] | 至少包含 MiniQMT gateway adapter contract，非 delivery target。 |
| `execution_adapters[].adapter_id` | string | 建议 `miniqmt_gateway_readonly`。 |
| `execution_adapters[].capabilities` | list[string] | 只允许 readonly capability；不包含 order-write。 |
| `not_authorization` | object | runtime / NAS / credential / account / trade / publish flags 必须 false。 |
| `forbidden_operation_counters` | object | 所有计数必须为 0。 |

## 4. 接口契约

`validate_manifest()`：

- 输入：manifest dict / path。
- 输出：通过时返回规范化 metadata；失败时抛出 `PackageValidationError`。
- 失败路径：缺 schema、缺 target、缺 adapter、legacy target、authorization true、path escape、forbidden counter 非零。

`StrategyPackage.to_adapter_payload()`：

- 输出应包含 `delivery_target_id`、`execution_adapter_id`、schema version、checksum、authorization false flags、strategy payload。
- 不连接 runtime，不读取凭据。

## 5. 控制流

1. 读取本地 manifest。
2. 校验 schema version。
3. 校验 `delivery_targets[]` 中 implemented target 数量等于 1，且 id 为 `qmt_terminal_direct`。
4. 校验 future target 若存在则 `implemented=false`。
5. 校验 `execution_adapters[]` 至少包含 MiniQMT readonly gateway adapter contract。
6. 校验 authorization false flags 与 forbidden counters。
7. 生成 adapter payload。

## 6. 异常处理

| 异常 | 行为 |
|---|---|
| legacy `miniqmt_runner` delivery target | fail closed，提示迁移到 `execution_adapters[]`。 |
| `runtime_authorized=true` 或同类权限 true | fail closed。 |
| future target `implemented=true` | fail closed。 |
| entrypoint path escape | fail closed。 |
| checksum 缺失或不匹配 | 交给 S02 checker fail closed。 |

## 7. 测试设计

- 正向：`qmt_terminal_direct` + `miniqmt_gateway_readonly` contract 通过。
- 负向：`miniqmt_runner` target 阻断。
- 负向：future target `implemented=true` 阻断。
- 负向：authorization flags true 阻断。
- 静态：runner core 不直接 import `xtquant` / QMT SDK。

## 8. 实施步骤

1. 在 `package_loader.py` 引入 CR101 schema 常量和允许 target / adapter 常量。
2. 扩展 manifest validator。
3. 扩展 adapter payload 字段，并保持旧字段兼容或提供明确迁移错误。
4. 增加 contract tests。
5. 与 S02 / S03 共享常量命名，避免重复定义漂移。

## 9. 风险与回退

| 风险 | 缓解 |
|---|---|
| 旧测试仍依赖 `adapter_type` 或 `miniqmt_runner` | 保留兼容映射仅用于读旧 fixture；新 manifest 不允许旧 delivery target。 |
| future slot 被误读为 ready | future target 必须 `implemented=false`，checker 阻断 true。 |
| schema 改动影响 CR100 | S02 在 checker / fixture 层统一迁移，S01 只冻结 contract。 |

回退：若实现成本超出范围，回退到 CP5，将 S01 拆成 schema-only 与 payload migration 两个 Story。

## 10. 不授权项

本 LLD 不授权 NAS、凭据、账户 / 持仓 / 委托 / 成交 / 原始日志、QMT/MiniQMT/XtQuant/gateway runtime、simulation/live、交易或 publish。
