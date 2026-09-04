---
doc_type: cp_check_result
id: CP5-CR037-S5-LLD-IMPLEMENTABILITY
story_id: "CR037-S5"
cr_id: "CR-037"
stage: CP5
checkpoint_type: auto
status: PASS
created_at: "2026-08-05"
owner: "meta-dev"
evidence_type: "full-lld"
evidence_path: "process/stories/STORY-CR037-S5-LLD.md"
---

# CP5-CR037-S5：设计证据可实现性自动预检

| 条目 | 结果 |
|------|:---:|
| lld_policy = full-lld 匹配 | PASS |
| 14 章节完整（0-15） | PASS |
| Goal 明确可量化 | PASS |
| Requirements 11 FR + 5 NFR 覆盖 | PASS |
| 模块拆分与职责清晰（export_env_file + 四段 builder + _check_link_count + failure report 系列 + CLI + 命名） | PASS |
| 代码结构与文件影响范围明确（新增 exporter.py，只读消费 case-execution/topo-config，无修改） | PASS |
| 数据模型定义完整（env-file 四段结构 + 9 类占位符映射 + 失败报告 schema + 持久化） | PASS |
| API/Interface 契约完整（export_env_file + build_failure_report + write_report + CLI + _descriptive_name） | PASS |
| 核心处理流程有流程图 + 映射成功/失败两分支 + 错误路径表 | PASS |
| 技术细节（端口引号化 Gotcha #11 + link 数一致性口径不含 Mock + IP 规划渲染 S6 接口 + 凭据禁止 ADR-02） | PASS |
| 安全与性能分析（无凭据 + YAML 安全 + 路径安全 + <50ms） | PASS |
| 测试设计（18 用例 + 3 fixture + 集成链路含 resolve_env_refs） | PASS |
| 实施步骤 11 TASK + 11h 预估 | PASS |
| 风险 5 项 + 缓解 | PASS |
| 实现灰区 4 项 + 取舍记录（link 口径/命名策略/报告双格式/IP 接入时序） | PASS |
| 回滚与发布策略 | PASS |
| DoD 13 项可验证 | PASS |
| 输出文件与 DEVELOPMENT-PLAN 一致（新增 skills/topo-planning/src/exporter.py） | PASS |
| 文件所有权无冲突（Wave 2 串行；S6 在 exporter.py#IP 规划段，属 S5 文件内后续追加） | PASS |
| 依赖 S5=[S4] precedence 满足 | PASS |
| tier="A" 合理 | PASS |
| open_items 无 | PASS |
| HLD §4.5 + §7 对齐（env-file 四段 + 9 类占位符 + 命名 + link 数一致性 + 凭据禁止 + 结构化失败报告 --report-out，O-c） | PASS |
| 产物契约硬性：端口名加引号（YAML 1.1 整数陷阱，Gotcha #11） | PASS |
| 失败报告 JSON + 人读双格式（O-c），--report-out 落盘 | PASS |
| lld-check: LLD Structure Check | OK |

## 开放项

无。

## 修订说明（2026-08-05 产出）

- **link 数一致性口径**：不含 Mock 端点的逻辑 link 数 = 实际 matched 数（Mock 链路不参与物理连通），写入 §8.2 与单测。
- **命名策略**：默认固定 `topology/topology.yaml`（case-execution CR-034 缺口2 统一消费路径），描述性命名仅 `--output` 显式值或 S7 向导可选增强。
- **IP 接入时序**：`ip_plan` 为外部输入，S6 未交付时 `ip_plan=None` 降级（IP 留空 + warning），保持 S5 独立交付与测试（NFR5）。
- **凭据禁止（ADR-02）**：导出路径不接收/不输出密码，产物正则断言无 `password`/`token`/`api_key`/`secret`。

## 修订说明 2（2026-08-05 NEEDS_REWORK 回修，评审 S5-A/B/C）

- **S5-A（BLOCKER）ip_plan=None 降级产物状态化**：`resolve_env_refs` 对空 IP 抛 ValueError（op_mapper.py:1123-1125），原「留空 + warning」会击穿 env-file 零适配。修订为：缺 IP 默认抛 `ExporterError(IP_PLAN_REQUIRED)`；仅显式降级（dry-run / `--allow-degraded`）产出 `metadata.executable=false` + `degraded_reason` 降级产物，且不进入 case-execution 消费路径；`ip_plan` 非 None 但部分缺 IP → `IP_PLAN_INCOMPLETE`。FR10/§6.1/§7.1/§7 错误路径/灰区4/测试/DoD 全部同步。
- **S5-B（BLOCKER）link 数一致性漏 L2 透传**：auto-inserted 透传 link（`LinkMapping.auto_inserted_switch` 非空，topo_mapper.py:583-595）不计入 matched 数与逻辑数；`LINK_COUNT_MISMATCH` 判定排除透传，只比较「真实逻辑 link（不含 Mock、不含透传）= 实际直连 matched」。§8.2/灰区1/单测同步。
- **S5-C（评审 D 组）ip_plan 权威 schema**：§8.3 明确以 `ipam.ip_plan_to_dict`（ipam.py:222-235）为准；DUT 接口 IP 来源明确（广播域 gateway → `gateway.ip`，DUT 为 endpoint → `endpoints[].ip`）；`next_hop` = DUT 出口广播域网关 `.254` 去掩码，不得取 TG IP；与 HLD §7 产物契约一致。
- **S5-D**：凭据禁止（ADR-02）+ `--report-out`（O-c）保持既有设计，无变更。
- lld-check 重跑：PASS。

## 修订说明 3（2026-08-05 CP5 第二轮复审 R1/R2）

- **R1（MAJOR）多节点/含 SW 拓扑 env-file 规则（已定范围决策）**：MVP = 单 TG + 单 DUT 直连（`node2_dut1_tg1_linkN` 系列）支持；多节点（≥2 DUT）/含 SW/Mock/PC 拓扑 `_validate_mvp_topology` 校验节点 node_type 分布，超边界显式抛 `ExporterError(TOPO_NODE_COUNT_UNSUPPORTED)` + 结构化明细，**不产出 nodes 缺 dutN 但 links 引用 dutN 的坏产物**。§1 目标 7 / §3 模块 / §6.1 主入口流程 / §7.1 流程 / §7 错误路径 / §8.5 技术细节 / §10 单测（`test_multinode_topology_rejected` / `test_sw_topology_rejected` / `test_mock_topology_rejected` / `test_mvp_single_dut_passes`）/ §11 TASK-S5-04b / §12 风险 / §13 灰区0 / §15 DoD 全部同步。HLD §1.3 非目标由 host-orchestrator 补充 MVP 边界说明（不改架构，仅声明范围）。
- **R2（MAJOR）NFR2 确定性 vs generated_at**：`metadata.generated_at` 移出确定性比较范围（剔除后比较），或 `_build_metadata(generated_at=...)` 注入固定值。§2 NFR2 / §3 `_build_metadata` 签名 / §7.1 步骤4 / §11 TASK-S5-11 / §13 灰区0b / §15 DoD 同步；单测 `test_export_deterministic_ignore_generated_at` + `test_export_deterministic_inject_generated_at`。
- 单元测试 22→28 用例（新增 R1 四例 + R2 两例）；fixture 3→6（新增单 DUT / 多 DUT / 含 SW 逻辑拓扑 fixture）。
- lld-check（v0.5）：**OK（PASS）**。
