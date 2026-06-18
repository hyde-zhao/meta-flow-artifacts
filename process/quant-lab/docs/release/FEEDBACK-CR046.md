# CR046 Feedback

> 反馈入口用于 CR046 关闭后的风险回流和后续 CR 分流。

## 反馈分类

| 类型 | 触发条件 | 建议路由 |
|---|---|---|
| 文档不清晰 | QMT / MiniQMT 交付框架、验证框架或安装设计存在歧义 | CR046 文档修订或后续 doc governance |
| 具体策略准备就绪 | 已有研究策略需要交付到 QMT / MiniQMT | CR047-candidate |
| QMT terminal shadow 准备就绪 | 用户准备在 QMT 终端做 shadow / 最小模拟验证 | CR048-candidate + runtime authorization |
| MiniQMT 权限准备就绪 | 用户开通 MiniQMT / XtQuant 并准备只读连接或 install dry-run | CR049-candidate |
| runner 未开发导致无法验证策略 | 需要研究、设计并实现 broker/runner 运行能力 | CR091 保持 blocked，用户明确启动后进入后续研究 / 设计 / 实现 |
| 账本或 canonical docs 债务 | CR-INDEX YAML parse、docs/product canonical 缺口等 | 独立 ledger / doc hygiene CR |

## 反馈记录模板

| 时间 | 来源 | 问题 | 影响 | 建议路由 | 状态 |
|---|---|---|---|---|---|
|  |  |  |  |  | open |

## 当前已知反馈

| 时间 | 来源 | 问题 | 影响 | 建议路由 | 状态 |
|---|---|---|---|---|---|
| 2026-06-18T07:41:38+08:00 | CP7 / CP8 | runner 未开发时不能验证真实策略运行 | 不阻断 CR046 framework-first 关闭；阻断 runtime verified 声明 | CR091 或后续 runner 实现 CR | tracked |
| 2026-06-18T07:41:38+08:00 | CP7 / CP8 | docs/product canonical scenarios/test matrix 缺失 | 不阻断 CR046；影响后续 canonical doc 追溯 | doc governance CR | tracked |
| 2026-06-18T07:41:38+08:00 | CP7 / CP8 | CR-INDEX 全量 YAML parse 历史债务 | 不阻断 CR046 局部一致性；影响账本维护体验 | ledger hygiene CR | tracked |

## 不授权提醒

反馈记录不授权任何外部动作。涉及 NAS、QMT/MiniQMT/XtQuant/gateway、凭据、账户、submit/cancel、simulation/live 或 provider/lake/catalog 的反馈，必须先转成独立 CR 并通过对应人工门禁。
