---
cr_id: "CR-033"
artifact_type: "architecture-decision"
version: "1.0"
created_at: "2026-07-28T11:40:00+08:00"
author: "meta-se"
status: "draft"
complexity: "high"
confirmed: false
confirmed_by: null
confirmed_at: null
source_hld: "docs/design/HLD-PTM-TE-EXEC.md"
---

# CR-033 ptm-te 执行引擎架构决策记录（ADR）

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-28 | meta-se | CR-033 ADR 初稿：8 条核心 ADR + Agent/Skill 组合 + 平台适配 + 设计确认点 |
| 1.1 | 2026-07-28 | meta-se | CP3 评审反馈 P4 落实：ADR-05 TREX_API_URL 注入收敛方案 a（扩展 _build_exec_env 签名），方案 b（环境变量直读）/方案 c（ptm-atomic 本体）标注不采用及理由 |
| 1.2 | 2026-07-28 | meta-se | CP3 二轮评审 R4 落实：ST-EX-14/EP-EX-08 归属从 FE-EX-03 移至 FE-EX-02（HLD §3/§17 Epic 与 Story 列表同步） |
| 1.3 | 2026-07-28 | meta-se | CP3 评审范围扩展：环境文件驱动 resolve_env_refs。新增 ADR-09（环境文件驱动 vs 硬编码/一次性适配脚本）；ADR-05 微调（TREX_API_URL 环境文件优先 ${ENV.tg.url}，devices.yaml fallback，ptm-atomic 约束）；ADR-08 补充（topology yaml 用途扩展为环境参数解析契约） |

## Agent/Skill 组合方案

| 组件 | 类型 | 路径 | CR-033 变更 |
|---|---|---|---|
| case-execution | Skill（新建） | skills/case-execution/SKILL.md + scripts/case_runner.py | 新建 |
| policy-route-execution | Skill（既有） | skills/policy-route-execution/scripts/op_mapper.py | 新增 fw_logout + TREX_API_URL 注入 |
| device-management | Skill（既有） | skills/device-management/SKILL.md + reference/ + templates/ | 新增 TG 流程 + 6 组合 + tg 块模板 |
| trex-traffic | Skill（既有） | skills/trex-traffic/SKILL.md | 不改（R-C-005） |
| install.py | 安装器 | script/ptm_team/install.py | 新增规则 + case-execution 安装 |
| ptm-te | Agent（既有） | agents/ptm-te.md | 规则块更新后自动生效 |

**组合方式**：case-execution skill 的 case_runner.py 直接 import policy-route-execution skill 的 op_mapper.py 模块（ADR-01），形成编排层 -> 映射+执行层 的进程内调用链。

## 平台适配差异

| 平台 | 适配点 | 说明 |
|---|---|---|
| ptm-team canonical | skill 源 | skills/case-execution/ 源在 ptm-team |
| ptm-te workspace | skill 安装目标 | install.py 安装到 .claude/skills/case-execution/ |
| ptm-atomic CLI | 不改本体（R-C-001） | op_mapper -> subprocess -> ptm-atomic |
| DUT（防火墙） | HTTPS Web API | op_mapper -> ptm-atomic -> DUT :443 |
| TG（trex） | HTTP trex-api | op_mapper -> ptm-atomic -> trex-api :8000 |

## 核心 ADR

### ADR-01: case-execution 与 op_mapper 集成方式

| 字段 | 值 |
|---|---|
| 决策点 | case_runner.py 如何调用 op_mapper |
| 推荐方案 | A：case_runner.py 直接 import op_mapper 模块（Python 函数调用） |
| 备选方案 | B：case_runner.py 通过 subprocess 调用 `python op_mapper.py execute`（exec_v4.py 方式） |
| 推荐优点 | 性能好（无新进程开销）；复用 envelope 解析、step-refs、回滚逻辑；单元测试容易 |
| 备选优点 | 进程隔离；op_mapper 崩溃不影响 case_runner；与 exec_v4.py 一致 |
| 推荐缺点 | case_runner 与 op_mapper 进程内耦合；跨 skill import 路径需显式声明 |
| 备选缺点 | 每次调用启动新 Python 进程，性能差；envelope 解析重复；step-refs 跨进程传递复杂 |
| 影响面 | case_runner.py / op_mapper.py 模块边界；跨 skill import 路径 |
| 风险 | op_mapper 异常会影响 case_runner（通过异常处理缓解） |
| 回退/切换条件 | case_runner 需独立部署到无 op_mapper 的环境时，切到方案 B |
| 回写到 HLD | §9 系统架构图 case_runner -> op_mapper import；§10 模块表 |

### ADR-02: dry-run 默认门与授权粒度

