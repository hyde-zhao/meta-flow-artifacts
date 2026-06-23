# process/ 目录索引

本目录包含 ptm-team 工作流 `WF-PTM-TEAM-20260520-001` 的运行时产物，以及从源仓库 [ptm-tde](https://gitlab.com/ptm-team/ptm-tde) 引入的上游设计基线。

> 最后更新：2026-05-28（补建 ptm-tde 上游产物）

---

## ptm-team 运行时文件

| 文件 | 用途 | 状态 |
|------|------|------|
| `STATE.md` | 工作流运行时状态，记录当前阶段、Agent 生命周期、检查点和历史事件 | `story-execution` |
| `VALIDATION-ENV.yaml` | 验证环境配置，记录用户审批 `approval.confirmed` | active |

### checks/ — 自动检查结果

| 文件 | 所属 CR | 状态 |
|------|---------|------|
| `CP6-CR-20260520-001-scenario-discovery-CODING-DONE.md` | CR-20260520-001 | PASS |
| `CP7-CR-20260520-001-scenario-discovery-VERIFICATION-DONE.md` | CR-20260520-001 | PASS |
| `CP6-CR-20260521-001-factor-library-resource-CODING-DONE.md` | CR-20260521-001 | PASS |
| `CP7-CR-20260521-001-factor-library-resource-VERIFICATION-DONE.md` | CR-20260521-001 | PASS |
| `CP6-CR-20260521-001-topology-factor-separation-CODING-DONE.md` | CR-20260521-001 | PASS |
| `CP7-CR-20260521-001-topology-factor-separation-VERIFICATION-DONE.md` | CR-20260521-001 | PASS |

### handoffs/ — 子 Agent 交接文件

| 文件 | 目标 Agent | 状态 |
|------|------------|------|
| `CR-20260520-001-meta-dev-scenario-discovery.md` | meta-dev (dev-kong) | completed |
| `CR-20260520-001-meta-qa-scenario-discovery.md` | meta-qa | completed |
| `CR-20260521-001-meta-dev-A-topology-core.md` | meta-dev (Worker A) | completed |
| `CR-20260521-001-meta-dev-B-design-guards.md` | meta-dev (Worker B) | completed |
| `CR-20260521-001-meta-dev-C-docs-coverage.md` | meta-dev (Worker C) | completed |
| `CR-20260521-001-meta-qa-topology-factor-separation.md` | meta-qa | completed |

### changes/ — 变更请求

**ptm-team 当前工作流变更：**

| 文件 | 内容 | 状态 |
|------|------|------|
| `CR-20260520-001.md` | 改进场景发现再发现流程 + 操作路径建模 | closed |
| `CR-20260521-001.md` | 公共因子库资源 + 拓扑角色/因子分离重构 | implemented（CP7 PASS，待审批） |

**从 ptm-tde 引入的历史变更：**

| 文件 | 内容 | 状态 |
|------|------|------|
| `CR-003-ptm-tde.md` | 全球重新设计（CAE、知识链路、设计输出） | closed |
| `CR-004-ptm-tde.md` | 知识库/MCP 只读查询 | closed |
| `CR-005-ptm-tde.md` | HLD 细化与一致性修复 | closed |
| `CR-006-ptm-tde.md` | 工具分析表作为正式交付件 | closed |
| `CR-007-ptm-tde.md` | 格式同步（Markdown 一致性） | closed |
| `CR-008-ptm-tde.md` | Topology 场景组网建模 | closed |
| `CR-009-ptm-tde.md` | 运行目录、检查点与交付口径重构 | closed |

---

## 从 ptm-tde 引入的设计基线

> 来源：`/home/hyde/projects/ptm-tde/process/`
> 引入日期：2026-05-28
> 引入目的：补建 ptm-team 工作流缺失的 CP0-CP5 上游产物

### 核心设计文档

| 文件 | 版本 | 用途 |
|------|------|------|
| `REQUEST-ptm-tde.md` | v2.0 | 用户原始请求（从头重新设计 ptm-tde） |
| `USE-CASES-ptm-tde.md` | v2.2 | 8 个使用场景（安装、导入、场景确认、MFQ 分析、设计、交付、需求变更、问题单分析） |
| `REQUIREMENTS-ptm-tde.md` | v6.1 | 28 条结构化需求，覆盖安装、场景、MFQ 分析、设计、交付、检索、变更 |
| `HLD-ptm-tde.md` | v7.1 | 高层设计，12 章完整设计（含 Topology Modeling） |
| `ARCHITECTURE-DECISION-ptm-tde.md` | v6.0 | 6 条架构决策记录（ADR-1 ~ ADR-6） |

### 计划与状态

| 文件 | 版本 | 用途 |
|------|------|------|
| `STORY-BACKLOG-ptm-tde.md` | v1.1 | 9 个 Story、6 个 Wave |
| `STORY-STATUS-ptm-tde.md` | — | Story 状态跟踪 |
| `DEVELOPMENT-PLAN-ptm-tde.yaml` | — | 开发计划 |
| `TEST-STRATEGY-ptm-tde.md` | v2.0 | 测试策略 |

### 流程日志

| 文件 | 用途 |
|------|------|
| `CLARIFICATION-LOG-ptm-tde.md` | 需求澄清日志（15 轮澄清） |

### 领域参考

| 文件 | 用途 |
|------|------|
| `REQUIREMENTS-FIREWALL-ATOMIC.md` | 防火墙原子特性需求（领域输入材料） |
| `USE-CASES-FIREWALL-ATOMIC.md` | 防火墙原子特性场景（领域输入材料） |

### stories/ — Story 与 LLD

9 个 Story + 9 个 LLD，全部从 ptm-tde 引入：

| Story | 文件 | LLD |
|-------|------|-----|
| Story 01 | `story-01-ptm-tde.md` | `story-01-lld-ptm-tde.md` |
| Story 02 | `story-02-ptm-tde.md` | `story-02-lld-ptm-tde.md` |
| Story 03 | `story-03-ptm-tde.md` | `story-03-lld-ptm-tde.md` |
| Story 04 | `story-04-ptm-tde.md` | `story-04-lld-ptm-tde.md` |
| Story 05 | `story-05-ptm-tde.md` | `story-05-lld-ptm-tde.md` |
| Story 06 | `story-06-ptm-tde.md` | `story-06-lld-ptm-tde.md` |
| Story 07 | `story-07-ptm-tde.md` | `story-07-lld-ptm-tde.md` |
| Story 08 | `story-08-ptm-tde.md` | `story-08-lld-ptm-tde.md` |
| Story 09 | `story-09-ptm-tde.md` | `story-09-lld-ptm-tde.md` |

---

## checkpoints/ — 人工确认稿

从 ptm-tde 引入的人工检查点留档：

| 文件 | 对应检查点 | 内容 |
|------|------------|------|
| `CP2-USE-CASES-ptm-tde.md` | CP2 需求基线门（场景确认） | ptm-tde v1.1 场景基线确认记录 |
| `CP3-HLD-REVIEW-ptm-tde.md` | CP3 HLD 架构评审门 | ptm-tde v7.1 HLD 检查点，含 8 项重点评审项 |

---

## 已知差异

当前 ptm-team 的 CR-20260521-001（拓扑角色/因子分离重构）引入了 `topology_role_refs`、`topology_bindings` 等新契约，这些在 ptm-tde 的 HLD v7.1 中尚未体现。后续需评估是否需要通过 CR 同步设计基线。
