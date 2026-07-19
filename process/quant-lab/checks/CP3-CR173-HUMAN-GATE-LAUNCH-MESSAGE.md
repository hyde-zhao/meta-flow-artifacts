请审查人工门禁 `CP3-CR173-HLD-REVIEW`。

checklist 路径: `process/checkpoints/CP3-CR173-HLD-REVIEW.md`
自动预检结论: 已生成 Decision Brief；发起前请以 `meta-flow check human-gate --checkpoint process/checkpoints/CP3-CR173-HLD-REVIEW.md` 的结果为准。

审批者摘要:
- 本次确认服务的整体目标: 请见 checkpoint `### 审批者摘要`；本消息只承载发起确认所需摘要。
- 推荐动作: 默认推荐 approve，除非你要求调整范围、风险、授权边界或推荐方案。
- approve 后会发生什么: 接受本轮 Decision Brief 中列出的推荐方案，并允许进入 checkpoint 声明的下一阶段。
- approve 不授权什么: 不授权真实运行、凭据读取、安全边界变更、外部接口调用、数据写入、publish、live / 交易类操作。
- 不确认会阻塞什么: 当前 checkpoint 后续推进会保持等待，直到你回复 approve / 修改 / reject。

Context Capsule: 请见 checkpoint 的 `### Context Capsule Summary`，其中包含 capsule、read_profile、默认读取策略和全文档读取边界。
决策收集覆盖: 请见 checkpoint 的 `### Decision Collection Coverage`，本消息只承载发起确认所需摘要。
决策分层:
- 必须用户决策: 2
- 高风险策略确认: 以 checkpoint `### 决策分层` 为准。
- agent 默认处理: 低风险、可逆、局部实现细节默认不进入用户主确认表，完整清单以 checkpoint 为准。
- 仅审计记录: 已按项目规则处理且无需用户拍板的事项只保留审计摘要。
本轮待人工决策项: 2
blocking / high-risk 决策摘要: DQ-CR173-CP3-001, DQ-CR173-CP3-002

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CR173-CP3-001 | architecture | 是否批准 `spectral_participation_ratio` estimator 及其限定 claim？ | 批准 `n_eff=(tr R)^2/tr(R²)`；只解释为 sealed-trial 二阶相关结构的 effective dimensionality。 | 转 methodology Spike，并让 effective count 全量保持 `typed_unavailable`；未来另评 alpha-specific copula/Li–Ji 类方法。 | 推荐方案在显式有效 `R` 上单值可识别、严格满足 `1≤n_eff≤n`、无随机积分且可确定验证；代价是不能解释 tail/FWER/DSR。备选避免模型误用，但暂不交付 offline count。 | 主要风险是把二阶有效维度误读为 error-rate 或 admission calibration；真实矩阵采样误差仍未解决。 |
| DQ-CR173-CP3-002 | scope | 是否批准 CR-173 收缩为 estimator-only，并把 public C1 projection 拆为后续 CR candidate？ | 批准 split：CR-173 只生成 standalone 七字段 evidence；current C1/Gate-1/admission 继续 unavailable。 | 暂停整个 CR-173，既不做 estimator 也不做 projection。 | 推荐方案保留离线方法价值，并把 `int | None`/4-field → decimal/7-field、跨 4 owner 域的迁移风险隔离；代价是 CR-173 CP8 仍不能让公共 C1 computable。备选回滚面最小，但会悬置方法风险。 |

如果你回复 approve，表示你接受以上 2 项推荐方案，不表示授权以下不授权项。
不授权项: 不授权真实运行、凭据读取、安全边界变更、外部接口调用、数据写入、publish、live / 交易类操作；若 checkpoint 中列有额外不授权项，以 checkpoint 为准。

请只回复以下三个 exact 选项之一：
- approve
- 修改: <具体修改点>
- reject
