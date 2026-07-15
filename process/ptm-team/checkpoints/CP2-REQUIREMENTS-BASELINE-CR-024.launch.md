请审查人工门禁 `CP2-REQUIREMENTS-BASELINE-CR-024`。

checklist 路径: `process/checkpoints/CP2-REQUIREMENTS-BASELINE-CR-024.md`
自动预检结论: 已生成 Decision Brief；发起前请以 `meta-flow check human-gate --checkpoint process/checkpoints/CP2-REQUIREMENTS-BASELINE-CR-024.md` 的结果为准。

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
blocking / high-risk 决策摘要: CR024-DQ-01, CR024-DQ-02, CR024-DQ-03, CR024-DQ-04

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| CR024-DQ-01 | runtime_authorization | runtime 写操作授权边界 | **dry-run 默认门**：首期 CP7 默认 `--dry-run`；`--execute` 写操作作为独立 runtime_authorization 决策项，需用户单次确认 | A: 完全放开 --execute（无人工确认）/ B: 全 dry-run（--execute 留 v2） | dry-run 已验证参数路由和 session 有效性；--execute 涉及真实设备策略变更需单次确认 | 选 A: 设备被意外修改不可追溯；选 B: 回滚/清理逻辑无法验证 |
| CR024-DQ-02 | scope | 验证设备范围 | **hg3250-51**（10.113.55.51，DAS-TGFW-A1300-HU，Web 443） | A: nxp1046-95（10.113.55.95，DAS-TGFW-1900）/ B: 两台都验证 | hg3250-51 是 manaul 已验证策略路由可通设备，风险最低 | 选 A: 不同型号 API 行为可能差异；选 B: CP7 工作量翻倍 |
| CR024-DQ-03 | implementation | ptm-te rule block 注入策略 | **v1 不注入**：执行规则写进 agents/ptm-te.md，不写 AGENTS.md/CLAUDE.md managed block | A: 注入 managed block（参考 ptm-tde 的 render_ptm_tde_rule_body） | ptm-te 首期流程简单，不需跨项目工作区隔离规则；ptm-tde 的 rule block 因三阶段框架+多特性隔离+Gate 才需要 | 选 A: 增加安装器复杂度 + managed block 跨项目残留风险；不注入: 跨项目使用靠 agent md 自述 |
| CR024-DQ-04 | scope | 用例来源 | **手写最小 PC 优先**：首期手写最小策略路由 PC（1 config + 1 verify + 1 delete）验证全链路；真实消费 ptm-tde PC 留 follow-up | A: 真实消费 ptm-tde 已产出 PC / B: 不消费 PC，纯 CLI 命令验证 | 手写最小 PC 覆盖端到端消费链路但不依赖 ptm-tde 产出质量 | 选 A: 验证阻塞在 ptm-tde 侧 args 命名等未锁定项；选 B: case_steps 消费链路不可验证 |

如果你回复 approve，表示你接受以上 4 项推荐方案，不表示授权以下不授权项。
不授权项: 不授权真实运行、凭据读取、安全边界变更、外部接口调用、数据写入、publish、live / 交易类操作；若 checkpoint 中列有额外不授权项，以 checkpoint 为准。

请只回复以下三个 exact 选项之一：
- approve
- 修改: <具体修改点>
- reject
