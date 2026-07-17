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
