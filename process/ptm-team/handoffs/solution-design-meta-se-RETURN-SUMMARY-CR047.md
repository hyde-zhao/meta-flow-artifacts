# 交还摘要 - solution-design（meta-se）-> host-orchestrator

- CR：CR-047（case_runner post_delay 独立回源）
- 阶段：solution-design（CP3 HLD N/A 审计 + Story 拆解 + technical-note 设计证据起草）
- 委托：host-orchestrator 委托 meta-se（HANDOFF-CR047-SD-META-SE-001，standard 模式，DQ-047-02=A）
- 交还时间：2026-08-19
- 下一动作：host-orchestrator 消费本摘要写 CP3 result（预判 N/A 仅审计确认）-> CP4 自动预检 -> CP5 technical-note 设计证据确认
- 审核模式：回源型 CR（工作区已实测 2 用例 26/28 step PASS，随 CR-039 验证载体），static-only 验证

---

## 1. CP3 HLD 适用性审计结论

- **判定：HLD N/A**（无架构设计内容），不产出 `docs/design/HLD.md`，不修改任何正式设计对象。
- **理由**：CR-047 为回源型 CR，post_delay 已在工作区实现并实测通过，canonical 落地仅为单函数 + 单行睡眠叠加 + 单测 + 一行文档，无新架构风格、无新组件边界、无跨 Feature/Epic 边界、无数据归属变更、无依赖方向设计（同 CR-039 CP3 N/A 与 CR-046 retroactive 先例）。

### 1.1 正式设计对象逐项 N/A 判定

| 设计对象 | 判定 | N/A 理由（一句话） |
|---|---|---|
| HLD.md | N/A | 回源型改动无新架构内容，`_parse_post_delay` 为 case_runner 模块内私有函数，睡眠行叠加复用既有 `OP_INTERVAL_SEC` 间隔机制（HLD §13.1 语义不变，post_delay 为其上的 step 级增量参数）。 |
| BLUEPRINT.md | N/A | 无新 Feature/Epic 能力边界，post_delay 属既有 case-execution Skill 的 step 级参数扩展，能力归属不变。 |
| DOMAIN-MAP.md | N/A | 无新领域对象/状态流/持久化边界，post_delay 为执行期瞬时参数，不落任何数据模型。 |
| DEPENDENCY-MAP.md | N/A | 无跨模块新依赖方向，改动收敛在 case_runner.py 单文件内，不新增 import 与调用方向。 |

- **Architecture Gray Areas**：N/A。范围语义决策（DQ-047-01 补 SKILL.md / DQ-047-02 standard / SGQ-047-03 并行推进基线 / SGQ-047-04 逐块一致）已全部在 CP2 由用户 approve，本阶段无新增架构灰区，不进入 advisor table 流程。
- **Feature 设计矩阵**：N/A。单 Story、`feature_design_refs=[]`、`lld_policy.required_level=technical-note`，不产出 FEATURE-DESIGN-MATRIX 增量与 `docs/features/` 三件套。

## 2. Story 拆解说明

单 Story `STORY-047-01`（单 Wave、无依赖、无并行需求），卡片：`process/stories/STORY-047-01.md`。

| 字段 | 值 |
|---|---|
| 文件所有权（primary） | `skills/case-execution/scripts/case_runner.py` + `tests/test_case_runner_post_delay.py`（新建）+ `tests/test_tg_op_mapping.py`（守护反转）+ `skills/case-execution/SKILL.md`（一行） |
| lld_policy | technical-note（trigger=workspace-verified-backport） |
| tier / shared_fragments | T3 / []（无共享片段） |
| 设计证据 | Story 卡片 `## 技术说明`（meta-se 委托内起草，CP5 待确认） |
| 依赖 / Wave | 无依赖 / W1 单 Wave |
| CP2 基线落地 | In 4 项全部映射到卡片验收标准（SM-047-01~04）；Out 6 项写入 security_constraints（rollback 不动 / op 口径不变 / 无语义扩展） |

技术说明覆盖（满足 lld-check technical-note 语义要点 + 卡片含「数据」「权限」token）：文件影响范围（符号定位三块）、接口/函数设计（`_parse_post_delay(val) -> float` 签名与 None/数字/"s"/"ms"/纯数字串解析规则、execute_steps `idx < n - 1` 分支睡眠叠加、rollback 不动）、异常处理（None/空串/非法串 -> 0.0；负数透传，OPI-047-01 固化）、测试设计（10 条用例清单，≥8 满足 + `test_tg_op_mapping.py:622-626` 守护单测语义反转：assertNotIn -> 定义 + 恰好 1 调用点）、实施步骤（6 步）、回滚策略（git revert + 守护反向恢复）、数据（无持久化）、权限（无边界变更）、风险（RA-047-01~03）。

