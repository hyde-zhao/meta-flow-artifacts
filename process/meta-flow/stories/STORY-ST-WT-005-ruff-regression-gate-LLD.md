---
story_id: "ST-WT-005"
title: "收敛 Ruff 并建立完整回归质量门"
story_slug: "ruff-regression-gate"
lld_version: "1.0"
tier: "L"
status: "approved"
confirmed: true
created_by: "host-orchestrator-inline/meta-dev"
created_at: "2026-07-14"
confirmed_by: "user"
confirmed_at: "2026-07-15T12:05:17Z"
feature_design_refs: ["process/docs/features/cr047-delivery/DESIGN.md", "process/docs/features/cr047-delivery/TEST-PLAN.md", "process/docs/features/cr047-delivery/TASKS.md"]
lld_policy: {required_level: "full-lld", trigger_reasons: ["repo-wide-lint", "regression", "shared-preflight"]}
open_items: 0
---

# LLD: ST-WT-005 — Ruff Regression Gate

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-14 | host-orchestrator-inline/meta-dev | 冻结串行 merge、Ruff 零错误与五门组合 preflight。 |

## 0. 工程依据与上游设计依据

工程依据为 REQ-WT-011/012/017、ADR-WT-004、Feature delivery design 和当前 Ruff 90 findings 基线。ST-WT-003/004 会修改共享 Python/guardrail，故本 Story 实现必须串行等待两者合并。

## 1. 目标

将 Ruff errors 从基线 90 收敛为 0，保持既有 377 tests + 70 subtests 不回退，并把 pytest、Ruff、delivery guardrail、Doctor、CR tracking 组合成能保留每门退出码的发布 preflight。

## 2. 需求（Functional / Non-Functional）

- 先运行 Ruff 记录 code/file 分布；安全 `--fix` 与人工 B/F 修复分批进行。
- 不使用 ignore/noqa 批量掩盖、不减少测试、不用语义不明的自动修复。
- ST-WT-003/004 合并前 dev gate关闭；合并后先复核共享 diff，再成为最终 merge owner。
- 五个 gate 全运行、分别记录命令/退出码；一个失败整体非零但不隐藏其他门结果。
- Python 命令统一 `uv run --python 3.11`；测试不写 bytecode/cache。

## 3. 模块拆分与职责

| 模块 | 职责 |
|---|---|
| `pyproject.toml` | Ruff target/rules与项目工具配置，不无故扩大 ignore |
| `meta_flow/**`, `tests/**` | 按 finding 做最小机械/人工修复 |
| `scripts/check_delivery_guardrails.py` | 复用发布 preflight编排或提供组合入口 |
| `tests/test_cr047_delivery_gate.py` | gate composition、失败聚合、命令 contract fixture |

## 4. 代码结构与文件影响范围

可能触及 repo-wide Python import/modernize/finding 行；`pyproject.toml` 和 delivery gate test 为 primary。共享文件必须基于 ST-WT-003/004 已合并版本，禁止覆盖其逻辑。源码改动将在 CP5 批准后另行记录逐文件清单。

## 5. 数据模型与持久化设计

`GateRun(name, command, exit_code, result_ref, status)`；`PreflightResult` 含五个 gate runs、overall status、started/finished。Ruff finding 以 code/path/line 分类。结果作为 CP6 evidence/ref，不建 CI 状态真相；命令退出码原样保留。

## 6. API / Interface 设计

组合入口调用现有 CLI/脚本：pytest、`ruff check .`、guardrail、`doctor all`、`check cr-tracking`。可实现为既有 guardrail扩展或仓库级最小 preflight函数，但不得创建 release orchestrator服务。输出人类摘要+机器 result refs。

## 7. 核心处理流程

1. 确认 ST-WT-003/004 upstream merged且 tests通过。
2. 保存 Ruff基线；执行仅安全自动修复，审查 diff与定向 tests。
3. 人工处理剩余 B/F findings，每批回归。
4. 运行全量 pytest与 Ruff，确认零错误/无基线回退。
5. 顺序运行五门并聚合退出码；任一失败整体 FAIL，不提前停止。

## 8. 技术细节与设计细节

I001/UP017等机械项可自动修复但必须 diff审查；F401/F841/B007/B033/B904 逐项理解语义。timezone modernize 保持 aware UTC。preflight 使用显式环境 `PYTHONDONTWRITEBYTECODE=1`、pytest no-cache，避免本身制造 cache blocker。

## 9. 安全与性能设计

不运行外部网络/凭据/生产写，不自动 commit/push。全量门预计为分钟级但无新 SLA；每门超时/异常均记录专用失败，不把未执行标为 PASS。命令使用固定 argv，禁止 shell 注入参数。

## 10. 测试设计

| 场景 | 预期 |
|---|---|
| Ruff full repo | errors=0 |
| 既有+新增 pytest | >=377 tests + 70 subtests，全部通过 |
| 某一 gate失败 | overall FAIL，其他门结果仍可见 |
| 五门成功 | overall PASS，5/5 refs/exit=0 |
| 自动修复 diff | 无 ignore扩大、无公共行为变化 |
| ST-WT-003/004 未合并 | dev gate拒绝 |

## 11. 实施步骤

1. `TASK-WT-005-01`：锁定上游合并状态和 Ruff基线；分批修复90项。
2. `TASK-WT-005-02`：接入五门组合 preflight与结构化结果。
3. `TASK-WT-005-03`：运行定向/全量回归，输出 Implementation/Return/Evidence。
4. 发现语义设计问题时停止自动修复并回 CP5 澄清。

## 12. 风险、难点与预研建议

风险是机械修复改变行为、跨 Story覆盖、Doctor在实现中仍因未分类 active artifact失败。通过小批次diff、串行merge、B0分类解释解决；Doctor非零不得被忽略，只能形成 blocker或已批准warning。

## 13. 回滚与发布策略

每类修复保持可独立回滚；出现回归只撤相关批次，不降低规则/测试。preflight接线可回滚为独立命令列表但证据保留。无 commit/push授权。

## 14. DoD（Definition of Done）

- [ ] 0–14 章节完整、`open_items=0`。
- [ ] Ruff errors=0；测试不低于377+70且新增 tests全通过。
- [ ] 五门逐项有命令、exit code、result ref，整体规则正确。
- [ ] ST-WT-003/004改动未被覆盖，测试/规则无放宽。
- [ ] CP5 未批准前 `confirmed=false`，不执行 `ruff --fix` 或源码实现。

## 人工确认区

- 结论：`approved`
- 审查人：user
- 审查时间：2026-07-15T12:05:17Z
- 修改意见：
- 风险接受项：inline fallback；CP7/CP8 风险上限。
