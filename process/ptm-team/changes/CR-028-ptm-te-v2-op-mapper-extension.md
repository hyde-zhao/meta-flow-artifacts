---
change_id: CR-028-ptm-te-v2-op-mapper-extension
workflow_id: WF-PTM-TEAM-20260520-001
created_at: "2026-07-13T19:30:00+08:00"
created_by: host-orchestrator（主进程）
status: "closed"
implementation_status: delivered
closed_at: "2026-07-13T20:10:00+08:00"
workflow_mode: standard-compressed
engagement_mode: production
impact_level: medium
complexity: moderate
rollback_to: requirement-clarification
approval_source: CR-027 follow-up（v2 op_mapper 覆盖面扩展）+ nxp290-171 实战 P0-1
depends_on:
  - CR-026（closed，op_id 真相源规则定稿，run_checkpoint.py op_id 命中校验落地）
  - CR-027（closed，op_mapper v1.1 基线：args 对齐 + 参数预检 + NO_PROXY）
  - CR-024（closed，op_mapper v1 基线 + ADR-01 三层映射决策）
plan_ref: 无（改动集中在 op_mapper 映射表扩展 + 覆盖矩阵文档，本 CR 即计划）
cross_references:
  - CR-026（op_id 规范前置，已 closed）
  - CR-027（op_mapper v1.1 基线，已 closed）
  - CR-024（op_mapper v1 + ADR-01，已 closed）
lifecycle_status: "closed"
---

# CR-028 - ptm-te v2 op_mapper 覆盖面扩展（operation-log / object / interface 族 + op 覆盖矩阵）

## 变更请求摘要

CR-027 评估的 P0-1：op_mapper 仅覆盖 8 个 op_id（auth + policy-route 2 族），ptm-atomic 共 16 族 / 118 op，覆盖率 ~7%。实战执行 PC-DATA-LC-M10-01-01 时暴露 op_mapper 覆盖窄，非 policy-route 族用例无法执行。

本 CR 扩展 op_mapper 覆盖面至 **operation-log / object / interface 三个族**（7 个新子命令，约 9-10 个新 op_id），并建立 **op 覆盖矩阵文档**（118 op × 映射状态），跟踪 ptm-atomic 全量 op 与 op_mapper 映射差距。

CR-026 已定稿 op_id 真相源规则（op_id 必须从 `ptm-atomic list` 真实清单选取 + GATE-4 命中校验），ptm-tde 产出 op_id 已对齐 ptm-atomic，故 CR-028 不再需要 op_id 容错（原 P0-1 的容错需求降级）。

## 范围

| 族 | CLI 子命令 | 对应 op_id（atoms/fw/） | 说明 |
|----|-----------|----------------------|------|
| operation-log | capture | `fw_capture_operation_log` | 抓取操作日志（capture 前缀） |
| object | config | `fw_config_object` | 配置地址对象 |
| object | （delete/batch 待确认） | `fw_delete_object` / `fw_config_batch_object` | object 族 CLI 是否有 delete 子命令待调研确认 |
| interface | create | `fw_config_interface`（待确认 create vs config） | 创建接口 |
| interface | update | `fw_update_interface` | 更新接口 |
| interface | delete | `fw_delete_interface` | 删除接口 |
| interface | delete-batch | `fw_delete_batch_interface` | 批量删除接口 |
| interface | verify | （待确认，atoms 无 fw_verify_interface） | 查询接口 |

> 精确 op_id -> (family, action) 映射待 Agent 调研 atoms/fw/ yaml 元数据确认（op_id 名 ≠ 子命令名的情况，如 fw_config_interface 可能对应 interface create）。调研结论补全后实施。

## 调研结论

### 已确认（ptm-atomic run --help）

- ptm-atomic 共 **16 族**：auth, object, policy-route, interface, ospf, ospfv3, iplink, linkgroup, iptun, operation-log, export-file, import-file, reboot, ha, haiplink, halinkgroup
- atoms/fw/ 共 **118 个 op yaml**
- 目标 3 族子命令：operation-log(1) + object(1) + interface(5) = 7 子命令

### 已确认（Agent 调研 + 自行核实，2026-07-13）

