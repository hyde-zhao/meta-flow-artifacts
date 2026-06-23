---
status: draft
version: "1.2"
complexity: "standard"
confirmed: false
confirmed_by: ""
confirmed_at: ""
change_id: "CR-013"
workflow_id: "WF-PTM-TEAM-20260520-001"
depends_on: "HLD-CR-011（PPDCS 架构基线）"
parent_hld: "process/HLD-CR-011.md"
---

# 高层设计（HLD）：CR-013 ptm-tde PPDCS 阶段改造

> 基于 `process/changes/CR-013-ptm-tde-ppdcs-phase.md`（已批准）和 HLD-CR-011 §8-§11（PPDCS 前瞻设计）输出。
> PPDCS 阶段架构已在 HLD-CR-011 中定义为"设计前瞻 ◇"，本 HLD 将其落地为可实施的设计。

---

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|------|------|--------|----------|
| 1.0 | 2026-06-03 | meta-po | 初始 HLD，覆盖问题定义、架构方案、模块职责、数据流、路径迁移映射、风险与 ADR、Story 拆解计划 |
| 1.1 | 2026-06-03 | meta-po | CP3 HLD 灰区讨论结论：灰区 1—保持原范围，新建 CR-014 跟踪 change-impact-analyzer/bug-gap-analyzer；灰区 2—STORY-013-03 增加创建 `ppdcs/delivery/` 目录；灰区 3—删除旧目录清理工作项 |
| 1.2 | 2026-06-03 | meta-po | 新增 §3.4 PPDCS 核心方法论基线：显式声明 CR-013 不重新设计五特征建模方法和 PC 生成方法，引用已有实现位置（5 设计 Skill 执行流程、data-flow-spec §实体 7、ppdcs-analysis-step-by-step.md）和方法论改动边界 |

---

## 1. 问题定义

### 问题陈述

CR-010/011/012 已完成三阶段框架建立、KYM 阶段和 MFQ 阶段的内容层改造。PPDCS 阶段（第三阶段）仍存在以下问题：

1. **路径未对齐新框架**：8 个 PPDCS 阶段 Skill 文件仍引用旧路径（`analysis/`、`design/`），与 CR-010 建立的 `kym/`/`mfq/`/`ppdcs/`/`process/plan/` 目录体系不一致。共计约 114 处旧路径引用。
2. **PPDCS Exit Gate 仅为骨架**：GATE-4 当前仅有 3 项 Entry Criteria + 7 项骨架 Checklist，缺少 PPDCS 阶段专属的设计过程检查（LC 建模完整性）、PC 生成质量检查和覆盖率验证检查。
3. **5 个设计 Skill 缺少方法论内容**：process-design、parameter-design、data-design、combination-design、state-design 的方法论节为空白占位，需补充方法论框架。
4. **交付物目录缺失**：`ppdcs/delivery/` 目录不存在，deliverable-renderer 产出无写入目标。

### 核心价值

- **完成三阶段路径迁移最后一环**：KYM（CR-011）和 MFQ（CR-012）路径迁移已完成后，PPDCS 路径迁移使全部 18 个 Skill 的路径与新框架对齐。
- **PPDCS Exit Gate 具备完整检查能力**：从骨架升级为 7 项自检项 + 4 项人工确认，覆盖设计过程、PC 质量、拓扑回链、覆盖率等关键维度。
- **设计 Skill 方法论可扩展**：5 个 PPDCS 设计 Skill 增加方法论占位框架，用户可根据项目特点补充具体规则。

### 目标

| 维度 | 目标 |
|------|------|
| 路径迁移 | 8 个 PPDCS Skill 的 114+ 处旧路径引用全部更新为新路径 |
| GATE-4 增强 | gate-spec.md + checkpoint-manager 增加 P1-P7 自检项 + 4 项人工确认 |
| 方法论充实 | 5 个设计 Skill 各增加 ~15 行方法论占位节 |
| 交付物目录 | `ppdcs/delivery/` 目录创建，供 deliverable-renderer 写入 |

