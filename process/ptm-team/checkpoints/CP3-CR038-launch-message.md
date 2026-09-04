请审查人工门禁 `CP3-CR038`。

checklist 路径: `process/checkpoints/CP3-CR038.md`
自动预检结论: 已生成 Decision Brief；发起前请以 `meta-flow check human-gate --checkpoint process/checkpoints/CP3-CR038.md` 的结果为准。

审批者摘要:
- 本次确认服务的整体目标: 请见 checkpoint `### 审批者摘要`；本消息只承载发起确认所需摘要。
- 推荐动作: 默认推荐 approve，除非你要求调整范围、风险、授权边界或推荐方案。
- approve 后会发生什么: 接受本轮 Decision Brief 中列出的推荐方案，并允许进入 checkpoint 声明的下一阶段。
- approve 不授权什么: 不授权真实运行、凭据读取、安全边界变更、外部接口调用、数据写入、publish、live / 交易类操作。
- 不确认会阻塞什么: 当前 checkpoint 后续推进会保持等待，直到你回复 approve / 修改 / reject。

Context Capsule: 请见 checkpoint 的 `### Context Capsule Summary`，其中包含 capsule、read_profile、默认读取策略和全文档读取边界。
决策收集覆盖: 请见 checkpoint 的 `### Decision Collection Coverage`，本消息只承载发起确认所需摘要。
决策分层:
- 必须用户决策: 1
- 高风险策略确认: 以 checkpoint `### 决策分层` 为准。
- agent 默认处理: 低风险、可逆、局部实现细节默认不进入用户主确认表，完整清单以 checkpoint 为准。
- 仅审计记录: 已按项目规则处理且无需用户拍板的事项只保留审计摘要。
本轮待人工决策项: 1
blocking / high-risk 决策摘要: CP3-CR038-ARCH-01

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| CP3-CR038-ARCH-01 | architecture | HLD 是否采用方案 A（扩展 topo 引擎）并接受 5 条 ADR 落地映射 | 方案 A + ADR-CR038-01~05（显式 SW reserved 互斥 / 多实例 TRex 归并 / PPPoE 数据源 pppoe_server 块 / 环回放行 1TG+1DUT+N SW / interface_kind 透传） | 方案 B 独立 PPPoE 规划器 | 推荐 A：复用 CR-037 映射/台账/产物/安装链路，最简增量；B：与 topo 引擎解耦但引入重复形态与额外共享运行时 | 影响全部 13 Story 的文件所有权与 LLD；方案错误导致返工 |

如果你回复 approve，表示你接受以上 1 项推荐方案，不表示授权以下不授权项。
不授权项: 不授权真实运行、凭据读取、安全边界变更、外部接口调用、数据写入、publish、live / 交易类操作；若 checkpoint 中列有额外不授权项，以 checkpoint 为准。

请只回复以下三个 exact 选项之一：
- approve
- 修改: <具体修改点>
- reject