- **7 新 op_id 精确映射**：见上方"终验摘要"
- **fw_config_interface 特殊映射**：op_id 含 config 但 action=create
- **安装版 object 族仅 config**：update/delete/delete-batch/verify 进 gap
- **fw_delete_interface 无 rollback 字段**：尊重真相源标 none

## 修改文件清单

| 文件 | 变更项 | 变更类型 |
|------|--------|---------|
| `skills/policy-route-execution/scripts/op_mapper.py` | OP_ID_TO_SUBCOMMAND + ARGS_TO_FLAGS + REQUIRED_FLAGS + ROLLBACK_STRATEGY + OP_METADATA 扩展 3 族；EXPECTED_OP_COUNT 更新；validate 校验扩展 | 脚本扩展 |
| `agents/ptm-te.md` | 三层映射章节补 3 族 op_id/子命令/flag 表；op_id 枚举说明从"8 个"改"15+ 个" | 文档扩展 |
| `docs/ptm-te/op-coverage-matrix.md` | 新建：118 op × 映射状态矩阵（mapped / gap / N/A） | 新增文档 |

### 文档处理决策

| 受影响对象 | 处理方式 | 旧基线保留方式 |
|-----------|---------|--------------|
| `agents/ptm-te.md` | 原文档增量更新 | 追加修订记录行 |
| `op_mapper.py` | 脚本扩展 | git 历史 |
| `op-coverage-matrix.md` | 新建 | 无旧基线 |

## 五维度影响分析

### 1. 需求影响
| 维度 | 影响 |
|------|------|
| 现有需求 | 不变。ptm-te 编排流程 [1]-[8] 不变，op_mapper 结构不变（表扩展） |
| 新增需求 | op_mapper 覆盖 operation-log/object/interface 族 + op 覆盖矩阵 |
| 需求冲突 | 无。无存量 run，不破坏基线 |

### 2. 设计影响
| 维度 | 影响 |
|------|------|
| 架构决策 | ADR-01（op_mapper 承载映射）不变；扩展是表数据增长，非结构变更 |
| HLD 影响 | 无。op_mapper 架构不变 |
| ADR 影响 | 无新增 ADR |

### 3. Story/实现影响
| 维度 | 影响 |
|------|------|
| 受影响文件 | 3 文件（op_mapper.py + agent md + 新矩阵文档） |
| 复杂程度 | 中等。映射表扩展 + 覆盖矩阵文档，需调研核对 op_id 精确映射 |
| 依赖 | 无 Story 拆解 |

### 4. 安全/权限影响
| 维度 | 影响 |
|------|------|
| 凭据 | 无 |
| 运行时授权 | 无。op_mapper 扩展是映射表，dry-run 默认门（ADR-04）不变 |
| 风险 | 低。object/interface config 类 op 有 side_effect，但 op_mapper 只承载映射，授权门控不变 |

### 5. 交付影响
| 维度 | 影响 |
|------|------|
| 安装器 | 无。ptm-te 已安装，脚本扩展不影响安装 |
| 文档 | agent md + 新覆盖矩阵文档 |
| 向后兼容 | 完全兼容。新增映射不影响现有 8 op_id |

## 文件所有权冲突分析

| 现有 CR | 状态 | 冲突 | 处理 |
|---------|------|------|------|
| CR-026 | closed | 无 | 已交付，op_id 规范为 CR-028 前置 |
| CR-027 | closed | 无 | 已交付，op_mapper v1.1 为 CR-028 基线 |
| CR-024 | closed | 无 | ADR-01 不变 |

## 验证方法

- `validation_mode=static-only`（映射表）+ `dry-run-only`（op_mapper map 子命令）
- op_mapper validate 扩展校验：3 族 op_id 全覆盖 + flag 格式 + rollback 一致性
- 新 op_id map 正例：每个新 op_id 用 args 输出正确 flag
- op 覆盖矩阵文档：118 op 全量，mapped/gap/N/A 标注完整
- 安装器 dry-run 验证 ptm-te 安装不受影响

## 风险与回退

