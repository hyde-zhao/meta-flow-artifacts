---
checkpoint_id: "CP6"
checkpoint_name: "编码完成检查"
type: "auto"
status: "PASS"
story_id: "CR037-S5"
story_slug: "exporter"
cr_id: "CR-037"
wave: 2
design_evidence_type: "full-lld"
executed_by: "meta-dev"
executed_at: "2026-08-05T00:00:00+00:00"
---

# CP6 编码完成检查 — CR037-S5 exporter.py

## 1. 实现对象清单

| 文件 | 操作 | 说明 |
|------|:---:|------|
| `skills/topo-planning/src/exporter.py` | 新建 | env-file 导出：metadata + port_mapping + nodes + links（9 类 `${ENV.*}` 占位符硬契约，端口名引号化）；命名规则（默认 topology/topology.yaml + 描述性命名）；link 数一致性（LINK_COUNT_MISMATCH）；凭据禁止（ADR-02）；结构化失败报告（JSON + 人读 + --report-out）；MVP 边界（R1：TOPO_NODE_COUNT_UNSUPPORTED）；IP 规划接入（缺 IP 默认报错 IP_PLAN_REQUIRED/IP_PLAN_INCOMPLETE，显式降级 executable=false）；确定性（generated_at 移出比较范围，R2）+ `compute_ip_plan`（S6 桥接，同文件 #IP 规划段）+ CLI main() |
| `skills/topo-planning/tests/test_exporter.py` | 新建 | 36 个 pytest 单元测试（S5 + S6） |
| `skills/topo-planning/tests/fixtures/pool_direct_tg_dut.yaml` | 新建 | 单 TG + 单 DUT 直连 3 链路池（MVP 边界内） |
| `skills/topo-planning/tests/fixtures/pool_sw.sample.yaml` | 新建 | 单 SW 双端口连接 TG/DUT 池（S6 SW 拓扑测试） |

## 2. 设计契约映射

| LLD / Story 契约 | 实现 |
|------|------|
| `export_env_file(mapping_result, logical_topo, pool, ip_plan=None, output_path=None, allow_degraded=False) -> str`（FR1） | ✅ `EnvFileExporter.export_env_file` + 模块级便捷入口 |
| 四段 env-file：metadata + port_mapping + nodes + links（FR2，DA-006） | ✅ 顶层四字段；metadata 含 executable/degraded_reason |
| `${ENV.*}` 9 类占位符可解析（FR3） | ✅ 测试 `test_export_placeholder_contract` 经 `op_mapper.resolve_env_refs` 全部解析 |
| 端口名引号化（FR4，Gotcha #11） | ✅ `_quote_port` 强制 str；测试 `test_export_port_quoted` |
| 命名：默认 topology/topology.yaml + --output 描述性（FR5） | ✅ CLI 默认 `topology/topology.yaml`；`_descriptive_name` 生成描述性名；`--output` 自定义 |
| link 数一致性（FR6，S5-B）：真实逻辑 link（不含 Mock、不含 L2 透传）= 直连 matched 数 | ✅ `_check_link_count`：`logical_physical -= pass_through_ids`，`actual_direct = matched 且非透传`；测试 `test_mock_link_excluded` / `test_l2_transparent_link_not_counted` |
| 凭据禁止（FR7，ADR-02） | ✅ 导出路径不读 password/token；`_CREDENTIAL_RE` 断言产物无凭据；测试 `test_export_no_credentials` |
| 结构化失败报告（FR8）：缺设备/缺 TE 端口/链路约束不满足 | ✅ `build_failure_report`（3 类问题 + link_count）；`render_failure_report` 人读 |
| --report-out 落盘 JSON + 人读（FR9，O-c） | ✅ `write_report`：`<path>.json` + `<path>.txt`；测试 `test_report_out_write` |
| IP 规划接入（FR10，S5-C）：权威 schema = ipam.ip_plan_to_dict；缺 IP 默认报错 / 显式降级 executable=false | ✅ `_resolve_ip_plan`（gateway.ip→DUT、endpoints[].ip→TG、endpoint.gateway→TG.gw、next_hop=gateway.ip 去掩码）；IP_PLAN_REQUIRED / IP_PLAN_INCOMPLETE / allow_degraded 降级 |
| MVP 边界（FR7b，R1）：单 TG + 单 DUT 直连；多 DUT/SW/Mock → TOPO_NODE_COUNT_UNSUPPORTED | ✅ `_validate_mvp_topology` 前置校验节点 node_type 分布；测试 `test_multinode_topology_rejected` / `test_sw_topology_rejected` / `test_mock_topology_rejected` / `test_mvp_single_dut_passes` |
| 失败不产出 env-file（FR11） | ✅ 映射失败路径走 `build_failure_report` + `write_report`，不调 export_env_file |
| 确定性（NFR2，R2）：generated_at 移出比较范围或注入固定值 | ✅ `export_env_file(generated_at=...)` 注入 + 剔除后逐字节一致（`test_export_deterministic_ignore_generated_at` / `_inject_generated_at`） |
| 产物通过 case_runner load_env_file（NFR1） | ✅ `test_load_env_file_accepts_export`（load_env_file + build_env_topology） |