| 字段 | 值 |
|---|---|
| 决策点 | case_runner 默认执行模式与授权粒度 |
| 推荐方案 | dry-run 默认门 + --execute --authorized 授权门（SGA-01=A） |
| 备选方案 | B：三级授权（dry-run -> --validate -> --execute）；C：默认 --execute |
| 推荐优点 | 安全默认（不执行写操作）；授权边界清晰；与 NO_PRODUCTION_WRITE 一致 |
| 备选优点 | B 更细粒度；C 更直接 |
| 推荐缺点 | 二级授权可能不够细 |
| 备选缺点 | B 增加复杂度；C 与 NO_PRODUCTION_WRITE 冲突 |
| 影响面 | case_runner 安全边界；runtime_authorization 审计；CP7 验证方式 |
| 风险 | --execute 模式触发真实设备写操作（RISK-CR033-DEVICE-WRITE） |
| 回退/切换条件 | 用户要求更细粒度验证时，切到方案 B（DEF-EX-02） |
| 回写到 HLD | §12.1 [4] dry_run=True 默认；§13.4 安全 |

### ADR-03: exec_v4.py 迁移与废弃策略

| 字段 | 值 |
|---|---|
| 决策点 | exec_v4.py 迁移后的处理方式 |
| 推荐方案 | 迁移后加废弃标记，不删除（RA-009） |
| 备选方案 | 立即删除 |
| 推荐优点 | 可回退；保留历史参考 |
| 备选优点 | 避免误用 |
| 影响面 | ptm-te workspace exec_v4.py |
| 风险 | exec_v4.py 仍被误用（低概率，通过废弃标记+README 缓解） |
| 回退/切换条件 | 若废弃标记后仍被误用，删除文件 |
| 回写到 HLD | §14 风险 RA-009 |

### ADR-04: fw_logout op 实现与降级

| 字段 | 值 |
|---|---|
| 决策点 | fw_logout op 的实现方式和未暴露时的降级策略 |
| 推荐方案 | op_mapper 映射 fw_logout -> (auth, logout) + 安装前验证 + 降级 session 文件清理（DQ-02） |
| 备选方案 | 强制要求 ptm-atomic 升级暴露 fw_logout |
| 推荐优点 | 保证可用性（降级方案兜底）；不超出 CR-033 范围 |
| 备选优点 | op 映射完整 |
| 推荐缺点 | 降级方案不通过 op_mapper（直接 os.remove session 文件） |
| 备选缺点 | 超出 CR-033 范围；ptm-atomic 升级周期长 |
| 影响面 | op_mapper 映射表；case_runner cleanup 逻辑；EXPECTED_OP_COUNT 21->22 |
| 风险 | ptm-atomic 安装版未暴露 fw_logout（RA-004） |
| 回退/切换条件 | ptm-atomic 升级暴露 fw_logout 后，移除降级逻辑 |
| 回写到 HLD | §12.1 [5] fw_logout + 降级；Gotcha #1/#4 |

### ADR-05: TREX_API_URL 注入边界  [方案 a 定稿，环境文件优先]

| 字段 | 值 |
|---|---|
| 决策点 | TREX_API_URL 环境变量的注入位置与实现方式 |
| 推荐方案（已定稿） | op_mapper._build_exec_env 注入：扩展签名新增 `tg_api_server: str = ""` 参数。TREX_API_URL 来源优先级：环境文件 ${ENV.tg.url}（= nodes.tg1.trex_api_url）> devices.yaml tg.api_server fallback（方案 a，R-F-018/029，#1 落点） |
| 备选方案 b（不采用） | case_runner 启动时 export TREX_API_URL，_build_exec_env 从 os.environ 读取 |
| 备选方案 c（不采用） | ptm-atomic 本体读取 TREX_API_URL 环境变量 |
| 推荐优点 | 不改 ptm-atomic 本体（R-C-001）；op_mapper 内部完成注入，职责内聚；环境文件优先支持一次编写多环境执行，devices.yaml fallback 保证向后兼容 |
| 备选 b 缺点 | 需调用方手动 export，破坏 op_mapper 单一职责；多调用方场景易遗漏 export 导致 tg op 走旧地址（RA-005） |
| 备选 c 缺点 | 改 ptm-atomic 本体，违反 R-C-001 |
| ptm-atomic 约束 | TG 操作仍经 ptm-atomic run tg trex <action> 原子操作下发，框架禁止直接调 TG REST API；环境文件驱动仅做参数解析与 TREX_API_URL 注入（_build_exec_env 环境变量），由 ptm-atomic 子进程消费 |
| 影响面 | op_mapper._build_exec_env 签名（新增 tg_api_server 参数）；case_runner 调用 execute_op 时传 tg_api_server（环境文件优先，devices.yaml fallback）；签名扩展实现细节由 ST-EX-03 full-lld 承载 |
| 风险 | 注入后 tg op 仍走旧地址（RA-005，低概率，集成测试验证） |
| 回退/切换条件 | ptm-atomic 本体支持读取 TREX_API_URL 后，移除 op_mapper 注入 |
| 用户确认 | 方案 a（2026-07-28，CP3 评审 P4 决策；范围扩展环境文件优先 2026-07-28，CP3 评审范围扩展） |
| 回写到 HLD | §12.2 _build_exec_env 注入；Gotcha #3 方案 a 定稿；§12.3 环境解析层（TREX_API_URL 环境文件优先） |

