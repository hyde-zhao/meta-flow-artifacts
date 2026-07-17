---
story_id: "ST-RA-02"
title: "六维分析引擎（reverse-analysis Skill 核心）"
status: "ready-for-verification"
priority: "P0"
wave: 2
source_cr: "CR-030"
source_story: "ST-RA-02"
feature: "FEAT-RA-ANALYSIS"
feature_design_refs:
  - "docs/features/feat-ra-analysis/DESIGN.md"
  - "docs/features/feat-ra-analysis/TASKS.md"
lld_policy:
  required_level: "full-lld"
  trigger_reasons: ["data-model", "cross-module-contract"]
  rationale: "六维分析核心引擎、根因四层状态机、MetricDefinition 契约"
depends_on:
  - "ST-RA-01"
output_files:
  - "skills/reverse-analysis/SKILL.md"
  - "skills/reverse-analysis/templates/ra-report.yaml"
  - "skills/reverse-analysis/templates/metric-definition.yaml"
created_by: "meta-se"
created_at: "2026-07-16"
---

# ST-RA-02: 六维分析引擎

## dev_context

### 背景
在 reverse-analysis Skill 中实现核心六维分析引擎。每个维度有独立的分析方法和输出要求。

### 输入文件
- `docs/features/feat-ra-analysis/DESIGN.md` §3（六维分析引擎）
- `skills/reverse-analysis/SKILL.md`（ST-RA-01 已创建）

### 输出文件
- `skills/reverse-analysis/SKILL.md`（追加六维引擎部分）
- `skills/reverse-analysis/templates/ra-report.yaml`
- `skills/reverse-analysis/templates/metric-definition.yaml`

### 六维要求
| 维度 | 关键能力 |
|---|---|
| 根因 | 四层状态机（raw→candidate→evidence-backed→confirmed），禁止自动跃迁 |
| 产品质量 | 数量/占比/Pareto/趋势；有分母时才启用缺陷密度 |
| 流出 | candidate/confirmed escape layer 分离；无证据只输出 candidate |
| 漏测 | PPDCS 归类；缺失字段标记 unknown |
| 改进 | CA/PA 候选草案（依据+类型+目标+Owner+验收） |
| 环比同比 | 窗口定义+基数+变化率+N/A/可信度 |

### AI 可执行任务清单
1. 实现根因四层状态机
2. 实现产品质量分析（含无分母降级）
3. 实现流出矩阵（candidate/confirmed 分离）
4. 实现漏测 PPDCS 归类
5. 实现改进 CA/PA 候选
6. 实现环比同比（窗口/N-A/可信度）
7. 实现 MetricDefinition 模板
8. 创建 ra-report 和 metric-definition 模板

## validation_context
- 关键验证：根因四层不可自动跃迁、无分母不称密度、无控制证据只输出 candidate

## acceptance_criteria
1. 六维各维度正确输出
2. 根因状态机不可自动从 AI candidate 跃迁至 evidence-backed
3. 无分母时输出数量/占比/Pareto/趋势，不称密度
4. 无控制证据时 escape layer 标记 candidate
5. MetricDefinition 含分子/分母/窗口/NA条件
