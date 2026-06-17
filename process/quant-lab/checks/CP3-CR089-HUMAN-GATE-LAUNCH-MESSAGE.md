请审查：process/checkpoints/CP3-CR089-QMT-INTERFACE-VALIDATION-HLD-REVIEW.md
自动预检结论：PASS
Context Capsule 摘要：process/context/CP3-CR089-QMT-INTERFACE-VALIDATION-CONTEXT.yaml（read_profile=compact，status=ready）
上下文胶囊：process/context/CP3-CR089-QMT-INTERFACE-VALIDATION-CONTEXT.yaml（read_profile=compact）
本轮待人工决策项：6
决策收集覆盖：已扫描 STATE、CP2 checkpoint、策略包交付规划、CP3 自动预检；候选问题 23 项，纳入待决策 6 项；discussion log N/A。
如果你回复 approve，表示你接受以下 6 项推荐方案，不表示授权以下不授权项。
待人工决策清单：
| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP3-CR089-01 | architecture | NAS 是否作为默认执行根？ | 不作为执行根，仅作为 package exchange。 | NAS 原地执行；U 盘；Git release。 | 推荐减少网络和半写入风险。 | 需要本地缓存。 |
| DQ-CP3-CR089-02 | architecture | 策略包结构如何确定？ | zip + sha256 + manifest.yaml + docs + validation。 | 单文件脚本；wheel；Git tag。 | 推荐可审计。 | 需要 manifest 校验。 |
| DQ-CP3-CR089-03 | architecture | 交易主机如何落地 package？ | 校验后复制到本地不可变缓存，active symlink 指向当前版本。 | NAS 直接运行；覆盖同名目录。 | 推荐可回滚。 | 需要本地空间。 |
| DQ-CP3-CR089-04 | implementation | index / approval / quarantine 如何设计？ | package-index、approvals、quarantine/rejection。 | 文件名约定；口头审批。 | 推荐可追溯。 | 增加审批文件。 |
| DQ-CP3-CR089-05 | security | qmt_interface_smoke 能否含交易写入能力？ | 不允许，只含授权后的 query_positions。 | submit/cancel；simulation。 | 推荐权限最小。 | 不验证写接口。 |
| DQ-CP3-CR089-06 | risk_acceptance | 如何处理 endpoint matrix 语义冲突？ | 作为 CP5/实现前风险，不在本门禁改代码。 | 立即改；忽略。 | 推荐先决策再改。 | 后续实现需重审。 |
不授权项：
- 不授权访问、列目录、读取、写入、复制或删除 NAS 内容。
- 不授权读取 .env、token、账号、密码、HMAC secret 或 QMT 凭据。
- 不授权启动 QMT / MiniQMT / XtQuant / gateway。
- 不授权账户原文查询或下单 / 撤单 / simulation / live。
- 不授权修改 query_positions 语义冲突；实现前需另行确认。
approve
修改: <具体修改点>
reject
