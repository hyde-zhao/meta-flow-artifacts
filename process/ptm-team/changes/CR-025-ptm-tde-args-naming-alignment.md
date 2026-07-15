---
change_id: CR-025-ptm-tde-args-naming-alignment
workflow_id: WF-PTM-TEAM-20260520-001
created_at: "2026-07-13T00:00:00+08:00"
created_by: host-orchestrator（主进程）
status: closed
implementation_status: delivered
workflow_mode: fast-lane
engagement_mode: production
impact_level: low
complexity: simple
rollback_to: requirement-clarification
approval_source: CR-024 T-01 follow-up
depends_on:
  - CR-024（ptm-te v1 已交付，closed）
  - ptm-atomic（外部真相源，op yaml inputs.params 命名）
plan_ref: 无（fast-lane，改动面小，本 CR 即计划）
cross_references:
  - CR-024（ptm-te，op_mapper 需适配简化后的 args 命名）
  - CR-019（PC case_steps atomic_op 契约，本 CR 修正 args 命名约定）
---

# CR-025 - ptm-tde args 命名对齐 op yaml 参数名

## 变更请求摘要

ptm-tde 产出物理用例（PC）时，`case_steps[].atomic_op.args` 的字段名使用自定义命名（如 `src_addr`），与 ptm-atomic op yaml `inputs.params` 的标准参数名（如 `source_network`）不一致。这导致 ptm-te 的 `op_mapper.py` 必须手写三层映射表（ptm-tde args → op yaml params → CLI flag），增加维护成本和漂移风险。

**将 ptm-tde 的 `args` 字段名强制对齐到 op yaml `inputs.params` 的标准参数名**（snake_case），使三层映射退化为两层（op yaml params → CLI flag），args→flag 映射从手写退化为机械 snake_case→kebab-case 转换。

## 背景

CR-024 实现 ptm-te 时，ADR-01 锁定了"ptm-te op_mapper 承载三层映射翻译，ptm-tde args 不变"的决策。当时约束是 ptm-tde 已交付，改 args 命名破坏基线。但 CR-024 CP8 明确将"真实消费 ptm-tde PC"（T-01）和"ptm-tde args 命名对齐"作为后续优化方向。

当前三层命名不一致：

| 层 | 来源 | 示例 |
|----|------|------|
| 第 1 层（当前 ptm-tde） | `case_steps[].atomic_op.args` | `src_addr` |
| 第 2 层（op yaml 真相源） | `atoms/fw/fw_*.yaml` 的 `inputs.params` | `source_network` |
| 第 3 层（CLI flag） | `ptm-atomic run ... --*` | `--source-network` |

**整改后**：第 1 层 = 第 2 层（`source_network`），只剩第 2→3 层机械转换。

## 调研结论

- **无存量 PC 受影响**：`ppdcs/` 下无已产出 PC 文件，不需要重生成。
- **op yaml 真相源**：`/home/hyde/projects/ptm-atomic/atoms/fw/fw_config_policy_route.yaml` 的 `inputs.params` 定义 `source_network`、`dst_network`、`next_hop_ip`、`in_interface`、`type` 等字段名，与 CLI flag `--source-network` 等 snake_case↔kebab-case 一一对应。
- **改动面**：2 个文件 × 各 1-3 行命名修正 + 1 条新增规则。

## 五维度影响分析

### 1. 需求影响

| 维度 | 影响 |
|------|------|
| 现有需求 | 不变。ptm-tde 编排流程、三阶段框架、MFQ(PPDCS) 分析链不受影响 |
| 新增需求 | `args` 字段名对齐 op yaml `inputs.params`（snake_case） |
| 需求冲突 | 无。ptm-tde 无存量 PC，不破坏任何基线 |

### 2. 设计影响

| 维度 | 影响 |
|------|------|
| 架构决策 | CR-024 ADR-01 的"ptm-tde args 不变"前提被本 CR 替换："ptm-tde args 对齐 op yaml params" |
| HLD 影响 | 无。ptm-tde 自身架构不变 |
| ADR 影响 | CR-024 ADR-01 回写：ptm-tde args 对齐后，op_mapper 的 args→flag 映射退化为机械转换 |

### 3. Story/实现影响

| 维度 | 影响 |
|------|------|
| 受影响文件 | `agents/ptm-tde.md`（PC 契约示例 + 约束规则）、`skills/deliverable-renderer/SKILL.md`（PC 步骤渲染示例） |
| 复杂程度 | 简单。命名修正，不涉及逻辑变更 |
| 依赖 | 无。单文件修改，无需 Story 拆解 |

### 4. 安全/权限影响

| 维度 | 影响 |
|------|------|
| 凭据 | 无 |
| 运行时授权 | 无 |
| 风险 | 极低。命名规范修正，无运行时影响 |

### 5. 交付影响

| 维度 | 影响 |
|------|------|
| 安装器 | 无。ptm-tde 已安装，命名修正不影响安装 |
| 文档 | `agents/ptm-tde.md` 增量更新 + `skills/deliverable-renderer/SKILL.md` 增量更新 |
| 向后兼容 | 完全兼容。无存量 PC，无下游依赖旧命名 |

