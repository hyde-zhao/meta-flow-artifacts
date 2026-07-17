---
story_id: "STORY-RA-01"
canonical_story_id: "ST-RA-01"
title: "资格检查与可信输入建立"
status: "lld-draft"
design_evidence_type: "full-lld"
lld_policy_required_level: "full-lld"
feature: "FEAT-RA-ANALYSIS"
source_cr: "CR-030"
wave: 2
tier: "Tier-A"
created_by: "meta-dev"
created_at: "2026-07-16"
depends_on: ["ST-RA-INGEST-DB"]
shared_fragments:
  - shared_file: "skills/reverse-analysis/SKILL.md"
    serial_group: "reverse-analysis"
    write_section: "qualification, eligibility, evidence boundary"
open_items: []
---

# ST-RA-01 LLD: 资格检查与可信输入建立

## 0. 工程依据与模板索引

| 来源 | 消费内容 |
|---|---|
| HLD / Feature DESIGN / Feature Matrix | 四边界、可信治理、资格检查与 `full-lld` 约束 |
| Story `ST-RA-01` | 输出文件、验收标准和依赖 |

本文保留原有详细章节；目标、需求、模块拆分、代码结构、数据模型、API、流程、技术细节、安全、测试、实施、风险与 DoD 分别由后续编号章节定义。

> 对应 HLD REV-03 的 `reverse-analysis` Skill 边界 — 入口资格控制。

---

## 1. 文件影响范围

| 文件 | 操作 | 变更性质 | 所有者 |
|------|------|---------|--------|
| `skills/reverse-analysis/SKILL.md` | 创建 | 新建 Skill 定义，写入 §1–§3（元数据、资格检查、证据边界） | FEAT-RA-ANALYSIS（shared，写资格/证据段） |
| `skills/reverse-analysis/templates/` | 创建 | 新建模板目录（ra-report 等模板由 ST-RA-02 写入） | FEAT-RA-ANALYSIS |
| `agents/ptm-tse.md` | 修改 | 在 Agent 定义中添加 `reverse-analysis` Skill 引用 | FEAT-RA-ANALYSIS（编排更新） |

**不修改**：
- `data/schema.sql`（只读依赖，不写入）
- `docs/design/HLD.md`、`docs/design/FEATURE-DESIGN-MATRIX.md`（不被 meta-dev 修改）
- `skills/itr-ticket-ingestion/SKILL.md`（不同 Feature）

---

## 2. 接口设计

### 2.1 Skill 入口契约

`reverse-analysis` Skill 的资格检查段对外暴露以下语义接口（不是代码 API，而是 Skill 文本中定义的执行入口）：

| 接口项 | 定义 |
|--------|------|
| **触发条件** | ptm-tse Agent 在收到问题单分析请求时，第一步调用资格检查 |
| **输入** | 从 SQLite 读取的 ticket 记录（source_ticket_id, severity, product, is_internal, 可用字段） |
| **输出 — 通过** | 资格标记为 `eligible`：返回分析就绪的 ticket 引用 + 已分类的证据线列表 |
| **输出 — 拒绝** | 资格标记为 `deferred`（内部问题）或 `rejected`（P3/P4/权限不足） |
| **输出 — 证据缺口** | 资格通过但证据不足时，标记 `eligible_with_gaps` + 缺口清单 |
| **禁止** | 不读取 credentials、不连接外部系统、不直接确认根因、不修改 ticket 表 |

### 2.2 资格判定规则矩阵

| 条件 | 标记 | 后续动作 |
|------|------|---------|
| P1 事件 | `eligible` | 进入证据分类 |
| P2 事件 + 用户显式选择 | `eligible` | 进入证据分类 |
| P2 事件 + 未被选择 | `eligible_on_request` | 暂停，等待用户确认 |
| P3/P4 事件 | `rejected` | 输出拒绝原因 + 建议人工流程 |
| 标记为内部问题 | `deferred` | 输出不适用说明 + 重新立项条件 |
| 无 source_ticket_id 或关键字段缺失 | `blocked` | 中止，输出数据缺口 |
| 来源快照未通过质量检查 | `blocked` | 回溯 IngestionQualityReport |

### 2.3 证据分类接口

五条证据线（来自 HLD §1.1 + Feature DESIGN §2.1）的分类输出：

```yaml
evidence_lines:
  - line_id: string
    category: enum  # fact | hypothesis | unknown | gap
    validity: enum  # valid | incomplete | invalid
    source: string  # 引用来源字段或文档
    gap_owner: string  # 当 category=gap 时的补充责任人
    clarification_status: enum  # clear | needs_clarification
```

