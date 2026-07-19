请审查人工门禁 `CP2-CR052-VNEXT-R2-PRODUCT-BASELINE`。

checklist 路径: `process/checkpoints/CP2-CR052-VNEXT-R2-PRODUCT-BASELINE.md`
自动预检结论: 已生成 Decision Brief；发起前请以 `meta-flow check human-gate --checkpoint process/checkpoints/CP2-CR052-VNEXT-R2-PRODUCT-BASELINE.md` 的结果为准。

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
blocking / high-risk 决策摘要: DQ-VNEXT-01, DQ-VNEXT-02, DQ-VNEXT-03, DQ-VNEXT-04, DQ-VNEXT-05

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-VNEXT-01 | architecture | 发布库复用现有源码/交付仓，还是另建第三仓或共享发布仓？ | 复用现有仓；每项目只新增独立过程仓，总计恰好 2 个逻辑仓。 | A. 新建专用发布仓；B. 多项目共享发布仓。 | 推荐仓库最少且发布代码/文档同 commit；第三仓同步复杂，共享仓违背隔离目标。 | 决定初始化、权限、迁移和发布证据绑定。 |
| DQ-VNEXT-02 | architecture | process main-only 如何避免多 Agent/设备竞争同一 ref？ | 单写 publisher 串行提交 main，fresh expected-OID CAS；漂移拒绝，不自动 merge。 | A. per-Work branch；B. 多写者自动 merge。 | 推荐状态少、失败清晰；分支恢复治理成本，自动 merge 风险最高。 | 决定并发吞吐、幂等、恢复和 receipt。 |
| DQ-VNEXT-03 | scope | 是否拆分旧 Git 历史或把旧 CP/CR/Story 无损转换？ | 只迁当前快照；旧仓永久只读审计，默认兼容读取在观察期后关闭；不拆史、不转换、双写=0。 | A. 按项目 rewrite 历史；B. 自动转换旧对象。 | 推荐风险最低且回滚直接；拆史改 OID，转换易制造伪语义。 | 决定迁移工具、审计、兼容期和回滚。 |
| DQ-VNEXT-04 | implementation | G0/G1 的 read/write/check/token 硬上限取值？ | G0=8/8/3/32k；G1=20/24/8/96k；高风险或超限前进入 G2。 | A. 更紧的 5/5/3/20k 与 10/16/6/40k；B. 只告警不设硬上限。 | 推荐有可验收硬边界并允许试点校准；更紧可能 underfit，只告警不能止住失控。 | 决定上下文、检查颗粒度、升级频率和 token 成本。 |
| DQ-VNEXT-05 | scope | 首版试点需要几个项目、几个 Work 周期和多长观察期？ | 2 项目×各至少2周期；meta-flow+1个切换前用户批准项目；各覆盖 G0/G1；观察30天。 | A. 仅 meta-flow；B. 一次迁移全部项目。 | 推荐是验证跨项目隔离的最小样本；单项目证据不足，批量迁移爆炸半径过大。 | 决定验收时间、真实仓授权、回滚和旧仓退役判断。 |

如果你回复 approve，表示你接受以上 5 项推荐方案，不表示授权以下不授权项。
不授权项: 不授权真实运行、凭据读取、安全边界变更、外部接口调用、数据写入、publish、live / 交易类操作；若 checkpoint 中列有额外不授权项，以 checkpoint 为准。

请只回复以下三个 exact 选项之一：
- approve
- 修改: <具体修改点>
- reject
