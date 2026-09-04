---
status: confirmed
version: "1.2"
confirmed_by: "user-via-host-orchestrator-relay"
confirmed_at: "2026-07-28T11:30:00+08:00"
ready_for_design: true
source_use_cases: [UC-EX-01, UC-EX-02, UC-EX-03, UC-EX-04, UC-EX-05, UC-EX-06, UC-EX-07, UC-EX-08, UC-EX-09, UC-EX-10, UC-EX-11]
source_cr: "CR-033"
---

# ptm-te 执行引擎 - 结构化需求

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 | 文档处理方式 |
|---|---|---|---|---|
| 1.0 | 2026-07-28 | meta-pm | CR-033 初始需求基线：20 条功能需求 + 8 条约束 + 5 条非功能 | 新建文件（CR-033 引入新产品线 ptm-te-exec） |
| 1.1 | 2026-07-28 | meta-pm | CP2 范围修改：R-F-004 执行入口改为单用例/按目录/按标签或关键字；新增 R-F-022..026（用例目录结构/命名/frontmatter 16 列/tags 列/标签执行）；R-F-021 扩大整改范围（目录迁移+重命名+frontmatter+tags+ARP 预热） | 原文档增量更新；保留 v1.0 R-F-001..020 语义，R-F-004/R-F-021 更新，R-F-022..026 新增 |
| 1.2 | 2026-07-28 | meta-pm | CP3 评审范围扩展（环境文件驱动 resolve_env_refs）：新增 R-F-027..029（用例与环境解耦/DUT 接口自动预配置/设备 URL 自动解析）。来源：CP3 评审范围扩展（环境文件驱动） | 原文档增量更新；保留 v1.1 R-F-001..026 语义不变，R-F-027..029 新增 |

## 功能需求

