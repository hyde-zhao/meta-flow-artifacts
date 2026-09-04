---
cr_id: "CR-033"
story_id: "STORY-EX-13"
title: "24 用例全量整改（目录迁移 + 重命名 + frontmatter 16 列 + tags + ARP 预热校验 + ${ENV.*} 引用改写）"
wave: 4
priority: "P2"
lld_policy: "full-lld"
feature_design_refs:
  - "docs/features/rule-fix-and-improvement/DESIGN.md"
depends_on: ["ST-EX-08", "ST-EX-15", "ST-EX-17"]
dependency_type: "hard"
status: "lld-ready"
design_evidence_type: "full-lld"
lld_policy_required_level: "full-lld"
source_hld: "docs/design/HLD-PTM-TE-EXEC.md"
source_adr: "docs/design/ARCHITECTURE-DECISION-PTM-TE-EXEC.md"
source_requirements: "docs/product/REQUIREMENTS-PTM-TE-EXEC.md"
created_at: "2026-07-28T16:30:00+08:00"
author: "meta-dev"
---

# ST-EX-13 24 用例全量整改（目录迁移 + 重命名 + frontmatter 16 列 + tags + ARP 预热校验 + ${ENV.*} 引用改写）

## 0. 概述

本 LLD 是 ST-EX-13 24 用例全量整改（目录迁移 + 重命名 + frontmatter 16 列 + tags + ARP 预热校验 + ${ENV.*} 引用改写）的 full-lld 设计证据，属于 CR-033 ptm-te 执行引擎的 Wave 4 用例整改 Story，承载 24 用例 md 文件从 v1 扁平入口迁移到新目录结构并完成环境参数化改写。

**需求溯源**：ST-EX-13 承载 R-F-004（case_steps md 驱动）、R-F-008（warming_up/post_op）、R-F-021（ARP 预热整改同步检查）、R-F-022..026（用例结构化）、R-F-027..029（环境文件驱动）等功能需求；详细需求映射见 §1 工程依据表。

**模块拆分**：24 用例 md 文件按职责拆分为目录迁移（cases/upload/ 或 exec_v4.py 硬编码基线 -> ptm-te/cases/IPv4策略路由/配置管理/策略配置/）/重命名（ADR-07 编号正则）/frontmatter 16 列补全（DQ-05）/tags 字段补全/${ENV.*} 引用改写（ADR-09）/ARP 预热校验（SM-EX-07）/known_issue 标注校验（DQ-CP3-03）/exec_v4.py 废弃标记（ADR-03）；不写引擎代码（case_runner.py 由 ST-EX-04 拥有）。

**代码结构**：详见 §2 文件影响范围。ptm-te/cases/IPv4策略路由/**/*.md 新建（迁移+整改）+ README.md 目录索引；ptm-te/exec_v4.py 废弃标记修改；用例 md 留 ptm-te workspace（R-C-008，不随 install.py 安装）。

**技术细节**：目录结构三级（§3.1，协议族/功能域/子域）；文件命名 ADR-07 编号正则 `^PC-[A-Z0-9]+-[A-Z0-9]+-\d{2}-\d{2}$`（§3.2）；frontmatter 16 列 8 必填 + 8 可选 N/A（§3.3，DQ-05）；${ENV.*} 改写（§3.5，ADR-09，环境参数占位符化，测试意图保持字面值）；ARP 预热校验（§4.1[6]，SM-EX-07 24/24）；exec_v4.py 废弃标记（§4.1[8]，ADR-03）等核心技术细节在各章节展开。

**Definition of Done (DoD)**：CP6 实现完成以 §6 测试矩阵全通过为准（24 用例 dry-run 全 PASS、0 残留字面端口/IP、24/24 warming_up 合规、编号正则全匹配）；CP7 验证采用 static-only + dry-run-only（DQ-01 推荐），runtime 端到端验证属 T-01 follow-up。DoD 详见 §6.2 测试矩阵。

## 1. 工程依据

| 依据类型 | 引用 | 关联点 |
|---|---|---|
| HLD | §12.3 环境解析层设计（用例迁移示例 PC-COMB-M4-01-01 改写前/改写后） | ${ENV.*} 占位符改写规则唯一参照 |
| HLD | §12.1 case_runner 执行流程 [3] 用例发现 / [5] 逐 step 执行 | 目录结构与 frontmatter tags/关键词消费 |
| HLD | §16.1 改进 #3（ARP 预热 24 用例校验）/ #5（用例文件驱动）/ #10（devices.yaml 取址）/ 环境文件驱动行 | 整改来源与 R-F- 映射 |
| HLD | §1 约束 R-C-008（24 用例 md 留 ptm-te workspace）/ R-C-002（不引入 pytest/robot） | 用例文件归属与框架约束 |
| HLD | §1 成功标准 SM-EX-07（24/24 用例 warming_up step 合规） | ARP 预热校验量化口径 |
| ADR | ADR-03（exec_v4.py 迁移后加废弃标记，不删除） | exec_v4.py 废弃标记 |
| ADR | ADR-06（ARP 预热规则+引擎双重保障） | warming_up/post_op 校验 |
| ADR | ADR-07（frontmatter 用例编号列为唯一标识；文件名按编号前缀正则匹配） | 重命名规则 |
| ADR | ADR-09（环境文件驱动，${ENV.*} 占位符 9 类） | ${ENV.*} 引用改写 |
| 需求 | R-F-004（case_steps md 驱动）/ R-F-008（warming_up/post_op）/ R-F-021（ARP 预热整改同步检查）/ R-F-022..026（用例结构化）/ R-F-027..029（环境文件驱动） | 整改覆盖需求 |
| 开发计划 | DEVELOPMENT-PLAN.yaml Wave 4 ST-EX-13 | file_ownership + depends_on |
| FDM | FEATURE-DESIGN-MATRIX-PTM-TE-EXEC.md ST-EX-13 行 | lld_policy=full-lld + feature_design_refs |
| DQ-CP3-03 | known_issue 标注纳入 R-F-021 ARP 整改同步检查 | O-03 整改时同步检查 known_issue 完整性 |

