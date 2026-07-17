# PTM Team Feature 设计矩阵

> 版本：v1.0 · 更新：2026-06-08 · 覆盖：全部 6 个 Agent

---

## Feature 总览

| Feature ID | Feature 名称 | Agent | 适用性 | lld_policy | 状态 |
|---|---|---|---|---|---|
| F-001 | 三阶段框架（KYM→MFQ→PPDCS） | ptm-tde | required | full-lld | ✅ delivered |
| F-002 | 场景发现与确认 | ptm-tde | required | full-lld | ✅ delivered |
| F-003 | M 分析（单功能拆分） | ptm-tde | required | full-lld | ✅ delivered |
| F-004 | F 分析（耦合分析） | ptm-tde | required | full-lld | ✅ delivered |
| F-005 | Q 分析（质量属性） | ptm-tde | required | full-lld | ✅ delivered |
| F-006 | PPDCS 五方法设计 | ptm-tde | required | full-lld | ✅ delivered |
| F-007 | 覆盖率验证与交付渲染 | ptm-tde | required | full-lld | ✅ delivered |
| F-008 | AskUserQuestion 交互 | ptm-tde | required | technical-note | ✅ delivered |
| F-009 | 原子操作集成 | ptm-tde | required | full-lld | ✅ delivered |
| F-010 | 因子库发现 | ptm-tde | required | full-lld | ✅ delivered |
| F-011 | 测试用例执行 | ptm-te | required | full-lld | ✅ delivered（CR-024） |
| F-012 | Topo 映射与环境管理 | ptm-te | required | full-lld | ✅ delivered（CR-024） |
| F-013 | 原子能力框架 | ptm-tae | required | full-lld | 🔄 in-progress |
| F-014 | 自动化翻译与回归 | ptm-tae | required | full-lld | ⬜ planned |
| F-015 | 公共 Skill 基础设施 | ptm-tae | required | full-lld | 🔄 in-progress |
| F-016 | 测试计划与调度 | ptm-tm | required | full-lld | ⬜ planned |
| F-017 | 需求分析与策略制定 | ptm-tse | required | full-lld | ⬜ planned |
| F-018 | 用例/执行/工具评审 | ptm-tse | required | full-lld | ⬜ planned |
| F-019 | 质量审计与度量 | ptm-qa | required | full-lld | ⬜ planned |
| F-020 | 问题单摄取与数据治理 | ptm-tse | required | full-lld | 🔄 in-progress（CR-030） |
| F-021 | 逆向问题分析 | ptm-tse | required | full-lld | 🔄 in-progress（CR-030） |
| F-022 | 改进输入治理 | ptm-tse | required | full-lld | 🔄 in-progress（CR-030） |
| F-023 | 闭环跟踪与有效性 | ptm-tse | required | full-lld | 🔄 in-progress（CR-030） |

---

## LLD 策略说明

| 级别 | 适用条件 | Agent 分配 |
|---|---|---|
| `full-lld` | 跨模块契约、数据模型、权限、安全、外部接口 | F-001~007, F-009~019 |
| `technical-note` | 低复杂度、纯适配、Story 内技术说明足够 | F-008 |
| `waived` | 明确不需要独立设计文档 | 无 |

---

## Feature 设计文档索引

| Feature ID | DESIGN | TEST-PLAN | TASKS | 状态 |
|---|---|---|---|---|
| F-001~010 | CR 级 LLD（约 25 个 STORY-*-LLD.md） | CP5 checks | CP6 checks | ✅ delivered |
| F-011~012 | STORY-024-01/02/03（full-lld）+ STORY-024-04（technical-note） | process/checkpoints/CP5-ALL-STORIES-LLD-BATCH-CR-024.md | process/checks/CP6-CR-024-CODING-DONE.md | ✅ delivered（CR-024） |
| F-013~015 | — | — | — | 🔄 in-progress |
| F-016 | — | — | — | ⬜ planned |
| F-017~018 | — | — | — | ⬜ planned |
| F-019 | — | — | — | ⬜ planned |
| F-020~023 | — | — | — | 🔄 in-progress（CR-030） |

---

## 统计

