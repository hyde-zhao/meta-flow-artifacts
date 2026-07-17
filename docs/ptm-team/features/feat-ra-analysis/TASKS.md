---
status: draft
version: "1.0"
source_cr: "CR-030"
feature_id: "FEAT-RA-ANALYSIS"
created_by: "meta-se"
created_at: "2026-07-16"
---

# FEAT-RA-ANALYSIS — 任务清单

## 交付物

| 文件 | Story |
|---|---|
| `skills/reverse-analysis/SKILL.md` | ST-RA-01, 02, 05.3, 06.2 |
| `skills/reverse-analysis/templates/ra-report.yaml` | ST-RA-02 |
| `skills/reverse-analysis/templates/analysis-run-manifest.yaml` | ST-RA-05.3 |
| `skills/reverse-analysis/templates/metric-definition.yaml` | ST-RA-02 |
| `agents/ptm-tse.md`（更新） | ST-RA-01 |

## Story 级任务

### ST-RA-01: 资格检查与可信输入建立

**TASK-ANL-01**: 创建 `skills/reverse-analysis/` 目录和 SKILL.md 骨架
**TASK-ANL-02**: 实现 P1/P2/P3/P4 资格判定逻辑
**TASK-ANL-03**: 实现内部问题识别与拒绝
**TASK-ANL-04**: 实现五条证据线分类（事实/假设/未知项）
**TASK-ANL-05**: 实现证据阈值检查（≥3 条有效线才可进入 confirmed）
**TASK-ANL-06**: 更新 `agents/ptm-tse.md` 添加 reverse-analysis Skill 引用

### ST-RA-02: 六维分析引擎

**TASK-ANL-07**: 实现根因四层状态机（raw→candidate→evidence-backed→confirmed）
**TASK-ANL-08**: 实现产品质量维度分析（数量/占比/Pareto/趋势 + 降级逻辑）
**TASK-ANL-09**: 实现流出控制逃逸矩阵（candidate/confirmed escape layer 分离）
**TASK-ANL-10**: 实现漏测维度 PPDCS 归类
**TASK-ANL-11**: 实现改进维度 CA/PA 候选草案生成
**TASK-ANL-12**: 实现环比/同比维度（窗口/基数/变化率/N-A/可信度）
**TASK-ANL-13**: 实现 MetricDefinition 模板与管理
**TASK-ANL-14**: 创建 ra-report 和 metric-definition 模板

### ST-RA-05.3-ANALYZE: S1 逐单与批量分析管线

**TASK-ANL-15**: 实现 SQLite 公共 DAO 查询适配（读取 ticket/ticket_version/ingestion_batch）
**TASK-ANL-16**: 实现 AnalysisRunManifest 生成与管理
**TASK-ANL-17**: 实现逐单分析管线（单 ticket → 六维分析 → 报告摘要）
**TASK-ANL-18**: 实现批量分析管线（多 ticket → 聚合趋势 + 模式识别）
**TASK-ANL-19**: 通过公共 DAO 写入 analysis_run 草案；发布必须走 reviewer 专用接口
**TASK-ANL-20**: 创建 analysis-run-manifest 模板

### ST-RA-06.2-REFRESH: S2 增量重算与差异报告

**TASK-ANL-21**: 实现受影响的维度检测（变更字段 → 分析维度映射）
**TASK-ANL-22**: 实现增量重算引擎（仅重算受影响维度）
**TASK-ANL-23**: 实现全量重算触发条件检测（规则/口径版本变化）
**TASK-ANL-24**: 实现差异报告生成（前后对比 + 变化量 + 变化率）
**TASK-ANL-25**: 实现环比/同比 N/A 条件判断

### ST-NRA-01: 证据不足保护

**TASK-NRA01-01**: 实现三线阈值硬阻断（<3 条有效线 → 禁止 confirmed）
**TASK-NRA01-02**: 实现缺口清单输出（缺失 Owner + 待澄清状态）

### ST-NRA-02: 权限边界拒绝

**TASK-NRA02-01**: 实现凭据/认证头请求拒绝
**TASK-NRA02-02**: 实现外部系统访问请求拒绝
**TASK-NRA02-03**: 实现生产操作请求拒绝
**TASK-NRA02-04**: 输出可审计的拒绝记录 + 替代路径 + 独立 CR 建议