## 3. 测试结果

- 测试文件：`skills/topo-planning/tests/test_exporter.py`
- 运行命令：`uv run --python 3.11 python -m pytest skills/topo-planning/tests/test_exporter.py`
- 结果：**36 passed**（0.54s）

覆盖：env-file 四段 / 端口引号化 / tg_api_server / dut_host / 无凭据 / 9 类占位符（resolve_env_refs）/ link 数 OK / mismatch / mock link 排除 / **L2 透传 link 不计入** / IP 渲染 / 缺 IP 拒绝 / 降级 flag / 不完整 / 权威 schema / **多节点拒绝（R1）** / SW 拒绝 / Mock 拒绝 / MVP 单 DUT 通过 / **确定性（R2：剔除 generated_at + 注入固定值）** / 失败报告 3 类 / report-out 落盘 / 人读渲染 / 自定义输出 / 描述性命名 / load_env_file 消费 / S6 compute_ip_plan（自动/preset/容量/SW vlan 缺失/SW L2 域缺失/权威 schema）。

## 4. 既有测试回归情况

- 全量 `skills/topo-config/tests/ + skills/topo-planning/tests/`：失败清单与基线（Wave 0-1 记录）**逐条一致**，无新增回归；topo-planning 全部 112 passed（S1 14 + S2 41 + S3 21 + exporter 36）。
- exporter.py 为新增文件，不修改既有模块；case-execution / policy-route-execution 只读消费。

## 5. 偏离 LLD 的设计点

| 偏离点 | 说明 |
|--------|------|
| `export_env_file` 增加 `generated_at` 可选参数 | LLD FR1 签名未含；为满足 R2 确定性注入（`_build_metadata(generated_at=...)`），在末尾追加可选 kwarg（向后兼容），默认 None=当前时间戳 |
| `compute_ip_plan` 增加 `_supplement_direct_domains` 补充 TG/PC↔DUT 直连 p2p 域 | S6 复用 `topology_exporter._compute_broadcast_domains`，但该函数仅覆盖 DUT-DUT 直连 L3 域；单 TG + 单 DUT 直连 MVP（node2_dut1_tg1_linkN）需补充 TG↔DUT 域才能产出完整 ip_plan（S6-A 评审边界内适配层，见 S6 CP6） |
| 广播域 gateway 为 None 时清洗为 {} | `_compute_l2_broadcast_domains` 对无 L3 网关的 SW L2 域置 gateway=None，`ipam.IPAM.plan` 对 None 崩溃；compute_ip_plan 清洗为 {} 再入 IPAM（S6 适配层） |
| `_quote_port` 仅强制 str，不做额外转义 | `yaml.safe_dump` 对含 `/` 或数字型字符串自动加引号；显式确保类型 str（Gotcha #11 语义） |
| CLI 逻辑 topo 支持 topo_id 或 YAML 路径 | 通过 `find_topology_file`（S1）解析 topo_id，便于直接传入 `node2_dut1_tg1_link3` 类 ID |
