---
checkpoint: "CP6"
story_id: "ST-RA-01"
story_slug: "qualification-evidence"
canonical_story_id: "ST-RA-01"
feature: "FEAT-RA-ANALYSIS"
wave: 2
result: "PASS"
executed_by: "meta-dev"
executed_at: "2026-07-16"
evidence_ref: "process/evidence/ST-RA-01-qualification-evidence.index.json"
design_evidence_ref: "process/stories/STORY-RA-01-qualification-evidence-LLD.md"
return_packet_ref: "process/returns/ST-RA-01-qualification-evidence.return.json"
---

# CP6: ST-RA-01 资格检查与可信输入建立 — 编码完成

## Entry Criteria

| 条件 | 状态 |
|------|------|
| Story `status=dev-ready` 或等价可执行状态 | 通过（用户直接指令实现） |
| 设计证据已确认（ST-RA-01-LLD.md v1.1） | 通过（LLD 完整，14 章节已覆盖） |
| 依赖 `ST-RA-INGEST-DB` 完成 | 通过（Wave 1 已完成，`data/dao.py` 可用） |
| 文件所有权不冲突 | 通过（`skills/reverse-analysis/SKILL.md` shared，写入 §1-§2；`agents/ptm-tse.md` 更新） |

---

## 实现前置检查

| 检查项 | 结果 |
|--------|------|
| DAO 公共接口可导入 | 通过：`get_tickets_by_batch()`, `get_batch()`, `get_ticket_by_source_id()` 可用 |
| `ticket` 表字段与 LLD §12.1 契约对比 | 通过：`severity`, `quality_flag`, `source_ticket_id` 存在；`is_internal` 缺失已在 Skill 中标注 |
| `quality_flag` 实际枚举值与 LLD 差异 | 已处理：实际值为 `clean/incomplete/anomaly/blocked`（非 PASS/BLOCKED/WARNING），Skill 使用实际值 |
| 上游 Story 产物完整性 | 通过：`data/schema.sql` + `data/dao.py` + `data/.gitignore` 就绪 |
| 输出目录存在 | 通过：`skills/reverse-analysis/` 和 `skills/reverse-analysis/templates/` 已创建 |

---

## 实现对象清单

| 文件 | 操作 | 行数（估） | 对应 TASK-ID | 状态 |
|------|------|----------|-------------|------|
| `skills/reverse-analysis/SKILL.md` | 创建 | ~360 | TASK-ANL-01 ~ TASK-ANL-05 | 完成 |
| `skills/reverse-analysis/templates/` | 创建（空目录） | — | TASK-ANL-01 | 完成 |
| `agents/ptm-tse.md` | 修改 | ~180（全量重写） | TASK-ANL-06 | 完成 |

---

## 设计契约映射

| LLD 章节 | 契约要点 | SKILL.md 对应 | 状态 |
|----------|---------|--------------|------|
| §2.1 入口契约 | 触发条件、输入、输出（eligible/rejected/deferred）、禁止项 | §1 + 安全与禁止事项 | 覆盖 |
| §2.2 资格判定矩阵 | P1→eligible, P2→确认, P3/P4→rejected, is_internal→deferred, quality_flag→blocked | §1.2 Step 3-6 + §1.4 | 覆盖 |
| §2.3 证据分类接口 | evidence_lines 结构（line_id, category, validity, source, gap_owner, clarification_status） | §2.3.3 | 覆盖 |
| §3.1 只读表消费 | ticket/ingestion_batch/ticket_version 只读 | §2.2 + 前置条件 | 覆盖 |
| §3.2 内存状态 | EligibilityResult + EvidenceLineSet 生命周期 | §2.5 | 覆盖 |
| §4 核心流程 | 5 步资格+证据流程 | §1.2 (Step 1-6) + §2 (Step 2.1-2.6) | 覆盖 |
| §5 资格状态枚举 | eligible/eligible_on_request/rejected/deferred/blocked | §1.3 | 覆盖 |
| §6 错误处理与降级 | SQLite 失败/ticket 不存在/quality_flag 缺失/severity 为空/全字段为空/batch 缺失 | §2.6 | 覆盖 |
| §8 安全与权限 | 只读/无凭据/无外部访问/无生产写入/脱敏 | 安全与禁止事项 | 覆盖 |
| §11 平台差异 | Claude Code/Codex/Qoder 路径差异 | 平台差异 | 覆盖 |
| §12 集成契约 | 上游依赖（ST-RA-INGEST-DB）+ 下游消费（RA-02/NRA-01/NRA-02/06.2） | 前置条件 + §3-§8 占位符 | 覆盖 |

**契约匹配结论**：全部 11 项 LLD 契约已映射到产物。`is_internal` 字段缺失（当前 schema 未包含）已在 Skill §1.2 Step 5 中标注为上游依赖，并在前置条件 §4 中记录。

---

## 单元测试与 Fixture 计划

### 本 Story 适用测试策略

