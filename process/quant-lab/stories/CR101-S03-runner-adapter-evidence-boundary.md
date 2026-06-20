---
story_id: CR101-S03-runner-adapter-evidence-boundary
change_id: CR-101
title: Runner adapter and evidence boundary realignment
status: verified
lld_policy: full-lld
wave_id: CR101-W2-CHECKER-EVIDENCE
depends_on:
  - CR101-S01-target-taxonomy-manifest-contract
owner: host-orchestrator
story_lld: process/stories/CR101-S03-runner-adapter-evidence-boundary-LLD.md
cp5_check: process/checks/CP5-CR101-S03-runner-adapter-evidence-boundary-LLD-IMPLEMENTABILITY.md
cp6_check: process/checks/CP6-CR101-S03-runner-adapter-evidence-boundary-CODING-DONE.md
cp7_check: process/checks/CP7-CR101-S03-runner-adapter-evidence-boundary-VERIFICATION-DONE.md
implementation_evidence: process/stories/CR101-S03-runner-adapter-evidence-boundary-IMPLEMENTATION.md
---

# CR101-S03 Runner adapter and evidence boundary realignment

## 用户价值

让 runner adapter 和 evidence 显式区分策略交付 target 与执行 adapter，防止 MiniQMT gateway adapter 被误读为 runner host 或 order-write 授权。

## 范围

- adapter payload / registry 设计补充 `delivery_target_id` 与 `execution_adapter_id`。
- evidence summary 记录 target / adapter taxonomy，并维持敏感信息命中 0。
- readonly gateway 保持 fake transport 默认路径和未授权 fail closed。

## 验收标准

| AC | 描述 |
|---|---|
| S03-AC-01 | evidence 能表达 target + adapter 双边界。 |
| S03-AC-02 | readonly gateway allowlist 不扩大到 order-write。 |
| S03-AC-03 | 缺 runtime authorization_ref 时仍阻断真实 gateway 调用。 |
| S03-AC-04 | sensitive field hits 和 forbidden operation counters 均为 0。 |

## 文件所有权

| 类型 | 文件 |
|---|---|
| primary | `trading/strategy_runner/adapters.py`、`trading/strategy_runner/evidence.py`、`trading/strategy_runner/readonly_gateway.py` |
| shared | `tests/test_cr091_strategy_runner_contracts.py`、`tests/test_cr098_runner_readonly_integration.py` |
| forbidden | QMT/MiniQMT/XtQuant runtime import、gateway start/connect、账户/持仓/委托/成交/原始日志 |

## 测试设计

- adapter payload contract tests。
- evidence redaction tests。
- CR098 readonly gateway regression。
- forbidden operation counter nonzero negative tests。
