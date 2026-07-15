# ptm-tde 高风险 Story 实现执行证据汇总

> 版本：v1.0 · 生成日期：2026-06-08 · 覆盖范围：CR-012、CR-013、CR-016、CR-017 中 M 层级及以上 Story

---

## 概述

ptm-tde v1.0 共交付 23 个 Story（7 条 CR）。其中以下 Story 属于 **M 层级（中等复杂度）及以上**，需记录完整实现执行证据。其余 S 层级 Story 的实现证据已在 CR 级 CP6 checks 中覆盖，低风险 N/A 理由见各 Story 卡片。

---

## CR-012 MFQ 阶段改造（8 Stories，高风险 Story = 3）

### STORY-012-03: M 分析器 v3.0 重写

| 维度 | 内容 |
|---|---|
| Tier | M |
| CP6 证据 | `process/checks/CP6-STORY-012-03-m-analyzer-v3-rewrite-CODING-DONE.md` PASS |
| LLD | `process/stories/STORY-012-03-m-analyzer-v3-rewrite-LLD.md` |
| 实现对象 | `skills/m-analyzer/SKILL.md`（10 步场景步骤驱动重写）、`skills/test-point-integrator/SKILL.md`（接口适配） |
| 设计契约 | MUST: PPDCS 标注 + CAE 测试点输出；MUST: Step 1.5 因子库加载（CR-017）；MUST: Step 1.6 atomic-ops CLI 查询（CR-016） |
| 测试/Fixture | CP7 全局验证 PASS（10/10），覆盖 M 分析输出格式、CAE 三元组完整性、因子库/atomic-ops 集成 |
| 最小切片 | Slice 1: 1-5 步基础 M 分析 → Slice 2: 6-8 步 PPDCS 标注 → Slice 3: 9-10 步 CAE 聚合 + atomic-ops 集成 |
| 平台差异 | Claude Code 环境，无 Codex 特定适配 |
| 交接摘要 | M 分析器从 v2.x 升级到 v3.0：新增场景步骤驱动、PPDCS 自动标注、因子库发现（CR-017）、原子操作消费（CR-016）；输出格式与 test-point-integrator 契约兼容 |

### STORY-012-04: F 分析器 v3.0 重写

| 维度 | 内容 |
|---|---|
| Tier | M |
| CP6 证据 | `process/checks/CP6-STORY-012-04-f-analyzer-v3-rewrite-CODING-DONE.md` PASS |
| LLD | `process/stories/STORY-012-04-f-analyzer-v3-rewrite-LLD.md` |
| 实现对象 | `skills/f-analyzer/SKILL.md`（9 步重写：Excel 批注 + 场景耦合 + 代码依赖） |
| 设计契约 | MUST: 输出 CAE 格式耦合测试点；MUST: 检测到新耦合点时触发检查点④ |
| 测试/Fixture | CP7 全局验证 PASS，覆盖耦合矩阵生成、Excel 批注提取、场景耦合识别 |
| 平台差异 | 无 |
| 交接摘要 | F 分析器从 v2.x 升级到 v3.0：扩展耦合维度（Excel 批注 + 场景 + 代码）、AskUserQuestion 交互增强（CR-015） |

### STORY-012-05: Q 分析器 v3.0 重写

| 维度 | 内容 |
|---|---|
| Tier | M |
| CP6 证据 | `process/checks/CP6-STORY-012-05-q-analyzer-v3-rewrite-CODING-DONE.md` PASS |
| LLD | `process/stories/STORY-012-05-q-analyzer-v3-rewrite-LLD.md` |
| 实现对象 | `skills/q-analyzer/SKILL.md`（6 步逐 TSP 驱动重写） |
| 设计契约 | MUST: HTSM 维度逐 TSP 评估；MUST: 输出 CAE 格式质量属性测试点 |
| 测试/Fixture | CP7 全局验证 PASS |
| 平台差异 | 无 |
| 交接摘要 | Q 分析器从 v2.x 升级到 v3.0：改为逐 TSP 驱动（每特性维度单独评估）、HTSM 维度扩展 |

---

## CR-013 PPDCS 阶段改造（4 Stories，高风险 Story = 2）

### STORY-013-01: 设计 Skill 路径迁移

