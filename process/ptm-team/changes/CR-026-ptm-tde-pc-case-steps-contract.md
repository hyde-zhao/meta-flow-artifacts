---
change_id: CR-026-ptm-tde-pc-case-steps-contract
workflow_id: WF-PTM-TEAM-20260520-001
created_at: "2026-07-13T17:30:00+08:00"
created_by: host-orchestrator（主进程）
status: closed
implementation_status: delivered
workflow_mode: standard
engagement_mode: production
impact_level: medium
complexity: moderate
rollback_to: requirement-clarification
approval_source: CR-025 follow-up（args 命名对齐后的 op_id / preconditions / 可执行参数契约强化）
depends_on:
  - CR-025（closed，args 命名对齐已完成，本 CR 在其上强化 op_id 与 preconditions 契约）
  - CR-019（PC case_steps atomic_op 契约基线，本 CR 扩展 preconditions 与 args 四约束）
  - CR-018（GATE-4 字段级结构检查，本 CR 在其上新增 op_id 命中校验）
  - ptm-atomic（外部真相源，atoms/fw/<op_id>.yaml 的 op_id 清单与 inputs.params/preconditions）
plan_ref: 无（改动集中在 PC case_steps 契约层 + GATE-4 自检扩展，本 CR 即计划）
cross_references:
  - CR-025（args 对齐，本 CR 补齐 op_id 对齐与 preconditions 透传）
  - CR-019（atomic_op.op_id 契约，本 CR 扩展 preconditions/args 约束）
  - CR-024（ptm-te op_mapper，本 CR 的 op_id 命中校验为 op_mapper 简化提供前置）
  - CR-018（GATE-4 字段级检查基线，本 CR 新增 op_id 命中项）
---

# CR-026 - ptm-tde PC case_steps 契约强化（op_id 真相源 + preconditions + 可执行参数）

## 变更请求摘要

CR-025 完成了 `args` 字段名对齐 op yaml `inputs.params`，但暴露出 PC `case_steps` 契约层仍有三项根因性缺口：

1. **op_id 选择无真相源约束**：ptm-tde.md:596 只规定 args 必须查 op yaml，却未规定 `atomic_op.op_id` 本身必须从 ptm-atomic 真实清单选取，导致可自创 op_id（如把抓取日志类误写为 `fw_verify_operation_log`，真相源实为 `fw_capture_operation_log`）。
2. **preconditions 字段缺失**：case_steps 契约无 preconditions 字段，16 列表"预置条件"列无结构化源；而 op yaml 真相源已含 `preconditions`/`postconditions`，可直接透传。
3. **args 可执行性无硬约束**：PC 的 args 允许占位、缺 required 参数、值类型不对齐（对象名 vs CIDR），且 step 间依赖无显式声明。

本 CR 在 CR-025 基础上，**强化 PC `case_steps` 契约层**：op_id 真相源规则 + 前缀语义、preconditions 双层透传（op 级 + step 级）、args 可执行四约束，并扩展 GATE-4 自检做 op_id 命中机器校验。

## 背景

### 阶段 0 收尾（已完成）

CR-025 同批示例存在 op_id 命名漏网：4 个 skill 的 `op_id` 示例用 kebab-case `config-policy-route`，与真相源 `fw_config_policy_route`（snake_case）不一致。已在本 CR 启动前作为 CR-025 收尾修正（8 处，已验证零残留），不纳入本 CR 改动清单。

### 四项整改诊断（审核结论）

