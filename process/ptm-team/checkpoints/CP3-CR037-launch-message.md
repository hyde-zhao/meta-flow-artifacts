请审查人工门禁 `CP3-CR037-HLD-DESIGN`。

checklist 路径: `process/checkpoints/CP3-CR037.md`
自动预检结论: 已生成 Decision Brief（4 项待决策）；以 `meta-flow check human-gate --checkpoint process/checkpoints/CP3-CR037.md` 的结果为准。

审批者摘要:
- 本次确认服务的整体目标: 确认 CR-037 HLD 架构（三层结构 + topo-planning 6 模块 + per-link 增强 + 产物导出 + 安装集成）与 3 个开放架构灰区决策 + F-CR037 waived 判定，批准后进入 Story 执行准备。
- 推荐动作: 默认推荐 approve，除非你要求调整架构、范围、风险、授权边界或推荐方案。
- approve 后会发生什么: 回填人工结果，进入 CP4 自动预检 + meta-dev 产出 Story 设计证据（full-lld/technical-note）。
- approve 不授权什么: 不授权真实设备配置写入（--execute 需 DQ-037-04 独立授权）、凭据读取、外部接口调用、生产数据写入。
- 不确认会阻塞什么: Story 设计证据写作、Story 执行推进保持等待。

Context Capsule: `process/context/CP3-CR037-CONTEXT.yaml`（read_profile=compact；默认读取 HLD + checkpoint + CR-037 + 讨论日志）。
决策收集覆盖: 已扫描 HLD / CP3-HLD-DISCUSSION-LOG（AG-1~7）/ CP2 决策 / 用户确认；候选 4 项，纳入待决策 4 项。
决策分层:
- 必须用户决策: 2（DQ-037-05 归并策略 A、DQ-037-06 agent 转换边界）
- 高风险策略确认: 2（DQ-037-07 install 生成位置、DQ-037-08 F-CR037 waived）
- agent 默认处理: HLD 内部一致性、Story lld_policy 分配
- 仅审计记录: ADR-CR037-01~07 摘要
本轮待人工决策项: 4

blocking / high-risk 决策摘要:
- DQ-037-05（归并策略）: host 关联 + devices 覆盖，冲突以 devices 为准 + warning。

本轮待人工决策项: 4。表格见 checkpoint `### 待人工决策清单`。

如果你回复 approve，表示你接受以上 4 项推荐方案，不表示授权以下不授权项。
不授权项: 真实设备配置写入（--execute）、凭据读取、外部接口调用、生产数据写入；以 checkpoint 为准。

请只回复以下三个 exact 选项之一:
- approve
- 修改: <具体修改点>
- reject
