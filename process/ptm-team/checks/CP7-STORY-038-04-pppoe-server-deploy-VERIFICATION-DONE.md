---
story_id: STORY-038-04
story_slug: pppoe-server-deploy
cr_id: CR-038
checkpoint: CP7
result: PASS_WITH_RISK
validation_mode: mixed
created_at: "2026-08-15"
author: meta-qa
design_evidence_type: technical-note
---

# CP7-STORY-038-04-pppoe-server-deploy-VERIFICATION-DONE

## 结论

| 项目 | 内容 |
|---|---|
| 阶段决策 | PASS_WITH_RISK |
| validation_mode | mixed（pytest + dry-run 等价验证） |
| 路由 | none（verified；真机 H3C telnet not_authorized） |

## 验证对象清单

| 文件 | 操作 | 验证结果 |
|---|---|---|
| `skills/topo-config/src/switch_configurator.py` | 修改（`configure_pppoe_server`/`_build_pppoe_server_commands` + 错误码常量 + `from_pool` 回填） | PASS |
| `skills/topo-config/tests/test_switch_configurator_pppoe.py` | 新增（15 用例） | PASS |
| `skills/topo-config/tests/fixtures/ledger_pppoe_server.yaml` | 新增 | PASS |

## 验证追踪矩阵

| 场景/契约 | 测试 | 结果 |
|---|---|---|
| SCN-038-POS-003（H3C dry-run 生成命令） | `test_configure_pppoe_server_h3c_dry_run_commands` / `test_configure_pppoe_server_from_pool_reads_block` | PASS |
| SCN-038-REC-001（幂等） | `test_configure_pppoe_server_idempotent_{skip_pool_and_user,partial,none_creates_all}` | PASS |
| 缺块/vendor 不支持 | `test_configure_pppoe_server_{missing_block,incomplete_block,non_dict_block,vendor_unsupported}` | PASS |
| R-NF-001 降级（ruijie/huawei 真机拒绝） | `test_configure_pppoe_server_non_h3c_real_deploy_denied[ruijie/huawei]` | PASS |
| 密文透传不反解 | `test_configure_pppoe_server_password_cipher_not_reversed` | PASS |

## 设计契约验证

- R-F-003：命令序列按 ip pool → local-user → virtual-template → bind interface 顺序构建，复用 `_build_vlan_commands` 模板渲染
- REC-001/R-NF-003：先查再建，`existing` 传入已存在资源时跳过不重建
- R-NF-001 降级：ruijie/huawei `dry_run=False` 拒绝真机下发（`PPPOE_NON_H3C_REAL_DEPLOY_DENIED`）
- 设计偏差 S04：virtual-template IP 复用 `ip_pool_gateway`（schema 无独立 server-ip 字段）、bind_interface 未指定物理接口（flat 模板），待真机确认，已留痕

## 分层验证计划

- 层A：`test_switch_configurator_pppoe.py` 独立复跑 15 passed；全量 340 passed / 0 failed
- 层B：N/A（H3C 真机 telnet not_authorized）

## 8 维度验收

| # | 维度 | 结果 |
|---|---|---|
| 1 | 完整性（BLOCKING） | PASS（3 产物 ≥ 验收标准） |
| 2 | 平台适配（BLOCKING） | PASS（N/A：纯命令构建，无平台路径） |
| 3 | 验收标准覆盖（BLOCKING） | PASS（POS-003/REC-001/缺块/vendor 全覆盖） |
| 4 | 安全合规（BLOCKING） | PASS（默认 dry-run，cipher 原样透传不反解，无明文日志） |
| 5 | 命名规范（REQUIRED） | PASS |
| 6 | Frontmatter（REQUIRED） | PASS |
| 7 | 可安装性（REQUIRED） | PASS（N/A：纯代码） |
| 8 | 文档覆盖（OPTIONAL） | 文档阶段检查 |

## 测试证据

- `uv run --python 3.11 pytest skills/topo-config/tests/test_switch_configurator_pppoe.py -q` → 15 passed
- 全量 → 340 passed, 0 failed

## 问题与缺陷

| # | 严重度 | 问题 | 状态 |
|---|---|---|---|
| Q1 | Minor（设计偏差） | virtual-template IP 复用 ip_pool_gateway + bind_interface 未指定物理接口，待真机确认 | 留痕，汇入 CP8 |

## 剩余风险

- virtual-template IP/bind_interface 映射待真机确认（设计偏差 S04）
- H3C 真机下发 not_authorized（独立 runtime_authorization）

## 阶段决策

PASS_WITH_RISK → verified；风险（设计偏差 S04 + 真机下发不授权）汇入 CP8。
