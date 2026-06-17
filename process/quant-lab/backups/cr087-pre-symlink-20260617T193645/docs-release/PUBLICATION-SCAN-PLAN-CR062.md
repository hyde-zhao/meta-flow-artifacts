# CR062 Publication Scan Plan

## 扫描原则

所有扫描均为 post-approval preflight。CP2 / CP3 / CP5 批准前，不执行 Git 写动作，也不读取 `.env*`、NAS、data lake、runtime 或账户材料。

扫描采用 fail-closed 策略：发现 BLOCKING / HIGH 风险时停止 publication，回到人工决策。

## 扫描项

| 扫描项 | 目的 | 范围 | 阻断条件 |
|---|---|---|---|
| Git status manifest scan | 冻结待提交路径 | post-approval `git status --short` 输出 | 出现未分类路径或 forbidden path。 |
| Secret pattern scan | 防止凭据入库 | approved candidate paths，不读取 `.env*` | token/key/password/session/private key 模式命中且未解释。 |
| Data body/path boundary scan | 防止数据和运行产物入库 | approved candidate paths | `data/**`、`reports/**`、`runs/**` 或疑似市场数据正文进入 manifest。 |
| Large file scan | 防止大文件和二进制误提交 | approved candidate paths | 单文件超过人工批准阈值或二进制不可解释。 |
| Process evidence scope scan | 防止批量历史过程文件误纳入 | CR058-CR062 scoped evidence | 历史 process 全量批量纳入或含敏感运行材料。 |

## 禁止事项

- 不读取 `.env` / `.env*` 正文。
- 不访问 NAS mount 或 `/mnt/**`。
- 不执行 provider fetch、lake write、catalog publish。
- 不启动 QMT / MiniQMT 或读取账户、资金、持仓、委托、成交事实。
- 不执行 tag、branch rename、history rewrite 或 force push。

## 扫描输出

post-approval 执行时必须生成 CR062 scoped 检查证据，至少记录：

- 扫描命令或等价检查方式
- 扫描范围
- 排除路径
- finding 数量和严重度
- 是否 PASS / BLOCKED
- 冻结的 commit manifest
