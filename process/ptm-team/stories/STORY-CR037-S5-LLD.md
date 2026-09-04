---
doc_type: LLD
story_id: "CR037-S5"
story_slug: "exporter"
cr_id: "CR-037"
tier: "A"
status: "draft"
created_at: "2026-08-05"
owner: "meta-dev"
source_hld: "process/changes/CR-037-HLD.md §4.5 + §7"
source_story: "process/stories/STORY-CR037-S5.md"
design_evidence_type: "full-lld"
lld_policy_required_level: "full-lld"
scope: "skills/topo-planning/src/exporter.py 新建（env-file 导出 + 命名 + link 数一致性 + 凭据禁止 + 结构化失败报告）"
---

# STORY-CR037-S5-LLD：exporter.py 详细设计

> **Tier**: A（产物契约 9 类占位符硬契约 + 命名/失败报告 schema 需完整设计）
> **Shared Fragments**: `skills/case-execution/templates/topology.yaml.example`（env-file 契约，只读消费）、`skills/case-execution/scripts/case_runner.py`（`load_env_file`/`resolve_env_refs`，只读消费）、`skills/topo-config/src/topo_mapper.py`（MappingResult，S4）、`skills/topo-config/src/physical_pool.py`（PhysicalPool，只读消费）
> **Open Items**: 无

---

## 0. 工程依据

本模块是 CR-037 topo-planning 链路的产物导出组件。工程依据源自 HLD §4.5（exporter：映射结果 → env-file，metadata + port_mapping + nodes + links，满足 `${ENV.*}` 9 类占位符硬契约；命名默认 `topology/topology.yaml` + `--output` 描述性命名；link 数一致性校验；凭据禁止 ADR-02；结构化失败报告 JSON + 人读 + `--report-out`，O-c），以及 HLD §7 产物契约表（9 类占位符，端口名加引号 Gotcha #11）。S5 消费 S4 `MappingResult`（含 `per_link_requirements` 与 `allocated_env`）与 S6 IP 规划结果（`ip_plan`）作为强输入，产出 case-execution 零适配消费的 env-file。核心价值：把「映射成功」固化为可被 ptm-te 直接执行的稳定产物，把「映射失败」固化为结构化限制校验报告（缺设备/缺 TE 端口/链路约束不满足），不兜底。

**评审修订（CP5 NEEDS_REWORK 综合审核打回）**：IP 规划缺失（`ip_plan=None`）时**不得**产出静默空 IP 的 env-file——`case-execution` 的 `resolve_env_refs`（op_mapper.py:1123-1125）对空 IP 字段抛 `ValueError`，会击穿「env-file 零适配」承诺。修订为：默认缺 IP 即显式报错（`IP_PLAN_REQUIRED` / `IP_PLAN_INCOMPLETE`）；仅显式允许降级（dry-run / `--allow-degraded`）时产出标记 `metadata.executable=false` + `degraded_reason` 的降级产物，且该产物**禁止进入 case-execution 消费路径**。link 数一致性口径同步修正：`TopoMapper._try_l2_pass_through` 自动透传 link（`LinkMapping.auto_inserted_switch` 非空，topo_mapper.py:583-595）由 L2 SW 承载、env-file 数据面透明不表达，**不计入** matched link 数与一致性比较。

**评审修订（CP5 第二轮复审 R1/R2）**：
- **R1（MVP 边界声明，MAJOR）**：本 CR MVP = **单 TG + 单 DUT 直连**（`node2_dut1_tg1_linkN` 系列：link2/link3/link5）支持；**多节点（≥2 DUT）或含 SW/Mock/其他节点类型的拓扑在 S5 显式报错 `TOPO_NODE_COUNT_UNSUPPORTED`**，不产出缺节点坏产物。原因是当前 `_build_nodes` 只建 `tg1`/`dut1` 节点，映射多节点拓扑成功会产出 nodes 缺 `dutN` 但 links 引用 `dutN` 的坏产物（§8.5）。MVP 边界声明同步写入 §1 范围/§12 风险/§13 灰区 5；HLD §1.3 非目标由 host-orchestrator 补充「MVP 边界」说明（不改架构，仅声明范围，本次 LLD 层契约闭合）。
- **R2（确定性 vs generated_at，MAJOR）**：`metadata.generated_at` 是时间戳，天然随调用变化，与 NFR2「同输入 → 逐字节一致」冲突。修订为：**确定性校验忽略 `metadata.generated_at`**（比较时剔除该字段），或调用方通过 `_build_metadata(generated_at=...)` 注入固定值。单测 `test_export_deterministic_ignore_generated_at` 覆盖（§10）。

---

## 1. 目标

新建 `skills/topo-planning/src/exporter.py`，实现：
1. **映射结果 → env-file**：`metadata + port_mapping + nodes + links` 四段，严格满足 `${ENV.*}` 9 类占位符硬契约（HLD §7，ADR-09）；端口名加引号（YAML 1.1 整数陷阱，Gotcha #11）。metadata 含 `executable` 与 `degraded_reason` 字段，用于显式标注产物可执行性。
2. **命名规则**：默认 `topology/topology.yaml`；`--output` 支持描述性命名（如 `node2_dut1(nxp290-171_10.113.55.171)_tg1(trex_10.113.55.175)_link3.yml`）。
3. **link 数一致性校验**：实际**直连 matched** link 数必须 = **真实逻辑 link** 数（不含 Mock 端点、不含 L2 透传），不一致报 `LINK_COUNT_MISMATCH`。
4. **凭据禁止**：产物/env-file 不含密码（ADR-02）；管理地址仅 host；凭据走 ptm-atomic auth / devices.yaml `${ENV_VAR}`。
5. **结构化失败报告**：JSON + 人读双格式，列出缺设备/缺 TE 端口/链路约束不满足明细；CLI 支持 `--report-out <path>` 落盘（缺省 stdout），报告可作 CP7 验证证据（O-c）。
6. **IP 规划接入与缺 IP 状态化**：`ip_plan`（S6 输出 dict）渲染 `nodes.*.interfaces.*.ip/gw` 与 `nodes.dut1.next_hop`；缺 IP（`ip_plan=None` 或部分域缺 IP）默认显式报错，不允许静默空 IP；仅显式允许降级（dry-run / `--allow-degraded`）时产出 `metadata.executable=false` + `degraded_reason` 的降级产物，且不进入 case-execution 消费路径。
7. **MVP 边界（R1，本 CR 范围声明）**：本 CR 导出 MVP 仅支持 **单 TG + 单 DUT 直连** 拓扑（collection `node2_dut1_tg1_linkN` 系列：link2 / link3 / link5）。`_build_nodes` 在导出前校验逻辑拓扑节点分布：**恰好 1 个 TG + 恰好 1 个 DUT、且无 SW/Mock/PC 等其他节点类型**；超出该边界（≥2 DUT、含 SW、含 Mock 等）显式抛 `ExporterError(code="TOPO_NODE_COUNT_UNSUPPORTED", ...)` + 结构化节点分布明细，**不产出缺 `dutN` 节点的坏产物**。HLD §1.3 非目标补充该 MVP 边界（不改架构，仅声明范围）。

---

## 2. Requirements

### Functional

