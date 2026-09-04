请审查人工门禁 `CP2-CR035-REQUIREMENT-BASELINE`。

checklist 路径: `process/checkpoints/CP2-CR035-REQUIREMENT-BASELINE.md`
自动预检结论: 已生成 Decision Brief；发起前请以 `meta-flow check human-gate --checkpoint process/checkpoints/CP2-CR035-REQUIREMENT-BASELINE.md` 的结果为准。

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
blocking / high-risk 决策摘要: DQ-035-01, DQ-035-02, DQ-035-03, DQ-035-04, DQ-035-05

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-035-01 | implementation | 用例"显式要改 TG IP"的标识方式 | step 级 `force_config: true` 标志：case_steps 中 tg_config_interface step 加 `force_config: true`，execute_steps 检测到则真实执行并重置 `tg_interface_configured | A: case 级 frontmatter `tg_reconfig_required: true`（整用例所有 tg_config_interface 真实执行）；B: 独立 op_id `tg_reconfig_interface | 推荐粒度最细（step 级），与 case_steps 契约一致，复用现有 skip 判断分支；A 粗粒度但简单；B 改 op_mapper 范围大、与 config_interface 重复 | 影响 case_steps schema（新增 force_config 字段）+ execute_steps 判断逻辑；低风险（增量字段，向后兼容） |
| DQ-035-02 | implementation | run 级 tg_user_id 生成规则（需同 run 一致、跨 run 唯一） | dev<运行机IP末段>-<run_id短哈希>`（如 `dev172-a3f2`），从运行机 IP + run_id 派生 | A: 从 devices.yaml `tg.api_server` 派生；B: 固定值 `dev-ptm-te | 推荐 cross-run 唯一、多用户并发隔离；A 依赖配置、同机多 run 冲突；B 多 run 共用 user_id 冲突 | 影响 acquire/release 命令 user_id；需确认运行机 IP 获取（socket.gethostname 解析）；低风险 |
| DQ-035-03 | implementation | run 级 tg_acquire_ports 失败时如何处理 | 中断整个 run（类比 `login_ok=False` 中断），记录 acquire 失败，不进入逐用例执行 | A: 降级走 default force 模式（不 acquire，每个 tg op 自动 acquire+release） | 推荐中断安全（避免无占用下跑用例导致端口抢占），与 login_ok 失败语义一致；降级兼容但失去 config-once 核心价值 | 影响 run 容错策略；中断可能掩盖环境问题（dry-run 不触发，仅 --execute 触发） |
| DQ-035-04 | implementation | dry-run 下 tg_config_interface config-once 行为（agent 默认） | 首次 build_command 后置 `tg_interface_configured=True`，后续 step skip（status=skipped），与 fw_login dry-run 一致 | A: 每次都 build_command（不 skip） | 推荐与 --execute 真实行为一致，dry-run 报告可预测；A 简单但不反映真实 skip 行为 | 影响 dry-run 报告准确性；低风险 |
| DQ-035-05 | scope | 是否在 op_mapper 注册 `tg_verify_ownership`（-> `tg trex verify-ownership`） | 注册（与 trex-api `GET /api/v1/ownership` 查询接口对齐，run 中可查询占用状态，便于调试） | A: 不注册（最小范围，EXPECTED_OP_COUNT 23->25 而非 26） | 推荐完整暴露 ownership 查询能力，与已实现服务端/CLI 对齐，调试友好；A 范围最小但缺查询能力 | 影响 EXPECTED_OP_COUNT（26 vs 25）+ tests/test_tg_op_mapping.py 断言；低风险（增量 op） |

如果你回复 approve，表示你接受以上 5 项推荐方案，不表示授权以下不授权项。
不授权项: 不授权真实运行、凭据读取、安全边界变更、外部接口调用、数据写入、publish、live / 交易类操作；若 checkpoint 中列有额外不授权项，以 checkpoint 为准。

请只回复以下三个 exact 选项之一：
- approve
- 修改: <具体修改点>
- reject