| Agent | Features | delivered | in-progress | planned |
|---|---|---|---|---|
| ptm-tde | 10 | 10 | 0 | 0 |
| ptm-te | 2 | 2 | 0 | 0 |
| ptm-tae | 3 | 0 | 2 | 1 |
| ptm-tm | 1 | 0 | 0 | 1 |
| ptm-tse | 2 | 0 | 0 | 2 |
| ptm-tse（CR-030） | 4 | 0 | 4 | 0 |
| ptm-qa | 1 | 0 | 0 | 1 |
| **合计** | **19** | **12** | **2** | **5** |
| **CR-030 合计** | **4** | **0** | **4** | **0** |

---

*本矩阵随 Agent 开发进展动态更新。*

---

# CR-030: ptm-tse 现网问题逆向分析能力 — Feature 设计矩阵

> 版本：v1.0 · 更新：2026-07-16 · 来源 CR：CR-030 · 来源 HLD：`docs/design/HLD.md` (v1.2, REV-03)
>
> 基于 CP3 已批准的四边界模型，将 STORY-MAP v1.3 的 9 个 Story 映射为 Feature/Epic，并为每个 Story 分配 `feature_design_refs` 和 `lld_policy`。

## CR-030 Feature 总览

HLD REV-03 定义了四个边界，对应四个 Feature。相比 BLUEPRINT v1.0（三个 Feature），新增 `FEAT-RA-INGESTION`。

| Feature ID | 名称 | 对应 HLD 边界 | Agent | 适用性 | lld_policy | 状态 |
|---|---|---|---|---|---|---|
| F-020 | 问题单摄取与数据治理 | `itr-ticket-ingestion` Skill + SQLite Data Owner | ptm-tse | required | full-lld | 🔄 in-progress |
| F-021 | 逆向问题分析 | `reverse-analysis` Skill | ptm-tse | required | full-lld | 🔄 in-progress |
| F-022 | 改进输入治理 | `improvement-tracker` Skill（CA/PA 侧） | ptm-tse | required | full-lld | 🔄 in-progress |
| F-023 | 闭环跟踪与有效性 | `improvement-tracker` Skill（跟踪侧）+ 人工 reviewer | ptm-tse | required | full-lld | 🔄 in-progress |

### Feature 边界与职责

| Feature | 负责事项 | 不负责事项 | 拥有数据 | 只读数据 | 禁止依赖 |
|---|---|---|---|---|---|
| F-020 | ITR HTTP GET 受控拉取、原始快照保存、字段映射、清洗、质量标记、变更检测、合并、SQLite schema（ticket/ticket_version/ingestion_batch/change_history） | 分析结论、CA/PA、措施状态、下游分发 | `ingestion_batch`、`IngestionQualityReport`、`ticket`、`ticket_version`、`change_history` | 无 | 凭据读取、ITR 写入、其他外部系统、Git/process 写入 |
| F-021 | 资格检查、证据目录、六维分析、逐单/批量报告草案、事实/假设分离、analysis_run 管理 | 生产采集、ITR 拉取、CA/PA 批准、行动项状态、关闭决策 | `analysis_run`、`RA Report` 草案及确认字段、`MetricDefinition` | `ticket`、`ticket_version`、`ingestion_batch`（皆只读） | 直接写 ticket 表、外部系统连接、自动确认根因 |
| F-022 | CA/PA 草案生成、批准状态管理、已批准改进输入创建、消费者映射、MeasureBaseline 管理 | 自动分发、下游任务创建、行动项跟踪、关闭决策 | `Approved Improvement Input`、`CA/PA Proposal`、`MeasureBaseline` | `RA Report`（只读已确认部分）、`analysis_run`（只读） | 未批准分发、直接修改下游 Agent 产物 |
| F-023 | 行动项状态、有效性检查、观察窗、复发指标、关闭决策、措施刷新提示 | 替代 Owner 完成行动、自动关闭、外部通知 | `Action Item`、`Effectiveness Check`、`Closure Decision`、`measure_link` | `Approved Improvement Input`、`RA Report`（皆只读） | 修改根因结论、自动关闭、通知渠道 |

