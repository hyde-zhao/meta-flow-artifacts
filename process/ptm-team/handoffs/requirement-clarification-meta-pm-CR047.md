---
doc_type: handoff
id: HANDOFF-CR047-RC-META-PM-001
cr_id: CR-047
stage: requirement-clarification
from_role: host-orchestrator
to_role: meta-pm
mode: subagent
canonical_role: meta-pm
dispatch_trigger: "CR-047 启动后增量澄清（CP1 + CP2 输入）"
created_at: "2026-08-19T07:30:00+00:00"
context_policy:
  read_profile: compact
  must_read:
    - process/changes/CR-047.md
    - process/BACKLOG-CR-039.md
  read_if_needed:
    - process/changes/CR-039-FOLLOW-UP-TRACKING-2026-08-19.md
    - process/changes/CR-039.md
    - docs/product/REQUIREMENTS.md
    - docs/product/USE-CASES.md
    - docs/product/SCENARIOS.yaml
    - docs/product/TEST-MATRIX.md
    - docs/product/MVP-SCOPE.md
  do_not_read_by_default:
    - process/archive/**
    - process/discussions/**
  capsule_ref: null
---

# 委托：CR-047 增量需求澄清（requirement-clarification，阶段零 + CP1/CP2 输入）

## 任务背景

CR-047（case_runner post_delay 独立回源）已由用户启动（候选 CR-039-FU-047 转 formal，冲突预检通过、并行推进）。本 CR 是 **ptm-te-manaul 工作区已验证改动的 canonical 回源**（同 CR-039 模式）：`_parse_post_delay` 函数 + execute_steps 睡眠行叠加，用途为断开 PPPoE 后等待会话 tear down 再发流。

## 你（meta-pm）的职责（被委托期间直接结论输出，不需直连用户）

1. **阶段零快速调研**：核对 CR-047.md 声明与 BACKLOG-CR-039 BL-039-01、台账 CR-039-FU-047 是否一致（锚点、排除边界、static-only 口径）。
2. **增量需求结构化**：判断 CR-047 对 `docs/product/REQUIREMENTS.md` / `USE-CASES.md` / `SCENARIOS.yaml` / `TEST-MATRIX.md` / `MVP-SCOPE.md` 是否需要增量更新（回源型 CR 优先最小增量；注意需求/场景变更需 CR 记录文档处理决策，默认增量更新保留旧基线+修订记录）。
3. **CP2 Decision Brief 输入挖掘**：列出需要用户决策的决策项（DQ-047-*）。已知候选：SKILL.md 是否补 post_delay step 级参数说明（量化验收第 4 条）；workflow_mode 判定（fast-lane vs standard）。低风险可回退实现细节默认归 agent 默认处理，不得升级为用户决策。
4. **范围基线草案**：输出 CP2 范围基线要点（in-scope / out-of-scope / 排除项），供 host-orchestrator 汇总成 CP2 Decision Brief。

## 约束

- 不修改 REQUIREMENTS.md / USE-CASES.md 等正式对象（只输出建议；实际修改由后续阶段按 CR 决策执行）。
- 不发起 CP2 正式人工检查点（host-orchestrator 唯一发起者）。
- 发现灰区问题写入 return summary 的 `open_questions`，由 host-orchestrator 汇总。

## 输出

写入 `process/handoffs/requirement-clarification-meta-pm-RETURN-SUMMARY-CR047.md`，包含：调研结论、文档增量判定（逐文档 新增/更新/N/A）、CP2 Decision Brief 输入（决策项列表含推荐+备选）、范围基线草案、open_questions。
