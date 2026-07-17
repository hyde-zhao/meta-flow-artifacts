---
story_id: "ST-RA-05.1-INGEST"
canonical_story_id: "ST-RA-05.1-INGEST"
title: "ITR 问题单受控摄取与原始快照保存"
source_cr: "CR-030"
feature: "FEAT-RA-INGESTION"
implemented_by: "meta-dev"
implemented_at: "2026-07-16T14:00:00+00:00"
stage: "CP6"
shared_skill: "skills/itr-ticket-ingestion/SKILL.md"
---

# ST-RA-05.1-INGEST 实现执行证据

## 实现前置检查

| 检查项 | 结论 |
|---|---|
| Story 状态 | CP5 已审批通过（`CP5-CR030-DESIGN-EVIDENCE-BATCH.md`，Round 4，`status=approved`） |
| 前置 Story ST-RA-INGEST-DB | 已完成：`data/dao.py`（34 DAO）、`data/schema.sql`（6 表 + 13 索引）、`data/.gitignore` |
| 文件所有权 | `skills/itr-ticket-ingestion/` 为新建目录，F-020 独占，无冲突 |
| LLD 完备性 | `STORY-RA-05.1-INGEST-LLD.md` v1.2，14 章节，open_items 为空 |
| Feature 设计完备性 | `feat-ra-ingestion/DESIGN.md` v1.1，已确认 |
| 设计契约一致性 | LLD、Feature 设计、HLD REV-03 无矛盾 |
| 输出路径明确 | `skills/itr-ticket-ingestion/SKILL.md`、`templates/batch-manifest.yaml`、`templates/allowlist-config.yaml` |
| 实现形态 | Prompt-Skill Workflow → 需生成 IMPLEMENTATION.md |

## 实现对象清单

| # | 文件 | 操作 | 行数 | 说明 |
|:---:|---|---|---|---|
| 1 | `skills/itr-ticket-ingestion/SKILL.md` | 新建 | 670 | Skill 主文件，共享写入（本 Story 写入 §1-§5，§6-§8 占位给后续 Story） |
| 2 | `skills/itr-ticket-ingestion/templates/batch-manifest.yaml` | 新建 | 28 | 批次清单模板 |
| 3 | `skills/itr-ticket-ingestion/templates/allowlist-config.yaml` | 新建 | 27 | Allowlist 白名单配置模板 |
| 4 | `skills/README.md` | 修改 | +14 | 新增 ptm-tse 章节与 itr-ticket-ingestion 条目 |

## 设计契约映射

| LLD 章节 | 设计要点 | SKILL.md 实现章节 | 验证状态 |
|---|---|---|---|
| §2.1-2.2 接口设计 | 触发接口 + ITR HTTP GET 请求接口 | §1（参数表格 + 触发语句示例） | ✅ |
| §2.3-2.4 Allowlist | 配置结构 + 校验算法（6 步） | §2.1（allowlist-config.yaml 引用）+ §2.2（6 步伪代码） | ✅ |
| §2.5 快照保存接口 | temp + 0600 + atomic + sha256 + 元数据 | §3.3（8 步伪代码） | ✅ |
| §2.6-2.7 Batch Manifest + 存储策略 | 模板结构 + 路径权限策略 | §3.1（策略表）+ §4.1（batch-manifest.yaml 模板）| ✅ |
| §4.1-4.2 核心流程 | ITR HTTP 摄取 + 分页 | §2.3（HTTP GET）+ §2.4（分页伪代码）+ §3 + §4 + §5 | ✅ |
| §5 状态机 | 无状态请求-响应（IngestionBatch 状态由 CLEAN 管理） | §5（DAO 写入） | ✅ N/A（本 Story 不维护状态机） |
| §6 错误处理与降级 | 6 种错误类型 + 降级原则 | "失败处理"章节（错误表 + 降级原则） | ✅ |
| §8 安全与权限 | deny-by-default + 不推断认证 | §2（allowlist）+ "不适用边界"（禁止操作清单） | ✅ |
| §9 实施步骤 | 4 个切片 | 切片 1-4，全部完成 | ✅ |
| §10 回滚策略 | 4 种回滚方式 | "不适用边界"中说明 | ✅ N/A（Skill 定义文件） |
| §11 平台差异 | 无差异（Python 标准库） | §1 平台差异段落 | ✅ |
| §12 集成契约 | 对 CLEAN/INGEST-DB/NRA-03 的契约 | §5.1（DAO 约束）+ §6-§8（占位符） | ✅ |
| §13 开放项与假设 | O-ING-01、O-ING-02 | 保留在 return packet open_items 中 | ✅ OPEN |
| §14 DoD | 全部 16 项 CP6 检查通过 | CP6 PASS | ✅ |

## 单元测试与 Fixture 计划

本 Story 输出为 Skill 定义文件（Markdown + YAML 模板），不包含可执行代码。LLD §7 定义了完整的测试计划：

