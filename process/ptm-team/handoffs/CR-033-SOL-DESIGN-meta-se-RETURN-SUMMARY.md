---
handoff_id: "CR-033-SOL-DESIGN-meta-se-RETURN"
cr_id: "CR-033"
phase: "solution-design"
canonical_role: "meta-se"
status: "completed"
completed_at: "2026-07-28T12:30:00+08:00"
context_ref: "process/changes/CR-033.md"
delegated_interaction: true
---

# CR-033 solution-design 交还摘要

## 推荐 HLD

**推荐方案**：方案 A（op_mapper 模式，case_runner.py 直接 import op_mapper 模块）。

**适用条件**：case-execution skill 与 policy-route-execution skill 同仓库（ptm-team canonical）；ptm-atomic CLI 已安装且在 PATH 中；devices.yaml 含 firewall.host 和 tg.api_server。

**主要权衡**：进程内耦合（case_runner 与 op_mapper）vs 性能好/复用/易测试。备选方案 B（subprocess）在跨仓库部署时切换。

**HLD 路径**：`docs/design/HLD-PTM-TE-EXEC.md`

## 备选方案

| 备选 | 适用条件 | When to switch |
|---|---|---|
| 方案 B（subprocess） | case_runner 需独立部署到无 op_mapper 的环境 | 跨仓库部署 |
| 方案 C（框架） | 用户撤销 R-C-002 约束 | 用户明确要求引入 pytest/robot |

## 蓝图适用性

**判定**：required（跨 TG建模/case-execution/执行改进 3 Feature，存在数据归属、依赖方向、共享能力等跨 Feature 边界问题）

**影响范围**：3 Feature / 8 Epic / 16 Story

**后续触发条件**：op_mapper TREX_API_URL 注入复杂度增加时，FE-EX-01 重访

## 蓝图产物路径

| 产物 | 路径 | 状态 |
|---|---|---|
| BLUEPRINT | `docs/design/BLUEPRINT-PTM-TE-EXEC.md` | draft |
| DOMAIN-MAP | `docs/design/DOMAIN-MAP-PTM-TE-EXEC.md` | draft |
| DEPENDENCY-MAP | `docs/design/DEPENDENCY-MAP-PTM-TE-EXEC.md` | draft |
| HLD | `docs/design/HLD-PTM-TE-EXEC.md` | draft |
| ARCHITECTURE-DECISION | `docs/design/ARCHITECTURE-DECISION-PTM-TE-EXEC.md` | draft |
| FEATURE-DESIGN-MATRIX | `docs/design/FEATURE-DESIGN-MATRIX-PTM-TE-EXEC.md` | draft |

## Architecture Gray Areas 处理结果

| 灰区 ID | 问题 | 推荐方案 | 用户确认 | 状态 |
|---|---|---|---|---|
| AGA-01 | case-execution 与 op_mapper 集成方式 | A（直接 import） | agent 默认处理 | resolved-by-recommendation |
| AGA-02 | TG 设备数据归属 | A（devices.yaml 元数据 + topology yaml 接口） | **已确认**（AGAQ-01=A，2026-07-28T12:30:00+08:00） | resolved |
| AGA-03 | frontmatter 16 列冗余字段 | C（摘要 + case_runner 忽略冗余列） | agent 默认处理 | resolved-by-recommendation |
| AGA-04 | extract_payload 抽象位置 | A（放在 case_runner.py） | agent 默认处理 | resolved-by-recommendation |

- **灰区数**：4
- **用户确认条数**：1（AGAQ-01=A，已确认）
- **ADR 数**：8（ADR-01..08，ADR-08 用户已确认）
- **discussion log**：`process/discussions/CP3-HLD-DISCUSSION-LOG-CR033.md`
- **checkpoint**：`process/checks/CP3-DISCUSSION-CHECKPOINT-CR033.json`

## Feature design triggers

| Feature | 是否需要 implementation-design | 触发原因 | 目标输出路径 | 阻塞状态 |
|---|---|---|---|---|
| FE-EX-01（TG 设备建模） | waived | 单 Story 小改，HLD 已承载边界 | N/A | 不阻塞 |
| FE-EX-02（case-execution 引擎） | required | cross-module-contract / data-model / concurrency / shared-story-boundary | docs/features/case-execution/DESIGN.md + TEST-PLAN.md + TASKS.md | CP3 通过后生成 |
| FE-EX-03（规则固化与执行改进） | required | migration / rollback / cross-module | docs/features/rule-fix-and-improvement/DESIGN.md + TEST-PLAN.md + TASKS.md | CP3 通过后生成 |

## 决策项

| DQ ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣/影响/风险/回退 |
|---|---|---|---|---|---|
| DQ-CP3-01 | architecture | ADR-08 TG 设备数据归属（AGAQ-01） | A：devices.yaml 元数据 + topology yaml 接口 | B：devices.yaml 含接口 | **已确认 A**（2026-07-28T12:30:00+08:00）。case_runner 需 --topology-yaml 参数 |
| DQ-CP3-02 | implementation | ADR-04 fw_logout op 暴露验证 | 安装前验证 + 降级 session 清理 | 强制 ptm-atomic 升级 | 推荐 DQ-02 降级方案；备选超出范围。回退：降级为 session 文件清理 |
| DQ-CP3-03 | scope | 24 用例 known_issue 标注完整性 | 纳入 R-F-021 同步检查 | 单独 CR | 推荐同步整改；备选增加协调成本。回退：只做 ARP 预热，known_issue 进 BACKLOG |