### ADR-06: ARP 预热双重保障

| 字段 | 值 |
|---|---|
| 决策点 | ARP 预热（warming_up -> post_op）的强制方式 |
| 推荐方案 | 规则 + 引擎双重保障（SGA-04=C） |
| 备选方案 | A：只靠规则；B：只靠引擎 |
| 推荐优点 | 防重装回退（规则被绕过时引擎兜底）；引擎在 case_runner 内重装不丢失 |
| 备选优点 | A 简单；B 不需要 install.py 规则 |
| 推荐缺点 | 双重维护 |
| 备选缺点 | A 重装后规则可绕过（RA-008）；B 规则不可见 |
| 影响面 | install.py 规则块；case_runner warming_up 解析逻辑 |
| 风险 | 重装后 ARP 预热规则被绕过（RA-008，低概率，引擎兜底） |
| 回退/切换条件 | 若引擎强制导致用例执行异常，切到方案 A（只靠规则） |
| 回写到 HLD | §12.1 [4] warming_up -> 强制 post_op；§13.5 可维护性 |

### ADR-07: 用例命名与编号解析

| 字段 | 值 |
|---|---|
| 决策点 | 用例文件名 <编号>-<名称>.md 的解析方式 |
| 推荐方案 | frontmatter 用例编号列为唯一标识；文件名按编号前缀正则匹配（DQ-06） |
| 备选方案 | 用下划线替代连字符做名称内部分隔 |
| 推荐优点 | 不改变命名习惯；frontmatter 编号为唯一标识 |
| 备选优点 | 解析简单 |
| 推荐缺点 | 正则匹配需覆盖编号格式 |
| 备选缺点 | 改变命名习惯；增加迁移成本 |
| 影响面 | case_runner 用例发现逻辑；24 用例重命名 |
| 风险 | 编号格式变化导致正则不匹配（RA-011） |
| 回退/切换条件 | 解析失败时报错让用户手动指定 |
| 回写到 HLD | Gotcha #6 |

### ADR-08: TG 设备数据归属  [用户已确认 A]

| 字段 | 值 |
|---|---|
| 决策点 | TG 设备 api_server 和接口拓扑的数据源归属 |
| 推荐方案 | A：devices.yaml 只存 TG 元数据（host/serial_url/sub_type/hardware_platform/ssh/api_server），接口拓扑留 traffic-skill topology yaml |
| 备选方案 | B：devices.yaml 同时存 TG 接口拓扑；C：traffic-skill topology yaml 作为唯一 TG 数据源（与 R-F-001 冲突） |
| 推荐优点 | 与 firewall 块对称；职责清晰；traffic-skill topology yaml 已有 type:TG + interfaces |
| 备选优点 | B 单文件取址；C TG 数据单一来源 |
| 推荐缺点 | case_runner 需读两个文件 |
| 备选缺点 | B devices.yaml 膨胀+冗余；C 与 R-F-001 冲突 |
| 影响面 | devices.yaml tg 块 schema；case_runner 取址逻辑（需 --topology-yaml 参数）；op_mapper TREX_API_URL 注入源。topology yaml 用途扩展（CP3 评审范围扩展）：从 TG 接口映射扩展为环境参数解析契约（port_mapping/nodes/links），env_topology 由 case_runner 从 --env-file 加载，resolve_env_refs 据此解析 ${ENV.*} 占位符（ADR-09） |
| 风险 | topology yaml 路径需参数化；env-file 与 topology yaml 职责边界需清晰（env-file 描述物理映射 + 环境契约，topology yaml 描述组网拓扑） |
| 回退/切换条件 | 若 traffic-skill topology yaml 不存在或 case_runner 需独立运行，切到方案 B |
| 用户确认 | A（2026-07-28T12:30:00+08:00，AGAQ-01 resolved；范围扩展 2026-07-28 CP3 评审） |
| 回写到 HLD | §3 蓝图承接；§10 模块表；§12.1 [1] 启动校验；§12.1 命令行示例含 --topology-yaml/--env-file；§12.3 环境解析层 |

