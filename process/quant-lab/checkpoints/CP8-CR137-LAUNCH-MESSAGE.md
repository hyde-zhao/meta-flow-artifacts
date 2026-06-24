请审查：process/checkpoints/CP8-CR137-DELIVERY-READINESS.md
自动预检结论：PASS
Context Capsule / 上下文胶囊：process/release/RELEASE-CONTEXT-CR137.yaml（read_profile=compact）
本轮待人工决策项：1

待人工决策清单：
| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| CP8-CR137-DQ-01 | risk_acceptance | 是否接受 CR137 以 READY_WITH_RISK 关闭：功能验证已通过，但 cr-tracking 仍有一个非阻塞 warning。 | 接受 READY_WITH_RISK 并关闭 CR137；warning 作为 FU-CR137-001 候选。 | 先修复 warning 后重提；或 reject。 | 推荐方案最快关闭已验证功能；备选 A 可实现零 warning 但扩大当前 CR；reject 最保守但延迟交付。 | 风险低；命令退出 OK 且明确列出 active formal CRs: CR-137。 |

不授权项：
- 读取 .env、credentials、account/session/raw logs
- 访问、列取、读取、写入、发布、pull 或删除 NAS
- 启动/连接 QMT/MiniQMT/XtQuant/gateway runtime
- provider fetch、lake write、catalog publish/current pointer
- submit/cancel/buy/sell/simulation/live
- git add/commit/push/remote write

用户已在对话中回复 approve，表示接受上表推荐方案；不表示授权任何不授权项，尤其不表示授权 runtime、NAS、QMT、credentials、provider/lake/catalog、trading 或 Git remote write。
