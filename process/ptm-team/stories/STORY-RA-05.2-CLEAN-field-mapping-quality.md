---
story_id: "ST-RA-05.2-CLEAN"
title: "字段映射、清洗与质量报告"
status: "planned"
priority: "P0"
wave: 1
source_cr: "CR-030"
source_story: "ST-RA-05"
feature: "FEAT-RA-INGESTION"
feature_design_refs:
  - "docs/features/feat-ra-ingestion/DESIGN.md"
  - "docs/features/feat-ra-ingestion/TASKS.md"
lld_policy:
  required_level: "full-lld"
  trigger_reasons: ["data-model", "cross-module-contract"]
  rationale: "IngestionQualityReport 是分析的前置契约，字段映射和清洗规则需完整 LLD"
depends_on:
  - "ST-RA-05.1-INGEST"
output_files:
  - "skills/itr-ticket-ingestion/SKILL.md"
  - "skills/itr-ticket-ingestion/templates/quality-report.yaml"
created_by: "meta-se"
created_at: "2026-07-16"
---

# ST-RA-05.2-CLEAN: 字段映射、清洗与质量报告

## dev_context

### 背景
在 itr-ticket-ingestion Skill 中追加清洗和质量报告能力。这是连接原始摄取和可信分析的桥梁，IngestionQualityReport 是 analysis_run 的前置条件。

### 输入文件
- `docs/features/feat-ra-ingestion/DESIGN.md` §3.3、§4.1
- `skills/itr-ticket-ingestion/SKILL.md`（ST-RA-05.1 已创建的摄取部分）
- `data/schema.sql` — ticket 表字段定义

### 输出文件
- `skills/itr-ticket-ingestion/SKILL.md`（追加清洗和质量报告部分）
- `skills/itr-ticket-ingestion/templates/quality-report.yaml`

### 接口约定
- 字段映射：ITR 原始字段 → ticket schema 字段
- 清洗规则：空值/异常检测、quality_flag 标记
- 未知字段保留原始引用（不静默丢弃）
- IngestionQualityReport: batch_id, quality_summary, overall_status, analyzable_ratio

### AI 可执行任务清单
1. 实现 ITR → ticket 字段映射表
2. 实现空值/异常值/重复检测
3. 实现未知字段保留逻辑
4. 实现 IngestionQualityReport 生成
5. 实现质量阈值判断（analyzable_ratio < threshold → blocked）
6. 实现敏感字段分类检查
7. 创建 quality-report 模板

## validation_context

### 验证入口
- fixture 测试：有效 JSON → cleaned records + quality report

### 关键验证场景
1. 正常数据清洗通过，quality_flag='clean'
2. 含空值/异常 → quality_flag 标记，不丢弃记录
3. 未知字段保留原始引用
4. analyzable_ratio < 阈值 → overall_status='blocked'
5. 未分类敏感字段不进 LLM/正文

## acceptance_criteria

1. ITR 字段正确映射到 ticket schema 字段
2. quality_flag 正确标记（clean/incomplete/anomaly/blocked）
3. IngestionQualityReport 包含所有必需字段
4. 质量不达标时 overall_status='blocked'
5. 未知字段保留原始引用不丢失