| ID | 需求 | 优先级 |
|----|------|:------:|
| FR1 | `export_env_file(mapping_result, logical_topo, pool, ip_plan=None, output_path=None) -> str` 生成 env-file（YAML 文本返回 + 可选落盘） | P0 |
| FR2 | env-file 顶层四段：`metadata` + `port_mapping` + `nodes` + `links`，结构对齐 `topology.yaml.example`（DA-006） | P0 |
| FR3 | `${ENV.*}` 9 类占位符全部可解析（HLD §7 契约表）：`tg.portN`/`tg.portN.ip`/`tg.portN.gw`/`dut.portN`/`dut.portN.ip`/`tg.url`/`dut.url`/`dut.next_hop`/`tg.ports[port1,port2]` | P0 |
| FR4 | 端口名加引号：`port_mapping` 与 `nodes.interfaces` 中所有物理端口名/`physical_port` 值输出为字符串（YAML 1.1 整数陷阱，Gotcha #11） | P0 |
| FR5 | 命名：默认 `topology/topology.yaml`；`--output` 描述性命名生成 | P0 |
| FR6 | link 数一致性校验：实际**直连 matched** link 数必须 = **真实逻辑 link** 数（不含 Mock 端点、不含 L2 透传）；不一致报 `LINK_COUNT_MISMATCH` 错误 | P0 |
| FR7 | 凭据禁止（ADR-02）：env-file 不输出任何 `password`/`api_key`/`token` 字段；管理地址仅 host（`nodes.dut1.host`） | P0 |
| FR8 | 结构化失败报告：`build_failure_report(mapping_result, logical_topo, reason=None) -> dict`，含缺设备/缺 TE 端口/链路约束不满足明细 | P0 |
| FR9 | 报告输出：`--report-out <path>` 落盘 JSON + 人读；缺省 stdout | P0 |
| FR10 | IP 规划接入：`ip_plan`（S6 输出 dict，以 `ipam.ip_plan_to_dict` 为权威 schema）渲染到 `nodes.*.interfaces.*.ip/gw` 与 `nodes.dut1.next_hop`；**缺 IP 状态化**——`ip_plan=None` 或部分接口缺 IP 时默认抛 `ExporterError`（`IP_PLAN_REQUIRED` / `IP_PLAN_INCOMPLETE`），不允许静默空 IP；仅显式允许降级（dry-run / `--allow-degraded`）时产出 `metadata.executable=false` + `degraded_reason` 的降级产物，且该产物不进入 case-execution 消费路径（`resolve_env_refs` 对空 IP 必抛 ValueError，op_mapper.py:1123-1125） | P0 |
| FR11 | 失败不产出 env-file：映射失败时只产出结构化报告，不生成部分 env-file | P0 |

### Non-Functional

| ID | 需求 |
|----|------|
| NFR1 | 产物通过 case-execution `load_env_file` 校验（`port_mapping`/`nodes`/`links` 三顶层字段，DA-006） |
| NFR2 | 确定性：同输入同映射同 IP 规划 → 逐字节一致 env-file（`yaml.safe_dump(sort_keys=False)`）。`metadata.generated_at` 为时间戳，**移出确定性比较范围**：确定性校验忽略该字段（剔除后比较），或调用方经 `_build_metadata(generated_at=...)` 注入固定值（R2） |
| NFR3 | 无凭据：导出路径全程不接收/不输出密码字段 |
| NFR4 | 幂等：`--output` 覆盖已有文件（原子写），不产生重复备份 |
| NFR5 | 独立可测：不依赖 S6 IPAM（`ip_plan` 为外部输入），可先于 S6 交付 |

---

## 3. 模块拆分与职责

模块内部函数划分（不拆子模块）：

| 函数 | 职责 |
|------|------|
| `export_env_file(mapping_result, logical_topo, pool, ip_plan=None, output_path=None, allow_degraded=False) -> str` | 主入口：MVP 边界校验 → IP 完备性检查 → 构建四段 → link 数校验 → 渲染 → 返回 YAML 文本（可选落盘） |
| `_build_metadata(mapping_result, logical_topo, executable=True, degraded_reason=None, generated_at=None) -> dict` | 元数据段（topology_id / schema_version / generated_at / env_name / executable / degraded_reason）；`generated_at` 可注入固定值（R2） |
| `_validate_mvp_topology(logical_topo) -> Optional[str]` | MVP 边界校验（R1）：节点 node_type 分布必须为恰好 1 TG + 1 DUT 且无 SW/Mock/PC；返回错误消息（None=通过）；超出边界 → `TOPO_NODE_COUNT_UNSUPPORTED` |
| `_build_port_mapping(mapping_result, logical_topo) -> dict` | 逻辑端口 → `{tg, dut}` 物理端口名（引号化） |
| `_build_nodes(mapping_result, logical_topo, pool, ip_plan) -> dict` | tg1/dut1 节点（trex_api_url / host / next_hop / interfaces.ip/gw）；**前置校验节点分布（R1），仅构建 1 TG + 1 DUT** |
| `_build_links(mapping_result) -> list` | links 列表（`{from, to, name}`） |
| `_check_link_count(mapping_result, logical_topo) -> Optional[str]` | link 数一致性校验（不一致返回错误消息） |
| `_quote_port(value) -> str` | 端口名强制字符串（Gotcha #11） |
| `build_failure_report(mapping_result, logical_topo, reason=None) -> dict` | 结构化失败报告（JSON 结构） |
| `render_failure_report(report: dict) -> str` | 人读渲染失败报告 |
| `write_report(report: dict, report_out: Optional[str]) -> str` | `--report-out` 落盘或 stdout |
| `_resolve_ip_plan(nodes, ip_plan, port_mapping) -> None` | IP 规划渲染（S6 输出，权威 schema = `ipam.ip_plan_to_dict`）；按物理端口反查逻辑端口键填充 ip/gw/next_hop；发现缺 IP 抛 `IP_PLAN_INCOMPLETE` |

CLI 入口（`main()`）与命名生成在 §6。

---

## 4. 代码结构与文件影响范围

### 新增文件

```
skills/topo-planning/src/exporter.py            # 核心导出模块（~400 行）
```

### 只读消费

```
skills/case-execution/templates/topology.yaml.example   # env-file 契约参考
skills/case-execution/scripts/case_runner.py            # load_env_file / resolve_env_refs 校验语义
skills/topo-config/src/topo_mapper.py                   # MappingResult（S4 输出，含 LinkMapping.auto_inserted_switch 透传标记）
skills/topo-config/src/physical_pool.py                 # PhysicalPool（设备/接口查询）
skills/topo-config/src/ipam.py                          # ip_plan_to_dict 权威 schema（ipam.py:222-235，IPAssignment 字段）
skills/topo-planning/src/exporter.py#IP 规划段            # S6 在 exporter.py 内实现 compute_ip_plan（S6 实现后由同文件接入 ip_plan）
```

### 不修改

无。全部为新增文件 + 只读消费。

---

## 5. 数据模型与持久化设计

### 5.1 env-file 输出结构（硬契约，对齐 topology.yaml.example）