## 2. 文件影响范围

| 文件 | 归属段 | 变更类型 | 说明 |
|---|---|---|---|
| `ptm-te/cases/IPv4策略路由/配置管理/策略配置/*.md` | 24 用例 md | 新建（迁移+整改） | 从 workspace `cases/upload/`（v1 扁平）迁移到新目录结构，逐用例整改 frontmatter + case_steps |
| `ptm-te/cases/IPv4策略路由/配置管理/策略配置/README.md` | 目录说明 | 新建 | 用例目录索引与命名约定说明 |
| `ptm-te/exec_v4.py` | 废弃标记 | 修改 | 文件头部追加废弃标记注释，指向 case_runner.py（ADR-03） |

**file_ownership 声明**（与 DEVELOPMENT-PLAN.yaml 一致）：
- `ptm-te/cases/IPv4策略路由/**/*.md`
- `ptm-te/exec_v4.py#废弃标记`

**不触碰文件**：
- `skills/case-execution/scripts/case_runner.py`（ST-EX-04 拥有；本 Story 只整改用例 md，不写引擎代码）
- `skills/case-execution/SKILL.md#用例结构化约定`（ST-EX-15 拥有目录结构 + 命名 + frontmatter 16 列约定文档）
- `script/ptm_team/install.py#render_ptm_te_rule_body`（ST-EX-02 拥有规则块文本）
- `skills/policy-route-execution/scripts/op_mapper.py#resolve_env_refs`（ST-EX-03/17 拥有环境解析层实现）
- `ptm-te/cases/` 下的 topology yaml / env-file（R-C-005 不改 traffic-skill，case_runner 只读）

**用例实体位置说明**：24 用例的 case_steps 内容现存在于 ptm-te workspace 的 `cases/upload/`（v1 扁平入口）或 `exec_v4.py` 硬编码（17 用例历史基线）。canonical 仓库不跟踪用例实体（R-C-008：24 用例 md 留 ptm-te workspace）。本 Story 在 workspace 内完成迁移与整改，canonical 仓库只跟踪规则块与引擎代码。

## 3. 接口设计

### 3.1 目录结构契约

```
ptm-te/cases/
└── IPv4策略路由/                    # 协议族（一级）
    └── 配置管理/                    # 功能域（二级）
        └── 策略配置/                # 子域（三级）
            ├── README.md            # 目录索引
            ├── PC-COMB-M4-01-01.md  # 用例文件（编号前缀命名）
            ├── PC-COMB-M4-01-02.md
            └── ...                  # 24 用例
```

目录层级对应 frontmatter 的"三级目录/四级目录/五级目录"列：三级目录=协议族、四级目录=功能域、五级目录=子域。case_runner `--cases-dir ptm-te/cases/IPv4策略路由/` 递归 glob 扫描 .md 文件。

### 3.2 用例文件命名规则（ADR-07）

| 规则 | 约束 | 来源 |
|---|---|---|
| 文件名格式 | `<用例编号>.md`，编号为 frontmatter"用例编号"列值 | ADR-07 |
| 编号正则 | `^PC-[A-Z0-9]+-[A-Z0-9]+-\d{2}-\d{2}$`（如 PC-COMB-M4-01-01） | ADR-07 / DQ-06 |
| 连字符处理 | 编号内部分隔符为 `-`；文件名不含用例名称（避免连字符与分隔符冲突，Gotcha #6） | ADR-07 |
| 唯一标识 | frontmatter 用例编号列为唯一标识；文件名按编号前缀正则匹配 | ADR-07 |
| 解析失败行为 | 正则不匹配时报错让用户手动指定（RA-011） | ADR-07 回退条件 |

### 3.3 frontmatter 16 列 schema（DQ-05，8 必填 + 8 可选）

frontmatter 以 YAML 块（`---` 分隔）写在用例 md 头部，含 16 列汇总表字段。`*` 标记为必填，缺失可选列填 `N/A`（RA-012）。

| 序号 | 字段名 | 必填 | 说明 | ST-EX-13 整改动作 |
|---|---|---|---|---|
| 1 | 三级目录 | 否 | 协议族（如 IPv4策略路由） | 补全为"IPv4策略路由" |
| 2 | 四级目录 | 否 | 功能域（如 配置管理） | 补全为"配置管理" |
| 3 | 五级目录 | 否 | 子域（如 策略配置） | 补全为"策略配置" |
| 4 | 用例名称* | 是 | 用例描述性名称 | 保留原文，不参与文件名 |
| 5 | 用例编号 | 否（但 ADR-07 作唯一标识，事实必填） | PC-COMB-M4-01-01 格式 | 必须存在且与文件名一致 |
| 6 | 用例级别* | 是 | P0/P1/P2 | 保留原文 |
| 7 | 组网描述* | 是 | DUT-TG / 单设备 等 | 保留原文 |
| 8 | 组网约束 | 否 | 组网限制条件 | 缺失填 N/A |
| 9 | 预置条件 | 否 | 前置数据（如"预置策略路由 ID=pr-001"） | 保留原文或补全 |
| 10 | 测试步骤* | 是 | 用 `<br>` 分隔多步（人工摘要，冗余） | 保留原文；case_steps 是真相源（AGA-03=C） |
| 11 | 预期结果* | 是 | 判定依据（人工摘要，冗余） | 保留原文；case_steps 的 expected_result 是真相源 |
| 12 | 首次创建版本* | 是 | 如 V1 / CR-033 | 补全为 CR-033 |
| 13 | 最后变更版本 | 否 | 如 CR-033 | 补全为 CR-033 |
| 14 | 关键词 | 否 | 含 op_id，可判断是否含流量操作 | 保留或补全 op_id 列表 |
| 15 | 测试类型* | 是 | 功能 / 性能 等 | 保留原文 |
| 16 | 是否自动化* | 是 | 是 / 否 | 补全为"是" |