---

## 2. 候选方案对比

### 方案 A（推荐）：分波次渐进式迁移

按 Skill 间依赖关系分 3 个 Wave 迁移：

**Wave A**：无上游依赖的独立 Skill（5 个设计 Skill + coverage-verifier）
**Wave B**：依赖 Wave A 产出的协调 Skill（design-ppdcs-analyzer + deliverable-renderer）
**Wave C**：跨 Skill 的共享基础设施（GATE-4 增强 + checkpoint-manager + 旧目录验证）

| 维度 | 评价 |
|------|------|
| **优点** | 依赖清晰，每波可独立验证；Wave A 完成后即可并行推进 B+C |
| **缺点** | 3 波需要 2-3 轮用户确认 |
| **风险** | 低，纯路径替换 + 内容追加 |

### 方案 B：全量一次性迁移

一次性修改全部 10 个文件（8 Skill + gate-spec.md + checkpoint-manager）。

| 维度 | 评价 |
|------|------|
| **优点** | 一次提交完成，流程最短 |
| **缺点** | 无法利用并行实施；验证面大（~120 处修改）；出错后回退面大 |
| **风险** | 中，一次性变更量大难以定位问题 |

### 推荐方案 A

分波次渐进式迁移，依赖关系清晰，每波独立可验证，降低回归风险。

---

## 3. 架构设计

### 3.1 PPDCS 阶段在框架中的位置

```
MFQ Phase (CR-012 已完成) → GATE-3 (MFQ Exit Gate)
                                    ↓
PPDCS Phase (CR-013 范围) ──────────────────────────────────────┐
  Sub-step 3.1: design-ppdcs-analyzer + 5 Design Skills          │
    ├── design-ppdcs-analyzer: 读取 plan → 匹配 LC → 调用 Skill    │
    ├── process-design: P-Process 流程图法                         │
    ├── parameter-design: P-Parameter 判定表法                     │
    ├── data-design: D-Data 等价类+边界值法                        │
    ├── combination-design: C-Combination 组合法                   │
    └── state-design: S-State 状态图法                             │
  Sub-step 3.2: PC 生成（5 设计 Skill 产出 → 16 列物理用例表）      │
  Sub-step 3.3: coverage-verifier（双层覆盖验证）                   │
  Sub-step 3.4: deliverable-renderer（交付物生成）                  │
                                                                    ↓
GATE-4 (PPDCS Exit Gate) → 执行层 (ptm-te/ptm-tae)
```

### 3.2 路径迁移映射

| 旧路径 | 新路径 | 说明 |
|--------|--------|------|
| `analysis/integration/` | `mfq/integration/` | 跨阶段读取 MFQ 产出（LC、TD） |
| `analysis/plan/` | `process/plan/` | 跨阶段读取设计计划 |
| `analysis/m-analysis/` | `mfq/m-analysis/` | 跨阶段读取 M 分析产出（PPDCS 标注、TSP） |
| `analysis/scenarios/` | `kym/scenarios/` | 跨阶段读取 KYM 产出（confirmed-scenarios） |
| `analysis/factor-usage/` | `mfq/factor-usage/` | 跨阶段读取因子绑定记录 |
| `analysis/coverage/` | `ppdcs/coverage/` | PPDCS 阶段内部（覆盖率报告） |
| `design/ppdcs/` | `ppdcs/ppdcs/` | PPDCS 阶段内部（设计过程文件） |
| `design/pc/` | `ppdcs/pc/` | PPDCS 阶段内部（物理用例文件） |
| `delivery/` | `ppdcs/delivery/` | PPDCS 阶段内部（交付物） |

### 3.3 GATE-4 增强设计