```yaml
metadata:
  topology_id: node2_dut1_tg1_link3
  schema_version: "1.0"
  generated_at: "2026-08-05T00:00:00+08:00"
  env_name: "case-IPv4策略路由"
  executable: true                 # true=可执行；false=仅 dry-run/结构校验，禁止进入 case-execution
  degraded_reason: null            # executable=false 时必填，说明降级原因（如 ip_plan_missing）

port_mapping:
  port1:
    tg: "1/1/1"        # 物理端口名，引号（Gotcha #11）
    dut: "eth0"
  port2:
    tg: "1/1/2"
    dut: "eth1"
  # ...

nodes:
  tg1:
    trex_api_url: "http://10.113.52.253:8450"   # ${ENV.tg.url}
    interfaces:
      port1:
        ip: "198.18.1.2/24"    # ${ENV.tg.port1.ip}
        gw: "198.18.1.254"     # ${ENV.tg.port1.gw}
      port2:
        ip: "198.18.2.2/24"
        gw: "198.18.2.254"
  dut1:
    host: "https://10.113.55.51"   # ${ENV.dut.url}
    next_hop: "198.18.2.254"       # ${ENV.dut.next_hop}
    interfaces:
      port1:
        id: "if-001"
        interface_kind: "physical"
        ip: "198.18.1.1/24"        # ${ENV.dut.port1.ip}
      port2:
        id: "if-002"
        interface_kind: "physical"
        ip: "198.18.2.1/24"

links:
  - {from: "tg1.port1", to: "dut1.port1", name: "link1"}
  - {from: "tg1.port2", to: "dut1.port2", name: "link2"}
```

**降级产物（`executable=false`）**：当 `ip_plan=None` 且调用方显式允许降级（dry-run / `--allow-degraded`）时，产出结构与上图一致但：`metadata.executable=false`、`metadata.degraded_reason="ip_plan_missing：S6 IP 规划未交付，IP/gw/next_hop 字段未填充；产物仅可作拓扑结构 dry-run/校验，禁止作为 case-execution env-file 执行"`，`nodes.*.interfaces.*.ip/gw` 与 `nodes.dut1.next_hop` 留空。该产物虽能通过 `load_env_file` 结构校验（case_runner.py:695-751 只校验 port_mapping/nodes/links 三顶层字段，metadata 为附加字段不参与校验），但**不可执行**：S7 skill / S8 编排不得将其传给 case_runner；即使误传，`resolve_env_refs` 对空 IP 也会抛 `ValueError`（op_mapper.py:1123-1125），从消费端兜底拒绝。

> **IP 值示意说明**：§5.1 上图 IP 值仅示意结构（沿用 `topology.yaml.example` 的 DUT=.1/TG=.2/gw=.254 形态）。实际 IP 值由 S6 `compute_ip_plan` 按 §8.3 权威 schema 填充：DUT 为该广播域网关时取 `gateway.ip`（.254），DUT 为 endpoint 时取 `endpoints[].ip`（.1 起）；`next_hop` 取 DUT 出口广播域网关 .254 去掩码。两处示例的 IP 具体值可能因广播域结构不同而异，非冲突。

### 5.2 9 类占位符 → 解析路径映射（HLD §7）

| 占位符 | 输出路径 |
|---|---|
| `${ENV.tg.port1}` / `${ENV.dut.port1}` | `port_mapping.port1.tg` / `.dut` |
| `${ENV.tg.port1.ip}` / `.gw` | `nodes.tg1.interfaces.port1.ip` / `.gw` |
| `${ENV.dut.port1.ip}` | `nodes.dut1.interfaces.port1.ip` |
| `${ENV.tg.url}` | `nodes.tg1.trex_api_url` |
| `${ENV.dut.url}` | `nodes.dut1.host` |
| `${ENV.dut.next_hop}` | `nodes.dut1.next_hop` |
| `${ENV.tg.ports[port1,port2]}` | `[port_mapping.port1.tg, port_mapping.port2.tg]`（聚合） |

### 5.3 结构化失败报告 schema

```json
{
  "status": "failed",
  "topology_id": "node2_dut1_tg1_link3",
  "reason": "mapping_failed",
  "summary": "2 个设备未映射，1 条链路约束不满足",
  "issues": [
    {"type": "missing_device", "logical_node": "dut2",
     "detail": "无满足 node_type=DUT + hardware_platform=HG3250 的空闲物理设备"},
    {"type": "missing_te_port", "logical_link": "link1",
     "detail": "物理池无 TE 端口（speed_class=TE）"},
    {"type": "link_constraint_unsatisfied", "logical_link": "link3",
     "detail": "include-at-least-one TE 约束 0 条满足"}
  ],
  "link_count": {"logical": 3, "actual": 2}
}
```

### 5.4 持久化

- env-file：默认 `topology/topology.yaml`（目标项目目录），`--output` 自定义；原子写（`os.replace`，对齐 `save_ledger` 模式）。
- 报告：`--report-out <path>` 落盘 JSON + `.txt`（人读）双文件；缺省 stdout。
- 无其他持久化。

---

## 6. API / Interface 设计

### 6.1 `export_env_file(mapping_result, logical_topo, pool, ip_plan=None, output_path=None, allow_degraded=False) -> str`

**主入口**。返回 env-file YAML 文本；`output_path` 非 None 时原子落盘并返回路径。`allow_degraded=False`（默认）时缺 IP 直接报错，防止静默空 IP 产物流入 case-execution。

**流程**：
1. **MVP 边界校验（R1）**：`err = _validate_mvp_topology(logical_topo)`；非 None → 抛 `ExporterError(code="TOPO_NODE_COUNT_UNSUPPORTED", ...)`（含节点分布明细），不进入构建。当前 `_build_nodes` 只建 `tg1`/`dut1`，多节点/SW 拓扑直接构建会产出缺 `dutN` 但 links 引用 `dutN` 的坏产物。
2. `err = _check_link_count(mapping_result, logical_topo)`；非 None → 抛 `ExporterError(code="LINK_COUNT_MISMATCH", ...)`。
3. **IP 规划完备性检查**：
   - `ip_plan is None`：若 `allow_degraded=False`（默认）→ 抛 `ExporterError(code="IP_PLAN_REQUIRED", ...)`；若 `allow_degraded=True`（dry-run / `--allow-degraded`）→ 进入降级构建（metadata.executable=false + degraded_reason，IP 字段留空）。
   - `ip_plan` 非 None 但存在接口 IP 缺失（`_resolve_ip_plan` 无法为某逻辑端口找到 IP/gw）→ 抛 `ExporterError(code="IP_PLAN_INCOMPLETE", ...)`，不产出部分 IP 的 env-file。
4. 构建四段（metadata / port_mapping / nodes / links）。
5. `yaml.safe_dump(env_dict, sort_keys=False, allow_unicode=True)`。
6. `output_path` 非 None → 原子写；返回路径。

**异常**：
- `ExporterError("TOPO_NODE_COUNT_UNSUPPORTED", "拓扑 'node3_dut2_tg1_link3' 超出 MVP 边界：节点分布 {TG:1, DUT:2}，MVP 仅支持单 TG + 单 DUT 直连（node2_dut1_tg1_linkN 系列）；不产出缺 dutN 节点的坏产物")` — 多节点/含 SW/Mock 拓扑（R1）
- `ExporterError("LINK_COUNT_MISMATCH", "真实逻辑链路 3 条（不含 Mock/L2 透传），实际直连映射 2 条")` — link 数不一致（FR6）
- `ExporterError("IP_PLAN_REQUIRED", "ip_plan 缺失：缺 IP 默认报错，不允许静默空 IP；dry-run/--allow-degraded 可产出 executable=false 降级产物")` — 缺 IP 默认拒绝（FR10）
- `ExporterError("IP_PLAN_INCOMPLETE", "ip_plan 缺少接口 port2 的 IP 分配")` — ip_plan 部分缺 IP（FR10）
- `ExporterError("OUTPUT_WRITE_FAILED", ...)` — 落盘失败

### 6.2 `build_failure_report(mapping_result, logical_topo, reason=None) -> dict`