> **术语约定（CP5 Round 3）**：上表"拥有数据"指**逻辑领域 ownership**（业务语义归属）。**物理 DAO 写入**统一由 F-020 的 `data/dao.py` 承载——F-020 在 Wave 1 一次性实现全部 6 张表的 CRUD 方法（含 `analysis_run` 和 `measure_link`），F-021/F-023 只调用 DAO 公共接口，不直接写 SQL、不修改 `data/dao.py`。详见 ST-RA-INGEST-DB LLD §12.3。

### 共享能力与依赖方向

```
F-020 (INGESTION) ──(raw data)──► SQLite Data Owner
                                        │
                            ┌───────────┼───────────┐
                            ▼           ▼           ▼
                     F-021 (ANALYSIS)  │   F-023 (TRACKING)
                            │           │           ▲
                            ▼           ▼           │
                     F-022 (IMPROVEMENT) ───────────┘
```

## CR-030 Story 清单与 Feature 归属

STORY-MAP v1.3 有 9 个用户任务级 Story。其中 ST-RA-05 和 ST-RA-06 跨越多个 Skill 边界，已按四边界模型分解为实现级 Story（共 14 个）。

| Story ID | 标题 | 来源 Story | Feature | 优先级 | lld_policy |
|---|---|---|---|---|---|
| **Wave 1: INGESTION 基础** | | | | | |
| ST-RA-05.1-INGEST | ITR 问题单受控摄取与原始快照保存 | ST-RA-05 | F-020 | P0 | full-lld |
| ST-RA-05.2-CLEAN | 字段映射、清洗与质量报告 | ST-RA-05 | F-020 | P0 | full-lld |
| ST-RA-INGEST-DB | SQLite 数据库 schema 与 DAO 层 | ST-RA-05 | F-020 | P0 | full-lld |
| ST-NRA-03 | 摄取失败保护 | ST-NRA-03 | F-020 | P0 | technical-note |
| **Wave 2: S1 分析引擎 + S2 变更检测** | | | | | |
| ST-RA-01 | 资格检查与可信输入建立 | ST-RA-01 | F-021 | P0 | full-lld |
| ST-RA-02 | 六维分析引擎 | ST-RA-02 | F-021 | P0 | full-lld |
| ST-RA-05.3-ANALYZE | S1 逐单与批量分析管线 | ST-RA-05 | F-021 | P0 | full-lld |
| ST-NRA-01 | 证据不足保护 | ST-NRA-01 | F-021 | P0 | technical-note |
| ST-NRA-02 | 权限边界拒绝 | ST-NRA-02 | F-021 | P0 | technical-note |
| ST-RA-06.1-DETECT | 变更检测、合并与版本历史 | ST-RA-06 | F-020 | P0 | full-lld |
| **Wave 3: 改进治理 + 闭环跟踪 + S2 增量重算** | | | | | |
| ST-RA-03 | 改进输入治理 | ST-RA-03 | F-022 | P0 | full-lld |
| ST-RA-04 | 闭环跟踪与有效性决策 | ST-RA-04 | F-023 | P1 | full-lld |
| ST-RA-06.2-REFRESH | S2 增量重算、差异报告 | ST-RA-06 | F-021 | P0 | full-lld |
| **Wave 4: S2 措施基线管理** | | | | | |
| ST-RA-06.3-TRACK | 措施基线管理与刷新提示 | ST-RA-06 | F-023 | P1 | technical-note |

### 原始 Story Map 追溯

| 原始 Story ID | 分解为实现 Story | 分解原因 |
|---|---|---|
| ST-RA-01 | ST-RA-01 | 单一 Feature 内 |
| ST-RA-02 | ST-RA-02 | 单一 Feature 内 |
| ST-RA-03 | ST-RA-03 | 单一 Feature 内 |
| ST-RA-04 | ST-RA-04 | 单一 Feature 内 |
| ST-RA-05 | ST-RA-05.1-INGEST + ST-RA-05.2-CLEAN + ST-RA-INGEST-DB + ST-RA-05.3-ANALYZE | 跨越 F-020(3) + F-021(1) |
| ST-RA-06 | ST-RA-06.1-DETECT + ST-RA-06.2-REFRESH + ST-RA-06.3-TRACK | 跨越 F-020(1) + F-021(1) + F-023(1) |
| ST-NRA-01 | ST-NRA-01 | 单一 Feature 内 |
| ST-NRA-02 | ST-NRA-02 | 单一 Feature 内 |
| ST-NRA-03 | ST-NRA-03 | 单一 Feature 内 |