| 项 | 现状 | 缺口 |
|---|---|---|
| P0-2 op_id 对齐真相源 | args 对齐已做（CR-025）；op_id 选择规则缺失；GATE-4 有 op_id 回链校验但无命中校验 | op_id 必须从 `ptm-atomic list` 真实清单选取的硬规则；capture/verify 前缀语义；GATE-4 命中机器校验；CAP-003 gap 登记 |
| P1-3 PC 可执行参数 | case_steps.args 有结构骨架 | 四约束：具体值（非占位）/ 完整性（覆盖 required）/ 语义对齐（对象名 vs CIDR）/ depends_on（step 间依赖） |
| P1-4 preconditions 字段 | case_steps 契约无 preconditions；op yaml 真相源已含 | case_steps 双层透传：`atomic_op.preconditions`（op 级，透传 op yaml）+ `step_preconditions`（step 级，用例自填） |
| P1-5 直接产出 case_steps | ✅ 已基本实现（deliverable-renderer 从结构化字段渲染 16 列） | 仅需 ptm-tde.md:626 追踪链表补注状态 |

### 关键洞察

**P0-2 是 P1-3/P1-4 的根**：op_id 真相源规则不立，P1-3 的"语义对齐"和 P1-4 的"preconditions 透传"都失去校验基准。整改从 P0-2 切入，合并单 CR 避免 case_steps 契约层跨文件一致性裂缝。

## 调研结论

### op_id 真相源

- **真实清单**：`/home/hyde/projects/ptm-atomic/atoms/fw/` 含 39 个 verify op + 4 个 capture op + config/update/delete/login/check/reset 等前缀。命名前缀是真相源约定，非穷举。
- **典型错配**：`fw_capture_operation_log`（capture 前缀，抓取日志）易被误写为 `fw_verify_operation_log`（verify 前缀，校验状态）。
- **访问方式**：ptm-tde 仓库无 atoms/fw/ 副本，消费方一律通过运行时 CLI：`ptm-atomic list --format json` 取清单、`ptm-atomic show <op_id>` 取 `inputs.params`/`preconditions`/`side_effect`。禁止拷贝或跨仓库软链。

### op yaml preconditions 契约（P1-4 真相源）

`fw_capture_operation_log.yaml` 已含 op 级 preconditions：
```yaml
preconditions:
  - External orchestration holds a valid session_ref or runner session file.
  - When timetype is custom, both starttime and endtime must be provided as Unix-second timestamps.
postconditions:
```
**结论**：P1-4 不是新造字段，是对齐已有真相源，透传即可。

### GATE-4 自检扩展性评估（阶段 1 前置评估结论）

| 校验能力 | 现状 | 扩展成本 | 决策 |
|---|---|---|---|
| op_id 命中 ptm-atomic 清单 | `probe_ptm_atomic()`（run_checkpoint.py:377-395）已调用 `ptm-atomic list --format json`；`extract_step_atomic_refs()`（:915）已能提取 step op_id | 低（复用已有 probe + 提取逻辑，做集合差集） | **内联 GATE-4** |
| args 覆盖 required params | 无；需新增 `ptm-atomic show <op_id>` 调用 + params schema 解析 | 中（新增 show 调用 + required 集合差集） | 先内联评估 |
| args 值 type/枚举对齐 | 无；需解析 op yaml `parameters` 的 type 与枚举 | 中高（schema 解析） | 先内联，过复杂则切备选脚本 |
| preconditions 字段存在性 | GATE-4 P2 有 case_steps/step_name/op_id 回链，无 preconditions | 低（字段存在性检查） | **内联 GATE-4** |

**P1-3 备选脚本决策**：主选内联 GATE-4 扩展（op_id 命中 + preconditions 存在性 + required 差集先内联）；若实施中 op yaml schema 解析超过预期复杂度，切换 `scripts/check_pc_case_steps.py` 备选。切换条件记录于风险表。

## 五维度影响分析

### 1. 需求影响

| 维度 | 影响 |
|------|------|
| 现有需求 | 不变。ptm-tde 编排流程、三阶段框架、MFQ(PPDCS) 分析链不受影响 |
| 新增需求 | ①op_id 必须从 ptm-atomic 真实清单选取 ②case_steps 新增 preconditions 双层 ③args 四约束 ④GATE-4 新增 op_id 命中机器校验 |
| 需求冲突 | 无。ptm-tde 无存量 PC（CR-025 已确认 `ppdcs/` 下无已产出 PC），不破坏基线 |

