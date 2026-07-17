---
check_id: CP6-STORY-RA-06.2-REFRESH
workflow_id: WF-PTM-TEAM-20260520-001
change_id: CR-030
cp: CP6
type: auto
status: PASS
checked_at: "2026-07-16T00:00:00+08:00"
checked_by: meta-dev
---

# CP6 - 编码完成（ST-RA-06.2-REFRESH: S2 增量重算与差异报告）

## Entry Criteria

| 条件 | 状态 | 证据 |
|------|------|------|
| Story card 完整 | ✓ | `process/stories/STORY-RA-06.2-REFRESH-s2-incremental.md`，含 dev_context、validation_context、acceptance_criteria、AI 任务清单 |
| LLD 设计证据就绪 | ✓ | `process/stories/STORY-RA-06.2-REFRESH-s2-incremental-LLD.md`（full-lld，14 章节），status=lld-approved |
| 前置依赖满足 | ✓ | ST-RA-05.3-ANALYZE（§6 S1 管线）已完成 + ST-RA-06.1-DETECT（change_history 表）已完成 |
| 文件所有权不冲突 | ✓ | `skills/reverse-analysis/SKILL.md` shared（ST-RA-06.2 写 §9），前置 Story 无冲突 |
| dev_gate 满足 | ✓ | Wave 3，当前 Story 无 runtime/file-conflict 阻塞 |

## Checklist

| # | 检查项 | 结果 | 说明 |
|---|--------|------|------|
| 1 | 不修改 §1-§8 | PASS | §1（资格检查）、§2（可信输入建立）、§3-§5（六维引擎）、§6（S1 管线）、§7（证据不足保护）、§8（权限边界拒绝）全部原样保留 |
| 2 | §9 占位符替换 | PASS | 原 "Artifact 交付与 Reviewer 确认" 占位符替换为完整 §9 S2 增量重算与差异报告章节 |
| 3 | §9.1 增量重算触发 | PASS | 含 S2 入口判定（输入条件）、change_set 消费（change_history 表 + change_type 处理规则）、前置校验（4 项 baseline/new batch 存在性校验 + 批次锁定） |
| 4 | §9.2 受影响维度判定 | PASS | 含变更字段→分析维度映射表（13 个 ticket 字段 × 6 个分析维度）、受影响维度计算流程（4 步 DM-1→DM-4）、受影响 ticket 集合（affected/unaffected/skipped 三分类） |
| 5 | §9.3 增量重算策略 | PASS | 含增量重算三大原则、逐维度增量策略（RC/PQ/ESC/TM/IMP/CMP 各维度独立策略）、增量重算执行流程（5 步 IR-1→IR-5）、聚合维度增量合并规则（PQ/TM/IMP 三示例） |
| 6 | §9.4 comparison_batch_ref 管理 | PASS | 含 overview（5 场景取值）、对比批次合法性校验（存在性+同口径+窗口完整性）、analysis_run 中 comparison_batch_ref 写入示例 |
| 7 | §9.5 recompute_mode 判定 | PASS | 含判定流程图、规则版本变更检测（schema_version/mapping_version/analysis_rule_version 三步检测）、full vs incremental 对比表、无受影响维度处理、before/after 来源表、环比同比 N/A 7 条件判定 |
| 8 | §9.6 差异报告生成 | PASS | 含差异计算流程（4 步 DR-1→DR-4）、空差异报告格式、显著变化阈值表（6 类变化 × 默认阈值）、措施刷新提示（keep/complete/needs_review/invalidated 四建议） |
| 9 | §9.7 差异报告输出格式 | PASS | 含完整差异报告 YAML 结构（difference ra-report，含 change_summary + 6 维 dimension_diffs + comparison + improvement_refresh + skipped_tickets）、与 S1 报告差异表、analysis_run 创建流程、S2 管线 7 步总览 |
| 10 | 安全约束完整 | PASS | §9.9 含 S2 专属禁止项（P-S2-01 至 P-S2-06）+ S2 允许操作范围表 |
| 11 | 集成契约完整 | PASS | §9.10 含上游依赖（4 Story）+ 下游消费（3 Story）+ S1/S2 管线边界对比表 |
| 12 | 测试设计完整 | PASS | §9.12 含 14 条测试场景（7 正向 + 7 负向/边界）+ 7 组 Fixture 设计 |
| 13 | 不修改 data/ 文件 | PASS | 仅通过 DAO 公共接口消费（get_changes_by_batch/get_tickets_by_batch/get_analysis_run/get_runs_by_batch/insert_analysis_run/update_analysis_run_draft/get_batch/get_tickets_by_time_range），不直接 SQL |
| 14 | 开放项显式声明 | PASS | §9.11 含 5 项 OPEN 决策/假设（OPEN-RA062-01 至 OPEN-RA062-05），逐项标注状态、回访条件和当前保守策略 |
| 15 | frontmatter + 版本更新 | PASS | shared_writers 中 ST-RA-06.2 描述从 "artifact 交付与 reviewer 人工确认" 更新为 "S2 增量重算与差异报告"，version 1.4→1.5 |
| 16 | 修订记录更新 | PASS | v1.5 修订记录追加，含实现摘要（13 章节 + 措施刷新 + 环比同比 + 上下游 + 测试 + Fixture）和来源（ST-RA-06.2-REFRESH LLD v1.1） |

