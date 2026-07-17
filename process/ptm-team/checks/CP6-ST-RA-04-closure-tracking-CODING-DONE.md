# CP6 编码完成检查 — ST-RA-04: 闭环跟踪与有效性决策

## Entry Criteria

| 条目 | 状态 | 说明 |
|---|---|---|
| CP5 设计证据已确认 | ✅ | CP5-CR030-DESIGN-EVIDENCE-BATCH.md approved 2026-07-16 |
| ST-RA-04 LLD 就绪 | ✅ | `process/stories/STORY-RA-04-LLD.md` — full-lld, 14 语义要点 |
| ST-RA-03 完成 | ✅ | §1-§3（CA/PA 治理）已写入 SKILL.md，§4 为原占位符 |
| 前置 Story 门控满足 | ✅ | ST-RA-03 (contract) — 接口已产出（CA/PA Proposal + Approved Input） |
| 文件所有权无冲突 | ✅ | §4 追加位于 §3 之后、§5 之前，串行写入链无并行冲突 |
| Wave 3 可执行 | ✅ | CP5 全量确认通过，Wave 3 为当前实现 Wave |

## Checklist

| # | 检查项 | 结果 | 证据 |
|---|---|---|---|
| 1 | SKILL.md §4 闭环跟踪完整 | ✅ PASS | §4.1（行动项状态机）、§4.2（有效性检查）、§4.3（观察窗逻辑）、§4.4（关闭决策四条件） |
| 2 | action-item.yaml 存在且完整 | ✅ PASS | 175 行，含 schema + 状态枚举 not-started/in-progress/done/overdue + 4 个示例 |
| 3 | effectiveness-check.yaml 存在且完整 | ✅ PASS | 175 行，含 schema + 复发统计字段 + 4 个示例 |
| 4 | closure-decision.yaml 存在且完整 | ✅ PASS | 183 行，含 schema + 四条件 + 硬断言 + 4 个示例 |
| 5 | 行动项状态机 not-started→in-progress→done | ✅ PASS | SKILL.md §4.1 含 ASCII 图、迁移规则表和禁止迁移列表 |
| 6 | overdue 自动标记 | ✅ PASS | 动态计算（now > due_date 且 status ∈ {not-started, in-progress}），不持久化 |
| 7 | done 不可回退 | ✅ PASS | 明确禁止迁移规则，done 是终态 |
| 8 | 有效性检查 result 枚举 | ✅ PASS | planned/passed/failed/inconclusive，初始 planned，仅 reviewer 可修改 |
| 9 | 复发统计查询定义 | ✅ PASS | 从 SQLite measure_link 只读查询，recurrence_rate = same/total |
| 10 | 观察窗 30 天默认值 | ✅ PASS | window_days=30，满足条件 now - checked_at >= window_days |
| 11 | 观察窗豁免规则 | ✅ PASS | reviewer 显式声明原因，不影响其他三个条件 |
| 12 | 四关闭条件全部满足 → closed | ✅ PASS | 硬断言：四个 conditions 全部 true + reviewer 签名 |
| 13 | 任一条件不满足 → open | ✅ PASS | residual_risks + follow_up_actions 必填 |
| 14 | 全部 done 但有效性 failed → open | ✅ PASS | Gotcha #11 + 不可绕过示例表明确覆盖 |
| 15 | 不自动关闭 RA | ✅ PASS | Skill/AI 设置 decision=closed → 拒绝，需 reviewer 签名 |
| 16 | 不自动完成行动项/判定有效性 | ✅ PASS | status=done 仅 Owner/Reviewer；result 仅 reviewer |
| 17 | SKILL.md §3（ST-RA-03 产出）未修改 | ✅ PASS | §3.1-§3.4 四个小节完整，仅追加 §4 |
| 18 | 前端元数据已更新 | ✅ PASS | version: 1.0→1.1，修订记录追加 ST-RA-04 条目 |
| 19 | Gotchas 已追加闭环侧条目 | ✅ PASS | #11-#17 覆盖完成率误区、overdue 误用、done 回退、豁免默认真、自动判定、作用域混淆、零复发误读 |
| 20 | 模板文件符合既存格式 | ✅ PASS | 与 capa-proposal.yaml/approved-input.yaml 风格一致（schema定义+约束+示例） |

## 实现执行证据

### 实现对象清单

| 对象 | 路径 | 操作 | 行数 |
|---|---|---|---|
| improvement-tracker Skill §4 | `skills/improvement-tracker/SKILL.md` | 追加（§4 从 4 行占位符改写为 215 行） | 570 (total) |
| Action Item 模板 | `skills/improvement-tracker/templates/action-item.yaml` | 创建 | 175 |
| Effectiveness Check 模板 | `skills/improvement-tracker/templates/effectiveness-check.yaml` | 创建 | 175 |
| Closure Decision 模板 | `skills/improvement-tracker/templates/closure-decision.yaml` | 创建 | 183 |