### 2. 设计影响

| 维度 | 影响 |
|------|------|
| 架构决策 | CR-019 的 `atomic_op.op_id` 契约扩展为 `atomic_op.{op_id, args, preconditions}`；case_steps 顶层新增 `step_preconditions`/`depends_on` |
| HLD 影响 | 无。ptm-tde 自身架构不变，仅 PC 契约字段扩展 |
| ADR 影响 | 新增 ADR-02：op_id 真相源规则（运行时 CLI 优先，禁止读 atoms/fw 文件）；ADR-03：preconditions 双层语义（op 级透传 vs step 级自填） |

### 3. Story/实现影响

| 维度 | 影响 |
|------|------|
| 受影响文件 | `agents/ptm-tde.md`、`skills/{design-ppdcs-analyzer,deliverable-renderer,process-design,state-design}/SKILL.md`、`docs/ptm-tde/gate-spec.md`、`skills/checkpoint-manager/scripts/run_checkpoint.py` |
| 复杂程度 | 中等。契约字段扩展 + GATE-4 自检逻辑扩展，不涉及编排流程变更 |
| 依赖 | 无 Story 拆解（契约层 + 自检扩展，单 CR 内完成） |

### 4. 安全/权限影响

| 维度 | 影响 |
|------|------|
| 凭据 | 无 |
| 运行时授权 | 无。GATE-4 自检调用 `ptm-atomic list/show` 为只读 dry-run，不触发 `--execute` |
| 风险 | 低。契约字段扩展与自检增强，无运行时副作用；ptm-atomic 不可用时 GATE-4 降级为 warning 不阻断（复用现有 probe 降级逻辑） |

### 5. 交付影响

| 维度 | 影响 |
|------|------|
| 安装器 | 无。ptm-tde 已安装，契约字段扩展不影响安装 |
| 文档 | `agents/ptm-tde.md` + 4 skill + `gate-spec.md` 增量更新；`run_checkpoint.py` 脚本扩展 |
| 向后兼容 | 完全兼容。无存量 PC，无下游依赖旧契约；preconditions/depends_on 为新增可选字段，旧 PC（无）不受影响 |

## 修改文件清单

### 阶段 1：P0-2 op_id 真相源规则

| 优先级 | 文件 | 变更内容 | 变更类型 |
|--------|------|----------|---------|
| P0 | `agents/ptm-tde.md` | 596 行 args 规则上方新增 op_id 选择规则：①op_id 必须从 `ptm-atomic list` 真实清单选取，禁止自创 ②前缀语义非穷举表（capture=抓取日志/verify=校验状态/config/update/delete=状态变更/login=会话/check=只读查询/reset=不可逆）③capture vs verify 禁令（抓取日志类禁用 verify 前缀）④选取步骤（list 定位->show 核对->无法命中走候选） | 追加规则 |
| P0 | `agents/ptm-tde.md` | 595 行 GATE-4 阻断条件追加：op_id 未命中 `ptm-atomic list` 清单且未登记为候选 -> 阻断 | 修改规则 |
| P0 | `docs/ptm-tde/gate-spec.md` | GATE-4 P2（323 行）扩展通过条件：`atomic_op.op_id` 必须命中 `ptm-atomic list --format json` 清单（ptm-atomic 不可用时降级 warning） | 修改 checklist |
| P0 | `skills/checkpoint-manager/scripts/run_checkpoint.py` | GATE-4 自检新增 op_id 命中校验：复用 `probe_ptm_atomic()` 的 list 结果 + `extract_step_atomic_refs()` 提取的 op_id 做集合差集，未命中且非候选 -> FAIL | 脚本扩展 |
| P0 | `process/changes/CR-026-...md` | impact_process_refs 登记 CAP-003 gap；实施时确认外置 artifact repo 是否已有 CAP-003 文件，无则关闭时创建 follow-up tracking | CR 内登记 |

