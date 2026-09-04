---
handoff_id: "CR-033-SOL-DESIGN-meta-se"
cr_id: "CR-033"
phase: "solution-design"
canonical_role: "meta-se"
delegated_interaction: true
agent_role: "meta-se"
status: "completed"
created_at: "2026-07-28T10:20:00+00:00"
owner: "host-orchestrator"
context_ref: "process/changes/CR-033.md"
cp2_checkpoint_ref: "process/checkpoints/CP2-CR033-REQUIREMENT-BASELINE.md"
---

# Handoff: CR-033 solution-design -> meta-se

## 委托背景

CR-033（统一目标包 parent CR）CP2 v1.1 已 approved，需求/场景/范围基线确认（26 功能需求 / 25 场景 / 16 Story 候选 / 6 DQ / 5 SGQ，ready_for_design=true）。按 Meta Flow 流程，solution-design 阶段委托 meta-se 产出蓝图、HLD、Story 拆解，准备 CP3。

**关键前提**：技术选型与范围决策已由用户在 CP2 确认，meta-se **不重新做技术选型决策**，基于已确认范围产出设计产物并补充 Architecture Gray Areas。

## 必读上下文

- `process/changes/CR-033.md`：完整范围目标 + 决策定稿 + 12 条改进分期
- `process/checkpoints/CP2-CR033-REQUIREMENT-BASELINE.md`：CP2 Decision Brief（6 DQ）
- `process/handoffs/CR-033-REQ-CLAR-meta-pm-RETURN-SUMMARY.md`：需求澄清交还摘要（v1.1）
- `docs/product/REQUIREMENTS-PTM-TE-EXEC.md`：26 功能需求
- `docs/product/SCENARIOS-PTM-TE-EXEC.yaml`：25 验证场景
- `docs/product/STORY-MAP-PTM-TE-EXEC.md`：16 Story 候选
- `docs/product/MVP-SCOPE-PTM-TE-EXEC.md`：MVP 范围
- 现有实现参考：`skills/policy-route-execution/scripts/op_mapper.py`、`skills/device-management/`、`skills/trex-traffic/`、`/home/hyde/projects/ptm-te/exec_v4.py`

## 任务清单

1. **蓝图适用性判定**：判定 CR-033 是否需要 BLUEPRINT（跨 Feature/Epic、数据归属、依赖方向）。
2. **Architecture Gray Areas**（CP3 前必须）：识别 3-4 个影响架构的灰区，advisor lane 用 `Option | Pros | Cons | Impact Surface | Recommendation | Assumptions/When to switch` 表格，至少 1 条用户确认，写入 `process/discussions/CP3-HLD-DISCUSSION-LOG-CR033.md`，恢复点 `process/checks/CP3-DISCUSSION-CHECKPOINT-CR033.json`。候选灰区：
   - case-execution skill 与 op_mapper 职责边界（编排层 vs 映射层，复用 execute 子命令的契约）
   - devices.yaml tg 块与 traffic-skill 拓扑的数据归属（TG 地址/接口数据源）
   - frontmatter 16 列与 case_steps YAML 字段映射（冗余字段处理，如测试步骤/预期结果）
   - envelope 统一解析 extract_payload 的抽象位置（case_runner 内 vs op_mapper 内）
   - fw_logout op 实现方式（op_mapper 映射 + ptm-atomic 子命令 vs 降级 session 文件清理）
   - 跨仓库 install.py 安装 case-execution skill 到 ptm-te 的机制
3. 产出 `docs/design/BLUEPRINT-PTM-TE-EXEC.md`（Feature/Epic 能力边界与数据归属）。
4. 产出 `docs/design/DOMAIN-MAP-PTM-TE-EXEC.md`（领域对象、状态、规则、术语）。
5. 产出 `docs/design/DEPENDENCY-MAP-PTM-TE-EXEC.md`（Feature/模块依赖方向与禁止依赖）。
6. 产出 `docs/design/HLD-PTM-TE-EXEC.md`（HLD，含候选方案对比、适用性矩阵、Use Case -> Architecture Traceability、场景模拟、技术选型理由、Gotchas）。
7. 产出 `docs/design/ARCHITECTURE-DECISION-PTM-TE-EXEC.md`（ADR）。
8. 产出 `docs/design/FEATURE-DESIGN-MATRIX-PTM-TE-EXEC.md`（Feature 适用性 + Story `feature_design_refs` + `lld_policy`）。
9. Story 拆解：基于 STORY-MAP 16 候选 + FEATURE-DESIGN-MATRIX，拆解为 Story（含文件布局 + TASK-ID），写入 `process/DEVELOPMENT-PLAN-CR-033.yaml`。
10. 产出 CP3 Decision Brief 输入（架构灰区处理、候选方案、ADR、风险），供 host-orchestrator 发起 CP3。
11. 写 CP3 自动检查输入草案。

## 已确认决策（不可推翻）

| 决策点 | 结论 |
|---|---|
| 引擎形态 | `skills/case-execution/`（SKILL.md + `scripts/case_runner.py` 兼 argparse CLI，op_mapper 模式） |
| 不引入框架 | 借鉴 pytest/robot 理念（用例发现/分级/fixture/报告），不引入 pytest/robot |
| TG 建模 | `type:TG` + `api_server`，纳入清单不纳入快照，不引入 pydantic |
| 执行入口 | 单用例 / 按目录 / 按标签或关键字（--tag/--keyword） |
| 用例结构 | cases/功能/子模块/用例组/ + frontmatter 16 列 + case_steps YAML + 命名<编号>-<名称>.md |
| #1 落点 | op_mapper._build_exec_env 对 tg_* op 注入 TREX_API_URL（不改 ptm-atomic 本体） |
| #4 | op_mapper 新增 fw_logout op + case_runner cleanup 登出（DQ-02 降级方案） |
| 6 DQ | DQ-01 static review / DQ-02 fw_logout 降级 / DQ-03 known_issue 同步 / DQ-04 24用例一次性迁移 / DQ-05 8必填+8可选 / DQ-06 编号正则匹配 |

## 约束

- `scenario_subject_type=target-artifact`，`engagement_mode=production`
- 跨仓库：exec_v4.py + 24 用例 md 在 `/home/hyde/projects/ptm-te/`（workspace）
- 设计产物用 `-PTM-TE-EXEC` 后缀（与产品文档一致，不混合 ptm-tse）
- 不修改 STATE.current.json，不发起 CP3 正式门禁（host-orchestrator 职责）
- HLD 必须含 Gotchas 章节（CLAUDE.md 设计评审规则 9）
- ADR 必须回写到 HLD/模块表/流程图（规则 12）

## dispatch

| 字段 | 值 |
|---|---|
| mode | subagent |
| canonical_role | meta-se |
| reasoning_profile | high |
| dispatch_trigger | CP2-approved-auto-advance |
| tool_name | Agent |
| spawned_at | 2026-07-28T10:20:00+00:00 |
| agent_id | ac895b9288e812ad5 |
| completed_at | 2026-07-28T10:40:00+00:00 |
