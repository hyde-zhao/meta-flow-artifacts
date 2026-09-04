# 交还摘要 — solution-design（meta-se）→ host-orchestrator

- CR：CR-039（ptm-te 工作区已验证代码回源 canonical：G1 静态路由回滚兜底 + G4 next_hop_a 占位符 + G5 sw3 族）
- 阶段：solution-design（CP3 HLD N/A 判定 + Story 拆解）
- 委托：host-orchestrator 委托 meta-se
- 交还时间：2026-08-18
- 下一动作：host-orchestrator 消费本摘要发起 CP3 人工门禁（HLD N/A + 4 Story 拆解），随后 CP4 自动预检 → CP5 全量 technical-note 设计证据确认
- 审核模式：同步/修复类 CR（`cr_nature=workspace-verified-code-backport-sync-fix`），static-only 验证，工作区已 26/28 step PASS 实测

---

## 1. 推荐 HLD

- **判定：HLD N/A**（无架构设计内容），不产出 `docs/design/HLD.md`。
- **理由**：CR-039 为同步回源 CR，G1/G4/G5 均为工作区已实现并实测通过的代码逻辑回源，无新架构风格、无新组件边界、无跨 Feature/Epic 边界、无数据归属变更、无依赖方向设计。架构语义决策（DQ-039-01 G1 无条件 verify / DQ-039-02 EXPECTED_OP_COUNT=45 口径 / DQ-039-03 追溯 B / DQ-039-05 CR-047 独立回源）已全部在 CP2 由用户 approve，无剩余架构开放项。
- **设计依据**：CR-039.md §需求理解框架 + meta-pm 交还摘要 §3 精确改动集（工作区行号级）。
- **CP3 判定**：`PASS`（`process/checks/CP3-CR039.result.json`，8 项全 PASS，blockers=[]，waivers=[]）。

## 2. 备选方案

| 备选 | 说明 | When to switch |
|---|---|---|
| HLD N/A 为唯一适用路径 | 同步回源 CR 不满足任何 HLD 触发条件 | 若后续出现新架构/跨 Feature 变更，回退到标准 HLD 流程 |
| （治理备选）将 G1/G4/G5 降级为纯手抄补丁 | 不做 Story 拆解，直接 meta-dev 手工合并 | 仅当 Story 拆解带来多余管理成本且无文件冲突时，不建议（本 CR 4 Story 文件所有权已无冲突） |

## 3. 蓝图适用性

- **判定**：`not-applicable`（同步回源不涉及）。
- **理由**：单 CR 内 4 Story 共享既有文件，无新 Feature/Epic 边界、数据归属争议、依赖方向设计或跨 Feature 用户旅程/发布切片。
- **影响范围**：无（不产出蓝图三件套）。
- **后续触发条件**：若后续出现多 Feature/多 Epic 或新领域对象变更，回退蓝图重新判定。

## 4. 蓝图产物路径（逐项 N/A）

| 产物 | 判定 | N/A 原因 |
|---|---|---|
| BLUEPRINT.md | N/A | 无新 Feature/Epic 边界与能力地图需求 |
| DOMAIN-MAP.md | N/A | 无新领域对象/状态流/持久化边界（测试执行领域对象不变） |
| DEPENDENCY-MAP.md | N/A | 无跨模块新依赖方向（sw3 仅消费 case_runner→op_mapper 既有 `map_op_id_to_subcommand` import） |

## 5. Architecture Gray Areas 处理结果

- **N/A**：架构语义决策全部在 CP2 由用户 approve（DQ-039-01/02/03/05），本阶段无新增架构灰区，不进入 advisor table 流程。
- CP2 灰区处理结果引用：`process/discussions/CP2-SCENARIO-DISCUSSION-LOG-CR039.md` + `process/checks/CP2-DISCUSSION-CHECKPOINT-CR039.json`（SGA-039-01~04 / SGQ-039-01~04 已回填）。

## 6. Feature design triggers

- **无 required Feature 设计**：4 Story 全部 `lld_policy.required_level=technical-note`（`feature_design_refs=[]`）。触发原因统一标注 `workspace-verified-backport`（低风险同步回源，工作区已验证行为基线）。技术说明证据由 meta-dev 在 CP5 前写入各 Story 卡片 `## 技术说明`。

## 7. 决策项

- **无新增待决策项**（CP3 不引入新 DQ）。CP2 已 approve 的 DQ-039-01~05 在本阶段落地到 DEV-PLAN security_constraints 与 Story 卡片。

| DQ | 决策 | 落地位置 |
|---|---|---|
| DQ-039-01 | G1 无条件优先 verify（放弃 step-refs fallback） | STORY-039-01 dev_context（含 L705-707 注释语义更新） |
| DQ-039-02 | EXPECTED_OP_COUNT=45 四处载体；op-coverage-matrix total/mapped 不变仅增 sw3 章节 | STORY-039-03（代码/单测）+ STORY-039-04（SKILL.md/op-coverage-matrix） |
| DQ-039-03 | 追溯补记 B（CR-043/044/046 正式变更单 + SKILL.md 修订记录补来源行） | STORY-039-04 |
| DQ-039-04 | 真机验证不授权（static-only） | DEV-PLAN security_constraints |
| DQ-039-05 | CR-047 独立另行回源（B），本 CR 精细排除 post_delay | STORY-039-03（排除）+ STORY-039-04（BACKLOG） |

## 8. CP3 自动预检

