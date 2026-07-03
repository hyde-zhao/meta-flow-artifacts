---
artifact: "process/HLD.md"
reviewer: "meta-dev acting_role on default agent"
lane: "lane-implementation"
round: 1
status: draft
governance_mode: review-gated
agent_execution:
  agent_type: "default"
  acting_role: "meta-dev"
  native_custom_agent_loaded: false
---

# Review Findings

## 1. 审查范围

- 目标对象：`process/HLD.md`
- 审查目标：implementation lane 可实施性审查，重点检查 HLD 是否足以支持 CP4 Story 拆解、后续 LLD、schema/docs/CLI/scripts 分层实现、文件影响面、模块边界、接口契约、迁移/回退和测试入口。
- 审查依据：`AGENTS.md` Design Review 规则 3/5/11/12/13、`README.md` 原生交付面与离线只读 CLI 事实、`.codex/agents/meta-dev.toml` review_mode、`review-artifact-protocol` findings 模板。
- 执行身份：当前 API 工具面没有暴露自定义 `agent_type=meta-dev`；本评审由 `agent_type=default, acting_role=meta-dev` 执行，未声称原生 meta-dev custom agent 已加载。

## 2. Findings

<!-- findings-table -->

| ID | Severity | Rule Ref | Evidence | Impact | Suggestion | Anchor |
|----|----------|----------|----------|--------|------------|--------|
| F-001 | required | `AGENTS.md Design Review #3/#13` | `process/HLD.md:103` 将 “schema 扩展字段命名与版本号” 标为 REQUIRED 缺失信息，并说明具体字段名在 CP3 后 ADR/Story 锁定；但推荐方案 B 依赖受控 schema 扩展（`process/HLD.md:123`-`process/HLD.md:135`），且当前 schema 明确 `additionalProperties: false`（`process/HLD.md:76`）。 | CP3 如果直接批准方案 B，CP4 Story 拆解仍无法确定哪些顶层字段或嵌套字段归属 schema/docs/CLI/scripts Story，LLD 也无法判断是否需要迁移现有 atom、schema 版本是否变更、字段参考如何同步，容易在 Story 阶段反复重开设计。 | 在 HLD 或 CP3 审查结论中补齐 schema 扩展决策下限：字段族候选清单、是否变更 `schema_version`、向后兼容策略、现有 atom 迁移要求、字段参考同步范围；若仍延后，应把 CP3 结论改为 “方案 B 原则通过但 ADR-1/字段冻结为 CP4 进入条件”。 | `process/HLD.md:103` |
| F-002 | required | `AGENTS.md Design Review #3/#5` | HLD 把登录状态模型选择为最小化 `session_ref` / `state_ref`（`process/HLD.md:300`），集成契约要求输出 `session_ref`、`state_status`、`expires_hint`、`diag_snapshot_ref`（`process/HLD.md:278`），但需求开放问题要求 HLD 关闭“持久化边界”问题（`process/REQUIREMENTS.md:139`），HLD 只说明“不含敏感认证载荷”，未明确引用是否只存在于 atom `returns.data`、测试用例上下文、文件缓存或外部编排上下文。 | Story/LLD 无法判定哪些文件需要修改：仅写 docs 与 atom 示例、扩展 schema 深层结构、还是修改 CLI/cache 展示逻辑；同时无法为登录守卫 atom 定义可测的输入/输出契约和失败路径。 | 在 HLD 增补状态引用持久化边界表：`state_ref`/`session_ref` 的生命周期、生成方、消费方、允许落盘位置、禁止字段、过期判定字段、CLI 是否解析/展示该字段；并明确当前版本是否仅作为 atom 返回契约，不进入 CLI 本地缓存元数据。 | `process/HLD.md:278` |
| F-003 | required | `AGENTS.md Design Review #3/#5` | HLD 的 Repository Checks 模块声称校验“布局、schema、敏感信息和 package 引用完整性”（`process/HLD.md:268`），NFR 只给出 `rg` 敏感模式扫描 + review checklist（`process/HLD.md:386`），阶段 4 又要求 `scripts/` 增加敏感与 gate 检查（`process/HLD.md:424`）；README 当前原生脚本入口只有 `scripts/layout_check.py` 和 `scripts/validate_schema.py`（`README.md:43`-`README.md:44`、`README.md:204`-`README.md:206`）。 | 后续 Story 无法判断应修改现有脚本、增加新脚本，还是只在 CI/人工 review 中执行 `rg`；CP6/CP7 验证入口也缺少稳定命令，导致“敏感信息零落盘”和“高风险 gate 100%”无法被机器化验收。 | 在 HLD 的阶段 4 或 NFR 中固定最小验证入口，例如新增 `scripts/security_gate_check.py` 或扩展 `scripts/layout_check.py`，列出检查对象、失败退出码、命令入口和至少 1 条敏感模式/高风险 gate 判定规则；同时说明是否纳入 `uv run --python 3.11` 验证链。 | `process/HLD.md:424` |
| F-004 | optional | `AGENTS.md Design Review #11` | HLD §11 分为 5 个阶段（`process/HLD.md:421`-`process/HLD.md:425`），§12 粗估为 5 个 Story/3 个 Wave（`process/HLD.md:431`-`process/HLD.md:436`）；其中 “安装/初始化/登录/守卫 atom 与 package” 被合并为 1 个 Story（`process/HLD.md:432`），capacity 10 个配置域也被合并为 1 个 Story（`process/HLD.md:433`）。 | 作为 HLD 粗估可以接受，但 CP4 若照此直接生成 Story，单个 Story 会同时拥有多个 atom、package、docs 和状态契约文件，LLD 的文件所有权、TASK-ID、验收粒度和并行开发边界会偏粗，容易把可并行工作串行化。 | CP4 Story planning 前增加拆分规则：安装/初始化/登录/守卫可按链路合并但需列出独立 TASK-ID 和文件所有权；capacity Story 若覆盖 10 域，应明确首批域清单、共享模板和可拆分条件，避免 LLD 阶段再临时裁剪。 | `process/HLD.md:432` |

## 3. 汇总结论

- blocking_count: 0
- required_count: 3
- optional_count: 1
- recommended_next_action: `revise-and-resubmit`
- highest_severity: `required`
- cp3_approval_recommendation: `do-not-approve-until-required-findings-addressed`

## 4. 待确认项

- Q-001：CP3 是否要求在 HLD 内冻结 schema 字段族和版本策略，还是允许把 ADR-1/字段冻结作为 CP4 进入条件。
- Q-002：登录状态引用当前版本是否仅作为 atom 返回契约存在，还是需要 CLI/cache 层读取、展示或校验。