| 类型 | 用例数 | 覆盖维度 |
|---|---|---|
| 安全拒绝 | 8（T-ING-02..09） | URL/方法/参数/认证头/分页上限 |
| HTTP 正常/异常 | 3（T-ING-01/10/11） | 成功/500/超时 |
| 快照保存 | 3（T-ING-12/13/15） | 路径权限/元数据/权限失败清理 |
| Batch Manifest | 1（T-ING-14） | 模板结构校验 |
| **总计** | **15** | |

Fixture：`fixtures/itr_valid_response.json`、`fixtures/itr_unauthorized_response.json`、`fixtures/allowlist_config.yaml`、`fixtures/mock_http_client.py`

当 Skill 被 Agent 执行时，对应的 Python 实现代码将通过上述 fixture 和 pytest 进行验证。

## 最小实现切片

| 切片 | 内容 | 对应 TASK-ID | 文件 | 状态 |
|:---:|---|---|---|---|
| 1 | Skill 骨架 + Allowlist 校验 | TASK-ING-01, TASK-ING-02 | SKILL.md §1-§2, allowlist-config.yaml | ✅ |
| 2 | HTTP GET 请求 + 响应处理 + 分页 | TASK-ING-03 | SKILL.md §2.3-§2.4 | ✅ |
| 3 | 原始快照保存（磁盘预检 + temp + 0600 + atomic + sha256） | TASK-ING-04 | SKILL.md §3 | ✅ |
| 4 | Batch Manifest 生成 + 数据库写入（DAO only） | TASK-ING-05, TASK-ING-06 | SKILL.md §4-§5, batch-manifest.yaml | ✅ |

## 平台差异处理

**结论：N/A**。Skill 定义文件（Markdown/YAML）不依赖任何平台特定能力。执行时代的 HTTP 客户端差异已在 SKILL.md §1 "平台差异"段落中说明，各 Agent 平台使用其内置 HTTP 能力，但必须遵守相同的 allowlist 校验规则。

## 验证结果

| # | 验证项 | 方法 | 结果 |
|:---:|---|---|---|
| 1 | SKILL.md frontmatter 完整 | 人工对照规范 | ✅ |
| 2 | 必需章节齐全（触发/输入/步骤/输出/不适用） | 人工对照 LLD | ✅ |
| 3 | allowlist 白名单逻辑（6 步伪代码） | 人工审查 | ✅ |
| 4 | 快照保存安全性（temp+0600+atomic+清理） | 人工审查 | ✅ |
| 5 | batch-manifest.yaml 模板完整性 | 人工审查 | ✅ |
| 6 | allowlist-config.yaml 配置正确 | 人工审查 | ✅ |
| 7 | 禁止文件未修改（`git diff data/` 为空） | git diff | ✅ |
| 8 | DAO 只调用公共接口（无直接 SQL） | 人工审查 §5.1 | ✅ |
| 9 | skills/README.md 已更新 | 人工审查 | ✅ |
| 10 | 共享写入标记（shared=true + shared_writers） | 人工审查 | ✅ |

## 未覆盖项

| 项 | 原因 | 后续 |
|---|---|---|
| LLD §7 15 个单元测试 | 本 Story 输出 Skill 定义文件，非可执行代码 | Skill 执行时由 Agent 平台运行 |
| O-ING-01 (ITR schema) | 需首次受控探测 | 首次执行时探测并记录 |
| O-ING-02 (ITR 分页) | 需首次受控探测 | 首次执行时确认并修正 |

## 设计缺口反馈

无。LLD v1.2 覆盖完整，14 章节设计契约全部映射到 Skill 文件中，无超出设计的新增内容。

## 安全合规确认

| 规则 | 状态 |
|---|---|
| 不读取系统凭据存储 | ✅ 无凭据读取逻辑 |
| 不向 ITR 发出写操作 | ✅ 方法白名单只允许 GET |
| allowlist deny-by-default | ✅ 6 步校验，任一不通过即拒绝 |
| 原始数据不进 Git | ✅ `data/snapshots/` 已在 `data/.gitignore` 排除 |
| 受限存储权限 | ✅ `0700` 目录 + `0600` 文件，硬断言 |
| 不修改 data/.gitignore, schema.sql, dao.py | ✅ git diff 确认 |

## 后续交接

| 接收方 | 阶段 | 交付物 |
|---|---|---|
| meta-qa | CP7 验证 | 本 IMPLEMENTATION.md + SKILL.md + 模板 + CP6 检查 |
| ST-RA-05.2-CLEAN | Wave 1 实现 | SKILL.md §6 占位 + batch-manifest.yaml |
| ST-RA-06.1-DETECT | Wave 2 实现 | SKILL.md §8 占位 + batch-manifest.yaml |
| ST-NRA-03 | Wave 2/3 实现 | SKILL.md §7 占位 |
