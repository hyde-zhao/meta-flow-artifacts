---
story_id: "ST-RA-05.3-ANALYZE"
title: "S1 逐单与批量分析管线"
status: "in-development"
priority: "P0"
wave: 2
source_cr: "CR-030"
source_story: "ST-RA-05"
feature: "FEAT-RA-ANALYSIS"
feature_design_refs:
  - "docs/features/feat-ra-analysis/DESIGN.md"
  - "docs/features/feat-ra-analysis/TASKS.md"
lld_policy:
  required_level: "full-lld"
  trigger_reasons: ["cross-module-contract", "data-model"]
  rationale: "SQLite 读取契约、analysis_run 管理、报告模板"
depends_on:
  - "ST-RA-02"
  - "ST-RA-05.2-CLEAN"
output_files:
  - "skills/reverse-analysis/SKILL.md"
  - "skills/reverse-analysis/templates/analysis-run-manifest.yaml"
created_by: "meta-se"
created_at: "2026-07-16"
---

# ST-RA-05.3-ANALYZE: S1 逐单与批量分析管线

## dev_context

### 背景
实现 S1 分析管线：从 SQLite 读取清洗后数据 → 执行六维分析 → 生成分析报告和 AnalysisRunManifest。

### 输出文件
- `skills/reverse-analysis/SKILL.md`（追加 S1 管线部分）
- `skills/reverse-analysis/templates/analysis-run-manifest.yaml`

### 核心能力
1. SQLite 公共 DAO 查询（ticket/ticket_version/ingestion_batch）+ analysis_run 草案写入
2. 逐单分析：单 ticket → 六维 → 报告摘要
3. 批量分析：多 ticket → 聚合趋势 + 模式识别
4. AnalysisRunManifest: batch_ref, schema_version, window, recompute_mode, report_refs

### AI 可执行任务清单
1. 实现 SQLite 公共 DAO 查询适配与 analysis_run 草案写入
2. 实现逐单分析管线
3. 实现批量分析管线（聚合 + 模式）
4. 实现 AnalysisRunManifest 生成
5. 实现 analysis_run 写入 SQLite
6. 创建 manifest 模板

## validation_context
- 关键验证：逐单报告含 facts/hypotheses/gaps；批量报告含趋势+模式

## acceptance_criteria
1. 从 SQLite 正确读取 cleaned 数据
2. 逐单分析输出 facts/hypotheses/gaps
3. 批量分析输出聚合趋势+模式识别
4. AnalysisRunManifest 含 batch/version/窗口/重算模式/报告引用
5. analysis_run 正确写入 SQLite