### 阶段 2：P1-4 preconditions 双层透传

| 优先级 | 文件 | 变更内容 | 变更类型 |
|--------|------|----------|---------|
| P1 | `agents/ptm-tde.md` | case_steps 契约新增 `atomic_op.preconditions`（透传 op yaml，op 级）+ `step_preconditions`（用例自填，step 级）；明确两者语义边界与正交关系 | 契约扩展 |
| P1 | `skills/{design-ppdcs-analyzer,process-design,state-design}/SKILL.md` | case_steps YAML 示例新增 preconditions 双层字段 | 示例扩展 |
| P1 | `skills/deliverable-renderer/SKILL.md` | 16 列表"预置条件"列改为从 `atomic_op.preconditions` + `step_preconditions` 并集渲染；渲染规则与"测试步骤"列对称 | 渲染规则扩展 |
| P1 | `docs/ptm-tde/gate-spec.md` | GATE-4 P2 追加：`atomic_op.preconditions` 存在性检查（op yaml 有则 PC 必须透传） | checklist 扩展 |

### 阶段 3：P1-3 可执行参数四约束

| 优先级 | 文件 | 变更内容 | 变更类型 |
|--------|------|----------|---------|
| P1 | `agents/ptm-tde.md` | case_steps 约束段新增四约束：①具体值（禁占位 `<xxx>`/`TBD`）②完整性（覆盖 op yaml required params）③语义对齐（值 type 对齐 op yaml type/description，对象名参数填对象名非 CIDR）④depends_on（step 间产出依赖，如 STEP-2 取 STEP-1 的 id） | 约束追加 |
| P1 | `agents/ptm-tde.md` | 明确 preconditions / step_preconditions / depends_on 三者正交：preconditions=op 执行前置，step_preconditions=用例数据前置，depends_on=step 间控制流依赖 | 规则澄清 |
| P1 | `docs/ptm-tde/gate-spec.md` | GATE-4 P2 追加：args 占位符扫描（正则）、required 差集校验（ptm-atomic show 取 required 集合） | checklist 扩展 |
| P1 | `skills/checkpoint-manager/scripts/run_checkpoint.py` | GATE-4 自检扩展 args 校验：占位符正则 + required 差集；type/枚举校验先内联，过复杂切备选脚本 | 脚本扩展 |

### 阶段 4：P1-5 状态固化

| 优先级 | 文件 | 变更内容 | 变更类型 |
|--------|------|----------|---------|
| P2 | `agents/ptm-tde.md` | 626 行追踪链表"PC -> 原子操作"行补注：case_steps -> 16 列渲染已闭环（deliverable-renderer），op_id 命中校验已纳入 GATE-4 | 状态补注 |

### 文档处理决策

| 受影响对象 | 处理方式 | 旧基线保留方式 |
|-----------|---------|--------------|
| `agents/ptm-tde.md` | 原文档增量更新 | 追加 `## 修订记录`，旧契约保留在 git 历史 |
| 4 个 skill SKILL.md | 原文档增量更新 | 追加 `## 修订记录`，旧示例保留在 git 历史 |
| `docs/ptm-tde/gate-spec.md` | 原文档增量更新 | 追加修订记录行（v2.0） |
| `run_checkpoint.py` | 脚本扩展 | git 历史 |

## 复杂度判定

**判定：standard（可压缩）**。

