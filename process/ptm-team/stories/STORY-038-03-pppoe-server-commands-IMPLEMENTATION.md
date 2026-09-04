---
doc_type: IMPLEMENTATION
story_id: "STORY-038-03"
story_slug: "pppoe-server-commands"
cr_id: "CR-038"
wave: 1
status: "done"
created_at: "2026-08-15"
owner: "meta-dev"
source_lld: "process/stories/STORY-038-03-pppoe-server-commands-LLD.md"
source_story: "process/stories/STORY-038-03.md"
design_evidence_type: "full-lld"
risk_level: "high"
---

# STORY-038-03 实现执行证据 — PPPoE Server 命令族 + 数据源契约

## 实现前置检查

| 检查项 | 结果 |
|--------|:---:|
| Story `status=dev-ready` | PASS |
| 设计证据（full-lld 14 段）已确认，`dev_gate=cp5-approved` | PASS |
| CP5 全量确认已通过（`process/checkpoints/CP5-ALL-STORIES-LLD-BATCH.md` + STATE.current.json `current_phase=story-execution`） | PASS |
| `depends_on: []`（W1 起点，无上游依赖） | PASS |
| 文件所有权不冲突：primary = commands.py + physical_pool.yaml；shared = physical_pool.py（本 Story 首次写入方，merge_owner=STORY-038-13） | PASS |
| 平台约束：不涉及安装路径/平台目录，`PLATFORM-CONTRACTS.yaml` N/A（仅命令模板 + 数据源 schema） | N/A |
| 未修改 REQUIREMENTS / HLD / ADR / 蓝图三件套 | PASS |
| 阻断 clarification item：`LCQ-STORY-038-03-01`（密文前缀 allowlist 精确性）为 non-blocking-open | PASS（不阻断） |

## 实现对象清单

| 文件 | 操作 | 说明 |
|------|:---:|------|
| `skills/topo-config/src/commands.py` | 修改 | 三厂商 `COMMANDS[vendor]['pppoe']` 五族键（enable_server/ip_pool/local_user/virtual_template/bind_interface，嵌套 dict 模板）+ `PPPOE_COMMAND_KEYS` 常量 + `has_pppoe_commands(brand)` 能力探测 |
| `skills/topo-config/src/physical_pool.py` | 修改 | 新增 `get_pppoe_server` / `validate_pppoe_server` / `_validate_password_cipher` 三方法 + 模块级凭据/字段校验正则常量（`_ENV_PLACEHOLDER_RE` 等 7 个） |
| `skills/topo-config/config/physical_pool.yaml` | 修改 | 新增 SW3 示例节点（`role: pppoe-server` + `pppoe_server` 块，`password_cipher` 用 `${ENV_*}` 占位） |
| `skills/topo-config/tests/test_commands_pppoe.py` | 新建 | 三厂商命令族单测（16 用例） |
| `skills/topo-config/tests/test_physical_pool_pppoe.py` | 新建 | 数据源 + 校验单测（29 用例） |
| `skills/topo-config/tests/fixtures/pool_pppoe_server.yaml` | 新建 | pppoe_server 测试造数 fixture（合成占位密文，无真实凭据） |

## 设计契约映射

| 契约 | 实现点 |
|------|--------|
| R-F-003（PPPoE Server 命令族） | `COMMANDS[vendor]['pppoe']` 五族键模板；完整命令序列由 STORY-038-04 `_build_pppoe_server_commands` 组装（本 Story 只定义模板，不组装） |
| R-F-004（三厂商命令模板） | h3c 参考实现 + ruijie/huawei dry-run 模板，五族键齐全、文本按厂商差异 |
| R-F-019（数据源同源一致） | `get_pppoe_server` 唯一真相源 = `_device_by_name[name]['pppoe_server']`，无 devices.yaml / 独立配置回退；`from_pool_data` / `_build_index` 未改结构 |
| R-C-001（凭据 `${ENV_VAR}` 占位禁止明文） | `_validate_password_cipher` 规则 1（`${ENV_*}` 放行）+ 规则 3（其余明文拒绝，不兜底放行） |
| R-C-002（local-user 密码保持 cipher 密文，不反解） | `_validate_password_cipher` 规则 2（设备密文前缀判定 + 原样透传，代码无任何解密逻辑）；注入 `{cipher}` 原样透传 |
| ADR-CR038-03（数据源 = physical_pool `pppoe_server` 块） | `get_pppoe_server` 读取路径 + `pppoe_server` schema（§4 数据模型） |
| FR1（三厂商五族键集一致） | `PPPOE_COMMAND_KEYS` 五元组 + `has_pppoe_commands` 校验齐全 |
| NFR1（向后兼容） | `get_commands` / `COMMANDS` 既有键不变；`pppoe` 键为增量新增 |
| NFR3（安全：不打印明文到日志） | `get_pppoe_server` / `validate_pppoe_server` / `_validate_password_cipher` 均不 `logger` 打印 `password_cipher` 值 |

## 单元测试与 Fixture 计划

