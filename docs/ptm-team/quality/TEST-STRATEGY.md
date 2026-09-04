---
cr_id: CR-038
title: SW/PPPoE/多实例 TRex 执行层修复 测试策略（核心组）
version: "1.0"
validation_mode: mixed
validation_target:
  sut_type: code-project
  sut_ref: skills/topo-config + skills/topo-planning（纯 Python 数据模型/映射/归并/导出）
istqb_framework: true
iso_25010: true
release_artifact_profile: compact
release_decision: PENDING
checker: meta-qa
checked_at: 2026-08-15T00:00:00+00:00
wave_scope: "CR-038 核心组 6 个实现 Story（S01/S02/S03/S04/S06/S13）"
created_at: "2026-08-15T00:00:00+00:00"
---

# 测试策略 — CR-038 SW/PPPoE/多实例 TRex 执行层修复（核心组）

## 1. 验证模式声明

`validation_mode = mixed`

| 层 | 模式 | 必达/可选 | 说明 |
|---|---|---|---|
| 层A | runtime（fixture 造数 + pytest 单测） | **必达** | `skills/topo-config/tests/` + `skills/topo-planning/tests/` 造数，dry-run 等价验证，0 次真机写 |
| 层B | review-only / N/A | 可选 | 真机 H3C telnet / trex 发流 / `--execute` → **not_authorized**（N/A + 独立 runtime_authorization） |
| 平台安装 | dry-run-only / N/A | N/A | 本组纯内存引擎改动，不涉及平台目录/安装脚本/`PLATFORM-CONTRACTS.yaml` |

**测试环境声明**：本策略不要求真实运行环境（fixture 层 + dry-run 等价验证），`VALIDATION-ENV.yaml` 的 `approval.confirmed` 不适用（无需真机）。真机 H3C telnet（S03/S04）、trex 真机发流（S13/S10）、`--execute` 真机执行必须由 host-orchestrator 发起独立 `runtime_authorization` 决策项，本 CP7 不授权、不执行、不假装验证通过。

## 2. 测试设计方法选择（ISTQB）

| 方法 | 适用场景 | 适用性 | 应用说明 |
|---|---|---|---|
| 等价分区 | vendor（h3c/ruijie/huawei）、node_type（SW/PC/Mock/TG/DUT）、cipher 类型（env_placeholder/device_cipher/plaintext）、TRex 实例声明（declared/single/ambiguous/absent） | 高 | 每分区取代表值（三厂商命令族 parametrize、`_validate_password_cipher` 三分区、`_resolve_tg_instance` 决策表） |
| 边界值分析 | 密文正则边界（H3C `$c$3$`/Huawei `%^%#`/Ruijie `$1$/$5$/$6$`）、interface_kind 缺省 physical vs pppoe、node2 单值回退 | 中 | `test_password_cipher_*_cipher_ok` / `test_interface_info_interface_kind_default` / `test_build_nodes_node2_no_new_keys` |
| 状态转换测试 | 归并流程（pool_merge `_merge_sw_pc`/`_merge_tg`）、幂等（configure_pppoe_server existing none/partial/full）、MVP 拓扑判定 | 中 | `test_sw_merge_*` / `test_configure_pppoe_server_idempotent_*` / `test_mvp_*_rejected` |
| 错误推测 | 凭据明文/密文、真机降级拒绝、环回自环/闭环、显式 SW 与自动透传互斥 | 高 | `test_password_cipher_plaintext_rejected` / `test_configure_pppoe_server_non_h3c_real_deploy_denied` / `test_loopback_self_loop` / `test_explicit_sw_reserved_*` |

## 3. ISO 25010 质量特征优先级

| 质量特征 | 优先级 | 验证重点 | 对应验收维度 |
|---|---|---|---|
| 功能适合性 | P0 | SW 显式映射/归并、PPPoE 命令族+下发、exporter 环回 MVP、多实例 TRex 透传 | 完整性 / 验收标准覆盖 |
| 可靠性 | P0 | 结构化失败 reason、幂等、node2 单值回退、回归无破坏 | 平台适配 / 可安装性 |
| 安全性 | P0 | 凭据 `${ENV_*}` 占位禁止明文、cipher 不反解、`_resolve_env` 拒绝 PASSWORD/TOKEN/SECRET、真机 not_authorized | 安全合规 |
| 可维护性 | P1 | 私有方法下划线命名、确定性决策表、设计偏差留痕 | 命名规范 / Frontmatter 完整性 |
| 可移植性 | P1 | 纯内存引擎，无平台路径硬编码（平台差异 N/A） | 平台适配 |
| 易用性 | P2 | 结构化错误码（PPPOE_SERVER_INCOMPLETE/TREX_INSTANCE_*） | 文档覆盖 |
| 兼容性 | P2 | S06 duck-typing 与 S13 真实字段零改动闭环、S13 merge_owner 与 S02/S03/S06 基线兼容 | 验收标准覆盖 |
| 性能效率 | P3 | 全量测试 ~4.8s，无性能敏感路径 | — |

## 4. 质量门定义

| 门 | 判定 | 出口条件 |
|---|---|---|
| BLOCKING 维度 | 完整性 / 平台适配 / 验收标准覆盖 / 安全合规 | 全部通过（平台适配为 N/A 并写明理由） |
| REQUIRED 维度 | 命名规范 / Frontmatter / 可安装性 | 通过或记录豁免理由（可安装性 N/A：纯代码无安装脚本） |
| OPTIONAL 维度 | 文档覆盖 | 仅文档阶段检查 |
| 全量测试 | 340 passed / 0 failed | meta-qa 独立复跑与 CP6 报告一致 |

## 5. 入口准则（Entry Criteria）

- [x] 核心组 6 Story 状态 `ready-for-verification`（S01/S02/S03/S04/S06/S13）
- [x] CP6 编码完成门 PASS（6/6 CP6-CODING-DONE.md 齐全）
- [x] `validation_mode=mixed` 已判定；层A fixture + dry-run 不需真实运行环境；真机 not_authorized
- [x] 全部产物文件已创建（DEV-LOG 任务清单完成）

## 6. 出口准则（Exit Criteria）

- [x] 8 维度 BLOCKING 通过（平台适配 N/A 有理由）
- [x] 层A 全量 pytest 独立复跑 340 passed / 0 failed
- [x] VERIFICATION-REPORT.md 结论 `PASS_WITH_RISK`（真机 not_authorized + 开放项/设计偏差汇入 CP8）
- [x] CP7-STORY-038-{01,02,03,04,06,13}-VERIFICATION-DONE.md 写入

## 7. 安全边界（CP7 验证不得违反）

- 凭据 `${ENV_*}` 占位禁止明文；local-user 密码 cipher 密文不反解
- 默认 `--dry-run`；真机 H3C telnet / trex 发流 / `--execute` 独立 runtime_authorization，本 CP7 not_authorized
- GE1_1~4 实例禁改动，只操作 TE 实例与 TE2_1~4 端口
- 不得修改 REQUIREMENTS/HLD/ADR/蓝图三件套；不得修改代码（NEEDS_REWORK 只写清单不代改）