```
当前 GATE-4（骨架）              增强后 GATE-4（CR-013）
─────────────────────           ───────────────────────
Entry Criteria (3 项)           Entry Criteria (3 项，不变)
Checklist (7 项骨架)            Checklist (7 项，内容充实)
  #1 设计完成 → 仅目录存在检查     P1: ppdcs/ppdcs/ 每个 LC 都有设计过程文件
  #2 PC 完成 → 仅目录存在检查      P2: ppdcs/pc/ 每个 LC 都有 16 列 PC 文件
  #3 拓扑回链 → 占位               P3: PC 端口回链至 topology_bindings → confirmed-scenarios.md
  #4 覆盖率 → 占位                 P4: 双层覆盖（需求 100% + 测试点 ≥ 95%）
  #5 拓扑状态 → 占位               P5: needs-confirmation 不静默提升
  #6 交付物预检 → 占位             P6: ppdcs/delivery/ 含测试方案 + 测试用例
  #7 计划完整性 → 占位             P7: 交付物保留 topology_bindings/topology_role/source/fact_status
Manual (3 项)                   Manual (4 项，增加 PC 质量确认)
```

### 3.4 PPDCS 核心方法论基线（CR-013 不重新设计）

CR-013 **不重新设计** PPDCS 五特征建模方法和 PC 生成方法。以下方法论已在现有 Skill 文件和相关文档中完整定义，CR-013 只做路径对齐和占位扩充。

#### 3.4.1 五特征建模方法（已有实现）

| PPDCS 特征 | 建模技术 | 实现位置 | 核心内容 |
|-----------|---------|---------|------|
| P-Process | 流程图/活动图 | `skills/process-design/SKILL.md` §执行流程 | 6 步：锁定 LC→建立流程节点模型→路径枚举表→覆盖策略→数据叠加→PC 生成 |
| P-Parameter | 判定表/因果图/决策树 | `skills/parameter-design/SKILL.md` §执行流程 | 5 步：factor catalog→规则抽取→判定结构建模→data row 分配→PC 输出 |
| D-Data | 等价类+边界值 | `skills/data-design/SKILL.md` §执行流程 | 5 步：factor catalog→等价类/边界值识别→独立性检查→data row 分配→PC 输出 |
| C-Combination | Pairwise/正交/全组合 | `skills/combination-design/SKILL.md` §执行流程 | 5+1 步：factor catalog→约束建模→压缩策略+PICT 检测→组合生成→data row 分配→PC 输出 |
| S-State | 状态图（Moore Machine） | `skills/state-design/SKILL.md` §执行流程 | 6 步：锁定 LC→状态/迁移模型→迁移表→守卫条件+数据叠加→覆盖策略（Chow's 0/1-switch）→PC 生成 |

> 以上执行流程在 CR-012 中已通过 STORY-012-06（上下游 Skill 适配）完成与 MFQ v3.0 的对齐。CR-013 不改动这些流程，仅更新其中的路径引用和追加方法论占位节。

#### 3.4.2 PC 生成方法（已有实现）

PC 生成规则在 `agents/ptm-tde.md` §物理用例字段规范 和 `docs/ptm-tde/data-flow-spec.md` §实体 7 中定义：

```
PC = LC × Model规则 × 因子具体值

P-Parameter:  每条规则 × data_row → PC
D-Data:       有效值合并 → 1 PC；边界/无效值逐点 → N PC
C-Combination: PICT Pairwise 每行 → 1 PC
P-Process:    MF + 每条 AF 路径 → PC
S-State:      Chow's 1-switch 事件对 → PC
```

PC 输出格式：16 列 Markdown 表格（用例名称*、用例级别*、组网描述*、测试步骤*、预期结果*、是否自动化* 等必填字段）。

#### 3.4.3 综合方法论参考