| 分类规则 | 说明 |
|---------|------|
| `fact` | 有明确字段值 + 可追溯到原始记录 |
| `hypothesis` | AI 分析候选，标注置信度和依据 |
| `unknown` | 字段存在但值不可信或矛盾 |
| `gap` | 字段缺失或无法从现有数据推导 |

---

## 3. 数据模型

本 Story 不创建 SQLite 表（属于 F-020 的写入范围），但消费以下只读结构：

### 3.1 依赖的 SQLite 表（只读）

来自 `data/schema.sql`（ST-RA-INGEST-DB 创建）：

| 表 | 关键消费字段 | 用途 |
|----|------------|------|
| `ticket` | `source_ticket_id`, `severity`, `product`, `status`, `quality_flag`, `is_internal`, `root_cause` | 资格判定 |
| `ingestion_batch` | `batch_id`, `quality_status` | 质量前置检查 |
| `ticket_version` | `source_ticket_id`, `version`, `field_changes` | 版本追溯（当前仅引用，不读取变更详情） |

### 3.2 内存/临时状态

| 对象 | 字段 | 生命周期 |
|------|------|---------|
| `EligibilityResult` | `status`, `reason`, `ticket_ref`, `quality_batch_ref` | Skill 单次调用内 |
| `EvidenceLineSet` | `lines[]`, `valid_count`, `gap_count` | Skill 单次调用内 |

注意：`quality_flag` 为 `BLOCKED` 时，资格检查结果为 `blocked`，不进入证据分类。

---

## 4. 核心流程

```
1. ptm-tse 接收分析请求（ticket 引用或 batch_id）
2. 资格检查
   2a. 从 SQLite 读取 ticket 记录
   2b. 检查 severity：P1 → eligible；P2 → 用户确认；P3/P4 → rejected
   2c. 检查 is_internal：true → deferred（输出重新立项条件）
   2d. 检查 quality_flag：BLOCKED → blocked（输出 IngestionQualityReport 引用）
   2e. 检查 source_ticket_id 有效性：空/冲突 → blocked
3. 证据分类（仅 eligible 分支）
   3a. 逐条读取可用字段（root_cause, improvement_measures, test_missed_analysis 等）
   3b. 按来源和确定性分类：fact / hypothesis / unknown / gap
   3c. 统计 valid 线数（category ∈ {fact, hypothesis} 且 validity = valid）
4. 阈值检查
   4a. valid_count >= 3 → 可进入 evidence-backed（但不自动确认）
   4b. valid_count < 3 → 输出缺口清单，禁止进入 confirmed
5. 输出 EligibilityResult + EvidenceLineSet
```

**关键决策点**：
- 资格通过但 evidence < 3 条：不影响资格状态，但下游 ST-RA-02 的根因状态机将限制在 AI candidate
- quality_flag = BLOCKED 时是硬阻断：不创建 analysis_run
- P2 未被用户显式选择时：暂停等待，不自动跳过

---

## 5. 状态机

本 Story 不包含需要跨调用持久化的状态机；资格结果是一次性判定。

但是，为下游提供进入条件的**资格状态枚举**：

```
eligible ──► 证据分类 → (valid_count >= 3) → 可进入根因四层
                    └── (valid_count < 3)  → eligible_with_gaps
eligible_on_request ──► 等待用户确认 → eligible 或 rejected
rejected ──► 终止，不创建 analysis_run
deferred ──► 终止，输出重新立项条件
blocked ──► 中止，输出 blockage 原因
```

**与根因四层状态机的衔接**（在 ST-RA-02 中实现）：
- 资格 `eligible` + valid_count >= 3：根因可从 raw_statement → AI candidate → evidence-backed
- 资格 `eligible` + valid_count < 3：根因只能到达 AI candidate

---

## 6. 错误处理与降级

| 场景 | 处理方式 | 降级输出 |
|------|---------|---------|
| SQLite 读取失败 | 抛出清晰错误 + batch 引用 | 不创建分析 |
| ticket 记录不存在 | `blocked` + source_ticket_id | 输出缺失记录清单 |
| quality_flag 缺失 | 视为 BLOCKED（保守） | 拒绝分析 |
| severity 字段为空 | 视为 P4 处理（默认拒绝） | `rejected` + 原因 |
| 证据字段（root_cause 等）全为空 | 全部标记 gap | 输出完整的缺口清单 |
| batch_id 指向的 batch 无 quality 记录 | `blocked` | 输出缺失 batch |