- 改动面：7 文件（4 文档 + 1 gate-spec + 1 脚本 + 本 CR）契约层扩展
- 无 Story 拆解（契约层 + 自检扩展，单 CR 内完成）
- 无新增文件（除本 CR）
- 无外部接口变更（ptm-atomic list/show 为只读复用）
- 无权限/安全边界变更（GATE-4 自检不触发 --execute）
- 无存量 PC 受影响（ppdcs/ 下无已产出 PC）
- **CP1-CP5 处理**：因是 meta-flow 自身 agent/skill 契约层整改、无存量 PC、无外部接口，CP1 场景/CP2/CP3 HLD/CP4 Story 标 n/a（无场景需求变更、无 HLD 架构变更、无 Story 拆解）；保留 CP0/CP6/CP7/CP8 追溯证据
- 风险：低。契约字段扩展与自检增强，ptm-atomic 不可用时降级不阻断

## 文件所有权冲突分析

| 现有 CR | 状态 | 冲突 | 处理 |
|---------|------|------|------|
| CR-025 | closed | 无（CR-025 改 args 命名，CR-026 改 op_id/preconditions/args 约束，同 case_steps 契约但字段正交） | 无冲突，CR-025 已关闭 |
| CR-019 | delivered | 无（CR-019 立 atomic_op.op_id 契约，CR-026 扩展 preconditions/args，向后兼容） | 扩展不破坏 CR-019 基线 |
| CR-024 | closed | 无（CR-024 是 ptm-te，CR-026 是 ptm-tde，不同文件；op_id 命中校验为 op_mapper 简化提供前置） | 无冲突 |
| CR-018 | delivered | 无（CR-018 立 GATE-4 字段级检查，CR-026 新增 op_id 命中项，同 gate-spec 但扩展不冲突） | 扩展不破坏 CR-018 基线 |

## 验证方法

- `validation_mode=static-only`（契约文档）+ `dry-run-only`（GATE-4 自检脚本逻辑，不触发 --execute）
- **P0-2 验证**：
  - `agents/ptm-tde.md` 含 op_id 选择规则 + 前缀语义表 + capture/verify 禁令
  - `gate-spec.md` GATE-4 P2 含 op_id 命中校验通过条件
  - `run_checkpoint.py` GATE-4 自检含 op_id 命中差集逻辑；ptm-atomic 不可用时降级 warning
  - 占位 PC（含 `fw_verify_operation_log` 错配）触发 GATE-4 FAIL；正确 op_id 通过
- **P1-4 验证**：case_steps 契约含 `atomic_op.preconditions` + `step_preconditions` 双层；deliverable-renderer 预置条件列从并集渲染
- **P1-3 验证**：args 占位符（`<xxx>`/`TBD`）触发 GATE-4 FAIL；required 差集（缺 page/size/timetype）触发 FAIL
- **P1-5 验证**：ptm-tde.md:626 追踪链表含 case_steps->16 列渲染闭环补注
- 安装器 dry-run 验证 ptm-tde 安装不受影响

## 风险与回退

| 风险 | 等级 | 缓解 | 回退路径 |
|------|------|------|---------|
| op yaml schema 解析复杂度超预期，type/枚举校验难内联 | 中 | 先内联 required 差集（低成本），type/枚举评估后决定 | 切换备选 `scripts/check_pc_case_steps.py`，GATE-4 只保留 op_id 命中 + preconditions 存在性 + 占位符 + required 差集 |
| GATE-4 自检扩展影响现有 P2 检查 | 低 | 新增校验作为 P2 子项追加，不改动现有 op_id 回链逻辑 | 移除新增子项，保留现有 P2 |
| ptm-atomic 不可用时 GATE-4 误阻断 | 低 | 复用 `probe_ptm_atomic()` 降级逻辑：不可用时 op_id 命中校验降级为 warning，不 FAIL | 降级路径已内建 |
| CAP-003 gap 已在外置 repo 登记，本 CR 重复登记 | 低 | 实施时先查外置 artifact repo `process/changes/` | 若已登记则引用已有路径，不重复 |

## CAP-003 gap 处理

