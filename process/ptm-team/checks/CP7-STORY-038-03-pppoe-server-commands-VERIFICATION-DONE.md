---
story_id: STORY-038-03
story_slug: pppoe-server-commands
cr_id: CR-038
checkpoint: CP7
result: PASS_WITH_RISK
validation_mode: mixed
created_at: "2026-08-15"
author: meta-qa
design_evidence_type: full-lld
---

# CP7-STORY-038-03-pppoe-server-commands-VERIFICATION-DONE

## 结论

| 项目 | 内容 |
|---|---|
| 阶段决策 | PASS_WITH_RISK |
| validation_mode | mixed（pytest + dry-run 等价验证） |
| 路由 | none（verified；真机 H3C telnet not_authorized） |

## 验证对象清单

| 文件 | 操作 | 验证结果 |
|---|---|---|
| `skills/topo-config/src/commands.py` | 修改（三厂商 `COMMANDS[vendor]['pppoe']` 五族键 + `has_pppoe_commands`） | PASS |
| `skills/topo-config/src/physical_pool.py` | 修改（`get_pppoe_server`/`validate_pppoe_server`/`_validate_password_cipher`） | PASS |
| `skills/topo-config/config/physical_pool.yaml` | 修改（SW3 示例 `pppoe_server` 块，`${ENV_*}` 占位） | PASS |
| `test_commands_pppoe.py`/`test_physical_pool_pppoe.py` + fixture | 新增（45 用例） | PASS |

## 验证追踪矩阵

| 场景/契约 | 测试 | 结果 |
|---|---|---|
| SCN-038-POS-004（三厂商命令族） | `test_commands_pppoe_five_keys[h3c/ruijie/huawei]` + `test_commands_pppoe_three_vendor_diff` | PASS |
| R-F-019（同源一致） | `test_get_pppoe_server_{present,single_source}` | PASS |
| R-C-001（`${ENV_*}` 占位） | `test_password_cipher_env_placeholder_ok` | PASS |
| R-C-002（cipher 不反解） | `test_password_cipher_{h3c,huawei,ruijie}_cipher_ok` + `test_password_cipher_not_reversed` | PASS |
| 明文拒绝 | `test_password_cipher_plaintext_rejected` | PASS |

## 设计契约验证

- R-F-003/004：五族键（enable_server/ip_pool/local_user/virtual_template/bind_interface）三厂商齐全，文本按厂商差异
- R-F-019：`get_pppoe_server` 唯一真相源 = physical_pool `pppoe_server` 块，无独立配置回退
- R-C-001/002：`_validate_password_cipher` 三分区确定性（`${ENV_*}` → env_placeholder；H3C `$c$3$`/Huawei `%^%#`/Ruijie `$1$/$5$/$6$` → device_cipher；其余明文拒绝），不反解
- 设计偏差 S03：Huawei 密文正则 LLD §10.1 `^` 未转义，实现转义 `%\^%#` 已留痕（语义等价修正）

## 分层验证计划

- 层A：`test_commands_pppoe.py` + `test_physical_pool_pppoe.py` 独立复跑 45 passed；全量 340 passed / 0 failed
- 层B：N/A（H3C 真机 telnet not_authorized）

## 8 维度验收

| # | 维度 | 结果 |
|---|---|---|
| 1 | 完整性（BLOCKING） | PASS（6 产物 ≥ 验收标准） |
| 2 | 平台适配（BLOCKING） | PASS（N/A：命令模板 + 数据源 schema，无平台路径） |
| 3 | 验收标准覆盖（BLOCKING） | PASS（POS-004/R-F-019/R-C-001/002 全覆盖） |
| 4 | 安全合规（BLOCKING） | PASS（`${ENV_*}` 占位，0 明文，密文不反解） |
| 5 | 命名规范（REQUIRED） | PASS |
| 6 | Frontmatter（REQUIRED） | PASS |
| 7 | 可安装性（REQUIRED） | PASS（N/A：随 topo-config skill 统一安装，不新增安装脚本） |
| 8 | 文档覆盖（OPTIONAL） | 文档阶段检查 |

## 测试证据

- `uv run --python 3.11 pytest skills/topo-config/tests/test_commands_pppoe.py skills/topo-config/tests/test_physical_pool_pppoe.py -q` → 45 passed
- 全量 → 340 passed, 0 failed

## 问题与缺陷

| # | 严重度 | 问题 | 状态 |
|---|---|---|---|
| Q1 | Minor | LCQ-STORY-038-03-01：密文前缀 allowlist 保守，可能拒绝未列入的合法设备密文 | 真机密文样例复核后固化 |

## 剩余风险

- cipher allowlist 保守（LCQ-03-01）
- ruijie/huawei 命令为 dry-run 模板未经真机验证（Q-038-002）
- H3C 真机下发 not_authorized（独立 runtime_authorization）

## 阶段决策

PASS_WITH_RISK → verified；风险（cipher allowlist + ruijie/huawei 未真机 + 真机下发不授权）汇入 CP8。