从 `MappingResult` 汇总失败明细（§5.3 schema）。问题类型枚举：
- `missing_device`：`device_mappings[].status == 'unmapped'` → 取 `reason`（含 hardware_platform/TE 缺口的结构化消息）
- `missing_te_port`：`reason` 含「缺 TE 端口」或 per-link TE 约束未满足
- `link_constraint_unsatisfied`：`link_mappings[].status != 'matched'` + include-at-least-one 0 条满足

### 6.3 `render_failure_report(report: dict) -> str`

人读渲染（表格/列表），每行含问题类型 + 逻辑对象 + detail。

### 6.4 `write_report(report: dict, report_out: Optional[str]) -> str`

- `report_out=None` → 打印 JSON + 人读到 stdout，返回 "stdout"。
- `report_out=<path>` → 写 `<path>.json` + `<path>.txt`（人读），返回路径。幂等覆盖。

### 6.5 CLI 入口 `main()`

```
exporter.py --mapping <MappingResult.json> --logical <topology.yaml> \
            --pool <physical_pool.yaml> --ledger <physical_ledger.yaml> \
            [--ip-plan <ip_plan.json>] [--output <path>] [--report-out <path>] \
            [--allow-degraded]
```

- 加载 `MappingResult`（S4 输出 JSON）、逻辑 topo（S1 parser）、物理池（PhysicalPool）、IP 规划（S6）。
- 映射成功（`status in (matched, partial)`）→ `export_env_file`；缺 IP 且未传 `--allow-degraded` → 抛 `IP_PLAN_REQUIRED`（默认报错，不产出静默空 IP）；`--allow-degraded` 或 `--dry-run` 场景 → 产出 `executable=false` 降级产物（仅供结构 dry-run，不进入 case-execution 消费）。
- 映射失败 → `build_failure_report` + `write_report`；退出码 1。

### 6.6 命名生成 `_descriptive_name(mapping_result) -> str`

`--output` 缺省且映射成功时，生成描述性名（HLD §4.5）：

```
node2_dut1(nxp290-171_10.113.55.171)_tg1(trex_10.113.55.175)_link3.yml
# 规则：<逻辑拓扑名>_<dutN(型号_管理host)>_<tgN(trex_管理host)>_<link数>.yml
```

仅作为 `--output` 缺省的增强命名；显式 `--output` 优先。

---

## 7. 核心处理流程

### 7.1 映射成功 → env-file

```
export_env_file(mapping_result, logical_topo, pool, ip_plan, output_path, allow_degraded)
 │
 ├─ 1. MVP 边界校验（R1）：_validate_mvp_topology(logical_topo)
 │       # 节点 node_type 分布必须 = 恰好 1 TG + 1 DUT，且无 SW/Mock/PC
 │       # 超出边界 → ExporterError(TOPO_NODE_COUNT_UNSUPPORTED) + 节点分布明细
 │       #   （当前 _build_nodes 只建 tg1/dut1；多节点直接构建会产出缺 dutN 坏产物）
 │
 ├─ 2. link 数校验：实际直连 matched link 数 vs 真实逻辑 link 数（不含 Mock、不含 L2 透传）
 │       ├─ 不一致 → ExporterError(LINK_COUNT_MISMATCH)
 │       └─ 一致 → 继续
 │
 ├─ 3. IP 规划完备性检查
 │       ├─ ip_plan=None 且 not allow_degraded → ExporterError(IP_PLAN_REQUIRED)
 │       ├─ ip_plan=None 且 allow_degraded → 标记 degraded（executable=false + degraded_reason）
 │       ├─ ip_plan 非 None 但接口缺 IP → ExporterError(IP_PLAN_INCOMPLETE)
 │       └─ 完备 → 继续
 │
 ├─ 4. metadata = _build_metadata(..., executable, degraded_reason, generated_at=None)
 │       # topology_id / schema_version / generated_at / env_name / executable / degraded_reason
 │       # generated_at 可注入固定值；确定性校验忽略该字段（R2）
 │
 ├─ 5. port_mapping = _build_port_mapping(...)
 │       # 逻辑端口 portN → {tg: 物理端口名, dut: 物理端口名}
 │       # 物理端口名 _quote_port() 强制字符串（Gotcha #11）
 │       # 顺序按 logical_topo 接口顺序稳定（port1, port2, ...）
 │
 ├─ 6. nodes = _build_nodes(mapping_result, logical_topo, pool, ip_plan, port_mapping)
 │       # tg1: trex_api_url = TG api_server（池 management.api_server）
 │       #       interfaces.portN.ip/gw ← ip_plan[domain].endpoints[].ip + gateway
 │       # dut1: host = 池 management.host（https:// 前缀对齐 topology.yaml.example）
 │       #       next_hop = DUT 出口广播域网关 IP（gateway.ip 去掩码，.254）
 │       #       interfaces.portN.ip ← ip_plan[domain].gateway.ip（DUT 为网关）或 endpoints[].ip（DUT 为 endpoint）
 │       # ip_plan=None 且 allow_degraded → ip/gw/next_hop 留空 + metadata.executable=false（降级产物）
 │       # 节点数/类型已由步骤1校验（恰好 1 TG + 1 DUT，R1）
 │
 ├─ 7. links = _build_links(mapping_result)
 │       # 直连 matched link → {from: "<tg|dutN>.<portN>", to: "...", name: link_id}
 │       # 自动透传 link（auto_inserted_switch 非空）→ 跳过（SW 不进入 env-file links，数据面透明）
 │
 ├─ 8. env_dict = {metadata, port_mapping, nodes, links}
 │       yaml.safe_dump(..., sort_keys=False, allow_unicode=True)
 │
 └─ 9. output_path 非 None → 原子写；返回 YAML 文本
```

### 7.2 映射失败 → 结构化报告

```
export 失败入口（CLI / S7 skill 调用）
 │
 ├─ 1. report = build_failure_report(mapping_result, logical_topo, reason)
 │       # 遍历 device_mappings/link_mappings 汇总 3 类问题
 │
 ├─ 2. 若 report_out: write_report(report, report_out)  # JSON + 人读落盘
 │
 └─ 3. stdout 打印人读摘要；退出码 1
```

### 错误路径

| 步骤 | 失败条件 | 行为 |
|------|---------|------|
| 1 | 逻辑拓扑节点分布非「恰好 1 TG + 1 DUT」或含 SW/Mock/PC（R1） | `ExporterError(TOPO_NODE_COUNT_UNSUPPORTED)`（含节点分布明细；不产出缺 dutN 坏产物） |
| 2 | 实际直连 matched link 数 ≠ 真实逻辑 link 数（不含 Mock/L2 透传） | `ExporterError(LINK_COUNT_MISMATCH)` |
| 3 | `ip_plan=None` 且非降级模式 | `ExporterError(IP_PLAN_REQUIRED)`（不产出静默空 IP） |
| 3 | `ip_plan` 非 None 但某逻辑端口缺 IP/gw | `ExporterError(IP_PLAN_INCOMPLETE)`（不产出部分 IP） |
| 5 | 逻辑端口无匹配物理端口（映射不完整） | `ExporterError(PORT_MAPPING_INCOMPLETE)` |
| 6 | TG 无 `api_server` | `trex_api_url` 留空 + warning（ADR-05 fallback 到 devices.yaml，由 case_runner 降级） |
| 6 | DUT 无 `host` | `host` 留空 + warning（ADR-05 fallback） |
| 6 | 降级模式（ip_plan=None + allow_degraded） | ip/gw/next_hop 留空；`metadata.executable=false` + `degraded_reason`；产物不进入 case-execution 消费路径 |
| 8 | 输出目录不可写 | `ExporterError(OUTPUT_WRITE_FAILED)` |

