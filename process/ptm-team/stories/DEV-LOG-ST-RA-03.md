# DEV-LOG — ST-RA-03: 改进输入治理（improvement-tracker CA/PA 侧）

| 时间 | 操作 | 说明 |
|---|---|---|
| 2026-07-16T16:00Z | 就绪检查 | LLD 就绪（`process/stories/STORY-RA-03-LLD.md` full-lld），Feature DESIGN 可用，依赖`ST-RA-05.3-ANALYZE` 分析报告草案可用，文件所有权无冲突 |
| 2026-07-16T16:10Z | 创建目录与模板 | 创建 `skills/improvement-tracker/` 目录，写入 `templates/capa-proposal.yaml`（CA/PA 草案 schema）和 `templates/approved-input.yaml`（Approved Input schema） |
| 2026-07-16T16:20Z | 写入 SKILL.md §3 | CA/PA 治理侧：§3.1 草案生成（前置校验+去重+必填字段）、§3.2 批准状态机（draft→approved/rejected 终态不可回退）、§3.3 Approved Input 生成（门控 4 条件+不可变标记）、§3.4 消费者映射（4 合法 agent→pending-consumer，非法→blocked） |
| 2026-07-16T16:25Z | SKILL.md 占位符 | §4/§5 写入占位符，明确标注"由 ST-RA-04/ST-RA-06.3-TRACK 追加"；frontmatter shared_writers 列出三个 Story |
| 2026-07-16T16:30Z | Agent 集成 | 修改 `agents/ptm-tse.md`：skills 列表追加 improvement-tracker，新增改进治理职责（#10-12），新增改进治理流程（3 步骤），检查点追加 3 条新 Gate；source_feature 追加 FEAT-RA-IMPROVEMENT |
| 2026-07-16T16:35Z | 索引更新 | 修改 `skills/README.md`：ptm-tse 章节追加 reverse-analysis 和 improvement-tracker 索引 |
| 2026-07-16T16:40Z | CP6 自检 | 写入 `process/checks/CP6-ST-RA-03-improvement-governance-CODING-DONE.md`，PASS |

## 实现摘要

### 创建的文件

| 文件 | 行数 | 说明 |
|------|------|------|
| `skills/improvement-tracker/SKILL.md` | 355 | Skill 主文件：frontmatter + §1 输入契约 + §2 数据模型 + §3 CA/PA 治理（4 子节）+ §4/§5 占位符 + Gotchas（10 条） |
| `skills/improvement-tracker/templates/capa-proposal.yaml` | 182 | CA/PA Proposal 模板：完整 schema 定义（15 字段）+ 约束汇总（8 条）+ 2 个完整示例 |
| `skills/improvement-tracker/templates/approved-input.yaml` | 181 | Approved Improvement Input 模板：完整 schema 定义（12 字段）+ 约束汇总（8 条）+ 字段映射表 + 2 个完整示例 |

### 修改的文件

| 文件 | 变更 |
|------|------|
| `agents/ptm-tse.md` | skills 列表追加 improvement-tracker；新增改进治理职责 3 条 + 流程 3 步骤 + 检查点 3 条；source_feature 追加 FEAT-RA-IMPROVEMENT |
| `skills/README.md` | ptm-tse 章节追加 reverse-analysis 和 improvement-tracker Skill 索引 |

### 状态机实现细节

- **CA/PA 批准状态机**：draft → (reviewer approve) → approved / (reviewer reject) → rejected
  - 只有人工 reviewer 能改变 approval_status
  - approved 和 rejected 是终态，不可回退或互转
  - 修改已批准的 CA/PA 须创建新 proposal（新 proposal_id）并重新走批准流程
- **Approved Input 不可变性**：生成时写入 immutable:true，生成后 Skill 规则拒绝任何修改
  - 需更新时：创建新 proposal → 批准 → 生成新 Approved Input（旧 Input 保留）
- **消费者映射**：target_agent 枚举值 | ptm-tde → pending-consumer | ptm-te → pending-consumer | ptm-tae → pending-consumer | ptm-qa → pending-consumer | 其他 → blocked
  - 首版不检测下游是否真正就绪

### 关键决策与偏差

- 无偏差。严格按照 LLD 的 14 个章节设计实现。
- `agents/ptm-tse.md` 的修改属于 Skill 引用更新的低风险追加（仅在 skills 列表、职责、流程和检查点追加 improvement-tracker 相关内容），不改变原有 reverse-analysis 逻辑。

### 已知限制

- 首版不检测下游 Agent 就绪状态（consumer_status=blocked 仅因 target_agent 非法触发）
- 下游消费回写 consumer_status 由 ST-RA-04 负责
- SKILL.md §4/§5 为占位符，等待后续 Story 写入

### 验证入口

- 主要入口：`skills/improvement-tracker/SKILL.md` — 检查 frontmatter 完整性、§3 四子节、Gotchas
- 模板验证：`skills/improvement-tracker/templates/capa-proposal.yaml` + `approved-input.yaml` — schema 定义完整性
- Agent 集成：`agents/ptm-tse.md` — skills 列表含 improvement-tracker
- CP6 检查：`process/checks/CP6-ST-RA-03-improvement-governance-CODING-DONE.md`

### 后续 Story 依赖

| Story | 依赖本 Story 的 |
|-------|----------------|
| ST-RA-04 | `improvement-tracker/SKILL.md` §4 写入位置 + Approved Input 作为跟踪基础 |
| ST-RA-06.3-TRACK | `improvement-tracker/SKILL.md` §5 写入位置 + CA/PA Proposal 的 proposal_ref |