## CP3 自动预检

- **路径**：`process/checks/CP3-CR033.result.json`
- **结论**：PASS（12 items 全部 PASS）
- **未豁免 FAIL 数量**：0
- **验证命令**：`meta-flow cp result-check --result process/checks/CP3-CR033.result.json --project-root .` → OK

## 关键取舍

1. **import vs subprocess（AGA-01）**：选 import，性能好但进程内耦合
2. **TG 数据归属（AGA-02）**：选 devices.yaml 元数据 + topology yaml 接口，与 firewall 块对称但需读两文件
3. **不引入框架（R-C-002）**：借鉴 pytest/robot 理念但不引入，保持轻量
4. **ARP 预热双重保障（SGA-04=C）**：规则 + 引擎双重，防重装回退
5. **fw_logout 降级（DQ-02）**：op_mapper 映射 + 降级 session 清理，保证可用性

## DEVELOPMENT-PLAN

- **路径**：`process/DEVELOPMENT-PLAN-CR-033.yaml`
- **Story 数**：16
- **Wave 数**：4
- **lld_policy 分布**：8 full-lld + 8 technical-note
- **文件所有权冲突**：无

## 风险

| 风险 ID | 风险 | 缓解 |
|---|---|---|
| RISK-CR033-CROSS-REPO | 跨仓库回填不一致 | install.py 验证 + op_mapper validate |
| RISK-CR033-DEVICE-WRITE | --execute 触发设备写操作 | dry-run 默认门 + --execute 授权 |
| RISK-CR033-MIGRATION-REGRESSION | exec_v4.py 迁移后行为不一致 | dry-run 校验 + 对比测试 |
| RA-004 | fw_logout 未暴露 | 安装前验证 + 降级 |
| RA-008 | 重装后 ARP 预热规则被绕过 | 双重保障（规则+引擎） |

## 未决项

| 未决项 | 状态 | 阻塞 |
|---|---|---|
| AGAQ-01（TG 数据归属） | **RESOLVED**（A，2026-07-28T12:30:00+08:00） | 不阻塞 CP3 |
| O-02（fw_logout 暴露） | pending install verification | 不阻塞 CP3，ST-EX-07 实现时验证 |
| O-03（known_issue 标注） | pending migration | 不阻塞 CP3，ST-EX-13 整改时检查 |

## CP3 Decision Brief 输入

供 host-orchestrator 发起 CP3 人工确认：

1. **推荐 HLD**：方案 A（op_mapper 模式，case_runner import op_mapper）
2. **核心 ADR**：8 条（ADR-01 import / ADR-02 dry-run 门 / ADR-03 exec_v4 废弃 / ADR-04 fw_logout 降级 / ADR-05 TREX_API_URL 注入 / ADR-06 ARP 预热双重 / ADR-07 编号正则 / ADR-08 TG 数据归属）
3. **架构灰区**：4 个（AGA-01..04），3 个 agent 默认处理，1 个待用户确认（AGAQ-01）
4. **待决策项**：3 个（DQ-CP3-01/02/03）
5. **风险**：5 个（RISK-CR033-CROSS-REPO/DEVICE-WRITE/MIGRATION-REGRESSION + RA-004/008）
6. **Story 计划**：16 Story / 4 Wave / 8 full-lld + 8 technical-note
7. **Feature 设计触发**：FE-EX-02 required + FE-EX-03 required（CP3 后生成 DESIGN/TEST-PLAN/TASKS）

## Evidence Refs

- `process/handoffs/CR-033-SOLUTION-DESIGN-meta-se.md`（handoff）
- `process/changes/CR-033.md`（CR 完整范围）
- `docs/product/REQUIREMENTS-PTM-TE-EXEC.md`（26 功能需求）
- `docs/product/STORY-MAP-PTM-TE-EXEC.md`（16 Story 候选）
- `docs/product/SCENARIOS-PTM-TE-EXEC.yaml`（25 验证场景）
- `docs/design/BLUEPRINT-PTM-TE-EXEC.md`（蓝图）
- `docs/design/DOMAIN-MAP-PTM-TE-EXEC.md`（领域图）
- `docs/design/DEPENDENCY-MAP-PTM-TE-EXEC.md`（依赖图）
- `docs/design/HLD-PTM-TE-EXEC.md`（HLD）
- `docs/design/ARCHITECTURE-DECISION-PTM-TE-EXEC.md`（ADR）
- `docs/design/FEATURE-DESIGN-MATRIX-PTM-TE-EXEC.md`（Feature 矩阵）
- `process/DEVELOPMENT-PLAN-CR-033.yaml`（Story 计划）
- `process/discussions/CP3-HLD-DISCUSSION-LOG-CR033.md`（灰区讨论日志）
- `process/checks/CP3-DISCUSSION-CHECKPOINT-CR033.json`（灰区恢复点）
- `process/checks/CP3-CR033.result.json`（CP3 自动检查）
- `process/state/READ-EXPANSION-LEDGER.ndjson`（读取扩展记录）

## N/A / WAIVED 项

| 项 | 原因 | 影响范围 | 后续触发条件 |
|---|---|---|---|
| FE-EX-01 Feature 设计 | waived，单 Story 小改 | ST-EX-01 用 technical-note，ST-EX-03 用 full-lld | op_mapper TREX_API_URL 注入复杂度增加时重访 |