---

## 8. 技术细节

### 8.1 端口名引号化（Gotcha #11）

```python
def _quote_port(value) -> str:
    """端口名强制为字符串（YAML 1.1 整数陷阱：'1/1/1' 不得被解析为整数）"""
    if value is None:
        return ""
    s = str(value)
    # YAML 1.1 中纯数字字符串（如 '8080'）必须加引号；带 '/' 的端口名天然是字符串
    return s
```

`yaml.safe_dump` 对含 `/` 的值自动加引号；对纯数字字符串（如 `8080`）需保证类型为 str（safe_dump 对 str 类型自动加引号）。实现时统一确保 `port_mapping.*.tg/dut`、`physical_port` 输出均为 str 类型。

### 8.2 link 数一致性校验

```python
def _check_link_count(mapping_result, logical_topo) -> Optional[str]:
    # 真实逻辑 link：不含 Mock 端点
    logical_physical = {
        lid for lid, link in logical_topo.links.items()
        if not _has_mock_endpoint(link, logical_topo)
    }
    # L2 透传 link（TopoMapper._try_l2_pass_through，LinkMapping.auto_inserted_switch 非空）
    # 由 L2 SW 承载、env-file 数据面透明不表达 → 从一致性比较中排除
    pass_through_ids = {
        lm.logical_link for lm in mapping_result.link_mappings
        if lm.auto_inserted_switch
    }
    logical_physical -= pass_through_ids

    # 实际直连 matched link：status == 'matched' 且非透传
    actual_direct = {
        lm.logical_link for lm in mapping_result.link_mappings
        if lm.status == 'matched' and not lm.auto_inserted_switch
    }

    if actual_direct != logical_physical:
        return (f"link count mismatch: logical={len(logical_physical)} "
                f"(excl Mock/L2-pass-through), actual_direct={len(actual_direct)}")
    return None
```

> 一致性口径（评审 S5-B 修正）：**真实逻辑 link = 不含 Mock 端点、不含 L2 透传的逻辑 link 数；实际 = 直连 matched link 数**。含 Mock 端点的 link 在 S4 中标 unmapped（虚拟不参与物理连通），不计入；L2 透传 link（`auto_inserted_switch` 非空）由自动插入的 SW 承载，env-file 的 `links` 段不表达（数据面透明），同样不计入。两条口径同时排除，避免「透传路径被误计为额外物理链路」或「env-file 缺 link 但校验通过」的假一致性。

### 8.3 IP 规划渲染（S6 接口，权威 schema = `ipam.ip_plan_to_dict`）

**权威 schema（评审 S5-C 修正）**：`ip_plan` 以 `skills/topo-config/src/ipam.py:222-235` 的 `ip_plan_to_dict` 输出为准，S6 `compute_ip_plan` 的返回必须与该结构逐字段一致，exporter 不做二次适配。权威输出示例：

```python
{
  "bd-1": {
    "subnet": "198.18.1.0/24",
    "gateway": {"device": "dut1", "port": "1/1/1", "ip": "198.18.1.254/24"},
    "endpoints": [
      {"device": "tg1", "port": "1/1/2", "ip": "198.18.1.1/24", "gateway": "198.18.1.254", "trex_port": "0/0/1"}
    ]
  }
}
```

字段语义（源自 `IPAM.plan`，ipam.py:108-141）：
- `gateway.ip` = `{gw_ip}/{prefixlen}`，其中 `gw_ip = str(subnet[-2])`（/24 即 `.254`，带掩码）；`gateway` 指示该广播域的三层网关设备与端口。
- `endpoints[].ip` = `{subnet[i+1]}/{prefixlen}`（从 `.1` 顺序分配，带掩码）；`endpoints[].gateway` = `gw_ip`（无掩码）；`endpoints[].trex_port` 可选。
- **endpoints 不含 gateway 设备**（DUT 作网关时在 `gateway` 字段，不在 endpoints）。每域可用 IP = `subnet.num_addresses - 2`，端点 + 1 网关 ≤ 可用，否则 `ValueError`。
- endpoint/gateway 的 `port` 为**物理端口名**（来自 broadcast_domains 的 phys_port_map），需经 S4 `port_mapping`（逻辑端口 → 物理端口）反查 env-file 的逻辑端口键（port1/port2）。

**DUT 接口 IP 来源（明确）**：对每个广播域，按物理端口反查逻辑端口键后：
- DUT 为广播域网关（`gateway.device == 'dut1'`）→ DUT 该接口 IP = `gateway.ip`（如 `198.18.1.254/24`）。
- DUT 为广播域 endpoint（如 DUT-DUT 直连 L3 域一端为 endpoint）→ DUT 该接口 IP = `endpoints[]` 中 `device == 'dut1'` 的 `.ip`。
- TG 接口 IP = `endpoints[]` 中 `device == 'tg1'` 的 `.ip`；TG 网关 `gw` = 该 endpoint 的 `.gateway`（无掩码）。

**`next_hop` 语义对齐（评审 S5-C）**：`nodes.dut1.next_hop` = DUT 业务出口广播域的 `gateway.ip.split('/')[0]`（即 ipam 分配的网关 `.254`，与 HLD §7 `${ENV.dut.next_hop}` → `nodes.dut1.next_hop` 契约一致）。**不得**取 TG endpoint 的 IP（如 `.1`/`.2`），避免 next_hop 指向 TG 端口 IP 与 DUT gateway 语义冲突。

**`_resolve_ip_plan(nodes, ip_plan, port_mapping)`**：遍历 `ip_plan` 各域 → 按 `(device, physical_port)` 匹配 gateway/endpoints → 经 `port_mapping` 反查逻辑端口键 → 填 `nodes.<device>.interfaces.<portN>.ip/.gw`；DUT 侧按上规则填 `nodes.dut1.next_hop`。若某逻辑端口（DUT/TG 接口）在 `ip_plan` 中找不到对应 IP 分配 → 抛 `ExporterError("IP_PLAN_INCOMPLETE")`（不允许静默空 IP）。

**降级渲染**：`ip_plan=None` 且 `allow_degraded=True` 时，`_resolve_ip_plan` 跳过，IP 字段留空，由 `_build_metadata(executable=False, degraded_reason=...)` 标记降级产物。

### 8.4 凭据禁止（ADR-02）

- `_build_nodes` 只读 `management.host` / `management.api_server`，**不读** `management.password` / `web_password` / `username`。
- `build_failure_report` 不包含任何凭据字段。
- env-file 校验（S10）：产物全文正则断言不含 `password:` / `token:` / `api_key:` / `secret:`。

### 8.5 MVP 边界校验（R1）

**问题**：当前 `_build_nodes` 只构建 `tg1`/`dut1` 两个节点。若逻辑拓扑为多节点（≥2 DUT）或含 SW/Mock/PC（如 `node3_dut2_tg1_link3`、`node4_dut2_tg1_sw2_link7`），直接导出会产出 `nodes` 段缺 `dutN`/`swN`/`mockN`，但 `links` 段仍引用这些节点的**坏产物**——下游 `resolve_env_refs` / `load_env_file` 能解析出引用但不存在的节点，属于「假成功」产物。

**校验算法**：

