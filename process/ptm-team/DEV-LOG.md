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
