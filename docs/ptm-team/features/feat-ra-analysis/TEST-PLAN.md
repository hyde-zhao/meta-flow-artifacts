---
status: draft
version: "1.0"
source_cr: "CR-030"
feature_id: "FEAT-RA-ANALYSIS"
created_by: "meta-se"
created_at: "2026-07-16"
---

# FEAT-RA-ANALYSIS — 测试计划

## 1. 测试场景

| ID | 场景 | Story | 类型 | 预期 |
|---|---|---|---|---|
| T-ANL-01 | P1 事件触发资格通过 | ST-RA-01 | positive | 资格标记为 eligible |
| T-ANL-02 | P2 事件显式选择触发 | ST-RA-01 | positive | 用户可显式选择启动 |
| T-ANL-03 | P3/P4 不自动进入 | ST-RA-01 | boundary | 拒绝或仅人工可启动 |
| T-ANL-04 | 内部问题拒绝 | ST-RA-01 | precheck | 标记 deferred + 重新立项条件 |
| T-ANL-05 | 证据不足（<3 条有效线） | ST-NRA-01 | negative | 根因状态不能为 confirmed |
| T-ANL-06 | 五条证据线分类（事实/假设/未知项） | ST-RA-01 | positive | 每条线有 validity + gap/fact/hypothesis 标注 |
| T-ANL-07 | 根因四层状态机不可自动跃迁 | ST-RA-02 | boundary | fixture 证明无自动跃迁路径 |
| T-ANL-08 | 六维分析正常输出 | ST-RA-02 | positive | 六个维度均产生输出 |
| T-ANL-09 | 无分母时不称密度 | ST-RA-02 | boundary | metric degraded_output 正确降级 |
| T-ANL-10 | 无控制证据只输出 candidate escape | ST-RA-02 | boundary | escape layer 标记为 candidate |
| T-ANL-11 | AnalysisRunManifest 完整性 | ST-RA-05.3 | positive | batch/version/窗口/重算模式/报告引用齐全 |
| T-ANL-12 | S1 逐单分析输出 | ST-RA-05.3 | positive | 每 ticket 产生 facts/hypotheses/gaps |
| T-ANL-13 | S1 批量分析趋势 | ST-RA-05.3 | positive | 批量趋势 + 模式识别 |
| T-ANL-14 | deny-by-default 拒绝外部访问 | ST-NRA-02 | security | 凭据/外部读取请求拒绝 |
| T-ANL-15 | deny-by-default 拒绝生产写入 | ST-NRA-02 | security | 生产修复请求拒绝 |
| T-ANL-16 | S2 增量重算（受影响维度） | ST-RA-06.2 | positive | 仅变更相关维度重算 |
| T-ANL-17 | S2 环比同比计算 | ST-RA-06.2 | positive | 窗口/基数/变化率/可信度 |
| T-ANL-18 | S2 窗口不足 N/A | ST-RA-06.2 | boundary | 标记 N/A + 原因 |
| T-ANL-19 | S2 规则变更全量重算 | ST-RA-06.2 | boundary | 映射/分析规则版本变化触发全量 |
| T-ANL-20 | 敏感字段不进 LLM/报告 | ST-RA-02 | security | 未分类字段阻断/脱敏 |

## 2. Fixture 设计

| Fixture | 用途 |
|---|---|
| `fixtures/eligible_p1_ticket.json` | T-ANL-01 |
| `fixtures/insufficient_evidence_ticket.json` | T-ANL-05 |
| `fixtures/six_dim_full_data.json` | T-ANL-08, 09, 10 |
| `fixtures/forbidden_request.json` | T-ANL-14, 15 |
| `fixtures/s2_before_after_datasets.json` | T-ANL-16, 17, 18 |
| `fixtures/sensitive_field_ticket.json` | T-ANL-20 |