| 风险 | 等级 | 缓解 | 回退路径 |
|------|------|------|---------|
| op_id -> 子命令映射误配（op_id 名 ≠ 子命令名） | 中 | Agent 调研 atoms/fw/ yaml 元数据确认，不凭文件名推断 | 改回正确映射 |
| object/interface 族有未发现的子命令 | 低 | 覆盖矩阵标 gap，后续 CR 补 | gap 跟踪 |
| 扩展后 validate 校验逻辑需更新 | 低 | EXPECTED_OP_COUNT + 族校验同步更新 | 改回 8 |

## Checkpoint Index

| CP | 类型 | 说明 | 状态 |
|----|------|------|------|
| CP0 | auto | 原始请求受理（本 CR 创建） | pass |
| CP1 | n/a | 场景完备（无场景需求变更，映射表扩展） | n/a |
| CP2 | n/a | 需求基线（无需求变更） | n/a |
| CP3 | n/a | HLD（无架构变更，op_mapper 结构不变） | n/a |
| CP4 | n/a | Story 拆解（无 Story，单 CR 内完成） | n/a |
| CP5 | n/a | LLD（映射表扩展，无 LLD） | n/a |
| CP6 | auto | 编码完成（3 文件修改） | pass |
| CP7 | auto | 验证完成（static-only + dry-run-only + 残留扫描） | pass |
| CP8 | auto | 交付就绪 | pass |

> standard-compressed 模式：meta-flow 自身 skill 映射表扩展 + 文档，无场景/HLD/Story 变更，CP1-CP5 标 n/a；保留 CP0/CP6/CP7/CP8 追溯证据与终验摘要。

## 终验摘要

CR-028 ptm-te v2 op_mapper 扩展已交付。op_mapper 从 8 op（2 族）扩展到 15 op（5 族），覆盖率从 ~7% 提升到 ~12.7%。

**调研确认结果**（Agent 调研 + 自行核实）：
- 7 新 op_id 精确映射已确认：operation-log(capture) + object(config) + interface(create/update/delete/delete-batch/verify)
- **关键发现**：`fw_config_interface` op_id 含 config 但 CLI action 是 `create`（`run_interface.py _run_create` 确认）
- **安装版 vs 源码差异**：安装版 ptm-atomic 0.1.0 的 object 族仅暴露 config；源码已有 5 子命令。op_mapper 按**安装版**（runtime 真相源）扩展，object 的 update/delete/delete-batch/verify 进覆盖矩阵 gap
- `ptm-atomic show` 与 yaml 不同步，扩展以 yaml + `build_subtree()` 代码为准
- `fw_delete_interface.yaml` 无 rollback_strategy 字段（实测），尊重真相源标 type=none

**验证结果**（static + dry-run + 残留扫描，继承 CR-027 教训）：

| 项 | 结果 | 证据 |
|----|------|------|
| op_mapper 语法 | ✅ | py_compile OK |
| validate 三表一致 + 5 族校验 | ✅ | PASS (15 op_id 全覆盖) |
| 7 新 op_id map 正例 | ✅ | action + flag 全部正确 |
| 原有 8 op 未破坏 | ✅ | policy-route / auth 仍正确 |
| 全局残留扫描 | ✅ | op_mapper 硬编码 8 残留 0；agent md '8 个 op' 残留 0 |
| op 覆盖矩阵文档 | ✅ | mapped 15 / gap 6 / unmapped 97 / total 118 |

**未验证项（runtime，留 T-01 follow-up）**：7 新 op 真实设备执行 / interface 嵌套 params 映射 / object config 回滚（inverse_op 未暴露）。

**不授权项**：映射表 + 文档扩展，不含运行授权；dry_run 默认门不变；新 op runtime 验证留 T-01。

## 后续 CR 候选

| 编号 | 描述 | 优先级 | 状态 |
|------|------|--------|------|
| CR-029 | op_mapper 扩展剩余族（ospf/iplink/linkgroup/iptun/ha 等） | P2 | candidate（视实战需求） |
| F-02 | P2-11 preconditions 消费（待 CR-026 契约定稿，已 closed 可启动） | P2 | candidate |

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|------|------|--------|---------|
| v1.0 | 2026-07-13 | host-orchestrator（主进程） | CR 创建，覆盖 operation-log/object/interface 3 族 + op 覆盖矩阵；精确 op_id 映射待 Agent 调研补全 |