**降级原则**：任何不确定时偏向于阻断分析，而不是冒险输出不合格结论。

---

## 7. 测试设计

### 7.1 正向测试

| ID | 场景 | 输入 | 预期 |
|----|------|------|------|
| T-ANL-01 | P1 事件触发资格通过 | P1 ticket, quality_flag=PASS | eligibility.status = eligible |
| T-ANL-02 | P2 事件显式选择 | P2 ticket, 用户确认 | eligibility.status = eligible |
| T-ANL-06 | 五条证据线分类 | ticket 含各类字段 | lines 正确标注 fact/hypothesis/unknown/gap |
| T-ANL-S01-01 | valid_count >= 3 通过阈值 | 3 条 fact + 1 条 hypothesis | valid_count = 4, 可进入 evidence-backed |

### 7.2 负向/边界测试

| ID | 场景 | 输入 | 预期 |
|----|------|------|------|
| T-ANL-03 | P3 不自动进入 | P3 ticket | eligibility.status = rejected |
| T-ANL-04 | 内部问题拒绝 | is_internal=true | eligibility.status = deferred |
| T-ANL-05 | 证据不足（< 3 条有效线） | 2 条 fact, 其他 gap | valid_count < 3, gaps 清单完整 |
| T-ANL-S01-02 | valid_count = 0 | 所有字段为空 | 全部标记 gap, valid_count = 0 |
| T-ANL-S01-03 | quality_flag=BLOCKED | ticket 关联 BLOCKED batch | eligibility.status = blocked |
| T-ANL-S01-04 | severity 为空 | severity=null | 默认 P4 → rejected |

### 7.3 Fixture 设计

| Fixture | 覆盖 |
|---------|------|
| `fixtures/eligible_p1_ticket.json` | T-ANL-01 |
| `fixtures/eligible_p2_ticket.json` | T-ANL-02 |
| `fixtures/p3_rejected_ticket.json` | T-ANL-03 |
| `fixtures/internal_deferred_ticket.json` | T-ANL-04 |
| `fixtures/insufficient_evidence_ticket.json` | T-ANL-05 |
| `fixtures/full_evidence_ticket.json` | T-ANL-06 |
| `fixtures/blocked_quality_ticket.json` | T-ANL-S01-03 |

所有 fixture 使用 mock SQLite 数据，不依赖真实 ITR。

---

## 8. 安全与权限

| 检查项 | 要求 | 实现位置 |
|--------|------|---------|
| SQLite 只读 | 仅 SELECT，不执行 INSERT/UPDATE/DELETE | SKILL.md 限制声明 + ptm-tse.md 引用说明 |
| 无凭据读取 | 不访问环境变量、配置文件、或 HTTP 端点 | SKILL.md `禁止事项` 节 |
| 无外部访问 | 不发起 HTTP/网络请求 | 同上 |
| 无生产写入 | 不修改 ticket/ticket_version/ingestion_batch 表 | 同上 |
| 脱敏 | eligible 输出不包含原始 ITR 响应体 | SKILL.md 输出格式约束 |

---

## 9. 实施步骤

### TASK-ID 映射

| TASK-ID | 内容 | 输出 |
|---------|------|------|
| TASK-ANL-01 | 创建 `skills/reverse-analysis/` 目录和 SKILL.md 骨架 | 目录 + SKILL.md §0（元数据、触发场景） |
| TASK-ANL-02 | 实现 P1/P2/P3/P4 资格判定逻辑 | SKILL.md §1（资格检查段） |
| TASK-ANL-03 | 实现内部问题识别与拒绝 | SKILL.md §1（deferred 分支） |
| TASK-ANL-04 | 实现五条证据线分类（fact/hypothesis/unknown/gap） | SKILL.md §2（证据分类段） |
| TASK-ANL-05 | 实现三线阈值检查（valid_count >= 3 判定） | SKILL.md §2（阈值判定子段） |
| TASK-ANL-06 | 更新 `agents/ptm-tse.md` 添加 reverse-analysis Skill 引用 | ptm-tse.md 编排流程更新 |

### 执行顺序

```
TASK-ANL-01（骨架）
  └─► TASK-ANL-02（资格判定）
        └─► TASK-ANL-03（内部问题识别）
              └─► TASK-ANL-04（证据分类）
                    └─► TASK-ANL-05（阈值检查）
                          └─► TASK-ANL-06（Agent 引用更新）
```

