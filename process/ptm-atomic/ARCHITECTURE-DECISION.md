---
version: "1.0"
status: "confirmed"
complexity: "standard"
confirmed: true
confirmed_by: "user-via-CP5-ALL-STORIES-LLD-BATCH"
confirmed_at: "2026-05-18T16:47:38+0800"
created_at: "2026-05-18T14:35:00+0800"
owner: "meta-se"
source_hld: "process/HLD.md"
selected_option: "Option B - schema-controlled native delivery"
agent_execution:
  agent_type: "default"
  acting_role: "meta-se"
  native_custom_agent_loaded: false
---

# Architecture Decision：atomic-ops 防火墙安装、配置与验证契约

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-05-18 | meta-se acting_role on default agent | 基于 CP3 approved 的 HLD v1.1 冻结 story-planning 阶段 ADR、平台边界、Story 拆分依据与 CP4 人工确认点。 |
| 1.1 | 2026-05-18 | meta-po | CP5 全量 LLD 批量人工确认中，用户接受 U-001 默认建议：以 CP3 approved + CR-003 + CP5 approval 作为 ADR 等价确认，并回填 `confirmed=true`。 |

## Agent/Skill 组合方案

本阶段只产出 Story planning 产物，不启动或模拟 `meta-dev` / `meta-qa`，不生成 `STORY-*-LLD.md`。

| 编排对象 | 使用方式 | 输出 | 边界 |
|---|---|---|---|
| meta-se | 当前执行角色，负责把 HLD v1.1 转成 ADR、Story backlog、开发计划和 Story 卡 | `process/ARCHITECTURE-DECISION.md`、`process/STORY-BACKLOG.md`、`process/DEVELOPMENT-PLAN.yaml`、Story 卡 | CP4 人工确认前停止，不进入 LLD |
| story-manager | 生成 Story 卡、生命周期状态和 `process/STORY-STATUS.md` | Story 卡三件套、状态汇总 | 不替代 meta-dev 写 LLD |
| dependency-mapper | 建立 Story 依赖、依赖类型和文件所有权 | `depends_on`、`dependency_type`、关键路径 | Story 草案稳定后使用 |
| wave-planner | 区分 LLD 并行、开发并行和验证并行 | Wave、`parallel_policy`、首批 LLD 建议 | Wave 不是硬门，仍以 DAG 和 gate 为准 |
| dag-validator | 校验 `DEVELOPMENT-PLAN.yaml` 中 Story DAG 无环、无无效引用 | CP4 checklist 证据 | 不对未来代码做测试 |
| checkpoint-manager | 生成 CP4 自动预检结构 | `process/checks/CP4-STORY-DAG-PARALLEL-SAFETY.md` | 不生成人工审查稿，交给 meta-po |

## 平台适配差异

本项目是 production 项目，交付面以 `README.md` 为准，不使用 `delivery/` 作为默认交付目录。

| 平台 / 目录 | 决策 | 依据 | Story 影响 |
|---|---|---|---|
| `atoms/` | 承载防火墙 atom 契约，不复制 `.input/` 源码或运行时资产 | README §2/§7；HLD §3 | STORY-002、STORY-003、STORY-004 |
| `schemas/` | 冻结 `schema_version="1.1"` 并显式声明字段族 | HLD §5 schema 扩展决策下限 | STORY-001 |
| `packages/` | 只作为 package 过滤视图，保存 `operations` op_id，不复制 atom | README §3/§10 | STORY-002、STORY-003、STORY-004 |
| `docs/` | 解释字段、安全边界、命名、用户手册，不保存凭据或日志 | README §10/§11；HLD §8 | STORY-001、STORY-006 |
| `src/atomic_ops/` | 维持 sync/list/show/packages/validate，只读本地缓存，不新增真实设备动作命令 | README §6/§10；HLD ADR-2 | STORY-005 |
| `scripts/` | 仓库级静态检查入口，不连接设备、不访问网络 | README §2；HLD §8 | STORY-005 |
| `pyproject.toml` / `uv.lock` | 仅当实现阶段确需依赖变化才修改；当前 Story 计划不要求新增依赖 | AGENTS uv 约束 | 当前 CP4 仅列为未来影响面，不修改 |
| `.input/` | 只读参考资料，禁止复制源码、env、日志、凭据或运行时资产 | R-C-008、R-C-009 | 所有 Story 的 forbidden 文件 |
| `delivery/` | N/A，不作为本项目交付出口 | production 输出隔离原则 | CP4 检查禁止作为正向产物 |

## 设计决策记录

### ADR-001：采用 schema v1.1 受控扩展

| 字段 | 内容 |
|---|---|
| 决策 | 冻结 `schema_version="1.1"`，新增字段族 `risk`、`credential_ref`、`session_ref`、`state_ref`、`gate`、`verification`、`batch`。 |
| 状态 | accepted-by-CP3, pending-CP4-story-plan |
| 理由 | HLD v1.1 已接受方案 B；现有 schema v1 使用 `additionalProperties: false`，新增字段必须进入 schema 与字段参考。 |
| 约束 | 字段只表达契约和校验元数据，不表达真实 executor；现有 v1.0 atom 不强制迁移。 |
| 落地 Story | STORY-001 |