P0-2 提及的 CAP-003 gap（op_id 命名错配追踪）在本仓库无命中（`process/` 软链至 `../meta-flow-artifacts/process/ptm-team`）。本 CR 在 `impact_process_refs` 登记 CAP-003，实施时确认：
1. 外置 artifact repo 是否已有 CAP-003 文件 -> 有则引用
2. 无则在 CR-026 关闭时创建 `CR-026-FOLLOW-UP-TRACKING-2026-07-13.md`，标注 CAP-003 状态为 `resolved-by-CR-026`

## Checkpoint Index

| CP | 类型 | 说明 | 状态 |
|----|------|------|------|
| CP0 | auto | 原始请求受理（本 CR 创建） | pass |
| CP1 | n/a | 场景完备（无场景需求变更，契约层整改） | n/a |
| CP2 | n/a | 需求基线（无需求变更） | n/a |
| CP3 | n/a | HLD（无架构变更，PC 契约字段扩展） | n/a |
| CP4 | n/a | Story 拆解（无 Story，单 CR 内完成） | n/a |
| CP5 | n/a | LLD（契约层文档，无 LLD） | n/a |
| CP6 | auto | 编码完成（7 文件修改，阶段 1-4 全部完成） | pass |
| CP7 | auto | 验证完成（static-only + dry-run-only：语法检查 + 函数就位 + op_id 命中拦截 + 占位符拦截 + preconditions 透传） | pass |
| CP8 | auto | 交付就绪（CP8 检查 PASS，交付清单与验证证据见 `process/checks/CP8-CR-026-DELIVERY-READY.md`） | pass |

> standard 模式（可压缩）：meta-flow 自身 agent/skill 契约层整改，无场景/HLD/Story 变更，CP1-CP5 标 n/a；保留 CP0/CP6/CP7/CP8 追溯证据与终验摘要。

## 后续 CR 候选

| 编号 | 描述 | 优先级 | 状态 |
|------|------|--------|------|
| F-01 | ptm-te op_mapper.py 简化（args->flag 退化为机械转换，op_id 命中校验为前置） | P1 | candidate（CR-024 follow-up） |
| 备选脚本 | `scripts/check_pc_case_steps.py`（若 GATE-4 type/枚举校验难内联） | P2 | candidate（视阶段 3 评估） |

## 参考

- CR-025：args 命名对齐（已 closed，本 CR 前置）
- CR-019：PC case_steps atomic_op 契约基线
- CR-018：GATE-4 字段级结构检查基线
- CR-024：ptm-te agent（op_mapper 适配的后续方）
- ptm-atomic 真相源：`/home/hyde/projects/ptm-atomic/atoms/fw/<op_id>.yaml`（op_id 清单 + inputs.params + preconditions）
- ptm-tde：`agents/ptm-tde.md`（PC 步骤结构化契约）
- gate-spec：`docs/ptm-tde/gate-spec.md`（GATE-4 P2 checklist）
- checkpoint-manager：`skills/checkpoint-manager/scripts/run_checkpoint.py`（GATE-4 自检执行器）

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|------|------|--------|---------|
| v1.0 | 2026-07-13 | host-orchestrator（主进程） | CR 创建，覆盖 P0-2/P1-3/P1-4/P1-5 四项整改，阶段 1 启动 |
| v1.1 | 2026-07-13 | host-orchestrator（主进程） | 阶段 1-4 全部完成：P0-2 op_id 真相源规则 + GATE-4 命中校验；P1-4 preconditions 双层透传；P1-3 args 四约束 + 机器校验；P1-5 状态补注。CP6/CP7 pass，7 文件 +255/-17。修正 [待确认] 归类（合法 needs-confirmation 标记，非占位符） |
| v1.2 | 2026-07-13 | host-orchestrator（主进程） | CP8 交付就绪检查 PASS（集成/降级/YAML/端到端验证全通过）；CR-026 关闭，CAP-003 gap 标记 resolved-by-CR-026（见 `CR-026-FOLLOW-UP-TRACKING-2026-07-13.md`） |
