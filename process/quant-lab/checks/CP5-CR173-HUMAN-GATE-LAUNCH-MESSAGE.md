请审查人工门禁 `CP5-CR173-ALL-STORIES-LLD-BATCH`。

checklist 路径: `process/checkpoints/CP5-CR173-ALL-STORIES-LLD-BATCH.md`
自动预检结论: 已生成 Decision Brief；发起前请以 `meta-flow check human-gate --checkpoint process/checkpoints/CP5-CR173-ALL-STORIES-LLD-BATCH.md` 的结果为准。

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
blocking / high-risk 决策摘要: DQ-CP5-CR173-IMPLEMENTATION-CONTRACT, DQ-CP5-CR173-WAVES-OWNERSHIP, DQ-CP5-CR173-LOCAL-AUTHORIZATION, DQ-CP5-CR173-CLAIM-AND-VERIFICATION-BOUNDARY

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP5-CR173-IMPLEMENTATION-CONTRACT | implementation | 是否批准三份 full LLD 作为 CR173 实现合同？ | 批准 S01 v1.2、S02 v1.2、S03 v1.3，并接受已披露的非阻断 authority-pointer 新鲜度项。 | 指定需修改的 Story/章节并保持门禁。 | 推荐方案已通过三轮独立评审且 required=0；继续修改会延后实现，但可进一步清理纯审计指针。 | 解锁设计确认；任何规范语义变化都可能破坏 computation identity 或验证 oracle。 |
| DQ-CP5-CR173-WAVES-OWNERSHIP | implementation | 是否批准三串行 Wave 与 7/7 文件 owner？ | S01 → S02 → S03，每 Wave 一个 Story，primary path `2+2+3=7` 唯一。 | 调整 DAG/owner 后重跑 CP4。 | 串行吞吐较低，但避免合同、算法、fixture 并发漂移和跨 Story 写冲突。 | 决定 merge 顺序和单写边界。 |
| DQ-CP5-CR173-LOCAL-AUTHORIZATION | security | 是否只授权 7 个计划路径的 repository-local 编辑和本地验证？ | 只授权两个 engine 文件、四个 test 文件/模块和一个 synthetic golden fixture；只允许本地 `uv run`。 | 保持 design-only；额外权限另开人工门禁/CR。 | 推荐是完成离线方法的最小权限；design-only 无法交付，扩大到真实/runtime/public 没有必要且越界。 | 解锁 CP6/CP7，但不产生真实 evidence、public C1 或 Stage 3 能力。 |
| DQ-CP5-CR173-CLAIM-AND-VERIFICATION-BOUNDARY | risk_acceptance | 是否接受 fixture-only 验证和 estimator-only claim ceiling？ | 由 meta-qa-critical 独立验证 6×3、8/8、7/7、9+6、8+12；保持 `public_effective_trial_count_populatable=false`、`c1_computable=false`。 | 暂停至 empirical matrix 或 public projection 另行获批。 | 推荐可先证明方法合同与安全边界；备选能等待更多外部证据但会把方法学风险继续悬置。 | participation ratio 仍不是 FWER/DSR 校正数；public C1 blocker 不解除。 |

如果你回复 approve，表示你接受以上 4 项推荐方案，不表示授权以下不授权项。
不授权项: 不授权真实运行、凭据读取、安全边界变更、外部接口调用、数据写入、publish、live / 交易类操作；若 checkpoint 中列有额外不授权项，以 checkpoint 为准。

请只回复以下三个 exact 选项之一：
- approve
- 修改: <具体修改点>
- reject
