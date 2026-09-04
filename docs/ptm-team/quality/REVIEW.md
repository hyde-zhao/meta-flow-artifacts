---
cr_id: CR-037
title: ptm-te 集成 topo 管理功能 代码/设计评审报告
version: "1.0"
review_scope: "CR-037 全部 12 个实现 Story 独立质量评审（对照 CP5 设计证据检查实现偏离合理性）"
review_mode: "mixed（层A fixture runtime + 平台安装 dry-run）"
reviewer: "meta-qa (independent, CP7)"
review_date: "2026-08-05T19:30:00+08:00"
---

# CR-037 独立质量评审报告（REVIEW）

> 独立性声明：本评审由 meta-qa 独立完成，未盲信 CP6 result。评审重点：实现是否偏离设计证据、偏离是否合理、是否有安全/安装/隔离风险。

## 1. 评审范围

- **设计证据基线**：`CR-037-HLD.md` v0.4 + `STORY-CR037-S1..S9-LLD.md`（full-lld）+ P-1/S6/S9/S10 technical-note + S11 waived
- **实现对象**：skills/topo-planning（6 模块 + CLI + SKILL + 模板）、skills/topo-config（mapper/pool/mapping_validator + 模板 + fixtures）、install.py ×2、agents/ptm-te.md、resource/component-resource-links.yaml、文档

## 2. 设计契约符合性总评

| 设计契约 | 实现偏离 | 偏离合理性 | 结论 |
|---|---|---|---|
| S1 `find_topology_file` 返回 YAML 路径（P0-1） | 增加开发仓库形态路径查找 + PTM_TOPOLOGY_CACHE_DIR 可覆盖 | 合理（兼容安装/开发双形态；环境变量隔离测试） | ✅ 采纳 |
| S1 parser.py 依赖补齐 | S1 未改 parser.py，由 S8 安装器统一 PYTHONPATH 承担 | 合理（避免越权修改 topo-config，file_ownership 边界正确） | ✅ 采纳 |
| S2 R4 冲突消歧 | `_disambiguate_alias_by_pool` 优先 hardware_platform 等价类交集（非 OR 逻辑） | 合理（修复 A1500-HU 双命中缺陷） | ✅ 采纳 |
| S3 `UnifiedPool.links` dict 形态 | LLD 注释写 List[Dict]，实现保留 dict | 合理（对齐 physical_pool.yaml 实际 schema） | ✅ 采纳 |
| S4 MappingResult 字段位置 | 落在 topo_mapper.py 而非 topology_model.py | 合理（MappingResult 实际定义于 topo_mapper.py） | ✅ 采纳 |
| S4 `_check_connectivity` 修复同设备对多 link 复用 cable | 增加 `used_ports \| ports_needed` | 合理（修复既有缺陷，per-link 语义正确） | ✅ 采纳 |
| S5 export 增加 generated_at 参数 | 为 R2 确定性注入 | 合理（可选 kwarg 向后兼容） | ✅ 采纳 |
| S6 `_supplement_direct_domains` 补充 TG/PC↔DUT p2p 域 | `_compute_broadcast_domains` 仅覆盖 DUT-DUT | 合理（MVP 单 TG 单 DUT 必需适配层） | ✅ 采纳 |
| S8 同步修改 `script/ptm_team/install.py` | LLD scope 只列 script/install.py | 合理（保持两安装器一致，既有测试要求） | ✅ 采纳 |
| S9 EnvironmentDeployer 懒加载 | 避免未安装 requests 环境 import 失败 | 合理（纯构造/测试场景可用） | ✅ 采纳 |

**总评**：全部 10 项实现偏离均为「合理修正 / 边界澄清 / 适配层」，无架构违背；无未记录偏离。

## 3. 关键设计点独立复核

| 设计点 | 独立复核 | 结论 |
|---|---|---|
| H4 叠加公式（用户 speed_class 优先 + TE 系强制 fiber 丢弃逻辑 copper） | 阅读 `_resolve_per_link_requirement` 源码：逻辑正确，`_logical_topo_link_requirement` 双端点 speed 不一致时不设 speed（保守） | ✅ |
| include-at-least-one 软约束（soft 分支位于硬剪枝前） | 阅读 `_port_meets_requirement`：`if not strict: return True` 在硬剪枝 return False 之前；post-match `_count_satisfied_links` ≥1 判定 | ✅ |
| ledger 闭环（allocate 写回 + save_ledger + release） | `_commit_allocation` 只写 used_ports；`_is_port_busy` 消费台账 allocated_to；`release_env` 薄封装 + save_ledger | ✅ |
| 端口兼容矩阵（GE=copper\|fiber、TE 系=fiber-only） | `PORT_COMPAT_MATRIX` 常量与 S2 limit_parser 同源语义 | ✅ |
| MVP 边界显式拒绝 | `_validate_mvp_topology` 在 export 前拦截多节点/SW/Mock | ✅ |

