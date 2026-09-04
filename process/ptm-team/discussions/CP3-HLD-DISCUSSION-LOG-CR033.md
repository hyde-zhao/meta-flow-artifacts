---
cr_id: "CR-033"
phase: "solution-design"
discussion_type: "CP3-architecture-gray-areas"
status: "resolved"
created_at: "2026-07-28T11:00:00+08:00"
resolved_at: "2026-07-28T12:30:00+08:00"
agent_role: "meta-se"
delegated_interaction: true
---

# CR-033 CP3 HLD 架构灰区讨论日志

## 上下文

CP2 v1.1 已 approved（26 功能需求 / 25 场景 / 16 Story 候选 / 6 DQ / 4 SGQ）。技术选型与范围决策已由用户在 CP2 确认，meta-se 不重新做技术选型决策，基于已确认范围补充 Architecture Gray Areas。

已确认决策（不可推翻）：
- 引擎形态：`skills/case-execution/`（SKILL.md + scripts/case_runner.py 兼 argparse CLI，op_mapper 模式）
- 不引入框架：借鉴 pytest/robot 理念，不引入 pytest/robot
- TG 建模：`type:TG` + `api_server`，纳入清单不纳入快照，不引入 pydantic
- 执行入口：单用例 / 按目录 / 按标签或关键字（--tag/--keyword）
- #1 落点：op_mapper._build_exec_env 对 tg_* op 注入 TREX_API_URL
- #4：op_mapper 新增 fw_logout op + case_runner cleanup 登出（DQ-02 降级方案）
- 6 DQ：DQ-01 static review / DQ-02 fw_logout 降级 / DQ-03 known_issue 同步 / DQ-04 24用例一次性迁移 / DQ-05 8必填+8可选 / DQ-06 编号正则匹配

## Architecture Gray Areas

从 handoff 候选中选 4 个影响架构形态的灰区。AGA-05（fw_logout 实现方式）已在 DQ-02 决策，AGA-06（跨仓库 install.py 安装机制）已在 SGA-02 决策，不重复讨论。

### AGA-01: case-execution skill 与 op_mapper 职责边界（编排层 vs 映射层集成方式）

**问题**：case_runner.py 作为编排层，需要调用 op_mapper 的 execute_op。集成方式影响性能、可测试性和模块边界。

**候选选项**：

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| A: case_runner.py 直接 import op_mapper 模块（Python 函数调用） | 性能好（无新进程开销）；复用 envelope 解析、step-refs、回滚逻辑；单元测试容易 | case_runner 与 op_mapper 进程内耦合；op_mapper 异常会影响 case_runner | case_runner.py / op_mapper.py 模块边界；跨 skill import 路径 | 推荐 | 假设 case-execution skill 与 policy-route-execution skill 同仓库（ptm-team canonical）。若 case_runner 需要独立部署到无 op_mapper 的环境，切到 B |
| B: case_runner.py 通过 subprocess 调用 `python op_mapper.py execute`（exec_v4.py 方式） | 进程隔离；op_mapper 崩溃不影响 case_runner；与 exec_v4.py 一致 | 每次调用启动新 Python 进程，性能差；envelope 解析重复；step-refs 跨进程传递复杂 | case_runner.py 执行性能；step-refs 目录管理 | 不推荐 | 当 op_mapper 与 case_runner 跨仓库部署时切换。当前同仓库不需要 |

**推荐方案**：A（直接 import）。case_runner.py 从 `skills/policy-route-execution/scripts/op_mapper.py` import execute_op / build_command / handle_rollback 等函数，复用现有映射层和执行层逻辑。

**理由**：
- exec_v4.py 用 subprocess 方式是历史包袱（未入库脚本），case_runner 应避免重蹈
- op_mapper 已有 CLI 入口和函数入口，import 不影响 CLI 可用性
- step-refs 落盘和 resolve_step_refs 插值在进程内更高效

### AGA-02: devices.yaml tg 块与 traffic-skill 拓扑的数据归属（TG 地址/接口数据源）  [需用户确认]

**问题**：TG 设备的 api_server 和接口信息，数据源在 devices.yaml 还是 traffic-skill topology yaml？这决定 case_runner 从哪里读取 TG 地址，以及 devices.yaml tg 块的字段范围。