**冗余列说明（AGA-03=C）**：第 10 列"测试步骤"和第 11 列"预期结果"是人工摘要，case_runner 忽略冗余列，只消费 `case_steps` YAML 块的 4 字段（step_name/op_id/args/expected_result）。case_steps 是真相源。

### 3.4 tags 字段（新增，用于 --tag 精确过滤）

| 字段 | 位置 | 用途 | ST-EX-13 整改动作 |
|---|---|---|---|
| tags | frontmatter YAML 块（16 列之外新增） | `--tag` 精确过滤（ST-EX-16） | 24 用例补全 tags 字段 |

tags 与第 14 列"关键词"是**不同字段**：关键词是 ptm-tde 产出的模糊描述（含 op_id），`--keyword` 模糊匹配；tags 是 ptm-te 执行层新增的精确标签（如 `[策略路由, ARP, M4]`），`--tag` 精确过滤。

> **澄清项 LCQ-EX-13-01**：tags 是否独立于 16 列新增字段（推荐），还是复用"关键词"列？见 §14 开放项与 QUESTION-LEDGER。本 LLD 暂按"独立新增字段"设计。

### 3.5 case_steps YAML ${ENV.*} 占位符映射（ADR-09，参照 HLD §12.3 示例）

**参数分层原则**（HLD §12.3）：环境相关参数（端口/IP/next_hop/URL）用 `${ENV.*}` 占位符，禁止字面值；测试意图参数保持字面值。

| 占位符 | 解析目标 | 用例中出现位置 | 说明 |
|---|---|---|---|
| `${ENV.tg.port1}` | port_mapping.port1.tg | tx_port / txport / ports[0] | TG 发送物理端口名 |
| `${ENV.tg.port2}` | port_mapping.port2.tg | rx_port / rxport / ports[1] | TG 接收物理端口名 |
| `${ENV.tg.ports[port1,port2]}` | 聚合数组 | tg_config_interface.interfaces | 框架自动构造 port/ip/gateway 数组 |
| `${ENV.dut.next_hop}` | nodes.dut1.next_hop | fw_config_policy_route.next_hop_ip | **next_hop 概念单一来源**（不用 `${ENV.tg.port2.ip}`） |
| `${ENV.tg.url}` | nodes.tg1.trex_api_url | 不直接出现在用例（框架注入） | TREX_API_URL 来源（ADR-05） |

**改写前后对照**（HLD §12.3 PC-COMB-M4-01-01 示例）：

改写前（环境耦合，端口/IP 写死）：
```yaml
- op_id: tg_config_interface
  args:
    interfaces: '[{"port":"2_3","ip":"192.168.101.1","gateway":"192.168.101.2"},
                   {"port":"2_4","ip":"192.168.102.1","gateway":"192.168.102.2"}]'
- op_id: fw_config_policy_route
  args: {in_interface: "any", next_hop_ip: "192.168.102.1", source_network: "OBJ-SRC-192"}
- op_id: tg_apply_traffic_template
  args: {tx_port: "2_3", rx_port: "2_4", src_ip: "192.168.1.100", dst_ip: "10.0.0.1"}
```

改写后（环境无关，任意 link 通用）：
```yaml
- op_id: tg_config_interface
  args:
    interfaces: ${ENV.tg.ports[port1,port2]}   # 框架自动构造 port/ip/gateway 数组
- op_id: fw_config_policy_route
  args: {in_interface: "any", next_hop_ip: ${ENV.dut.next_hop},
         source_network: "OBJ-SRC-192", dst_network: "OBJ-DST-10", type: "ipv4"}
- op_id: tg_apply_traffic_template
  args: {tx_port: ${ENV.tg.port1}, rx_port: ${ENV.tg.port2},
         src_ip: "192.168.1.100", dst_ip: "10.0.0.1",   # 测试意图，字面值
         l4_protocol: "udp", l4_sport: 1234, l4_dport: 5678,
         traffic_mode: "count", rate: "100pps", count: 100}
- op_id: tg_start_traffic_stream
  args: {ports: "${ENV.tg.port1},${ENV.tg.port2}", txport: ${ENV.tg.port1}, rxport: ${ENV.tg.port2}}
```

### 3.6 测试意图参数（保持字面值，禁止 `${ENV.*}`）

| 参数 | 出现 op_id | 说明 |
|---|---|---|
| max_loss | tg_verify_traffic_loss | 期望丢包阈值（改进 #2 参数化，但不环境化） |
| object_name / template | fw_create_object / tg_apply_traffic_template | 对象名/模板名（测试意图） |
| src_ip / dst_ip | tg_apply_traffic_template | 流量源/目的 IP（测试意图） |
| l4_protocol / l4_sport / l4_dport | tg_apply_traffic_template | 四层协议参数（测试意图） |
| traffic_mode / rate / count | tg_apply_traffic_template | 流量模式（测试意图） |
| source_network / dst_network / type | fw_config_policy_route | 地址对象名/策略类型（测试意图） |
| in_interface | fw_config_policy_route | 一般为 "any"（字面值）；除非用例意图要求特定 DUT 接口 |
| name | tg_start_traffic_stream / tg_stop_traffic_stream | 流名称（测试意图） |

