# CR046 Migration Notes

> 结论：本次无运行时迁移、无数据迁移、无环境迁移。

## 迁移范围

| 对象 | 是否迁移 | 说明 |
|---|---|---|
| Python 依赖 / 虚拟环境 | 否 | 不安装依赖，不同步环境。 |
| QMT / MiniQMT / XtQuant | 否 | 不启动、不安装、不连接。 |
| NAS / 策略包传输通道 | 否 | 不访问 NAS，不传输 package。 |
| 数据湖 / catalog | 否 | 不 fetch、不 write、不 publish。 |
| 账户 / 交易数据 | 否 | 不读取账号、账户、资金、持仓、委托、成交或日志原文。 |
| 文档与流程台账 | 是 | 新增 CR046 release context、release docs、CP8 check 和 checkpoint。 |

## 兼容性说明

CR046 交付的是后续策略交付与验证的框架合同。它不会改变现有研究代码、回测流程、交易接口、运行目录或权限配置。后续 CR047 / CR048 / CR049 / CR091 若要实施 runner、真实连接或策略包传输，必须单独设计迁移与回滚方案。

## 用户动作

当前 CP8 不要求用户在机器上执行任何安装、导入、复制或运行动作。用户只需要确认是否接受 `READY_WITH_RISK` 关闭当前 framework-first 交付。
