---
story_id: "ST-RA-05.1-INGEST"
title: "ITR 问题单受控摄取与原始快照保存"
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
  trigger_reasons: ["external-interface", "security", "data-model"]
  rationale: "外部 HTTP 接口、allowlist 安全策略、原始快照存储"
depends_on:
  - "ST-RA-INGEST-DB"
output_files:
  - "skills/itr-ticket-ingestion/SKILL.md"
  - "skills/itr-ticket-ingestion/templates/batch-manifest.yaml"
created_by: "meta-se"
created_at: "2026-07-16"
---

# ST-RA-05.1-INGEST: ITR 问题单受控摄取与原始快照保存

## dev_context

### 背景
创建 `itr-ticket-ingestion` Skill 的 HTTP 摄取部分。这是整个分析链的入口，必须严格遵守 deny-by-default 安全原则。

### 输入文件
- `docs/features/feat-ra-ingestion/DESIGN.md` §3.1、§4.1、§5
- `data/schema.sql` — 参考 ticket/batch 表结构
- `docs/design/HLD.md` REV-03 — 安全约束

### 输出文件
- `skills/itr-ticket-ingestion/SKILL.md`（HTTP 摄取 + allowlist + 快照保存部分）
- `skills/itr-ticket-ingestion/templates/batch-manifest.yaml`

### 接口约定
- HTTP GET only，allowlist URL 校验
- 参数白名单（product, time_range, pagination）
- 保存原始响应 JSON + request params + timestamp + hash 到非 Git 受限数据区
- 返回 snapshot_ref + batch 摘要

### 设计约束
- 无凭据、无认证头推断、无 ITR 写入
- 超时 30s（可配置）
- 不自动重试
- 原始快照不进 Git

### AI 可执行任务清单
1. 创建 `skills/itr-ticket-ingestion/` 目录和 SKILL.md 骨架
2. 实现 allowlist URL 校验逻辑
3. 实现 HTTP GET 请求（超时、状态码检查）
4. 实现原始快照保存（文件路径 + 元数据）
5. 实现 batch-manifest 模板
6. 实现基本测试（allowlist 拒绝、HTTP 方法限制）

## validation_context

### 验证入口
- fixture 测试：有效 URL → 200 OK + 快照保存
- security fixture：非白名单 URL → 拒绝；POST → 拒绝；含 Authorization → 拒绝

### 验证方式
- fixture + static analysis + manual review
- 无 runtime 依赖，使用 mock HTTP 响应

### 关键验证场景
1. allowlist URL 通过，非白名单拒绝
2. POST/PUT/DELETE 方法拒绝
3. 含 Authorization header 的请求拒绝
4. 原始快照正确保存到非 Git 路径
5. batch-manifest 包含 request/response 完整元数据

## acceptance_criteria

1. `skills/itr-ticket-ingestion/SKILL.md` 包含 HTTP ingestion 部分
2. allowlist URL 校验：匹配通过，非匹配拒绝并记录
3. 仅允许 GET 方法，其他方法拒绝
4. 原始快照保存到非 Git 路径（`.gitignore` 排除）
5. batch-manifest 模板包含 request_url、params、timestamp、hash、snapshot_ref
6. 无凭据读取或 ITR 写入路径
