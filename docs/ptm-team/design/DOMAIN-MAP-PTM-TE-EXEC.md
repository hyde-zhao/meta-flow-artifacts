---
cr_id: "CR-033"
artifact_type: "domain-map"
version: "1.1"
created_at: "2026-07-28T11:15:00+08:00"
author: "meta-se"
status: "draft"
source_blueprint: "docs/design/BLUEPRINT-PTM-TE-EXEC.md"
---

# CR-033 ptm-te 执行引擎领域图

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-28 | meta-se | CR-033 领域对象/状态/规则/术语初稿 |
| 1.1 | 2026-07-28 | meta-se | CP3 评审范围扩展：环境文件驱动 resolve_env_refs。新增领域对象 EnvTopology/EnvRefPlaceholder/resolve_env_refs/load_env_file；术语表追加 env_topology/${ENV.*}/resolve_env_refs/load_env_file/--env-file；业务规则追加 BR-11（环境文件驱动） |

## 1. 领域对象

| 对象 | 定义 | 归属 Feature | 状态流 | 持久化 | 备注 |
|---|---|---|---|---|---|
| TGDevice | TG 测试仪设备（type:TG） | FE-EX-01 | 静态清单 | devices.yaml tg 块 | sub_type(ixia-c/trex) × hardware_platform(EP/C236/J1900) = 6 组合 |
| DUTDevice | 被测防火墙设备（type:firewall） | 既有 | 静态清单 | devices.yaml firewall 块 | host 字段用于 DUT_URL |
| TestCase | 用例文件（frontmatter 16 列 + case_steps YAML） | FE-EX-02 | draft -> ready -> running -> done | cases/三级/四级/五级/<编号>-<名称>.md | 执行真相源是 case_steps YAML |
| CaseStep | 用例步骤（step_id + target + atomic_op + expected_result） | FE-EX-02 | pending -> running -> pass/fail/known_fail/error | case_steps YAML 内 | target=DUT 或 TG |
| Envelope | op 执行结果封装（op_id/status/data/error_type） | 既有 | success / error | runs/<run-id>/step-refs/<step_id>.json | op_mapper 输出 |
| RunResult | 用例执行结果（四态 + 步骤结果 + 清理记录） | FE-EX-02 | running -> PASS/FAIL/KNOWN_FAIL/ERROR | runs/<run-id>/result.json | overall 字段四态之一 |
| RunReport | 结构化报告（四态统计 + 失败诊断 + 幂等容错） | FE-EX-02 | generated | runs/<run-id>/report.md | 人类可读 |
| Session | DUT 登录会话 | 既有 | login -> active -> invalid -> logout | ~/.local/state/ptm-atomic/ngfw/session-*.json | STATE_INVALID 自动重连 1 次 |
| StepRef | 步骤间引用数据包（step_id + op_id + args + envelope） | 既有 | written -> resolved | runs/<run-id>/step-refs/<step_id>.json | ${STEP-N.id} 插值源 |
| RuleBlock | install.py 规则托管块 | FE-EX-03 | rendered -> installed | CLAUDE.md managed block | ptm-te-workflow block_id |
| EnvTopology | 环境拓扑契约（port_mapping/nodes/links），由 case_runner load_env_file 加载 | FE-EX-02 | loaded -> resolved -> consumed | --env-file YAML（内存对象 env_topology） | ${ENV.*} 占位符解析源（ADR-09，CP3 评审范围扩展） |
| EnvRefPlaceholder | ${ENV.*} 占位符（引用环境参数），由 resolve_env_refs 解析 | FE-EX-02 | unresolved -> resolved | case_steps YAML args 内 | 8 类：tg.port1/port1.ip/port1.gw/url, dut.port1/port1.ip/url, tg.ports[] |
| resolve_env_refs | op_mapper 环境解析函数（与 resolve_step_refs 并列，build_command 前执行） | FE-EX-02 | pending -> resolved -> failed | op_mapper.py 内存 | 顺序：resolve_env_refs -> resolve_step_refs -> validate_args -> build_command |
| load_env_file | case_runner 环境文件加载函数（--env-file -> env_topology） | FE-EX-02 | not_loaded -> loaded | case_runner.py 内存 | ST-EX-04 实现 hook，ST-EX-17 实现 build_env_topology 契约校验 |

## 2. 状态流

### TestCase 状态流

```
draft --> ready --> running --> done
                       |            |
                       v            v
                    cleanup      PASS/FAIL/KNOWN_FAIL/ERROR
```

