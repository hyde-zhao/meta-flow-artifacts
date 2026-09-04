---
cr_id: "CR-033"
artifact_type: "blueprint"
version: "1.2"
created_at: "2026-07-28T11:10:00+08:00"
author: "meta-se"
status: "draft"
source_requirements: "docs/product/REQUIREMENTS-PTM-TE-EXEC.md"
source_story_map: "docs/product/STORY-MAP-PTM-TE-EXEC.md"
---

# CR-033 ptm-te 执行引擎蓝图

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-28 | meta-se | CR-033 蓝图初稿：3 Feature / 8 Epic / 数据归属 / 依赖方向 |
| 1.1 | 2026-07-28 | meta-se | CP3 二轮评审 R4 落实：EP-EX-08（verify_loss 消费侧）从 FE-EX-03 移至 FE-EX-02（ST-EX-14 双重归属消除）；FE-EX-02 Epic 表加 EP-EX-08，FE-EX-03 Epic 表移除 EP-EX-08；1.0 行 Epic 数更正为 8 |
| 1.2 | 2026-07-28 | meta-se | CP3 评审范围扩展：环境文件驱动 resolve_env_refs。FE-EX-02 范围扩展（环境解析层属 case-execution 引擎能力）；新增 EP-EX-09（环境解析层 + DUT 接口预配置）-> ST-EX-17；FE-EX-02 来源加 R-F-027..029/UC-EX-11；集成契约加 resolve_env_refs + load_env_file + ptm-atomic 约束；RS 切片 RS-EX-03 加 ST-EX-17/EP-EX-09/SCN-EX-26..29；1.0 行 Epic 数更正为 9 |

## 1. Feature / Epic 能力边界

### Feature FE-EX-01: TG 设备建模

| 字段 | 值 |
|---|---|
| Feature ID | FE-EX-01 |
| 名称 | TG 设备建模 |
| 来源 | UC-EX-01, R-F-001/002/003/018 |
| 优先级 | P0 |
| 描述 | 为 devices.yaml 新增 TG 类型设备块，支持 type:TG + sub_type(ixia-c/trex) + hardware_platform(EP/C236/J1900) + api_server；device-management skill 补 TG 流程和 6 组合型号对照；op_mapper 对 tg_* op 注入 TREX_API_URL |

**Epic 拆解**：

| Epic ID | 名称 | 范围 | Story |
|---|---|---|---|
| EP-EX-01 | devices.yaml tg 块 + 6 组合 | devices.yaml tg 块 schema + device-reference.md 6 组合 + SKILL.md TG 流程 | ST-EX-01 |
| EP-EX-02 | op_mapper TREX_API_URL 注入 | op_mapper._build_exec_env 对 tg_* op 注入 TREX_API_URL | ST-EX-03 |

**能力边界**：
- 做：devices.yaml tg 块定义、6 组合型号对照、SKILL.md TG 流程、TG_SSH_PASSWORD 环境变量、op_mapper TREX_API_URL 注入
- 不做：collect_sysinfo 不采 TG 快照（R-C-006）；不引入 pydantic（R-C-003）；TG 接口拓扑留 traffic-skill topology yaml（AGA-02=A）

**数据归属**：
- devices.yaml tg 块：`type:TG + host + serial_url + sub_type + hardware_platform + ssh + api_server`（**不含 interfaces**）
- traffic-skill topology yaml：TG 接口列表（ports/txport/rxport 映射）
- case_runner 取址：devices.yaml 读 tg.api_server + firewall.host；topology yaml 读接口映射

### Feature FE-EX-02: case-execution 执行引擎

| 字段 | 值 |
|---|---|
| Feature ID | FE-EX-02 |
| 名称 | case-execution 执行引擎 |
| 来源 | UC-EX-02..09/11, R-F-004..017/020/022..029 |
| 优先级 | P0-P1 |
| 描述 | 新建 skills/case-execution/ skill，含 SKILL.md + scripts/case_runner.py（argparse CLI），借鉴 pytest/robot 理念不引入框架，复用 op_mapper execute_op，支持三入口/dry-run/--execute/逆序清理/fw_logout/四态分级/失败诊断/结构化报告/extract_payload/环境文件驱动（resolve_env_refs + ${ENV.*} + --env-file + DUT 接口预配置） |

**Epic 拆解**：