## 实现文件

| 文件 | 操作 | 行/字节 | 说明 |
|------|------|--------|------|
| `skills/reverse-analysis/SKILL.md` | 修改（替换 §9 占位符） | +约 880 行 | 替换原 11 行占位符为完整 S2 增量重算章节（§9.1–§9.13，13 个子章节） |

## 实现对象清单

| 对象 | 位置 | 说明 |
|------|------|------|
| S2 入口路由逻辑 | §9.1.1 | new_batch_id + baseline_batch_id + change_set 判定 |
| change_set 消费规则 | §9.1.2 | change_history 表消费 + 4 种 change_type 处理 |
| 变更字段→维度映射表 | §9.2.1 | 13 字段 × 6 维度完整映射 |
| 受影响维度计算流程 | §9.2.2 | DM-1→DM-4 四步计算 |
| 逐维度增量策略 | §9.3.2 | 6 维度各自的增量/复用策略 |
| 聚合维度合并规则 | §9.3.4 | PQ/TM/IMP 三维度的增量合并示例 |
| comparison_batch_ref 管理 | §9.4 | 5 场景取值 + 3 步合法性校验 |
| recompute_mode 判定 | §9.5 | 规则版本变更检测 + full/incremental 决策树 |
| 差异报告生成 | §9.6 | before/after 比较 + 显著变化 20% 阈值 + 措施刷新 |
| 差异报告输出格式 | §9.7 | 完整 difference ra-report YAML 结构 + 与 S1 差异表 |
| S2 管线 7 步总览 | §9.7.4 | 端到端管线串联 |
| 错误处理与降级 | §9.8 | 10 种异常场景 × 处理方式 |
| S2 安全禁止项 | §9.9 | 6 项 P-S2-01 至 P-S2-06 + 允许操作范围 |
| S1/S2 协同路由 | §9.13 | S1/S2 角色分配表 + 入口路由规则 |

## 设计契约映射

