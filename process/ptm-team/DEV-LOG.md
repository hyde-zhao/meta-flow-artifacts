---
workflow_id: WF-PTM-TEAM-20260520-001
change_id: CR-010
phase: story-execution
created_at: "2026-06-01T18:00:00+08:00"
updated_at: "2026-06-01T19:00:00+08:00"
---

# DEV-LOG — CR-010 ptm-tde 三阶段框架改造

## LLD 设计批次

| Story ID | LLD 文件 | CP5 自动预检 | 状态 | 依赖 | tier |
|----------|---------|-------------|------|------|------|
| STORY-010-01 | `process/stories/STORY-010-01-archive-and-gate-spec-LLD.md` | `process/checks/CP5-STORY-010-01-archive-and-gate-spec-LLD-IMPLEMENTABILITY.md` → PASS | lld-ready-for-review | 无 | S |
| STORY-010-03 | `process/stories/STORY-010-03-LLD.md` | `process/checks/CP5-STORY-010-03-...IMPLEMENTABILITY.md` → PASS | lld-ready-for-review | STORY-010-01（gate-spec.md） | S |
| STORY-010-04 | `process/stories/STORY-010-04-LLD.md` | `process/checks/CP5-STORY-010-04-...IMPLEMENTABILITY.md` → PASS | lld-ready-for-review | STORY-010-03（SKILL.md Gate 模式） | M |

---

## STORY-010-01: 归档旧 checkpoint-spec + 创建 gate-spec

### LLD 摘要

- **当前状态**：部分已实施——`checkpoint-spec-v1-archived.md`（归档副本）和 `gate-spec.md`（5 Gate 规范）已创建。LLD 为补写。
- **已实施产物**：
  - `docs/ptm-tde/checkpoint-spec-v1-archived.md`：旧 CP01-CP12 规范的完整归档，头部追加归档标记（归档日期 2026-06-01、原因 CR-010、替代路径 `gate-spec.md`）
  - `docs/ptm-tde/gate-spec.md`：5 个 Gate（GATE-1 至 GATE-5）完整规范，含 CP↔Gate 映射表、跨阶段拓扑绑定检查，309 行
- **待实施差异**（2 项）：
  1. `docs/ptm-tde/gate-spec.md` 路径引用更新：`doc/STATE.yaml`→`process/STATE.yaml`（7 处）、`checkpoints/`→`process/checkpoints/`（20+ 处）
  2. `docs/ptm-tde/checkpoint-spec.md` 替换为指向 `gate-spec.md` 的简短重定向说明（约 20 行，替代当前 160 行旧内容）
- **TASK-ID**：5 个（TASK-010-01-01/02 已实施，TASK-010-01-03 路径更新，TASK-010-01-04 重定向替换，TASK-010-01-05 验证）
- **验证入口**：6 项测试（T01-T06），覆盖 gate-spec.md 路径替换完整性/正确性、checkpoint-spec.md 重定向、归档完整性、Gate 章节结构、CP↔Gate 映射表

### 未决点

| ID | 类型 | 问题 | 影响 | 下一动作 |
|----|------|------|------|----------|
| LCQ-STORY-010-01-01 | 已决策 | gate-spec.md 路径更新方向（doc/→process/） | gate-spec.md 30+ 处引用 | 用户指令已明确采用推荐方案（`process/STATE.yaml` + `process/checkpoints/`）；LLD §4 已列出逐行替换清单 |
| O-010-01-01 | OPEN | HLD-CR-010.md 中 `doc/STATE.yaml` 和 `checkpoints/` 路径是否需要同步更新 | HLD-CR-010.md 一致性 | 待 CP5 全量 LLD 确认时由 meta-po 决策（推荐：HLD 保持设计原样不改） |

### 依赖与门控

- **依赖类型**：无前置 Story 依赖（CR-010 起点 Story）
- **被依赖**：STORY-010-02（主 Agent 框架重写，contract 依赖 gate-spec.md）、STORY-010-03（checkpoint-manager SKILL.md，contract 依赖 gate-spec.md）
- **文件所有权**：3 个文件（checkpoint-spec.md、checkpoint-spec-v1-archived.md、gate-spec.md），全部为本 Story primary，无冲突
- **CP5 自动预检**：PASS（14 项 checklist，0 项 FAIL，2 项 N/A）

### 待确认项（风险提示）

- gate-spec.md 路径替换后，`process/checkpoints/` 与 Meta Flow 的 `process/checks/` 命名相近，需在 gate-spec.md 首部追加区分说明（已在 LLD §12.1 R3 风险缓解措施中规划）
- checkpoint-spec.md 替换为重定向后，外部引用（wiki、旧项目文档）可能通过文件名访问到重定向而非预期旧内容——风险接受，旧内容已在 archived 文件中保留

---

## STORY-010-02: 重写主 Agent 框架部分

### 实现摘要

- **实现文件名**：`agents/ptm-tde.md`
- **变更类型**：框架重写（7 处变更，约 +60 行净增，305 → 368 行）
- **实施方式**：Python 脚本逐章节组装，通过章节标题定位边界并替换
- **变更统计**：

| 变更 | 章节 | 行号范围 | 说明 |
|------|------|----------|------|
| 1 | `## 状态机` → `## 三阶段框架` | L44-110 | 删除 11 步线性列表，替换为 ASCII 流程图 + 3 阶段/Gate 总览表 + CP↔Gate 映射表 + 追踪链 + 扩展分支 |
| 2 | `## 运行时工作目录` | L112-192 | 目录树从 `analysis/`/`design/` 迁至 `kym/`/`mfq/`/`ppdcs/`/`process/`；新增目录迁移对照表；路径规则更新为 6 条 |
| 3 | `## 用户确认点` | L259-275 | CP02/CP09/CP11 → GATE-2/3/4；新增 GATE-3 MFQ Exit Gate（标注为新增）；5 条确认点通用规则保留不变 |
| 4 | `## Skill 触发词映射` | L277-298 | 阶段列从旧步骤名改为 KYM/MFQ/PPDCS/共享工具/扩展；checkpoint-manager 触发词新增 GATE-1~5 + CP01；design-planner 阶段改为 MFQ |
| 5 | `## 初始化流程` | L301-311 | 目录创建列表更新为 12+ 子目录；STATE.yaml 路径改为 `process/STATE.yaml`，记录 `current_phase: kym` + `current_step: feature-parser`；GATE-1 替代 CP01 |
| 6 | `## 追踪链` | L349-355 | `confirmed-scenarios.md` 路径从 `analysis/scenarios/` 改为 `kym/scenarios/` |
| 7 | `## 约束` | L361-368 | 保持完全不变 |

- **交叉引用更新**：
  - `公共因子库`：`analysis/factor-usage/` → `mfq/factor-usage/`（L215）
  - `测试因子`：`analysis/scenarios/confirmed-scenarios.md` → `kym/scenarios/confirmed-scenarios.md`（L251）

### 关键决策与偏差

- **无偏差**：全部 7 处变更严格按 LLD §3 实施
- **旧路径保留策略**：所有 `analysis/` 和 `design/` 旧路径仅存在于目录迁移对照表（历史参考）和错误示例中，不用于任何活动引用
- **section ordering**：扩展分支保留在三阶段框架末尾（### 扩展分支），作为独立于三阶段的补充流程

### 已知限制

1. 旧 CP 编号仅在 CP↔Gate 映射表和 checkpoint-manager 触发词（CP01 兼容别名）中出现
2. `analysis/factor-usage/` 和 `analysis/scenarios/confirmed-scenarios.md` 的旧引用仅剩迁移表一行
3. 未修改 Skills SKILL.md 文件（路径对齐由 CR-011/012/013 承接）

### 提供给 meta-qa 的验证入口

| # | 验证项 | 命令/方式 |
|---|--------|----------|
| 1 | 11 步消除 | `grep "11 步\|11步" agents/ptm-tde.md` → 0 结果 |
| 2 | 旧路径仅存迁移表 | `grep "analysis/scenarios/confirmed-scenarios\|analysis/factor-usage/" agents/ptm-tde.md` → 仅迁移表行 |
| 3 | doc/STATE.yaml 消除 | `grep "doc/STATE.yaml" agents/ptm-tde.md` → 0 结果 |
| 4 | Gate 全部存在 | `grep -c "GATE-[1-5]" agents/ptm-tde.md` → 25 次 |
| 5 | 三阶段框架完整性 | 检查 L44-110 含 ASCII 图 + 阶段表 + Gate 表 + 映射表 |
| 6 | 8 个保留章节完整 | 理论基础/CAE 三元组/公共因子库/测试因子/目录层级规范/物理用例字段规范/交付物/约束 均存在 |

### 风险提示

- **`process/plan/` 与 ptm-team 运行时 `process/` 同名不同义**：主 Agent 描述的是特性项目根目录，需在文档中区分（Gotcha #1 已标注）
- **过渡期 Skill 仍写旧路径**：CR-010 只改主 Agent 框架，Skill 路径对齐由 CR-011/012/013 完成

### CP6 编码完成门

- **CP6 文件**：`process/checks/CP6-STORY-010-02-rewrite-main-agent-framework-CODING-DONE.md`
- **结论**：PASS（27 项 checklist 全部通过，0 项 FAIL，0 项 N/A）
- **Agent Dispatch**：inline-fallback（meta-dev 直接执行）

---

## STORY-010-03: 更新 checkpoint-manager SKILL.md（Gate 模式）

### LLD 摘要

- **当前状态**：SKILL.md 已被前序 meta-dev 完全更新为 Gate 模式，12 项 CR 变更需求中 11 项已实现。
- **差异修正**：1 处偏差 —— `--gate` 参数在 SKILL.md §脚本用法中使用位置参数格式 `GATE-1`，应改为命名参数 `--gate GATE-1` 以匹配 `run_checkpoint.py` argparse 接口。
- **TASK-ID**：3 个（TASK-010-03-01 已实现，TASK-010-03-02 差异修正 5 行，TASK-010-03-03 验证）
- **验证入口**：8 项测试（T1-T8），覆盖 frontmatter/Gate 章节/映射表/gate-spec 引用/脚本用法/Gotchas/旧标题清除/Gate 模式描述

### 未决点

| ID | 类型 | 问题 | 状态 |
|----|------|------|------|
| LCQ-STORY-010-03-01 | 实现灰区（已推荐） | `--gate` 参数格式：位置 vs 命名 | 推荐 `--gate GATE-1`（命名参数） |
| O-STORY-010-03-01 | OPEN | `--cp` 阶段内自检输出路径是否需在 SKILL.md 显式定义 | 待 CP5 人工确认 |

### 依赖类型

- `lld_gate`: contract（依赖 STORY-010-01 `gate-spec.md` 中的 Gate 编号和命名定义）
- `dev_gate`: runtime（SKILL.md 引用 gate-spec.md 路径，不阻塞开发）

### 文件所有权

- `primary`: `skills/checkpoint-manager/SKILL.md`

### 实现结果（2026-06-01）

- **CP6 文件**：`process/checks/CP6-STORY-010-03-checkpoint-manager-skill-gate-mode-CODING-DONE.md`
- **结论**：PASS
- **实施变更**：
  1. 路径修正：`doc/STATE.yaml` → `process/STATE.yaml`（7 处），`checkpoints/` → `process/checkpoints/`（20+ 处），`doc/` 可创建 → `process/` 可创建
  2. 脚本用法格式修正：5 处位置参数 `GATE-N` → `--gate GATE-N`
  3. 输出目录修正：`kym/、mfq/、ppdcs/、process/、checkpoints/、doc/` → `kym/、mfq/、ppdcs/、process/、process/checkpoints/`
- **验证命令**：
  ```bash
  grep -c "doc/STATE.yaml" skills/checkpoint-manager/SKILL.md  # → 0
  grep -c '`checkpoints/' skills/checkpoint-manager/SKILL.md     # → 0
  grep -c "\-\-gate GATE-" skills/checkpoint-manager/SKILL.md    # → 5
  grep -c "^## GATE-" skills/checkpoint-manager/SKILL.md         # → 5
  ```
- **Agent Dispatch**：inline-fallback（meta-dev 直接执行）

### OPEN / Spike 状态更新

| ID | 状态 | 更新 |
|----|------|------|
| LCQ-STORY-010-03-01 | resolved | 已按推荐方案实施：`--gate GATE-N` 命名参数格式 |
| O-STORY-010-03-01 | 未变更 | 待 CP5 人工确认时决策 |

---

## STORY-010-04: 改造 run_checkpoint.py（双模式路由）

### LLD 摘要

- **当前状态**：`run_checkpoint.py`（170 行）仅支持 CP01 模式，使用旧目录 `analysis/`/`design/`。
- **改造范围**：新增 `--gate` 和 `--cp` 互斥参数；CP↔Gate 路由映射表（dict 常量）；目录迁移至 `kym/`/`mfq/`/`ppdcs/`/`process/`；5 个 Gate handler（GATE-1 完整 + GATE-2-5 骨架）；阶段内滚动自检处理；STATE.yaml 字段更新；向后兼容 `dispatch_legacy`。
- **TASK-ID**：10 个（TASK-010-04-01 至 TASK-010-04-10），约 170 行 → 350 行
- **验证入口**：14 项测试（T1-T14），覆盖 Gate 基本路径、CP 路由一致性、交叉对比（CP09 vs CP11→GATE-4）、互斥、向后兼容、目录迁移、STATE.yaml

### 关键决策

- Gate handler 粒度：推荐骨架实现（Entry Criteria 校验 + 产物目录存在性），详细 Checklist 由 CR-012/013 扩展
- 向后兼容范围：`dispatch_legacy` 仅保留 CP01 位置参数兼容（与原行为一致），其他 CP 编号只能通过 `--cp` 命名参数使用

### 未决点

| ID | 类型 | 问题 | 状态 |
|----|------|------|------|
| LCQ-STORY-010-04-01 | 实现灰区（已推荐） | Gate handler 粒度：骨架 vs 完整 Checklist | 推荐骨架（由 CR-012/013 扩展） |
| LCQ-STORY-010-04-02 | 实现灰区（已推荐） | legacy 兼容范围：仅 CP01 位置参数 vs 扩展 CP01-CP12 | 推荐仅 CP01 位置参数 |
| O-STORY-010-04-01 | OPEN | GATE-2 至 GATE-5 骨架确切粒度 | 待 CP5 人工确认 |
| O-STORY-010-04-02 | OPEN | dispatch_legacy 是否扩展到 CP01-CP12 | 待 CP5 人工确认 |

### 依赖类型

- `lld_gate`: contract（依赖 STORY-010-03 SKILL.md 中的 `--gate` 参数约定和 Gate 名称）
- `dev_gate`: runtime（SKILL.md 与 run_checkpoint.py 可并行开发，但交互契约需一致）

### 文件所有权

- `primary`: `skills/checkpoint-manager/scripts/run_checkpoint.py`

### 实现结果（2026-06-01）

- **CP6 文件**：`process/checks/CP6-STORY-010-04-run-checkpoint-script-dual-mode-CODING-DONE.md`
- **结论**：PASS（10/10 TASK-ID 全部完成）
- **实施变更**：
  1. 新增 `CP_TO_GATE`（12 条目）和 `GATE_NAMES`（5 条目）模块级常量
  2. 新增 `--gate`（5 选项）和 `--cp`（CP01-CP12）互斥参数组 + `checkpoint_id` 位置参数（CP01）向后兼容
  3. 目录迁移：`REQUIRED_DIRS` 从 12 个旧路径改为 13 个新路径（`kym/`/`mfq/`/`ppdcs/`/`process/`）
  4. 重构 `run_cp01` 核心逻辑为 `run_gate_1`，`run_cp01` 改为包装委托
  5. 新增 `run_gate_2` 至 `run_gate_5` 四个骨架 handler（Entry Criteria + 目录存在性 + PENDING 标注）
  6. 新增 `dispatch_gate` / `dispatch_cp` / `dispatch_legacy` 三个路由分发函数
  7. 新增 `run_internal_check` 阶段内滚动自检骨架
  8. 修改 `write_state`：写入 `process/STATE.yaml`，含 `current_phase` + `current_step`
  9. 新增 `gate_output_path` 辅助函数
  10. 所有输出文件头部含 `check_depth: skeleton` 标注
- **脚本规模**：170 行 → 470+ 行，20 个函数定义
- **Agent Dispatch**：inline-fallback（meta-dev 直接执行）

### OPEN / Spike 状态更新

| ID | 状态 | 更新 |
|----|------|------|
| LCQ-STORY-010-04-01 | resolved | 已按推荐方案实施：骨架实现 |
| LCQ-STORY-010-04-02 | resolved | 已按推荐方案实施：仅 CP01 位置参数 |

---

## LLD Clarification Queue 汇总

以下为实现灰区记录，已记录推荐方案和备选方案，等待 meta-po 在 CP5 人工确认时统一裁定。

| queue_id | story_id | 问题 | 推荐方案 | 阻断 LLD | 状态 |
|----------|----------|------|----------|----------|------|
| LCQ-STORY-010-01-01 | STORY-010-01 | gate-spec.md 路径更新方向（doc/→process/） | process/STATE.yaml + process/checkpoints/ | false | resolved（用户指令已明确） |
| LCQ-STORY-010-03-01 | STORY-010-03 | `--gate` 参数格式：位置 vs 命名 | `--gate GATE-1`（命名） | false | open |
| LCQ-STORY-010-04-01 | STORY-010-04 | Gate handler 粒度：骨架 vs 完整 | 骨架（CR-012/013 扩展） | false | open |
| LCQ-STORY-010-04-02 | STORY-010-04 | legacy 兼容范围 | 仅 CP01 位置参数 | false | open |

---

## 待确认项（风险提示）

- **STORY-010-01**：O-010-01-01（HLD-CR-010.md 路径一致性是否需同步更新）待 CP5 确认时决策；gate-spec.md 路径替换后 `process/checkpoints/` 与 Meta Flow `process/checks/` 命名相近风险需在 gate-spec.md 首部追加区分说明
- 两项 LLD 的 OPEN 项均不阻断实现，但 Gate handler 骨架实现粒度需在 CP5 确认时明确，以便 TASK-010-04-05 的函数体设计
- SKILL.md 与 run_checkpoint.py 之间的 `--gate` 参数格式一致性需在双方实现（或修正）后交叉验证（STORY-010-04 T4）

---

## CR-030 Wave 3: 改进与闭环 LLD 设计证据（meta-dev, 2026-07-16）

### 设计范围

产出 CR-030 Wave 3 中 improvement + tracking 组的 3 个 Story 设计证据：

| Story ID | 类型 | LLD 路径 | CP5 检查点 | 状态 |
|---|---|---|---|---|
| ST-RA-03 | full-lld | `process/stories/STORY-RA-03-LLD.md` | `process/checks/CP5-ST-RA-03-improvement-governance-LLD-IMPLEMENTABILITY.md` | lld-ready-for-review |
| ST-RA-04 | full-lld | `process/stories/STORY-RA-04-LLD.md` | `process/checks/CP5-ST-RA-04-closure-tracking-LLD-IMPLEMENTABILITY.md` | lld-ready-for-review |
| ST-RA-06.3-TRACK | technical-note | `process/stories/STORY-RA-06.3-TRACK-measure-baseline-refresh.md## 技术说明` | `process/checks/CP5-ST-RA-06.3-TRACK-measure-baseline-refresh-LLD-IMPLEMENTABILITY.md` | lld-ready-for-review |