| ID | 需求描述 | 优先级 | 验收条件 | 来源场景 |
|----|---------|--------|---------|---------|
| R-F-001 | devices.yaml 新增 tg 块，支持 type:TG + host + serial_url + sub_type(ixia-c/trex) + hardware_platform(EP/C236/J1900) + ssh + api_server | P0 | Given devices.yaml When 新增 TG 设备 Then tg 块含全部字段且 sub_type×hardware_platform 覆盖 6 组合 | UC-EX-01 |
| R-F-002 | device-management SKILL.md 新增 TG 设备添加/查询流程，含 TG_SSH_PASSWORD 环境变量 | P0 | Given SKILL.md When 查阅 TG 流程 Then 含 TG 添加流程 + TG_SSH_PASSWORD + 6 组合型号对照 | UC-EX-01 |
| R-F-003 | device-reference.md 补全 6 组合型号对照（2 子类型 × 3 硬件平台） | P0 | Given device-reference.md When 查阅 TG 型号 Then 6 组合全覆盖（ixia-c×EP/C236/J1900 + trex×EP/C236/J1900） | UC-EX-01 |
| R-F-004 | case_runner.py 支持三入口：--case-file(单用例) / --cases-dir(目录glob) / --tag(精确过滤)或--keyword(模糊匹配)。原 --case-files(文件列表) 已替换为 --tag/--keyword | P0 | Given case_runner.py When 用三种入口运行 Then 均能正确解析用例并执行，零代码新增用例。--tag 按 frontmatter tags 列精确匹配；--keyword 按 frontmatter 关键词列模糊匹配 | UC-EX-02, UC-EX-03 |
| R-F-005 | case_runner.py 从 devices.yaml 读取 DUT host 和 TG api_server，消除 DUT_URL/TG_URL/DEFAULT_API_URL 硬编码 | P0 | Given case_runner.py When 读取 devices.yaml Then DUT/TG 地址从 firewall.host 和 tg.api_server 获取，代码中无硬编码 IP | UC-EX-02, #10 |
| R-F-006 | case_runner.py 默认 dry-run 模式，不实际执行写操作，只校验命令构建和参数映射 | P0 | Given case_runner.py 不加 --execute When 运行 Then 每步调用 build_command 不调 execute_op，输出命令和校验状态 | UC-EX-04, SGA-01=A |
| R-F-007 | case_runner.py 支持 --execute --authorized 授权门，实际执行 op 并校验业务结果 | P0 | Given --execute --authorized When 运行 Then 实际连接设备执行，输出 PASS/FAIL/KNOWN_FAIL/ERROR 四态 | UC-EX-05, SGA-01=A |
| R-F-008 | case_runner.py 解析 case_steps 中的 warming_up 字段，识别为 true 时执行主 op 后强制执行 post_op | P0 | Given step 含 warming_up:true When 执行 Then 主 op 后自动执行 post_op(tg_stop_traffic_stream)，即使 md 未写 post_op 也自动补充 | UC-EX-07, SGA-04=C |
| R-F-009 | case_runner.py 解析 retry 字段，按 interval/max_attempts/success_condition 轮询执行 | P1 | Given step 含 retry:{interval:3,max_attempts:5,success_condition:"rx_packets==0"} When 执行 Then 每 3s 轮询最多 5 次，任意一次满足条件即通过 | UC-EX-05 |
| R-F-010 | case_runner.py 解析 known_issue 字段，标记的 step 失败时输出 KNOWN_FAIL，未标记输出 FAIL | P0 | Given step 含 known_issue 标记 When step 失败 Then 结果为 KNOWN_FAIL；Given step 无标记 When 失败 Then 结果为 FAIL | UC-EX-06, SGA-03=A |
| R-F-011 | case_runner.py 输出四态结果：PASS / FAIL / KNOWN_FAIL / ERROR | P0 | Given 用例执行结束 When 输出 result.json Then overall 字段为四态之一 | UC-EX-06, #12 |
| R-F-012 | case_runner.py 失败自动诊断：提取 error_type / error_code / reason / details / command | P1 | Given step 失败 When 诊断 Then report.md 含 error_type/error_code/reason/details/command | UC-EX-06, #7 |
| R-F-013 | case_runner.py 输出结构化 report.md，含四态统计 + 失败诊断 + 幂等容错记录 | P1 | Given 用例执行结束 When 输出 Then report.md 含四态统计表 + 失败 step 诊断 + 幂等容错记录 | UC-EX-06, #11 |
| R-F-014 | case_runner.py 实现 extract_payload(op_id, envelope) 统一解析函数，替代手动 extract_hitcount | P1 | Given 不同 op_id 的 envelope When extract_payload(op_id, envelope) Then 按 op_id 提取对应字段（policy_route_id / hitcount / tx_packets / rx_packets 等） | UC-EX-06, #6 |
| R-F-015 | case_runner.py 每用例结束后逆序清理 mutation ops（fw_delete_policy_route / fw_delete_object / tg_stop_traffic_stream） | P0 | Given 用例含 mutation ops When 用例结束 Then 逆序执行清理 op，清理结果记入 result.json | UC-EX-02 |
| R-F-016 | op_mapper.py 新增 fw_logout op 映射（OP_ID_TO_SUBCOMMAND: fw_logout -> auth, logout） | P0 | Given op_mapper.py When 调用 fw_logout Then 映射为 ptm-atomic run auth logout 命令 | UC-EX-09, #4 |
| R-F-017 | case_runner.py 全部用例结束后调用 fw_logout 登出共享 session，清理 session 文件 | P0 | Given case_runner 执行结束 When cleanup Then 调用 fw_logout 登出，session 文件清理，结果记入 result.json | UC-EX-09, #4 |
| R-F-018 | op_mapper.py _build_exec_env() 对 tg_* op 注入 TREX_API_URL 环境变量（从 devices.yaml tg.api_server 读取） | P0 | Given tg_* op 执行 When _build_exec_env Then env 含 TREX_API_URL=<tg.api_server>，不改 ptm-atomic 本体 | UC-EX-02, #1 |
| R-F-019 | install.py ptm-te-workflow 规则块新增 ≥4 条规则：TG路由/max_loss/ARP预热/session生命周期 | P0 | Given install.py When 安装 ptm-te Then CLAUDE.md 含 ≥4 条新规则，安装后验证通过 | UC-EX-08, #2 |
| R-F-020 | case_runner.py verify_loss 消费侧提取 tx/rx/loss_ratio，从 envelope.data 按 op_id 提取 | P2 | Given tg_verify_traffic_loss envelope When extract_payload Then 提取 tx_packets/rx_packets/loss_ratio | UC-EX-06, #9 |
| R-F-021 | 24 用例 md 全量整改：目录迁移到三级结构 + 重命名(<编号>-<名称>.md) + frontmatter 16 列补全 + tags/关键词列标注 + ARP 预热校验/补充 | P2 | Given 24 用例 md When 整改完成 Then 24/24 用例迁移到 cases/三级/四级/五级/ 结构，文件名符合 <编号>-<名称>.md 命名，frontmatter 16 列完整，tags 列和关键词列已标注，warming_up step 均有 post_op 或引擎自动补充 | UC-EX-07, UC-EX-10, #3 |
| R-F-022 | 用例目录结构约定：cases/三级(功能模块)/四级(子模块)/五级(用例组)/<用例编号>-<用例名称>.md。三级=功能模块(如 IPv4策略路由)，四级=子模块(如 配置管理)，五级=用例组(如 策略配置) | P1 | Given 用例文件 When 放置 Then 路径符合 cases/三级/四级/五级/<编号>-<名称>.md 结构。现有 cases/upload/ 24 用例需迁移到新结构 | UC-EX-10 |
| R-F-023 | 用例命名约定：<用例编号>-<用例名称>.md。用例编号保留组网前缀(如 PC-M1-01-01)，用例名称来自 frontmatter 用例名称列。文件名中的编号与名称之间用单个连字符分隔；用例名称内部的连字符保留 | P1 | Given 用例文件 When 命名 Then 文件名格式为 <编号>-<名称>.md，编号保留组网前缀，名称来自 frontmatter。命名冲突（连字符歧义）时以 frontmatter 用例编号列为唯一标识 | UC-EX-10 |
| R-F-024 | 用例 md frontmatter 16 列：三级目录/四级目录/五级目录/用例名称/用例编号/用例级别/组网描述/组网约束/预置条件/测试步骤/预期结果/首次创建版本/最后变更版本/关键词/测试类型/是否自动化。保留现有 case_steps YAML 块 | P1 | Given 用例 md When 解析 frontmatter Then 16 列字段全部存在（必填/可选见 DQ-05），case_steps YAML 块保留不变 | UC-EX-10 |
| R-F-025 | 用例 md frontmatter 含 tags 列（结构化列表，如 [策略路由, IPv4, PC, 正向, P0]）和关键词列（逗号分隔，如 fw_config_policy_route, fw_verify_policy_route, S01）。--tag 按 tags 列精确过滤，--keyword 按关键词列模糊匹配 | P1 | Given frontmatter 含 tags 和关键词列 When --tag 策略路由 Then 精确匹配 tags 含"策略路由"的用例；When --keyword fw_config Then 模糊匹配关键词含"fw_config"的用例 | UC-EX-02, UC-EX-03 |
| R-F-026 | case_runner.py --tag 按 frontmatter tags 列精确过滤（支持多 tag AND/OR 逻辑）；--keyword 按关键词列模糊匹配（子串匹配）。两者可组合使用 | P1 | Given --tag 策略路由,P0 Then 返回 tags 同时含"策略路由"和"P0"的用例；Given --keyword fw_config Then 返回关键词列含"fw_config"子串的用例；Given --tag 策略路由 --keyword S01 Then 返回同时满足两个条件的用例 | UC-EX-02 |
| R-F-027 | 用例与环境解耦：用例 case_steps 的 args 中环境相关参数（端口/IP/next_hop/URL）使用 ${ENV.*} 占位符引用环境文件字段（如 ${ENV.dut.port1} / ${ENV.tg.port1} / ${ENV.tg.url}），case_runner resolve_env_refs 在 build_command 前自动解析为物理值。环境相关参数禁止字面值；测试意图参数（如 max_loss / object_name / template name）保持字面值。未含 ${ENV.*} 的字面值 args 原样透传（向后兼容，现有用例不改造也能跑） | P1 | Given 用例 args 含 ${ENV.dut.port1} When resolve_env_refs 按环境文件解析 Then 替换为环境文件 dut.port1 物理值；Given args 含字面值 "GE0_3" When 无 --env-file Then 原样透传不报错；Given ${ENV.dut.port1} 但环境文件无 dut.port1 字段 When 解析 Then 返回 VALIDATION_FAILED envelope | UC-EX-11, CP3 评审范围扩展（环境文件驱动） |
| R-F-028 | DUT 接口自动预配置：case_runner 在 fw_login 后、用例主体前，按环境文件 nodes.dut1.interfaces 自动执行 fw_update_interface 配置接口 IP；用例结束后自动逆序清理预配置接口。用例 case_steps 不写 fw_update_interface 预配置步骤（由框架自动完成） | P1 | Given 环境文件 nodes.dut1.interfaces 含 3 个接口 When case_runner 启动 Then login 后自动执行 3 次 fw_update_interface 配 IP，用例结束后逆序清理；Given 用例 case_steps 不含 fw_update_interface 预配置 Then 接口已由框架预配置完成 | UC-EX-11, CP3 评审范围扩展（环境文件驱动） |
| R-F-029 | 设备 URL/端口自动解析：TREX_API_URL 从环境文件 nodes.tg1.trex_api_url（${ENV.tg.url}）解析注入 _build_exec_env；devices.yaml tg.api_server 作 fallback（环境文件无该字段时从 devices.yaml 取）。DUT/TG URL 从环境文件解析，case_runner 执行脚本无硬编码 IP。TG 操作仍经 ptm-atomic 原子操作（run tg trex <action>），框架不直接调 TG REST API | P1 | Given 环境文件 nodes.tg1.trex_api_url=http://10.113.55.170:8000 When tg_* op 执行 Then _build_exec_env 注入 TREX_API_URL=http://10.113.55.170:8000；Given 环境文件无 trex_api_url When tg_* op 执行 Then 从 devices.yaml tg.api_server fallback；Given case_runner 代码 When 审查 Then 无硬编码 DUT/TG IP | UC-EX-11, CP3 评审范围扩展（环境文件驱动） |