- `test_commands_pppoe.py`（16 用例）：五族键齐全（3 厂商 parametrize）、H3C 参考文本精确断言、三厂商 diff（SCN-038-POS-004）、dry-run 模板非空（R-NF-001）、`has_pppoe_commands` 正/反例、`get_commands` 向后兼容 + 未知品牌仍抛 ValueError。
- `test_physical_pool_pppoe.py`（29 用例）：`get_pppoe_server` present/absent/single-source（R-F-019）、`validate_pppoe_server` 全字段（含 `PPPOE_SERVER_INCOMPLETE` 短路、INVALID_TYPE、SUBNET/GATEWAY/VIRTUAL_TEMPLATE/AUTH/LOCAL_USERS_EMPTY/USERNAME 错误码）、`_validate_password_cipher`（`${ENV_*}` 占位 / H3C `$c$3$` / Huawei `%^%#` / Ruijie `$1$` 密文放行、明文拒绝、畸形占位符、不反解）。
- Fixture：`pool_pppoe_server.yaml`（sw_pppoe 完整块 / sw_no_pppoe 无块 / sw_plain 普通 SW）；密文样例均为合成占位密文（`$c$3$QUJDREVGR0g=`、`%^%#ABCDEFG12345#%^%#`、`$1$salt$abcdefghijklmnopqrstuv`），不落真实凭据。
- TDD：按 LLD §8 四切片（01a+01c → 02 → 01b → 04）先写测试再实现。

## 最小实现切片

| 切片 | TASK-ID | 内容 | 验证 |
|------|---------|------|:---:|
| 1 | TASK-E2-03-01a + 01c | `get_pppoe_server` 只读 + physical_pool.yaml SW3 `pppoe_server` 示例块 | R-F-019 同源一致 |
| 2 | TASK-E2-03-02 | commands.py 三厂商 `pppoe` 五族键 + `has_pppoe_commands` | R-F-004 键齐全 + diff |
| 3 | TASK-E2-03-01b | `validate_pppoe_server` + `_validate_password_cipher` 校验逻辑 | ADR-02 / R-C-001/002 |
| 4 | TASK-E2-03-04 | 全量单测收口 | 45 用例全绿 |

## 平台差异处理

- 本 Story 只产出「命令模板定义 + 数据源 schema + 校验函数」，不涉及安装路径/平台目录，`PLATFORM-CONTRACTS.yaml` 与 `PLATFORM-INSTALL-SPEC.md` 均 **N/A**。
- 命令模板随 `topo-config` skill 统一安装（沿用 CR-037 安装链路），本 Story 不新增安装脚本。

## 验证结果

- 命令：`uv run --python 3.11 pytest skills/topo-config/tests/test_commands_pppoe.py skills/topo-config/tests/test_physical_pool_pppoe.py` → **45 passed**。
- 命令：`uv run --python 3.11 pytest skills/topo-config/tests/` → **135 passed**（新增 45 + 回归 90，全绿）。
- 安全校验：`password_cipher` 明文拒绝、`${ENV_*}` 占位 / 三厂商 cipher 密文放行（ADR-02）、设备密文不反解（R-C-002）——见 `test_physical_pool_pppoe.py` 凭据系列用例。

## 未覆盖项

- 命令序列组装（`_build_pppoe_server_commands`）→ STORY-038-04 `switch_configurator.py`。
- 真机下发（H3C telnet）→ 独立 runtime_authorization（非本 Story）。
- SCN-038-POS-004 三厂商渲染后 diff → STORY-038-04 渲染后验证（本 Story 只验证键齐全 + 文本互异）。
- 设备 cipher 密文前缀 allowlist 精确性（`LCQ-STORY-038-03-01`）→ 保守前缀白名单已确定性写入，真机密文样例复核后固化。
- 回滚闭环（DEF-038-04）→ 延后。

## 设计缺口反馈

- **LLD §10.1 Huawei 密文正则未转义 `^`**：LLD 写 `^%^%#[A-Za-z0-9+/=_.]+#%^%#$`，其中 `^` 在 Python `re` 中（模式中段）仍被当作「行首锚点」而非字面量，导致按原文实现的 regex 无法匹配真实 `%^%#...#%^%#` 密文。实现按 LLD 语义意图（`%^%#` 字面前/后缀包裹）转义为 `%\^%#...#%\^%#`。此为对 LLD 正则字面拼写的实现层修正，不改语义；已在单测 `test_password_cipher_huawei_cipher_ok` 覆盖。若后续正式文档需要同步，可走 design delta 回填 LLD 正则。

## 后续交接

- 消费接口（给 STORY-038-04 / meta-qa）：`COMMANDS[vendor]['pppoe']`（五族键模板）、`get_pppoe_server`、`validate_pppoe_server`、`has_pppoe_commands`、`_validate_password_cipher`。
- 验证入口：`uv run --python 3.11 pytest skills/topo-config/tests/test_commands_pppoe.py skills/topo-config/tests/test_physical_pool_pppoe.py -v`。
- 风险提示：ruijie/huawei 命令为 dry-run 模板未经真机验证（Q-038-002）；设备 cipher 前缀 allowlist 为保守白名单（LCQ-STORY-038-03-01）。
- 文件所有权：physical_pool.py 本 Story 为首次写入方，只新增 `pppoe_server` 读取/校验三方法 + 7 个模块级常量；未触碰 `_build_index` / `from_pool_data` / `get_free_ports` / `get_interfaces`（STORY-038-13 merge_owner 后续合并 trex_instances）。
