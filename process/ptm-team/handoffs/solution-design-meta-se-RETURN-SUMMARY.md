# 交还摘要 — solution-design（meta-se）→ host-orchestrator

- CR：CR-038（PPPoE 链路规划能力，parent_cr=CR-037）
- 阶段：solution-design（HLD 草案）
- 委托：host-orchestrator 委托 meta-se
- 交还时间：2026-08-15
- 下一动作：host-orchestrator 汇总并发起 CP3 正式人工门禁（meta-se 不自行发起）

---

## 1. 推荐 HLD

- **推荐方案**：在 CR-037 topo 引擎上增量扩展（方案 A）——SW 显式映射（reserved 集互斥）+ PPPoE Server/Client 配置 + 环回 env-file 放行（1TG+1DUT+N SW）+ 多实例 TRex + interface_kind 透传。
- **适用条件**：physical_pool 含 SW 节点与 TG `trex_instances`；参考场景单 DUT 环回。
- **主要权衡**：复用 CR-037 映射/台账/产物/安装链路，最简增量；代价是需确定性协调「显式 SW vs 自动透传」与「exporter/pool_merge 多 Story 文件所有权」。
- **HLD 路径**：`docs/design/HLD-CR-038.md`

## 2. 备选方案

| 备选 | 说明 | When to switch |
|---|---|---|
| 方案 B 独立 PPPoE 规划器 | 新建独立模块与 topo 引擎并列 | PPPoE 能力需与 topo 引擎彻底解耦交付时 |
| 方案 C 固化单链路脚本 | 硬编码参考场景 | 已被用户边界排除，仅作范围回退记录（不推荐） |

## 3. 蓝图适用性

- **判定**：`required`（跨 4 能力域 + 数据归属 + 依赖方向，非单 Feature 小改）。
- **理由**：5 Epic/13 Story、SW 归属、PPPoE 数据源、多实例 TRex 归属、跨模块透传。
- **影响范围**：HLD §3 承接；Story/LLD 消费 BLUEPRINT 边界。
- **后续触发条件**：若出现多 DUT 拓扑或独立交付形态，需回退蓝图重新判定。

## 4. 蓝图产物路径

| 产物 | 路径 | 状态 |
|---|---|---|
| BLUEPRINT | `docs/design/BLUEPRINT-CR-038.md` | 已产出（required） |
| DOMAIN-MAP | `docs/design/DOMAIN-MAP-CR-038.md` | 已产出 |
| DEPENDENCY-MAP | `docs/design/DEPENDENCY-MAP-CR-038.md` | 已产出 |

（无 N/A / WAIVED，三项均适用。）

## 5. Architecture Gray Areas 处理结果

4 个灰区全部经 advisor table 收敛为 agent 默认确定性契约，**无新增需用户拍板决策**（CP2 已锁方向性决策）：

| 灰区 | 结论 |
|---|---|
| AGA-038-01 显式 SW vs 自动透传 | reserved 集互斥（explicit_sw_reserved 从透传候选池剔除） |
| AGA-038-02 多实例 TRex 契约 | pool 侧 trex_instances 为准按 name 合并 |
| AGA-038-03 PPPoE 数据源 + 命令族 | physical_pool `pppoe_server` 块 + 三厂商命令 |
| AGA-038-04 环回 env-file 放行 | 1TG+1DUT+N SW + links 校验 + interface_kind |

- 讨论日志：`process/discussions/CP3-HLD-DISCUSSION-LOG.md`
- 恢复点：`process/checks/CP3-DISCUSSION-CHECKPOINT.json`

## 6. Feature design triggers

| Feature | 适用性 | 目标输出路径 | 阻塞状态 |
|---|---|---|---|
| F-CR038-E1 SW 映射与台账 | required | `docs/features/cr038-sw-mapping/{DESIGN,TEST-PLAN,TASKS}.md` | CP3 通过后生成 |
| F-CR038-E2 PPPoE 配置 | required | `docs/features/cr038-pppoe-config/{DESIGN,TEST-PLAN,TASKS}.md` | CP3 通过后生成 |
| F-CR038-E3 环回 env-file | required | `docs/features/cr038-loopback-envfile/{DESIGN,TEST-PLAN,TASKS}.md` | CP3 通过后生成 |
| F-CR038-E4 SW 设备管理 | waived | N/A | — |
| F-CR038-E5 验证与集成 | waived | N/A | — |

## 7. Story 拆解（草案，CP3 通过后生成 STORY-*.md 卡片）

- **Story 数**：13（P0 7 / P1 6）
- **Wave 数**：4
  - W1 地基（6）：STORY-038-01/02/03/08/09/12（文件互斥并行）
  - W2 能力（3）：STORY-038-04/05/06
  - W3 透传（2）：STORY-038-07/13
  - W4 验证/集成（2）：STORY-038-10/11
- **TASK-ID**：以 Story 为粒度（STORY-038-NN），每 Story 内的 TASK 清单由 full-lld Story 在 LLD 阶段展开（沿用 CR-037 TASK-ID 规范）。
- **文件所有权**：见 `docs/design/DEPENDENCY-MAP-CR-038.md` §2（exporter.py / pool_merge.py 被多 Story 共享时通过 Wave 串行）。
- 完整计划：`process/DEVELOPMENT-PLAN-CR-038.yaml`（status=draft，pending_cp3）
- Feature 矩阵：`docs/design/FEATURE-DESIGN-MATRIX-CR-038.md`

## 8. 待决策项（供 CP3 Decision Brief）

CP3 无新增用户决策项（CP2 已锁定 7 项方向性决策；本轮灰区均为 agent 默认实现层决策）。CP3 确认范围：

| 确认点 | 类型 | 推荐 |
|---|---|---|
| HLD 推荐方案（扩展 topo 引擎，方案 A） | architecture | 采用方案 A |
| ADR-CR038-01~05 落地映射 | implementation | 接受 |

不授权项（沿用 CR-037/CP2）：真机下发（H3C telnet / NGFW web / trex 发流）、GE1_1~4 实例改动、凭据明文落盘。

## 9. CP3 自动预检

- 路径：`process/checks/CP3-HLD-CONSISTENCY.md`
- 结论：PASS（未豁免 FAIL 0 项）

## 10. 风险与未决项

- RA-038-001（ptm-atomic op 可用性）：OPEN，LLD 阶段核实（DQ-038-03 三选一闭环），non-blocking。
- RA-038-002（H3C 命令厂商差异）：OPEN，dry-run diff + 真机独立授权核对。
- 非 BLOCKING 缺失信息：无。

---

## 产物路径清单

- `docs/design/BLUEPRINT-CR-038.md`
- `docs/design/DOMAIN-MAP-CR-038.md`
- `docs/design/DEPENDENCY-MAP-CR-038.md`
- `docs/design/HLD-CR-038.md`
- `docs/design/ARCHITECTURE-DECISION-CR-038.md`
- `docs/design/FEATURE-DESIGN-MATRIX-CR-038.md`（draft）
- `process/DEVELOPMENT-PLAN-CR-038.yaml`（draft）
- `process/discussions/CP3-HLD-DISCUSSION-LOG.md`
- `process/checks/CP3-DISCUSSION-CHECKPOINT.json`
- `process/checks/CP3-HLD-CONSISTENCY.md`