```python
def _validate_mvp_topology(logical_topo) -> Optional[str]:
    """MVP 边界校验：逻辑拓扑节点分布必须为恰好 1 TG + 1 DUT，且无 SW/Mock/PC 等其他节点类型。

    返回错误消息（None=通过）。超出边界 → TOPO_NODE_COUNT_UNSUPPORTED。
    """
    node_type_counts: Dict[str, int] = {}
    for node_id, node in (logical_topo.nodes or {}).items():
        nt = getattr(node, 'node_type', None) or 'UNKNOWN'
        node_type_counts[nt] = node_type_counts.get(nt, 0) + 1

    # MVP 边界：恰好 1 个 TG + 恰好 1 个 DUT，且无其他 node_type
    if node_type_counts.get('TG') == 1 and node_type_counts.get('DUT') == 1 \
            and set(node_type_counts) == {'TG', 'DUT'}:
        return None

    topo_id = getattr(getattr(logical_topo, 'metadata', None), 'topology_id', None)
    return (
        f"topology '{topo_id}' 超出 MVP 边界：节点分布 {node_type_counts}。"
        f"MVP 仅支持单 TG + 单 DUT 直连（node2_dut1_tg1_linkN 系列）。"
        f"多 DUT（≥2）、含 SW / Mock / PC 节点的拓扑不产出 env-file，"
        f"避免 nodes 缺 {sorted(set(node_type_counts) - {'TG', 'DUT'})} 的坏产物。"
    )
```

**调用点**：`export_env_file` 第 1 步（§6.1 / §7.1），在任何构建动作前。CLI（§6.5）映射成功后、进入 `export_env_file` 前可先行提示，但以主入口校验为强制门。

**错误码**：`ExporterError(code="TOPO_NODE_COUNT_UNSUPPORTED", details=<返回消息>)`。失败报告（§5.3）在 `reason="topo_node_count_unsupported"` 时在 `issues[]` 增加 `{"type": "topo_node_count_unsupported", "detail": <分布明细>}`，保证错误可结构化消费。

**HLD 边界声明**：HLD §1.3 非目标由 host-orchestrator 补充「本 CR MVP 仅覆盖单 TG + 单 DUT 直连；多节点/SW 拓扑不支持」；本 LLD 不改架构，仅声明导出层范围。

---

## 9. 安全与性能设计

### 安全

- **凭据禁止（ADR-02）**：导出路径不接收/不输出密码；管理地址仅 host；凭据走 ptm-atomic auth / devices.yaml `${ENV_VAR}`（HLD §4.5）。
- **YAML 安全**：`yaml.safe_dump`，不序列化任意对象。
- **路径安全**：`--output`/`--report-out` 为显式路径，不做路径拼接注入。

### 性能

- 单次导出 < 50ms（纯内存 dict 构建 + YAML dump）。
- 失败报告构建 O(映射结果规模)。
- env-file 大小量级 < 20KB（与 topology.yaml.example 一致）。

---

## 10. 测试设计

### 单元测试（`skills/topo-planning/tests/test_exporter.py`）

| 测试用例 | 覆盖要点 | 预期 |
|---------|---------|------|
| `test_export_full_env_file` | 完整映射 → env-file 四段齐全 | `port_mapping`/`nodes`/`links` 三顶层字段存在，结构对齐 example |
| `test_export_placeholder_contract` | 对产物跑 `resolve_env_refs` 9 类占位符 | 全部解析成功（S10 集成 case_runner） |
| `test_export_port_quoted` | 物理端口名 `1/1/1`、`8080` | YAML 中为字符串（引号），非整数 |
| `test_export_link_count_ok` | 实际=逻辑 | 通过 |
| `test_export_link_count_mismatch` | 实际 2，逻辑 3 | `ExporterError(LINK_COUNT_MISMATCH)` |
| `test_export_no_credentials` | 产物全文扫描 | 无 `password`/`token`/`api_key`/`secret`（ADR-02） |
| `test_export_ip_plan_rendered` | 传 ip_plan（权威 schema） | `nodes.tg1.interfaces.port1.ip/gw`、`nodes.dut1.next_hop`（网关 .254）正确 |
| `test_export_ip_plan_missing_rejected` | ip_plan=None（默认） | `ExporterError(IP_PLAN_REQUIRED)`，不产出静默空 IP env-file |
| `test_export_ip_plan_degraded_flag` | ip_plan=None + allow_degraded | 产出 `metadata.executable=false` + `degraded_reason`，IP 留空；产物仍可被 `load_env_file` 加载但不进入执行路径 |
| `test_export_ip_plan_incomplete` | ip_plan 缺某接口 IP | `ExporterError(IP_PLAN_INCOMPLETE)`，不产出部分 IP 产物 |
| `test_export_ip_plan_authoritative_schema` | ip_plan 对齐 `ipam.ip_plan_to_dict` | DUT 接口 IP 取 gateway.ip、endpoint 不含 gateway 设备、next_hop=gateway.ip 去掩码 |
| `test_export_tg_api_server` | TG api_server | `nodes.tg1.trex_api_url` 正确 |
| `test_export_dut_host` | DUT host | `nodes.dut1.host` 正确（https:// 前缀） |
| `test_multinode_topology_rejected` | 多 DUT 拓扑（如 `node3_dut2_tg1_link3`，2 DUT） | `ExporterError(TOPO_NODE_COUNT_UNSUPPORTED)`，错误含节点分布明细，不产出 env-file（R1） |
| `test_sw_topology_rejected` | 含 SW 拓扑（如 `node4_dut2_tg1_sw2_link7`，2 SW） | `ExporterError(TOPO_NODE_COUNT_UNSUPPORTED)`，不产出坏产物（R1） |
| `test_mock_topology_rejected` | 含 Mock 节点拓扑（如 `node3_dut1_mock1_link2`） | `ExporterError(TOPO_NODE_COUNT_UNSUPPORTED)`，不产出坏产物（R1） |
| `test_mvp_single_dut_passes` | 单 TG + 单 DUT 直连（`node2_dut1_tg1_link3`） | 通过 MVP 边界校验，正常导出（R1 边界内） |
| `test_export_deterministic_ignore_generated_at` | 同输入两次导出，`generated_at` 不同 | 剔除 `metadata.generated_at` 后两次产物逐字节一致（R2） |
| `test_export_deterministic_inject_generated_at` | 同输入两次导出，`_build_metadata(generated_at=固定值)` | 含 `generated_at` 的产物逐字节一致（R2） |
| `test_failure_report_missing_device` | 设备 unmapped | `issues[].type=missing_device` + detail |
| `test_failure_report_missing_te` | 缺 TE 端口 | `issues[].type=missing_te_port` |
| `test_failure_report_link_constraint` | include-at-least-one 0 条 | `issues[].type=link_constraint_unsatisfied` |
| `test_report_out_write` | `--report-out` | `<path>.json` + `<path>.txt` 落盘 |
| `test_output_descriptive_name` | `--output` 缺省 | 描述性命名含逻辑拓扑名/型号/host/link 数 |
| `test_output_custom_path` | 显式 `--output` | 自定义路径生效 |
| `test_load_env_file_accepts_export` | 产物经 case_runner `load_env_file` | 校验通过（DA-006） |
| `test_mock_link_excluded` | 含 Mock 端点 link | link 数一致性按「不含 Mock」口径计算 |
| `test_pass_through_link_excluded` | 含 L2 透传 link（auto_inserted_switch 非空） | 透传 link 不计入 matched 数与逻辑数，一致性按「真实逻辑 link = 直连 matched」口径通过 |