### ADR-09: 环境文件驱动（resolve_env_refs + ${ENV.*}）  [范围扩展新增]

| 字段 | 值 |
|---|---|
| 决策点 | 用例与环境解耦方式：用例如何引用环境相关参数（端口/IP/next_hop/URL） |
| 推荐方案 | 环境文件驱动：op_mapper 新增 resolve_env_refs(args, env_topology)，与 resolve_step_refs 并列，在 execute_op 的 build_command 前执行（顺序：resolve_env_refs -> resolve_step_refs -> validate_args -> build_command）；用例用 ${ENV.*} 占位符引用环境参数；case_runner 加载 --env-file（port_mapping/nodes/links）解析为 env_topology 传入 op_mapper（R-F-027/028/029） |
| 备选方案 a（不采用） | 硬编码：参数直接写在用例 case_steps 中（exec_v4.py 现状，换环境需改用例） |
| 备选方案 b（不采用） | 一次性适配脚本：每环境写一个 Python 适配脚本转换参数（重复劳动，不可复用） |
| 推荐优点 | 用例一次编写多环境执行（换环境只换 --env-file）；测试意图保持字面值，环境相关用 ${ENV.*}；向后兼容（无 ${ENV.*} 字面值原样透传） |
| 备选 a 缺点 | 换环境需改用例，硬编码风险（exec_v4.py 痛点） |
| 备选 b 缺点 | 重复劳动，每环境一个脚本，不可复用 |
| ptm-atomic 约束 | TG 操作仍经 ptm-atomic run tg trex <action> 原子操作下发，框架禁止直接调 TG REST API；resolve_env_refs 仅做参数解析，TREX_API_URL 经 _build_exec_env 注入由 ptm-atomic 子进程消费 |
| 占位符语法 | ${ENV.tg.port1}(物理端口名)/${ENV.tg.port1.ip}/${ENV.tg.port1.gw}/${ENV.dut.port1}/${ENV.dut.port1.ip}/${ENV.tg.url}/${ENV.dut.url}/${ENV.tg.ports[port1,port2]}(聚合数组)；port1/port2 是逻辑序号对齐 port_mapping，不随环境变化 |
| 参数分层 | 环境相关参数（端口/IP/next_hop/URL）用 ${ENV.*} 占位符，禁止字面值；测试意图参数（op_id/期望结果/known_issue）保持字面值 |
| 失败行为 | resolve_env_refs 解析失败（占位符无对应 env_topology 键）-> VALIDATION_FAILED envelope，step 标记 error |
| 影响面 | op_mapper.execute_op 签名（新增 env_topology 参数）+ resolve_env_refs；case_runner load_env_file + build_env_topology + preconfigure_dut_interfaces；ST-EX-03/04/17 承载；ST-EX-13 24 用例改写为 ${ENV.*} |
| 风险 | env-file schema 不完整导致解析失败（DA-006/007 校验缓解）；YAML 1.1 整数陷阱（端口名需加引号，Gotcha #11） |
| 回退/切换条件 | 无 ${ENV.*} 占位符的用例原样透传（向后兼容）；env-file 缺失时降级为 devices.yaml 直读（ADR-05 fallback） |
| 回写到 HLD | §12.3 环境解析层；§12 execute_op 流程；§12.1 设备准备 [2]；Gotcha #11/#12/#13；DA-006/007 |

## 设计确认点（需人工确认）

| DQ ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣/影响/风险/回退 |
|---|---|---|---|---|---|
| DQ-CP3-01 | architecture | ADR-08 TG 设备数据归属（AGAQ-01） | A：devices.yaml 元数据 + topology yaml 接口 | B：devices.yaml 含接口 | 已确认 A（2026-07-28T12:30:00+08:00）。case_runner 需 --topology-yaml 参数 |
| DQ-CP3-02 | implementation | ADR-04 fw_logout op 在 ptm-atomic 安装版是否暴露 | 安装前验证 + 降级 session 清理 | 强制 ptm-atomic 升级 | 推荐 DQ-02 降级方案；备选超出范围。回退：降级为 session 文件清理 |
| DQ-CP3-03 | scope | 24 用例 known_issue 标注完整性（DQ-03） | 纳入 R-F-021 同步检查 | 单独 CR | 推荐同步整改；备选增加协调成本。回退：只做 ARP 预热，known_issue 进 BACKLOG |

## 变更记录

| 变更 | 日期 | 内容 | 触发条件 |
|---|---|---|---|
| CP3 后落地映射补充 | _待 CP3 通过后_ | 补充 Story -> ADR 落地映射 | CP3 人工确认通过 |
