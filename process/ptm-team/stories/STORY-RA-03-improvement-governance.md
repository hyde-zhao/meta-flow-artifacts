---
story_id: "ST-RA-03"
title: "改进输入治理（improvement-tracker CA/PA 侧）"
status: "lld-ready-for-review"
priority: "P0"
wave: 3
source_cr: "CR-030"
source_story: "ST-RA-03"
feature: "FEAT-RA-IMPROVEMENT"
feature_design_refs:
  - "docs/features/feat-ra-improvement/DESIGN.md"
  - "docs/features/feat-ra-improvement/TASKS.md"
lld_policy:
  required_level: "full-lld"
  trigger_reasons: ["cross-module-contract", "security", "permission"]
  rationale: "CA/PA 批准门、Approved Improvement Input 契约"
depends_on:
  - "ST-RA-05.3-ANALYZE"
output_files:
  - "skills/improvement-tracker/SKILL.md"
  - "skills/improvement-tracker/templates/capa-proposal.yaml"
  - "skills/improvement-tracker/templates/approved-input.yaml"
created_by: "meta-se"
created_at: "2026-07-16"
---

# ST-RA-03: 改进输入治理

## dev_context

### 背景
创建 improvement-tracker Skill 的 CA/PA 侧。实现从已确认分析结论生成 CA/PA 草案、批准门和 Approved Improvement Input 产出。

### 输入文件
- `docs/features/feat-ra-improvement/DESIGN.md` §2、§3
- `docs/design/HLD.md` REV-03

### 输出文件
- `skills/improvement-tracker/SKILL.md`（CA/PA 侧）
- `skills/improvement-tracker/templates/capa-proposal.yaml`
- `skills/improvement-tracker/templates/approved-input.yaml`

### 核心能力
1. CA/PA Proposal 草案生成（依据+类型+目标+Owner+验收+副作用）
2. 批准状态机：draft→approved/rejected
3. 未批准门控：未确认 RA → 拒绝生成 CA/PA
4. Approved Improvement Input 生成（source_ra+proposal_id+target_agent+constraints+approval_ref）
5. 消费者映射（target_agent → consumer_status）

### 设计约束
- 只有人工 reviewer 能改变 approval_status
- 未批准 CA/PA 不生成 Approved Input
- Approved Input 生成后不可变

### AI 可执行任务清单
1. 创建 `skills/improvement-tracker/` 目录和 SKILL.md
2. 实现 CA/PA Proposal 草案生成
3. 实现批准状态机
4. 实现 Approved Input 生成
5. 实现消费者映射
6. 创建模板文件

## validation_context
- 关键验证：未确认 RA → 拒绝 CA/PA；未批准 CA/PA → 不生成 Input；批准后 Input 不可变

## acceptance_criteria
1. CA/PA 草案含依据/类型/目标/Owner/验收
2. 未确认 RA 不能生成 CA/PA
3. reviewer 批准/拒绝 CA/PA
4. 批准后生成 Approved Improvement Input（含 source_ra/proposal_id/approval_ref）
5. 批准前不分发；Input 生成后不可变
