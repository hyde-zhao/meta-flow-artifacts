---
story_id: CR101-S02-package-checker-fixture-fail-closed
change_id: CR-101
title: Package checker and fixture fail-closed realignment
status: verified
lld_policy: full-lld
wave_id: CR101-W2-CHECKER-EVIDENCE
depends_on:
  - CR101-S01-target-taxonomy-manifest-contract
owner: host-orchestrator
story_lld: process/stories/CR101-S02-package-checker-fixture-fail-closed-LLD.md
cp5_check: process/checks/CP5-CR101-S02-package-checker-fixture-fail-closed-LLD-IMPLEMENTABILITY.md
cp6_check: process/checks/CP6-CR101-S02-package-checker-fixture-fail-closed-CODING-DONE.md
cp7_check: process/checks/CP7-CR101-S02-package-checker-fixture-fail-closed-VERIFICATION-DONE.md
implementation_evidence: process/stories/CR101-S02-package-checker-fixture-fail-closed-IMPLEMENTATION.md
---

# CR101-S02 Package checker and fixture fail-closed realignment

## 用户价值

让 CR100 fake package exchange 与本地 checker 按 CR101 contract fail closed，避免旧 `qmt_terminal` + `miniqmt_runner` 硬编码继续污染策略包交付语义。

## 范围

- 调整 `package_exchange.py` 中 target / entrypoint 验证逻辑。
- 增加或更新本地 fake package fixtures。
- 增加 8 类负向校验。

## 验收标准

| AC | 描述 |
|---|---|
| S02-AC-01 | checker 覆盖 schema、target missing、adapter missing、permission nonfalse、checksum、path escape、sensitive filename、forbidden counter。 |
| S02-AC-02 | 历史 `miniqmt_runner` delivery target 被阻断并提示迁移。 |
| S02-AC-03 | fake exchange 不访问真实 NAS、不 publish、不读取凭据。 |
| S02-AC-04 | tests 只使用临时目录、脱敏 fixture 和本地文件。 |

## 文件所有权

| 类型 | 文件 |
|---|---|
| primary | `trading/strategy_runner/package_exchange.py`、`tests/test_cr100_package_exchange.py` |
| shared | package fixtures、CR101 checker tests |
| forbidden | 真实 NAS、publish、mount、凭据、账户 / 持仓 / 委托 / 成交 / 原始日志 |

## 测试设计

- CR100 package exchange 回归。
- CR101 negative fixture matrix。
- path traversal / sensitive filename / forbidden counter tests。
- checksum mismatch tests。
