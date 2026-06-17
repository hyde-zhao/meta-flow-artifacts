# CP5-CR060 Execution Readiness 人工检查点

## 自动预检摘要

自动预检结论：PASS_WITH_RISK。CR060 的 execution manifest 已冻结，当前只等待人工授权。用户回复 `approve` 后，授权范围仅限 `README.md` 和 `docs/USER-MANUAL.md` 的 future-facing 项目身份文本改写；不授权任何文件移动、删除、Git remote、NAS、data lake 或 runtime 操作。

## Decision Brief

推荐决策：批准 CR060 最小真实执行切片，执行 `CR060-ACT-001` 和 `CR060-ACT-002`。备选方案 A：只保留门禁，不执行文本改写；备选方案 B：扩大到包名/import/Git remote/NAS/lake 一次性迁移；备选方案 C：先执行 GitHub 初始化再改文档。推荐方案优点是风险最小且能开始真实迁移；备选 A 无法推进最终迁移；备选 B 风险和回滚复杂度过高；备选 C 会把发布身份和 repo-local 改写混合。

回退 / 切换条件：如果执行前 HEAD 变化，先刷新 manifest 和 rollback evidence；如果用户希望先初始化远端，切换到 CR062；如果用户希望包名/import 同步改，切换到 CR061；如果用户希望 NAS 数据先迁移，切换到 CR063。

Decision Collection Coverage：已扫描来源包括 `process/changes/CR-060-REPO-LOCAL-MECHANICAL-MIGRATION-EXECUTION-GATE-2026-06-15.md`、`docs/release/MIGRATION-EXECUTION-MANIFEST-CR060.yaml`、`docs/release/REPO-LOCAL-MOVE-PLAN-CR060.md`、`docs/release/PRESERVE-AUDIT-DECISIONS-CR060.md`、`docs/release/ROLLBACK-REF-CR060.md`、CR058 candidate / preserve-audit / rollback 输入和 CR059 manifest seed。候选问题数 6，纳入待决策数 6，N/A 0。

待人工决策清单：

| 决策 ID | 决策类型 | 问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP5-CR060-01 | runtime_authorization | 是否授权执行 CR060-ACT-001 / ACT-002？ | 授权执行 README 和 USER-MANUAL 文档身份改写。 | 只保留门禁不执行。 | 推荐方案开始真实迁移；备选继续停留在计划。 | 只影响两个用户文档。 | 发现未纳入改动时暂停。 |
| DQ-CP5-CR060-02 | scope | 执行范围是否禁止扩展到其他文件？ | 禁止扩展；只改 manifest 两项。 | 允许临场追加文件。 | 推荐方案可审计；备选易越界。 | 未覆盖文件留给后续 CR。 | 需要新增文件时重开 CP5。 |
| DQ-CP5-CR060-03 | risk_acceptance | 是否接受复用 CR059 rollback refs？ | 接受 docs-only 复用；HEAD 变化则刷新。 | 执行前创建新 bundle。 | 推荐方案足够且快；备选更强但增加操作。 | 若 HEAD 变化必须阻断。 | 用户要求新 bundle 或 HEAD 变化时刷新。 |
| DQ-CP5-CR060-04 | security | 是否确认不读取凭据和 `.env`？ | 确认不读取。 | 读取配置辅助判断。 | 推荐方案符合安全边界；备选不必要且高风险。 | 无法验证真实运行配置。 | 凭据处理需独立 CR。 |
| DQ-CP5-CR060-05 | implementation | 是否保留 `MARKET_DATA_LAKE_ROOT` 和 package/import 不变？ | 保留不变。 | 同步替换为 `QUANT_LAB_*` 或改包名。 | 推荐方案避免破坏运行；备选需要 CR061/CR064。 | 部分旧标识会保留。 | 后续 CR061/CR064 处理。 |
| DQ-CP5-CR060-06 | runtime_authorization | 是否确认本次不执行 Git remote / push / NAS / runtime？ | 确认不执行。 | 合并执行远端、NAS 或 runtime。 | 推荐方案边界清晰；备选风险高且需更多前置。 | 远端和数据迁移延后。 | 后续 CR062/CR063/CR065。 |

不授权项：

| Item ID | 不授权操作 |
|---|---|
| NA-CP5-CR060-001 | manifest 两项之外的任何 text rewrite |
| NA-CP5-CR060-002 | file move / rename / delete |
| NA-CP5-CR060-003 | 历史 `process/**`、checks、checkpoints、handoffs、Story、LLD、DEV-LOG 或历史 CR 改写 |
| NA-CP5-CR060-004 | `pyproject.toml`、import path、package name、dependency lock 改动 |
| NA-CP5-CR060-005 | `MARKET_DATA_LAKE_ROOT` replacement、lake write、catalog publish、provider fetch |
| NA-CP5-CR060-006 | NAS 数据 copy / move / delete / archive promote |
| NA-CP5-CR060-007 | Git remote add / push / tag / branch rename / history rewrite |
| NA-CP5-CR060-008 | `.env`、token、password、cookie、session、private key、账户或交易事实读取 |
| NA-CP5-CR060-009 | QMT / MiniQMT runtime、账户查询、下单、撤单、simulation/live |
| NA-CP5-CR060-010 | 恢复或推进 CR046 CP7 |

auto_final_authorization: false

## Entry Criteria

| 条件 | 结果 |
|---|---|
| CR060 正式 CR 已创建 | PASS |
| execution manifest 已冻结 | PASS |
| rollback refs 存在 | PASS_WITH_RISK |
| preserve-audit 决策已写明 | PASS |

## Checklist

| 检查项 | 结果 |
|---|---|
| 执行范围最小且明确 | PASS |
| 决策项完整 | PASS |
| 不授权项完整 | PASS |
| 回滚策略存在 | PASS_WITH_RISK |
| 用户授权 | PENDING |

## Exit Criteria

| 条件 | 结果 |
|---|---|
| 用户回复 `approve` 后可执行 CR060-ACT-001 / ACT-002 | PENDING |
| 用户回复 `修改: <具体修改点>` 后返工 | PENDING |
| 用户回复 `reject` 后停止 CR060 | PENDING |

## Deliverables

- `docs/release/MIGRATION-EXECUTION-MANIFEST-CR060.yaml`
- `docs/release/REPO-LOCAL-MOVE-PLAN-CR060.md`
- `docs/release/PRESERVE-AUDIT-DECISIONS-CR060.md`
- `docs/release/ROLLBACK-REF-CR060.md`
- `process/checks/CP5-CR060-EXECUTION-READINESS.md`
- `process/checks/CP5-CR060-HUMAN-GATE-LAUNCH-MESSAGE.md`

## 人工审查结果

- 结论：approved
- 审查人：user
- 审查时间：2026-06-15T06:43:41+08:00
- 修改意见：用户回复“同意”，按 `approve` 解析，接受 DQ-CP5-CR060-01..06 推荐方案。
- 风险接受项：接受 docs-only 改写复用 CR059 rollback refs；HEAD 变化时必须刷新。
