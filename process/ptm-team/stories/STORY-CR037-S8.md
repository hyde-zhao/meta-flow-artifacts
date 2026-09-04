---
story_id: "CR037-S8"
story_slug: "ptm-te-integration-install"
cr_id: "CR-037"
title: "ptm-te 编排 [1]→[2] 集成 + install.py（topo_id 双源提取 + PTM_TE_SKILLS + 模板副本 + release 挂接）"
priority: "P0"
wave: 3
status: "lld-ready"
depends_on: ["CR037-S7", "CR037-P1"]
dependency_type: "precedence"
assignee: null
dev_context: "agents/ptm-te.md 编排流程 [1] 用例解析 → [2] 设备准备之间插入 [1.5] 拓扑映射步骤（E1）；topo_id 双源提取：优先「组网约束」字段、回退「组网描述」括号内提取（E2）；PTM_TE_SKILLS 5→7 增 topo-config + topo-planning（方案 A）；install.py 首次生成目标项目 topology/physical_pool.yaml + physical_ledger.yaml 模板副本（P1-8：源=skills/topo-config/config/、幂等跳过、.gitignore 沿用 topology/*.yaml、不含造数、台账 list+port_status P0-3）；copy_skill_tree 裁剪 tests/__pycache__/*.pyc 保证造数隔离成立（S8-A）；dry-run 落预览 env-file 到 runs/<run-id>/（executable:false，S8-B）；component-resource-links.yaml 增 ptm-te→network-topology 关联（S8-C）；PhysicalPool.release(env_name) 挂接编排 [7] 清理（P1-7）；授权分层：映射/台账占用=执行前自动步骤不需 DQ-037-04，真机下发（S9）才需 DQ-037-04（S8-E）。"
validation_context: "映射失败 TOPO_MAPPING_FAILED 终止；release 挂接 [7] 清理（dry-run 跳过、失败记 cleanup_summary 不阻断 [8]）；模板副本幂等（已有跳过）、不含测试造数；copy_skill_tree 裁剪后 .claude/skills/ 无 tests/；dry-run 预览 env-file 可被 load_env_file 解析（S8-B）；layer2 collection 资源随装（S8-C）；PTM_TE_SKILLS 安装后 topo-config + topo-planning 可 import。"
acceptance_criteria:
  - "编排 [1.5] 拓扑映射步骤插在 [1]→[2] 之间，失败 TOPO_MAPPING_FAILED 终止"
  - "topo_id 双源提取：组网约束优先、组网描述括号回退（E2）"
  - "PTM_TE_SKILLS 5→7 增 topo-config + topo-planning（方案 A）"
  - "install.py 首次生成目标项目模板副本：源=skills/topo-config/config/、幂等跳过、.gitignore 沿用、不含造数、台账 list+port_status（P1-8/P0-3）"
  - "copy_skill_tree 裁剪 tests/、__pycache__/、*.pyc（S8-A），安装产物 .claude/skills/ 无测试造数"
  - "dry-run 落预览 env-file 到 runs/<run-id>/（executable:false），下游 load_env_file 可解析（S8-B）"
  - "component-resource-links.yaml 增 ptm-te→network-topology 关联，install 后 layer2 collection.md 可及（S8-C）"
  - "PhysicalPool.release 挂接编排 [7] 清理（P1-7），dry-run 跳过、失败记 cleanup_summary 不阻断 [8]"
  - "授权分层（S8-E）：[1.5] 映射+台账占用不需 DQ-037-04；真机下发（S9）才需 DQ-037-04"
output_files:
  - "agents/ptm-te.md"
  - "script/install.py"
  - "resource/component-resource-links.yaml"
file_ownership:
  primary:
    - "agents/ptm-te.md#编排流程 [1]→[2]"
    - "script/install.py#PTM_TE_SKILLS + copy_skill_tree 裁剪 + 模板副本生成"
    - "resource/component-resource-links.yaml#ptm-te 组件关联（S8-C）"
  shared: []
  merge_owner: null
  forbidden: []
feature_design_refs:
  - "F-CR037"
lld_policy:
  required_level: "full-lld"
  trigger_reasons:
    - "跨 skill 编排集成 + 安装流程修改"
  rationale: "编排接入点 + 模板副本生成机制 + release 挂接需完整设计"
dev_gate: "cp5-approved"
design_evidence_ref: "process/stories/STORY-CR037-S8-LLD.md"
---

# STORY-CR037-S8：ptm-te 集成 + install.py

## 目标

ptm-te 编排流程集成拓扑映射步骤 + install.py 安装增强（skill 关联 + 模板副本生成 + release 挂接）。

## 依赖

CR037-S7（topo-planning skill + CLI）——编排 [1.5] 调用入口。
**CR037-P1（physical-pool-template）——模板副本源前置依赖（Minor-2）**：`ensure_target_project_templates` 的源 `skills/topo-config/config/{physical_pool,physical_ledger}.yaml` 必须是 P-1 占位化后的模板；若 P-1 未实现则源仍为真实数据（A1600-HU + 密码），模板副本会泄漏。实现前置校验「P-1 模板已占位化」（源含具体型号特征串即 fail-fast，见 LLD §6.2 `_assert_p1_template_placeholderized`）。

## 输出文件

- `agents/ptm-te.md`
- `script/install.py`

## AI 任务清单

- [ ] TASK-S8-01：编排 [1.5] 拓扑映射步骤插入（E1）+ TOPO_MAPPING_FAILED 终止 + dry-run 预览 env-file 链路（S8-B）
- [ ] TASK-S8-02：topo_id 双源提取（组网约束优先/组网描述回退，E2）
- [ ] TASK-S8-03：PTM_TE_SKILLS 5→7（增 topo-config + topo-planning）
- [ ] TASK-S8-04：copy_skill_tree 过滤复制（排除 tests/__pycache__/*.pyc，S8-A）
- [ ] TASK-S8-05：install.py 首次生成模板副本（P1-8，幂等 + .gitignore + 不含造数）
- [ ] TASK-S8-06：component-resource-links.yaml ptm-te→network-topology 关联（S8-C）
- [ ] TASK-S8-07：PhysicalPool.release 挂接编排 [7]（P1-7）
- [ ] TASK-S8-08：授权分层对齐（S8-E：[1.5] 映射+台账占用不需 DQ-037-04；真机下发 S9 才需）

## 设计证据

见 `process/stories/STORY-CR037-S8-LLD.md`（full-lld）。
