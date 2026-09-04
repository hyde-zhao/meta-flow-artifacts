---
doc_type: handoff
id: HANDOFF-CR047-SD-META-SE-001
cr_id: CR-047
stage: solution-design
from_role: host-orchestrator
to_role: meta-se
mode: subagent
canonical_role: meta-se
dispatch_trigger: "CP2 approve 后自动推进（CP3 审计 + Story 拆解 + technical-note 设计证据）"
created_at: "2026-08-19T08:10:00+00:00"
context_policy:
  read_profile: compact
  must_read:
    - process/changes/CR-047.md
    - process/checkpoints/CP2-CR047.md
  read_if_needed:
    - process/handoffs/requirement-clarification-meta-pm-RETURN-SUMMARY-CR047.md
    - process/changes/CR-046.md
    - process/stories/STORY-039-03.md
  do_not_read_by_default:
    - process/archive/**
    - process/discussions/**
  capsule_ref: null
---

# 委托：CR-047 CP3 审计 + Story 拆解（solution-design / story-planning，轻量）

## 任务背景

CR-047（case_runner post_delay 独立回源，standard 模式）CP2 已 approve（DQ-047-01=A 补 SKILL.md 一行；DQ-047-02=A standard）。回源型 CR：单文件（case_runner.py）+ 单测 + SKILL.md 一行，无新架构、无新接口、无数据模型变更。

## 你（meta-se）的职责

1. **CP3 HLD 适用性审计（结论输出，非门禁发起）**：判定 HLD / BLUEPRINT / DOMAIN-MAP / DEPENDENCY-MAP 是否 N/A（预期：均 N/A，理由=回源型无新架构；参照 CR-046 retroactive 模式与 CR-039 CP3 先例）。输出审计结论供 host-orchestrator 写 CP3 result。
2. **Story 拆解**：单 Story `STORY-047-01`（case_runner post_delay 回源 + 单测 + 守护反转 + SKILL.md 一行）。写 Story 卡片 `process/stories/STORY-047-01.md`，含 technical-note 设计证据（CR-039 STORY 卡片同模式），必须覆盖：文件影响范围、接口/函数设计（`_parse_post_delay` 签名与解析规则、睡眠行叠加）、异常处理（None/空/非法/负数）、测试设计（≥8 用例清单 + 守护反转）、实施步骤、回滚策略、数据（无持久化）、权限（无边界变更）、tier/shared_fragments/open_items 声明。
3. **DEVELOPMENT-PLAN.yaml 条目**：在 `process/DEVELOPMENT-PLAN.yaml` 追加 CR-047 / STORY-047-01（单 Wave、无依赖、文件所有权 case_runner.py + tests + SKILL.md；注意与 CR-036 声明重叠但代码已落地，写明无冲突）。

## 约束

- 不修改 HLD.md / BLUEPRINT.md 等正式设计对象（N/A 判定只输出结论）。
- 不发起 CP3 / CP5 正式人工门禁（host-orchestrator 唯一发起者）。
- technical-note 是本 Story 的 lld_policy.required_level（低风险单 Story）。

## 输出

1. 交还摘要 `process/handoffs/solution-design-meta-se-RETURN-SUMMARY-CR047.md`：CP3 审计结论（各设计对象 N/A 判定 + 理由）、Story 拆解说明、CP5 Decision Brief 输入（预期决策项数，若为 0 需说明原因）。
2. `process/stories/STORY-047-01.md`（含 technical-note）。
3. `process/DEVELOPMENT-PLAN.yaml` 追加条目（保留既有内容，增量追加）。
