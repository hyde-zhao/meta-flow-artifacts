---
title: "CR-168 验证报告"
status: "cp7-complete-with-risk"
version: "1.1"
cr_id: "CR-168"
validation_mode: "static-only"
created_at: "2026-07-14T14:55:00+08:00"
owner: "host-orchestrator inline meta-qa"
---

# CR-168 验证报告

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-14 | host-orchestrator inline meta-qa | 完成五个 Story、fixture/static-only 合同、全仓回归与失败归因验证；结论为 PASS_WITH_RISK。 |
| 1.1 | 2026-07-14 | host-orchestrator inline meta-qa | §4、§5：补充状态压缩审计归档的 hygiene 分类整改，并在最终门禁前重跑全仓 suite。 |

## 1. 验证范围与授权边界

本报告验证 CR-168 的 fixture/static-only C3 Economic Cost / Slippage / transparent Impact approximation foundation。验证对象为 `economic_cost@v1`、其 C2 envelope compatibility、局部 C3→Gate4 projection、两类 synthetic fixture、QAC/authorization/CR155 regression，以及其仓库治理登记。

未执行且不被本报告授权：真实数据/lake/NAS/provider/credential、真实 TCA 或 market-impact calibration、C4 calculator/capacity、canonical Gate4 修改、aggregate integration、runtime/broker/QMT/simulation/paper/live、catalog/store/registry write、publish/deploy/tag/release 或 Git remote write。

## 2. 验证对象清单

| Story | 对象 | CP6 | CP7 | 结论 |
|---|---|---:|---:|---|
| S01 | 9-family input、normalization、validation、subject-neutral component identity | PASS | PASS_WITH_RISK | 10 类输入错误 fail-closed。 |
| S02 | Decimal 五分项、quantization、gross-to-net reconciliation | PASS | PASS_WITH_RISK | static square-root only；无真实 TCA。 |
| S03 | `economic_cost@v1` catalog activation 与 envelope attachment | PASS | PASS_WITH_RISK | component hash subject-neutral，envelope hash 保留 subject identity。 |
| S04 | C3 local Gate4 projection guard | PASS | PASS_WITH_RISK | C4 absent/无 reason 时 canonical BLOCKED；canonical/aggregate 修改=0/0。 |
| S05 | fixtures、QAC、authorization、CR155/full-suite attribution | PASS | PASS_WITH_RISK | full suite 2077 passed；初始治理遗漏均已修复。 |

## 3. 需求、场景与设计契约追踪

| 追踪面 | 已验证精确值 | 主要证据 |
|---|---:|---|
| requirements | 9/9 | `test_economic_cost_cr168_qac.py`、S05 CP7 return。 |
| scenarios | 17/17，其中 P0=16、P1=1 | 同上；N01..N10、B01、B02、A01、G01、E01 均有具名断言。 |
| QAC | 15/15 | 同上。 |
| P0 input fail-closed | 10/10 | S01/S05：gross、trade/notional、model、finite、negative、basis、currency/calendar、arithmetic、lineage/auth、hash tamper。 |
| deterministic identity | 10 runs → 1 component hash | S05 QAC；daily/ML 同 families 2–9 得同 component hash，不同 subject 得不同 envelope hash。 |
| fixture families | 2/2 | daily multifactor synthetic；multi-strategy-type compatibility。 |
| Gate4 containment | B01/B02=2/2；reason escape=8/8 blocked | S04；C4 三字段 absent，candidate-release 下精确三项 missing claim。 |
| authorization/registry | forbidden operations=0；registry persistent/parallel write=0 | S05 authorization；registry missing 为 existing Feature/module refs 的 N/A-with-reason。 |
| non-promotion | C4/capacity/aggregate PASS=0；CR155 promotion=0 | S04/S05；CR155=`BLOCKED` 且 `paper_candidate=false`。 |

## 4. 分层验证执行结果

