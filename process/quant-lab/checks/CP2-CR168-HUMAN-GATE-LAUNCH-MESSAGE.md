请审查人工门禁 `CP2-CR168-REQUIREMENTS-BASELINE`。

checklist 路径: `process/checkpoints/CP2-CR168-REQUIREMENTS-BASELINE.md`
自动预检结论: 已生成 Decision Brief；发起前请以 `meta-flow check human-gate --checkpoint process/checkpoints/CP2-CR168-REQUIREMENTS-BASELINE.md` 的结果为准。

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
blocking / high-risk 决策摘要: DQ-CP2-CR168-METHOD, DQ-CP2-CR168-C3-C4, DQ-CP2-CR168-GATE4, DQ-CP2-CR168-FIXTURE, DQ-CP2-CR168-CLAIM

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP2-CR168-METHOD | scope | C3 是否包含 transparent impact approximation？ | 纳入 fee/tax/spread/slippage/impact 分解，但 impact 只使用显式 synthetic/static 参数，并输出 `cost_underestimation_status`、limitations、`no_real_tca_claim=true`。 | 只计算 fee/tax/spread/slippage，把 impact approximation 延后到独立 CR。 | 推荐方案给 Gate 4 提供完整 C3 cost/impact 语义，审计价值高；代价是 CP3 要冻结方法族、参数与风险状态。备选更简单，但 C3 对 impact 的解释能力和 Gate 4 兼容价值下降。 | 影响 schema、10 类负向场景、fixture 和 Gate 4 C3 fields；任何真实数据校准仍被禁止。 |
| DQ-CP2-CR168-C3-C4 | architecture | 是否在本 CR 冻结 C3/C4 最小共享输入 header？ | 冻结 identity、basis、unit/currency/calendar、lineage/auth 的最小共享 header；C4-exclusive capacity/ADV/liquidity/alpha-decay 字段保持 reserved，C4 calculator=0。 | C3 完全独立设计；C4 后续再建立共享合同并接受迁移/兼容成本。 | 推荐方案减少未来 C4 重复和 envelope 迁移，同时通过 owner 表防止预占；备选局部更简单，但未来字段重命名、迁移和兼容测试成本更高。 | 影响公共 schema、字段 owner、版本演进和 CP3 ADR；若边界失控可能把 C4 偷渡进 CR168。 |
| DQ-CP2-CR168-GATE4 | implementation | existing-gate integration 在 CR168 做到何种粒度？ | 实现 `1` 条 C3-to-Gate-4 compatibility projection；只填 C3 四字段；C4 `reserved/not-built/typed_unavailable` 必须映射为三个 C4 refs absent，且不得输出字段级 `*_na_reason` / `*_n_a_reason` 或通用 `na_reason` / `n_a_reason`。任何 reason 逃逸输入由 projection `BLOCKED/REJECTED`；不得修改 canonical Gate 4 validator 或 aggregate orchestration；capacity/aggregate PASS=0。新增 P0 `SC-CR168-B02`，原 B01 保留。 | CR168 只输出 C3 component，把所有 Gate 4 projection 延后 `FU-CR161-007`。 | 推荐方案能验证 C3 contract 与现有 consumer 的真实兼容性，并在 projection 侧封死 canonical Gate 4 的 N/A PASS 逃逸；代价是 CP3 必须冻结完整 availability→flat-payload 映射和 reason-key denylist。备选隔离更强，但无法在本 CR 证明现有 Gate 4 消费契约。 | 影响 cross-strategy reliability Gate 4 的适配测试；错误翻译会虚假产生 Gate 4/capacity/aggregate PASS。canonical Gate 4 与 aggregate orchestration 修改数必须为 0。 |
| DQ-CP2-CR168-FIXTURE | scope | fixture 适用面是否采用两族？ | 精确采用 `2/2`：daily multifactor synthetic；daily multifactor + ML multi-strategy-type compatibility。event-specific 明确 N/A/deferred。 | 只保留 daily multifactor synthetic 一族，把 ML compatibility 延后。 | 推荐方案验证 C3 算术不依赖 strategy type，并复用既有 ML package contract；代价是多一组 attach/compatibility 验证。备选范围更小，但不能证明 neutral envelope 的跨策略一致性。 | 影响 TEST-MATRIX、兼容性 claim 和后续 Story；不授权真实 model training 或 event feed。 |
| DQ-CP2-CR168-CLAIM | security | 是否批准当前 claim ceiling 与不授权边界？ | 保持 Stage2 complete=true、Stage3 started=false；在实际完成前 C3 foundation=false；真实 TCA/calibration/data/runtime=false；C4=0、event=false、CR155 promotion=false。 | 暂缓 CR168 并维持 `economic_cost@reserved`，直到用户愿意批准该保守 ceiling；不提供扩大真实数据/runtime 的 CP2 选项。 | 推荐方案允许安全进入 CP3，且防止 fixture 被误读为真实能力；备选零风险但不产生 C3 foundation。 | 这是高风险安全/声明边界。任何真实数据、TCA、runtime 或 CR155 提升都需要新授权和新 CR。 |

如果你回复 approve，表示你接受以上 5 项推荐方案，不表示授权以下不授权项。
不授权项: 不授权真实运行、凭据读取、安全边界变更、外部接口调用、数据写入、publish、live / 交易类操作；若 checkpoint 中列有额外不授权项，以 checkpoint 为准。

请只回复以下三个 exact 选项之一：
- approve
- 修改: <具体修改点>
- reject
