---
change_id: "CR-030-REV-02"
parent_cr: "CR-030"
type: "external-read-persistence-and-analysis-scope-expansion"
status: "active"
rollback_to: "requirement-clarification"
required_gate: "CP2"
created_at: "2026-07-15T09:15:00+00:00"
trigger: "用户要求 ptm-tse 获取、保存并分析 ITR 现网问题单，输出总结和改进措施。"
---

# CR-030-REV-02：ITR 问题单摄取、保存与分析影响分析

## 变更内容

用户提供 ITR 问题单列表请求，要求 `ptm-tse`：

1. 受控获取 TGFW 现网问题单；
2. 保存问题单信息和摄取追溯信息；
3. 依据问题单进行逐单与批量逆向分析；
4. 输出人工可审查的分析总结；
5. 输出可追溯、需人工确认的改进措施；
6. 在必要时扩充分析字段。

指定来源仅为：

`GET http://10.113.53.108/itr/v1/itrs?startDate=2026-01-01&endDate=2026-12-31&pageSize=10000&current=1&product=TGFW`

## 合并而非新建 CR 的理由

该请求服务于同一个 `ptm-tse` 逆向分析目标、相同用户旅程和同一份 HLD / 多 Story 交付价值。它改变了 CR-030 的输入获得方式和数据边界，但不形成独立审批人、发布节奏或回滚边界，因此作为活跃 `CR-030` 的 REV-02 处理，不另起并行正式 CR。

## 五维度影响分析

| 维度 | 是否受影响 | 受影响对象 | 处理结论 |
|---|---|---|---|
| 需求层 | 是 | 使用场景、REQ、数据/分析输出 | 新增“获取并保存问题单”“逐单/批量分析总结”“改进措施”用户结果与验收条件。 |
| 场景层 | 是 | SCN、TEST-MATRIX | 新增成功摄取、分页/重复、响应 schema 不匹配、网络失败、敏感字段、保存失败、证据不足、总结/措施需 review 等场景。 |
| 计划层 | 是 | Story Map、MVP、release slice、HLD 输入 | 新增摄取/持久化/规范化、分析输出和改进措施三段能力，原 CP3 设计不可复用。 |
| 安全层 | 是 | 外部 HTTP 读取、本地数据保存、AI 消费数据 | 原“无外部读取”边界被定向替换为“单一 ITR 来源、GET-only、无凭据、无写入、待 CP2 授权”；响应数据分类、保留、脱敏与存储位置为阻断决策。 |
| 交付层 | 是 | ptm-tse Agent/Skills、模板、存储契约、fixture、文档 | 将需要新增摄取适配器、存储契约、schema 校验、总结/措施模板和隔离测试；未授权前不实现或执行。 |

## 推荐能力边界

| 能力 | 推荐行为 | 明确不做 |
|---|---|---|
| ITR 摄取 | 仅 allowlist 的 HTTP `GET`；参数、拉取时间、HTTP 状态和响应校验值可追溯；没有提供认证信息时不推断认证头或读取凭据。 | POST/PUT/PATCH/DELETE、其他主机/路径、自动重试风暴、认证探测。 |
| 保存 | 保存原始响应快照引用与规范化问题单记录；每条记录关联来源请求、拉取时间、payload hash、schema 版本和同步状态。 | 将原始问题单内容提交 Git、写入 `process/` 状态文档、无限保留、静默覆盖历史快照。 |
| 分析 | 分离事实、假设、反证和人工确认；逐单输出问题摘要/根因候选/引入点/流出点/风险；批量输出模式、趋势与优先级。 | 将推测写成事实、用单一问题单自动确认根因、代替事故恢复。 |
| 改进措施 | 输出 CA/PA 候选，包含目标、依据、Owner、优先级、验收标准、验证方式、状态和 reviewer 决策。 | 自动创建下游任务、自动改测试资产、自动关闭或执行生产修复。 |

## 候选数据字段

具体 ITR 字段名必须在 CP2 批准后，通过一次受控读取取得响应 schema 后映射；不得现在假定其 ID、状态、严重度或文本字段名称。

