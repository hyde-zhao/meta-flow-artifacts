# CP2-CR060 Execution Baseline 人工检查点

## 自动预检摘要

自动预检结论：PASS。CR060 可作为 CR059 后续正式执行门禁启动，但当前只冻结 repo-local 文档身份收敛范围，不直接执行真实改写。

## Decision Brief

推荐决策：接受 CR060 baseline，使用最小真实迁移切片推进，即 README / USER-MANUAL future-facing identity convergence。备选方案 A 是继续只做文档门禁不执行任何真实改写；备选方案 B 是把包名、Git remote、NAS、data lake 同时纳入一个大 CR。推荐方案优点是风险可控、可快速验证；备选 A 过于保守，不能推进真实迁移；备选 B 覆盖完整但失败面过大，回滚复杂。

回退 / 切换条件：如果用户要求先完成 Git remote，则暂停 CR060 并启动 CR062；如果用户要求先迁移 NAS 数据，则暂停 CR060 并启动 CR063；如果用户要求包名和 import 一起改，则转 CR061。

Decision Collection Coverage：已扫描 `CR058-CANDIDATE-LIST.md`、`CR058-PRESERVE-AUDIT-ALLOWLIST.md`、`CR058-ROLLBACK-GATE.md`、`MIGRATION-MANIFEST-SEED-CR059.yaml`、CR059 CP8 和当前 STATE。候选问题数 6，纳入待决策数 6，N/A 0。

待人工决策清单：

| 决策 ID | 决策类型 | 问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP2-CR060-01 | scope | 是否创建 CR060 作为 CR059 后续正式执行门禁？ | 创建 CR060，来源追溯 CR059。 | 保持为 follow-up candidate。 | 推荐方案可推进真实迁移；备选继续停留在计划。 | 创建 CR 不等于立即执行改写。 | 用户只要候选不执行时切换备选。 |
| DQ-CP2-CR060-02 | scope | 第一切片是否限定 README / USER-MANUAL？ | 限定为 future-facing 用户文档身份收敛。 | 纳入包名、import、Git remote、NAS 和 lake。 | 推荐方案风险小；备选覆盖大但回滚难。 | 后续 CR 仍需继续完成完整迁移。 | 用户要求一次性大迁移时转多 CR 并重新门禁。 |
| DQ-CP2-CR060-03 | risk_acceptance | 是否接受历史审计默认 preserve-audit？ | 接受 `process/**` 等历史证据不改写。 | 单项豁免。 | 推荐方案保护审计链；豁免可精确但需额外门禁。 | 历史文档仍出现 `local_backtest`。 | 具体外发文档需要改写时走豁免。 |
| DQ-CP2-CR060-04 | implementation | 是否保留 `MARKET_DATA_LAKE_ROOT`？ | 保留，不在 CR060 改名或切换 root。 | 改成 `QUANT_LAB_*`。 | 推荐方案避免破坏数据入口；备选需 CR064。 | 数据湖身份收敛延后。 | 用户要求 root replacement 时启动 CR064。 |
| DQ-CP2-CR060-05 | runtime_authorization | 是否确认 CR060 不授权 Git remote / push？ | 确认不授权，后续 CR062。 | 本 CR 内完成 remote add/push。 | 推荐方案保持边界；备选会混合发布风险。 | 远端初始化延后。 | 用户明确启动 Git cutover 时进入 CR062。 |
| DQ-CP2-CR060-06 | runtime_authorization | 是否确认不恢复 CR046？ | 不恢复 CR046。 | 合并恢复 CR046。 | 推荐方案避免交易 runtime 误启动；备选与暂停状态冲突。 | CR046 继续暂停。 | 用户明确恢复 CR046 时另行处理。 |

不授权项：未通过后续 CP5 前，不授权真实 text rewrite；任何情况下本 CP2 不授权 NAS 数据迁移、data lake root 切换、Git remote/push、凭据读取、QMT/MiniQMT runtime 或历史审计 rewrite。

auto_final_authorization: false

## Entry Criteria

| 条件 | 结果 |
|---|---|
| CR059 关闭且证据存在 | PASS |
| CR058 输入存在 | PASS |
| CR046 paused | PASS |

## Checklist

| 检查项 | 结果 |
|---|---|
| 影响分析完成 | PASS |
| 决策项完整 | PASS |
| 不授权项列明 | PASS |

## Exit Criteria

| 条件 | 结果 |
|---|---|
| 用户确认后可进入 CP3/CP5 | PENDING |

## Deliverables

- `process/checks/CP2-CR060-EXECUTION-BASELINE.md`
- `process/checkpoints/CP2-CR060-EXECUTION-BASELINE.md`

## 人工审查结果

待用户审查。