### Fixture 需求

- `tests/fixtures/mapping_result_matched.json`：S4 输出的 matched MappingResult（fixture 池）。
- `tests/fixtures/mapping_result_failed.json`：unmapped 设备 + 缺 TE 端口 + link 约束不满足。
- `tests/fixtures/ip_plan_sample.json`：S6 IP 规划样例（对齐 `ipam.ip_plan_to_dict` 权威 schema，§8.3）。
- `tests/fixtures/logical_topo_single_dut.yaml`：单 TG + 单 DUT 直连逻辑拓扑（`node2_dut1_tg1_link3`，MVP 边界内）。
- `tests/fixtures/logical_topo_multidut.yaml`：多 DUT 逻辑拓扑（`node3_dut2_tg1_link3`）——R1 拒绝路径。
- `tests/fixtures/logical_topo_with_sw.yaml`：含 SW 逻辑拓扑（`node4_dut2_tg1_sw2_link7`）——R1 拒绝路径。
- 复用 P-1 fixture 池 + S4 fixture。

### 集成测试

- S10：`S4 match → S5 export_env_file → case_runner load_env_file + resolve_env_refs` 全链路（参考用例 `IPv4策略路由`，TOPO-01）。
- S10：失败路径 `S4 match(failed) → build_failure_report → write_report` → CP7 证据。

---

## 11. 实施步骤

| TASK-ID | 描述 | 预估 | 产物 |
|---------|------|:---:|------|
| TASK-S5-01 | `ExporterError` + 常量 + 模块骨架 | 30min | 骨架 |
| TASK-S5-02 | `_build_metadata` / `_build_links` | 30min | 简单段 |
| TASK-S5-03 | `_build_port_mapping`（端口引号化） | 1h | port_mapping |
| TASK-S5-04 | `_build_nodes`（tg1/dut1 + trex_api_url/host/next_hop + 接口 IP） | 1.5h | nodes |
| TASK-S5-04b | `_validate_mvp_topology` MVP 边界校验（R1：node_type 分布校验 + `TOPO_NODE_COUNT_UNSUPPORTED` 结构化错误） | 45min | 边界校验 |
| TASK-S5-05 | `_resolve_ip_plan`（S6 接口接入） | 1h | IP 渲染 |
| TASK-S5-06 | `_check_link_count` 一致性校验 | 45min | link 数校验 |
| TASK-S5-07 | `export_env_file` 主入口（四段组装 + safe_dump + 原子写） | 1h | 主导出 |
| TASK-S5-08 | `build_failure_report` + `render_failure_report` | 1.5h | 失败报告 |
| TASK-S5-09 | `write_report`（`--report-out` JSON+人读）+ CLI `main()` | 1h | 报告输出 |
| TASK-S5-10 | `_descriptive_name` 描述性命名 | 30min | 命名 |
| TASK-S5-11 | `_build_metadata` 支持 `generated_at` 注入（R2） | 20min | 确定性 |
| TASK-S5-12 | 单元测试（28 用例 + 6 fixture，含 R1 多节点/SW/Mock 拒绝 + R2 确定性） | 2.5h | 测试 |
| **合计** | | **~11.5h** | |

---

## 12. 风险、难点与预研建议

| 风险 | 级别 | 缓解 |
|------|:---:|------|
| 产物与 case-execution 解析契约偏差（如 DUT 接口 `id`/`interface_kind` 字段缺失） | 中 | 对齐 `topology.yaml.example`（R-F-028 `preconfigure_dut_interfaces` 消费）；S10 用 `load_env_file` 校验 |
| YAML 1.1 整数陷阱遗漏（纯数字端口名） | 中 | `_quote_port` 统一强制 str；S10 校验 `port_mapping` 端口名为字符串 |
| link 数一致性口径分歧（含 Mock 端点 / L2 透传） | 中 | §8.2 明确口径：真实逻辑 link（不含 Mock、不含透传）= 直连 matched 数；单测覆盖 Mock 与透传两种排除 |
| IP 规划缺失导致「假成功」env-file | 中 | 缺 IP 默认抛 `IP_PLAN_REQUIRED` / `IP_PLAN_INCOMPLETE`；仅显式降级（dry-run / `--allow-degraded`）产出 `executable=false` 降级产物且不进入 case-execution 消费路径（FR10） |
| **多节点/含 SW/Mock 拓扑产出坏产物（nodes 缺 dutN 但 links 引用 dutN）**（R1） | 高 | `_validate_mvp_topology` 前置校验节点 node_type 分布；超 MVP 边界（非「恰好 1 TG + 1 DUT」）显式抛 `TOPO_NODE_COUNT_UNSUPPORTED` + 分布明细，不构建（§8.5）；单测覆盖多 DUT/SW/Mock 三类拒绝路径 |
| TG/DUT 管理地址在池中为占位符 | 低 | 从 S3 UnifiedPool 读真实 host/api_server；占位符场景 warning + 提示填写 |

**预研建议**：
1. 用 `topology.yaml.example` 实测 case_runner `load_env_file` 对字段缺失的容错度（哪些缺失只是 warning）。
2. 与 S6 对齐 `ip_plan` 结构（§8.3），确保 exporter 无需二次适配。

---

## 13. 实现灰区与取舍记录

### 灰区 0：多节点/含 SW 拓扑的 env-file 处理（R1）

**问题**：collection.md 12 个拓扑中 6+ 个为多 DUT（`node3_dut2_*`、`node4_dut3_*`）或含 SW（`node4_dut2_*_sw*`）/ Mock（`node3_dut1_mock1_link2`、`node3_dut1_tg1_pppoe_link4`）。当前 `_build_nodes` 只建 `tg1`/`dut1`，对这类拓扑导出会产出缺节点的坏产物。

**选项**：
- A. **MVP 边界声明**：本 CR 只支持单 TG + 单 DUT 直连（`node2_dut1_tg1_linkN` 系列），多节点/SW/Mock 拓扑 S5 显式报 `TOPO_NODE_COUNT_UNSUPPORTED`，不产出坏产物。多节点 env-file 支持留待后续 CR。
- B. 扩展 `_build_nodes` 支持全部节点类型（多 DUT/SW/Mock 全量导出），本 CR 完成全部 12 拓扑导出。
- C. 只对多节点抛 warning 继续导出坏产物（不可接受）。

**决策**：选 A（R1，已定范围决策）。MVP = 单 TG + 单 DUT 直连；`_validate_mvp_topology` 校验 node_type 分布，超边界显式报错 + 结构化明细。理由：多节点 env-file 涉及 SW 端口 VLAN/广播域/IP 规划复杂组合（S6 IPAM 需按 SW 广播域展开），超出本 CR 核心场景「海光3250 + TE 链路」（单 DUT）范围；显式报错优于产出坏产物。HLD §1.3 非目标由 host-orchestrator 补充该 MVP 边界（不改架构，仅声明范围）。重访条件：后续 CR 扩展 `_build_nodes` 支持多节点/SW 时，移除该校验或按节点类型分级支持。

### 灰区 0b：`generated_at` 与逐字节确定性的冲突（R2）

**问题**：NFR2 声明「同输入 → 逐字节一致 env-file」，但 `metadata.generated_at` 是时间戳，每次调用不同，直接导致逐字节不一致。二者冲突。