### ADR-002：CLI 保持离线只读，不新增真实设备动作命令

| 字段 | 内容 |
|---|---|
| 决策 | CLI 命令集保持 `sync/list/show/packages/validate`；不新增 `run/execute/apply/configure` 等真实设备动作命令。 |
| 状态 | accepted-by-CP3 |
| 理由 | README 明确当前 CLI 同步和本地查询/校验；真实设备执行超出 CP2/CP3 范围。 |
| 约束 | `src/atomic_ops/cli.py` 的命令清单不得新增真实动作命令；脚本只做静态检查。 |
| 落地 Story | STORY-005 |

### ADR-003：登录状态只表达不含敏感载荷的引用

| 字段 | 内容 |
|---|---|
| 决策 | `session_ref` / `state_ref` 仅作为 atom 返回契约和外部编排上下文引用，不写入 CLI `_metadata.json`，不由 CLI 判定真实会话有效性。 |
| 状态 | accepted-by-CP3 |
| 理由 | 满足 UC-07/UC-08，同时避免原始密码、完整 token、cookie 或真实设备地址落盘。 |
| 约束 | 示例只能使用 `<session-ref>`、`<state-ref>` 等占位符；敏感明文字段与 `credential_ref` 互斥。 |
| 落地 Story | STORY-001、STORY-002、STORY-005 |

### ADR-004：capacity 首批覆盖 10 个配置域，按域拆 TASK-ID

| 字段 | 内容 |
|---|---|
| 决策 | 首批覆盖 `interface`、`object`、`acl_policy`、`policy_route`、`static_route`、`nat`、`bandwidth`、`black_white_list`、`ssl_vpn`、`dynamic_routing` 10 个配置域。 |
| 状态 | accepted-by-CP3, refined-in-CP4 |
| 理由 | R-F-018 要求至少覆盖 HLD 指定 10 个配置域；为控制 Story 数量，将其保留在一个 Story 中但拆分每域 TASK-ID。 |
| 约束 | 每个域必须列明配置 atom、验证 atom 和参数契约；不得直接搬运 `.input/capacity` 实现。 |
| 落地 Story | STORY-003 |

### ADR-005：多设备批次配置独立 Story

| 字段 | 内容 |
|---|---|
| 决策 | 多防火墙设备配置作为独立 Story，不隐藏在单设备 capacity Story 内。 |
| 状态 | accepted-by-CP3, refined-in-CP4 |
| 理由 | 批次并发、失败隔离、幂等、逐设备结果和验证汇总具有独立风险面。 |
| 约束 | 必须包含 `device_inventory_ref`、`device_selector`、`batch_ref`、`batch.max_concurrency` 默认 1 最大 5、高风险默认 1、`idempotency_key`、`per_device_results[]`、`failed_devices[]`、`verification_summary_ref`。 |
| 落地 Story | STORY-004 |

### ADR-006：验证失败只诊断和人工处理，不自动回滚

| 字段 | 内容 |
|---|---|
| 决策 | 安装/配置验证失败输出错误类别、诊断引用和人工处理信号，不默认执行自动回滚。 |
| 状态 | accepted-by-CP2-and-CP3 |
| 理由 | R-C-012 已排除自动回滚；HLD 风险 R5 明确自动回滚需新 CR。 |
| 约束 | atom、docs、scripts 不得把 rollback/revert 写为默认自动动作。 |
| 落地 Story | STORY-002、STORY-003、STORY-004、STORY-005、STORY-006 |

## 设计确认点（需人工确认）

| ID | 确认点 | 推荐值 | 备选值 | 切换条件 | 影响 |
|---|---|---|---|---|---|
| CP4-Q1 | Story 总数 | 6 个 Story | 拆成 8+ Story | 若 CP4 认为 capacity 10 域无法在单 Story 中保持可实现性 | 影响 Wave 和首批 LLD 队列 |
| CP4-Q2 | 首批 LLD 设计批次 | STORY-001、STORY-002、STORY-003 | STORY-001、STORY-003、STORY-004 | 若用户优先多设备批次而非安装登录链路 | 不影响开发门控，CP4 通过前不得启动 LLD |
| CP4-Q3 | capacity 10 域落地方式 | 单 Story + 每域 TASK-ID | 每 2-3 域一个 Story | 若单 Story 预计修改 atom 数超过 20 或 package 超过 4 个 | 影响 Story 粒度和并行度 |
| CP4-Q4 | schema version 表达 | `"1.1"` | `"1.1.0"` | 若实现阶段决定沿用 schema 当前 semver 正则 | 需同步 schema、docs、示例和校验脚本 |
| CP4-Q5 | CLI 命令边界 | 不新增真实动作命令 | 新增只读 explain/inspect 类命令 | 只有用户明确要求查询增强且不触达设备 | 影响 `src/atomic_ops/cli.py` |

## 变更记录

| 日期 | 变更 | 来源 |
|---|---|---|
| 2026-05-18 | CP3 接受方案 B，进入 story-planning。 | `checkpoints/CP3-HLD-REVIEW.md` |
| 2026-05-18 | CP4 将 HLD 粗估 5 个 Story 拆为 6 个可调度 Story，并独立多设备批次配置。 | `process/HLD.md` §12、CP4 拆分规则 |