| 层 | 命令/对象 | 结果 | 说明 |
|---|---|---|---|
| contract/unit | C3 input、validator、reason codes、identity | PASS | 缺失或无效输入短路 calculator。 |
| calculation | Decimal five components、rounding、reconciliation | PASS | `square_root` v1，显式 static values only。 |
| compatibility | C2 envelope / catalog / daily-ML attachment | PASS | 未创建平行 envelope/registry。 |
| security integration | Gate4 B01/B02、denylist、public DI double | PASS | no private helper/monkeypatch；canonical/aggregate source diff=0/0。 |
| targeted regression | C3/C2/S05 pytest | PASS | `98 passed in 0.33s`。 |
| governance regression | design surface、artifact hygiene、test taxonomy | PASS | `5 passed in 0.20s`；design errors=0；hygiene unclassified=0。 |
| repository final suite | `uv run --python 3.11 pytest -q --cache-clear` | PASS | 首次最终回归 `2077 passed in 97.67s`；卫生分类整改后重跑 `2077 passed in 99.34s`。 |

## 5. 首次全仓失败归因与整改

首次全仓 suite 记录到 5 个失败，均判定 `introduced_by_cr168=true`，原因是新交付资产未登记到现有仓库治理，而非生产逻辑、计算结果或既有测试回归：

| 初始失败组 | 数量 | 根因 | 整改 | 最终证据 |
|---|---:|---|---|---|
| CR131 design surface | 2 | 五份 CR-named CP3 design 正文仍位于 current design root | 移入 `process/archive/design-cr-docs/`，更新 current index/Feature refs/archive index | 定向 2/2 PASS。 |
| CR132 artifact hygiene | 2 | CR-168 source/process assets 未纳入 active-CR 分类 | 登记 CR168 source/process asset set | 定向 2/2 PASS，unclassified=0。 |
| test provenance taxonomy | 1 | 七个 C3 测试没有 `tests/PROVENANCE.yaml` 条目 | 为 S01–S05 测试写入 CR/Story provenance | 定向 1/1 PASS。 |

最终 full-suite failures=0，unknown attribution=0。该整改没有改变 `engine/cross_strategy_reliability_gates.py`、`engine/strategy_admission_package.py` 或任何真实操作边界。

补充整改：状态 v2 slim 会生成 `archive/state/*/{archived-fields,slim-report}.json` 审计文件；卫生检查将其错误归为未分类。已将这类固定形态归入共享工作流资产，`test_process_artifact_hygiene.py=21 passed`，并完成上述最终全仓重跑。该修复只影响过程审计分类，不改变任何 C3 算术、Gate4 canonical/aggregate 或授权边界。

## 6. Claim ceiling 验证

| 声明 | 验证值 |
|---|---:|
| `stage2_complete` | true（既有阶段事实；本 CR 不重新解释）。 |
| `stage3_started` | false |
| `c3_fixture_static_foundation` | true |
| `real_tca_available` / `real_market_impact_calibrated` / `real_data_connected` | false / false / false |
| `runtime_ready` | false |
| `c4_calculators` / `event_specific_producer` | 0 / false |
| `cr155_promoted` | false |

## 7. 剩余风险与阶段决定

| 风险 | 严重度 | 状态 | CP8 处理 |
|---|---|---|---|
| R-CR168-VERIFIER-INDEPENDENCE | 低 | 用户批准的 inline CP7，未伪造独立验证 | 必须披露；使交付 readiness 为 `READY_WITH_RISK`。 |
| R-CR168-GATE4-C3-C4-SEMANTIC | 高 | S04 只对本地 adapter 封闭 C4 N/A reason escape；不代表 canonical 所有 direct caller 均已修复 | CP8 说明局部性；FU-007/独立 remediation 处理全局/aggregate。 |
| R-CR168-TRUE-TCA-OVERCLAIM | 高 | 由 static-only claim ceiling 控制 | CP8 重申没有真实 TCA、校准或数据连接。 |

**阶段决定：PASS_WITH_RISK。** 所有可执行验证均通过；风险为明确、非阻断且不扩大授权的残余项。下一步只能准备 CP8 人工交付就绪门禁。