| LLD 章节 | §9 实现位置 | 覆盖率 |
|----------|-----------|--------|
| LLD §2.1 S2 入口契约 | §9.1.1 + §9.1.3 | 完整（入口条件 + 前置校验） |
| LLD §2.2 变更字段→维度映射表 | §9.2.1 | 完整（13 字段映射） |
| LLD §2.3 环比同比接口 | §9.5.6 + §9.7.1（comparison 段） | 完整（mode/window/metrics/credibility/na_reasons） |
| LLD §3.1 差异报告数据模型 | §9.7.1（完整 YAML 结构） | 完整（change_summary + dimension_diffs + improvement_refresh） |
| LLD §3.2 SQLite 表依赖 | §9.1.2 + §9.4.2 + §9.10.1 | 完整（5 张表 + DAO 接口） |
| LLD §4 核心流程（8 步） | §9.7.4（7 步 S2 管线总览） | 合并为 7 步（合并差异生成与环比同比），逻辑完整 |
| LLD §5 状态机（recompute_mode） | §9.5 | 完整（full/incremental 决策树 + 条件表） |
| LLD §6 错误处理与降级 | §9.8 | 完整（10 场景） |
| LLD §7 测试设计 | §9.12 | 完整（14 测试场景 + 7 Fixture） |
| LLD §8 安全与权限 | §9.9 | 完整（6 项禁止 + 允许操作范围） |
| LLD §9 实施步骤（5 TASK-ID） | §9 各子章节 | 完整（TASK-ANL-21 至 TASK-ANL-25 均已在对应章节实现） |
| LLD §10 回滚策略 | 通过 analysis_run 分离设计覆盖 | 新 run 不覆盖旧 run，回滚只需删除新 run |
| LLD §12 集成契约 | §9.10 | 完整（上游 4 + 下游 3 + S1/S2 边界） |
| LLD §13 开放项 | §9.11 | 完整（5 项 OPEN） |

## 单元测试与 Fixture 计划

本 Story 为 Skill Markdown 规则定义，实现形态为声明式规范文本。测试通过以下方式覆盖：

| 验证方式 | 范围 | 覆盖 |
|---------|------|------|
| **CP6 自检** | 检查项 1-16（本文件） | 结构完整性、设计契约映射、安全约束、开放项 |
| **LLD 覆盖追溯** | 设计契约映射表（本文件） | 14/14 LLD 章节有对应 §9 实现位置 |
| **变更字段→维度映射表** | §9.2.1 | 13/13 ticket 字段有映射目标维度 |
| **错误/边界场景** | §9.8 + §9.12.2 | 10 错误场景 + 7 边界测试场景 |
| **Fixture 设计** | §9.12.3 | 7 组 Fixture 覆盖核心+边界场景 |

**N/A 理由**：本 Story 实现 Skill Markdown 规则定义，非可执行代码，不适用传统单元测试。声明式规范的正确性通过 CP6 checklist、LLD 设计契约映射和结构完整性校验保证。CP7 验证时由 meta-qa 通过 static-only/review-only 方式验证规范完整性和一致性。

## 最小实现切片

| 切片 | 内容 | 依赖 | 状态 |
|------|------|------|------|
| Slice 1 | §9.1 增量重算触发 + §9.2 受影响维度判定 | ST-RA-06.1-DETECT 的 change_history 表 | ✓ 完成 |
| Slice 2 | §9.3 增量重算策略 + §9.4 comparison_batch_ref 管理 | Slice 1 | ✓ 完成 |
| Slice 3 | §9.5 recompute_mode 判定 + §9.6 差异报告生成 | Slice 2 | ✓ 完成 |
| Slice 4 | §9.7 差异报告输出格式 + §9.8–§9.13 补充章 | Slice 3 | ✓ 完成 |

## 平台差异处理

**N/A**：本 Story 仅追加 Skill Markdown 规则定义到 `skills/reverse-analysis/SKILL.md`，不涉及平台特定路径或代码。差异报告的环比同比计算为纯数据聚合逻辑，所有平台一致。

## 关键决策与偏差

| 决策 | 理由 |
|------|------|
| 环比同比始终重算 | 即使所有 ticket 字段不变，窗口时间推进后聚合结果也可能变化 |
| conservative fail：任一规则版本变更→全量重算 | OPEN-RA062-01 未决策前采用最安全策略 |
| 差异报告显著变化阈值默认 20% | OPEN-RA062-05 假设，可在 metric-definition.yaml 中覆盖 |
| change_set 通过 change_history 表消费 | ST-RA-06.1-DETECT 已确认格式（OPEN-RA062-03 RESOLVED） |