### 3.7 ptm-atomic 约束（HLD §12.3 / ADR-05/09）

TG 操作经 `ptm-atomic run tg trex <action>` 原子操作下发，框架禁止直接调 TG REST API。用例 case_steps 只声明 op_id + args，由 op_mapper build_command 构造 ptm-atomic 子进程命令。环境文件驱动仅做参数解析与 TREX_API_URL 注入（_build_exec_env 环境变量），由 ptm-atomic 子进程消费。用例整改不得引入直接 REST API 调用。

## 4. 核心流程

### 4.1 24 用例整改主流程

```
[1] 目录迁移
  ├─ 在 workspace 创建 ptm-te/cases/IPv4策略路由/配置管理/策略配置/ 目录
  ├─ 从 cases/upload/（v1 扁平）或 exec_v4.py 硬编码基线提取 24 用例 case_steps
  └─ 旧 cases/upload/ 保留废弃标记（RA-010，不删除）
      │
[2] 重命名（ADR-07）
  ├─ 文件名改为 <用例编号>.md（如 PC-COMB-M4-01-01.md）
  ├─ 编号正则 ^PC-[A-Z0-9]+-[A-Z0-9]+-\d{2}-\d{2}$ 校验
  └─ 编号与 frontmatter 用例编号列一致
      │
[3] frontmatter 16 列补全（DQ-05）
  ├─ 8 必填列校验（用例名称/用例级别/组网描述/测试步骤/预期结果/首次创建版本/测试类型/是否自动化）
  ├─ 8 可选列缺失填 N/A（RA-012）
  ├─ 三级/四级/五级目录补全为 IPv4策略路由/配置管理/策略配置
  └─ 首次创建版本/最后变更版本补全为 CR-033
      │
[4] tags 字段补全
  ├─ 每用例新增 tags 字段（如 [策略路由, ARP, M4]）
  └─ LCQ-EX-13-01 确认后定稿（暂按独立新增字段）
      │
[5] ${ENV.*} 引用改写（ADR-09，HLD §12.3 示例）
  ├─ 扫描 case_steps 中环境相关参数（端口/IP/next_hop）
  ├─ tg_config_interface.interfaces -> ${ENV.tg.ports[port1,port2]}
  ├─ fw_config_policy_route.next_hop_ip -> ${ENV.dut.next_hop}（不用 ${ENV.tg.port2.ip}）
  ├─ tg_apply_traffic_template.tx_port/rx_port -> ${ENV.tg.port1}/${ENV.tg.port2}
  ├─ tg_start_traffic_stream.ports/txport/rxport -> ${ENV.tg.port1}/${ENV.tg.port2}
  ├─ 测试意图参数保持字面值（max_loss/src_ip/dst_ip/object_name/l4_*/...）
  └─ 无 ${ENV.*} 的字面值原样透传（Gotcha #12 向后兼容）
      │
[6] ARP 预热校验（R-F-021，SM-EX-07 24/24）
  ├─ 含 warming_up:true 的 step 必须有 post_op 或引擎自动补充
  ├─ warming_up step 的 post_op（如 tg_stop_traffic_stream）参数从主 op 继承
  ├─ 无 warming_up step 的用例标记 N/A（不强制）
  └─ 24 用例 warming_up step 合规率 24/24
      │
[7] known_issue 标注校验（DQ-CP3-03，O-03，R-F-021 同步检查）
  ├─ 已知 DUT 行为差异的 step 必须标注 known_issue:true
  ├─ M4-01-09 STEP-007 删除被引用对象（eBeingReferenced）典型 known_issue
  ├─ 缺失 known_issue 标注的疑似场景标记 OPEN 进 BACKLOG
  └─ known_issue 完整性纳入 R-F-021 ARP 整改同步检查
      │
[8] exec_v4.py 废弃标记（ADR-03）
  ├─ 文件头部追加废弃标记注释
  └─ 指向 case_runner.py 新入口
```

### 4.2 用例分类与整改矩阵

24 用例按 op_id 组合分三类，整改重点不同：

| 类别 | op_id 特征 | 用例数（估） | 整改重点 |
|---|---|---|---|
| A: 纯 DUT 用例 | 仅 fw_* op，无 tg_* | ~8 | frontmatter 补全 + tags；无 ${ENV.*} 改写 |
| B: DUT + TG 流量用例 | fw_* + tg_config_interface + tg_apply_traffic_template + tg_start/stop + tg_verify_traffic_loss | ~12 | 全量整改：${ENV.*} 改写 + ARP 预热校验 + max_loss 参数化 |
| C: TG only / 特殊用例 | tg_* 主导或含 known_issue | ~4 | ${ENV.*} 改写 + known_issue 标注校验 |

> 用例数为整改前估算，实施时以 workspace 实际 24 用例为准。三类覆盖 24 用例全量。

## 5. 异常处理