- draft：用例 md 编写中（frontmatter 不完整或 case_steps 缺失）
- ready：用例 md frontmatter 16 列完整 + case_steps YAML 可解析
- running：case_runner 正在执行
- done：执行结束，overall 字段为 PASS/FAIL/KNOWN_FAIL/ERROR 四态之一

### Session 状态流

```
login --> active --> (STATE_INVALID) --> reconnect(1次) --> active
                  --> logout --> cleaned
```

- login：fw_login_web_management 成功，session 文件创建
- active：session 有效，op 可执行
- STATE_INVALID：session 过期，op_mapper 自动重连 1 次
- logout：fw_logout 成功，session 文件清理

### RunResult 四态分级规则

| overall | 条件 | 说明 |
|---|---|---|
| PASS | 全部 step pass | 用例通过 |
| FAIL | 存在 step fail 且无 known_issue 标记 | 脚本 bug 或 DUT 异常 |
| KNOWN_FAIL | step fail 但有 known_issue 标记 | DUT 行为差异，符合预期 |
| ERROR | step error（OP_NOT_FOUND/PARAM_INVALID/EXEC_FAILED/UNKNOWN_ERROR） | 执行错误，非业务失败 |

### CaseStep 幂等容错规则

| op_id | error 条件 | 容错结果 | 说明 |
|---|---|---|---|
| fw_config_object | eRequestParamWithDetail + "存在同名"/"同名" | PASS（对象已存在） | 幂等 |
| tg_stop_traffic_stream | RESOURCE_NOT_FOUND | PASS（流不存在，已停止） | 幂等 |
| fw_delete_object | eBeingReferenced + expected_result 含"被阻止" | PASS（符合异常用例预期） | 已知行为 |

## 3. 业务规则

### BR-01: dry-run 默认门

- case_runner 默认 dry_run=True，不调用 execute_op，只调 build_command 校验命令构建
- --execute 标志才设 dry_run=False，且必须配合 --authorized
- dry_run=False 且 authorized=False 时返回 error_type=EXEC_FAILED

### BR-02: ARP 预热引擎强制（SGA-04=C）

- case_runner 解析 case_steps 中的 warming_up 字段
- warming_up:true 时，主 op（如 tg_start_traffic_stream）执行后强制执行 post_op（tg_stop_traffic_stream）
- 即使 md 未写 post_op，case_runner 自动补充，参数从主 op 继承
- 双重保障：install.py 规则定义 + case_runner 引擎强制

### BR-03: 逆序清理 mutation ops

- case_runner 每用例结束后，按 step-refs 逆序执行清理 op
- 清理 op：fw_delete_policy_route / fw_delete_object / tg_stop_traffic_stream
- 清理结果记入 result.json cleanup 字段

### BR-04: fw_logout 会话清理（DQ-02）

- case_runner 全部用例结束后调用 fw_logout
- op_mapper 映射 fw_logout -> (auth, logout)
- 安装前验证 `ptm-atomic show fw_logout`；未暴露时降级为清理 session 文件
- result.json 含 logout 状态

### BR-05: known_issue 四态分级（SGA-03=A）

- case_runner 解析 case_steps 中的 known_issue 字段
- known_issue 标记的 step 失败时输出 KNOWN_FAIL
- 未标记的 step 失败输出 FAIL
- ERROR 状态（OP_NOT_FOUND/PARAM_INVALID 等）不影响 known_issue 判定

### BR-06: retry 轮询

- case_runner 解析 case_steps 中的 retry 字段
- retry:{interval:3, max_attempts:5, success_condition:"rx_packets==0"}
- 每 interval 秒轮询，最多 max_attempts 次
- 任意一次满足 success_condition 即通过

### BR-07: 用例目录结构（R-F-022）

- cases/三级(功能模块)/四级(子模块)/五级(用例组)/<用例编号>-<用例名称>.md
- 三级=功能模块（如 IPv4策略路由）
- 四级=子模块（如 配置管理）
- 五级=用例组（如 策略配置）

### BR-08: 用例命名（R-F-023, DQ-06）

- <用例编号>-<用例名称>.md
- 用例编号保留组网前缀（如 PC-M1-01-01）
- 用例名称来自 frontmatter 用例名称列
- 文件名解析时按编号前缀（如 PC-M1-01-01）正则匹配，剩余部分为名称
- 名称内部连字符保留

### BR-09: frontmatter 16 列（R-F-024, DQ-05）

