---
status: draft
version: "1.0"
source_cr: "CR-030"
feature_id: "FEAT-RA-INGESTION"
created_by: "meta-se"
created_at: "2026-07-16"
---

# FEAT-RA-INGESTION — 测试计划

## 1. 测试策略

| 维度 | 策略 |
|---|---|
| 测试层级 | fixture + static analysis + manual review |
| 验证模式 | static-only（无 runtime 依赖） |
| 安全测试 | 优先：deny-by-default（凭据/写入/外部访问拒绝） |
| 数据完整性 | fixture 驱动：schema 校验、版本化、失败回滚 |

## 2. 测试场景

| ID | 场景 | Story | 类型 | 输入 | 预期 |
|---|---|---|---|---|---|
| T-ING-01 | HTTP GET 成功拉取 + 保存 | ST-RA-05.1 | positive | 有效 URL + 参数 | 200 OK，snapshot 保存，batch 记录 |
| T-ING-02 | 非 allowlist URL 拒绝 | ST-RA-05.1 | security | 非白名单 URL | 拒绝，错误记录 |
| T-ING-03 | HTTP 方法限制（POST/PUT 拒绝） | ST-RA-05.1 | security | POST 请求 | 拒绝，无写入 |
| T-ING-04 | 凭据/认证头拒绝 | ST-RA-05.1 | security | 含 Authorization header | 拒绝，不读取 |
| T-ING-05 | 字段映射 + 清洗正常路径 | ST-RA-05.2 | positive | 有效 JSON 响应 | cleaned records，quality_flag='clean' |
| T-ING-06 | 空值/异常值检测 | ST-RA-05.2 | negative | 含空值/异常 JSON | quality_flag 标记，不丢弃记录 |
| T-ING-07 | 未知字段保留 | ST-RA-05.2 | boundary | 含未知字段 JSON | 保留原始引用，标注 unclassified |
| T-ING-08 | IngestionQualityReport 生成 | ST-RA-05.2 | positive | 清洗结果 | 完整 quality report |
| T-ING-09 | SQLite schema 创建 | ST-RA-INGEST-DB | positive | DDL | 所有表、索引、约束正确 |
| T-ING-10 | ticket upsert（首次 + 更新） | ST-RA-INGEST-DB | positive | 新/已有 source_ticket_id | INSERT 或 UPSERT 正确 |
| T-ING-11 | ticket_version 递增 | ST-RA-INGEST-DB | positive | 更新已有 ticket | 新 version 号，field_diffs 正确 |
| T-ING-12 | source_ticket_id 唯一性 | ST-RA-INGEST-DB | boundary | 重复 ID | UNIQUE 约束生效 |
| T-ING-13 | HTTP 失败保护 | ST-NRA-03 | negative | 非 2xx 响应 | 不覆盖历史，错误记录 |
| T-ING-14 | Schema 不匹配保护 | ST-NRA-03 | negative | 不匹配 JSON | 批次 blocked，已有数据安全 |
| T-ING-15 | SQLite 写入失败回滚 | ST-NRA-03 | negative | 模拟写入异常 | 事务回滚，数据不变 |
| T-ING-16 | 质量不达标阻断 | ST-RA-05.2 | boundary | analyzable_ratio < 阈值 | overall_status='blocked' |
| T-ING-17 | 敏感字段不进 LLM/正文 | ST-RA-05.2 | security | 未分类字段 | 阻断或脱敏 |
| T-ING-18 | 原始数据不进 Git | ST-RA-05.1 | boundary | snapshot 路径 | 在 .gitignore 排除范围内 |

## 3. Fixture 设计

| Fixture | 内容 | 用途 |
|---|---|---|
| `fixtures/itr_valid_response.json` | 有效 ITR JSON 响应 | T-ING-01, 05, 06, 07, 08 |
| `fixtures/itr_malformed_response.json` | 格式错误 JSON | T-ING-14 |
| `fixtures/itr_partial_response.json` | 含空值/缺失字段 | T-ING-06 |
| `fixtures/itr_unknown_fields.json` | 含未知字段 | T-ING-07 |
| `fixtures/allowlist_config.yaml` | allowlist 配置 | T-ING-02 |
| `fixtures/forbidden_request_fixture.json` | 含凭据/写入请求 | T-ING-03, 04 |
| `fixtures/expected_schema.sql` | 预期 SQLite DDL | T-ING-09 |

## 4. 覆盖缺口

| Gap | 原因 | 处理 |
|---|---|---|
| 真实 ITR 端点测试 | 无 runtime 授权 | dry-run fixture 模拟 |
| 大规模数据性能 | 首版 MVP 不要求 | 延后至性能 CR |
| 真实敏感数据 | 无生产数据授权 | 使用脱敏 fixture |