| 异常场景 | 检测时机 | 失败行为 | 错误码/标记 | 来源 |
|---|---|---|---|---|
| frontmatter 16 列必填缺失 | 整改校验 | 报错列出缺失列，该用例标记 PENDING_FIX | FRONTMATTER_INCOMPLETE | RA-012 / DQ-05 |
| 用例编号正则不匹配 | 重命名校验 | 报错让用户手动指定文件名 | CASE_ID_FORMAT_INVALID | RA-011 / ADR-07 |
| 编号与文件名不一致 | 重命名校验 | 报错，以 frontmatter 编号为准重命名 | CASE_ID_MISMATCH | ADR-07 |
| ${ENV.*} 占位符无对应 env_topology 键 | dry-run 校验（resolve_env_refs） | 该 step 标记 error，envelope 返回 VALIDATION_FAILED | VALIDATION_FAILED | HLD §12.3 / ADR-09 |
| warming_up step 无 post_op 且引擎未补充 | ARP 预热校验 | 引擎强制补充 post_op（ST-EX-08）；md 缺失不阻断（双重保障） | auto_post_op=true | ADR-06 / R-F-008 |
| known_issue 标注缺失（疑似场景） | known_issue 校验 | 标记 OPEN 进 BACKLOG，不阻断整改 | KNOWN_ISSUE_REVIEW_NEEDED | O-03 / DQ-CP3-03 |
| 用例含直接 REST API 调用 | 整改校验 | 报错，改为 op_id 声明经 ptm-atomic | REST_API_VIOLATION | ADR-05/09 |
| 旧 cases/upload/ 缺失 | 目录迁移 | 跳过迁移，从 exec_v4.py 硬编码基线重建 | LEGACY_SOURCE_FALLBACK | RA-010 |
| ${ENV.*} 改写后字面值残留（端口/IP） | dry-run 校验 | 报告残留字面值列表，人工复核 | LITERAL_PORT_IP_REMNANT | ADR-09 |

**校验失败不阻断整体整改**：单用例校验失败标记 PENDING_FIX，其余用例继续整改；PENDING_FIX 用例清单写入 README.md 的"待修复"章节。

## 6. 测试设计

### 6.1 验证层与 validation_mode

validation_mode = static-only / dry-run-only（CP7 采用 static review + dry-run，DQ-01 推荐）。不真实执行设备写操作（NO_PRODUCTION_WRITE）。runtime 端到端验证属 T-01 follow-up（需设备 + --execute 授权），CP7 写明 N/A 理由与未覆盖风险。

### 6.2 测试矩阵

| 测试项 | 验证方法 | 预期结果 | 关联指标 |
|---|---|---|---|
| 目录结构合规 | `find ptm-te/cases/IPv4策略路由/配置管理/策略配置/ -name "*.md"` | 24 用例 + README.md | R-F-022 |
| 文件名编号正则 | `ls | grep -E '^PC-[A-Z0-9]+-[A-Z0-9]+-[0-9]{2}-[0-9]{2}\.md$'` | 24 文件全匹配 | ADR-07 / SM-EX-02 |
| frontmatter 16 列完整 | 解析 YAML 块，校验 8 必填非空 + 8 可选填 N/A | 24 用例全合规 | DQ-05 / RA-012 |
| 编号与文件名一致 | frontmatter 用例编号 == 文件名前缀 | 24 用例全一致 | ADR-07 |
| tags 字段存在 | 解析 YAML 块，tags 字段非空 | 24 用例全有 tags | R-F-026（LCQ-EX-13-01 待确认） |
| ${ENV.*} 改写完整 | grep case_steps 无残留字面端口/IP（2_3/2_4/192.168.*） | 0 残留（测试意图 IP 除外） | ADR-09 / SM-EX-03 |
| next_hop 单一来源 | grep next_hop_ip 值为 ${ENV.dut.next_hop} | 不含 ${ENV.tg.port2.ip} | HLD §12.3 约束 |
| 测试意图参数字面值 | grep max_loss/src_ip/dst_ip/object_name 为字面值 | 非 ${ENV.*} | HLD §12.3 约束 |
| ARP 预热 24/24 合规 | 含 warming_up 的 step 有 post_op 或引擎补充 | 24/24 合规 | SM-EX-07 / R-F-021 |
| known_issue 标注 | 被引用对象删除等场景有 known_issue:true | 疑似场景全标注，缺失进 BACKLOG | O-03 / DQ-CP3-03 |
| dry-run 批量校验 | `case_runner.py run --cases-dir ptm-te/cases/IPv4策略路由/ --dry-run` | 24 用例全 PASS（dry-run） | R-F-004 / DA-002 |
| ${ENV.*} 解析预校验 | dry-run 模式 resolve_env_refs 全部占位符可解析 | 0 VALIDATION_FAILED | ADR-09 / RA-013 |
| ptm-atomic 约束 | grep case_steps 无直接 REST API 调用 | 0 违规 | ADR-05/09 |
| exec_v4.py 废弃标记 | head exec_v4.py 含废弃注释 | 含"已废弃，改用 case_runner.py" | ADR-03 / RA-009 |
| 向后兼容（无 ${ENV.*} 字面值透传） | 含纯 DUT 用例 dry-run | 字面值原样透传，PASS | Gotcha #12 |

### 6.3 dry-run 校验脚本逻辑（不引入 pytest，R-C-002）

```
case_runner.py run --cases-dir ptm-te/cases/IPv4策略路由/ --devices-yaml devices.yaml \
  --topology-yaml <path> --env-file <path> --dry-run
  ├─ [1] which ptm-atomic（DA-001）
  ├─ [2] devices.yaml 含 firewall.host + tg.api_server（DA-003）
  ├─ [3] env-file 含 port_mapping/nodes/links（DA-006/007）
  ├─ [4] 逐用例解析 frontmatter 16 列 + case_steps YAML
  ├─ [5] 逐 step resolve_env_refs（${ENV.*} 解析）+ resolve_step_refs + validate_args
  ├─ [6] build_command（dry_run=True，不调 ptm-atomic）
  └─ [7] 输出 24 用例 dry-run 结果（PASS/VALIDATION_FAILED/PENDING_FIX）
```

## 7. 实施步骤

