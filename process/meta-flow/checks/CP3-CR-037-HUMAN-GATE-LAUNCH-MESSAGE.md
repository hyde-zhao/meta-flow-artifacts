CR-037 CP3 蓝图 / HLD / ADR 架构评审准备完成。

checklist 路径: `process/checkpoints/CP3-CR-037-HLD-REVIEW.md`

自动预检结论: `process/checks/CP3-CR-037-HLD-CONSISTENCY.result.json` 为 PASS；CP2 已 approved；CP3 context capsule ready；human-gate 发起前请以 `meta-flow check human-gate --checkpoint process/checkpoints/CP3-CR-037-HLD-REVIEW.md --launch-message-file process/checks/CP3-CR-037-HUMAN-GATE-LAUNCH-MESSAGE.md` 的结果为准。

Context Capsule: `process/context/CP3-CR-037-CONTEXT.yaml`，read_profile=`compact`，默认读取 CP2 approval、Blueprint、Domain Map、Dependency Map、HLD、ADR、Feature Matrix 和 CP3 discussion 证据；不默认读取 quant-lab、完整归档或完整会话 transcript。

审批者摘要:
- 本次确认服务的整体目标: 确认 CR-037 的蓝图、HLD、ADR 和 Feature Design Matrix，允许进入后续 per-feature 设计准备。
- 推荐动作: approve 当前架构基线；自动预检无 blocker。
- approve 后会发生什么: CR-037 可进入 story-planning / Feature 设计准备；后续仍需 CP5 才能进入实现。
- approve 不授权什么: 不授权实现、CP5、runtime、生产写入、publish、live、真实交易、读取凭据、修改 quant-lab 发布库，也不关闭、恢复或完成 CR-036。
- 不确认会阻塞什么: 阻塞 per-feature DESIGN / TEST-PLAN / TASKS、Story 拆解确认和后续 P0/P1/P2 实现。

决策分层:
- 必须用户决策: 2
- 高风险策略确认: 5
- agent 默认处理: 2
- 仅审计记录: 4

决策收集覆盖: 已扫描 STATE / dispatch / 自动预检 / discussion log / 设计基线 / CP2 approval；候选问题数 18，纳入待决策 7，N/A / 缺失来源原因见 checkpoint 的 `### Decision Collection Coverage`。

本轮待人工决策项: 7

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| CP3-CR037-DQ-01 | `architecture` | 是否批准 enforcement-first HLD / Blueprint / ADR 架构基线作为后续 Feature 设计输入？ | 批准方案 A，先 P0 current-state enforcement，再叠加 project governance、roadmap refresh 和 migration。 | 退回修改 HLD；或只批准 P0，P1/P2 后续重开架构审查。 | 推荐方案和 CP2 scope 对齐，依赖顺序清楚。 | 会影响全部 FEAT-PG-001..009 设计输入。 |
| CP3-CR037-DQ-02 | `architecture` | 是否确认 `STATE.current.json` allowlist 与 `PROJECT.current.json` 分层架构？ | current state 只存轻量字段和 `project_state_ref`，长期项目对象进入 `PROJECT.current.json`。 | 先只收紧 current state；或把 project fields 放入 current state。 | 推荐方案保持默认入口瘦身。 | 影响 P0/P1 schema、writer、workspace scaffold。 |
| CP3-CR037-DQ-03 | `security` | 是否确认 roadmap refresh 使用独立 result/checker，且 cascade 只写过程归档库？ | ROADMAP-REFRESH 独立 schema/checker；过程侧自动更新，发布库只输出 stale / FU-RF。 | 只生成报告不 cascade；或未来独立 CR 授权发布库写入。 | 推荐方案语义清楚且权限最小。 | 影响 roadmap、GATE-LEDGER、stale-check 和发布库授权边界。 |
| CP3-CR037-DQ-04 | `implementation` | 是否确认 registry-backed feature/capability refs 与 impact surface normalization 架构？ | refs 引用标准 registry，impact_surface 只保留治理面，路径进入 affected_paths。 | proposed registry entries 后人工确认；或短期 warning 自由字符串。 | 推荐方案恢复冲突检测和迁移可信度。 | 影响 capability claims、CR conflicts 和 quant-lab migration。 |
| CP3-CR037-DQ-05 | `follow_up_tracking` | 是否确认 roadmap follow-up 使用 FU-RF / SP-RF / RA-RF，不进入 RELEASE-CONTEXT？ | 项目治理 follow-up 走 RF 前缀和 CR tracking；release context 只保留发布上下文。 | 复用 FU-CR 前缀；或写入 RELEASE-CONTEXT。 | 推荐方案来源清晰，避免发布上下文污染。 | 影响 cr_tracking regex、模板和状态查询。 |
| CP3-CR037-DQ-06 | `implementation` | 是否确认 9 个 required Feature 的 per-feature DESIGN / TEST-PLAN / TASKS 必须在 CP3 后、CP5 前生成？ | FEAT-PG-001..009 均 required；umbrella 设计包不能替代 per-feature 证据。 | 先只生成 FEAT-PG-001..003；或用 umbrella 设计包直接进入 CP5。 | 推荐方案证据完整，符合 complex 变更风险。 | 影响 CP5 是否可发起和 Story lld_policy。 |
| CP3-CR037-DQ-07 | `architecture` | 后续正式 Story / slice / evidence 命名是否直接使用 CR-A..CR-H，还是映射为正式 CR-037 派生编号并保留 CR-A..CR-H 作为 alias？ | 正式文件和 evidence 命名使用 CR-037 派生 Story / slice ID，CR-A..CR-H 仅作计划 alias。 | 直接使用 CR-A..CR-H；或 CP3 后第一步单独确认命名并阻塞 Story 文件落地。 | 推荐方案避免非正式 CR 编号进入 CR index、Story 和 evidence。 | 影响 Story 文件、Feature design refs、evidence index 和 CR tracking。 |

如果你回复 approve，表示接受上表 7 项推荐方案；不表示授权实现、CP5、runtime、production write、publish、live、真实交易、凭据读取、quant-lab 发布库修改，也不表示关闭、恢复或完成 CR-036。

不授权项:
- 不进入实现或 CP5。
- 不执行 runtime、production write、publish、live 或真实交易。
- 不读取凭据、账户、token、cookie 或私钥。
- 不自动修改 quant-lab 发布库代码、测试、正式设计文档或发布文档。
- 不关闭、恢复或完成 CR-036。

请只回复以下三个 exact 选项之一:

- approve
- 修改: <具体修改点>
- reject