### 设计证据摘要

**ST-RA-03（改进输入治理）**：
- improvement-tracker Skill CA/PA 侧：CA/PA Proposal 草案生成、批准状态机（draft→approved/rejected）、Approved Improvement Input 生成（不可变）、消费者映射
- 写入 `skills/improvement-tracker/SKILL.md` §3 + `templates/capa-proposal.yaml` + `templates/approved-input.yaml`
- 核心约束：未确认 RA 拒绝生成 CA/PA、未批准拒绝生成 Approved Input、生成后不可变、只有 reviewer 能变更 approval_status

**ST-RA-04（闭环跟踪与有效性决策）**：
- improvement-tracker Skill 跟踪侧：Action Item 状态机（not-started→in-progress→done/overdue）、Effectiveness Check 模型、Closure Decision 四条件（行动项完成+有效性通过+观察窗满足+无同类复发）
- 追加 `skills/improvement-tracker/SKILL.md` §4 + `templates/action-item.yaml` + `templates/effectiveness-check.yaml` + `templates/closure-decision.yaml`
- 核心约束：关闭条件不可绕过、不自动关闭、不自动完成行动项、不自动判定有效性

**ST-RA-06.3-TRACK（措施基线管理与刷新提示）**（technical-note）：
- MeasureBaseline 管理 + S2 措施刷新规则（5 种状态：needs-baseline/active/completed/needs-review/superseded）
- 追加 `skills/improvement-tracker/SKILL.md` §5 + `templates/measure-baseline.yaml`
- 核心约束：只产出提示不自动改变批准状态、不自动修改下游任务、无基线不判失效

### 文件所有权与串行约束

`improvement-tracker/SKILL.md` 串行写入：ST-RA-03（§3 CA/PA 侧）→ ST-RA-04（§4 跟踪侧）→ ST-RA-06.3-TRACK（§5 MeasureBaseline 侧）。每个 Story 在独立章节内写入，使用显式分节标记确保追加安全。

### CP5 自动预检结论

全部 3 个 Story 的 CP5 自动预检 **PASS**：
- ST-RA-03：14 个语义要点完整，16 个测试场景覆盖全部 AC 和 Feature TEST-PLAN
- ST-RA-04：14 个语义要点完整，18 个测试场景覆盖全部 AC 和 Feature TEST-PLAN
- ST-RA-06.3-TRACK：技术说明 7 个必备章节完整，8 个测试入口覆盖 5 条 AC

### 开放项（非阻塞）

| ID | Story | 主题 | 重访条件 |
|---|---|---|---|
| OPEN-RA-03-01 | ST-RA-03 | consumer_status 回写协调 | ST-RA-04 实现时协调 |
| OPEN-RA-03-02 | ST-RA-03 | Proposal 文件存储路径约定 | CP5 Decision Brief 确认 |
| OPEN-RA-04-01 | ST-RA-04 | measure_link schema 需 ST-RA-INGEST-DB 确认 | ST-RA-INGEST-DB LLD 确认后关闭 |
| OPEN-RA-04-02 | ST-RA-04 | 观察窗豁免权限边界 | CP7 fixture 验证 |

### Clarification Queue

无 blocking clarification item。所有灰区问题已在 LLD / technical-note 中记录决策。

### 待全量 CP5 确认

本批次（ST-RA-03、ST-RA-04、ST-RA-06.3-TRACK）是 W3 的 improvement + tracking 组。W3 的另一 Story（ST-RA-06.2-REFRESH，F-021，independent）的设计证据由另一个 meta-dev 线程产出。W1/W2 的 ingestion + analysis 组设计证据也由其他线程产出。全部目标 Story 设计证据统一确认后，本组进入实现阶段。

### 文件清单

- `process/stories/STORY-RA-03-LLD.md`（full-lld, 16 小节）
- `process/stories/STORY-RA-04-LLD.md`（full-lld, 16 小节）
- `process/stories/STORY-RA-06.3-TRACK-measure-baseline-refresh.md`（## 技术说明 更新）
- `process/checks/CP5-ST-RA-03-improvement-governance-LLD-IMPLEMENTABILITY.md`
- `process/checks/CP5-ST-RA-04-closure-tracking-LLD-IMPLEMENTABILITY.md`
- `process/checks/CP5-ST-RA-06.3-TRACK-measure-baseline-refresh-LLD-IMPLEMENTABILITY.md`

---

## CR-030 Wave 1: INGESTION + DB LLD 设计证据（meta-dev, 2026-07-16）

### 设计范围

产出 CR-030 Wave 1/2 中 ingestion + DB 组的 5 个 Story 设计证据：

| Story ID | 类型 | LLD 路径 | CP5 检查点 | 状态 |
|---|---|---|---|---|
| ST-RA-INGEST-DB | full-lld | `process/stories/STORY-RA-INGEST-DB-LLD.md` | `process/checks/CP5-ST-RA-INGEST-DB-LLD-IMPLEMENTABILITY.md` | lld-ready-for-review |
| ST-RA-05.1-INGEST | full-lld | `process/stories/STORY-RA-05.1-INGEST-LLD.md` | `process/checks/CP5-ST-RA-05.1-INGEST-LLD-IMPLEMENTABILITY.md` | lld-ready-for-review |
| ST-RA-05.2-CLEAN | full-lld | `process/stories/STORY-RA-05.2-CLEAN-LLD.md` | `process/checks/CP5-ST-RA-05.2-CLEAN-LLD-IMPLEMENTABILITY.md` | lld-ready-for-review |
| ST-RA-06.1-DETECT | full-lld | `process/stories/STORY-RA-06.1-DETECT-LLD.md` | `process/checks/CP5-ST-RA-06.1-DETECT-LLD-IMPLEMENTABILITY.md` | lld-ready-for-review |
| ST-NRA-03 | technical-note | `process/stories/STORY-NRA-03-ingestion-failure-protection.md## 技术说明` | `process/checks/CP5-ST-NRA-03-LLD-IMPLEMENTABILITY.md` | lld-ready-for-review |

### 设计证据摘要

**ST-RA-INGEST-DB（SQLite schema 与 DAO 层）**：
- 4 张表 DDL（ticket, ticket_version, ingestion_batch, change_history）+ 9 个索引 + 8 个约束
- DAO 层接口：13 个公共方法（insert/upsert/select + 事务管理）
- 核心约束：SQLite WAL 模式、FK 强制、source_ticket_id UNIQUE、事务原子性、数据库文件不进 Git

**ST-RA-05.1-INGEST（ITR 受控摄取与快照）**：
- Allowlist 校验算法（URL pattern + 参数白名单 + 方法白名单 + 拒绝认证头）
- HTTP GET 请求（30s 超时、不自动重试、分页支持）
- 原始快照保存（data/snapshots/ 目录、sha256 hash、元数据记录）
- 安全约束：deny-by-default（6 条拒绝规则）

**ST-RA-05.2-CLEAN（字段映射、清洗与质量报告）**：
- ITR → ticket 字段映射配置（14 个字段映射）
- quality_flag 四级判定（clean/incomplete/anomaly/blocked）
- IngestionQualityReport 生成（含质量阈值判断、analyze_ratio、overall_status）
- 敏感字段三级分类（raw/cleaned/report）+ 未知字段保留策略

**ST-RA-06.1-DETECT（变更检测、合并与版本历史）**：
- 字段级 diff 算法 + 语义冲突判定（status/severity/product 变更 → manual_queue）
- 4 种 change_type（new/modified/unchanged/conflict）+ 3 种 resolution（auto_merged/manual_queue/rejected）
- ticket_version 版本递增 + change_history 写入
- conflict-queue.yaml 输出供人工 reviewer 决策

**ST-NRA-03（摄取失败保护）**（technical-note）：
- 4 个失败场景的守卫位置/动作/设计依据（HTTP 非 2xx、Schema 不匹配、SQLite 写入失败、无 stable ID）
- 失败路径消费正向 Story LLD 的 §6（错误处理与降级）和 §4（核心流程）
- 豁免完整 LLD 的 4 条理由

### 共享文件串行约束

`skills/itr-ticket-ingestion/SKILL.md` 串行写入：ST-RA-05.1-INGEST（§HTTP 摄取与快照保存）→ ST-RA-05.2-CLEAN（§清洗与质量报告）→ ST-NRA-03（§失败保护）→ ST-RA-06.1-DETECT（§变更检测与版本历史）。每个 Story 在独立章节内写入，使用显式分节标记确保追加安全。

### 依赖关系

```
ST-RA-INGEST-DB (schema/DAO)
        ▲
        │ depends on (soft)
        │
ST-RA-05.1-INGEST ────────────── ST-RA-05.2-CLEAN
        │                               │
        └───────┬───────────────────────┘
                │ extends (failure paths)
                ▼
          ST-NRA-03
                │
                │ depends on (hard)
                ▼
        ST-RA-06.1-DETECT (Wave 2, LLD 同批次产出)
```

### CP5 自动预检结论

全部 5 个 Story 的 CP5 自动预检 **PASS**：
- 4 个 full-lld Story：14 个语义要点完整覆盖，测试用例矩阵完整
- 1 个 technical-note Story：7 个必备章节完整，豁免理由充分

### 开放项与灰区

| ID | Story | 主题 | blocks_lld | 状态 |
|---|---|---|---|---|
| LCQ-ST-RA-INGEST-DB-01 | ST-RA-INGEST-DB | analysis_run 和 measure_link 表 DDL 归属 | true | open（已入 QUESTION-LEDGER） |
| O-DB-01 | ST-RA-INGEST-DB | 数据库文件默认路径确认 | false | open |
| O-ING-01 | ST-RA-05.1-INGEST | ITR 真实响应 schema 结构 | false | open（需首次受控探测） |
| O-CLN-01 | ST-RA-05.2-CLEAN | ITR 字段实际名称 | false | open（需首次受控探测） |
| O-DET-01 | ST-RA-06.1-DETECT | 冲突字段白名单完整性 | false | open |

### Clarification Queue

1 个 blocking clarification：`LCQ-ST-RA-INGEST-DB-01`（analysis_run/measure_link 表归属），已写入 `process/state/QUESTION-LEDGER.ndjson`。不阻塞同批次其他 Story 的 LLD 产出。

### 待全量 CP5 确认

本批次（ST-RA-INGEST-DB、ST-RA-05.1-INGEST、ST-RA-05.2-CLEAN、ST-RA-06.1-DETECT、ST-NRA-03）是 W1/W2 的 ingestion + DB 组。W2 的 analysis 组（ST-RA-01、ST-RA-02、ST-RA-05.3-ANALYZE、ST-NRA-01、ST-NRA-02、ST-RA-06.2-REFRESH）和 W3 的 improvement + tracking 组（ST-RA-03、ST-RA-04、ST-RA-06.3-TRACK）设计证据由其他 meta-dev 线程产出。全部目标 Story 设计证据统一确认后，按 Wave 进入实现阶段。

### 文件清单

- `process/stories/STORY-RA-INGEST-DB-LLD.md`（full-lld, 14 章节）
- `process/stories/STORY-RA-05.1-INGEST-LLD.md`（full-lld, 14 章节）
- `process/stories/STORY-RA-05.2-CLEAN-LLD.md`（full-lld, 14 章节）
- `process/stories/STORY-RA-06.1-DETECT-LLD.md`（full-lld, 14 章节）
- `process/stories/STORY-NRA-03-ingestion-failure-protection.md`（## 技术说明 更新）
- `process/checks/CP5-ST-RA-INGEST-DB-LLD-IMPLEMENTABILITY.md`
- `process/checks/CP5-ST-RA-05.1-INGEST-LLD-IMPLEMENTABILITY.md`
- `process/checks/CP5-ST-RA-05.2-CLEAN-LLD-IMPLEMENTABILITY.md`
- `process/checks/CP5-ST-RA-06.1-DETECT-LLD-IMPLEMENTABILITY.md`
- `process/checks/CP5-ST-NRA-03-LLD-IMPLEMENTABILITY.md`
- `process/state/QUESTION-LEDGER.ndjson`（追加 1 条：LCQ-ST-RA-INGEST-DB-01）

---

## Wave 2 实现 — ST-RA-01: 资格检查与可信输入建立

**执行时间**：2026-07-16
**执行 Agent**：meta-dev（dev-yang 线程）
**设计证据**：`process/stories/STORY-RA-01-qualification-evidence-LLD.md` (full-lld, 14 章节, v1.1)
**依赖类型**：`depends_on: ST-RA-INGEST-DB`（contract 依赖，上游已完成）
**文件所有权**：`skills/reverse-analysis/SKILL.md`（shared，serial_group=reverse-analysis，写入 §1-§2），`agents/ptm-tse.md`（更新）

### 实现对象清单

| 文件 | 操作 | 行数 | TASK-ID |
|------|------|------|---------|
| `skills/reverse-analysis/SKILL.md` | 创建 | ~360 | TASK-ANL-01 ~ TASK-ANL-05 |
| `skills/reverse-analysis/templates/` | 创建（空） | — | TASK-ANL-01 |
| `agents/ptm-tse.md` | 修改（全量重写） | ~180 | TASK-ANL-06 |

### 设计契约映射

全部 11 项 LLD 契约（§2.1 入口契约 / §2.2 资格矩阵 / §2.3 证据分类 / §3.1 只读消费 / §3.2 内存状态 / §4 核心流程 / §5 状态枚举 / §6 错误处理 / §8 安全权限 / §11 平台差异 / §12 集成契约）已映射到产物。

### 验收标准覆盖

| AC | 状态 | 位置 |
|----|------|------|
| P1 必做/P2 可选/P3/P4 不自动进入 | 覆盖 | SKILL.md §1.2 Step 4 + §1.4 |
| 内部问题 deferred | 覆盖 | SKILL.md §1.2 Step 5 |
| 五条证据线 fact/hypothesis/unknown/gap | 覆盖 | SKILL.md §2.3.2 |
| <3 条有效线时不能 confirmed | 覆盖 | SKILL.md §2.4.2（threshold + max_rc_state） |
| ptm-tse.md 引用 reverse-analysis | 覆盖 | ptm-tse.md skills + 调用流程 |

### 资格检查规则实现细节

**资格判定矩阵**（SKILL.md §1.4）:
- P1 + clean/incomplete/anomaly + 有效 source_ticket_id → `eligible`
- P2 + clean/incomplete/anomaly + 有效 source_ticket_id → `eligible_on_request`
- P3/P4/severity 为空 → `rejected`
- is_internal=true → `deferred`
- quality_flag=blocked / source_ticket_id 空或无效 → `blocked`

**三线阈值**（SKILL.md §2.4）:
- valid_count = category ∈ {fact, hypothesis} AND validity=valid 的线数
- >= 3: threshold=passed, max_rc_state=evidence-backed
- < 3: threshold=insufficient, max_rc_state=AI candidate

### 安全禁止项

9 项禁止操作已逐一声明：直接 SQL、修改 dao.py/schema.sql、读取 raw_json、外部连接、凭据读取、自动确认根因、自动分发 CA/PA、修改 ticket 表、raw_statement 直接展示为结论。

### 设计缺口

| ID | 描述 | 严重度 | 处理 |
|----|------|--------|------|
| GAP-RA01-01 | quality_flag 枚举值与 LLD 不一致（clean vs PASS） | low | 使用实际 schema 值 |
| GAP-RA01-02 | is_internal 列在 schema 中缺失 | medium | 降级为视为 false，需上游补充 |
| GAP-RA01-03 | severity 列无 CHECK 约束 | low | 降级为非 P1-P4 → rejected |

### 未决 OPEN 项

- OPEN-RA01-01: P3/P4 不可绕过（假设）
- OPEN-RA01-02: quality_flag=clean 是最低门槛（假设）
- OPEN-RA01-03: 五条线覆盖 ITR 字段（假设）
- OPEN-RA01-04: severity 枚举一致性（依赖上游）

### CP6 结论

**PASS** — `process/checks/CP6-ST-RA-01-qualification-evidence-CODING-DONE.md`

### 交接

- Return Packet: `process/returns/ST-RA-01-qualification-evidence.return.json`
- Evidence Index: `process/evidence/ST-RA-01-qualification-evidence.index.json`
- Story 状态: `ready-for-verification`
- 验证类型: 静态审查（Skill 文本定义，非可执行代码）
- 风险提醒: is_internal 缺失需 mock fixture；quality_flag 枚举不一致不是实现缺陷

---

## ST-RA-02: 六维分析引擎

### 实现摘要

- **Story ID**: ST-RA-02
- **Wave**: 2
- **来源**: ST-RA-02 LLD v1.1
- **产品**: 3 个文件，共 1419 行
- **实现证据**: `process/checks/CP6-ST-RA-02-six-dim-analysis-engine-CODING-DONE.md` → PASS

### 实现产物

| 产物 | 操作 | 行数 |
|------|------|------|
| `skills/reverse-analysis/SKILL.md` | 修改（追加 §3§4§5，重编号 §5-§8 → §6-§9） | 1097 |
| `skills/reverse-analysis/templates/ra-report.yaml` | 新建 | 169 |
| `skills/reverse-analysis/templates/metric-definition.yaml` | 新建 | 153 |