1. **确认 24 用例实体**：在 ptm-te workspace 定位 24 用例 case_steps 来源（`cases/upload/` 或 `exec_v4.py` 硬编码 17 用例基线 + 新增 7 用例）。若 `cases/upload/` 缺失，从 `exec_v4.py` 提取 17 用例并补建 7 用例（来源由实施时 workspace 现状决定）
2. **创建目录结构**：`mkdir -p ptm-te/cases/IPv4策略路由/配置管理/策略配置/`
3. **逐用例迁移与重命名**（ADR-07）：24 用例文件命名为 `<编号>.md`，编号正则校验，编号与 frontmatter 一致
4. **frontmatter 16 列补全**（DQ-05）：8 必填校验，8 可选填 N/A，三级/四级/五级目录补全，版本补全 CR-033
5. **tags 字段补全**：24 用例新增 tags（LCQ-EX-13-01 确认后定稿，暂按独立新增字段，如 `[策略路由, ARP, M4]`）
6. **${ENV.*} 引用改写**（ADR-09，HLD §12.3 示例）：
   - tg_config_interface.interfaces -> `${ENV.tg.ports[port1,port2]}`
   - fw_config_policy_route.next_hop_ip -> `${ENV.dut.next_hop}`（不用 `${ENV.tg.port2.ip}`）
   - tg_apply_traffic_template.tx_port/rx_port -> `${ENV.tg.port1}`/`${ENV.tg.port2}`
   - tg_start_traffic_stream.ports/txport/rxport -> `${ENV.tg.port1}`/`${ENV.tg.port2}`
   - 测试意图参数保持字面值
7. **ARP 预热校验**（R-F-021，SM-EX-07）：含 warming_up 的 step 校验 post_op，无 post_op 由引擎补充（ST-EX-08）；24/24 合规
8. **known_issue 标注校验**（DQ-CP3-03，O-03）：疑似 DUT 行为差异场景标注 known_issue:true，缺失进 BACKLOG
9. **exec_v4.py 废弃标记**（ADR-03）：文件头部追加废弃注释，指向 case_runner.py
10. **README.md 创建**：目录索引 + 命名约定 + 待修复用例清单（PENDING_FIX）
11. **dry-run 批量校验**：`case_runner.py run --cases-dir ... --dry-run`，24 用例全 PASS
12. **旧 cases/upload/ 废弃标记**（RA-010）：保留但不删除，追加废弃说明指向新目录

## 8. 回滚策略

| 回滚场景 | 回滚方式 | 影响 | 来源 |
|---|---|---|---|
| 用例整改有误 | git revert（workspace 版本控制） | 恢复整改前 case_steps | ADR-03 |
| exec_v4.py 废弃标记误加 | 移除废弃注释 | exec_v4.py 恢复可用 | ADR-03 |
| 目录迁移后 case_runner 找不到用例（RA-010） | 旧 cases/upload/ 保留废弃标记，可回退 | 旧入口仍可用 | RA-010 |
| ${ENV.*} 改写导致解析失败 | 恢复字面值（向后兼容，Gotcha #12） | 字面值原样透传 | ADR-09 |
| frontmatter 补全破坏原数据 | git revert 单用例 | 恢复原 frontmatter | RA-012 |

**回退条件**：废弃标记后 exec_v4.py 仍被误用（RA-009），删除文件（ADR-03 回退条件）。整改后 dry-run 批量失败率 > 10%，回退整批用例并人工复核。

## 9. 平台差异

本 Story 是纯 md 用例文件整改，不涉及 Claude Code / Codex / Qoder 平台安装差异。用例文件留 ptm-te workspace（R-C-008），不随 install.py 安装到 workspace `.claude/skills/`。

| 维度 | 评估 | 说明 |
|---|---|---|
| 安装路径 | 无差异 | 用例 md 不经 install.py 安装 |
| 规则块 | 无差异 | install.py 规则块由 ST-EX-02 拥有 |
| 平台规则文件 | 无差异 | CLAUDE.md / AGENTS.md 规则块不含用例路径硬编码 |
| 路径分隔符 | 无差异 | 用例目录用中文 + `/`，跨平台一致 |

## 10. 数据模型

### 10.1 用例 md 文件结构

