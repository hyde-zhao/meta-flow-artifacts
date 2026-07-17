---
checkpoint: "CP6"
story_id: "ST-RA-03"
title: "改进输入治理（improvement-tracker CA/PA 侧）"
feature: "FEAT-RA-IMPROVEMENT"
wave: 3
check_date: "2026-07-16"
check_summary: "PASS"
---

# CP6-ST-RA-03: 改进输入治理 — 编码完成检查

## Entry Criteria

- [x] Story `ST-RA-03` LLD（`process/stories/STORY-RA-03-LLD.md`）已确认
- [x] Feature DESIGN（`docs/features/feat-ra-improvement/DESIGN.md`）可消费
- [x] 依赖 `ST-RA-05.3-ANALYZE` 分析报告草案已可用
- [x] `data/dao.py` DAO 接口可读（只读消费）
- [x] 输出文件所有权不冲突

## Checklist

### 实现对象清单

| 文件 | 操作 | 状态 | 行数 |
|------|------|------|------|
| `skills/improvement-tracker/SKILL.md` | 创建 | DONE | 355 |
| `skills/improvement-tracker/templates/capa-proposal.yaml` | 创建 | DONE | 182 |
| `skills/improvement-tracker/templates/approved-input.yaml` | 创建 | DONE | 181 |
| `agents/ptm-tse.md` | 修改（追加 Skill 引用） | DONE | 160→161 |
| `skills/README.md` | 修改（追加 Skill 索引） | DONE | 62→64 |

### 设计契约映射

| LLD 契约 | 实现位置 | 验证方式 |
|----------|----------|----------|
| CA/PA Proposal schema（§5.1） | `templates/capa-proposal.yaml` 完整 schema + 约束汇总 | 必填字段 7 个、枚举值 3 组、终态约束 |
| Approved Input schema（§5.2） | `templates/approved-input.yaml` 完整 schema + 约束汇总 | 必填字段 10 个、不可变标记、consumer_status 枚举 4 值 |
| §3.1 草案生成流程 | `SKILL.md §3.1` — 步骤 1-5 + 失败处理 3 场景 | 前置校验、去重检查、必填字段校验 |
| §3.2 批准状态机 | `SKILL.md §3.2` — 状态图 + 迁移表 + 强制约束 | draft/approved/rejected 三态 + 不可回退 |
| §3.3 Approved Input 生成 | `SKILL.md §3.3` — 门控 + 步骤 1-6 + 失败处理 3 场景 | 门控校验 4 条件、不可变标记 |
| §3.4 消费者映射 | `SKILL.md §3.4` — 映射表 + 约束 | 4 合法 target_agent + blocked 兜底 |
| §8.1 批准状态机（不可回退） | `SKILL.md §3.2` 强制约束 3-4 + `capa-proposal.yaml` | approved/rejected 终态 |
| §8.2 Approved Input 不可变性 | `SKILL.md §3.3` immutable=true + `approved-input.yaml` | 生成后拒绝修改规则 |
| §6.1 Skill 接口（4 模式） | `SKILL.md` 执行模式表 | generate-proposal / review-proposal / publish-input / check-gate |
| §6.4 输入契约（上游依赖） | `SKILL.md §1` 输入契约 | analysis-status=analysis-confirmed gate |
| Gotchas（Feature DESIGN §5） | `SKILL.md` Gotchas 10 条 | 覆盖不直接分发、不自动批准、Input 不可变、拒绝保留等 |

### 测试 / Fixture 覆盖

| 测试场景 ID | 覆盖内容 | 验证位置 |
|------------|----------|----------|
| T-RA-03-SCHEMA-01 | CA/PA Proposal 必填字段完备 | `capa-proposal.yaml` schema 定义 + 示例 1-2 |
| T-RA-03-SCHEMA-02 | CA/PA Proposal 缺少必填字段 | schema 约束汇总 #2（basis/target/owner 等必填） |
| T-RA-03-SCHEMA-03 | Approved Input 必填字段完备 | `approved-input.yaml` schema 定义 + 示例 1-2 |
| T-RA-03-GATE-01 | 未确认 RA 拒绝生成 | `SKILL.md §3.1` 步骤 1 前置校验 |
| T-RA-03-GATE-02 | 已确认 RA 生成草案 | `SKILL.md §3.1` 步骤 3-5 |
| T-RA-03-GATE-03 | 未批准 proposal 拒绝生成 Input | `SKILL.md §3.3` 失败处理 |
| T-RA-03-FSM-01 | reviewer 批准 CA/PA | `SKILL.md §3.2` 状态迁移 draft→approved |
| T-RA-03-FSM-02 | reviewer 拒绝 CA/PA | `SKILL.md §3.2` 状态迁移 draft→rejected |
| T-RA-03-FSM-03 | 不可回退已批准 | `SKILL.md §3.2` 强制约束 #3 |
| T-RA-03-FSM-04 | 不可回退已拒绝 | `SKILL.md §3.2` 强制约束 #3 |
| T-RA-03-IMMUT-01 | Approved Input 不可变 | `SKILL.md §3.3` + `approved-input.yaml` immutable:true |
| T-RA-03-CONS-01 | 合法 target_agent → pending-consumer | `SKILL.md §3.4` 映射表 4 项 |
| T-RA-03-CONS-02 | 非法 target_agent → blocked | `SKILL.md §3.4` "其他" 映射 |
| T-RA-03-CONS-03 | 追溯字段完整 | `approved-input.yaml` source_ra + proposal_id + approval_ref |

