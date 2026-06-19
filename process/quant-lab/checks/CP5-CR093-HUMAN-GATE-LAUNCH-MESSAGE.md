请审查：process/checkpoints/CP5-CR093-LEDGER-HYGIENE-READINESS.md

自动预检结论：PASS

Context Capsule：process/context/CP5-CR093-LEDGER-HYGIENE-CONTEXT.yaml（read_profile=compact，完整来源见 checklist）

本轮待人工决策项：5

决策收集覆盖：已扫描 6 个来源，发现候选问题 24 个，纳入待决策 5 个；N/A 来源 0 个，原因见 checklist 的 Decision Collection Coverage。

如果你回复 approve，表示你接受以下 5 项推荐方案，不表示授权以下 8 项禁止操作。

待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP5-CR093-01 | implementation | 是否批准 CR093 单 Story full-lld 作为实现设计证据？ | 批准当前 LLD，允许进入受限实施。 | 修改 LLD 后重提；reject 回 CP3。 | 推荐方案已覆盖文件、接口、测试、回退。 | CP5 通过后可改 checker / tests / CR019 tracking。 |
| DQ-CP5-CR093-02 | implementation | 是否按 TASKS 的串行顺序实施？ | 先写测试，再改 checker，再最小规范化 CR019 tracking。 | 先改 checker；先改旧台账。 | 推荐方案降低回归风险。 | 有利于证明目标缺陷被修复。 |
| DQ-CP5-CR093-03 | security | CP5 approve 是否继续禁止外部动作？ | 继续禁止；只允许读写列明的仓库文本文件。 | 授权 NAS / runtime / 账户证据；授权清理 broader warnings。 | 推荐方案符合 ledger-only；备选扩大风险。 | 无外部副作用。 |
| DQ-CP5-CR093-04 | risk_acceptance | broader warning-only 项是否仍不作为本轮 DoD？ | 接受；DoD 只要求 CR019 / CR025 阻断项收敛。 | 本轮清所有 warnings；全部 suppress。 | 推荐方案聚焦目标。 | CP7 可能仍有 warning。 |
| DQ-CP5-CR093-05 | implementation | CP5 approve 后允许的实施范围是什么？ | 只允许修改 checker、新增 CR093 测试、最小修改 CR019 tracking 和必要 process 证据。 | 仅改 checker；仅改文档；全量重写 checker。 | 推荐方案覆盖根因且可验证。 | 超出文件范围必须回 CP5 或新 CR。 |

不授权项：

- QMT / MiniQMT / XtQuant / gateway / runner runtime 启动、连接、安装或运行。
- NAS 访问、列取、读取、复制、拉取、写入、发布或删除。
- 读取 `.env`、凭据、账号、账户、资金、持仓、委托、成交或日志原文。
- submit / cancel、buy / sell。
- simulation / live。
- provider fetch、lake write、catalog publish。
- 自动启动 CR089、CR091-FU-02、CR091-FU-03 或恢复 CR020 用户删除的 gateway 路线。
- 清理 broader `source=cp8-follow-up` warning-only 项。

请回复以下三者之一：

- `approve`
- `修改: <具体修改点>`
- `reject`