**选项**：
- A. **`generated_at` 移出确定性比较范围**：确定性校验（单测）剔除 `metadata.generated_at` 后比较；NFR2 声明补充「剔除 generated_at」。真实产物仍带时间戳。
- B. `generated_at` 支持注入固定值（`_build_metadata(generated_at=...)`），调用方在确定性场景传固定值。
- C. 移除 `generated_at` 字段（丢失产物审计时间信息）。

**决策**：选 A + B。NFR2 明确「确定性校验忽略 `metadata.generated_at`（剔除后比较）」，同时 `_build_metadata` 支持 `generated_at` 参数注入固定值；单测双覆盖（`test_export_deterministic_ignore_generated_at` + `test_export_deterministic_inject_generated_at`）。`generated_at` 保留（审计用途），但不纳入确定性契约。重访条件：若下游要求产物含可复现时间，由调用方注入固定值。

### 灰区 1：link 数一致性口径（Mock 端点 + L2 透传）

**问题**：逻辑 topo 可能含 Mock 节点（虚拟），其 link 在 S4 中标 unmapped；也可能因无直连 cable 走 `TopoMapper._try_l2_pass_through` 自动透传（`LinkMapping.auto_inserted_switch` 非空，topo_mapper.py:583-595）。一致性校验如何处理这两类 link？

**选项**：
- A. 逻辑全部 link 数 = 实际 matched 数（Mock link 与透传路径都会导致误报/口径混乱）。
- B. 不含 Mock 端点、不含 L2 透传的逻辑 link 数 = 实际直连 matched 数（物理可表达口径）。
- C. 含 partial 也通过（宽松）。

**决策**：选 B（评审 S5-B 强制修正）。Mock 链路不参与物理连通，不计入；L2 透传 link 由自动插入的 SW 承载、env-file 的 `links` 段不表达（数据面透明），同样不计入。校验口径 = **真实逻辑 link（不含 Mock、不含透传）数 vs 实际直连 matched 数**，两侧同时排除透传，避免「env-file 缺 link 但校验通过」的假一致性。单测 `test_mock_link_excluded` 与 `test_pass_through_link_excluded` 覆盖。口径写入 §8.2 与文档，避免歧义。

### 灰区 2：`--output` 描述性命名缺省策略

**问题**：HLD 默认 `topology/topology.yaml`，同时提供描述性命名示例。何时用默认、何时用描述性名？

**选项**：
- A. 默认总是 `topology/topology.yaml`（稳定消费路径），描述性名仅显式 `--output` 时使用。
- B. 默认生成描述性名，`--output` 覆盖。
- C. 默认 `topology/topology.yaml`；描述性名作为可选参数（如 `--descriptive-name`）。

**决策**：选 A。case-execution 约定固定消费路径 `topology/topology.yaml`（CR-034 缺口2 统一环境文件路径），默认路径保证零适配；描述性命名作为 `--output` 显式值或 S7 skill 向导的可选增强，不改变默认消费路径。

### 灰区 3：失败报告人读格式载体

**问题**：`--report-out` 是单文件还是双格式？

**选项**：
- A. 单文件（仅 JSON）——机器可读，人读体验差。
- B. 双文件（`<path>.json` + `<path>.txt`）——JSON 机器 + 人读文本。
- C. 单文件 JSON + stdout 人读。

**决策**：选 B。`--report-out <path>` 落盘 `<path>.json`（机器，CP7 证据）+ `<path>.txt`（人读，排障）；缺省 stdout 同时打印两者摘要。对齐 HLD「JSON + 人读」（O-c）。

### 灰区 4：IP 规划接入时序（S5 与 S6 边界）

**问题**：IP 规划在 exporter 内是必选还是可选？`ip_plan=None` 时如何处理？

**选项**：
- A. 必选：exporter 内嵌 IPAM 调用（S5 依赖 S6，破坏 S5 独立交付）。
- B. 可选：`ip_plan` 为外部输入，None 时降级留空（S5 独立，S6 补充）。
- C. 纯必选，无降级。

**决策**：选 B + 缺 IP 状态化（评审 S5-A 强制修正）。`export_env_file(ip_plan=None)` 在 S6 未就绪时**默认抛 `ExporterError(IP_PLAN_REQUIRED)`**，不允许产出静默空 IP 的「可执行」env-file（`case-execution` 的 `resolve_env_refs` 对空 IP 抛 ValueError，op_mapper.py:1123-1125，会击穿零适配承诺）。仅显式允许降级（dry-run / `--allow-degraded`）时产出 `metadata.executable=false` + `degraded_reason` 的降级产物，仅作拓扑结构 dry-run/校验，**不进入 case-execution 消费路径**。`ip_plan` 非 None 但部分接口缺 IP → `ExporterError(IP_PLAN_INCOMPLETE)`。S6 交付 `compute_ip_plan` 后由 S7 CLI / S8 编排传入 `ip_plan`；S5 保持可独立测试（NFR5）。

---

## 14. 回滚与发布策略

### 回滚

- `exporter.py` 为新增文件，回滚即删除。
- 无对其他模块的修改；case-execution 不改。

### 发布

- S5 随 `topo-planning` skill（S7）统一安装，`exporter.py` 加入 PYTHONPATH（S8）。
- 依赖顺序：S5 依赖 S4（MappingResult 结构）；S6 依赖 S5（`ip_plan` 接入点）。
- 产物契约（9 类占位符）随 S5 固定，S10 用 `load_env_file` 回归校验，确保 case-execution 零适配。

---

## 15. Definition of Done（DoD）

- [ ] `export_env_file` 产出四段 env-file（metadata + port_mapping + nodes + links），结构对齐 `topology.yaml.example`；metadata 含 `executable`/`degraded_reason`（FR2）
- [ ] `${ENV.*}` 9 类占位符全部可解析（FR3，S10 用 case_runner 校验）
- [ ] 端口名引号化，无 YAML 1.1 整数陷阱（FR4，Gotcha #11）
- [ ] 命名：默认 `topology/topology.yaml`，`--output` 自定义 + 描述性命名（FR5）
- [ ] link 数一致性校验：不一致报 `LINK_COUNT_MISMATCH`（FR6，口径 = 真实逻辑 link（不含 Mock、不含 L2 透传）vs 直连 matched）
- [ ] 凭据禁止：产物无 `password`/`token`/`api_key`/`secret`（FR7，ADR-02）
- [ ] 结构化失败报告：3 类问题（缺设备/缺 TE 端口/链路约束不满足）+ 明细（FR8）
- [ ] `--report-out` 落盘 JSON + 人读（FR9，O-c）
- [ ] `ip_plan` 接入（权威 schema = `ipam.ip_plan_to_dict`）：渲染 IP/gw/next_hop（DUT 接口取 gateway.ip，next_hop=网关 .254 去掩码）；缺 IP 默认抛 `IP_PLAN_REQUIRED`/`IP_PLAN_INCOMPLETE`，仅显式降级产出 `executable=false` 降级产物且不进入 case-execution 消费路径（FR10）
- [ ] MVP 边界：`_validate_mvp_topology` 校验节点分布（恰好 1 TG + 1 DUT，无 SW/Mock/PC）；多节点/SW/Mock 拓扑抛 `TOPO_NODE_COUNT_UNSUPPORTED` + 结构化明细，不产出缺节点坏产物（FR7b，R1）
- [ ] 映射失败不产出 env-file，只产出结构化报告（FR11）
- [ ] 产物通过 `load_env_file` 校验（NFR1）
- [ ] 确定性输出（`sort_keys=False`，同输入同输出；`metadata.generated_at` 移出确定性比较范围或注入固定值，R2）
- [ ] 27 个单元测试全部通过