由于产出物是 Skill 文本定义和 Agent 配置（非可执行代码），测试方式为**结构化审查**和**契约验证**：

| 测试层 | 方式 | 覆盖范围 |
|--------|------|---------|
| L1 结构完整性 | Skill frontmatter 字段校验 | name, description, shared, shared_writers, status |
| L2 契约一致性 | LLD 章节交叉引用检查 | §1-§12 全部 11 项契约 |
| L3 安全规则 | 禁止操作枚举完整性 | 9 项禁止操作全部声明 |
| L4 流程完整性 | 资格判定→证据分类→阈值检查→错误处理 全路径覆盖 | 所有 6 条资格分支 + 4 条错误路径 |
| L5 下游消费 | 占位符章节完整性 | §3-§8 全部 6 个占位符，每项包含实现责任方和交付范围 |

### 验收标准覆盖

| AC | 验收条件 | SKILL.md 对应 | 状态 |
|----|---------|--------------|------|
| AC1 | P1 必做、P2 可选、P3/P4 不自动进入 | §1.2 Step 4 | 覆盖 |
| AC2 | 内部问题正确识别并标记 deferred | §1.2 Step 5 | 覆盖 |
| AC3 | 五条证据线分为事实/假设/未知项 | §2.3.2 | 覆盖 |
| AC4 | <3 条有效线时根因状态不能为 confirmed | §2.4.2（标记 sufficient/insufficient + max_rc_state） | 覆盖 |
| AC5 | `agents/ptm-tse.md` 正确引用 reverse-analysis | ptm-tse.md skills: [reverse-analysis] + 调用流程 | 覆盖 |

> **注**：AC4 的硬阻断（不能为 confirmed）在 ST-NRA-01（§6 占位符）中完整实现。本 Story 完成阈值判定和 max_rc_state 标记，为下游阻断提供输入。

---

## 最小实现切片

| Slice | 内容 | 文件 | 验证方式 |
|-------|------|------|---------|
| S1 | Skill 骨架 + 元数据 | SKILL.md frontmatter + 目标/前置 | 字段完整性检查 |
| S2 | 资格检查规则 | SKILL.md §1 | P1→eligible, P3→rejected, is_internal→deferred 文本路径验证 |
| S3 | 证据分类规则 | SKILL.md §2 | fact/hypothesis/unknown/gap 分类定义 + valid_count 计算规则验证 |
| S4 | 三线阈值 | SKILL.md §2.4 | valid_count >= 3 判定 + 缺口清单输出逻辑验证 |
| S5 | Agent 引用 | ptm-tse.md | skills frontmatter + 调用流程引用 |

**全部切片完成**。

---

## 平台差异处理

| 平台 | 项 | 处理 | 状态 |
|------|---|------|------|
| Claude Code | Skill 路径 `skills/reverse-analysis/SKILL.md` | 标准路径，无需特殊处理 | N/A |
| Codex | 安装后映射至 `.agents/skills/reverse-analysis/SKILL.md` | 安装器负责映射；Skill canonical 路径不变 | N/A |
| Qoder | 安装后映射至 `.qoder/skills/reverse-analysis/SKILL.md` | 同上 | N/A |

**结论**：本 Story 无平台差异。资格检查与证据分类语义在所有平台一致。

---

## 验证结果

### 结构检查

- `skills/reverse-analysis/SKILL.md`：frontmatter 包含所有必需字段（name, description, argument-hint, user-invokable, status, shared, shared_writers, version, source_cr, source_feature, source_lld）
- 正文包含：目标、前置条件、§1-§8（§1-§2 完整实现，§3-§8 占位符）、安全与禁止事项、平台差异、不适用边界、修订记录
- `agents/ptm-tse.md`：frontmatter skills 列表包含 `reverse-analysis`；正文包含调用流程、权限边界和检查点

### 契约一致性

- LLD §2.1 入口契约 → SKILL.md §1（触发条件、输入、输出、禁止）
- LLD §2.2 资格判定矩阵 → SKILL.md §1.2 Step 4 + §1.4
- LLD §2.3 证据分类接口 → SKILL.md §2.3.3
- LLD §6 错误处理 → SKILL.md §2.6
- LLD §8 安全 → 安全与禁止事项（9 项禁止操作，6 项允许操作）
- LLD §12 集成契约 → 前置条件（上游）+ §3-§8 占位符（下游）

### 禁止项覆盖