### 设计契约映射

| LLD 章节 | 实现位置 | 覆盖状态 |
|---|---|---|
| §5.1 Action Item Schema | action-item.yaml | ✅ 字段映射一致 |
| §5.2 Effectiveness Check Schema | effectiveness-check.yaml | ✅ 字段映射一致，含 recurrence_measure |
| §5.3 Closure Decision Schema | closure-decision.yaml | ✅ 四条件 + 硬断言完整 |
| §7.1 Action Item 状态机 | SKILL.md §4.1 | ✅ 状态迁移规则 + 禁止迁移 + 失败处理 |
| §7.2 有效性检查流程 | SKILL.md §4.2 | ✅ 5 步流程 + result 判定规则 |
| §7.3 关闭决策四条件评估 | SKILL.md §4.4 | ✅ 条件表 + 硬断言 + 不可绕过示例 |
| §7.4 关闭条件不可绕过 | SKILL.md §4.4 | ✅ 硬断言伪代码 + 5 个场景示例表 |
| §7.5 异常路径 | SKILL.md §4.1/§4.2/§4.4 | ✅ 各自的失败处理表覆盖 |

### 单元测试与 Fixture 计划

本 Story 产出为文本定义（Skill 章节 + YAML 模板），无运行时代码。验证方式为静态审查：

- **状态机验证**：人工检查 SKILL.md §4.1 中状态迁移规则与 LLD §7.1 一致
- **门控断言验证**：人工检查 SKILL.md §4.4 中四条件硬断言代码块与 LLD §7.4 一致
- **模板 Schema 验证**：人工检查三个 YAML 模板的必填字段与 LLD §5 一致
- **串行追加验证**：diff 检查 SKILL.md §3.1-§3.4 未被修改
- **CP7 阶段**：由 meta-qa 执行静态审查，逐条核对 LLD 测试场景清单（T-RA-04-FSM-01~05, CLOSE-01~06, SEC-01~03, OVERDUE-01, EFF-01~02, RECUR-01）

N/A 理由：本 Story 无 Python/Shell 等可执行代码，不适用单元测试框架。

### 最小实现切片

单切片实现（4 个文件一次性产出）：模板文件独立于 SKILL.md，可任意顺序创建；SKILL.md 在模板创建后追加。

### 平台差异处理

N/A。本 Story 产出为跨平台 Skill 文本定义和 YAML 模板，与 Codex/Claude Code/Qoder 平台无关。

### 验证结果

| 验证项 | 方法 | 结果 |
|---|---|---|
| SKILL.md §3 完整性 | grep §3 小节 | ✅ 4 个小节完整（§3.1-§3.4） |
| SKILL.md §4 完整性 | grep §4 小节 | ✅ 4 个小节完整（§4.1-§4.4） |
| 模板文件存在 | ls | ✅ 3 个模板文件均非空 |
| 模板文件格式 | 人工审查 | ✅ 与既存模板风格一致 |
| Gotchas 覆盖 | 人工审查 | ✅ #1-#10（CA/PA 侧）+ #11-#17（闭环侧） |

### 未覆盖项

| 项目 | 说明 |
|---|---|
| T-RA-04 测试场景 fixture | 测试场景已在 LLD §10.2 定义，fixture 数据由 CP7 meta-qa 创建 |
| measure_link 运行时查询 | OPEN-RA-04-01 需上游 ST-RA-INGEST-DB 确认 proposal_ref 字段 |
| 观察窗豁免权限校验 | OPEN-RA-04-02，首版依赖 reviewer 身份约定 |

### 设计缺口反馈

无新增设计缺口。LLD §5 数据模型、§7 流程、§9 安全约束均在实现中完整落地。

### 后续交接

交给 meta-qa 执行 CP7 静态审查。关键验证入口：
1. `skills/improvement-tracker/SKILL.md` §4 — 验证四个子节与 LLD §7 契约一致
2. `skills/improvement-tracker/templates/action-item.yaml` — 验证状态枚举和迁移规则
3. `skills/improvement-tracker/templates/effectiveness-check.yaml` — 验证复发统计字段和 result 枚举
4. `skills/improvement-tracker/templates/closure-decision.yaml` — 验证四条件硬断言
5. 确认 §3（ST-RA-03 产出）未被修改

## Exit Criteria

| 条目 | 状态 |
|---|---|
| 所有输出文件存在且非空 | ✅ |
| §3 内容未被修改 | ✅ |
| 四条件关闭逻辑完整 | ✅ |
| 不可自动关闭门控完整 | ✅ |
| Story Return Packet 已生成 | ✅ |
| Evidence Index 已生成 | ✅ |
| CP6 检查完成 | ✅ |

## 结论

**CP6 结论：PASS**

全部 20 项 checklist 通过。4 个输出文件完整且符合 LLD 契约。ST-RA-03 产出的 §3 内容未被修改。准备交付 meta-qa 进行 CP7 静态审查验证。
