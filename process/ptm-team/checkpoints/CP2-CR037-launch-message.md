请审查人工门禁 `CP2-CR037-DECISION-BRIEF`。

checklist 路径: `process/checkpoints/CP2-CR037.md`
自动预检结论: 已生成 Decision Brief（3 项待决策）；以 `meta-flow check human-gate --checkpoint process/checkpoints/CP2-CR037.md` 的结果为准。

审批者摘要:
- 本次确认服务的整体目标: 锁定 CR-037（ptm-te 集成 topo 管理）需求/场景/范围基线（三层文件结构 + topo-limit 约束语义 + 映射流程 + 集成方式），批准后进入 HLD（CP3）。
- 推荐动作: 默认推荐 approve，除非你要求调整范围、风险、授权边界或推荐方案。
- approve 后会发生什么: 回填人工结果，进入 solution-design（meta-se 产出 BLUEPRINT/HLD + Story 拆解）；CP3 前不再追加需求变更。
- approve 不授权什么: 不授权真实设备配置写入（--execute 需 DQ-037-04 独立授权）、凭据读取、外部接口调用、生产数据写入。
- 不确认会阻塞什么: HLD 设计、Story 拆解、实现推进保持等待。

Context Capsule: `process/context/CP2-CR037-CONTEXT.yaml`（read_profile=compact；默认读取 CR-037 + checkpoint + summary）。
决策收集覆盖: 已扫描 CR-037 / 评审核实 / 用户澄清 / manaul 调研 / 仓库证据；候选 5 项，纳入待决策 3 项；DQ-037-03 与三层结构已定稿转审计。
决策分层:
- 必须用户决策: 2（DQ-037-01 接线假设、DQ-037-02 平台别名识别范围 = 消费 manaul device-reference.md 完整映射）
- 高风险策略确认: 1（DQ-037-04 真机下发 runtime_authorization）
- agent 默认处理: DQ-037-03 链路约束语义（resolved-by-user，不占本轮）
- 仅审计记录: 三层结构（已认可）、评审核实、方案 A
本轮待人工决策项: 3

blocking / high-risk 决策摘要:
- DQ-037-04（真机下发授权）: 独立 runtime_authorization，不隐含于本 CP 批准。

本轮待人工决策项: 3。表格见 checkpoint `### 待人工决策清单`。

如果你回复 approve，表示你接受以上 3 项推荐方案，不表示授权以下不授权项。
不授权项: 真实设备配置写入（--execute）、凭据读取、外部接口调用、生产数据写入；以 checkpoint 为准。

请只回复以下三个 exact 选项之一:
- approve
- 修改: <具体修改点>
- reject