| Epic ID | 名称 | 范围 | Story |
|---|---|---|---|
| EP-EX-03 | case_runner 核心引擎 | 三入口 + devices.yaml 取址 + dry-run + --execute + 逆序清理 + fw_logout | ST-EX-04, ST-EX-05, ST-EX-06, ST-EX-07 |
| EP-EX-04 | 引擎增强 | warming_up/post_op + retry + known_issue 四态 + 失败诊断 + 结构化报告 + extract_payload | ST-EX-08, ST-EX-09, ST-EX-10, ST-EX-11, ST-EX-12 |
| EP-EX-05 | 用例结构化与标签执行 | 目录结构 + 命名 + frontmatter 16 列 + tags/keyword 执行 | ST-EX-15, ST-EX-16 |
| EP-EX-08 | verify_loss 消费侧 | case_runner verify_loss 提取 tx/rx/loss_ratio | ST-EX-14 |
| EP-EX-09 | 环境解析层 + DUT 接口预配置 | env_topology 契约（port_mapping/nodes/links）+ ${ENV.*} 解析集成 + DUT 接口自动预配置（fw_update_interface）+ 用例后清理 + TREX_API_URL 环境文件优先 | ST-EX-17 |

**能力边界**：
- 做：case_runner.py 三入口（--case-file/--cases-dir/--tag/--keyword）；dry-run 默认门；--execute --authorized 授权门；逆序清理；fw_logout 登出；warming_up/post_op 引擎强制；retry 轮询；known_issue 四态；失败诊断；结构化报告；extract_payload；verify_loss 消费侧；环境文件驱动（load_env_file + env_topology 契约 + ${ENV.*} 解析集成 + DUT 接口自动预配置）
- 不做：不引入 pytest/robot（R-C-002）；不做 HTML 报告（R-C-004）；不改 ptm-atomic 本体（R-C-001）

**集成契约**：
- case_runner.py 直接 import op_mapper 模块（AGA-01=A），调用 execute_op / build_command / handle_rollback
- case_runner.py 从 devices.yaml 读取 firewall.host 和 tg.api_server（R-F-005）
- case_runner.py 从 topology yaml 读取接口映射（ports/txport/rxport）
- op_mapper 新增 fw_logout op 映射（R-F-016）
- op_mapper._build_exec_env 对 tg_* op 注入 TREX_API_URL（R-F-018）
- op_mapper 新增 resolve_env_refs(args, env_topology)，在 execute_op 的 build_command 前执行，解析 ${ENV.*} 占位符（R-F-027/029）；顺序：resolve_env_refs -> resolve_step_refs -> validate_args -> build_command
- case_runner 加载 --env-file（port_mapping/nodes/links）解析为 env_topology 传入 op_mapper（R-F-027）；DUT 接口自动预配置（fw_update_interface 按 nodes.dut1.interfaces）+ 用例后清理（R-F-028）
- ptm-atomic 约束：TG 操作仍经 ptm-atomic run tg trex <action> 原子操作下发，框架禁止直接调 TG REST API；环境文件驱动仅做参数解析与 TREX_API_URL 注入（_build_exec_env 环境变量），由 ptm-atomic 子进程消费

### Feature FE-EX-03: 规则固化与执行改进

| 字段 | 值 |
|---|---|
| Feature ID | FE-EX-03 |
| 名称 | 规则固化与执行改进 |
| 来源 | UC-EX-07/08/10, R-F-008/019/021 |
| 优先级 | P0-P2 |
| 描述 | install.py ptm-te-workflow 规则块新增 ≥4 条规则（TG路由/max_loss/ARP预热/session生命周期）；24 用例 md 全量整改（目录迁移+重命名+frontmatter 16 列+tags+ARP 预热校验） |

**Epic 拆解**：

| Epic ID | 名称 | 范围 | Story |
|---|---|---|---|
| EP-EX-06 | install.py 规则固化 | ptm-te-workflow 规则块新增 ≥4 条规则 | ST-EX-02 |
| EP-EX-07 | 24 用例全量整改 | 目录迁移 + 重命名 + frontmatter 16 列 + tags + ARP 预热校验 | ST-EX-13 |

**能力边界**：
- 做：install.py 规则块新增 ≥4 条；24 用例 md 目录迁移到三级结构；重命名 <编号>-<名称>.md；frontmatter 16 列补全；tags/关键词列标注；ARP 预热校验
- 不做：不改 traffic-skill / ngfw-install skill（R-C-005）；不采集 TG 快照（R-C-006）

## 2. 数据归属矩阵