## 约束需求

| ID | 需求描述 | 优先级 | 验收条件 | 来源 |
|----|---------|--------|---------|------|
| R-C-001 | 不改 ptm-atomic CLI 本体，只改 op_mapper / trex-api / case_runner 消费侧 | P0 | 代码 diff 不触及 ptm-atomic 包源码 | CR-033 非目标 |
| R-C-002 | 不引入 pytest / robot / 外部 eval 框架 | P0 | case_runner.py 无 pytest/robot import | CR-033 非目标 |
| R-C-003 | 不为 devices.yaml 引入 pydantic | P0 | device-management 无 pydantic 依赖 | CR-033 非目标 |
| R-C-004 | 不做 HTML 报告（result.json + report.md 足够） | P1 | case_runner 不输出 HTML 文件 | CR-033 非目标 |
| R-C-005 | 不改 traffic-skill / ngfw-install skill | P0 | 代码 diff 不触及 traffic-skill / ngfw-install | CR-033 非目标 |
| R-C-006 | 不采集 TG 系统快照（collect_sysinfo 不覆盖 TG） | P0 | collect_sysinfo.py 不新增 TG 快照逻辑 | UC-EX-01 排除 |
| R-C-007 | skill 源在 ptm-team canonical，install.py 安装回 ptm-te workspace | P0 | skills/case-execution/ 在 ptm-team；install.py 回填到 .claude/skills/ | SGA-02=A |
| R-C-008 | 24 用例 md 留 ptm-te workspace，不迁入 ptm-team | P0 | 24 用例 md 不出现在 ptm-team 仓库 | SGA-02=A |
| R-C-009 | TG 操作仍经 ptm-atomic 原子操作（run tg trex <action>），框架不直接调 TG REST API；环境文件驱动仅做参数解析与 TREX_API_URL 注入 | P0 | case_runner.py 无直接 HTTP 调用 TG REST API 的代码 | CP3 评审范围扩展（环境文件驱动） |
| R-C-010 | 向后兼容：未含 ${ENV.*} 的字面值 args 原样透传，现有用例不改造也能跑 | P0 | 无 ${ENV.*} 的 args 经 resolve_env_refs 后原样返回 | CP3 评审范围扩展（环境文件驱动） |
| R-C-011 | 门控不变：dry-run 默认门 + --execute 授权门不变；环境文件驱动不影响安全门控 | P0 | dry-run 模式不触发设备连接；--execute 仍需显式授权 | CP3 评审范围扩展（环境文件驱动） |

