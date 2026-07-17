---
story_id: "ST-RA-06.1-DETECT"
title: "变更检测、合并与版本历史（S2）"
status: "planned"
priority: "P0"
wave: 2
source_cr: "CR-030"
source_story: "ST-RA-06"
feature: "FEAT-RA-INGESTION"
feature_design_refs:
  - "docs/features/feat-ra-ingestion/DESIGN.md"
  - "docs/features/feat-ra-ingestion/TASKS.md"
lld_policy:
  required_level: "full-lld"
  trigger_reasons: ["data-model", "cross-module-contract"]
  rationale: "变更检测算法、冲突队列、ticket_version/change_history 写入"
depends_on:
  - "ST-NRA-03"
  - "ST-RA-INGEST-DB"
output_files:
  - "skills/itr-ticket-ingestion/SKILL.md"
created_by: "meta-se"
created_at: "2026-07-16"
---

# ST-RA-06.1-DETECT: 变更检测、合并与版本历史（S2）

## dev_context

### 背景
在 itr-ticket-ingestion Skill 中追加 S2 增量摄取能力。消费新拉取批次，与已保存版本比较，检测新增/修改/未变化记录，合并可靠数据，保留版本历史。

### 输入文件
- `docs/features/feat-ra-ingestion/DESIGN.md` §4.2
- `skills/itr-ticket-ingestion/SKILL.md`（Wave 1 完成版本）

### 输出文件
- `skills/itr-ticket-ingestion/SKILL.md`（追加变更检测和版本历史部分）

### 核心能力
1. 按 source_ticket_id 匹配已有 ticket
2. 字段级 diff：新增/修改/未变化/冲突
3. 冲突队列：无稳定 ID 或语义冲突 → manual_queue
4. UPSERT ticket + INSERT ticket_version + INSERT change_history

### AI 可执行任务清单
1. 实现新批次 vs 已有版本匹配
2. 实现字段级 diff 检测
3. 实现冲突队列处理（manual_queue）
4. 实现版本化 UPSERT（version 递增 + field_diffs）
5. 实现 change_history 写入

## validation_context
- 关键验证：新 ID → type='new'；已有 ID 无变化 → type='unchanged'；有变化 → type='modified' + field_diffs

## acceptance_criteria
1. 新增 ticket 正确标记 new
2. 已有 ticket 无变化标记 unchanged
3. 字段变更正确记录 field_diffs + version 递增
4. 无稳定 ID 进入 manual_queue
5. change_history 含 type/resolution