---

## 10. 回滚策略

| 回滚场景 | 操作 |
|---------|------|
| 资格规则需要修改 | 修改 SKILL.md §1 中的判定矩阵表，不涉及数据迁移 |
| 证据分类规则变更 | 修改 SKILL.md §2 中的分类规则 |
| 需要完全撤销 reverse-analysis | 删除 `skills/reverse-analysis/` 目录；从 ptm-tse.md 移除引用段落 |
| 依赖的 SQLite schema 变更 | ST-RA-INGEST-DB 负责 schema 迁移；本 Story 只修改 SQL 查询适配 |

**重要**：本 Story 不写入 SQLite，因此回滚不涉及数据迁移。

---

## 11. 平台差异检查

| 平台 | 差异项 | 影响 | 处理 |
|------|--------|------|------|
| Claude Code | Skill 发现：`skills/reverse-analysis/SKILL.md` | 无差异 | 标准文件路径 |
| Codex | Skill 发现：`.agents/skills/reverse-analysis/SKILL.md`（安装后） | 安装路径不同 | 安装器负责映射；LLD/Skill 内使用 `skills/reverse-analysis/` canonical 路径 |
| Qoder | Skill 发现：`.qoder/skills/reverse-analysis/SKILL.md`（安装后） | 同上 | 同上 |

**结论**：本 Story 不依赖平台特定能力；资格检查逻辑在所有平台一致。

---

## 12. 与相邻模块的集成契约

### 12.1 上游依赖

| 上游 Story | 输出 | 本 Story 消费方式 |
|-----------|------|-----------------|
| ST-RA-INGEST-DB | `data/schema.sql`（ticket 表结构） | 只读 SELECT，不修改 |

**契约要求**：
- `ticket` 表必须在 `severity` 列中使用 `P1`/`P2`/`P3`/`P4` 枚举
- `ticket` 表必须有 `quality_flag` 列，取值为 `PASS`/`BLOCKED`/`WARNING`
- `ticket` 表必须有 `is_internal` 布尔列
- `source_ticket_id` 非空且稳定

### 12.2 下游消费

| 下游 Story | 消费内容 | 传递方式 |
|-----------|---------|---------|
| ST-RA-02 | `EligibilityResult` + 证据分类结果 | 通过 SKILL.md 中定义的上下文传递（Skill 内部子步骤间共享） |
| ST-NRA-01 | 三线阈值检查结果 | 同 Skill 内调用（ST-RA-01 定义阈值，ST-NRA-01 实现硬阻断） |
| ST-NRA-02 | 权限边界检查 | 同 Skill 内调用 |

### 12.3 SQLite 读取契约

- **只允许 SELECT** 操作
- **禁止** DDL、INSERT、UPDATE、DELETE
- 使用 batch_id 查询时，WHERE 子句固定为 batch_id = :batch_id 且 quality_flag != 'BLOCKED'（仅 eligible 查询时过滤）
- **不读取** ticket 表中的原始 ITR 响应体字段（raw_response），仅读取规范化列

---

## 13. 开放项与假设

| ID | 类型 | 描述 | 状态 | 重访条件 |
|----|------|------|------|---------|
| OPEN-RA01-01 | 假设 | P3/P4 默认拒绝，用户不可手动绕过 | OPEN | CP7 验证时如用户要求 P3 手工启动 |
| OPEN-RA01-02 | 假设 | quality_flag=PASS 是最低资格门槛 | OPEN | 未来可能需要 WARNING 级也可分析（但需标记风险） |
| OPEN-RA01-03 | 假设 | 五条证据线分类足以覆盖 ITR 实际字段 | OPEN | 发现 ITR 有证据字段不在分类中时扩展 |
| OPEN-RA01-04 | 假设 | SQLite 中的 ticket.severity 枚举值与 P1/P2/P3/P4 一致 | 依赖 ST-RA-INGEST-DB | schema 完成后做字段校验 |

---

## 14. LLD 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|------|------|--------|---------|
| 1.0 | 2026-07-16 | meta-dev | 初始 LLD，覆盖资格检查、证据分类、三线阈值和集成契约 |
| 1.1 | 2026-07-16 | host-orchestrator | CP5 B12：迁移为当前 full-lld 证据兼容格式，增加 canonical Story ID 与 §0 工程依据/章节索引；不改变原设计契约。 |