## 非功能需求

| ID | 需求描述 | 优先级 | 验收条件 | 来源 |
|----|---------|--------|---------|------|
| R-NF-001 | dry-run 是默认安全门，--execute 需显式授权 | P0 | case_runner 默认 dry_run=True，--execute 才设 dry_run=False | SGA-01=A, UC-EX-04 |
| R-NF-002 | --execute 模式下 runtime_authorization 审计字段可追溯 | P0 | result.json 含 runtime_authorization(who/scope/authorized_at/reason) | UC-EX-05 |
| R-NF-003 | install.py 安装后规则与 skill 不丢失（重装一致性） | P0 | 重装后 CLAUDE.md 含规则块 + skills/ 目录存在 + op_mapper validate 通过 | UC-EX-08 |
| R-NF-004 | STATE_INVALID 自动重连（最多 1 次） | P1 | 遇 STATE_INVALID 时自动重新 login 后重试原命令 | UC-EX-05 |
| R-NF-005 | ConnectTimeout 自动重试（TG 最多 3 次，DUT 最多 1 次） | P1 | TG op ConnectTimeout 按 15/20/25s 递增重试；DUT op 失败等待 30s 重试 1 次 | UC-EX-05 |

## 风险与假设

| ID | 类型 | 内容 | 关联需求 | 缓解措施 |
|----|------|------|---------|---------|
| RA-001 | RISK | 跨仓库回填后 ptm-te workspace 的 skill 与 ptm-team canonical 源不一致 | R-C-007, R-NF-003 | install.py 安装后验证规则块和 skill 存在；op_mapper validate 通过 |
| RA-002 | RISK | devices.yaml 格式变更导致 case_runner 取址失败 | R-F-005 | case_runner 启动时校验 devices.yaml 含 firewall.host 和 tg.api_server |
| RA-003 | RISK | 24 用例 md 中 known_issue 字段标注不完整，导致 FAIL 误判为 KNOWN_FAIL 或反之 | R-F-010 | ARP 预热批量整改时同步检查 known_issue 标注；校验脚本检查 |
| RA-004 | RISK | fw_logout op 在 ptm-atomic 安装版未暴露（类似 fw_delete_object 问题） | R-F-016 | 安装前 ptm-atomic show fw_logout 验证；未暴露时降级为清理 session 文件 |
| RA-005 | RISK | _build_exec_env 注入 TREX_API_URL 后 tg op 仍走旧地址 | R-F-018 | 集成测试验证 tg op 实际连接 devices.yaml 配置的 api_server |
| RA-006 | ASSUMPTION | ptm-te workspace 已安装 ptm-atomic CLI 并在 PATH 中 | R-F-004 | case_runner 启动时 which ptm-atomic 检查 |
| RA-007 | ASSUMPTION | 24 用例 md 的 case_steps YAML 格式一致（parse_steps 兼容） | R-F-004 | dry-run 模式批量校验 24 用例解析无异常 |
| RA-008 | RISK | 重装后 ARP 预热规则被绕过（只靠规则不靠引擎） | R-F-008, R-NF-003 | SGA-04=C 双重保障：规则 + 引擎强制；引擎在 case_runner 内，重装不丢失 |
| RA-009 | RISK | 迁移后 exec_v4.py 仍被误用 | R-C-007 | 迁移完成后在 exec_v4.py 顶部加废弃标记；README 指向 case_runner.py |
| RA-010 | RISK | 24 用例目录迁移后路径变更导致 case_runner --cases-dir 找不到用例 | R-F-021, R-F-022 | 迁移后 dry-run 校验新路径下 24 用例全部解析成功；旧 cases/upload/ 保留废弃标记 |
| RA-011 | RISK | 用例名称中含连字符与文件名分隔符冲突（如"创建策略路由-有效参数-策略ID不存在-创建成功"中多个连字符） | R-F-023 | 以 frontmatter 用例编号列为唯一标识；文件名解析时按编号前缀(如 PC-M1-01-01)匹配，剩余部分为名称 |
| RA-012 | RISK | frontmatter 16 列补全工作量大，部分列信息缺失（如首次创建版本/最后变更版本） | R-F-024 | 缺失列填 N/A 或 TBD；必填/可选见 DQ-05 决策 |
| RA-013 | RISK | 环境文件格式与用例 ${ENV.*} 占位符路径不匹配，导致 resolve_env_refs 解析失败 | R-F-027 | dry-run 模式预校验全部 ${ENV.*} 占位符能否在环境文件中找到对应字段；解析失败返回 VALIDATION_FAILED envelope |
| RA-014 | RISK | DUT 接口自动预配置（fw_update_interface）在多用例间状态泄漏（前用例未清理干净影响后用例） | R-F-028 | 每用例结束后逆序清理预配置接口；case_runner 启动时检测接口是否已被预配置（幂等容错） |
| RA-015 | RISK | 环境文件 nodes.tg1.trex_api_url 与 devices.yaml tg.api_server 冲突时取值不一致 | R-F-029 | 环境文件优先（nodes.tg1.trex_api_url），devices.yaml 作 fallback；冲突时记录 warning 到 exec-log |
| RA-016 | ASSUMPTION | 环境文件格式为 YAML（如 topology-link3.yaml），含 nodes.dut1.interfaces / nodes.tg1.trex_api_url / nodes.dut1.url 等字段 | R-F-027,028,029 | case_runner 启动时校验环境文件 YAML 语法和必需字段 |