| 参考文献 | 路径 | 内容 |
|---------|------|------|
| PPDCS 分析详解 | `ppdcs-analysis-step-by-step.md`（外部 llm-wiki） | PPDCS 五特征定义、TEST Heuristics、五方法完整步骤（§4-§10）、PC 生成规则（§11）、覆盖率验证（§12） |
| 海盗派原书 | `raw/testing/海盗派测试分析_MFQPPDCS.md` | PPDCS 五特征原始定义、建模原则、覆盖策略 |
| 数据流规格 | `docs/ptm-tde/data-flow-spec.md` | 8 数据实体生产→消费链路、PC 字段规范、拓扑回链契约 |
| 设计计划器 | `skills/design-planner/SKILL.md` | CAE→PPDCS 推断规则（A/C/E 三维 12 条）、设计计划表格式 |

#### 3.4.4 CR-013 方法论改动的边界

```
已有（不变）                    CR-013 追加（新增）
─────────────                  ─────────────────
5 设计 Skill 执行流程          → 不修改
5 设计 Skill 输入/输出契约      → 路径更新
5 设计 Skill 验收标准          → 不修改
PC 生成规则和 16 列格式        → 不修改
GATE-4 骨架检查项              → 替换为 P1-P7 完整自检
—                              → 方法论占位节（~15 行/Skill，可扩展空框架）
```

### 3.5 方法论占位设计

每个 PPDCS 设计 Skill 新增统一格式的方法论占位节，追加在现有执行流程之后、验收标准之前。占位节**不替代**已有执行流程，仅提供用户可扩展的领域知识框架。

```markdown
## 方法论细则（用户可定制）

> 以下为设计方法的指导框架。用户可根据项目特点和领域知识补充具体规则。
> 详细的 PPDCS 方法论参见 `ppdcs-analysis-step-by-step.md`。

### [方法名称] 设计步骤

**目标**：[方法解决什么设计问题]

**核心步骤**：
1. [步骤 1]
2. [步骤 2]
...

**关键决策点**：
- [决策 1]
- [决策 2]

**示例**（防火墙领域）：
[具体示例]

**下游影响**：
[设计产出如何影响 PC 生成和覆盖率验证]
```

各 Skill 的具体占位维度见 CR-013 draft §二。

---

## 4. 模块职责与修改范围

### 4.1 CR-013 范围内模块

| 模块名称 | 类型 | 职责 | 改动类型 | 改动量 |
|---------|------|------|---------|:---:|
| design-ppdcs-analyzer | Skill（修改） | PPDCS 协调器，匹配 LC 特征→设计 Skill，拓扑绑定校验 | 路径迁移（~19 处） | ~19 |
| process-design | Skill（修改） | P-Process 流程图法 | 路径迁移（~12 处）+ 方法论占位（~15 行） | ~27 |
| parameter-design | Skill（修改） | P-Parameter 判定表法 | 路径迁移（~11 处）+ 方法论占位（~15 行） | ~26 |
| data-design | Skill（修改） | D-Data 等价类+边界值法 | 路径迁移（~10 处）+ 方法论占位（~15 行） | ~25 |
| combination-design | Skill（修改） | C-Combination 组合法 | 路径迁移（~10 处）+ 方法论占位（~15 行） | ~25 |
| state-design | Skill（修改） | S-State 状态图法 | 路径迁移（~13 处）+ 方法论占位（~15 行） | ~28 |
| coverage-verifier | Skill（修改） | 双层覆盖验证 | 路径迁移（~23 处） | ~23 |
| deliverable-renderer | Skill（修改） | 交付物生成 | 路径迁移（~16 处） | ~16 |
| gate-spec.md | 文档（修改） | GATE-4 检查项定义 | 内容充实（P1-P7 + 人工确认项） | ~30 |
| checkpoint-manager | Skill（修改） | GATE-4 描述与 gate-spec 对齐 | 内容更新（~7 处路径 + GATE-4 检查项） | ~20 |

### 4.2 模块边界规则