### 章节详细内容

**§3 六维分析引擎**（370 行）:
- 3.1 分析维度总览：六维度表（RC/PQ/ESC/TM/IMP/CMP），每个维度的输入、方法、输出契约
- 3.2 逐维度分析规则（6 个子章节）:
  - 3.2.1 根因（RC）：raw_statement 提取 → 5 Whys/鱼骨/组合方法 AI 候选生成 → 置信度判定（high/medium/low）
  - 3.2.2 产品质量（PQ）：按维度聚合 → Pareto 排序 → 条件缺陷密度（denominator.trusted=true 才启用）
  - 3.2.3 流出（ESC）：9 层控制层标准集合 → confirmed/candidate 分离 → nearest_intercept 推理
  - 3.2.4 漏测（TM）：PPDCS 归类（Prevention/Protection/Detection/Containment/Sustainment）→ 缺失字段标记 unknown
  - 3.2.5 改进（IMP）：前置条件检查 → CA（纠正）生成 → PA（预防）生成 → 六项必填字段约束
  - 3.2.6 环比同比（CMP）：窗口验证 → 样本量检查（>= 10）→ 同口径聚合 → 可信度评估 → N/A 标记
- 3.3 事实/假设分离规则：
  - 3.3.1 fact/hypothesis/unknown/gap 四类分类标准
  - 3.3.2 六个维度的分类应用矩阵
  - 3.3.3 四条强制规则（hypothesis 不可标 fact、不可自动 confirmed、unknown/gap 必有补充建议、未知默认 hypothesis）

**§4 根因状态机**（100 行）:
- 4.1 四层递进状态表（raw_statement → ai_candidate → evidence_backed → reviewer_confirmed）
- 4.2 状态转换规则与门控条件：
  - 4.2.1 转换规则表（5 种转换 + 触发方式 + 前置条件 + 证据要求）
  - 4.2.2 不可自动跃迁规则（ai_candidate → evidence_backed 需阈值触发；evidence_backed → reviewer_confirmed 需 reviewer 人工确认）
  - 4.2.3 transition_log 格式
- 4.3 三线阈值硬阻断：
  - valid_count 计算规则（category ∈ {fact, hypothesis} AND validity=valid）
  - valid_count >= 3 → passed, max_state=evidence_backed
  - valid_count < 3 → insufficient, hard cap at ai_candidate, confidence ≤ medium
  - 注意：只定义状态机门控；完整策略由 ST-NRA-01（§7）实现

**§5 指标定义与降级**（120 行）:
- 5.1 MetricDefinition 契约：10 个必填字段规范、版本化规则、指标管理约束
- 5.2 无可信分母降级策略：
  - 降级触发条件 4 种
  - 允许输出 4 类（数量/占比/Pareto/趋势）vs 禁止输出 3 类（密度/DPMO/速率）
  - 降级声明模板
  - 质量趋势降级（时间序列缺失）
- 5.3 流出证据分类：
  - confirmed escape layer 三条件（执行证据 + 覆盖范围匹配 + 来源可追溯）
  - candidate escape layer 三条件
  - 默认行为：无证据时全部 candidate

### 模板文件详情

**ra-report.yaml**（169 行，8 个 section）:
- §A 资格检查摘要 — eligibility 状态 + threshold 信息
- §B 根因分析 — 四层状态机全部字段（raw_statement / ai_candidates[] / evidence_backed / reviewer_confirmed / transition_log）
- §C 产品质量分析 — metrics[] / top_risk_modules[] / quality_trend / defect_density / degraded_notice
- §D 流出分析 — escape_points[] / candidate_layers[] / confirmed_layers[] / nearest_intercept
- §E 漏测分析 — ppdcs_category / missed_pattern / recommended_method / unknown_fields[]
- §F 改进候选 — capa_items[]（type / basis / target / owner_candidate / priority / acceptance_criteria / effectiveness_check）
- §G 环比同比 — mode / baseline_period / current_period / changes[] / na_reasons[]
- §H 证据集 — 来自 ST-RA-01 的 evidence_set

**metric-definition.yaml**（153 行，6 个示例指标）:
| metric_id | 维度 | 有分母? | 说明 |
|-----------|------|---------|------|
| m-p1p2-count | product_quality | 无 | P1/P2 绝对数量 |
| m-module-pareto | product_quality | 无 | 模块 Pareto 分布 |
| m-defect-density | product_quality | 有（默认不可信） | 缺陷密度示例 |
| m-escape-distribution | escape | 无 | 逃逸点分布 |
| m-test-missed-phase | test_missed | 无 | 漏测阶段 PPDCS |
| m-p1p2-mom-change | comparison | 有（可信） | 环比变化率 |

每个指标均包含 numerator/denominator/window/cutoff_date/metric_version/na_condition/degraded_output 完整字段。

### LLD 契约覆盖

| LLD 章节 | 实现位置 | 状态 |
|----------|---------|------|
| §2.1 六维对外契约 | SKILL.md §3.2 | ✅ |
| §2.2 根因状态机接口 | SKILL.md §4.1 | ✅ |
| §2.3 状态转换规则 | SKILL.md §4.2 | ✅ |
| §2.4 MetricDefinition 模板 | metric-definition.yaml | ✅ |
| §3.1 ra-report 完整 schema | ra-report.yaml | ✅ |
| §4 核心流程 | SKILL.md §3.2 各维度 Step-by-Step | ✅ |
| §5 状态机 | SKILL.md §4.1 四层转换表 | ✅ |
| §6 错误处理与降级 | SKILL.md §3.2 各 Data Sources 表 + §5.2 | ✅ |
| §10 回滚策略 | 模板文件可独立回退 | ✅ |
| §11 平台差异 | 无实际差异 | ✅ |
| §12 集成契约 | 模板间契约（metrics[] 引用 metric_id） | ✅ |

### 设计缺口与开放项

| ID | 描述 | 严重度 | 处理 |
|----|------|--------|------|
| OPEN-RA02-01 | CA/PA 候选与 improvement-tracker 交接格式 | medium | capa_items[] 已定义六项必填字段；ST-RA-03 只能追加批准状态 |
| OPEN-RA02-02 | 环比同比最小样本量阈值 | low | 默认值 10，Skill 文本中已标注 |
| OPEN-RA02-03 | 5 Whys / 鱼骨图方法为 AI 文本引导 | low | 不是代码实现；通过 Skill 执行步骤文本驱动 |
| OPEN-RA02-04 | PPDCS 归类映射表 | low | 以对照表形式在 Skill 文本中维护 |
| OPEN-RA02-05 | 环比同比自然月/季对齐 | low | 当前默认值；自定义窗口需扩展 |

### 禁止事项新增

在原有 9 项禁止操作基础上新增 3 项：
- 将降级数据标注为"缺陷密度"或"DPMO"
- 将无控制证据的 escape layer 标记为 confirmed
- 跳过证据不足的维度或输出虚假结论

### CP6 结论

**PASS** — `process/checks/CP6-ST-RA-02-six-dim-analysis-engine-CODING-DONE.md`

### 交接

- Story 状态: `ready-for-verification`
- 验证类型: 静态审查（Skill 文本定义 + YAML 模板，非可执行代码）
- 风险提醒: OPEN-RA02-01（CA/PA 交接格式）需在 CP7 或 ST-RA-03 前对齐；OPEN-RA02-02（样本量阈值）可在 CP7 确认时决策
- 验证入口: 参见 CP6 检查 §10 后续交接

---

## ST-NRA-01: 证据不足保护（拒绝虚假根因）

### 实现日期

2026-07-16

### Story 信息

- **Story ID**: ST-NRA-01
- **Feature**: FEAT-RA-ANALYSIS（F-021）
- **Wave**: 2
- **设计证据类型**: technical-note
- **来源**: CP5-CR030-DESIGN-EVIDENCE-BATCH（approved）
- **依赖**: ST-RA-01（资格检查阈值逻辑，已完成）
- **实现证据**: `process/checks/CP6-ST-NRA-01-evidence-threshold-guard-CODING-DONE.md` → PASS

### 实现摘要

在 `skills/reverse-analysis/SKILL.md` §7 替换占位符为完整证据不足保护章（约 200 行）：

| 小节 | 内容 | 关键规则 |
|------|------|---------|
| §7.1 阈值硬阻断 | valid_count < 3 时二次校验，两个时机（分析输出前 + report_refs 写入前） | root_cause 上限 ai_candidate，confidence 上限 medium |
| §7.2 缺失证据分类 | gap_source 四类：ITR缺失/测试缺失/流程缺失/外部依赖 | 与 §2.3.2 gap_owner 互补（人员维度 vs 根因维度） |
| §7.3 证据不足降级 | insufficient_evidence 报告类型结构定义 | 替代伪造结论，含重新分析触发条件 |
| §7.4 禁止行为清单 | P-01~P-11 共 11 项硬禁止规则 | 三组：禁伪造结论(4项)、禁填补证据(4项)、禁降级阈值(3项) |
| §7.5 适用范围 | S1 逐单/批量、报告类型、根因状态 | S2 继承标记 |
| §7.6 章节关系 | 与 §2.4/§4.3/§5.2 的复用/扩展/互补/并列/消费关系矩阵 | — |

### 文件变更

| 文件 | 操作 | 变更量 |
|------|------|--------|
| `skills/reverse-analysis/SKILL.md` | modify（§7 占位符替换为完整章） | +约 200 行 |
| `process/returns/ST-NRA-01-evidence-threshold-guard.return.json` | create | 新建 |
| `process/evidence/ST-NRA-01-evidence-threshold-guard.index.json` | create | 新建 |
| `process/checks/CP6-ST-NRA-01-evidence-threshold-guard-CODING-DONE.md` | create | 新建 |

### SKILL.md 其他变更

- frontmatter version: `1.2` → `1.3`
- 实现状态章：追加 ST-NRA-01 已完成标记
- 禁止事项表：追加 §7 相关 8 项禁止操作（P-01/P-05/P-06/P-08/P-09/P-11 及互补项）
- 修订记录：追加 v1.3 行

### 未决点

| ID | 类型 | 描述 | 影响 | 下一动作 |
|----|------|------|------|----------|
| OPEN-NRA01-01 | OPEN | §7.1 P-04（valid_count<3 禁止生成 CA/PA）与 §3.2.5 Step IMP-1 前置条件双重门控 | 无冲突，冗余检查 | CP7 验证时确认不导致误判 |
| OPEN-NRA01-02 | OPEN | §7.5 声明 S2 不覆盖但 S2 继承 evidence_set 分类规则应继承 §7 硬阻断 | S2 实现时需确认 | 由 ST-RA-06.2 实现时处理 |

### 交接

- Story 状态: `ready-for-verification`
- 验证类型: 静态审查（Skill 文本定义，非可执行代码）
- 风险提醒: §7 是负向防护文本定义，无可执行代码或测试 fixture；禁止行为 P-01~P-11 的违规检测由 ptm-tse Agent 执行而非 Skill 自检
- Return Packet: `process/returns/ST-NRA-01-evidence-threshold-guard.return.json`
- Evidence Index: `process/evidence/ST-NRA-01-evidence-threshold-guard.index.json`
- 验证入口: 参见 CP6 检查 §6 验证入口

## ST-NRA-02: 权限边界拒绝与越权保护（外部访问/生产操作）

### 实现日期

2026-07-16

### Story 信息

- **Story ID**: ST-NRA-02
- **Feature**: FEAT-RA-ANALYSIS（F-021）
- **Wave**: 2
- **设计证据类型**: technical-note
- **来源**: CP5-CR030-DESIGN-EVIDENCE-BATCH（approved）
- **依赖**: ST-RA-01（资格检查，已完成）
- **实现证据**: `process/checks/CP6-ST-NRA-02-permission-boundary-denial-CODING-DONE.md` → PASS

### 实现摘要

在 `skills/reverse-analysis/SKILL.md` §8 替换占位符为完整权限边界拒绝与越权保护章（约 370 行）：

| 小节 | 内容 | 关键规则 |
|------|------|---------|
| §8.1 deny-by-default 总则 | 四类操作（外部连接/凭据/写入/分发）+ 三层防护（§1→§7→§8）+ 三类越权 | 未在允许表中的操作自动拒绝 |
| §8.2 外部访问拒绝 | 8.2.1 外部连接 / 8.2.2 凭据读取 / 8.2.3 HTTP 写入 | 每类独立检测逻辑 + 拒绝响应 + 来源追溯 |
| §8.3 生产操作拒绝 | 8.3.1 CA/PA 分发 / 8.3.2 下游任务 / 8.3.3 关闭工单 | 每类独立检测逻辑 + 拒绝响应 + 来源追溯 |
| §8.4 越权检测与阻断 | 三次二次检查 + 10 项检测规则表 + denial_record 结构 | DENIED-HTTP-ACCESS / DENIED-CREDENTIAL-READ / DENIED-CAPA-DISTRIBUTION 等 |
| §8.5 审计日志 | denial_record 即审计事件 + 存储位置 + §7 协同优先级 + P-12~P-19 | 新增 8 项禁止行为，总数达 19 项 |
| §8.6 拒绝矩阵速查 | 9 类请求的拒绝条件、type、响应码、替代路径 | 一站式查表 |
| §8.7-8.9 | 适用范围 + 章节关系 + 正向 LLD 覆盖追溯 | ST-RA-01/05.1/03/02 测试 ID + 接口定义 |

### 文件变更

| 文件 | 操作 | 变更量 |
|------|------|--------|
| `skills/reverse-analysis/SKILL.md` | modify（§8 占位符替换为完整章） | +约 370 行 |
| `process/returns/ST-NRA-02-permission-boundary-denial.return.json` | create | 新建 |
| `process/evidence/ST-NRA-02-permission-boundary-denial.index.json` | create | 新建 |
| `process/checks/CP6-ST-NRA-02-permission-boundary-denial-CODING-DONE.md` | create | 新建 |

### SKILL.md 其他变更

- frontmatter version: `1.3` → `1.4`
- shared_writers: ST-NRA-02 描述更新为「权限边界拒绝与越权保护」
- source_lld: 追加 ST-NRA-02 technical-note 引用
- 实现状态章：追加 ST-NRA-02 已完成标记
- 修订记录：追加 v1.4 行

### 未决点

| ID | 类型 | 描述 | 影响 | 下一动作 |
|----|------|------|------|----------|
| OPEN-NRA02-01 | OPEN | §8 是纯文本防护定义，越权检测的运行时执行由 ptm-tse Agent 负责 | Skill 自身不执行代码级检测 | CP7 验证时确认 Agent 层是否覆盖检测 |
| OPEN-NRA02-02 | OPEN | §8.4.2 部分检测项（HTTP 写入、凭据读取）依赖调用方在调用 Skill 前进行输入过滤 | Skill 层面为文本契约声明 | CP7 验证时确认 Agent 编排层调用前过滤 |

### 交接

- Story 状态: `ready-for-verification`
- 验证类型: 静态审查（Skill 文本定义，非可执行代码）
- 风险提醒: §8 与 §7 同为负向防护文本定义，无可执行代码或测试 fixture。越权检测（P-12~P-19）的运行时执行由 ptm-tse Agent 负责。§8 与 §7 的协同优先级为 §8 > §7（越权优先于证据不足）。
- Return Packet: `process/returns/ST-NRA-02-permission-boundary-denial.return.json`
- Evidence Index: `process/evidence/ST-NRA-02-permission-boundary-denial.index.json`
- 验证入口: 参见 CP6 检查 §6 验证入口

---

## ST-RA-06.2-REFRESH: S2 增量重算、差异报告

### 实现上下文

- **Story ID**: ST-RA-06.2-REFRESH
- **Feature**: FEAT-RA-ANALYSIS
- **Wave**: 3
- **LLD**: `process/stories/STORY-RA-06.2-REFRESH-s2-incremental-LLD.md` (full-lld v1.1, 14 章节)
- **输出文件**: `skills/reverse-analysis/SKILL.md`（追加 §9，替换占位符）
- **Tier**: Tier-A
- **前置依赖**: ST-RA-05.3-ANALYZE (§6 S1 管线)、ST-RA-06.1-DETECT (change_history 表)
- **CP6**: `process/checks/CP6-STORY-RA-06.2-REFRESH-s2-incremental-CODING-DONE.md` → PASS

### 实现摘要

在 `skills/reverse-analysis/SKILL.md` 中替换原 §9 "Artifact 交付与 Reviewer 确认"占位符，写入完整 S2 增量重算与差异报告章节（约 880 行，13 个子章节）：

| 子章节 | 内容 |
|--------|------|
| §9.1 | 增量重算触发：S2 入口判定（new_batch_id + baseline_batch_id + change_set）、change_set 消费（change_history 表 + 4 种 change_type 处理规则）、前置校验（4 项 baseline/new batch 存在性校验 + 批次锁定） |
| §9.2 | 受影响维度判定：变更字段→分析维度映射表（13 个 ticket 字段 × 6 个分析维度）、受影响维度计算流程（DM-1→DM-4）、受影响 ticket 集合三分类（affected/unaffected/skipped） |
| §9.3 | 增量重算策略：三大原则（最小重算、引用复用、聚合隔离）、逐维度增量策略表（6 维度独立策略）、增量重算执行流程（IR-1→IR-5 五步）、聚合维度增量合并规则（PQ/TM/IMP 三示例） |
| §9.4 | comparison_batch_ref 管理：5 种场景取值、对比批次合法性校验（存在性+同口径+窗口完整性 3 步）、analysis_run 写入示例 |
| §9.5 | recompute_mode 判定：判定流程图、规则版本变更检测（schema_version/mapping_version/analysis_rule_version）、full vs incremental 对比表、无受影响维度处理（空差异报告）、before/after 来源表、环比同比 N/A 7 条件判定 |
| §9.6 | 差异报告生成：差异计算流程（DR-1→DR-4）、空差异报告格式、显著变化阈值表（6 类变化×判定条件×默认阈值）、措施刷新提示（keep/complete/needs_review/invalidated 四建议） |
| §9.7 | 差异报告输出格式：完整 difference ra-report YAML 结构（change_summary + 6 维 dimension_diffs + comparison + improvement_refresh + skipped_tickets）、与 S1 报告的 8 项差异表、analysis_run 创建流程（created→in_progress→completed）、S2 管线 7 步总览 |
| §9.8 | 错误处理与降级：10 种异常场景×处理方式矩阵 |
| §9.9 | S2 安全与禁止事项：6 项 S2 专属禁止项（P-S2-01 至 P-S2-06）+ S2 允许操作范围表 |
| §9.10 | 与相邻模块的集成契约：上游 4 Story + 下游 3 Story + S1/S2 管线边界对比表 |
| §9.11 | 开放项与假设：5 项 OPEN（OPEN-RA062-01 至 05），含状态、重访条件和当前策略 |
| §9.12 | 测试设计：14 条测试场景（7 正向 + 7 负向/边界）+ 7 组 Fixture 设计 |
| §9.13 | 与 §6 S1 管线的协同：S1/S2 角色分配表 + 入口路由规则 |

