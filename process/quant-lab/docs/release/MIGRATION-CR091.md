# CR091 Migration

> release_decision: `READY_WITH_RISK`
> profile: `compact`
> generated_at: `2026-06-18T15:24:12+08:00`

## 迁移结论

CR091 不包含生产数据迁移、账户数据迁移、NAS 迁移、配置迁移或运行时安装迁移。

| 维度 | 结论 | 说明 |
|---|---|---|
| 状态 schema | N/A | 未修改 Meta Flow schema，仅追加 CR091 过程状态。 |
| 模板字段 | N/A | 未修改共享模板。 |
| 配置 | N/A | 未读取或新增 `.env` / credentials / account config。 |
| 安装路径 | N/A | 未交付安装脚本或平台安装目标。 |
| Agent / Skill frontmatter | N/A | 未修改 Agent / Skill 产物。 |
| 命令参数 | N/A | 只新增 CR091 offline checker 命令，不影响现有 CLI。 |
| 数据结构 | PASS_WITH_RISK | 新增本地 strategy runner dataclass / dict 合同；由 pytest 和 checker 覆盖。 |
| 外部接口 | N/A | 仅 fake readonly transport；未接真实 gateway / QMT。 |

## 兼容性判断

| 对象 | 判断 |
|---|---|
| 现有研究 / 回测流程 | 不应受影响；CR091 新增独立包和测试。 |
| QMT / MiniQMT 环境 | 未触碰；不存在运行时兼容性结论。 |
| NAS package exchange | 未触碰；未来需独立 gate。 |
| 历史 CR020 gateway 路线 | 不恢复、不迁移、不作为当前入口。 |
| CR089 | 保持 blocked-readiness-approved，不自动启动。 |

## 后续迁移触发条件

若未来要把离线 runner 接入真实交易主机，需要新建独立 gate，并至少补充：

- runtime authorization。
- 交易主机本地安装 / 路径设计。
- 配置和凭据读取边界。
- 脱敏 evidence 输出目录。
- NAS package exchange 或人工包传递规则。
- 回滚和禁用 runner 的操作步骤。