- **路径迁移仅限 PPDCS 阶段**：`analysis/m-analysis/`、`analysis/f-analysis/` 等 MFQ 阶段路径已由 CR-012 迁移完成，CR-013 不重复处理。
- **5 个设计 Skill 独立并行**：互不依赖，各自独立处理对应 PPDCS 特征的 LC。
- **design-ppdcs-analyzer 是协调器**：它消费 design-plan.md 并为每个 LC 调用对应设计 Skill，路径迁移后需确保引用一致。
- **coverage-verifier 是跨阶段消费者**：读 `kym/scenarios/` + `mfq/integration/` + `ppdcs/ppdcs/` + `ppdcs/pc/` + `ppdcs/coverage/`，路径迁移影响面最大（~23 处）。
- **GATE-4 增强不改变现有 Gate 框架**：CR-013 沿用 CR-010 建立的 5 Gate 体系，仅将 GATE-4 从骨架充实为完整检查。

---

## 5. 数据流与信息消费

### 5.1 PPDCS 阶段数据流

```
输入（消费）                          输出（生产）
─────────                            ──────────
mfq/integration/logic-cases.md ─┐
mfq/integration/test-data.md ───┤
mfq/integration/design-plan.md ─┤
process/plan/design-plan.md ────┤
process/plan/design-planner- ───┤
  reasoning.md                  │
mfq/m-analysis/ppdcs-annotation.md ┤
kym/scenarios/confirmed- ───────┤
  scenarios.md                  │
mfq/factor-usage/factor- ───────┤
  bindings.md                   │
resource/factor-libraries/ ─────┤
                                ├──→ PPDCS → ppdcs/ppdcs/<LC>.md（过程文件）
                                │    阶段    ppdcs/pc/<LC>.md（物理用例）
                                │            ppdcs/coverage/coverage-report.md
                                │            ppdcs/delivery/<特性>测试方案.md
                                │            ppdcs/delivery/<特性>测试用例.md
                                │
                                │          （供下游消费）
                                │          └──→ ptm-te/ptm-tae 执行层
```

### 5.2 关键消费路径

| 消费者 | 来源 | 消费字段 |
|--------|------|---------|
| design-ppdcs-analyzer | `process/plan/design-plan.md` | LC→PPDCS 特征映射 |
| design-ppdcs-analyzer | `process/plan/design-planner-reasoning.md` | 推断明细 |
| 5 设计 Skill | `mfq/integration/logic-cases.md` | 因子-取值表、动作路径、拓扑绑定 |
| 5 设计 Skill | `mfq/integration/test-data.md` | 因子值域、数据叠加 |
| 5 设计 Skill | `kym/scenarios/confirmed-scenarios.md` | 拓扑回链验证 |
| coverage-verifier | `ppdcs/pc/*.md` | PC 逐行扫描 |
| coverage-verifier | `mfq/integration/logic-cases.md` | SR→LC→PC 覆盖链 |
| deliverable-renderer | `ppdcs/pc/*.md` + `ppdcs/coverage/*.md` | 汇总交付 |

---

## 6. 风险与 ADR

### 6.1 风险

| 风险 | 可能性 | 影响 | 缓解措施 |
|------|:---:|:---:|---------|
| 路径替换不完整（遗漏旧引用） | 中 | 低 — 下游消费时路径不存在 | 每波完成后运行 `grep -rn "analysis/\|design/" skills/<target>/` 验证 |
| 路径替换误伤（改错路径） | 低 | 中 — 可能破坏正确的引用 | 逐 Skill diff 审查；保留路径替换前后对比 |
| GATE-4 增强与现有 gate-spec 冲突 | 低 | 中 — gate-spec 定位混乱 | 增强仅追加 CR-013 专属检查项，不修改现有 GATE-1~GATE-3 |
| 方法论占位与 Skill 原有执行流程冲突 | 低 | 低 — 占位节为独立章节 | 占位节放在原章节之后、Gotchas 之前 |