## 4. 安全与隔离评审

- **凭据边界（ADR-02）**：模板 8 特征串 0 命中；env-file 导出路径不读 password/token；DeployBridge 只收环境变量名。✅
- **造数隔离**：install dry-run 打印「排除 tests/__pycache__/*.pyc」（S8-A）；模板副本源为 config 占位模板；fixture 是唯一造数位置。✅
- **授权分层（S8-E）**：[1.5] 映射/台账占用 = workspace 写（S7 `--authorized` 显式门）；真机下发 = S9 `DeployBridge` 独立 DQ-037-04。✅
- **危险命令**：新增脚本 grep 扫描 0 命中（rm -rf/sudo/eval/os.system 等）。✅

## 5. Findings（按严重度排序）

### Minor-1（建议回修）参考用例 TG 设备组与池 TG 节点关联失败

- **位置**：`skills/topo-planning/tests/fixtures/devices_direct.yaml`（tg-dut1 组）+ `pool_direct_tg_dut.yaml`（tg-dut1 节点）
- **现象**：参考用例 CLI dry-run 输出「未匹配 devices.yaml 节点: tg-dut1」
- **根因**：`devices_direct.yaml` TG 组仅 `tg.host: 10.113.52.253`（无 `tg.api_server`）；池 TG 节点仅 `management.api_server: http://10.113.52.253:8450`（无 `management.host`）。S3-C 关联主键为 api_server（归一化精确匹配），host 兜底需要**池侧** `management.host`；两者均不满足 → 关联失败。
- **影响**：TG 仍从池数据直接可用（device_type/hardware_platform 池侧已含），映射产物正确；告警仅 cosmetic。参考用例未覆盖「TG api_server 主键关联成功」路径（该路径由 S3 `test_merge_tg_api_server_association` 单测覆盖）。
- **建议**：修正 `devices_direct.yaml` 的 tg-dut1 组增加 `api_server: "http://10.113.52.253:8450"`（或池节点补 `management.host`），使参考用例演示完整 TG 关联路径。非阻断。
- **决策类型**：implementation（follow-up candidate）

### Minor-2（已知限制）`script/ptm_team/install.py` 未加 `ensure_target_project_templates`

- **位置**：`script/ptm_team/install.py`（包级兼容入口）
- **现象**：主安装器 `script/install.py` 具备模板副本生成；`script/ptm_team/install.py` 只扩展了 PTM_TE_SKILLS 5→7 与 copy_skill_tree 裁剪语义，无模板副本生成。
- **影响**：经包级入口安装时目标项目不会自动生成 `topology/physical_pool.yaml` + `physical_ledger.yaml` 模板副本（用户需手动复制或走主安装器）。
- **建议**：后续 CR 将 `ensure_target_project_templates` 提取为共享函数并在两入口复用；当前主安装器路径不受影响。非阻断。
- **决策类型**：implementation（follow-up candidate）

### Info-1（状态同步）`STATE.current.json.active_change` 未同步 CR-037

- **现象**：`process/state/STATE.current.json` 的 `active_change` 仍为 "CR-036"。
- **处理**：交由 host-orchestrator 在 CP8 / CR-037 关闭时通过 `meta-flow cr status-sync` 同步。

## 6. 对 CP6 报告的独立发现对照

- CP6 S10 报告「适配前 28 failed / 适配后 82 passed」：meta-qa 独立复跑确认 topo-config 82 passed / 0 failed，适配未改变断言语义（既有 28 项失败归零，S4 新增 26 全绿）。
- CP6 S7 报告 `missing_te_port` 断言调整为「TE 约束失败被结构化报告」：meta-qa 复核 `build_failure_report` 仍含 `missing_te_port` 分支（S4 部分匹配 + 链路级 TE 失败场景），CLI 无 TE 池场景下走 `link_constraint_unsatisfied` + 设备级剪枝，属合理覆盖。
- CP6 S4 报告「修复同设备对多 link 复用 cable 缺陷」：meta-qa 复核改动正确，未回退既有失败基线（全量 271 一致）。

## 7. 结论

**REVIEW 结论：通过（无 Blocker/High/Medium）**。1 项 Minor（参考用例 TG 关联 fixture，建议回修）+ 1 项 Minor（安装器模板副本不对称，后续 CR）+ 1 项 Info（STATE 同步）。全部实现偏离设计证据均合理并有记录。