- **路径**：`process/checks/CP3-CR039-HLD-NA.md`（人类摘要）+ `process/checks/CP3-CR039.result.json`（机器真相源）。
- **结论**：PASS，8 项全 PASS（HLD N/A 豁免 / 蓝图逐项 N/A / Story 拆解完整性 / design evidence 策略 / 文件所有权与并行安全 / 安全边界 / CR-047 范围缺口 / CP2 决策回写），未豁免 FAIL 数 = 0，waivers=[]。
- **evidence_ref**：本交还摘要。

## 9. Story 清单与 Wave 规划

| Story | 标题 | 文件所有权（primary） | 依赖 | Wave | lld_policy |
|---|---|---|---|---|---|
| STORY-039-01 | G1 静态路由回滚 verify 兜底 | op_mapper.py（G1 区）、test_op_mapper_rollback.py | 无 | W1 | technical-note |
| STORY-039-02 | G4 next_hop_a 第 10 类占位符 | test_op_mapper_next_hop_a.py（新建）；op_mapper.py（G4 区，shared） | 01（op_mapper.py 串行） | W2 | technical-note |
| STORY-039-03 | G5 sw3 族回源 | case_runner.py、test_tg_op_mapping.py；op_mapper.py（G5 区，shared，merge_owner） | 02（op_mapper.py 串行） | W3 | technical-note |
| STORY-039-04 | 文档与追溯 | SKILL.md、op-coverage-matrix.md、CR-043/044/046、BACKLOG-CR-039、env-file 样例 | 无 | W1 | technical-note |

- **依赖关系**：STORY-039-01 → 02 → 03 为 `file-ownership` 串行（op_mapper.py 同文件三块写入，区域互不重叠）；STORY-039-04 无依赖，与 01 并行（W1，文件互斥）。
- **并行安全**：case_runner.py / test_tg_op_mapping.py 仅 03 独占；单测文件各 Story 独占；文档文件仅 04。无输出文件冲突（CP4 自动预检项）。
- **路径说明**：DEVELOPMENT-PLAN 按项目既有惯例写入 `process/DEVELOPMENT-PLAN-CR-039.yaml`（CR-033 之后的 CR-034/037/038 均用 CR 后缀），未覆盖 `process/DEVELOPMENT-PLAN.yaml`（该文件为 CR-033 的机器真相源，overwrite 会破坏既有记录）。

## 10. 风险

- RA-039-01：G1 verify 兜底在真实大路由表场景超时或匹配歧义 —— 缓解：next_hop_addr 参与匹配 + 单测覆盖多路由（STORY-039-01）。
- RA-039-02：sw3 telnet 依赖 `SW3_PASSWORD` 环境变量，真机验证前无法完整验证 —— 缓解：static-only + dry-run 计划断言（STORY-039-03）。
- RA-039-03：CR-047 post_delay 静默遗漏导致 case_runner 回源不完整 —— 缓解：DQ-039-05 显式决策 + STORY-039-03 精细排除 + STORY-039-04 BACKLOG 记录。
- RA-039-04：canonical 无 sw3 单测，回源后无机器验证 —— 缓解：STORY-039-03 补齐断言（44→45 + 六处映射 + dry-run envelope）。
- RA-039-05：op-coverage-matrix 结构限制旁路章节 —— 缓解：降级方案 B（仅代码/单测/SKILL.md 同步），CP2 已确认不改计数。
- RA-039-06：逐块合并时 canonical 领先导致反向覆盖 —— 缓解：canonical HEAD 为基底 + 工作区为 canonical 超集（b4719e8 已含），dict 内容级差异逐块核对。

## 11. 未决项

- 无 BLOCKING 未决项。CP3 人工门禁待 host-orchestrator 发起（HLD N/A + Story 拆解确认）。
- 后续 CP4 / CP5 按标准流程推进；CP5 前 meta-dev 为 4 Story 补齐 technical-note 设计证据。

## 12. 给 host-orchestrator 的 CP3 Decision Brief 输入（建议）

- **推荐决策**：approve（接受 HLD N/A + 4 Story 拆解 + 全部 technical-note 设计证据策略）。
- **不授权项**：approve 不隐含真机下发授权（trex 发流 / SSH 重启 trex-stl-te 为独立 runtime_authorization，DQ-039-04 不授权）；不授权凭据明文落盘（sw3 密码走 SW3_PASSWORD）。
- **approve 后发生**：host-orchestrator 消费 route_plan 推进 CP4 自动预检 → meta-dev 为 4 Story 补齐 technical-note → CP5 全量设计证据确认 → 按 Wave 调度实现（W1 并行 01+04 / W2=02 / W3=03）。
- **不确认阻塞**：Story 拆解保持草案，不进 CP4/CP5/实现。
- **审批者摘要与决策分层**：本确认服务 HLD N/A 豁免与 Story 边界确认；决策分层——必须用户决策 0 项（CP2 已全覆盖）、高风险策略确认 0 项、agent 默认处理 0 项（本阶段无新决策）、仅审计记录 HLD N/A + 4 Story 拆解。

## 13. 阶段产物路径清单

- `process/checks/CP3-CR039.result.json`（机器真相源）
- `process/checks/CP3-CR039-HLD-NA.md`（人类摘要）
- `process/DEVELOPMENT-PLAN-CR-039.yaml`（机器真相源：Story/Wave/状态/任务/依赖/文件所有权/security_constraints）
- `process/stories/STORY-039-01.md` / `STORY-039-02.md` / `STORY-039-03.md` / `STORY-039-04.md`
- `process/handoffs/solution-design-meta-se-RETURN-SUMMARY-CR039.md`（本交还摘要）