### 平台差异

- N/A：本 Story 产出为 Skill 定义文件（Markdown + YAML 模板），不涉及平台特定代码或安装路径。

### 最小实现切片

| 切片 | 内容 | 状态 |
|------|------|------|
| Slice 1: 目录与模板 | `skills/improvement-tracker/` + 两个模板文件 | DONE |
| Slice 2: SKILL.md §3 CA/PA 治理 | 草案生成 + 批准状态机 + Approved Input + 消费者映射 | DONE |
| Slice 3: Agent 集成 | `agents/ptm-tse.md` Skill 引用 + 流程 + 检查点 | DONE |
| Slice 4: 索引更新 | `skills/README.md` Skill 索引 | DONE |

### 验证结果

| 检查项 | 结果 | 说明 |
|--------|------|------|
| 文件存在且非空 | PASS | 3 个新建文件 + 2 个修改文件 |
| SKILL.md frontmatter 完整 | PASS | name, description, argument-hint, user-invokable, status, shared, shared_writers, version, source_cr, source_feature |
| SKILL.md §3 四子节完整 | PASS | §3.1-§3.4 覆盖草案生成、状态机、Input 生成、消费者映射 |
| capa-proposal.yaml 必填字段 | PASS | proposal_id, analysis_ref, kind, title, basis, target, owner, due_date, priority, validation_method, side_effects |
| approved-input.yaml 必填字段 | PASS | input_id, source_ra, proposal_id, kind, title, target_agent, scope, acceptance_criteria, priority, constraints, approval_ref |
| CA/PA 状态机三态 + 终态约束 | PASS | draft→approved/rejected，不可回退 |
| Approved Input 不可变标记 | PASS | immutable: true + 生成后拒绝修改规则 |
| consumer_status 映射表 | PASS | 4 合法 + blocked 兜底 |
| ptm-tse.md skills 列表含 improvement-tracker | PASS | 追加到 skills 列表 |
| ptm-tse.md 改进治理流程 | PASS | 3 步骤流程完整 |
| ptm-tse.md 检查点含改进治理 | PASS | 3 条新 Gate |
| skills/README.md 含 improvement-tracker | PASS | Skill Index 追加 |
| §4/§5 占位符（供后续 Story） | PASS | 明确标注"由 ST-RA-04/ST-RA-06.3-TRACK 追加" |
| shared_writers 列出后续 Story | PASS | ST-RA-04 + ST-RA-06.3-TRACK |
| 不修改 data/ 文件 | PASS | 未修改 data/dao.py |
| 文件名 kebab-case | PASS | 所有文件均符合 |
| Gotchas 至少 10 条 | PASS | 10 条 Gotchas |

## 未覆盖项

- 下游 Agent 就绪检测（首版不实现，LLD §8.3 说明）
- 下游消费回写 consumer_status 逻辑（由 ST-RA-04 负责）
- MeasureBaseline 创建与管理（由 ST-RA-06.3-TRACK 负责）

## 设计缺口反馈

无。LLD 的 14 个章节完整覆盖了实现所需的所有设计契约。

## 后续交接

| 接收方 | 交接内容 | 说明 |
|--------|----------|------|
| meta-qa | CP7 验证 | 验证入口：`skills/improvement-tracker/SKILL.md`（CA/PA 治理）、`templates/capa-proposal.yaml`、`templates/approved-input.yaml` |
| ST-RA-04 | §4 闭环跟踪追加 | improvement-tracker SKILL.md §4 章节（当前占位符） |
| ST-RA-06.3-TRACK | §5 措施基线管理追加 | improvement-tracker SKILL.md §5 章节（当前占位符） |

**风险提示：**
- 三 Story 串行写入 SKILL.md，后续 Story 写入时必须尊重 §3/§4/§5 节边界，不得跨节修改
- 首版 consumer_status 不检测下游就绪，`blocked` 仅因 target_agent 非法触发

## Exit Criteria

- [x] 所有输出文件存在且非空
- [x] SKILL.md 包含完整 §3 CA/PA 治理（草案生成、批准状态机、Approved Input、消费者映射）
- [x] 模板文件含完整 schema、约束汇总和示例
- [x] ptm-tse.md 已追加 improvement-tracker Skill 引用和改进治理流程
- [x] skills/README.md 已追加 Skill 索引
- [x] §4/§5 占位符已明确标注后续 Story 负责
- [x] shared_writers frontmatter 列出后续 Story
- [x] 未修改 data/ 文件
- [x] DEV-LOG.md 已追加

## Agent Dispatch Evidence

| 字段 | 值 |
|------|-----|
| canonical_role | meta-dev |
| dispatch_trigger | Story ST-RA-03 实现 |
| mode | inline（host-orchestrator 直接路由） |
| completed_at | 2026-07-16 |

---

**CP6 结论：PASS** — 所有检查项通过，Story ST-RA-03 进入 ready-for-verification。