## CR-030 Feature 级设计触发判定

| Feature ID | 是否需要 Feature 设计 | 触发原因 | 产物路径 |
|---|---|---|---|
| F-020 | **required** | data-model, cross-module-contract, external-interface, migration, shared-story-boundary | `docs/features/feat-ra-ingestion/DESIGN.md`、`TEST-PLAN.md`、`TASKS.md` |
| F-021 | **required** | data-model, cross-module-contract, security, permission, shared-story-boundary | `docs/features/feat-ra-analysis/DESIGN.md`、`TEST-PLAN.md`、`TASKS.md` |
| F-022 | **required** | cross-module-contract, security, permission, shared-story-boundary | `docs/features/feat-ra-improvement/DESIGN.md`、`TEST-PLAN.md`、`TASKS.md` |
| F-023 | **required** | cross-module-contract, shared-story-boundary | `docs/features/feat-ra-tracking/DESIGN.md`、`TEST-PLAN.md`、`TASKS.md` |

## CR-030 文件所有权

| 文件 | 写入 Feature | 读取 Feature | 备注 |
|---|---|---|---|
| `skills/itr-ticket-ingestion/SKILL.md` | F-020 | — | 唯一 owner |
| `skills/itr-ticket-ingestion/templates/` | F-020 | F-021（只读 quality report schema） | 模板目录 |
| `skills/reverse-analysis/SKILL.md` | F-021 | — | 唯一 owner |
| `skills/reverse-analysis/templates/` | F-021 | F-022（只读 report schema） | 模板目录 |
| `skills/improvement-tracker/SKILL.md` | F-022 + F-023 | — | 共享 Skill，F-022 先写 CA/PA 部分，F-023 追加跟踪部分 |
| `skills/improvement-tracker/templates/` | F-022 + F-023 | — | 模板目录 |
| `agents/ptm-tse.md` | F-021（编排更新） | — | 更新已有 Agent 定义 |
| `data/schema.sql` | F-020 | F-021, F-022, F-023（只读） | SQLite DDL |
| `data/dao.py` | F-020 | F-021、F-023（仅调用已授权公共接口），F-022（只读） | F-020 是唯一物理写入者；消费 Feature 不直接 SQL、不修改 DAO 文件 |
| `docs/ptm-tse/` | 各 Feature 按职责写入 | — | 文档目录 |

### 关键决策项

| Decision ID | 决策类型 | 问题 | 推荐方案 | 备选方案 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|
| CP4-DQ-CR030-01 | architecture | ST-RA-05/06 是否按 Skill 边界分解？ | 是，分解为 7 个子 Story | 保持原 2 个大 Story | 单写规则清晰、并行度高 vs Story 卡片少但耦合高 | CP5 证实无独立测试价值时可合并 |
| CP4-DQ-CR030-02 | architecture | improvement-tracker 是否由两个 Feature 共享？ | 是，共享同一 SKILL.md，分节写入 | 拆为两个独立 Skill | 减少 Skill 数但需串行写入 vs 更清晰所有权但增加注册复杂度 | 若耦合过弱可拆分 |
| CP4-DQ-CR030-03 | implementation | SQLite 数据库文件存放路径？ | `data/ptm-tse.db`，不进 Git/process | 用户指定路径 | 固定路径便于 CI fixture vs 灵活但需额外配置 | 用户指定其他路径时改配置 |
| CP4-DQ-CR030-04 | risk_acceptance | 负向 Story 使用 technical-note 的风险？ | 接受，正向 Story LLD 已覆盖 | 全部升级为 full-lld | 减少 3 个 LLD vs 更完整但增加工作量 | CP7 发现安全/边界缺陷时升级 |
