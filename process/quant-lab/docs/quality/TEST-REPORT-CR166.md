---
title: "CR166 Test Report"
status: "PASS"
version: "1.1"
cr_id: "CR-166"
executed_at: "2026-07-13T14:32:00+08:00"
closure_validated_at: "2026-07-13T15:19:46+08:00"
---

# CR166 测试报告

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-13 | host-orchestrator inline meta-qa-critical | 固化 CR166 目标、相邻、治理与全量测试结果。 |
| 1.1 | 2026-07-13 | host-orchestrator inline | 追加 CP8 批准后的关闭态 artifact-hygiene 回归、回修和最终全量结果。 |

## 范围与环境

- Python：3.11，由 `uv run --python 3.11` 执行。
- Pytest：禁用 cache provider；`PYTHONDONTWRITEBYTECODE=1`。
- 数据：只使用内存值和 `tests/fixtures/walk_forward_oos/` synthetic fixtures。
- 未执行：真实数据、历史重算、lake/NAS/provider、credential、external framework、runtime/broker/trading、publish/deploy、Git remote write。

## 最终结果

| 测试层 | Passed | Failed | Skipped/waived | 结论 |
|---|---:|---:|---:|---|
| CR166 目标 + 相邻 consumer 回归 | 78 | 0 | 0 | PASS |
| 治理 guardrail 子集 | 25 | 0 | 0 | PASS |
| CP8 关闭态治理回归 | 42 | 0 | 0 | PASS |
| Repository full suite（关闭态最终） | 1987 | 0 | 0 | PASS |

`git diff --check` 与受影响模块 `py_compile` 均通过。

## CP8 批准后关闭态验证

用户批准 CP8 后，`status-sync` 将 active CR 清空并进入 `delivered`。首次关闭态治理回归有 2 个失败，均来自同一原因：artifact-hygiene 仅把 CR166 资产分类为 active current assets，关闭后 125 个已交付过程资产变为 unclassified。该问题登记为 `GR-CR166-002`，通过新增 `closed_cr166_asset` 分类和 1 条关闭态回归测试修复。

回修后 artifact-hygiene 为 unclassified=0，六组治理测试 42/42 通过；仓库关闭态最终全量为 1987 passed、0 failed，耗时 99.87s。该修复只影响仓库治理分类，不修改 Walk-forward/OOS 业务合同、producer、projection 或 Stage claim。

## 首轮失败归因

首次全量为 1979 passed / 5 failed：

| 失败组 | 数量 | 触发路径 | 归因 | 处理 |
|---|---:|---|---|---|
| design surface | 2 | CR166 专题 HLD/ADR/Blueprint/Domain/Dependency 位于 current root | CR166 过程路由缺口 | 迁入 `process/archive/design-cr-docs/` 并更新索引 |
| process artifact hygiene | 2 | active CR 新 source/feature files 未分类 | CR166 登记缺口 | 增加 CR166 source/process asset 分类 |
| test provenance taxonomy | 1 | 7 个新 `test_*.py` 未登记 | CR166 登记缺口 | 更新 `tests/PROVENANCE.yaml` |

归因覆盖率 5/5=100%；修复后相关 25 项和最终全量均通过。这 5 项不是 CR165 的历史 14 项；最终也没有触发任何历史失败豁免。

## 关键测试断言

- 8 类 fail-closed 每类至少 1 个明确 reason；missing metric 为 typed-unavailable，非法/tamper/未授权为 blocked。
- C1 golden bytes/hash 固定，禁止以更新 golden 规避兼容问题。
- 10 次相同 normalized input 产生 1 个 component/envelope hash。
- semantic tamper 即使未来重算外层 hash，也因 metric/outcome 不一致被阻断。
- typed-unavailable 可被准确投影但无法产生 PASS；tampered component 三消费者全部 blocked。
- CR155 blocked package 保持 `paper_candidate=false`，historical backfill count=0。

## 结论

测试结论为 **PASS**。关闭态最终全量 1987/1987，新增代码路径失败为 0；结论不覆盖真实数据或 runtime readiness。