### 2.1 开放项（open_items）

- **OPI-047-01（open，非阻塞）**：负数 post_delay 按 SGQ-047-04（与工作区实测一致）原样透传（`_parse_post_delay(-1) == -1.0`）；当负值绝对值 ≥ `OP_INTERVAL_SEC` 时 sleep 总和为负会抛 `ValueError`。本 Story 不做 clamp（回源逐块一致原则），clamp 语义扩展随「post_delay 语义扩展」deferred 项记入 BACKLOG。

### 2.2 与其他 CR 的关系（写入 DEV-PLAN security_constraints）

- **CR-036（active，收尾态）**：声明 case_runner.py 所有权重叠，但 A/B/C/D 代码已全部落地（CP6/CP7 PASS）仅剩流程收尾不再改代码，**无写冲突**；用户已确认并行推进（2026-08-19），CR-036 收尾验证基线需注明包含 post_delay 增量（SGQ-047-03 agent 默认处理）。
- **CR-039（closed）**：精细排除已验证（canonical grep post_delay 零残留 + 守护单测存在），本 CR 是唯一回源通道。
- **CR-046（active，档案收尾态）**：无后续开发，无冲突。

## 3. DEVELOPMENT-PLAN 条目

- **路径**：`process/DEVELOPMENT-PLAN-CR-047.yaml`（新建，CR-047 机器真相源）。
- **委托偏差说明**：委托原文写「process/DEVELOPMENT-PLAN.yaml 增量追加」，但该文件为 CR-033 的项目级真相源（project_id=CR-033，ST-EX-* 体系），CR-034/037/038/039 起仓库既有惯例为每 CR 独立 `DEVELOPMENT-PLAN-CR-<NN>.yaml`（CR-039 交还摘要 §9 同款说明）。本 CR 按既有惯例新建 CR 后缀文件，**未改动** `process/DEVELOPMENT-PLAN.yaml` 既有内容（保留既有内容的委托意图由此满足）。
- 条目内容：单 Wave（W1 post_delay 回源）/ STORY-047-01 单 Story 无依赖 / 文件所有权四文件 / technical-note / security_constraints 承载 CP2 基线 + CR-036 重叠无冲突说明 + 符号定位与逐块合并约束。

## 4. CP5 Decision Brief 输入

- **预期待人工决策项数量：0**。
- **为 0 的原因**：本阶段（solution-design）不引入任何新 DQ。全部范围与策略决策已在 CP2 由用户 approve（DQ-047-01=A 补 SKILL.md 一行 / DQ-047-02=A standard）；实现细节类事项（函数落点、单测文件组织、守护反转写法、符号定位口径、rollback 守护、dry-run 断言）均属 CP2 已 approve 的「agent 默认处理」6 项范畴；唯一的实现灰区（负数透传 vs clamp）受 SGQ-047-04「与工作区实测一致」约束，无裁量空间，以 open_items（OPI-047-01）+ BACKLOG deferred 记录，不构成用户决策项。
- **CP5 门禁仍为人工门禁**（DQ-047-02=A standard 全链路保留）：发起时 Decision Brief 决策分层预期为「必须用户决策 0 项 / 高风险策略确认 0 项 / agent 默认处理沿用 CP2 既有 6+2 项 / 仅审计记录：CP3 四对象 N/A + 单 Story 拆解 + technical-note 设计证据确认」。
- **不授权项（沿用 CP2）**：真机三不授权（trex 发流 / SSH 重启 trex-stl-te / sw3 telnet）、凭据接触（ADR-02）、git commit / push；本 CR 全程 static-only。

## 5. 风险

- RA-047-01：负数 post_delay 透传，极端负值使 sleep 总和为负抛 `ValueError` -> 缓解：OPI-047-01 + 单测固化行为 + BACKLOG deferred clamp。
- RA-047-02：工作区锚点行号漂移（L2521 vs BACKLOG 卡片 L2437-2440）-> 缓解：符号定位口径（`OP_INTERVAL_SEC` 常量 + `idx < n - 1` 分支睡眠行），CR-036 同款约束。
- RA-047-03：守护单测反转遗漏导致守护与源码失配 -> 缓解：SM-047-03 纳入验收，反转与回源同一实现块完成。

## 6. 未决项

- 无 BLOCKING 未决项。CP3 result / CP4 自动预检 / CP5 人工门禁均待 host-orchestrator 发起（meta-se 不发起任何人工门禁，委托约束）。

## 7. 阶段产物路径清单

- `process/stories/STORY-047-01.md`（Story 卡片 + technical-note 设计证据）
- `process/DEVELOPMENT-PLAN-CR-047.yaml`（CR-047 机器真相源）
- `process/handoffs/solution-design-meta-se-RETURN-SUMMARY-CR047.md`（本交还摘要）