## 里程碑建议

| 里程碑 | 包含需求 | 交付物 | 前置里程碑 |
|--------|---------|--------|-----------|
| M1：TG 建模 + 规则固化（P0） | R-F-001, R-F-002, R-F-003, R-F-018, R-F-019 | devices.yaml tg 块 + SKILL.md + device-reference.md + install.py 规则块 + op_mapper TREX_API_URL | - |
| M2：case_runner 核心 + dry-run（P0） | R-F-004, R-F-005, R-F-006, R-F-007, R-F-015, R-F-016, R-F-017 | case_runner.py（三入口含 --tag/--keyword + dry-run + --execute + 逆序清理 + fw_logout） | M1 |
| M3：引擎增强 + 四态分级 + 用例结构化 + 环境文件驱动（P1） | R-F-008, R-F-009, R-F-010, R-F-011, R-F-012, R-F-013, R-F-014, R-F-022, R-F-023, R-F-024, R-F-025, R-F-026, R-F-027, R-F-028, R-F-029 | case_runner.py（warming_up/post_op/retry/known_issue/四态/诊断/报告/extract_payload/resolve_env_refs/DUT接口预配置）+ 用例目录结构/命名/frontmatter 16 列/tags 列约定 + 环境文件驱动 | M2 |
| M4：用例整改 + 消费侧（P2-P3） | R-F-020, R-F-021 | 24 用例全量整改（目录迁移+重命名+frontmatter+tags+ARP 预热）+ verify_loss 消费侧提取 | M3 |

## 默认假设

| ID | 假设内容 | 关联需求 |
|----|---------|---------|
| DA-001 | ptm-atomic CLI 已安装且在 PATH 中 | R-F-004 |
| DA-002 | 24 用例 md 的 case_steps 格式与 exec_v4.py parse_steps 兼容 | R-F-004 |
| DA-003 | devices.yaml 中 firewall.host 和 tg.api_server 字段存在 | R-F-005 |
| DA-004 | ptm-atomic 安装版暴露 fw_logout op（需安装前验证） | R-F-016 |
| DA-005 | TG host 上 trex-api 服务已启动且 api_server 可达 | R-F-018 |

## 明确排除项（Out of Scope）

- 改 ptm-atomic CLI 本体
- 引入 pytest / robot / 外部 eval 框架
- 为 devices.yaml 引入 pydantic
- HTML 报告（进 BACKLOG）
- 改 traffic-skill / ngfw-install skill
- 采集 TG 系统快照
- 读取凭据 / secret / 账户
- 真实设备 --execute 写操作的自动授权
