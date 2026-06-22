# CP2 CR106 Human Gate Launch Message

请审查：`process/checkpoints/CP2-CR106-REDESIGN-BASELINE-HYGIENE-REVIEW.md`

自动预检结论：`PASS`

Context Capsule：`process/context/CP2-CR106-REDESIGN-BASELINE-HYGIENE-CONTEXT.yaml`（read_profile=`minimal`）

本轮待人工决策项：3

决策收集覆盖：已扫描 4 个适用来源，发现候选问题 8 个，去重后纳入待决策 3 个；N/A 来源 2 个，原因见 checklist 的 Decision Collection Coverage。

待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP2-CR106-01 | scope | 是否批准 CR106 只做低风险 baseline hygiene 与只读代码健康预检？ | 批准 CR106 只做 process / ledger / static / readonly health precheck。 | 暂停整改；直接进入 redesign scope intake。 | 推荐方案能用事实判断是否需要代码整改，成本低；备选更保守或更快但跳过健康基线。 | 不改源码，风险低；维护一次预检证据。 |
| DQ-CP2-CR106-02 | implementation | 若只读检查发现失败，是否先形成后续小范围整改 CR，而不是在 CR106 内直接改源码？ | 是，CR106 只产出发现和建议。 | 允许 CR106 内修低风险小问题；只记录问题不创建修复 CR。 | 推荐方案避免 scope creep；备选更快或更保守。 | 审计清晰；代价是可能多一个后续 CR。 |
| DQ-CP2-CR106-03 | runtime_authorization | 是否继续禁止 CR105/runtime/交易写/NAS/凭据/provider publish 被 CR106 隐式启动？ | 是，CR106 不启动 CR105 和任何高风险动作。 | 另起 CR105；另起独立 NAS/runtime gate。 | 推荐方案符合低风险路线；备选能处理高风险需求但需独立门禁。 | 避免误触交易写和凭据边界。 |

不授权项：

- 不启动 CR105。
- 不执行 submit / cancel / buy / sell。
- 不执行 simulation / live。
- 不启动新的 QMT / MiniQMT / XtQuant / gateway runtime。
- 不读取 `.env`、凭据、账户原文、raw log 或未脱敏 runtime evidence。
- 不访问 NAS。
- 不执行 provider fetch、lake write、catalog publish。

该文件包含本检查点的 Entry Criteria、Checklist、Exit Criteria、Deliverables、自动预检摘要、Decision Brief、待人工决策清单和人工审查结果区。

如果你回复 approve，表示你接受上表全部推荐方案，但不表示授权任何不授权项；如需调整，请用 `修改: <具体修改点>` 指明决策 ID 和修改内容。

可回复：

approve

修改: <具体修改点>

reject
