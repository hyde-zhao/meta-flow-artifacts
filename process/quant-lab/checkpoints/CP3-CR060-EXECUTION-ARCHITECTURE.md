# CP3-CR060 Execution Architecture 人工检查点

## 自动预检摘要

自动预检结论：PASS_WITH_RISK。架构采用分层迁移：CR060 做 repo-local 文档身份收敛，CR061 做 package/import，CR062 做 Git remote，CR063 做 NAS archive，CR064 做 data lake root/catalog，CR065 或恢复 CR046 做 trading runtime。

## Decision Brief

推荐决策：接受分层执行架构，并将 CR060 的执行面限制为 `README.md` 和 `docs/USER-MANUAL.md`。备选方案 A 是把 CR061-CR064 合并到 CR060；备选方案 B 是只继续补设计不执行。推荐方案优点是可在一个小切片中真实推进并验证保护规则；备选 A 会扩大回滚面；备选 B 无法达成用户要求的逐步真实迁移。

回退 / 切换条件：如果 CP5 前发现 `README.md` 或 `docs/USER-MANUAL.md` 存在未纳入用户改动，暂停执行并生成 include/exclude manifest；如果用户决定先做 GitHub 初始化，转 CR062。

Decision Collection Coverage：已扫描 CR060 manifest、repo-local move plan、preserve-audit decisions、rollback ref、CR058/CR059 输入。候选问题数 6，纳入待决策数 6，N/A 0。

待人工决策清单：

| 决策 ID | 决策类型 | 问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP3-CR060-01 | architecture | 迁移是否按 CR060-CR065 分层？ | 分层推进。 | 合并大 CR。 | 分层便于回滚；大 CR 上下文和失败面大。 | 总迁移周期较长。 | 用户要求一次性迁移时重新规划。 |
| DQ-CP3-CR060-02 | architecture | CR060 是否不做目录移动？ | 不做 `mv`，只做文档身份收敛。 | 做根目录或目录树移动。 | 推荐方案适合当前工作区；目录移动需更多门禁。 | 真实目录重排延后到 CR061。 | 需要目录重排时转 CR061。 |
| DQ-CP3-CR060-03 | security | 是否继续不读取 `.env`？ | 不读取。 | 读取以确认路径。 | 推荐方案符合凭据边界；备选风险高。 | 真实 runtime 配置不在本 CR 验证。 | 独立凭据方案获批后再处理。 |
| DQ-CP3-CR060-04 | implementation | 是否保留 legacy alias 段？ | 保留并明确 `local_backtest` 是 legacy alias。 | 全删旧名。 | 推荐方案兼容审计和旧命令；全删会误导恢复。 | 用户文档会同时出现新旧名。 | 稳定后可另 CR 缩减 legacy 段。 |
| DQ-CP3-CR060-05 | risk_acceptance | 是否复用 CR059 bundle/backup？ | 对 docs-only 改写复用；HEAD 变化则刷新。 | 立即创建新 bundle。 | 推荐方案够用且省时；备选更强但增加操作。 | 如果 HEAD 变化必须刷新。 | HEAD 变更或用户要求更强回滚时刷新。 |
| DQ-CP3-CR060-06 | runtime_authorization | 是否确认 GitHub 初始化不在 CR060？ | 不在本 CR，进入 CR062。 | 本 CR 内 remote add/push。 | 推荐方案保持发布门禁独立；备选混合风险。 | 远端仍为空。 | 用户启动 CR062 后执行。 |

不授权项：CP3 通过不授权任何真实 text rewrite、file move、NAS copy、lake write、Git remote/push、凭据读取、runtime 或历史审计 rewrite。

auto_final_authorization: false

## Entry Criteria

| 条件 | 结果 |
|---|---|
| CR060 baseline 和 manifest 存在 | PASS |

## Checklist

| 检查项 | 结果 |
|---|---|
| 分层架构清晰 | PASS |
| 回滚和保护边界清晰 | PASS_WITH_RISK |
| 后续 CR 边界清晰 | PASS |

## Exit Criteria

| 条件 | 结果 |
|---|---|
| 用户确认后可进入 CP5 execution readiness | PENDING |

## Deliverables

- `process/checks/CP3-CR060-EXECUTION-ARCHITECTURE.md`
- `process/checkpoints/CP3-CR060-EXECUTION-ARCHITECTURE.md`

## 人工审查结果

待用户审查。