**候选选项**：

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| A: devices.yaml 只存 TG 设备元数据（host/serial_url/sub_type/hardware_platform/ssh/api_server），接口拓扑留 traffic-skill topology yaml | 与 firewall 块对称（firewall 只存设备元数据）；职责清晰（devices.yaml=设备清单，topology yaml=组网拓扑）；traffic-skill topology yaml 已有 type:TG + interfaces | case_runner 需读两个文件（devices.yaml + topology yaml）；拓扑文件路径需参数化 | devices.yaml schema；case_runner 取址逻辑；op_mapper TREX_API_URL 注入源 | 推荐 | 假设 traffic-skill topology yaml 是接口拓扑的天然归属。若用例 md 的"拓扑端口映射"表已足够，case_runner 可只读 devices.yaml，切到 B |
| B: devices.yaml 同时存 TG 接口拓扑（interfaces 列表），case_runner 只读 devices.yaml | 单文件取址；case_runner 逻辑简单 | devices.yaml 膨胀（含接口拓扑数据）；与 traffic-skill topology yaml 数据冗余；拓扑变更需同步两处 | devices.yaml schema 复杂度；数据一致性维护成本 | 不推荐 | 当 traffic-skill topology yaml 不存在或 case_runner 需要独立运行时切换 |
| C: traffic-skill topology yaml 作为唯一 TG 数据源，devices.yaml 不存 TG | TG 数据单一来源；无冗余 | devices.yaml 不含 TG 设备清单，无法统一管理；device-management skill 无法查询 TG 设备；与 R-F-001 冲突 | device-management skill 职责；R-F-001 验收 | 不推荐（与 R-F-001 冲突） | 不适用 |

**推荐方案**：A（devices.yaml 只存 TG 元数据，接口拓扑留 topology yaml）。

**理由**：
- 与 firewall 块对称：firewall 块只存 host/device_type/web/telnet/ssh，不存接口拓扑
- traffic-skill topology yaml 已有 `type: TG` + `host` + `api_server` + `interfaces` 列表（见 `skills/traffic-skill/configs/node2_tg_dut.yml`），是接口拓扑的天然归属
- case_runner 从 devices.yaml 读 `tg.api_server`（用于 op_mapper `_build_exec_env` 注入 TREX_API_URL），从 topology yaml 读接口映射（ports/txport/rxport）
- R-F-005 验收条件要求"DUT/TG 地址从 firewall.host 和 tg.api_server 获取"，与方案 A 一致

**用户确认**：A（2026-07-28T12:30:00+08:00，通过 host-orchestrator relay 确认）

**复述确认**：devices.yaml 只存 TG 元数据（host/serial_url/sub_type/hardware_platform/ssh/api_server），接口拓扑留 traffic-skill topology yaml。case_runner 从 devices.yaml 读 tg.api_server（注入 TREX_API_URL），从 topology yaml 读接口映射（需 --topology-yaml 参数）。

**影响面**：
- devices.yaml tg 块 schema：`type:TG + host + serial_url + sub_type + hardware_platform + ssh + api_server`（不含 interfaces）
- case_runner 取址：`devices.yaml` 读 tg.api_server + firewall.host；`--topology-yaml` 参数指定 topology yaml 路径，读接口映射
- op_mapper `_build_exec_env`：从 devices.yaml 读 tg.api_server 注入 TREX_API_URL

### AGA-03: frontmatter 16 列与 case_steps YAML 字段映射（冗余字段处理）

**问题**：frontmatter 16 列中有"测试步骤"和"预期结果"两列，case_steps YAML 块也有 step_name/expected_result。冗余字段如何处理？

**候选选项**：

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| A: frontmatter 的"测试步骤"和"预期结果"列留空或填概述，详细步骤在 case_steps YAML | 人类可读摘要 + 机器执行真相源分离；16 列完整 | 两处信息可能不一致；维护成本 | 用例 md 解析逻辑；frontmatter 校验 | 不推荐 | 当 16 列必须全部有值时切到 C |
| B: frontmatter 不含"测试步骤"和"预期结果"，只有 case_steps YAML | 无冗余；单一真相源 | 16 列变 14 列，与 DQ-05 决策冲突 | DQ-05 决策；frontmatter schema | 不推荐（与 DQ-05 冲突） | 不适用 |
| C: frontmatter 的"测试步骤"和"预期结果"作为 case_steps 的摘要，case_runner 解析时忽略这两列，只读 case_steps YAML | 16 列完整；case_runner 逻辑简单；摘要与详细分离 | 摘要与详细可能不一致；需校验脚本检查 | case_runner 解析逻辑；用例校验脚本 | 推荐 | 假设 case_steps YAML 是执行真相源。若用户要求 frontmatter 摘要与 case_steps 强一致，需加校验脚本 |

**推荐方案**：C（frontmatter 摘要 + case_runner 忽略冗余列）。

**理由**：
- DQ-05 确认 16 列（8 必填 + 8 可选），不能删列
- case_steps YAML 是执行真相源，frontmatter 的"测试步骤"/"预期结果"仅作人类可读摘要
- case_runner 解析时只读 case_steps YAML，忽略 frontmatter 这两列
- 校验脚本（ST-EX-13 整改时）检查 frontmatter 16 列存在性，不校验内容一致性

### AGA-04: envelope 统一解析 extract_payload 的抽象位置

**问题**：extract_payload(op_id, envelope) 统一解析函数放在 case_runner.py 还是 op_mapper.py？

