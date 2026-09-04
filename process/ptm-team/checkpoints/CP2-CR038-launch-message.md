请审查人工门禁 `CP2-CR038`。

checklist 路径: `process/checkpoints/CP2-CR038.md`
自动预检结论: 已生成 Decision Brief；发起前请以 `meta-flow check human-gate --checkpoint process/checkpoints/CP2-CR038.md` 的结果为准。

审批者摘要:
- 本次确认服务的整体目标: 请见 checkpoint `### 审批者摘要`；本消息只承载发起确认所需摘要。
- 推荐动作: 默认推荐 approve，除非你要求调整范围、风险、授权边界或推荐方案。
- approve 后会发生什么: 接受本轮 Decision Brief 中列出的推荐方案，并允许进入 checkpoint 声明的下一阶段。
- approve 不授权什么: 不授权真实运行、凭据读取、安全边界变更、外部接口调用、数据写入、publish、live / 交易类操作。
- 不确认会阻塞什么: 当前 checkpoint 后续推进会保持等待，直到你回复 approve / 修改 / reject。

Context Capsule: 请见 checkpoint 的 `### Context Capsule Summary`，其中包含 capsule、read_profile、默认读取策略和全文档读取边界。
决策收集覆盖: 请见 checkpoint 的 `### Decision Collection Coverage`，本消息只承载发起确认所需摘要。
决策分层:
- 必须用户决策: 7
- 高风险策略确认: 以 checkpoint `### 决策分层` 为准。
- agent 默认处理: 低风险、可逆、局部实现细节默认不进入用户主确认表，完整清单以 checkpoint 为准。
- 仅审计记录: 已按项目规则处理且无需用户拍板的事项只保留审计摘要。
本轮待人工决策项: 7
blocking / high-risk 决策摘要: DQ-038-01, DQ-038-02, DQ-038-03, DQ-038-05, SGQ-038-01, SGQ-038-02, SGQ-038-03

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-038-01 | scope | PPPoE Server 如何建模 | 逻辑 `pppoe` 节点改真实 `SW` 节点（`node_type=SW, role=pppoe-server`） | 保留 `Mock` + 映射期替换为真实 SW | 推荐：复用 SW 匹配/role 字段、最简、无特殊路径；备选：需新增 Mock→SW 替换逻辑，复杂度高 | 影响 topo_mapper 匹配与建模文档；不改则 Mock 无法下发配置 |
| DQ-038-02 | architecture | SW 在 env-file 契约中的表达 | SW 进 `nodes`（`nodes.swN`），**不进** `port_mapping`，**不新增** `${ENV.sw.*} | (B) SW 全量进 env-file；(C) SW 独立产物 | 推荐：复用 CR-037 部署链路、case-execution 零适配；B 破坏 9 类占位符契约；C 多一个产物面 | 影响 exporter/case_runner/ADR-09；B 需改 case-execution 占位符 |
| DQ-038-03 | scope | PPPoE Client 配置归属 + ptm-atomic op 缺失处理路径 | **ptm-atomic 原子操作**（执行层）；op 缺失三选一：本仓库 op_mapper 映射到可用 op / 提外部依赖 / 降级手工步骤 | factor-library 因子扩展（设计层） | 推荐：拨号是运行时配置动作，非测试设计变量；op 缺失首选 op_mapper 映射（最简） | 影响 op_mapper/因子库边界；op 缺失不闭环则 SCN-038-POS-005 无法落地 |
| DQ-038-05 | architecture | PPPoE 地址池网段 + local-user 用户名/密文的数据源契约 | physical_pool 新字段（与设备台账同源，与 NGFW 拨号 163.0.0.x 同源一致） | devices.yaml / 独立配置文件 | 推荐：physical_pool 已是设备台账真相源，ip pool 段与设备强绑定；devices.yaml 与 SW 设备分离，独立配置引入第三产物面 | 影响 R-F-019 数据读取路径；不同源会破坏「同源一致」约束 |
| SGQ-038-01 | scope | PPPoE 地址池 vs 静态 IPAM + SW IP/VLAN 归属 | 方案 A：SW 配置与地址池解耦，动态地址不进静态 IPAM | (B) SW 全量进 env-file；(C) SW 独立产物 | 推荐：最简、复用 CR-037 链路、PPPoE 动态地址语义天然属配置参数；B 破坏静态 IP 规划语义；C 多产物面 | 影响 IPAM 集成（R-C-006）；错误会破坏 env-file 静态 IP 规划语义 |
| SGQ-038-02 | implementation | 环回 links 方向性 | links 无向，发流方向由用例 txport/rxport 指定 | links 有向 | 推荐：与 CR-037 links 无向端点对契约一致，方向本就用例层关注点；加 direction 字段属过度设计 | 影响 exporter links 契约 |
| SGQ-038-03 | architecture | PPPoE Server 是否强制 VRF | 首版全局路由域，VRF 可选（复用 ip binding vpn-instance） | 首版强制 VRF | 推荐：参考场景未涉及 VRF，最小实现；备选：无谓复杂度 | 影响 PPPoE 命令族；强制 VRF 需额外 vpn-instance 配置 |

如果你回复 approve，表示你接受以上 7 项推荐方案，不表示授权以下不授权项。
不授权项: 不授权真实运行、凭据读取、安全边界变更、外部接口调用、数据写入、publish、live / 交易类操作；若 checkpoint 中列有额外不授权项，以 checkpoint 为准。

请只回复以下三个 exact 选项之一：
- approve
- 修改: <具体修改点>
- reject