- 16 列：三级目录/四级目录/五级目录/用例名称/用例编号/用例级别/组网描述/组网约束/预置条件/测试步骤/预期结果/首次创建版本/最后变更版本/关键词/测试类型/是否自动化
- 必填 8 列：用例编号/用例名称/三级目录/四级目录/五级目录/用例级别/测试类型/是否自动化
- 可选 8 列：组网描述/组网约束/预置条件/测试步骤/预期结果/首次创建版本/最后变更版本/关键词/tags
- case_runner 解析时忽略"测试步骤"和"预期结果"列（AGA-03=C），只读 case_steps YAML

### BR-10: 标签执行（R-F-025/026）

- frontmatter tags 列：结构化列表（如 [策略路由, IPv4, PC, 正向, P0]）
- frontmatter 关键词列：逗号分隔（如 fw_config_policy_route, fw_verify_policy_route, S01）
- --tag 按 tags 列精确过滤（多 tag AND 逻辑，逗号分隔）
- --keyword 按关键词列模糊匹配（子串匹配）
- --tag 和 --keyword 可组合使用（AND 逻辑）

### BR-11: 环境文件驱动（R-F-027/028/029，ADR-09，CP3 评审范围扩展）

- case_runner 加载 --env-file（port_mapping/nodes/links）-> env_topology
- op_mapper execute_op 签名加 env_topology 参数；执行顺序：resolve_env_refs -> resolve_step_refs -> validate_args -> build_command
- ${ENV.*} 占位符 8 类：tg.port1/port1.ip/port1.gw/url, dut.port1/port1.ip/url, tg.ports[port1,port2]
- 环境相关参数用 ${ENV.*}（禁止字面值）；测试意图参数保持字面值
- resolve_env_refs 解析失败 -> VALIDATION_FAILED envelope
- 未含 ${ENV.*} 字面值原样透传（向后兼容）
- TREX_API_URL 来源：环境文件 ${ENV.tg.url} 优先，devices.yaml tg.api_server fallback（ADR-05）
- DUT 接口自动预配置：--execute 模式 fw_update_interface 按 nodes.dut1.interfaces；用例后逆序清理
- ptm-atomic 约束：TG 操作经 ptm-atomic run tg trex <action>，框架禁止直接调 TG REST API

## 4. 术语表

| 术语 | 定义 | 来源 |
|---|---|---|
| TG | Traffic Generator，测试仪（ixia-c 或 trex） | traffic-skill |
| DUT | Device Under Test，被测设备（防火墙） | 既有 |
| op_id | 原子操作 ID（如 fw_config_policy_route） | op_mapper |
| envelope | op 执行结果封装（op_id/status/data/error_type） | op_mapper |
| case_steps | 用例步骤 YAML 块，执行真相源 | exec_v4.py / case_runner |
| frontmatter | 用例 md 顶部元数据（16 列） | CP2 v1.1 |
| dry-run | 默认安全门，只校验命令构建不执行写操作 | SGA-01=A |
| --execute | 实际执行写操作，需 --authorized 授权 | SGA-01=A |
| runtime_authorization | 运行时授权审计字段（who/scope/authorized_at/reason） | R-NF-002 |
| known_issue | DUT 行为差异标记，step 失败输出 KNOWN_FAIL | SGA-03=A |
| warming_up | ARP 预热探测流标记，引擎强制执行 post_op | SGA-04=C |
| mutation op | 有副作用的 op（config/update/delete/start-stream），需清理 | op_mapper |
| inverse_op | 回滚清理 op（如 config 的 inverse 是 delete） | op_mapper |
| step-refs | 步骤间引用数据包，支持 ${STEP-N.id} 插值 | op_mapper |
| extract_payload | 统一解析函数，按 op_id 从 envelope 提取字段 | R-F-014 |
| fw_logout | 新增 op，映射 auth logout，用于会话清理 | R-F-016 |
| TREX_API_URL | TG api_server 环境变量，op_mapper 注入 | R-F-018 |
| env_topology | 环境拓扑契约（port_mapping/nodes/links），${ENV.*} 解析源 | R-F-027/ADR-09 |
| ${ENV.*} | 环境参数占位符（端口/IP/URL），resolve_env_refs 解析 | R-F-027/ADR-09 |
| resolve_env_refs | op_mapper 环境解析函数，与 resolve_step_refs 并列 | R-F-029/ADR-09 |
| load_env_file | case_runner 环境文件加载函数（--env-file -> env_topology） | R-F-027/ADR-09 |
| --env-file | 环境文件路径参数，含 port_mapping/nodes/links | R-F-027/ADR-09 |
