---
story_id: CR101-S02-package-checker-fixture-fail-closed
change_id: CR-101
title: Package checker and fixture fail-closed realignment LLD
status: implemented-and-verified
created_at: "2026-06-20T09:20:00+08:00"
owner: host-orchestrator
source_hld: docs/qmt/CR101-CROSS-PLATFORM-STRATEGY-DELIVERY-ADAPTER-REALIGNMENT-HLD.md
cp6_check: process/checks/CP6-CR101-S02-package-checker-fixture-fail-closed-CODING-DONE.md
cp7_check: process/checks/CP7-CR101-S02-package-checker-fixture-fail-closed-VERIFICATION-DONE.md
---

# CR101-S02 Package checker and fixture fail-closed realignment LLD

## 1. 目标

迁移 CR100 fake exchange / package checker / fixtures，使其按 CR101 target + adapter contract 校验，并对高风险异常 fail closed。

## 2. 文件影响范围

| 文件 | 操作 | 说明 |
|---|---|---|
| `trading/strategy_runner/package_exchange.py` | 修改 | 替换 `qmt_terminal` + `miniqmt_runner` 硬编码 target 验证。 |
| `tests/test_cr100_package_exchange.py` | 修改 / 新增用例 | 保留 CR100 fake exchange 回归，新增 CR101 negative matrix。 |
| `tests/test_cr091_strategy_runner_contracts.py` | 协调修改 | 复用 S01 contract fixture。 |
| `packages/` 或测试临时 fixture | 修改 / 新增 | 仅允许脱敏本地 fixture；不得引用真实 NAS。 |

禁止：真实 NAS list/read/write/copy/publish/pull/delete、mount、凭据读取、账户数据读取。

## 3. Checker 规则

| 失败类 | 触发条件 | 预期 |
|---|---|---|
| schema | schema version 缺失或不支持 | fail closed |
| target missing | 无 implemented `qmt_terminal_direct` | fail closed |
| adapter missing | 无 MiniQMT readonly adapter contract 或 adapter capability 非 readonly | fail closed |
| permission nonfalse | 任一 authorization flag 为 true | fail closed |
| checksum | checksum 缺失或不匹配 | fail closed |
| path escape | entrypoint / payload 路径越过包根 | fail closed |
| sensitive filename | 文件名含 `.env`、credential、secret、token、account、raw_orders、raw_positions、qmt_log 等 | fail closed |
| forbidden counter | 任一 forbidden operation counter 非 0 | fail closed |

## 4. 控制流

1. fake exchange 在本地临时目录创建 package。
2. checker 读取 manifest 和 checksum。
3. 调用 S01 manifest validator。
4. 逐项校验路径、文件名、checksum、forbidden counters。
5. 返回 PASS 或结构化错误列表。

## 5. 测试设计

- `test_cr101_package_checker_accepts_qmt_direct_target_and_miniqmt_readonly_adapter`
- `test_cr101_package_checker_rejects_legacy_miniqmt_runner_delivery_target`
- `test_cr101_package_checker_rejects_missing_target`
- `test_cr101_package_checker_rejects_missing_adapter`
- `test_cr101_package_checker_rejects_permission_nonfalse`
- `test_cr101_package_checker_rejects_checksum_mismatch`
- `test_cr101_package_checker_rejects_path_escape`
- `test_cr101_package_checker_rejects_sensitive_filename`
- `test_cr101_package_checker_rejects_forbidden_counter`

全部测试必须使用 `tmp_path` 或等价临时目录，不读取真实 NAS / `.env`。

## 6. 实施步骤

1. 从 S01 引用 CR101 target / adapter 常量。
2. 更新 `_validate_manifest_shape()` 的 required target / entrypoint 逻辑。
3. 增加 sensitive filename 扫描。
4. 增加 forbidden counter 聚合校验。
5. 更新 CR100 fake exchange 正向 fixture。
6. 增加 8 类负向测试。

## 7. 风险与回退

| 风险 | 缓解 |
|---|---|
| CR100 历史测试大面积变更 | 保留 fake exchange 流程，只替换 manifest contract。 |
| sensitive filename 误杀普通文件 | 规则先限定高风险关键词和扩展名，并在测试中覆盖允许文件。 |
| checker 与 loader 校验重复 | checker 负责包完整性，loader 负责 contract；共享常量但错误信息分层。 |

回退：若 CR100 fake exchange 迁移范围过大，保留 CR100 schema 读入兼容层，新 CR101 fixture 单独创建，并在 CP7 标注兼容风险。

## 8. 不授权项

本 LLD 不授权真实 NAS、凭据、QMT/MiniQMT/XtQuant/gateway runtime、simulation/live、交易或 publish。
