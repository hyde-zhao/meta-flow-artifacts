请审查人工门禁 `CP5-CR163-ALL-STORIES-LLD-BATCH`。

checklist 路径: `process/checkpoints/CP5-CR163-ALL-STORIES-LLD-BATCH.md`
自动预检结论: 已生成 Decision Brief；发起前请以 `meta-flow check human-gate --checkpoint process/checkpoints/CP5-CR163-ALL-STORIES-LLD-BATCH.md` 的结果为准。

审批者摘要:
- 本次确认服务的整体目标: 请见 checkpoint `### 审批者摘要`；本消息只承载发起确认所需摘要。
- 推荐动作: 默认推荐 approve，除非你要求调整范围、风险、授权边界或推荐方案。
- approve 后会发生什么: 接受本轮 Decision Brief 中列出的推荐方案，并允许进入 checkpoint 声明的下一阶段。
- approve 不授权什么: 不授权真实运行、凭据读取、安全边界变更、外部接口调用、数据写入、publish、live / 交易类操作。
- 不确认会阻塞什么: 当前 checkpoint 后续推进会保持等待，直到你回复 approve / 修改 / reject。

Context Capsule: 请见 checkpoint 的 `### Context Capsule Summary`，其中包含 capsule、read_profile、默认读取策略和全文档读取边界。
决策收集覆盖: 请见 checkpoint 的 `### Decision Collection Coverage`，本消息只承载发起确认所需摘要。
决策分层:
- 必须用户决策: 5
- 高风险策略确认: 以 checkpoint `### 决策分层` 为准。
- agent 默认处理: 低风险、可逆、局部实现细节默认不进入用户主确认表，完整清单以 checkpoint 为准。
- 仅审计记录: 已按项目规则处理且无需用户拍板的事项只保留审计摘要。
本轮待人工决策项: 5
blocking / high-risk 决策摘要: DQ-CP5-CR163-001, DQ-CP5-CR163-002, DQ-CP5-CR163-003, DQ-CP5-CR163-004, DQ-CP5-CR163-005

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP5-CR163-001 | implementation | 是否批准 S01/S02 core contract、canonical primitive owner、sealed event boundary 与 immutable supersession 设计？ | 批准两份 LLD；core 定义 canonical primitive，store 复用；manifest 固化 event count/last sequence。 | 合并 core/store 为单 Story；或回退 CP3 重设对象边界。 | 推荐维持职责分离且旧版本可复算；合并减少文件但扩大单 Story；回退成本最高。 | 决定所有下游 producer/consumer 的稳定 API 与审计真实性。 |
| DQ-CP5-CR163-002 | implementation | 是否批准 S03 的 2/2 chains、4/4 mappings、orchestration 单 session owner 与 post-hook 单 selection writer？ | 批准；wrapper 不开 session、hook 不写 lineage。 | hook 写 selection；或两层各开 session。 | 推荐消除重复 trial/selection；备选更局部但产生 double-count/ownership 歧义。 | 直接决定 `trial_count` 是否是可信系统事实。 |
| DQ-CP5-CR163-003 | implementation | 是否批准 S04 只复用 CR151/CR154/package，三者消费同一 validation-bound projection并 fail closed？ | 批准；无 native seal=typed_unavailable，mismatch/tamper=blocked，不建新 gate。 | 各 consumer 重算；或新建 CR163 gate。 | 推荐单一事实、最小 policy 面；重算会漂移；新 gate 重复治理。 | 决定 legacy manual count 是否还能绕过 lineage。 |
| DQ-CP5-CR163-004 | implementation | 是否批准 S05 的完整性/恢复/权限/CR155 fixture 设计与 13 个零操作 counter？ | 批准 12/12 场景、5/5 negative classes、10-seal determinism、CR155 blocked/no backfill。 | 缩减为 happy-path 单测；或执行真实研究验证。 | 推荐以 fixture/static 证据覆盖全部风险且不触碰真实 runtime；缩减证据不足；真实运行超授权。 | 决定 tamper、incomplete、supersession 和历史负回归能否独立验证。 |
| DQ-CP5-CR163-005 | security | 是否批准五 Story 全量设计并授权按 DAG 开始源码/测试实现，同时维持 deny-default runtime 边界？ | 批准；W1 S01→W2 S02→W3 S03∥S04→W4 S05，逐 Story CP6/独立 CP7。 | 请求具体 LLD 修改；或暂停 CR163。 | 推荐让实现严格受文件 owner/依赖门控制；修改可降低返工但重跑相关 precheck；暂停最安全但不产生价值。 | approve` 将授权仓库内源码/测试变更，但不会授权真实研究、数据、凭据、交易、发布或统计校正。 |

如果你回复 approve，表示你接受以上 5 项推荐方案，不表示授权以下不授权项。
不授权项: 不授权真实运行、凭据读取、安全边界变更、外部接口调用、数据写入、publish、live / 交易类操作；若 checkpoint 中列有额外不授权项，以 checkpoint 为准。

请只回复以下三个 exact 选项之一：
- approve
- 修改: <具体修改点>
- reject