## 已知限制

1. 聚合维度增量合并规则（§9.3.4）为声明式规范，实际执行时需实现相应 SQL 聚合逻辑
2. 措施刷新提示（§9.6.4）依赖 ST-RA-03 产出已批准 CA/PA 列表，该集成链路尚未端到端验证
3. 环比同比的最小样本阈值 10（OPEN-RA062-02）为默认值，无历史数据支撑该值的统计显著性

## 未覆盖项

| 项 | 说明 |
|----|------|
| 实际运行验证 | 本 Story 实现声明式规范，运行时正确性由 ptm-tse Agent 消费方验证 |
| ST-RA-06.3-TRACK 集成 | improvement_refresh 的消费逻辑在 ST-RA-06.3 中实现 |
| metric-definition.yaml 阈值配置 | analysis_rule_version 变更检测需 metric-definition.yaml 的实际版本信号 |

## 设计缺口反馈

1. §9.5.2 规则版本变更检测中的 `analysis_rule_version` 来源为 `metric-definition.yaml`，当前 Skill 只定义了检测逻辑，未定义如何从 YAML 模板文件中提取活跃版本号。需在 metric-definition.yaml 中显式声明 `active_metric_version` 字段。
2. §9.2.1 映射表中 `title` 和 `description` 变更标记为"影响根因"，但该影响的精确性取决于实际 AI 分析对标题/描述字段的依赖程度，需在 runtime 验证中确认。

## 后续交接

### 给 meta-qa 的验证入口

| 入口 | 路径 | 验证内容 |
|------|------|---------|
| **SKILL.md 结构完整性** | `skills/reverse-analysis/SKILL.md#§9` | 验证 §9.1–§9.13 共 13 个子章节存在且非空 |
| **设计契约映射** | 本文件「设计契约映射」表 | 验证 14/14 LLD 章节有对应 §9 实现 |
| **变更字段映射覆盖** | §9.2.1 映射表 | 验证 13/13 ticket 字段有映射目标维度 |
| **安全禁止项** | §9.9 | 验证 6 项 P-S2-* 禁止项 + 允许操作范围完整 |
| **差异报告 YAML 结构** | §9.7.1 | 验证 YAML 结构覆盖 change_summary/dimension_diffs/comparison/improvement_refresh/skipped_tickets |
| **错误处理矩阵** | §9.8 | 验证 10 场景覆盖 blocked/skipped/N-A/degraded |
| **前后一致性** | §1-§8 vs §9 | 验证 §9 未修改 §1-§8，placeholders 语法一致 |
| **前端元数据** | SKILL.md frontmatter | 验证 version=1.5, shared_writers 中 ST-RA-06.2 描述更新，修订记录 v1.5 存在 |

### 风险提示

| 风险 | 严重度 | 说明 |
|------|--------|------|
| change_set 格式变化 | medium | 如 ST-RA-06.1-DETECT 后续调整 change_history 表结构，§9.1.2 的消费逻辑需同步更新 |
| 环比同比阈值误用 | low | 默认阈值 10 可能导致小窗口误 N/A，但可通过 metric-definition.yaml 覆盖 |
| 措施刷新提示未闭环 | medium | improvement_refresh 的 keep/needs_review/invalidated 提示需要 ST-RA-06.3-TRACK 消费；当前链路未端到端验证 |

## Conclusion

CP6 **PASS**。ST-RA-06.2-REFRESH（S2 增量重算与差异报告）Skill Markdown 规则定义实现完成。13 个子章节覆盖 LLD 14 章的全部设计契约，安全禁止项、错误处理、测试设计和集成契约均已闭环。可移交 meta-qa 进行 CP7 验证。