```markdown
---
三级目录: "IPv4策略路由"
四级目录: "配置管理"
五级目录: "策略配置"
用例名称: "策略路由配置-基础创建-正向"
用例编号: "PC-COMB-M4-01-01"
用例级别: "P0"
组网描述: "DUT-TG"
组网约束: "N/A"
预置条件: "已登录 DUT"
测试步骤: "STEP-001 创建对象<br>STEP-002 配置策略路由<br>..."
预期结果: "STEP-001 创建成功<br>STEP-002 返回策略 ID<br>..."
首次创建版本: "CR-033"
最后变更版本: "CR-033"
关键词: "fw_create_object,fw_config_policy_route,tg_apply_traffic_template"
测试类型: "功能"
是否自动化: "是"
tags: ["策略路由", "M4", "ARP"]
---

## case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 创建源地址对象
    target: DUT
    atomic_op:
      op_id: fw_create_object
      args: {name: "OBJ-SRC-192", type: "ipv4", subnet: "192.168.0.0/24"}
    expected_result: 创建成功，返回对象 ID
  - step_id: STEP-002
    step_name: 配置策略路由
    target: DUT
    atomic_op:
      op_id: fw_config_policy_route
      args:
        in_interface: "any"
        next_hop_ip: ${ENV.dut.next_hop}
        source_network: "OBJ-SRC-192"
        dst_network: "OBJ-DST-10"
        type: "ipv4"
    expected_result: 创建成功，返回策略 ID
    depends_on: STEP-001
  - step_id: STEP-003
    step_name: 配置 TG 接口
    target: TG
    atomic_op:
      op_id: tg_config_interface
      args:
        interfaces: ${ENV.tg.ports[port1,port2]}
    expected_result: TG 接口配置成功
  - step_id: STEP-004
    step_name: 应用流量模板并打流（ARP 预热）
    target: TG
    atomic_op:
      op_id: tg_apply_traffic_template
      args:
        tx_port: ${ENV.tg.port1}
        rx_port: ${ENV.tg.port2}
        src_ip: "192.168.1.100"
        dst_ip: "10.0.0.1"
        l4_protocol: "udp"
        l4_sport: 1234
        l4_dport: 5678
        traffic_mode: "count"
        rate: "100pps"
        count: 100
    expected_result: 流量模板应用成功
    warming_up: true
  - step_id: STEP-005
    step_name: 启动流量
    target: TG
    atomic_op:
      op_id: tg_start_traffic_stream
      args:
        ports: "${ENV.tg.port1},${ENV.tg.port2}"
        txport: ${ENV.tg.port1}
        rxport: ${ENV.tg.port2}
        name: "stream-M4-01-01"
    expected_result: 流量启动成功
    depends_on: STEP-004
  - step_id: STEP-006
    step_name: 停止流量并校验丢包
    target: TG
    atomic_op:
      op_id: tg_stop_traffic_stream
      args:
        ports: "${ENV.tg.port1},${ENV.tg.port2}"
    expected_result: 流量停止成功
  - step_id: STEP-007
    step_name: 校验丢包率
    target: TG
    atomic_op:
      op_id: tg_verify_traffic_loss
      args:
        tx_port: ${ENV.tg.port1}
        rx_port: ${ENV.tg.port2}
        max_loss: 0
    expected_result: loss_ratio <= max_loss
    depends_on: STEP-006
```
```

### 10.2 frontmatter 16 列 + tags 字段清单

见 §3.3 + §3.4。frontmatter YAML 块字段顺序遵循 16 列汇总表顺序，tags 追加在末尾。

### 10.3 case_steps YAML 字段

| 字段 | 必填 | 说明 | ${ENV.*} 改写 |
|---|---|---|---|
| step_id | 是 | STEP-001 格式 | 否 |
| step_name | 是 | 测试动作意图 | 否 |
| target | 是 | DUT / TG | 否 |
| atomic_op.op_id | 是 | 原子操作标识 | 否 |
| atomic_op.args | 是 | 操作参数 | 环境相关参数改写为 ${ENV.*} |
| expected_result | 是 | 预期结果 | 否 |
| depends_on | 否 | 跨步骤依赖（STEP-N） | 否 |
| warming_up | 否 | ARP 预热标记（true 时引擎强制 post_op） | 否 |
| post_op | 否 | 预热后清理 op（缺失由引擎补充） | 否 |
| retry | 否 | 轮询次数/间隔 | 否 |
| known_issue | 否 | 已知 DUT 行为差异标记 | 否 |

## 11. 性能

| 指标 | 目标 | 验证方式 | 来源 |
|---|---|---|---|
| 24 用例 dry-run 批量校验时间 | < 1 分钟 | dry-run 模式计时 | 估算（无进程开销，纯解析） |
| 24 用例整改人工耗时 | ~2 人日 | 含迁移+重命名+frontmatter+${ENV.*} 改写+校验 | 工作量 L（HLD §19） |
| 单用例 md 解析时间 | < 100ms | parse_case_file 计时 | 估算 |

> dry-run 性能指标在 static-only / dry-run-only 下可直接验证（无设备依赖）。runtime 执行时间属 T-01 follow-up。

## 12. 安全

| 约束 | 说明 | 来源 |
|---|---|---|
| NO_CREDENTIAL_READ | 用例 md 不含凭据；password_env 用 `${ENV_VAR}` 占位（如 FW_WEB_PASSWORD） | 执行指导 §3.2 |
| NO_PRODUCTION_WRITE | 整改校验用 dry-run，不真实执行设备写操作 | ADR-02 / DQ-01 |
| NO_EXTERNAL_PUBLISH | 用例 md 留 ptm-te workspace，不发布外部 | R-C-008 |
| 凭据隔离 | devices.yaml 不入库明文，${ENV_VAR} 占位，--password-env 传 Web 密码 | 执行指导 关键设计 |
| ptm-atomic 约束 | TG 操作经 ptm-atomic，禁止直接调 TG REST API | ADR-05/09 |

## 13. 依赖

### 13.1 依赖关系（DEVELOPMENT-PLAN.yaml）

| 依赖 Story | 依赖类型 | 依赖原因 | 阻塞 LLD |
|---|---|---|---|
| ST-EX-08（ARP 预热引擎） | hard | warming_up/post_op 引擎强制逻辑由 ST-EX-08 实现；本 Story 只校验用例 warming_up 标记合规，引擎补充 post_op 依赖 ST-EX-08 | 否（LLD 可并行，实施需 ST-EX-08 就绪） |
| ST-EX-15（用例结构化约定） | hard | 目录结构 + 命名 + frontmatter 16 列约定文档由 ST-EX-15 定义；本 Story 遵循其约定 | 否（LLD 可并行，实施需 ST-EX-15 约定定稿） |
| ST-EX-17（环境解析层） | hard | ${ENV.*} 占位符解析由 ST-EX-17 resolve_env_refs 实现；本 Story 改写用例为 ${ENV.*} 引用，依赖 ST-EX-17 解析能力 | 否（LLD 可并行，实施需 ST-EX-17 就绪） |

**依赖门控说明**：3 个 hard 依赖均为实施时门控（dev_gate），不阻塞 LLD 写作（CP5 并行 LLD）。ST-EX-13 的 ${ENV.*} 改写规则已在 HLD §12.3 定稿，LLD 据此设计，不依赖 ST-EX-17 LLD 内容。

### 13.2 下游影响

| 下游 | 影响 | 说明 |
|---|---|---|
| ST-EX-14（verify_loss 消费侧） | 无直接依赖 | ST-EX-14 消费 case_runner extract_payload，不直接消费用例 md |
| ST-EX-16（标签/关键字执行） | 软依赖 | --tag 过滤依赖 tags 字段补全（LCQ-EX-13-01） |
| runtime 验证（T-01 follow-up） | 数据依赖 | 24 用例整改后才能 runtime 端到端验证 |

## 14. 开放项

| 问题 ID | 问题 | 状态 | 推荐方案 | 影响 | 阻塞 LLD |
|---|---|---|---|---|---|
| LCQ-EX-13-01 | frontmatter tags 字段与 16 列"关键词"列关系：tags 是独立新增字段（推荐）还是复用关键词列？ | OPEN（已写 QUESTION-LEDGER） | A：tags 独立新增字段，用于 --tag 精确过滤；关键词列保留用于 --keyword 模糊匹配 | ST-EX-13 frontmatter 补全规则 + ST-EX-16 --tag 过滤实现 | 否（LLD 按推荐方案 A 设计，实施时确认） |
| O-03 | 24 用例 known_issue 标注完整性 | OPEN（整改时同步检查） | DQ-CP3-03 纳入 R-F-021 ARP 整改同步检查；缺失标 OPEN 进 BACKLOG | ST-EX-13 §4.1 [7] known_issue 校验 | 否 |
| 24 用例实体来源 | 24 用例 case_steps 现存位置（cases/upload/ 或 exec_v4.py 硬编码） | OPEN（实施时 workspace 现状确认） | cases/upload/ 优先，缺失时从 exec_v4.py 17 用例基线重建 + 补建 7 用例 | ST-EX-13 实施步骤 [1] | 否（LLD 不依赖具体实体位置） |
| 用例数估算 | 三类用例数（A/B/C）为估算，实施时以 workspace 实际 24 用例为准 | OPEN（实施时核对） | §4.2 估算 ~8/~12/~4 | 整改工作量分配 | 否 |

## 与 HLD/ADR 追溯一致性

| HLD/ADR 条目 | 本 Story 落实点 | 一致性 |
|---|---|---|
| HLD §12.3 用例迁移示例（PC-COMB-M4-01-01） | §3.5 ${ENV.*} 占位符映射 + §4.1 [5] 改写 + §10.1 用例结构示例 | 一致（严格参照改写前/改写后） |
| HLD §12.1 [3] 用例发现（--tag/--keyword） | §3.4 tags 字段 + §3.3 关键词列 | 一致（tags 独立、关键词保留） |
| HLD §16.1 改进 #3（ARP 预热 24 用例校验） | §4.1 [6] ARP 预热校验 + §6.2 24/24 合规 | 一致（SM-EX-07） |
| HLD §16.1 改进 #5（用例文件驱动） | §3.1 目录结构 + §3.3 frontmatter 16 列 | 一致（R-F-022..026） |
| HLD §16.1 改进 #10（devices.yaml 取址） | §3.5 ${ENV.*} 改写消除字面 IP | 一致（SM-EX-03） |
| HLD §16.1 环境文件驱动行（R-F-027/028/029） | §3.5 ${ENV.*} 占位符 + §3.6 测试意图字面值 | 一致（ADR-09） |
| HLD §1 约束 R-C-002（不引入 pytest/robot） | §6.3 dry-run 校验脚本逻辑（case_runner 内置） | 一致 |
| HLD §1 约束 R-C-008（24 用例 md 留 workspace） | §2 文件影响范围（ptm-te workspace） | 一致 |
| HLD §1 SM-EX-07（24/24 warming_up 合规） | §4.1 [6] + §6.2 ARP 预热 24/24 | 一致 |
| HLD §1 SM-EX-03（0 硬编码 IP） | §3.5 ${ENV.*} 改写 + §6.2 无残留字面端口/IP | 一致 |
| ADR-03（exec_v4.py 废弃标记） | §4.1 [8] + §6.2 废弃标记校验 | 一致 |
| ADR-06（ARP 预热双重保障） | §4.1 [6] 引擎强制 post_op + §5 异常处理 | 一致 |
| ADR-07（编号正则匹配） | §3.2 文件命名规则 + §4.1 [2] 重命名 | 一致 |
| ADR-09（环境文件驱动 ${ENV.*}） | §3.5 占位符映射 + §4.1 [5] 改写 + §5 解析失败处理 | 一致 |
| ADR-05（TREX_API_URL 注入，ptm-atomic 约束） | §3.7 ptm-atomic 约束 + §6.2 无直接 REST API | 一致 |
| DQ-CP3-03（known_issue 纳入 R-F-021） | §4.1 [7] known_issue 校验 + §14 O-03 | 一致 |
| RA-010（目录迁移路径变更） | §4.1 [1] + §8 回滚（旧 cases/upload/ 保留） | 一致 |
| RA-011（连字符冲突） | §3.2 文件名只用编号、不含名称 | 一致 |
| RA-012（frontmatter 补全工作量） | §3.3 8 必填+8 可选 N/A | 一致 |
| RA-013（${ENV.*} 解析失败） | §5 VALIDATION_FAILED + §6.2 dry-run 预校验 | 一致 |
| Gotcha #6（连字符与分隔符冲突） | §3.2 文件名只用编号 | 一致 |
| Gotcha #11（YAML 1.1 整数陷阱，端口名加引号） | §10.3 args 中端口名由 ${ENV.*} 解析（环境文件侧加引号） | 一致 |
| Gotcha #12（向后兼容，无 ${ENV.*} 字面值透传） | §5 + §6.2 纯 DUT 用例字面值透传 | 一致 |
| Gotcha #13（ptm-atomic 约束，禁止直接 REST API） | §3.7 + §6.2 无直接 REST API | 一致 |