### 设计契约映射

14/14 LLD 章节均有对应 §9 实现位置（详见 CP6 检查 §设计契约映射表）。

### 关键决策

1. **环比同比始终重算**：即使所有 ticket 字段不变，窗口时间推进后聚合结果也可能变化（不受 affected_dimensions 约束）
2. **规则版本变更检测保守策略**：任一版本不同（schema/mapping/analysis_rule）即触发全量重算（OPEN-RA062-01 未决策前采用最安全策略）
3. **显著变化阈值默认 20%**：可在 metric-definition.yaml 中覆盖（OPEN-RA062-05）
4. **change_set 通过 change_history 表消费**：ST-RA-06.1-DETECT 已确认格式（OPEN-RA062-03 RESOLVED）

### 文件变更

| 文件 | 操作 | 说明 |
|------|------|------|
| `skills/reverse-analysis/SKILL.md` | 修改 | §9 占位符（11 行）→ 完整 S2 章节（约 880 行, §9.1–§9.13）; version 1.4→1.5; shared_writers 更新; 修订记录追加 v1.5 |

### dev 中受影响的文件

无（仅追加 SKILL.md §9，不修改 data/、templates/ 或其他 Story 的文件）

### 交接

- Story 状态: `ready-for-verification`
- 验证类型: 静态审查（Skill 文本定义，非可执行代码）
- CP6 结论: PASS（16/16 检查项通过）
- 风险提醒: 聚合维度增量合并规则（§9.3.4）为声明式规范，运行时需实现对应 SQL 聚合逻辑；措施刷新提示（§9.6.4）依赖 ST-RA-03 产出已批准 CA/PA 列表，集成链路尚未端到端验证
- 验证入口: 参见 CP6 检查「后续交接」→「给 meta-qa 的验证入口」表（8 项入口）

---

## ST-RA-06.3-TRACK: 措施基线管理与刷新提示

- **Story ID**: ST-RA-06.3-TRACK
- **Feature**: FEAT-RA-TRACKING (F-023)
- **Wave**: 4
- **lld_policy**: technical-note
- **设计证据**: Story 卡片内嵌技术说明（8 章节）
- **前置依赖**: ST-RA-04（闭环跟踪）、ST-RA-06.1-DETECT（S2 变更检测）
- **文件所有权**: §5（shared_writers 已声明），模板文件独占创建
- **CP5**: `process/checks/CP5-ST-RA-06.3-TRACK-measure-baseline-refresh-LLD-IMPLEMENTABILITY.md` → PASS
- **CP6**: `process/checks/CP6-ST-RA-06.3-TRACK-measure-baseline-refresh-CODING-DONE.md` → PASS

### 实现摘要

在 `skills/improvement-tracker/SKILL.md` 中替换原 §5 "措施基线管理"占位符（4 行），写入完整 MeasureBaseline 管理与刷新提示章节（117 行，5 个子章节），并创建 `measure-baseline.yaml` 模板：

| 子章节 | 内容 |
|--------|------|
| §5.1 | MeasureBaseline 概念：HLD 七项可信治理契约之一、核心原则（无基线=needs-baseline 不判失效、reviewer 唯一状态变更者、不自动创建基线）|
| §5.2 | 基线创建：前置条件（proposal 已批准+Approved Input 已生成）、reviewer 操作步骤（创建 baseline_id→insert_measure_link→reviewer_update_measure_status）、4 种失败场景 |
| §5.3 | 刷新提示规则：6 种事件驱动的触发条件表（needs-baseline/needs-review/completed/active/needs-review(复发)/superseded）、系统写入约束（只能 update_measure_refresh_hint 写 proposed_status/refresh_hint）、reviewer 正式状态变更表（5 种操作路径）|
| §5.4 | reviewer 是唯一状态变更者：权限模型表（系统 vs reviewer 操作边界）、调用路径 ASCII 图（事件→系统提示→reviewer 决策）|
| §5.5 | 禁止行为：6 项硬禁止（不自动创建基线、不自动改 status、不判失效、不绕过 DAO 禁止字段、不修改下游、不修改 Approved Input）|

### 模板文件

| 文件 | 操作 | 内容 |
|------|------|------|
| `skills/improvement-tracker/templates/measure-baseline.yaml` | 创建 | schema 定义（12 字段：baseline_id→created_at）、约束汇总（11 条）、刷新提示枚举表（6 种触发条件）、状态迁移 ASCII 图、3 个示例（新建基线/系统写入提示/reviewer 变更） |

### 设计契约映射

技术说明 8 章节全部有对应实现位置：§5.1 落地设计依据、§5.2 落地基线创建流程、§5.3 落地刷新规则、§5.4 落地权限模型、§5.5 落地禁止行为，measure-baseline.yaml 落地 §3.1 Schema 定义。

### 关键决策

1. **proposed_status 与 status 分离**：系统只写提示字段（proposed_status/refresh_hint），正式 status 仅 reviewer 可变更。DAO 层 `update_measure_refresh_hint()` 硬性拒绝 status/approval_ref/baseline_id（ValueError），`reviewer_update_measure_status()` 是唯一正式状态变更路径。
2. **首版 version 固定为 1**：不实现自动递增，需多轮 S2 增量重算后才需要版本管理。
3. **needs-baseline 不判失效**：与 HLD 契约一致——无基线只表示未建立 MeasureBaseline，reviewer 确认后即可创建基线并转为 active。

### 文件变更

| 文件 | 操作 | 说明 |
|------|------|------|
| `skills/improvement-tracker/SKILL.md` | 修改 | §5 占位符（4 行）→ 完整章节（117 行, §5.1–§5.5）; shared_writers 更新; 实现状态更新; Gotchas #18-#22 追加; 修订记录追加 v1.2 |
| `skills/improvement-tracker/templates/measure-baseline.yaml` | 创建 | 完整模板（schema + 约束 + 枚举 + 迁移图 + 3 示例） |

### dev 中受影响的文件

无（仅追加 SKILL.md §5 + 创建 measure-baseline.yaml，不修改 §3/§4、data/dao.py 或其他 Story 的文件）

### 交接

- Story 状态: `ready-for-verification`
- 验证类型: 静态审查（Skill 文本定义 + YAML 模板，非可执行代码）
- CP6 结论: PASS（20/20 检查项通过）
- 风险提醒: 刷新提示的 6 种触发条件中，S2 增量重算触发路径（ST-RA-06.2-REFRESH 差异报告→needs-review）和闭环跟踪事件触发路径（ST-RA-04→completed/active）均为跨 Story 事件契约，集成链路尚未端到端验证；proposed_status 在 measure_link 表中的写入和读取需 CP7 阶段确认 DAO 层字段完整
- 验证入口: 参见 CP6 检查「后续交接」表（5 项入口 + 5 条 AC 验收映射）

---

## CR-037 Wave 1 设计证据修订（2026-08-05，设计审查结论落地）

- **Story ID**: CR037-P1 / CR037-S1 / CR037-S2 / CR037-S3（Wave 1 + 前置 P-1）
- **lld_policy**: P-1=technical-note；S1/S2/S3=full-lld
- **设计证据**: `process/stories/STORY-CR037-P1.md`（内嵌技术说明）、`STORY-CR037-S1-LLD.md`、`STORY-CR037-S2-LLD.md`、`STORY-CR037-S3-LLD.md`
- **CP5 自动预检**: 4 份 `process/checks/CP5-CR037-{P1,S1,S2,S3}-LLD-IMPLEMENTABILITY.md` → PASS；lld-check 全部 OK

### 修订项（对照审查结论逐项落地）

| 项 | 处理 |
|----|------|
| P0-1 find_topology_file 返回语义 | S1 返回 YAML 文件路径（写 `~/.ptm-team/cache/topology/{topo_id}.yaml`），对齐 `parser.py:70-116` 的 `detect_topology_format(filepath)` + `_parse_yaml_topology(filepath)` 消费契约；类型决策不再甩给 S7/S8 |
| P0-2 索引校验 | S1 `_load_index` 校验 `tgfw-topo-collection` 条目 compatible_consumers 含 ptm-te + topo_id 在 `_parse_collection_md` key 集合；不要求 node* 顶层条目；补 `test_all_12_topos` 命中率 100% |
| P0-3 台账 schema | P-1 台账模板改为现有 **list + `port_status: {port: {allocated_to, allocated_vlan}}`** schema（兼容 physical_pool.py）；S3 LedgerData 改 list + `ledger_by_id`（device_id 关联）；physical_pool.py 不改 |
| P1-1 拓扑数量 | 全部「13」→「12」：S1-LLD/S1 卡片/CR-037 §三层文件结构/HLD §1.2/§1.3/§4.1/§11；示例改用真实 `node5_dut3_tg1_sw1_link6`；`test_all_13_topos`→`test_all_12_topos` |
| P1-2 硬件系列数 | 全部「14」→「13」：S2-LLD（§5.2/§8.5/§11/§12/§15）/S2 卡片/CR-037 DQ-037-02/HLD §1.2/§4.2 |
| P1-3 消歧语义 | S2 `_disambiguate_model` 与 S3 `_resolve_hardware_platform` 统一 `hardware_platform` 取值 = **等价类**（HG3250/JL-HG/HD-D2000/D2000/nxp1046-16g/C3758）；conflicts 映射到等价类；aliases 增 `equiv_class` 字段 |
| P1-4 TG 归并 | S3 关联前 `_normalize_api_server`（去 scheme/统一 IP:port）；§6.5/§8.4/§13 灰区1 关闭 |
| P1-5 node_id | S3 `_merge_dut`/`_merge_tg`/`_check_devices_existence` 改 `pool_node.get('node_id', key)` |
| P2-3 platform_alias 主键 | S2 `platform_alias.yaml` 主键=规范系列名（乐研海光3250），`also_known_as` 收别名（海光3250/HG3250） |
| P2-4 importlib → PYTHONPATH | S1 §8.3/§13 灰区2 由 importlib 临时代码改为与 install.py 统一 PYTHONPATH（sys.path 过渡） |
| P2-5 model↔device_type | P-1 台账 model 用完整 device_type（同值直通）；S3 §5.3 直通规则 + 短型号规范化 |

### 未决点 / 开放项

- 无新增 blocking 项。S3 保留 O-S3-01（P-1 模板产物 schema 需在实现前对齐 UnifiedPool 数据类，非阻断）。
- 未产生新的 clarification queue item（本次为既有设计审查结论落地，无 blocks_lld 灰区）。

### 依赖与文件所有权

- P-1（前置 precedence）→ S3；S1/S2 无依赖。
- 文件所有权：S1 primary=`skills/topo-planning/src/topology_collection.py`；S2 primary=`skills/topo-planning/src/limit_parser.py`+`docs/limit-syntax.md`；S3 primary=`skills/topo-planning/src/pool_merge.py`；P-1 primary=`skills/topo-config/config/physical_pool.yaml`+`physical_ledger.yaml`+`tests/fixtures/physical_pool.sample.yaml`。无冲突。

### 待确认项

- Wave 1 四份设计证据修订完成，等待 host-orchestrator 收齐全部目标 Story 设计证据后发起统一 CP5 人工确认。

---

## CR-037 Wave 2 设计证据产出（2026-08-05，S4/S5/S6）

- **Story ID**: CR037-S4 / CR037-S5 / CR037-S6（Wave 2）
- **lld_policy**: S4=full-lld；S5=full-lld；S6=technical-note（Story 卡片内嵌）
- **设计证据**: `process/stories/STORY-CR037-S4-LLD.md`、`STORY-CR037-S5-LLD.md`、`STORY-CR037-S6.md`（内嵌技术说明）
- **CP5 自动预检**: 3 份 `process/checks/CP5-CR037-{S4,S5,S6}-LLD-IMPLEMENTABILITY.md` → PASS；lld-check 全部 OK

### 设计证据摘要

| Story | 模块 | 关键设计决策 |
|-------|------|-------------|
| S4 | `topo_mapper.py` 增强 | per-link `port_requirement`（None/全局/per-link 三态归一化）；`hardware_platform` 等价类 + `device_types` 候选过滤（P1-3）；端口兼容矩阵（GE=copper\|fiber，TE/XTE/QTE=fiber-only）；ledger 占用闭环 H1（`_is_port_busy` 消费 `allocated_to` + 成功写回 `allocate` + 新增 `PhysicalPool.release(env_name)`，S8 挂接编排 [7]）；约束叠加 H4（用户 > 逻辑自带 media_type+端点 speed_class P2-2 > 默认，并集语义）；`mapping_validator.py` R2 per-link 聚合不静默跳过（P1-6）；include-at-least-one 软约束 + post-match ≥1 |
| S5 | `exporter.py` 新建 | env-file 四段（metadata+port_mapping+nodes+links）+ 9 类占位符硬契约（ADR-09）+ 端口引号化（Gotcha #11）；命名默认 `topology/topology.yaml` + `--output` 描述性命名；link 数一致性（口径不含 Mock 端点）；凭据禁止（ADR-02）；结构化失败报告 JSON+人读 + `--report-out`（O-c）；`ip_plan` 可选接入（S6 降级） |
| S6 | `exporter.py#IP 规划段` | 复用 `ipam.IPAM`（198.18.0.0/15 自动 / per-broadcast-domain preset）+ `topology_exporter._compute_broadcast_domains`（含 DUT-DUT L3 直连域）；新增 `compute_ip_plan` 桥接；容量不足结构化错误 |

### 未决点 / 开放项

- 无新增 blocking 项。S4 保留灰区已闭合（include-at-least-one 软约束 / H4 并集 / release 归属 / validator 聚合均已在 §13 记录取舍）。
- 未产生新的 clarification queue item（Wave 2 无 blocks_lld 灰区）。

### 依赖与文件所有权

- S4=[S1,S2,S3] precedence；S5=[S4] precedence；S6=[S5] precedence（Wave 2 串行）。
- 文件所有权：S4 primary=`topo_mapper.py#per-link 约束 + ledger 闭环`，范围含 `mapping_validator.py`（P1-6）+ `physical_pool.py#release`（P1-7，加性方法，台账 schema 不变 P0-3）+ `topology_model.py`；S5 primary=`skills/topo-planning/src/exporter.py`；S6 primary=`skills/topo-planning/src/exporter.py#IP 规划段`。S5↔S6 同文件但串行，无冲突。

### 待确认项

- Wave 2 三份设计证据产出完成（含 CP5 自动预检），与 Wave 1 一并等待 host-orchestrator 收齐全部目标 Story 设计证据后发起统一 CP5 人工确认。

---

## CR-037 Wave 3 设计证据产出（2026-08-05，S7/S8/S9）

- **Story ID**: CR037-S7 / CR037-S8 / CR037-S9（Wave 3）
- **lld_policy**: S7=full-lld；S8=full-lld；S9=technical-note（Story 卡片内嵌）
- **设计证据**: `process/stories/STORY-CR037-S7-LLD.md`、`STORY-CR037-S8-LLD.md`、`STORY-CR037-S9.md`（内嵌技术说明）
- **CP5 自动预检**: 3 份 `process/checks/CP5-CR037-{S7,S8,S9}-LLD-IMPLEMENTABILITY.md` → PASS；lld-check 全部 OK

### 设计证据摘要