| 维度 | 内容 |
|---|---|
| Tier | M |
| CP6 证据 | `process/checks/CP6-STORY-013-01-design-skills-migration-CODING-DONE.md` PASS |
| LLD | `process/stories/STORY-013-01-LLD.md` |
| 实现对象 | 8 个 PPDCS 设计 Skill（P-Process/P-Parameter/D-Data/C-Combination/S-State + 3 辅助 Skill） |
| 设计契约 | MUST: 每个 Skill 支持 PPDCS 五特征之一的设计方法；MUST: 输出 16 列物理用例格式 |
| 测试/Fixture | CP7 全局验证 PASS（10/10） |
| 平台差异 | 无 |
| 交接摘要 | 8 个设计 Skill 从旧路径迁移到 `skills/ppdcs/` 目录，新增 5 个方法论占位文档 |

### STORY-013-04: GATE-4 增强

| 维度 | 内容 |
|---|---|
| Tier | M |
| CP6 证据 | `process/checks/CP6-STORY-013-04-gate4-enhancement-CODING-DONE.md` PASS |
| LLD | `process/stories/STORY-013-04-LLD.md` |
| 实现对象 | `skills/checkpoint-manager/SKILL.md`（GATE-4 P1-P7 检查项）、`docs/ptm-tde/gate-spec.md` |
| 设计契约 | MUST: GATE-4 覆盖 P1-P7 全部检查项 |
| 测试/Fixture | CP7 全局验证 PASS |
| 平台差异 | 无 |
| 交接摘要 | GATE-4 从基础检查扩展为 P1-P7 七项检查（方法匹配/因子覆盖/组合压缩/边界覆盖/状态覆盖/流程完整性/交付格式） |

---

## CR-016 原子操作消费缺口修复（1 Story）

### STORY-016-01

| 维度 | 内容 |
|---|---|
| Tier | M |
| CP6 证据 | `process/checks/CP6-STORY-016-01-atomic-ops-consumption-gap-CODING-DONE.md` PASS |
| LLD | `process/stories/STORY-016-01-atomic-ops-consumption-gap-LLD.md` |
| 实现对象 | `skills/m-analyzer/SKILL.md`（Step 1.6 + Step 2C）、`skills/test-point-integrator/SKILL.md`（4.5.1.6）、`atomic-op-usage/` 跟踪目录 |
| 设计契约 | MUST: CLI `atomic-ops show <op_id>` 查询；MUST: L1-L4 五维语义匹配（含 ALIASES_WEIGHT=1.5）；MUST: 交叉验证 |
| 测试/Fixture | atomic-ops CLI 验证（79 ops all with tags + parameters_summary）、CP7 全局 PASS |
| 平台差异 | atomic-ops CLI 需独立 `uv tool install` |
| 交接摘要 | 解决 m-analyzer 原子操作消费缺口：从硬编码同义词改为 atomic-ops aliases 字段实时查询；覆盖 32/79 ops |

---

## CR-017 因子库发现缺口修复（1 Story）

### STORY-017-01

| 维度 | 内容 |
|---|---|
| Tier | M |
| CP6 证据 | `process/checks/CP6-STORY-017-01-factor-library-discovery-CODING-DONE.md` PASS |
| LLD | `process/stories/STORY-017-01-factor-library-discovery-LLD.md`（14 章节） |
| 实现对象 | `skills/m-analyzer/SKILL.md`（Step 1.5 + Step 2B）、`skills/test-point-integrator/SKILL.md`（4.5.1.5）、GATE-3 M8 |
| 设计契约 | MUST: 因子库清单加载（YAML schema）；MUST: match_confidence 四级分级；MUST: 反查验证 |
| 测试/Fixture | 6 TASK-ID 全部完成，10 个测试场景，CP7 18/18 PASS |
| 平台差异 | 无 |
| 交接摘要 | m-analyzer 新增 Step 1.5 因子库发现：自动加载 `resource/` 下因子库 YAML 文件，match_confidence 四级分级输出 |

---

## 低风险 Story N/A 理由

以下 S 层级 Story 的实现执行证据已在 CR 级 CP6 checks 中充分覆盖，不需要独立 IMPLEMENTATION.md：

| CR | S 层级 Story | N/A 理由 |
|---|---|---|
| CR-010 | STORY-010-01/02/03/04/05/06 | 结构层改造，文件路径调整为主 |
| CR-011 | STORY-011-01/02/03/04 | 内容层 KYM，4 文件修改 |
| CR-012 | STORY-012-01/02/06/07/08 | 路径迁移/适配/文档，低复杂度 |
| CR-013 | STORY-013-02/03 | 验证器/渲染器迁移，低复杂度 |
| CR-015 | fast-lane 单 Story | fast-lane 模式，CP6 已覆盖全部实现细节 |

---

*本文件为 ptm-tde v1.0 高风险 Story 的实现执行证据汇总。CP6 编码完成检查文件为权威证据来源。*
