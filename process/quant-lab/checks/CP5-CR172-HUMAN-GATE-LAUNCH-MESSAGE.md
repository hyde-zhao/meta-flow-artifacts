请审查人工门禁 `CP5-CR172-ALL-STORIES-LLD-BATCH`。

checklist 路径: `process/checkpoints/CP5-CR172-ALL-STORIES-LLD-BATCH.md`
自动预检结论: 已生成 Decision Brief；发起前请以 `meta-flow check human-gate --checkpoint process/checkpoints/CP5-CR172-ALL-STORIES-LLD-BATCH.md` 的结果为准。

审批者摘要:
- 本次确认服务的整体目标: 请见 checkpoint `### 审批者摘要`；本消息只承载发起确认所需摘要。
- 推荐动作: 默认推荐 approve，除非你要求调整范围、风险、授权边界或推荐方案。
- approve 后会发生什么: 接受本轮 Decision Brief 中列出的推荐方案，并允许进入 checkpoint 声明的下一阶段。
- approve 不授权什么: 不授权真实运行、凭据读取、安全边界变更、外部接口调用、数据写入、publish、live / 交易类操作。
- 不确认会阻塞什么: 当前 checkpoint 后续推进会保持等待，直到你回复 approve / 修改 / reject。

Context Capsule: 请见 checkpoint 的 `### Context Capsule Summary`，其中包含 capsule、read_profile、默认读取策略和全文档读取边界。
决策收集覆盖: 请见 checkpoint 的 `### Decision Collection Coverage`，本消息只承载发起确认所需摘要。
决策分层:
- 必须用户决策: 4
- 高风险策略确认: 以 checkpoint `### 决策分层` 为准。
- agent 默认处理: 低风险、可逆、局部实现细节默认不进入用户主确认表，完整清单以 checkpoint 为准。
- 仅审计记录: 已按项目规则处理且无需用户拍板的事项只保留审计摘要。
本轮待人工决策项: 4
blocking / high-risk 决策摘要: DQ-CP5-CR172-IMPLEMENTATION-CONTRACT, DQ-CP5-CR172-WAVES-OWNERSHIP, DQ-CP5-CR172-LOCAL-AUTHORIZATION, DQ-CP5-CR172-CLAIM-AND-RUNTIME-BOUNDARY

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP5-CR172-IMPLEMENTATION-CONTRACT | implementation | 是否批准五份 full LLD 作为 PATH-I 实现合同？ | 批准 S01-S05 v1.2，并把 v1.4 pointer-only refresh 设为 CP6 前置。 | 指定 Story/章节修改并保持门禁。 | 推荐方案已通过四轮独立审查且 required=0；备选更细但会阻断全部串行依赖。 | 解锁实现合同；任何规范变化都可能破坏跨 Story seal/authorization 边界。 |
| DQ-CP5-CR172-WAVES-OWNERSHIP | implementation | 是否批准五个串行 Wave 与 12/12 primary owner？ | S01→S02→S03→S04→S05`，每 Wave 一个 Story。 | 调整 DAG/owner 后重跑 CP4。 | 串行吞吐较低，但能避免共享合同、seal、selection 和 QAC 并发漂移。 | 决定 merge 顺序和单写边界。 |
| DQ-CP5-CR172-LOCAL-AUTHORIZATION | security | 是否只授权 12 个计划路径的 repository-local 编辑和本地验证？ | 只授权 4 个 engine 路径、7 个 test 路径/模块和 1 个 fixture root；只允许本地 `uv run`。 | 保持 design-only；额外权限另开人工门禁/CR。 | 推荐是交付 fixture 合同的最小权限；design-only 无法验证实现，扩到真实操作又没有依据。 | 不会产生真实 evidence 或 runtime 能力。 |
| DQ-CP5-CR172-CLAIM-AND-RUNTIME-BOUNDARY | risk_acceptance | 是否接受 fixture-only 验证和 PATH-I design-ready claim ceiling？ | 接受 current-v1 real actions 仍不可用、REQ013 runtime deferred、CP8 最高 `path_i_design_ready=true`。 | 暂停至真实 producer/adapter 获独立授权。 | 推荐先证明合同与 fail-closed 边界；备选独立性更强但会继续悬置基础合同。 | 不会使 C1 computable 或 Stage3 entry-ready。 |

如果你回复 approve，表示你接受以上 4 项推荐方案，不表示授权以下不授权项。
不授权项: 不授权真实运行、凭据读取、安全边界变更、外部接口调用、数据写入、publish、live / 交易类操作；若 checkpoint 中列有额外不授权项，以 checkpoint 为准。

请只回复以下三个 exact 选项之一：
- approve
- 修改: <具体修改点>
- reject
