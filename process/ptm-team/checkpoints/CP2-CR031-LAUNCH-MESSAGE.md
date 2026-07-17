请审查人工门禁 `CP2-CR031-PRODUCT-BASELINE`。

checklist 路径: `process/checkpoints/CP2-CR031-PRODUCT-BASELINE.md`
自动预检结论: 已生成 Decision Brief；发起前请以 `meta-flow check human-gate --checkpoint process/checkpoints/CP2-CR031-PRODUCT-BASELINE.md` 的结果为准。

审批者摘要:
- 本次确认服务的整体目标: 请见 checkpoint `### 审批者摘要`；本消息只承载发起确认所需摘要。
- 推荐动作: 默认推荐 approve，除非你要求调整范围、风险、授权边界或推荐方案。
- approve 后会发生什么: 接受本轮 Decision Brief 中列出的推荐方案，并允许进入 checkpoint 声明的下一阶段。
- approve 不授权什么: 不授权真实运行、凭据读取、安全边界变更、外部接口调用、数据写入、publish、live / 交易类操作。
- 不确认会阻塞什么: 当前 checkpoint 后续推进会保持等待，直到你回复 approve / 修改 / reject。

Context Capsule: 请见 checkpoint 的 `### Context Capsule Summary`，其中包含 capsule、read_profile、默认读取策略和全文档读取边界。
决策收集覆盖: 请见 checkpoint 的 `### Decision Collection Coverage`，本消息只承载发起确认所需摘要。
决策分层:
- 必须用户决策: 3
- 高风险策略确认: 以 checkpoint `### 决策分层` 为准。
- agent 默认处理: 低风险、可逆、局部实现细节默认不进入用户主确认表，完整清单以 checkpoint 为准。
- 仅审计记录: 已按项目规则处理且无需用户拍板的事项只保留审计摘要。
本轮待人工决策项: 3
blocking / high-risk 决策摘要: CP2-DQ-CR031-01, CP2-DQ-CR031-02, CP2-DQ-CR031-03

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| CP2-DQ-CR031-01 | scope | 是否以用户已确认的项目级安装与运行根语义作为 CR-031 基线？ | 默认安装根为执行 `ptm-team` 安装命令的项目目录；显式目标覆盖；`--runtime-root` 默认安装根；数据只写 `<runtime-root>/data/`。 | 全局用户目录默认；或任意运行时 CWD 默认。 | 推荐路径可预测、项目隔离明确；全局/CWD 方案会产生泄漏和路径漂移。 | 改变安装器、Skill 路径解析与数据权限。 |
| CP2-DQ-CR031-02 | security | 无效根或 support 缺失时是否拒绝所有回退？ | 是：阻断、清理未提交数据、说明修复方式；不访问 `ptm-team/data`、全局目录或任意 CWD。 | 自动回退到源码或全局目录。 | 推荐能避免跨项目数据泄漏；回退看似方便但会掩盖安装故障并破坏隔离。 | 影响首次运行失败体验和受限存储可靠性。 |
| CP2-DQ-CR031-03 | implementation | 源码根旧 `data/` support 副本何时清理？ | 完成新安装、路径隔离、权限、卸载/回滚验证后清理；不迁移真实运行数据到源码仓库。 | 立即删除；或永久保留为运行时依赖。 | 推荐保留可回退能力且最终清除错误边界；立即删除有中断风险，永久依赖则无法独立安装。 | 影响 `data/schema.sql`、`data/dao.py` 和安装包设计。 |

如果你回复 approve，表示你接受以上 3 项推荐方案，不表示授权以下不授权项。
不授权项: 不授权真实运行、凭据读取、安全边界变更、外部接口调用、数据写入、publish、live / 交易类操作；若 checkpoint 中列有额外不授权项，以 checkpoint 为准。

请只回复以下三个 exact 选项之一：
- approve
- 修改: <具体修改点>
- reject