| 数据对象 | 归属 Feature | 存储位置 | 写入方 | 读取方 | 备注 |
|---|---|---|---|---|---|
| devices.yaml tg 块 | FE-EX-01 | workspace devices.yaml | 测试平台开发者（手动 + device-management skill） | case_runner / op_mapper | type:TG + host + serial_url + sub_type + hardware_platform + ssh + api_server |
| devices.yaml firewall 块 | 既有 | workspace devices.yaml | 既有 | case_runner | host 字段用于 DUT_URL |
| traffic-skill topology yaml | 既有 | workspace configs/*.yml | 既有 | case_runner | TG 接口映射（ports/txport/rxport） |
| case_steps YAML | FE-EX-02 | workspace cases/三级/四级/五级/*.md | 测试执行工程师 | case_runner | 执行真相源 |
| frontmatter 16 列 | FE-EX-02/03 | workspace cases/*.md | 测试执行工程师 | case_runner（忽略测试步骤/预期结果列） | AGA-03=C |
| op_mapper 映射表 | FE-EX-01/02 | skills/policy-route-execution/scripts/op_mapper.py | meta-dev | case_runner / CLI | 新增 fw_logout，EXPECTED_OP_COUNT 21->22 |
| install.py 规则块 | FE-EX-03 | script/ptm_team/install.py | meta-dev | install.py 安装时 | render_ptm_te_rule_body 新增 ≥4 条 |
| result.json | FE-EX-02 | workspace runs/<run-id>/result.json | case_runner | 测试执行工程师 / 测试经理 | 四态分级 + 清理记录 + runtime_authorization |
| report.md | FE-EX-02 | workspace runs/<run-id>/report.md | case_runner | 测试执行工程师 / 测试经理 | 四态统计 + 失败诊断 + 幂等容错 |

## 3. 跨 Feature 用户旅程

### 旅程 1: 新增 TG 设备并执行用例（FE-EX-01 -> FE-EX-02）

1. 测试平台开发者通过 device-management skill 在 devices.yaml 新增 tg 块（FE-EX-01）
2. 测试执行工程师编写 case_steps md 文件（FE-EX-02）
3. 测试执行工程师执行 `case_runner.py run --cases-dir cases/IPv4策略路由/ --devices-yaml devices.yaml`（FE-EX-02）
4. case_runner 从 devices.yaml 读取 firewall.host 和 tg.api_server（FE-EX-01/02 协作）
5. case_runner import op_mapper，调用 execute_op 执行步骤（FE-EX-02）
6. op_mapper 对 tg_* op 注入 TREX_API_URL（FE-EX-01 EP-EX-02）
7. case_runner 输出 result.json + report.md（FE-EX-02）

### 旅程 2: 重装后规则不丢失（FE-EX-03）

1. 测试平台开发者执行 `install.py install ptm-te --component full`（FE-EX-03）
2. install.py 渲染 ptm-te-workflow 规则块到 CLAUDE.md，含 ≥4 条新规则（FE-EX-03 EP-EX-06）
3. install.py 安装 case-execution skill 到 workspace .claude/skills/（FE-EX-03）
4. 重装后 op_mapper validate 通过，规则和 skill 不丢失（R-NF-003）

## 4. 发布切片与 Feature 映射

| 切片 | Feature | Epic | Story | 验证场景 |
|---|---|---|---|---|
| RS-EX-01（P0） | FE-EX-01 + FE-EX-03 | EP-EX-01/02/06 | ST-EX-01,02,03 | SCN-EX-01,18,19 |
| RS-EX-02（P0） | FE-EX-02 | EP-EX-03 | ST-EX-04,05,06,07 | SCN-EX-02,03,04,07,08,16 |
| RS-EX-03（P1） | FE-EX-02 | EP-EX-04/05/09 | ST-EX-08,09,10,11,12,15,16,17 | SCN-EX-05,09,10,17,21,22,23,24,26,27,28,29 |
| RS-EX-04（P2-P3） | FE-EX-02/03 | EP-EX-07/08 | ST-EX-13,14 | SCN-EX-20,25 |

## 5. N/A / WAIVED 项

| 项 | 原因 | 影响范围 | 后续触发条件 |
|---|---|---|---|
| 无 N/A 项 | 跨 Feature 边界、数据归属、依赖方向均需蓝图承载 | N/A | N/A |
| 无 WAIVED 项 | 全部 Feature 边界已定义 | N/A | N/A |