### 6.2 架构决策记录（ADR）

| ADR ID | 决策 | 理由 |
|--------|------|------|
| ADR-013-01 | 采用分波次渐进式迁移（方案 A） | 依赖关系清晰、每波独立可验证、支持并行实施 |
| ADR-013-02 | 路径迁移使用 grep+sed 精确替换，不人工逐行对比 | 114+ 处引用人工对比容易出错，脚本替换更一致 |
| ADR-013-03 | 方法论占位使用统一模板格式 | 保证 5 个设计 Skill 的一致性，降低用户认知负担 |
| ADR-013-04 | GATE-4 增强沿用现有 gate-spec.md 的 GATE-4 位置 | 不引入新的 Gate 编号，保持 5 Gate 体系不变 |
| ADR-013-05 | `ppdcs/delivery/` 目录在 STORY-013-03 中创建 | 交付物渲染器必须有写入目标目录 |
| ADR-013-06 | change-impact-analyzer 和 bug-gap-analyzer 的 13 处旧路径引用不纳入 CR-013 | 范围边界：CR-013 聚焦 PPDCS 阶段 8 Skill，这两个 Skill 属于变更管理和缺陷分析，创建 CR-014 跟踪 |
| ADR-013-07 | 不执行旧目录清理 | `analysis/` 和 `design/` 物理目录在 ptm-team 仓库中不存在，无需清理 |

---

## 7. 分阶段落地

### 7.1 Story 拆解总览

| Story ID | 名称 | 涉及文件数 | 改动量 | tier | Wave |
|----------|------|:---:|:---:|:---:|:---:|
| STORY-013-01 | 5 设计 Skill 路径迁移 | 5 | ~55 处路径 + ~75 行方法论 | M | A |
| STORY-013-02 | coverage-verifier 路径迁移 | 1 | ~23 处路径 | S | A |
| STORY-013-03 | design-ppdcs-analyzer + deliverable-renderer 路径迁移 | 2 | ~35 处路径 | M | B |
| STORY-013-04 | GATE-4 增强（gate-spec.md + checkpoint-manager） | 2 | ~50 行内容更新 | M | B |

### 7.2 Wave 依赖关系

```
Wave A（独立并行）
  STORY-013-01: 5 设计 Skill（路径 + 方法论）
  STORY-013-02: coverage-verifier
         │
         ▼
Wave B（依赖 A：需确认 A 路径已迁移后协调器才能对齐）
  STORY-013-03: design-ppdcs-analyzer + deliverable-renderer
  STORY-013-04: GATE-4 增强（gate-spec.md + checkpoint-manager）
```

**依赖类型**：
- 013-03 → 013-01/013-02：`runtime` 依赖 — 协调器和交付渲染器引用的路径需 Wave A 先完成迁移
- 013-04 独立 — gate-spec.md 和 checkpoint-manager 不依赖其他 Skill 产出

### 7.3 实施策略

- Wave A：2 Story 可并行实施（max_parallel_dev=2）
- Wave B：2 Story 可并行实施（max_parallel_dev=2）
- 总计：2 Wave，4 Stories，~240 处修改
- 预计总实施时间：可在单会话内完成

---

## 8. 引用源

| 源文件 | 用途 |
|--------|------|
| `process/HLD-CR-011.md` | PPDCS 前瞻架构设计（§8-§11） |
| `process/changes/CR-013-ptm-tde-ppdcs-phase.md` | CR-013 需求定义 |
| `agents/ptm-tde.md` | ptm-tde 三阶段框架、PPDCS 子步骤定义 |
| `docs/ptm-tde/gate-spec.md` | GATE-4 当前骨架定义 |
| `docs/ptm-tde/data-flow-spec.md` | PPDCS 数据实体生产消费链路 |
| `docs/ptm-tde/skill-references.md` | Skill 间引用关系 |