| 禁止操作 | SKILL.md 声明位置 | ptm-tse.md 声明位置 |
|----------|------------------|-------------------|
| 直接 SQL | 禁止事项第 1 条 | 权限边界 DAO 只读 |
| 修改 dao.py/schema.sql | 禁止事项第 2 条 | —（Skill 级约束） |
| 读取 raw_json | 禁止事项第 3 条 + §1.2 Step 1 + §2.2 | 权限边界 禁止原始数据读取 |
| 外部连接 | 禁止事项第 4 条 | 权限边界 无外部访问 |
| 读取凭据 | 禁止事项第 5 条 | 权限边界 无凭据读取 |
| 自动确认根因 | 禁止事项第 6 条 + 安全声明 | 权限边界 不自动确认 |
| 自动分发 CA/PA | 禁止事项第 7 条 | 权限边界 不自动分发 |
| 修改 ticket 表 | 禁止事项第 8 条 | 权限边界 不修改源数据 |
| raw_statement 直接展示为结论 | 禁止事项第 9 条 | —（Skill 级约束） |

**结论**：全部 9 项禁止操作在 Skill 和/或 Agent 层面有显式声明。

---

## 未覆盖项

| 项 | 原因 | 责任 Story | 重访条件 |
|----|------|-----------|---------|
| `is_internal` 字段的 schema 定义 | 当前 schema 不包含该字段；已在 Skill 中标注缺失行为和重访条件 | ST-RA-INGEST-DB 或后续 CR | schema 新增该列后更新 Skill 中的字段依赖说明 |
| 三线阈值硬阻断 | 在 ST-NRA-01 中实现（§6 占位符） | ST-NRA-01 | CP7 验证时确认阻断逻辑与阈值规则一致 |
| 根因四层状态机实现 | 在 ST-RA-02 中实现（§3 占位符） | ST-RA-02 | — |
| DAO `is_internal` 查询支持 | DAO 当前无 `is_internal` 列的读取逻辑 | ST-RA-INGEST-DB 或 F-021 CR | 新增列后 DAO 需提供对应查询方法 |

---

## 设计缺口反馈

| 发现 | 类型 | 说明 | 建议 |
|------|------|------|------|
| `quality_flag` 枚举值与 LLD 不一致 | 契约 gap | LLD §12.1 要求 `PASS/BLOCKED/WARNING`，实际 schema 为 `clean/incomplete/anomaly/blocked` | 已在 Skill 中使用实际值；建议 CP5 或后续 CR 对齐 LLD 与 schema 的枚举定义 |
| `is_internal` 列缺失 | schema gap | LLD §12.1 要求存在该列，当前 schema 不包含 | 已在 Skill 中标注缺失行为和降级策略（视为 false）；需上游 Story 补充 |
| `severity` 无 CHECK 约束 | schema 松弛 | LLD 依赖 `P1/P2/P3/P4` 枚举，但 schema 中 severity 为普通 TEXT 列 | 已在 Skill 中定义非 P1-P4 值的降级策略（视为空→rejected） |

---

## 后续交接

### 验证入口（供 meta-qa 使用）

1. **Skill 结构完整性**：
   - 检查 `skills/reverse-analysis/SKILL.md` frontmatter 字段齐全
   - 检查 §1-§2 为完整实现、§3-§8 为占位符
   - 检查 shared_writers 列出所有 6 个下游 Story

2. **资格检查规则验证**：
   - P1 → eligible, P2 → eligible_on_request, P3/P4 → rejected
   - is_internal=true → deferred
   - quality_flag=blocked → blocked
   - severity 为空 → rejected

3. **证据分类规则验证**：
   - fact/hypothesis/unknown/gap 分类定义完整
   - validity 判定规则（fact/hypothesis → valid/incomplete, unknown/gap → invalid）
   - 三线阈值 valid_count >= 3 逻辑正确

4. **安全规则验证**：
   - 9 项禁止操作逐一声明
   - deny-by-default 原则明确
   - ptm-tse.md 权限边界与 Skill 禁止项一致

5. **Agent 引用验证**：
   - ptm-tse.md frontmatter skills 包含 `reverse-analysis`
   - 正文调用流程引用 reverse-analysis Skill 的 §1 和 §2
   - 权限边界与 Skill 禁止操作一致

### 风险提示

- **OPEN-RA01-01**：P3/P4 默认拒绝不可手动绕过（假设）
- **OPEN-RA01-02**：quality_flag=clean 是最低门槛（假设；incomplete/anomaly 已允许但带风险标记）
- **OPEN-RA01-04**：severity 枚举值假设与 P1/P2/P3/P4 一致（schema 无 CHECK 约束，已做降级处理）
- **设计缺口**：`is_internal` 列缺失，当前所有 ticket 视为非内部问题

---

## Exit Criteria

| 条件 | 状态 |
|------|------|
| 所有输出文件存在且非空 | 通过 |
| LLD 11 项契约全部映射到产物 | 通过 |
| TASK-ANL-01 ~ TASK-ANL-06 全部完成 | 通过 |
| 验收标准 5 项全部覆盖 | 通过 |
| 安全禁止项 9 项全部声明 | 通过 |
| 设计缺口 3 项已记录并分配责任 Story | 通过 |
| 下游 Story 占位符 6 个完整 | 通过 |

**CP6 结论**：**PASS** — Story ST-RA-01 编码完成，产物就绪，可交给 meta-qa 进行 CP7 验证。