| 对象 | 建议字段 | 目的 |
|---|---|---|
| 摄取记录 | `source_request`、`retrieved_at`、`http_status`、`payload_hash`、`snapshot_ref`、`schema_version`、`sync_status` | 证明问题单来自何处、何时取得、是否完整。 |
| 规范化问题单 | `source_ticket_id`、`source_updated_at`、`product`、`severity`、`status`、`title`、`description_ref`、`raw_record_ref` | 在不丢失来源的前提下支持去重、增量和分析。 |
| 分析记录 | `analysis_id`、`ticket_ref`、`facts`、`hypotheses`、`evidence_refs`、`root_cause_candidate`、`introduction_point`、`escape_point`、`analysis_summary`、`review_status` | 形成可审计、可反证、待人工确认的逆向分析。 |
| 改进措施 | `measure_id`、`analysis_ref`、`type`（CA/PA）、`target`、`rationale`、`owner`、`priority`、`acceptance_criteria`、`effectiveness_check`、`status`、`review_ref` | 把结论转成可验证但不自动执行的改进输入。 |

## 文档处理决策

| 受影响文档 | 处理方式 | 旧基线保留方式 | 审批前状态 |
|---|---|---|---|
| `docs/product/USE-CASES.md` | 原文档更新 | 保留 REV-01 六个用户旅程，新增/扩展摄取、总结和改进旅程并追加修订记录 | pending |
| `docs/product/REQUIREMENTS.md` | 原文档更新 | 保留 REQ-RA-001..013，新增连续编号需求与验收条件 | pending |
| `docs/product/SCENARIOS.yaml` | 原文档更新 | 保留 SCN-RA-01..07，新增摄取/保存/失败/审查场景 | pending |
| `docs/product/TEST-MATRIX.md` | 原文档更新 | 保留已有映射，追加新 SCN/REQ/Story 覆盖 | pending |
| `docs/product/STORY-MAP.md` | 原文档更新 | 保留用户活动与 Story，新增数据摄取和分析交付用户任务 | pending |
| `docs/product/MVP-SCOPE.md`、`RELEASE-SLICES.md`、`BACKLOG.md` | 原文档更新 | 保留 REV-01 范围，记录 ITR 读取/保存的授权边界和后续集成项 | pending |
| `docs/design/*`、`agents/ptm-tse.md`、新 Skill/模板 | 原文档更新 / 新增 | 只在新的 CP2 和 CP3 后设计，旧 CP3 保留为历史证据 | blocked by CP2 |

## 必须人工确认的决策

| Decision ID | 类型 | 待确认问题 | 推荐方案 | 备选方案 | 风险与回退 |
|---|---|---|---|---|---|
| CP2-DQ-CR030-REV02-01 | runtime_authorization | 是否允许读取用户指定 ITR 接口？ | 仅允许该固定来源的 GET；无凭据、无自定义认证头、无外部写入；先在 CP2 后做一次受控 schema 探测。 | 保持人工导入；或另建独立 runtime CR。 | HTTP 非 TLS 或 schema/数据分类不满足时，回退人工提供的脱敏摘要。 |
| CP2-DQ-CR030-REV02-02 | security | 如何保存可能含敏感信息的问题单？ | 原始快照与规范化记录存放在非 Git、访问受限的专用数据区；保留期和脱敏规则由数据 Owner 确认后配置。 | 仅保存脱敏规范化字段；或不保存原始快照。 | 数据分类/保留不明确时，禁止持久化原始响应，只保留人工脱敏摘要。 |
| CP2-DQ-CR030-REV02-03 | scope | 分析与改进输出的自动化边界是什么？ | 自动生成草案、批量趋势和 CA/PA 候选；人工 reviewer 才能确认、分发或关闭。 | 只生成总结；或允许自动创建下游任务。 | reviewer 不确认时保持草案；禁止自动执行任何下游或生产动作。 |

## 回退与门禁

- `impact_level`: high
- `workflow_mode_after_change`: standard
- `rollback_to`: requirement-clarification
- `product_baseline_refresh_required`: true
- `required_gate`: CP2
- `block_story_decomposition_until`: CP2-approved
- CP1、CP2、CP3 的 CR-030 REV-01 结果仅保留历史审计，不可用作本修订设计依据。
- 新 CP2 未批准前：不调用 ITR，不创建持久化数据区，不更新 HLD/蓝图/Story，也不开始实现。