## 修改文件清单

| 优先级 | 文件 | 行号 | 旧内容 | 新内容 | 变更类型 |
|--------|------|------|--------|--------|---------|
| P0 | `agents/ptm-tde.md` | L586-587 | `src_addr: OBJ_SRC_WEB` | `source_network: OBJ_SRC_WEB` | 修改示例 |
| P0 | `agents/ptm-tde.md` | L591-595 | 约束段（4 条） | 追加第 5 条：`args` 字段名必须使用 op yaml `inputs.params` 参数名（snake_case），禁止自定义别名 | 追加规则 |
| P0 | `skills/deliverable-renderer/SKILL.md` | L152-154 | `src-addr: OBJ_SRC_WEB` | `source_network: OBJ_SRC_WEB` | 修改示例 |
| P0 | `skills/deliverable-renderer/SKILL.md` | L163-168 | 渲染示例 `src-addr=OBJ_SRC_WEB` | `source_network=OBJ_SRC_WEB` | 修改示例 |

### 文档处理决策

| 受影响对象 | 处理方式 | 旧基线保留方式 |
|-----------|---------|--------------|
| `agents/ptm-tde.md` | 原文档增量更新 | 追加 `## 修订记录`，旧示例保留在 git 历史 |
| `skills/deliverable-renderer/SKILL.md` | 原文档增量更新 | 追加 `## 修订记录`，旧示例保留在 git 历史 |

## 复杂度判定

**判定：fast-lane**。

- 改动面：2 文件 × 共 4 处命名修正 + 1 条规则追加
- 无 Story 拆解（单文件修改，无依赖）
- 无新增文件
- 无外部接口变更
- 无权限/安全边界变更
- 低风险：命名规范修正，无运行时影响，无存量 PC 需重生成

## 文件所有权冲突分析

| 现有 CR | 状态 | 冲突 | 处理 |
|---------|------|------|------|
| CR-024 | closed | 无（CR-024 是 ptm-te，CR-025 是 ptm-tde，不同文件） | 无冲突 |
| ptm-tde 已交付基线 | delivered | 本 CR 修改 ptm-tde 基线 | 明确 CR 范围：仅修正 args 命名约定，不改 ptm-tde 编排流程/三阶段框架 |

## 对 ptm-te 的影响（CR-024 后续）

ptm-tde args 对齐后，ptm-te 的 `op_mapper.py` 可简化：

- **前**：`ARGS_TO_FLAGS` 手写映射表（7 个 op × 各 4-5 行，约 30 行）
- **后**：`_to_flag(arg_name: str) -> str: return "--" + arg_name.replace("_", "-")`（1 行机械转换）

op_mapper 的简化留给 CR-024 follow-up（T-01 或独立 CR），本 CR 不直接改 op_mapper.py。

## 验证方法

- `validation_mode=static-only`
- 检查 `agents/ptm-tde.md` L586-587 示例使用 `source_network`（非 `src_addr`）
- 检查约束段含"args 字段名必须使用 op yaml inputs.params 参数名"
- 检查 `skills/deliverable-renderer/SKILL.md` 示例使用 `source_network`
- 安装器 dry-run 验证 ptm-tde 安装不受影响

## 风险与回退

| 风险 | 等级 | 缓解 | 回退路径 |
|------|------|------|---------|
| 未来 ptm-tde 产出 PC 时 args 命名漂移 | 低 | 新增规则写入 agent md 约束段 | 规则执行 |

## Checkpoint Index

| CP | 类型 | 说明 | 状态 |
|----|------|------|------|
| CP0 | auto | 原始请求受理（本 CR 创建） | pending |
| CP1 | auto | 场景完备（n/a，fast-lane，命名规范修正无需场景） | n/a |
| CP2 | n/a | fast-lane 跳过 CP2 | n/a |
| CP3 | n/a | fast-lane 跳过 HLD/蓝图 | n/a |
| CP4 | n/a | fast-lane 跳过 Story 拆解 | n/a |
| CP5 | n/a | fast-lane 跳过 LLD | n/a |
| CP6 | auto | 编码完成（2 文件修改） | pending |
| CP7 | auto | 验证完成（static-only） | pending |
| CP8 | auto | 交付就绪 | pending |

> fast-lane 模式：低风险轻量实现，跳过 CP1-CP5，保留 CP0/CP6/CP7/CP8 追溯证据。

## 后续 CR 候选

| 编号 | 描述 | 优先级 | 状态 |
|------|------|--------|------|
| F-01 | ptm-te op_mapper.py 简化（args→flag 退化为机械转换） | P1 | candidate |
| T-01 | 真实消费 ptm-tde PC + runtime 端到端验证 | P1 | candidate（CR-024 follow-up） |

## 参考

- CR-024：ptm-te agent 实现（ADR-01 三层映射决策）
- ptm-atomic 真相源：`/home/hyde/projects/ptm-atomic/atoms/fw/fw_config_policy_route.yaml`（`inputs.params`）
- ptm-tde：`agents/ptm-tde.md`（PC 步骤结构化契约）
- deliverable-renderer：`skills/deliverable-renderer/SKILL.md`（PC 步骤渲染规则）