| Story | 模块 | 关键设计决策 |
|-------|------|-------------|
| S7 | `topo-planning` skill + CLI | skill 向导 5 步（展示池→选/建逻辑 topo→映射→导出→部署/验证）；CLI `topo-plan` 10 参数（--topo-id/--case-file/--limit/--limit-file/--devices-yaml/--pool-path/--ledger-path/--output/--report-out/--dry-run）；`--case-file` 双源提取 topo_id（E2）；NLP→CLI 由 agent 转换（G5/DQ-037-06，引用 S2 limit-syntax.md）；import 集成统一 PYTHONPATH（S1 P2-4，无 importlib）；默认 dry-run（不写台账/不落盘/打印 would-allocate 清单） |
| S8 | ptm-te 编排 + install.py | topo_id 双源提取（组网约束优先/组网描述括号回退，E2）；编排 [1]→[2] 间插入 [1.5] 拓扑映射步骤（E1）；`PTM_TE_SKILLS` 5→7 增加 topo-config + topo-planning（方案 A）；首次生成目标项目模板副本（P1-8：源=skills/topo-config/config/、topology/ 目录、幂等跳过、.gitignore 追加 topology/*.yaml、不含测试造数、台账 list+port_status P0-3）；P1-7 release 挂接编排 [7] 清理（teardown/环境删除） |
| S9 | `deploy_bridge.py` 新建 | 复用 `EnvironmentDeployer`（deployer.py）；`DeployBridge` 类 plan/deploy/verify/teardown；`--execute` 为独立 runtime_authorization 决策项（O3/DQ-037-04），`authorized` gate 未授权抛 `RuntimeAuthorizationError`；dry-run 0 次真机写；凭据仅经 fw_password_env 环境变量名 |

### 未决点 / 开放项

- 无新增 blocking 项。S7/S8/S9 灰区均已闭合（S7：topo-id vs case-file 优先级 / dry-run 不写台账 / NLP→CLI 归属 / env-file 落盘时机 / 模板副本源；S8：双源优先级 / CLI 调用方式 / 模板源 / release 时机 / .gitignore 粒度；S9：--execute gate / 桥接边界 / env-file 兼容）。
- 未产生新的 clarification queue item（Wave 3 无 blocks_lld 灰区）。

### 依赖与文件所有权

- S7=[S1,S2,S3,S5,S6] precedence；S8=[S7] precedence；S9=[S5,S7] precedence（Wave 3 串行）。
- 文件所有权：S7 primary=`skills/topo-planning/SKILL.md`+`scripts/topo_plan_cli.py`+`templates/*.example`；S8 primary=`agents/ptm-te.md#编排流程 [1]→[2]`+`script/install.py#PTM_TE_SKILLS`；S9 primary=`skills/topo-planning/src/deploy_bridge.py`。三 Story 文件互斥，无冲突（CP4 已确认 Wave 3 串行）。

### 待确认项

- Wave 3 三份设计证据产出完成（含 CP5 自动预检），与 Wave 1/2 一并等待 host-orchestrator 收齐全部目标 Story 设计证据后发起统一 CP5 人工确认。

## CR-037 CP5 综合审核 NEEDS_REWORK 修订（2026-08-05，S1/S2 数据层修正组）

- **Story ID**: CR037-S1（topology_collection）/ CR037-S2（limit_parser）
- **触发**: CP5 综合审核返回 NEEDS_REWORK（2 BLOCKER + 25 MAJOR），S1/S2 数据层修正组按修订路径打回 meta-dev。
- **修订证据**: `process/stories/STORY-CR037-S1-LLD.md` / `STORY-CR037-S2-LLD.md`（v0.4）+ Story 卡片 `STORY-CR037-S1.md` / `STORY-CR037-S2.md`（acceptance_criteria / validation_context / AI 任务清单同步）+ CP5 自动预检文件 `process/checks/CP5-CR037-S{1,2}-LLD-IMPLEMENTABILITY.md` 追加 v0.4 修订说明。

### S1（topology_collection）修订项

| 项 | 修订 | 落点 |
|----|------|------|
| S1-A list_topologies 候选集来源 | 候选集改以 `_parse_collection_md` 解析出的 topo_id key 集合为准（非 index.yaml 顶层，index 仅 2 个 collection 级资源条目），`list_topologies()` 返回 12 个 node* 拓扑 | §3/§6.3/§10/§11/§15 + Story 卡片 |
| S1-B §5 示例失真 | 核对 topology-collection.md 实况：仅 `node2_dut1_tg1_link2` 含 speed_class/media_type 全字段，其余 11 个为简写 `{link: linkN}`；修正示例 + 注明缺省字段由推导/下游补充，不虚造 | §5/§10/§12/§15 + Story 卡片 |

### S2（limit_parser）修订项

| 项 | 修订 | 落点 |
|----|------|------|
| S2-A "10GE" 不在枚举 | speed_class 别名表：`10GE/万兆/TE`→`TE`、`QTE/40GE`→`QTE`、`TTE/25GE`→`TTE`；语法规范 + 单测覆盖 | §5.2/§6.5/§8.5/§10/§15 |
| S2-B TG 平台无法展开 | `node_type: TG` 独立分支映射 sub_type/hardware_platform（ixia-c/trex × EP/C236/J1900，6 组合），不套 DUT 型号表 | §3/§5.1/§6.4/§8.5 |
| S2-C link_id vs link | 字段名统一为 HLD 的 `link`，与 collection links 命名一致 | §5.1/§6.1/§7/§8.5 |
| S2-D nxp1043 等价类不精确 | 拆分 `nxp1043-4g`(→160) vs `nxp1043-8g`(→160-PRO/290/490/690)；`equiv_class_group` 反查 + 消歧说明修正 | §5.2/§6.2/§8.1/§8.5 |
| S2-E A1500-HU 消歧决策 | 改为「用户平台约束等价类 + devices.yaml 实际设备」联合决策，非首中；conflicts 逐一给消歧策略 | §6.3/§8.3/§8.5 |
| S2-F 保持 | 多节点 all 语义（H3）/ 空 limit 默认（O-a）/ 端口兼容矩阵（GE=copper\|fiber，TE/XTE/QTE=fiber-only）保留 | 全文保持 |

### 校验状态

- `meta-flow story lld-check`（full-lld）：S1 **OK** / S2 **OK**（均 PASS）。
- CP5 自动预检文件状态保持 PASS 并追加 v0.4 修订说明；重新发起 CP5 由 host-orchestrator 统一执行。

## CR-037 CP5 综合审核 NEEDS_REWORK 修订（2026-08-05，S7/S8/S9/S10 集成/部署/测试组）

- **Story ID**: CR037-S7（topo-planning skill + CLI）/ CR037-S8（ptm-te 集成 + install.py）/ CR037-S9（deploy_bridge）/ CR037-S10（测试）
- **触发**: CP5 综合审核返回 NEEDS_REWORK（2 BLOCKER + 25 MAJOR），集成/部署/测试组按修订路径（S1+S7+S9+S10 / S8+P-1 的 S7-S10 部分）打回 meta-dev。
- **修订证据**: `STORY-CR037-S7-LLD.md` / `STORY-CR037-S8-LLD.md`（v1.1）+ `STORY-CR037-S9.md` / `STORY-CR037-S10.md`（technical-note v1.1）+ Story 卡片 `STORY-CR037-S{7,8,9,10}.md`（dev_context / validation_context / acceptance_criteria / output_files / AI 任务清单同步）。

### S7（topo-planning skill + CLI）修订项

| 项 | 修订 | 落点 |
|----|------|------|
| S7-A FR8 与流程不一致 | CLI `--execute` 边界 = 写台账+落盘 env-file；真机部署由 S9 deploy_bridge 独立入口承接（CLI 不直接调 deployer）；FR8/向导 Step 5/一致性说明对齐 | §2 FR8/§6/§7.2/§13 |
| S7-B --execute 无 --authorized 硬门 | `--execute` 必须配 `--authorized`（DQ-037-04 分层：台账/落盘=workspace 写执行前自动步骤），未授权抛 RuntimeAuthorizationError（exit 3）；dry-run 0 次真机写 | §2 FR12/NFR7/§6.2/§6.5/§7.1/§7.3/§9/§10/§15 |
| S7-C 保持 | dry-run 默认 / NLP→CLI agent 转换（G5/DQ-037-06）/ PYTHONPATH 集成（S1 P2-4）不变 | 全文保持 |
| S7 联动 S8-B | dry-run + `--output` 指向 run 目录 → 落预览 env-file（metadata.executable:false），不写真实 topology/ 路径 | §2 FR5/FR7/§7.1/§8.3/§13 灰区 7 |

### S8（ptm-te 集成 + install.py）修订项

| 项 | 修订 | 落点 |
|----|------|------|
| S8-A 造数隔离击穿（BLOCKER） | copy_skill_tree 从整树 copytree 改为过滤复制：排除 tests/、__pycache__/、*.pyc、*.pyo；模板源占位 + 裁剪双保险 | §6.1b/§8.4/§10/§11/§14/§15/§13 灰区 6 |
| S8-B dry-run 不落盘 vs 下游消费（BLOCKER） | dry-run 落预览 env-file 到 runs/<run-id>/topology.preview.yaml（executable:false），下游 load_env_file 可解析；不落真实 topology/ 路径 | §2 FR3/§5.2/§6.4/§7.2/§8.5/§13 灰区 7 |
| S8-C 不装层2 collection 资源 | component-resource-links.yaml 增 ptm-te→network-topology（tgfw-topo-collection required）；output_files/ownership 扩展 | §2 FR12/§6.6/§7.1/§5.4/§10/§13 灰区 8 |
| S8-D 模板双真相源 | 模板副本源唯一 = skills/topo-config/config/（P-1 后 device_type 全占位）；S7 templates 仅人工参考，install 不读取 | §5.3/§8.4 |
| S8-E 授权门错配（评审 D 组） | 授权分层重写：映射/台账占用=执行前自动步骤（不需 DQ-037-04）；真机下发/EnvironmentDeployer（S9）=DQ-037-04 独立授权；[1.5] 不要求真机下发授权 | §6.4/§7.2/§9/§15 |
| S8-F .gitignore 粒度 | 沿用 topology/*.yaml 单条；runs/ 运行态不入库，预览副本不需新增忽略项 | §13 灰区 5 |

### S9（deploy_bridge）修订项

| 项 | 修订 | 落点 |
|----|------|------|
| S9-A 构造注入不可行 | EnvironmentDeployer 构造参数仅路径，无法注入 env/devices/broadcast_domains；新增 _build_deployment_view() 转换层从 env-file + physical_pool 推导三段 | 设计依据/接口变化/数据变化 |
| S9-B env-file vs deployer 消费结构不匹配（评审 D 组） | env/devices/broadcast_domains 由 port_mapping/nodes/links + 池推导（env.name←metadata、devices←nodes+池、broadcast_domains←links+池）；推导失败抛 DEPLOY_VIEW_INCOMPLETE，不崩溃/不静默空转 | 数据变化/异常失败回退 |
| S9-C 独立运行台账不释放 | 台账释放由 S4 PhysicalPool.release + S8 编排 [7] 承载，S9 不重复实现；独立运行调用方自行承担 | 设计依据/teardown/风险表 |

### S10（测试）修订项

| 项 | 修订 | 落点 |
|----|------|------|
| S10-A 补专项单测 | H4 合并语义（TE+copper 不可满足）/ include-at-least-one 软约束（池仅 1 条 TE 链路）/ L2 透传 link 不计入 LINK_COUNT_MISMATCH 三项专项单测 | 技术说明「层 A 专项单测」 |
| S10-B 层B 第 10 条 N/A | HLD §11 层B 第 10 条（--execute 真实执行）标注 N/A（依赖 DQ-037-04 独立授权），不默认执行 | 技术说明「层 B」/权限变化 |
| S10-C fixture 所有权去重 | tests/fixtures 造数所有权明确归 P-1，S10 只消费不重复声明（output_files 移除该文件） | frontmatter output_files/技术说明「层 A」 |

### 校验状态

- `meta-flow story lld-check`：S7 full-lld **OK** / S8 full-lld **OK** / S9 technical-note **OK** / S10 technical-note **OK**（均 PASS）。
- CP5 自动预检文件（S7/S8 IMPLEMENTABILITY）追加 v1.1 修订说明；重新发起 CP5 由 host-orchestrator 统一执行。

## CR-037 CP5 第二轮复审 Minor 修订（2026-08-05，S3/S8/P-1/S10 补修）

- **Story ID**: CR037-S3（pool_merge）/ CR037-S8（ptm-te 集成 + install）/ CR037-P1（物理池模板化）/ CR037-S10（测试）
- **触发**: CP5 第二轮复审发现 4 项关键 Minor（Minor-1 host 兜底伪代码 / Minor-2 S8 模板安全隐性依赖 P-1 / Minor-3 P-1 模板注释残留 / Minor-4 S10 断言 case-sensitive），本 meta-dev 线程补修。
- **修订证据**:
  - `process/stories/STORY-CR037-S3-LLD.md`（v1.2，§13 灰区1 决策表 + §10 单测 `test_tg_host_fallback_decision_table`）+ `STORY-CR037-S3.md`（TASK-S3-06 决策表约束）
  - `process/stories/STORY-CR037-S8-LLD.md`（v1.2，frontmatter `depends_on` 显式含 P-1 + §6.2 `_assert_p1_template_placeholderized` 前置校验 + §10 单测 + DoD/发布依赖）+ `STORY-CR037-S8.md`（depends_on + 依赖说明）+ `process/DEVELOPMENT-PLAN-CR-037.yaml`（S8 `depends_on=["S7","P-1"]`）
  - `process/stories/STORY-CR037-P1.md`（v1.2，模板 schema 注释清理为纯占位 `${HARDWARE_PLATFORM}`/`${DEVICE_TYPE}`，示例块统一标注「示例/占位，勿直接使用」，fixture 为唯一造数位置）
  - `process/stories/STORY-CR037-S10.md`（technical-note 增「模板占位化断言（Minor-4）」case-sensitive 精确特征串清单，acceptance_criteria/文件影响/异常回退同步）

### Minor 处理明细

| Minor | Story | 处理结果 | 落点 |
|-------|-------|---------|------|
| Minor-1 host 兜底伪代码首组误命中 | S3 | §13 灰区1 伪代码改为显式决策表（api_server 主键 → host 兜底 → unmatched）+ 两遍完整循环（先遍历全部 candidates 找 api_server 主键，再遍历找 host 兜底），消除首组 host 兜底误命中遮蔽后续 api_server 主键候选；§10 新增 `test_tg_host_fallback_decision_table`；语义与 §6.5 `_find_device_group_by_api_server` 对齐 | S3-LLD §13/§10/修订记录；S3 卡片 TASK-S3-06 |
| Minor-2 S8 模板安全隐性依赖 P-1 | S8 | `depends_on` 显式含 CR037-P1（LLD frontmatter + Story 卡片 + DEVELOPMENT-PLAN 三处一致，precedence）；§6.2 新增 `_assert_p1_template_placeholderized` 前置校验（源含 A1600-HU/hg3250 等特征串即 fail-fast，防止 P-1 未落地时模板副本泄漏真实型号+密码）；§10 新增 `test_generate_templates_requires_p1_placeholder`；DoD/发布依赖同步 | S8-LLD frontmatter/§6.2/§10/§14/§15；S8 卡片依赖；DEVELOPMENT-PLAN-CR-037.yaml |
| Minor-3 P-1 模板注释残留 | P-1 | 模板 schema 代码块中 `HG3250`/`A1300-HU`/`A1600-HU` 全部清理为纯占位（`${DEVICE_TYPE}`/`${HARDWARE_PLATFORM}`/`${MODEL}`）；示例块统一标注「示例/占位，勿直接使用」且不进入模板正文/模板副本；明确 fixture 为唯一允许造数位置 | P1 卡片技术说明「模板 Schema 设计」/关键改动/P2-5/fixture/acceptance_criteria/修订记录 |
| Minor-4 S10 断言 case-sensitive | S10 | 模板占位化断言改为 case-sensitive 精确特征串清单（`A1600-HU`/`A1300-HU`/`A1500-HU`/`HG3250`/`hg3250`/`IXIA-C`/`Ngfw@123`/`10.113.` 逐字精确匹配），断言对象限定 config 模板 + 模板副本，fixture 不纳入扫描 | S10 卡片「模板占位化断言（Minor-4）」/acceptance_criteria/文件影响/异常回退/设计依据 |

### 校验状态

- `meta-flow story lld-check`：S3 full-lld **OK** / S8 full-lld **OK** / P-1 technical-note **OK** / S10 technical-note **OK**（均 PASS）。
- 本轮为 CP5 第二轮复审补修；统一人工确认仍由 host-orchestrator 发起。

## CR-037 CP5 第二轮复审 S5/S1/S2 修订（2026-08-05，R1/R2/R3/R4 + Minor）

- **Story ID**: CR037-S5（exporter）/ CR037-S1（topology_collection）/ CR037-S2（limit_parser）
- **触发**: CP5 第二轮复审 4 项 MAJOR 残留（R1 S5 多节点/SW env-file 规则、R2 S5 NFR2 确定性 vs generated_at、R3 S1 §7 模糊搜索旧表述、R4 S2 消歧未接入主流程）+ 1 项 Minor（"海光"简写首中杰伦海光）。R1 已定范围决策。
- **修订证据**:
  - `process/stories/STORY-CR037-S5-LLD.md`（v0.5）+ `STORY-CR037-S5.md`
  - `process/stories/STORY-CR037-S1-LLD.md`（v0.5）+ `STORY-CR037-S1.md`
  - `process/stories/STORY-CR037-S2-LLD.md`（v0.5）+ `STORY-CR037-S2.md`
  - CP5 IMPLEMENTABILITY 检查文件三份（S5/S1/S2）追加第二轮复审修订说明
- **lld-check**: S5/S1/S2 full-lld 均 **OK（PASS）**。

### R1-R4 / Minor 处理明细

| 项 | Story | 处理结果 | 落点 |
|----|-------|---------|------|
| R1 多节点/SW 拓扑 env-file 规则（已定范围决策） | S5 | MVP = 单 TG + 单 DUT 直连（node2_dut1_tg1_linkN 系列）；新增 `_validate_mvp_topology` 校验节点 node_type 分布，超边界（≥2 DUT / 含 SW/Mock/PC）显式抛 `TOPO_NODE_COUNT_UNSUPPORTED` + 结构化明细，不产出 nodes 缺 dutN 坏产物；HLD §1.3 非目标由 host-orchestrator 补充 MVP 边界（不改架构） | S5-LLD §0/§1 目标7/§3/§6.1/§7.1/§7 错误路径/§8.5/§10（test_multinode_topology_rejected/test_sw_topology_rejected/test_mock_topology_rejected/test_mvp_single_dut_passes）/§11 TASK-S5-04b/§12/§13 灰区0/§15 |
| R2 NFR2 确定性 vs generated_at | S5 | `generated_at` 移出确定性比较范围（剔除后比较）或 `_build_metadata(generated_at=...)` 注入固定值；NFR2/§3/§7.1/§11/§13 灰区0b/§15 同步；单测 test_export_deterministic_ignore_generated_at + test_export_deterministic_inject_generated_at | S5-LLD §2 NFR2/§3/§7.1/§10/§11 TASK-S5-11/§13/§15 |
| R3 §7 模糊搜索旧表述残留 | S1 | 候选集明确为 `_parse_collection_md` 解析出的 12 个 node* topo_id（同步 §6.3）；删除不存在的 `node4_dut1_tg1_link4` 示例，改用真实候选 node2_dut1_tg1_link3/link2/link5；§10 test_fuzzy_suggestion 用真实候选集 | S1-LLD §0/§7/§10/§15；修订记录 v0.5 |
| R4 消歧未接入主流程 | S2 | `parse_limit` 新增 `pool_devices` 入参；`_parse_device_constraints` 对 conflicts 中 device_type 逐一调 `_disambiguate_model`，`resolved_equiv_class`/`disambiguation` 写入 DeviceConstraint（输出契约含消歧后 device_type 集合）；`pool_devices=None` 且无显式 platform → AMBIGUOUS_MODEL；NFR3 确定性依赖关系修正；§7 流程图 + §8.1b 调用链 + §10 test_disambiguate_in_main_flow* | S2-LLD §0/§1/§2 FR8+NFR3/§3/§5.1/§6.1/§6.3/§7/§8.1b/§10/§11/§12/§13 灰区7/§15 |
| Minor "海光"简写首中杰伦海光 | S2 | `_expand_platform` 完整平台名优先精确匹配（"海光3250"→乐研海光3250）；简写"海光"多命中抛 PLATFORM_AMBIGUOUS 或按 pool_devices 实际设备交集消歧，非首中；FR12 + §5.2 注释/§6.2/§8.1/§10/§13 灰区8 | S2-LLD §0/§1/§2 FR12/§5.2/§6.2/§8.1/§10/§12/§13 灰区8/§15 |

### 范围与计数

- S5 单元测试 22→28 用例、fixture 3→6（新增单 DUT/多 DUT/含 SW 逻辑拓扑 fixture）；实施 ~11h→~11.5h。
- S2 单元测试 32→40 用例、fixture 2→3（新增 devices_pool_ambiguous.yaml）；实施 ~13h→~15h。
- S1 单测用例计数维持 12（表格含 13 行 test_，DoD 沿用既有 12 表述，R3 不改计数口径）。
- 本轮不修改 REQUIREMENTS.md / HLD.md / ARCHITECTURE-DECISION.md（LLD 层契约闭合 + Story 卡片同步；HLD §1.3 MVP 边界补充由 host-orchestrator 执行）。
- 状态：LLD 修订完成，等待 host-orchestrator 收齐本轮目标 Story 设计证据后统一重新发起 CP5 确认。

## CR-037 Wave 4 实现（2026-08-05，S10 测试 + S11 文档）

- **Story ID**: CR037-S10（测试，technical-note）/ CR037-S11（文档，waived）
- **触发**: CP5 已 approve；Wave 0-3（P-1/S1-S9）实现完成，Wave 4 进入实现。
- **实现证据**: `tests/test_topo_plan_cr037.py`（S10 层A 集成测试 20 用例）+ `skills/topo-config/tests/fixtures/{pool_legacy,ledger_legacy}.sample.yaml`（既有测试适配 fixture）+ `skills/topo-planning/tests/fixtures/{case_ipv4_policy_route,devices_direct}.*`（参考用例端到端 fixture）+ `skills/topo-planning/README.md` + `docs/ptm-te/执行指导.md`（[1.5] 拓扑映射）+ `process/checks/CP6-CR037-S10-test-fixture-two-layer-CODING-DONE.md`。

### S10（测试）实现明细

| 项 | 结果 |
|----|------|
| 层A 8 条完成准则 | 全部覆盖：collection 12 命中 100%（`test_collection_12_topos_hit_rate_100`）/ limit 解析（all + 3 链路模式 + 13 平台别名 + 空 limit 默认）/ per-link TE 约束（含 TE 成功 + 无 TE 结构化失败 + `--report-out`）/ ledger 闭环（allocate 写回 / 避让 / release）/ 约束叠加 + env-file `load_env_file` + `LINK_COUNT_MISMATCH` / install dry-run（PTM_TE_SKILLS 7 + 模板副本不含造数 + copy_skill_tree 裁剪 tests/）+ Minor-4 case-sensitive / 参考用例 IPv4策略路由端到端 dry-run |
| 3 项专项单测（S10-A） | `test_h4_te_copper_unsatisfiable`（TE+copper 池 + specific 硬约束 → {TE,fiber} 不可满足 → 结构化失败）/ `test_include_at_least_one_soft_constraint`（pool_direct 仅 link2 TE fiber → at least 1 命中即成功）/ `test_l2_transparent_link_not_counted`（L2 透传 link 排除 LINK_COUNT_MISMATCH + 不进 env-file links） |
| 28 项既有失败适配 | 基线 28 failed/54 passed（P-1 模板占位符）；新建 legacy fixture（结构与 P-1 前真机池一致），5 个既有测试数据源切换 → **82 passed/0 failed**（与 P-1 前真实池 82 passed 一致，语义不变） |
| Minor-4 前置修复 | `skills/topo-config/config/physical_pool.yaml` TG1 `device_type` 注释删除示例型号 `IXIA-C`（S8 `_MODEL_FEATURE_TOKENS` 不含 IXIA-C；S10 严格 case-sensitive 断言必须清理） |
| 层 B / 第 10 条 | 层 B P-2 未就绪 → 未验证风险，不阻断 CP7 fixture 结论；`--execute` 真实执行 N/A（DQ-037-04，S10-B） |

### 全量测试结果

```
uv run --python 3.11 python -m pytest skills/topo-config/tests/ skills/topo-planning/tests/ tests/test_topo_plan_cr037.py tests/test_install_templates.py tests/test_install_mapping.py
271 passed / 0 failed（4.60s）
```

- `skills/topo-config/tests/`：82 passed（含适配后既有 28）
- `skills/topo-planning/tests/`：141 passed
- `tests/test_topo_plan_cr037.py`：20 passed
- install tests：28 passed

### 参考用例 dry-run 链路结果

`topo_plan_cli.py plan --case-file case_ipv4_policy_route.md --pool-path pool_direct_tg_dut.yaml --ledger-path ledger_direct.yaml --devices-yaml devices_direct.yaml --dry-run`：

- Step 2 选用 `node2_dut1_tg1_link3`（来源: 组网描述，E2 双源提取）
- Step 3 映射 matched（2 设备 / 3 链路直连）
- 导出 env-file 四段（metadata/port_mapping/nodes/links），经 case-execution `load_env_file` + `build_env_topology` 校验通过（nodes: dut1/tg1, links: 3）
- dry-run 0 次真机写、不写台账

### S11（文档）实现明细

| 文件 | 说明 |
|------|------|
| `skills/topo-planning/README.md` | 新建：安装 / 三层文件结构 / 使用（向导 5 步 + CLI）/ limit 语法（引用 S2 docs/limit-syntax.md）/ 安全与授权 / 相邻对象边界 / Gotchas |
| `docs/ptm-te/执行指导.md` | 补充 [1.5] 拓扑映射：§5.2 八步编排流程表增 [1.5] 行 + §5.2.1 说明（E2 双源提取 / execute vs dry-run / 授权分层 S8-E / 异常路径）+ §6.1 产物目录补 `topology.preview.yaml` + 「版本与已知偏差」补 CR-037 说明 |

### 状态

- S10/S11 更新为 `ready-for-verification`；CP6 证据写入 `process/checks/CP6-CR037-S10-test-fixture-two-layer-CODING-DONE.md`（S11 waived 用卡片技术说明）。
- return packet / evidence index：`process/returns/STORY-CR037-S{10,11}.return.json` + `process/evidence/STORY-CR037-S{10,11}.index.json`。
- 等待 host-orchestrator 拉起 meta-qa CP7 验证。

## STORY-039-01: G1 静态路由回滚 verify 兜底（CR-039 / W1）

- 调度批次：CR-039 Wave 1（与 STORY-039-04 并行，文件互斥）；design_evidence_confirmed=true（CP5 approve technical-note，DQ-039-01=方案 A）。
- 依赖类型：none（W1 起点）；文件所有权：primary=op_mapper.py + test_op_mapper_rollback.py，无冲突。

### 实现摘要

- `skills/policy-route-execution/scripts/op_mapper.py` 三处 G1 区块改动（以 canonical HEAD 逐块合并，非整文件覆盖）：
  1. L705-707 注释语义更新：『step-refs 或 verify 查询兜底』->『无条件 verify 查询获取；匹配失败即 EXEC_FAILED，无 step-refs fallback（DQ-039-01 方案 A）』。
  2. 新增 `_query_static_route_id()`（L2340-2395）：dst_network 必填；经 `fw_verify_static_route`（page=1/size=100）查询；vals 取 data.response.vals 或 data.vals；next_hop_addr 含 `${` 占位符时跳过匹配；多路由取首个非空 id；无匹配返回 None。
  3. `handle_rollback` inverse_op 分支（L2448-2478）：`fw_config_static_route` 无条件优先 verify 查询（放弃 step-refs fallback）；rid None -> EXEC_FAILED envelope（reason 含 dst_network 与『无法清理』）；匹配 -> `fw_delete_static_route --id`（透传 env_topology/tg_api_server）。
- `tests/test_op_mapper_rollback.py` 新增 `HandleRollbackStaticRouteVerifyTests`：SM-039-02 四分支（有匹配 / 多路由 next_hop_addr 消歧 / 无匹配 EXEC_FAILED / 占位符跳过）+ 扩展 data.vals 形态，mock execute_op（static-only）。

### 关键决策与偏差

- 无偏离：handle_rollback 静态路由分支与工作区 L2729-2754 逐行一致；`_query_static_route_id` 逻辑（剔注释）与工作区 L2618-2671 逐行一致。仅分支前导注释多 2 行（补 DQ-039-01/CR-039 归属标注，注释级）。
- 逐行 diff 对拍通过（dict 内容级差异人工核对，遵守 CR-039 security_constraints 回源方式）。

### 未决点

- 无 blocks_lld 项；无 clarification queue item（DQ-039-01 已 approve）。
- RA-039-01（大路由表 verify 超时 / dst_network 歧义）：next_hop_addr 消歧 + 单测缓解；真机超时按 DQ-039-01 备选 B 重评，不静默改。

### 单测与验证结果

- `uv run --python 3.11 pytest tests/test_op_mapper_rollback.py -x`：**32 passed**（原 27 + 新增 5）。
- 回归 `tests/test_tg_op_mapping.py` + `tests/test_op_mapper_pppoe_client.py`：**55 passed**；`validate_mapping_consistency` 不回归，EXPECTED_OP_COUNT=44（44->45 归 STORY-039-03）。
- 未触碰：G4 resolve_env_refs（STORY-039-02）、G5 sw3 族（STORY-039-03）、INTENT_TO_OP/ARGS_TO_FLAGS dict、case_runner.py、SKILL.md（工作区 SKILL.md v1.7/v1.8 改动属并行 STORY-039-04，本线程未触碰）。

### 提供给 meta-qa 的验证入口

- 入口：`uv run --python 3.11 pytest tests/test_op_mapper_rollback.py`（32 passed）；回归上述两文件（55 passed）。
- 证据：`process/stories/STORY-039-01-IMPLEMENTATION.md` + `process/returns/STORY-039-01.return.json`。
- 风险提示：真机 verify 超时未覆盖（static-only CR，DQ-039-04 不授权真机），属 RA-039-01。

### CP6 编码完成门

- Story 状态：dev-complete（待 CP6）；实现执行证据闭环（对象清单 / 契约映射 / 测试计划 / 最小切片 / 本地对拍 / 验证结果 / 未覆盖项 / 交接摘要）见 IMPLEMENTATION.md。
- G6 M9 归属（T-3 台账项）与 sw3 用例样例来源（工作区 CR-046 node3_dut1_tg1_sw1_pppoe_link3）已在 IMPLEMENTATION.md 留痕补记。

---

## 2026-08-19 CR-039 STORY-039-04（文档与追溯）dev-complete

- **Wave**: W1（与 STORY-039-01 并行，文件互斥）；meta-dev；validation_mode=static-only + review-only。
- **改动文件**（7 项 primary 全落地，零代码触碰）：
  - `skills/case-execution/SKILL.md`：Gotcha#8 `EXPECTED_OP_COUNT` 40->45（40 + CR-043 +2 + CR-044 +2 + CR-046 +1）；修订记录补 v1.7（对齐工作区 CR-046 sw3 族）+ v1.8（CR-039 追溯补记来源行）。
  - `docs/ptm-te/op-coverage-matrix.md`：新增 sw3 族 telnet 旁路章节（DQ-039-02 两套口径说明）+ 修订记录 v1.3；**total=124 / mapped=21 不变**。
  - `process/changes/CR-043.md` / `CR-044.md` / `CR-046.md`：追溯补记档案（DQ-039-03=B），含代码标注行号、Checkpoint Index 追溯补记标注、与 CR-039 关系。
  - `process/BACKLOG-CR-039.md`：CR-047 post_delay 独立回源候选（DQ-039-05=B，排除行 L114-133 `_parse_post_delay` + 睡眠改动实测 L2511）+ 真机回归 + DEF-039 候选。
  - `skills/case-execution/templates/env-file.pppoe-sw3.next_hop_a.example.yaml`：dut1.next_hop_a=80.0.0.2 + sw1 SW 节点契约（password_env=SW3_PASSWORD，无明文）。
- **验证**：静态核对脚本 5 项断言全 PASS（计数 45 / total-mapped 不变 / 档案齐备 / YAML 解析 / 无明文凭据）。
- **已知中间态**：canonical 代码 `EXPECTED_OP_COUNT=44`（G5 未回源，STORY-039-03/W3 事），文档=45 为 DQ-039-02 锁定先行基线；CP7 终验须在 W3 后。
- **证据**：`process/stories/STORY-039-04-IMPLEMENTATION.md`（含 G6 M9 归属 T-3 台账留痕）+ `process/returns/STORY-039-04.return.json`；Story status=dev-complete 待 CP6。

## STORY-039-02: G4 next_hop_a 第 10 类占位符（CR-039 / W2）

- 调度批次：CR-039 Wave 2（STORY-039-01 file-ownership 串行后同文件第二写入块）；design_evidence_confirmed=true（CP5 approve technical-note，随 STORY-039-01~04 全量批次）。
- 改动（canonical `skills/policy-route-execution/scripts/op_mapper.py`，G4 区 4 处，逐块合并非整文件覆盖）：
  - `resolve_env_refs` 新增 `dut.next_hop_a` 分支：`path=="dut.next_hop_a"` -> `dut_node.get("next_hop_a")`，空值抛 `_missing(val, "nodes.dut1.next_hop_a")`；与 `next_hop` 分支逐行同构（工作区 L1566-1569 回源）。
  - docstring 占位符表：表头 9 类->10 类，新增 `${ENV.dut.next_hop_a} -> nodes.dut1.next_hop_a（策略路由下一跳1 GW-A）`（工作区 L1488 逐字回源）。
  - 未知路径报错文案：「不在 9 类」->「不在 10 类」（工作区未同步处，按 DQ 口径对齐）。
  - 2 处计数注释对齐（模块注释 L1435 + 分支段注释补 dut.next_hop_a）：表注释/报错文案/实际分支三者计数一致。
- 新建 `tests/test_op_mapper_next_hop_a.py`：6 用例（成功解析 80.0.0.2 / 缺失字段报错含 nodes.dut1.next_hop_a / 空值报错 / next_hop 回归 110.0.0.2 / docstring 表第 10 类校验 / 未知路径报错含「不在 10 类」）。
- 验证：新增 6 passed；回归 rollback 32 + tg_op_mapping 37 + env_unsupported 8 + pppoe_client 18 全 PASS；py_compile 通过；EXPECTED_OP_COUNT=44 不变（44->45 归 STORY-039-03/W3）。
- 未触碰：G1 区块（W1 已落码）、G5/sw3 族、case_runner.py、SKILL.md、op-coverage-matrix.md、test_op_mapper_rollback.py。
- 有意偏离（注释级，已在 IMPLEMENTATION 留痕）：工作区报错文案与 2 处计数注释仍为 9 类，canonical 按 10 类口径对齐（方案评审规则第 1 条一致性要求）。
- 证据：`process/stories/STORY-039-02-IMPLEMENTATION.md` + `process/returns/STORY-039-02.return.json`；Story status=dev-complete 待 CP6。

## STORY-039-03: G5 sw3 族回源（CR-039 / W3 最终 Wave）

- 调度批次：CR-039 Wave 3（op_mapper.py 同文件第三个且最后写入块，merge_owner）；design_evidence_confirmed=true（CP5 approve technical-note，随 STORY-039-01~04 全量批次；DQ-039-02=A / DQ-039-04 / DQ-039-05=B 均 approve）。
- 改动（canonical，以含 W1 G1 + W2 G4 的工作区为基底逐块回源，工作区 `/home/hyde/projects/ptm-te-manaul/.claude/skills/` 为真相源）：
  - `skills/policy-route-execution/scripts/op_mapper.py`（G5 区 9 处）：六处映射表 sw3 条目（OP_ID_TO_SUBCOMMAND `("sw3","sync-pppoe-route")` / ARGS_TO_FLAGS `vpn_instance -> --vpn-instance` 形式 flag / REQUIRED_FLAGS `[]` / ROLLBACK_STRATEGY `irreversible` / OP_METADATA `state_mutation+irreversible+idempotent`）+ `EXPECTED_OP_COUNT` 44->45 + execute_op `family=="sw3"` 旁路 + sw3 执行段三函数（`_execute_sw3_op` / `_exec_sw3_telnet` telnetlib 惰性导入不打印密码 / `_parse_sw3_pppoe_route`，239 行与工作区 diff 逐行一致）+ validate `[5j]` sw3 子命令校验。
  - `skills/case-execution/scripts/case_runner.py`（sw3 dry-run envelope 3 处）：import `map_op_id_to_subcommand` + `_build_sw3_dry_run_envelope`（error 透传 / success -> `status=dry_run, data.command=计划 dict` 非 ptm-atomic 命令）+ dry-run `_dry_run_fn` sw3 分支走 `execute_op(dry_run=True)`。**CR-047 post_delay 精细排除（DQ-039-05=B）**：未引入 `_parse_post_delay` 与 post_delay 睡眠改动，grep 零命中。
  - `tests/test_tg_op_mapping.py`：44->45 断言 2 处 + 新增 11 用例（sw3 六处映射 / dry-run 计划 dict / 无 SW 节点 PARAM_INVALID / 未授权 EXEC_FAILED / SW3_PASSWORD 未设置 PARAM_INVALID / 路由表解析 / envelope 成功与错误透传 / post_delay 零残留源码断言）。
  - `tests/test_op_mapper_pppoe_client.py`（偏离记录，最小计数载体同步）：`test_expected_op_count` 44->45，否则该载体断言必然回归。
- 验证：指定三件套 85 passed（tg_op_mapping 47 含新增 11 + rollback/next_hop_a 38）；env_unsupported + pppoe_client 26 passed；全量 tests/ 244 passed + 2 存量失败（test_cr018_p2 GATE-4 脚本问题，git stash 对拍确认改动前即失败，与本 Story 无关）；`validate_mapping_consistency()` PASS（mismatches=[]）；py_compile 通过；post_delay grep 零命中。
- EXPECTED_OP_COUNT=45 四处最终一致性（SM-039-01）：代码常量 + validate 三表 + 单测断言（tg_op_mapping + pppoe_client）+ SKILL.md Gotcha#8（W1 已落 45）。
- 未触碰：G1/G4 区块（零回退）、SKILL.md、op-coverage-matrix.md、env-file 样例（W1 产物）、test_op_mapper_rollback.py / test_op_mapper_next_hop_a.py。
- 有意偏离（3 处，IMPLEMENTATION 偏离记录留痕）：pppoe_client 计数载体同步（SM-039-01 必然要求）；dry-run 成功 envelope error_type 按工作区实际行为断言空串（非 NONE，真相源优先）；工作区行号漂移（CR-047 改动致 case_runner 锚点偏移，按内容块定位）。
- 未覆盖（RA-039-02 / DQ-039-04）：真机 SW3 telnet 同步端到端不授权（static-only + dry-run 口径）；telnetlib py>=3.13 分支无 3.13 环境验证。
- 证据：`process/stories/STORY-039-03-IMPLEMENTATION.md`（含 G6 M9 归属 + sw3 用例样例来源 node3_dut1_tg1_sw1_pppoe_link3 + CR-047 排除 grep 零残留证据）+ `process/returns/STORY-039-03.return.json`；Story status=dev-complete 待 CP6。

## STORY-056-02: tde/te 路径约定固化（CR-056 / W1）dev-complete → CP6 PASS

- **调度批次**: CR-056 Wave 1（与 STORY-056-01 并行，文件所有权零交集）；meta-dev（ADE-CR056-META-DEV-001）；dispatch evidence 见 `process/state/AGENT-DISPATCH-LEDGER.ndjson`（platform agent_id/thread_id 待 host-orchestrator 回填）。
- **设计证据**: technical-note（design-already-in-hld）——`process/HLD-CR-056.md` v0.8 §4.1/§4.3/§4.4/§6/§8/§12 + `process/changes/CR-056.md` IMP-056-06/07/08；无 blocks_lld 灰区，未直接问用户。
- **改动文件**（ptm-team 仓库）：
  - `script/ptm_team/install.py`：新增 `RULE_BLOCK_VERSIONS`（ptm-te-workflow=1.1.0 / ptm-tde-workflow=1.0.0 / 未登记回退 MANAGED_VERSION）+ `managed_block_markers` 按块取版本；`render_ptm_te_rule_body` 规则 4 修订（执行入口 `cases/te/<协议族>/<功能域>/<子域>/<用例编号>.md`，`--cases-dir` 指向 `cases/te`，`cases/tde/` ptm-tde 设计源只读）+ 新增规则 14 冻结用例禁改 / 15 mark 唯一入口（--set 值域路由 + 台账人工区保留 + CICD⊆工厂不变式）/ 16 验收打标 exec 容器证据 + 同容器连续 3 轮 PASS / 17 作者元数据四字段 + archive_check 归档推送前检查 FAIL 阻断 push；renderer docstring 补 v1.1.0。
  - `agents/ptm-te.md`（canonical）：路径 9 处 `cases/<协议族>` → `cases/te/...`（角色表上游/执行入口、编排图、[1] 输入、[1] 异常路径、PC 消费契约输入路径、运行时目录树、目录表、`ppdcs/delivery/` 行、异常路径汇总）+ 目录树/目录表补 `cases/tde/` 只读 + 新增 `### 用例状态 mark 与运行准入（CR-056 简述）`（mark/cicd_mark 双字段 + 三角色场景圈池 5 行表 + 冻结禁改/mark 唯一入口/3 轮 PASS/作者元数据与归档检查）+ 修订记录 v2.6。
  - `skills/case-execution/SKILL.md`：仅 1 处路径文字级同步（「旧格式 -> 新格式转换规则」表头新格式列 → `cases/te/<协议族>/.../`，host-orchestrator 授权范围）；ST-EX-18/19/20、R-F-027/028 深度收口归 STORY-056-07。
- **安装副本刷新**（ptm-team 仓库外 2 项目，5 文件）：
  - 方式：installer 渲染函数直调（`RULE_BLOCK_RENDERERS` + `upsert_managed_rule_block` + `render_claude_agent` 布局）。受阻点：CLI `install claude --agent ptm-te` 会连带重拷 7 个 skill + 资源到外部项目（超 Story 所有权 + Wave-2 WIP 拷贝风险），无规则块-only 子命令；既有刷新痕迹（manaul manifest installed_at 2026-09-02T02:38:59Z ≠ 块 generated 2026-09-03T01:09:45Z）证明历史上即函数级刷新。幂等脚本 `/tmp/cr056-refresh.py`，刷新前备份 `/tmp/cr056-backup/`。
  - 结果：`ptm-te/CLAUDE.md`（claude）、`ptm-te-manaul/CLAUDE.md`（claude）、`ptm-te-manaul/AGENTS.md`（codex）ptm-te-workflow 块 → `v=1.1.0 commit=ad89670 generated=2026-09-03T11:24:08Z`；`ptm-te/.claude/agents/ptm-te.md`、`ptm-te-manaul/.claude/agents/ptm-te.md` 保留 frontmatter（diff 0 行）+ 正文整体替换 + managed 行刷新。
- **一致性校验**：三份规则文件 ptm-te-workflow 块正文剔除 header 后逐字节一致（te/CLAUDE ≡ manaul/CLAUDE；manaul/AGENTS 仅平台标签句「Codex / Qoder」差异）；三份 agent md 正文逐字节一致；托管块外内容不变；ptm-tde-workflow 块 v=1.0.0 不变；外部项目 19:20 后改动文件 == 5 个目标文件（find -newermt 实测）。
- **验证结果**：渲染器断言（规则 1..17、v= 值、规则 4/14/15/16/17 全文与 HLD 逐字比对）PASS；`uv run --python 3.11 pytest tests/test_install_mapping.py -q` 10 passed。
- **已知限制 / 未覆盖**：① `ptm-te-manaul/.codex/agents/ptm-te.toml`（codex 平台 agent 副本，正文 v2.5）不在本 Story「三副本」声明面，未刷新——已作设计缺口反馈上报，建议 STORY-056-07 收口或 waived（OQ-056-02-01）；② 规则 14~17 机器行为由 STORY-056-03~06/08 实现、07 回归；③ `dev_gate=cp5-pending` 字段未动，由门禁账务回填；④ GATE-LEDGER 无 CR-056 gate 事件（host-orchestrator 补记）；⑤ 仓库 guardrail 脚本 `scripts/check_delivery_guardrails.py` 不存在，N/A。
- **给 meta-qa 的验证入口**：`uv run --python 3.11 pytest tests/test_install_mapping.py -q`；三副本 diff（规则块正文 / agent 正文）；幂等重跑 `uv run --python 3.11 python /tmp/cr056-refresh.py` 应零 diff（generated 时间戳除外）。风险提示：R-056-02-A（存量 --cases-dir 旧路径，运行时零代码改动兜底）、R-056-02-B（codex toml 滞后）。
- **证据**: `process/stories/STORY-056-02-te-path-rule-block-sync-IMPLEMENTATION.md` + `process/checks/CP6-STORY-056-02-te-path-rule-block-sync-CODING-DONE.md` + `process/checks/CP6-STORY-056-02-te-path-rule-block-sync.result.json`（meta-flow cp result-check OK）+ `process/returns/STORY-056-02-te-path-rule-block-sync-CP6.return.json` + `process/evidence/STORY-056-02-te-path-rule-block-sync-CP6.index.json`（validator NONE/NONE）；Story status=ready-for-verification。

## STORY-056-04: case_runner mark/cicd_mark schema + run 准入 + 冻结校验（CR-056 / W2）dev-complete → CP6 PASS

- **调度批次**: CR-056 Wave 2（depends_on=[STORY-056-03] interface 依赖已满足；same-file-serial，056-05 待接力）；meta-dev（ADE-CR056-META-DEV-STORY-056-04，agent_id/thread_id 待 host-orchestrator 回填）。
- **设计证据**: technical-note（design-already-in-hld）——`process/HLD-CR-056.md` v0.9 §4.3/§4.4/§7.1/§7.2 C-1~C-8 锚点；无 blocks_lld 灰区，未直接问用户；锚点按符号定位。
- **改动文件**（工程资产仅 1 个，+678/-7 行）：
  - `skills/case-execution/scripts/case_runner.py`：
    - **C-1**：`_FM_MARK_KEY`/`_FM_CICD_MARK_KEY`（`_FM_TAGS_KEYS` 后）+ case_ledger 值域/池常量导入（try/except 降级门控，同 exec_task 委托先例，模块缺失时既有功能零影响）。
    - **C-2**：parse_frontmatter 新增 [3c] mark 枚举校验（缺省 debug）、[3d] cicd_mark 枚举校验（缺省 None）、[3e] 不变式读点（cicd_mark≠无 ⇒ mark=accept，G-13 双点之一）；返回 dict 与 4 个早退分支统一补 mark/cicd_mark 键；16 列+tags/legacy 标签解析零破坏。
    - **C-3**：`filter_by_mark`（in 并集、空透传）/ `filter_by_cicd_mark`（none→cicd_mark is None、多值并集、空透传）。
    - **C-4**：run parser 增 `--mark/--cicd-mark/--role/--scene(默认 daily)/--run-scope`（run-scope 缺省 None，显式传参覆盖角色推导，G-11 正交）。
    - **C-5**：main() run 段最前 `[0-pre]` 参数级准入（`_validate_role_pool_args`：值域/feature·acceptance 非法组合/显式圈选 ⊆ ROLE_SCENE_POOLS 池域/跨域圈选拒绝，非法 exit 2 快速失败不触设备）；ST-EX-16 过滤段扩展为 `--tag→--keyword→--mark→--cicd-mark→--role 池` 链（`apply_role_pool` 默认池圈选 + cmo mark≠accept 自动排除子集语义；`describe_role_pool` 观测输出）；无 --role 行为零变化；--case-file 过滤参数 WARNING 忽略（同 --tag 惯例）。
    - **C-6**：`[0-pre2]` 台账定位（`_resolve_ledger_root` 自 --cases-dir/--case-file 向上 ≤2 级精确匹配 case_ledger.yaml，symlink 场景 cases/te→cases/ 命中）+ 加载（异常显式收敛）+ LEDGER_MISSING fail-closed（factory/cicd exit 2，local 降级）；执行循环 parse 后、execute 判定前逐用例 `_case_state_snapshot`→`verify_case_state`（**dry-run 同样生效**，真实构造冒烟未 mock）：失配记 ERROR `CASE_MARK_STALE`（固定话术"状态已失效（用例被修改），需重新走流程"，细节落 details.verify_detail，G-6 可区分）不执行，local 仅 WARNING 放行。
    - **C-7**：case 记录（4 个 append 点）统一附 `case_state: {mark, cicd_mark, fingerprint_ok(True/False/None)}` 审计快照 + result_json 顶层 `mark_admission` 语境（增量，见缺口反馈）；既有字段零改动（generate_report/exec_task 按键消费不受影响）。
    - **C-8**：mark/ledger/archive-check 子命令 parser（参数面 = HLD §7.3/§7.4/§5.4 全量）+ 委托 `case_ledger.cmd_mark/cmd_ledger/cmd_archive_check`（validate/exec-report 同层，无启动校验/设备配置，模块缺失 exit 5）。
- **锚点外增量（已留痕待认可，均可一行回退）**：① deprecated 吸收态被显式圈选仍不执行（HLD G-8/G-06，任务锚点未列且无其他 Story 承接）→ 记 ERROR `CASE_MARK_DEPRECATED`；② mark_admission run 级审计语境；③ 跨域圈选拒绝对齐 §4.4 规则 2（G-8 旧措辞为 v0.7 前残留，仅无 --role 模式成立）。
- **验证结果**：py_compile 3.11/3.12 PASS；`pytest skills/case-execution/tests/ -q` **88 passed 零回归**；交互冒烟 `/tmp/smoke_cr056_04.py`（不落仓库）**69 pass / 0 fail**（dry-run 端到端：篡改正文+--role factory → CASE_MARK_STALE 阻塞 rc=1 真实构造，无 mock）；真实 te 库 312 md 只读解析 311 用例 0 error、mark 全缺省 debug（G-01 口径）。
- **已知限制 / 未覆盖**：① cmd_mark/cmd_ledger/cmd_archive_check 为骨架（路由校验通过返回 5），实体归 056-05/06/08；② 正式 pytest 后置 056-07（冒烟即蓝本）；③ 4 副本脚本树同步、SKILL.md ST-EX-18/19/20 深度收口归 056-07；④ --execute 真实路径冻结校验与 dry-run 共用前置代码，未真机验证（冒烟全 dry-run，无 --execute）。
- **给 STORY-056-05 的交接**：main() 接线点 = `if args_ns.command in ("mark","ledger","archive-check")` 分支（exec-report 委托后、run 段前），mark 已委托 cmd_mark，056-05 只需实体化 case_ledger.cmd_mark，case_runner 侧无需再改；cmd_mark 签名 `cmd_mark(args) -> int`（args.set/case_file/cases_dir/mark/cicd_mark/note/evidence/rounds/operator/role/scene/no_commit/cases_root）；可复用件 `_split_csv/_resolve_ledger_root(max_up=2)/filter_by_mark/filter_by_cicd_mark/_case_state_snapshot`；现场无 PyYAML 需 catch 收敛（056-03 风险提示）。
- **给 meta-qa 的验证入口**：`uv run --python 3.11 --with pytest --with pyyaml pytest skills/case-execution/tests/ -q`（88 基线）+ `uv run --python 3.11 --with pyyaml python /tmp/smoke_cr056_04.py`（69 断言重放）。风险提示：run-scope 生效值看 result.json.mark_admission.run_scope（argparse 原始值为 None）；GAP-STORY-056-04-01 三项增量待 host-orchestrator 认可。
- **证据**: `process/stories/STORY-056-04-case-runner-mark-admission-IMPLEMENTATION.md` + `process/checks/CP6-STORY-056-04-case-runner-mark-admission-CODING-DONE.md` + `.result.json` + `process/returns/STORY-056-04-case-runner-mark-admission-CP6.return.json` + `process/evidence/STORY-056-04-case-runner-mark-admission-CP6.index.json`；Story status=ready-for-verification。

## STORY-056-03（补记）: case_ledger.py 新模块（CR-056 / W2）dev-complete → CP6 PASS

- **补记说明**: 本条目为 STORY-056-05 实施时按 host-orchestrator 指示补记（056-03 段此前漏记）；完整证据见 `process/stories/STORY-056-03-case-ledger-module-IMPLEMENTATION.md` 与 `process/checks/CP6-STORY-056-03-case-ledger-module-CODING-DONE.md`（PASS 10/10）。
- **摘要**: case_ledger.py 新模块（常量 11 组 + 函数 10 个：双域迁移矩阵 / 指纹 G-4 / 台账读写 G-5·G-7 / verify_case_state / append_mark_history / resolve_evidence_rounds / 三个子命令入口骨架）；冒烟 103 断言全过、py_compile 3.11/3.12、回归 88 passed；灰区 LCQ-STORY-056-03-01（dry-run 轮计入验收轮数）已于 STORY-056-05 定案落地（仅 mode=execute 轮计入）。

## STORY-056-05: case_runner mark 子命令实体化（CR-056 / W2）dev-complete → CP6 PASS

- **调度批次**: CR-056 Wave 2（depends_on=[STORY-056-04] same-file-serial，056-04 已验收后接力）；meta-dev（ADE-CR056-META-DEV-STORY-056-05，agent_id/thread_id 待 host-orchestrator 回填）。
- **设计证据**: technical-note（design-already-in-hld）——`process/HLD-CR-056.md` v0.9 §7.3 六步内部动作 + §5.3 git 自动 commit + §4.1 迁移矩阵与连带清除 + §10 失败路径；无 blocks_lld 灰区，未直接问用户。
- **改动文件**（工程资产仅 1 个；case_runner.py 零改动，git diff --stat 维持 056-04 的 +678/-7 实测）：
  - `skills/case-execution/scripts/case_ledger.py`：
    - **cmd_mark 骨架 → 实体**（[P0] 参数校验 → [P1] cases_root 定位（`_resolve_ledger_root` 向上≤2 级精确匹配，与 case_runner C-6 同口径）+ 台账加载（无 PyYAML/损坏收敛 exit 5）→ [P2] 圈选（--case-file 直取 / --cases-dir 递归 + --mark/--cicd-mark 圈选 + 值域校验 + 圈选空 exit 2）→ [P3] 逐用例六步判定与写面 → [P4] stdout 摘要表 + rc 0/1/2 分级）。
    - **六步落点**：① 台账为准 + 漂移 WARNING + validate_transition + 同值自环预判跳过（056-03 缺口 #5 定案）+ 台账无登记 WARNING 建档；② 回退/废弃缺 --note 拒绝该项，debug→verify 缺 note 仅 WARNING；③ EVIDENCE_REQUIRED 三边（verify→accept 与 cicd_verify→cicd_accept 需 evidence+3 轮、None→cicd_verify 需容器 execute 追溯）；④ 指纹（写入面与指纹共用 `_FRONTMATTER_RE`，G-4）→ `_frontmatter_set_line`（原位替换/块尾插入/连带清除删行）→ `append_mark_history`（先）→ entry 更新（后）→ `save_ledger`（全程 `ledger_lock`，新建条目 path 派生 cases/te 前缀）；⑤ `_git_commit_mark`（rev-parse 向上发现仓库根、add md+台账、`mark(case): <id> <旧>→<new>` 双域串、--no-commit、非 git/失败降级 WARNING、**永不 push/force**、锁外执行）；⑥ 降级收敛。
    - **resolve_evidence_rounds 落地 LCQ-STORY-056-03-01 定案**：仅 mode=execute 轮参与连续 PASS 计数，dry-run 不计入且打断连续（核实发现 056-03 实际未过滤 mode，本 Story 在自身写入范围落地，见设计缺口反馈 #2）。
    - 新增私有辅助 10 个；`--rounds` 为台账审计声明值（不降低 DQ-056-02 硬门槛 N=3）。
- **锚点外决策（已留痕）**：① 圈选纯函数 case_ledger 本地对齐实现而非 import case_runner（循环 import + op_mapper 耦合，缺口反馈 #1）；② 摘要表失败行 stdout 单一真相源（stderr 只承载 WARNING，失败感知依赖 rc，缺口反馈 #3）；③ 台账 path 派生以 cases_root 目录名为前缀（symlink 方案天然 cases/te 口径，缺口反馈 #4）。
- **验证结果**：py_compile 3.11/3.12 PASS；pytest **88 passed 零回归**；冒烟 `/tmp/smoke_cr056_05.py`（不落仓库）**55 pass / 0 fail**（> 任务要求 30），经 `case_runner.main(["mark", ...])` 端到端；056-04 冒烟回归 70/0（F2 过渡断言随实体化更新）。git 产物在 /tmp 临时 git init 仓库实测：逐用例独立 commit、恰含 md+台账两文件、--no-commit 零 commit、非 git 降级 WARNING、无 push；**真实 ptm-cases 与 312 条用例零触碰**。
- **已知限制 / 未覆盖**：① cmd_ledger/cmd_archive_check 仍骨架（056-06/08）；② 正式 pytest 后置 056-07（冒烟即蓝本）+ 4 副本脚本同步；③ `--evidence` 相对路径按 CWD 解析（建议操作卡用绝对路径）；④ git 身份缺失时 commit 降级 WARNING（打标仍成功，审计靠台账 by 字段）。
- **给 STORY-056-06 的交接**：接线点 = main() mark/ledger/archive-check 委托分支已就绪，06 只实体化 `case_ledger.cmd_ledger`（case_runner 零改动）；签名 `cmd_ledger(args) -> int`（args.cases_root required / args.mark 为 **list**（parser 已拆分，骨架已校验值域）/ args.out）；可复用件 `load_ledger`/`verify_case_state`（漂移判定语义源）/`_fm_mark_fields`/`_iter_case_files`/`_extract_case_id`/`compute_case_fingerprint`；条目结构 `{path, mark, cicd_mark, content_sha256, mark_history[8 字段], first_submitted_at?}`；`--out` 沿用 exec-report splice 惯例（机器区 marker 重写 + 人工区原样保留，人工区键 submitter/notes/open_issues 不得触碰）；无 PyYAML 收敛 exit 5（catch 模板复用 cmd_mark）。
- **给 meta-qa 的验证入口**：`uv run --python 3.11 --with pytest --with pyyaml pytest skills/case-execution/tests/ -q`（88 基线）+ `/tmp/smoke_cr056_05.py`（55 断言重放）+ `/tmp/smoke_cr056_04.py`（70 断言）。风险提示：摘要表在 stdout（stderr 仅 WARNING）；GAP-STORY-056-05-01 三项决策待认可（均不阻塞）。
- **证据**: `process/stories/STORY-056-05-mark-subcommand-IMPLEMENTATION.md` + `process/checks/CP6-STORY-056-05-mark-subcommand-CODING-DONE.md` + `.result.json` + `process/returns/STORY-056-05-mark-subcommand-CP6.return.json` + `process/evidence/STORY-056-05-mark-subcommand-CP6.index.json`；Story status=ready-for-verification。

## STORY-056-06: case_runner ledger 子命令实体化（CR-056 / W2）dev-complete → CP6 PASS

- **调度批次**: CR-056 Wave 2（depends_on=[STORY-056-05] same-file-serial，056-05 已验收后接力；Wave 2 至此全部完成）；meta-dev（ADE-CR056-META-DEV-STORY-056-06，agent_id/thread_id 待 host-orchestrator 回填）。
- **设计证据**: technical-note（design-already-in-hld）——`process/HLD-CR-056.md` v0.9 §7.4（计数表/待办清单/漂移清单/submit_time 摘要）；无 blocks_lld 灰区，未直接问用户。
- **改动文件**（工程资产仅 1 个；case_runner.py 零改动，git diff --stat 维持 056-04 的 +678/-7 实测；resolve_evidence_rounds/cmd_mark 等 056-03/05 既有函数零改动）：
  - `skills/case-execution/scripts/case_ledger.py`：
    - **cmd_ledger 骨架 → 实体**（[P0] 参数校验（骨架 cases_root/--mark 值域保留）+ exec_task 门控 → [P1] load_ledger 收敛 exit 5 → [P2] 只读聚合视图 stdout → [P3] --out splice）。
    - **§7.4 四段视图**（stdout 与 --out 同一渲染源 `_ledger_view_markdown`）：① mark 5 态 × cicd_mark 4 档交叉矩阵 + 行/列/总计 + deprecated 计数（G-10 语义标注）；② 待办三类——accept 未提交 CICD（"验收证据"列自 mark_history 派生三态：达标/轮数未记录/未达标，覆盖 HLD"稳定轮达标待提交"，不扫 runs 保持纯台账只读）、verify_fail+cicd_verify_fail 待整改、漂移/异常清单（`verify_case_state` 单一判定语义 + 用例文件缺失 + "md 存在但台账无登记"磁盘扫描——仅认 frontmatter 用例编号，tde/README 天然忽略）；③ first_submitted_at 提交记录摘要（N 例 + 最早/最近 + 全列表，DQ-056-09）。
    - **--out 复用 exec_task.splice_report**（先读 parts={frontmatter,machine,manual_template} 与 GEN marker 后直接调用，零重复实现）：首生成含人工区占位模板；二次运行机器区 marker 重写而人工区原样保留；无 marker/错序拒绝覆盖 exit 3（fail-closed 防吞人工区）；tmp+os.replace 原子写 + 父目录 makedirs（generate_report 同惯例）。
    - **降级**：exec_task 缺失 → --out exit 5（stdout 视图可用）；无 PyYAML/台账损坏 → exit 5 无 traceback（cmd_mark 同款模板）；退出码 3 档已补进模块头。
    - 新增视图渲染辅助 7 个（_scoped_entries/_accept_evidence_of/_resolve_case_file/_drift_rows/_ledger_view_markdown/_ledger_frontmatter/_LEDGER_MANUAL_TEMPLATE）。
- **顺手项销项**: 协调者描述"resolve_evidence_rounds 的 execute 过滤只在 cmd_mark 调用处实现（函数本身未过滤）"与现状不符——056-05 已实现于**函数体内**（直接证据：056-05 冒烟 L01~L05 直接调用该函数断言 mode 过滤）；本 Story 零改动该函数，无双口径需统一（缺口反馈 #1 留痕供销项）。
- **验证结果**：py_compile 3.11/3.12 PASS；pytest **88 passed 零回归**；冒烟 `/tmp/smoke_cr056_06.py`（不落仓库）**29 pass / 0 fail**（> 任务要求 20），经 `case_runner.main(["ledger", ...])` 端到端；前序冒烟回归 056-05 55/0、056-04 70/0（F4 过渡断言随 ledger 实体化更新为 rc 0，预期演进留痕）；真实环境只读探针（ptm-te-manaul/cases）rc=0 零写入。
- **已知限制 / 未覆盖**：① cmd_archive_check 仍骨架（056-08）；② 正式 pytest 后置 056-07 + 4 副本脚本同步 + SKILL.md ST-EX-18~20 深度收口；③ `--json` 输出未排期（HLD §9.3"后续可加"）；④ 漂移清单对同号多文件只记首见路径（编号唯一性归 CR-053 GATE-5 域）。
- **给 STORY-056-07 的收口清单**：pytest 蓝本 = 03~06 冒烟（指纹 8/迁移 33/verify_case_state/resolve_evidence_rounds 仅 mode=execute/cmd_mark 六步与降级/cmd_ledger 四段与 splice 三态/run 准入与冻结校验 70 断言）；4 副本脚本同步（ptm-te/.claude、ptm-te-manaul/.claude、ptm-te-manaul/.agents）；SKILL.md ST-EX-18/19/20 + R-F-027/028 + Gotchas；文档提示项（--cases-root 指向 cases 目录、--evidence 建议绝对路径、--run-scope 与 --role 正交 G-11、跨域圈选拒绝口径、人工区勿贴生成区 marker）。
- **给 STORY-056-08 的交接**：接线点 = main() archive-check 委托已就绪（占位 exit 5 防门禁 fail-open）；parser 参数面 --cases-root/--author/--date/--fix/--json 已按 HLD §5.4 备齐；可复用件 `_extract_case_id`/`_FRONTMATTER_RE`（A/M 类 frontmatter 读写）、`compute_case_fingerprint`（M 类内容实质差异 = 指纹变化，G-15 打标提交不触发）、git 根发现语义（056-05 `_git_commit_mark` rev-parse 模式）；规则 17 门禁语义 FAIL → exit 1 阻断 push、非 git → exit 2。
- **给 meta-qa 的验证入口**：`uv run --python 3.11 --with pytest --with pyyaml pytest skills/case-execution/tests/ -q`（88 基线）+ `/tmp/smoke_cr056_06.py`（29）+ `/tmp/smoke_cr056_05.py`（55）+ `/tmp/smoke_cr056_04.py`（70）。风险提示：stdout 与 --out 同源渲染（frontmatter 仅在文件）；无 frontmatter 文件被视图静默跳过；GAP-STORY-056-06-01 三项待认可（均不阻塞）。
- **证据**: `process/stories/STORY-056-06-ledger-subcommand-IMPLEMENTATION.md` + `process/checks/CP6-STORY-056-06-ledger-subcommand-CODING-DONE.md` + `.result.json` + `process/returns/STORY-056-06-ledger-subcommand-CP6.return.json` + `process/evidence/STORY-056-06-ledger-subcommand-CP6.index.json`；Story status=ready-for-verification。

## STORY-056-07: 测试与同步收口（CR-056 / W3）dev-complete → CP6 PASS

- **调度批次**: CR-056 Wave 3（depends_on=01~06 all-waves 全部就绪后接力）；meta-dev（ADE-CR056-META-DEV-STORY-056-07，agent_id/thread_id 待 host-orchestrator 回填）。
- **设计证据**: technical-note（design-already-in-hld）——`process/HLD-CR-056.md` v0.9 §13 验收矩阵 + §8 文档同步面 + §2 F-01/F-06 副本清单；无 blocks_lld 灰区。
- **改动/同步文件**：
  - `skills/case-execution/tests/test_cr056_mark_lifecycle.py`（新建）：**118 用例**（Fingerprint 8 / ValidateTransition 27 / VerifyCaseState 6 / ResolveEvidenceRounds 6 / CmdMarkGuards 7 / CmdMarkHappyPath 16 / CmdLedger 12 / ParseFrontmatterMark 7 / FilterByMark 4 / RolePoolGuards 17 / FreezeCheck 6 / SubcommandDelegation 2），蓝本 = 056-03~06 冒烟；全部 tmp_path/mock exec 容器/临时 git 仓库，零真实设备与真实 ptm-cases。
  - 4 副本脚本同步：case_runner/case_ledger/init_cases_repo × ptm-te/.claude、ptm-te-manaul/.claude、ptm-te-manaul/.agents（case_ledger/init_cases_repo 为新增分发），同步后 **12/12 diff 一致**；exec_task 三处本就一致零改动跳过；同步前备份 /tmp/cr056-07-backup。
  - `ptm-te-manaul/.codex/agents/ptm-te.toml`：**OQ-056-02-01 收口**——直调 install.py `render_codex_agent`（header 与 claude 副本同口径 version=1.0.0 commit=ad89670），正文 v2.5 → v2.6（canonical 剥 frontmatter），同口径转义后与 canonical 逐字节一致；规则块不落 TOML（核实既有机制：规则走 CLAUDE.md/AGENTS.md 托管块）；tools 数组无引号为安装器既有口径保持 parity（缺口反馈 #2 转候选台账）。
  - `skills/case-execution/SKILL.md`（v2.2）：ST-EX-15 区域补 R-F-027/028 + 两个新小节；新增 ST-EX-18（圈选与运行准入）/ ST-EX-19（mark/ledger 子命令与台账）/ ST-EX-20（归档检查，056-08 占位）；Gotchas #12~#15；命令示例与目录树路径 cases/te（旧路径残留 grep=0，cases/upload 为转换规则表历史语义保留）。
  - `docs/ptm-te/执行指导.md`：新增 §9「mark 状态流转与三场景运行」（状态字段表 / feature·factory·cmo 三场景命令序列 / 失败打回全链 / ledger 台账查看 / archive-check 引用 056-08）；§8 历史实战记录保留并加注现入口 cases/te。
- **验证结果**：全量 `pytest skills/case-execution/tests/ -q` **206 passed**（88 既有 + 118 新增零回归）——完成门槛达标；py_compile 3.11/3.12 PASS；agent 三方正文一致（canonical/2 claude 规范化一致 + TOML 同口径转义一致）；SKILL.md 结构校验 PASS。
- **已知限制 / 未覆盖**：① 真机三场景端到端属运行授权域（单测 dry-run/mock）；② 存量元数据回填 O-056-06、`ledger --json` 未排期；③ TOML tools 数组严格解析不适用为既有安装器口径（GAP-07-01 候选）。
- **给 STORY-056-08 的交接**：接线点 = main() archive-check 委托已就绪（占位 exit 5 防门禁 fail-open）；parser 参数面 --cases-root/--author/--date/--fix/--json 已按 HLD §5.4 备齐；可复用件 = `_extract_case_id`/`_FRONTMATTER_RE`/`compute_case_fingerprint`（M 类实质差异 = 指纹变化，G-15）/`_git_commit_mark` rev-parse 模式；SKILL.md ST-EX-20 占位文案已就位（实体交付无需回改）；规则 17 门禁语义 FAIL → exit 1、非 git → exit 2；交付后按 07 同清单分发 3 处 + canonical。
- **给 meta-qa 的验证入口**：`uv run --python 3.11 --with pytest --with pyyaml pytest skills/case-execution/tests/ -q`（206 基线）+ 4 副本 diff + TOML 三方一致性（命令见 IMPLEMENTATION §验证结果，可重放）。风险提示：分发覆盖式同步（备份仅本会话有效）；TestFreezeCheck/RolePoolGuards 依赖 op_mapper 可导入（与既有 88 例同前提）。
- **证据**: `process/stories/STORY-056-07-tests-and-sync-closure-IMPLEMENTATION.md` + `process/checks/CP6-STORY-056-07-tests-and-sync-closure-CODING-DONE.md` + `.result.json` + `process/returns/STORY-056-07-tests-and-sync-closure-CP6.return.json` + `process/evidence/STORY-056-07-tests-and-sync-closure-CP6.index.json`；Story status=ready-for-verification。

## STORY-056-08: archive_check 归档推送前检查实体化（CR-056 / W3 收官）dev-complete → CP6 PASS

- **调度批次**: CR-056 Wave 3（depends_on=[STORY-056-07]）；meta-dev（ADE-CR056-META-DEV-STORY-056-08，agent_id/thread_id 待 host-orchestrator 回填）。**本 Story 完成后 CR-056 八 Story 全部 ready-for-verification**。
- **设计证据**: technical-note（design-already-in-hld）——`process/HLD-CR-056.md` v0.9 §5.4 六步语义 + §4.1a 作者元数据 + §12 G-15 + 规则 17；无 blocks_lld 灰区。
- **改动/同步文件**：
  - `skills/case-execution/scripts/case_ledger.py`：cmd_archive_check 占位 → **实体**（① git 根向上发现 NOT_GIT_REPO exit 2 → ② A/M/D 变更集（porcelain + diff HEAD，过滤 cases/ 前缀，`-c core.quotepath=false` 中文路径）→ ③④ A 类四字段 + 日期口径 / M 类指纹差异判定（G-15 打标提交跳过）→ ⑤ --fix 补齐/刷新 + 写后重校验 → ⑥ 逐文件清单 + PASS/FAIL 汇总 + --json）+ 辅助 6 个 + json 导入。
  - `skills/case-execution/scripts/archive_check.py`（新建薄壳）：独立 CLI（HLD §5.4 契约 `python archive_check.py --cases-root ...`），import case_ledger 委托零重复实现。
  - 分发同步：archive_check.py（新增）+ case_ledger.py（更新）× 3 分发目录，**15/15 diff 一致**（5 脚本 × 3 分发全量校验）。
  - `skills/case-execution/SKILL.md`：ST-EX-20 占位文案转正式（双入口/检查语义/退出码/push 永不自动执行 + 冻结用例 --fix 操作顺序提示）。
- **实现位置决策（派发授权自行判断）**：case_ledger 内实体（承接 case_runner 委托）+ 薄壳 CLI（规则 17 独立调用习惯）双入口，检查逻辑单一来源。
- **验证结果**：py_compile 3.11/3.12 PASS；全量回归 **206 passed 零回归**；冒烟 `/tmp/smoke_cr056_08.py`（不落仓库）**30 pass / 0 fail**（临时 git init 仓库 + 真 git add/commit 构造 HEAD 基线，A/M/D 变更集 fixture）；分发 15/15 一致；**真实 cases 只读探针 rc=0 零写入零 --fix**（git_root 正确解析 /home/hyde/projects/ptm-cases，symlink 场景向上发现成立）。
- **产品缺陷自纠（已修复并断言锁定）**：初版 `git diff HEAD --name-only` 把已删除文件泄入 modified（工作区无文件 → 误报 READ_ERROR FAIL）——修复为 diff 循环排除 deleted 集合，冒烟 A10/A12/B01 锁定（缺口反馈 #1）。
- **已知限制 / 未覆盖**：① rename（R 类）按新路径计入 M 类；② archive_check 正式 pytest 合并随后续收口 CR（本 Story 冒烟 30 断言为蓝本）；③ A 类创建时间过期记 STALE_MODIFIED（7 类错误码约束，可一行改独立码）。
- **给 meta-qa 的验证入口（CR-056 全量 CP7）**：`uv run --python 3.11 --with pytest --with pyyaml pytest skills/case-execution/tests/ -q`（206 基线）+ 冒烟重放 03~08（103/69/55/29/70/30）+ 4 副本 diff（5 脚本 × 3 分发）+ agent 三方正文 + TOML 一致性 + SKILL.md/执行指导结构；validation_mode 建议 static-only + 冒烟重放（真机三场景端到端属 runtime_authorization 独立域）。
- **证据**: `process/stories/STORY-056-08-archive-check-IMPLEMENTATION.md`（含 CR-056 八 Story 最终交接摘要附录）+ `process/checks/CP6-STORY-056-08-archive-check-CODING-DONE.md` + `.result.json` + `process/returns/STORY-056-08-archive-check-CP6.return.json` + `process/evidence/STORY-056-08-archive-check-CP6.index.json`；Story status=ready-for-verification。