**候选选项**：

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| A: 放在 case_runner.py（消费侧） | 职责清晰（op_mapper=映射+执行，case_runner=消费+断言）；与 verify_loss/hitcount 提取同模块；不膨胀 op_mapper | case_runner 需理解 envelope 结构；extract_payload 与 op_mapper 的 _parse_atomic_output 有部分重叠 | case_runner.py 代码组织 | 推荐 | 假设 extract_payload 是消费侧逻辑。若多个消费者都需要 extract_payload，切到 C |
| B: 放在 op_mapper.py（映射层） | op_mapper 统一管理 envelope 解析；其他消费者可复用 | op_mapper 职责膨胀（从映射+执行到消费）；违反单一职责 | op_mapper.py 职责边界 | 不推荐 | 当 op_mapper 需要对外提供 envelope 解释 API 时切换 |
| C: 放在新模块 envelope_utils.py | 独立模块可复用；不污染 op_mapper 和 case_runner | 新增模块增加复杂度；当前只有 case_runner 一个消费者 | 模块数量；import 路径 | 不推荐（过度设计） | 当有 ≥2 个消费者时切换 |

**推荐方案**：A（放在 case_runner.py）。

**理由**：
- extract_payload 是消费侧逻辑（case_runner 从 envelope 提取字段做断言），不是映射层逻辑
- op_mapper 的职责是"op_id -> CLI 命令 -> envelope"，不负责解释 envelope 含义
- 放在 case_runner 内，与 verify_loss / hitcount 提取逻辑同模块，便于维护
- 当前只有 case_runner 一个消费者，不需要新模块

## advisor lane 汇总

| Lane | 视角 | 意见 |
|---|---|---|
| lane-product | 场景/范围一致性 | AGA-02 推荐方案 A 与 R-F-001（devices.yaml tg 块）和 R-F-005（取址）一致；AGA-03 方案 C 与 DQ-05 16 列决策一致 |
| lane-architecture | 设计边界/依赖 | AGA-01 方案 A（import）使 case_runner 依赖 op_mapper 模块，需在 DEPENDENCY-MAP 显式声明；AGA-04 方案 A 保持 op_mapper 职责单一 |
| lane-quality | 可验证性/风险 | AGA-01 方案 A 便于单元测试；AGA-02 方案 A 需 case_runner 启动时校验两个文件存在；AGA-03 方案 C 需校验脚本检查 16 列存在性 |

## 用户确认交互

### AGAQ-01: TG 设备数据归属（devices.yaml tg 块 vs traffic-skill topology yaml）

**问题摘要**：TG 设备的 api_server 和接口拓扑数据源归属，影响 devices.yaml schema 和 case_runner 取址逻辑。

**候选选项**：
- A（推荐）：devices.yaml 只存 TG 元数据（host/serial_url/sub_type/hardware_platform/ssh/api_server），接口拓扑留 traffic-skill topology yaml
- B：devices.yaml 同时存 TG 接口拓扑，case_runner 只读 devices.yaml
- C：traffic-skill topology yaml 作为唯一 TG 数据源，devices.yaml 不存 TG（与 R-F-001 冲突）

**推荐方案**：A

**理由**：与 firewall 块对称；traffic-skill topology yaml 已有 type:TG + interfaces；R-F-005 验收要求从 tg.api_server 取址

**影响/风险**：case_runner 需读两个文件；拓扑文件路径需参数化

**回退/切换条件**：若 traffic-skill topology yaml 不存在或 case_runner 需完全独立运行，切到 B

**用户确认**：_待用户回复（通过 host-orchestrator relay）_

## Deferred Options

| ID | 内容 | 延后原因 | 重启条件 |
|---|---|---|---|
| DEF-AGA-01 | case_runner 独立部署模式（subprocess 调用 op_mapper） | 当前同仓库不需要 | case_runner 需部署到无 op_mapper 的环境 |
| DEF-AGA-02 | devices.yaml 含 TG 接口拓扑（方案 B） | 与 traffic-skill topology yaml 冗余 | traffic-skill topology yaml 废弃或 case_runner 需单文件取址 |
| DEF-AGA-03 | frontmatter 摘要与 case_steps 强一致校验 | 当前只需 16 列存在性校验 | 用户要求摘要与详细步骤一致 |
| DEF-AGA-04 | envelope_utils.py 独立模块 | 当前只有 case_runner 一个消费者 | 出现 ≥2 个 envelope 消费者 |

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-28 | meta-se | CR-033 CP3 架构灰区初稿：4 个灰区（AGA-01..04）+ advisor table + 1 条用户确认（AGAQ-01） |
| 1.1 | 2026-07-28 | meta-se | AGAQ-01 用户确认 A（devices.yaml 元数据 + topology yaml 接口 + --topology-yaml 参数）；status 改为 resolved |
