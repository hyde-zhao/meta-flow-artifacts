# CP6 编码完成检查 — ST-RA-06.3-TRACK: 措施基线管理与刷新提示

## Entry Criteria

| 条目 | 状态 | 说明 |
|---|---|---|
| CP5 设计证据已确认 | ✅ | CP5 auto-precheck PASS（CP5-ST-RA-06.3-TRACK-measure-baseline-refresh-LLD-IMPLEMENTABILITY.md），technical-note 完整 |
| Story 设计证据 | ✅ | 技术说明内嵌于 Story 卡片 `## 技术说明`（8 节完整），lld_policy=technical-note |
| ST-RA-04 完成 | ✅ | §4 闭环跟踪已写入 SKILL.md |
| ST-RA-06.1-DETECT 完成 | ✅ | S2 变更检测渠道已就绪 |
| 前置 Story 门控满足 | ✅ | ST-RA-04 (runtime) 和 ST-RA-06.1-DETECT (runtime) 接口已产出 |
| 文件所有权无冲突 | ✅ | §5 追加位于 §4 之后，shared_writers 已声明，串行写入链无并行冲突 |
| Wave 4 可执行 | ✅ | CR-030 最后 Story，全部前置已完成 |

## Checklist

| # | 检查项 | 结果 | 证据 |
|---|---|---|---|
| 1 | SKILL.md §5 措施基线管理完整 | ✅ PASS | §5.1（概念）、§5.2（基线创建）、§5.3（刷新提示规则）、§5.4（reviewer 是唯一状态变更者）、§5.5（禁止行为）|
| 2 | measure-baseline.yaml 存在且完整 | ✅ PASS | 含 schema 定义（12 字段）、约束汇总（11 条）、刷新提示枚举、状态迁移图、3 个示例 |
| 3 | MeasureBaseline 概念符合 HLD 契约 | ✅ PASS | "无基线 = needs-baseline（不判失效）" 在 §5.1 和 Gotcha #19 中明确 |
| 4 | 刷新提示规则完整（6 种触发条件） | ✅ PASS | §5.3 触发规则表覆盖 needs-baseline/needs-review/completed/active/needs-review(复发)/superseded |
| 5 | proposed_status 与 status 分离 | ✅ PASS | §5.1 原则、§5.3 约束、§5.4 权限模型中均区分系统提示与正式状态 |
| 6 | reviewer 是唯一状态变更者 | ✅ PASS | §5.4 权限模型 + 调用路径图 + reviewer_update_measure_status() 仅 reviewer 可调用 |
| 7 | 系统只能写提示字段 | ✅ PASS | update_measure_refresh_hint() 允许字段 vs 禁止字段明确，DAO 层 hard reject |
| 8 | 不自动创建基线 | ✅ PASS | §5.2 失败处理 + §5.5 禁止行为 #1 + Gotcha #20 |
| 9 | 不自动改变批准状态 | ✅ PASS | §5.3 "系统只能写提示字段" + §5.5 禁止行为 #2 |
| 10 | 不自动修改下游任务 | ✅ PASS | §5.5 禁止行为 #5 |
| 11 | 无基线不判失效 | ✅ PASS | §5.1 核心原则 + §5.5 禁止行为 #3 + Gotcha #19 |
| 12 | SKILL.md §3（ST-RA-03 产出）未修改 | ✅ PASS | §3.1-§3.4 保持完整，仅 §5 占位符替换 |
| 13 | SKILL.md §4（ST-RA-04 产出）未修改 | ✅ PASS | §4.1-§4.4 保持完整 |
| 14 | 前端元数据已更新 | ✅ PASS | shared_writers 更新为完整标题，实现状态已更新为"已实现" |
| 15 | Gotchas 已追加 §5 侧条目 | ✅ PASS | #18-#22 覆盖 proposed/status 区分、needs-baseline 误判、自动基线、hint 解析、superseded 保留 |
| 16 | 修订记录已更新 | ✅ PASS | version 1.2 条目含 §5 完整内容 + 模板 + Gotchas |
| 17 | 模板文件符合既存格式 | ✅ PASS | 与 capa-proposal.yaml 等风格一致（schema 定义 + 约束 + 示例） |
| 18 | measure-baseline.yaml 含刷新提示枚举表 | ✅ PASS | 模板文件底部的刷新提示枚举表与 SKILL.md §5.3 一致 |
| 19 | measure-baseline.yaml 含状态迁移图 | ✅ PASS | ASCII 迁移图覆盖 needs-baseline→active、active↔needs-review、→completed/superseded |
| 20 | DAO 层未修改 | ✅ PASS | data/dao.py 未被修改（仅消费 insert_measure_link / update_measure_refresh_hint / reviewer_update_measure_status 接口） |

## 实现执行证据

### 实现对象清单

| 对象 | 路径 | 操作 | 说明 |
|---|---|---|---|
| improvement-tracker Skill §5 | `skills/improvement-tracker/SKILL.md` | 修改（§5 从 4 行占位符改写为 117 行完整内容） | §5.1-§5.5 五个子节 |
| MeasureBaseline 模板 | `skills/improvement-tracker/templates/measure-baseline.yaml` | 创建 | schema（12 字段）+ 约束（11 条）+ 枚举 + 迁移图 + 3 示例 |

### 设计契约映射

| 技术说明章节 | 实现位置 | 覆盖状态 |
|---|---|---|
| 1. 设计依据 | SKILL.md §5.1 | ✅ HLD MeasureBaseline 契约 + REQ-RA-022 |
| 2. 文件影响 | SKILL.md §5 (追加) + measure-baseline.yaml (创建) | ✅ 路径明确 |
| 3.1 MeasureBaseline Schema | measure-baseline.yaml (schema 定义) | ✅ 12 字段完整映射（含 proposed_status/refresh_hint） |
| 3.2 措施刷新规则 | SKILL.md §5.3 (触发规则表) | ✅ 6 种触发条件 + proposed_status + refresh_hint |
| 3.3 权限模型 | SKILL.md §5.4 | ✅ 系统 vs reviewer 权限表 + 调用路径图 |
| 3.4 上游读取依赖 | SKILL.md §5.3 (触发来源列) | ✅ ST-RA-03/04/06.2 依赖声明 |
| 4. 核心流程 | SKILL.md §5.4 (调用路径 ASCII 图) | ✅ 事件 → 系统提示 → reviewer 决策 |
| 5. 错误处理与降级 | SKILL.md §5.2 (基线创建失败表) | ✅ 4 种失败场景 |
| 7. 风险与重访条件 | Gotcha #18-#22 | ✅ 5 项 Gotcha 覆盖关键风险 |
| 8. 偏离记录 | 首版 version 固定为 1、不自动递增 | ✅ measure-baseline.yaml 约束 #11 |

### 单元测试与 Fixture 计划

本 Story 产出为文本定义（Skill 章节 + YAML 模板），无运行时代码。验证方式为静态审查：

- **Schema 完整性验证**：人工检查 measure-baseline.yaml 的 12 字段与 Story 技术说明 §3.1 一致
- **刷新规则覆盖验证**：人工检查 SKILL.md §5.3 触发规则表的 6 种条件与 Story 的 6 种触发条件一致
- **权限模型验证**：人工检查 SKILL.md §5.4 的禁止操作表与 Story §3.3 权限模型一致
- **串行追加验证**：diff 检查 SKILL.md §3.1-§3.4 和 §4.1-§4.4 未被修改
- **模板格式验证**：人工检查 measure-baseline.yaml 与既存模板风格一致
- **CP7 阶段**：由 meta-qa 执行静态审查，逐条核对 Story 测试场景清单（T-TRK-06/07/08/10, T-RA-063-BASELINE-01~04, T-RA-063-GATE-01~02）

N/A 理由：本 Story 无 Python/Shell 等可执行代码，不适用单元测试框架。

### 最小实现切片

单切片实现（2 个文件一次性产出）：measure-baseline.yaml 独立创建，SKILL.md §5 在其后追加。

### 平台差异处理

N/A。本 Story 产出为跨平台 Skill 文本定义和 YAML 模板，与 Codex/Claude Code/Qoder 平台无关。

### 验证结果

| 验证项 | 方法 | 结果 |
|---|---|---|
| SKILL.md §3 完整性 | grep §3 小节 | ✅ §3.1-§3.4 四个小节完整 |
| SKILL.md §4 完整性 | grep §4 小节 | ✅ §4.1-§4.4 四个小节完整 |
| SKILL.md §5 完整性 | grep §5 小节 | ✅ §5.1-§5.5 五个小节完整 |
| 模板文件存在 | ls | ✅ measure-baseline.yaml 非空 |
| 模板文件格式 | 人工审查 | ✅ 与既存模板风格一致 |
| Gotchas 覆盖 | 人工审查 | ✅ #1-#17（CA/PA + 闭环侧）+ #18-#22（基线侧） |
| 不修改 DAO | git diff data/dao.py | ✅ 无变更 |
| 技术说明 §8 偏离记录 | 人工审查 | ✅ 首版 version=1 在模板约束 #11 中声明 |

### 未覆盖项

| 项目 | 说明 |
|---|---|
| T-RA-06.3 测试场景 fixture | 测试场景已在 Story 技术说明 §6 定义，fixture 数据由 CP7 meta-qa 创建 |
| 基线版本自动递增 | 首版 version 固定为 1，自动递增需后续 S2 增量重算多轮后实现 |
| proposed_status 事件驱动实时推送 | 当前实现为 SKILL.md 中的规则定义和 DAO 接口契约，实时事件推送为运行时增强 |

### 设计缺口反馈

无新增设计缺口。技术说明 §1-§8 的 8 个章节在 SKILL.md §5 中完整落地。偏离记录中的 3 项偏离已通过约束声明和 Gotcha 覆盖。

### 后续交接

交给 meta-qa 执行 CP7 静态审查。关键验证入口：

1. `skills/improvement-tracker/SKILL.md` §5 — 验证五个子节与技术说明一致
2. `skills/improvement-tracker/templates/measure-baseline.yaml` — 验证 schema 字段、约束、刷新枚举、状态迁移图
3. 确认 §3（ST-RA-03）和 §4（ST-RA-04）未被修改
4. 确认 data/dao.py 未被修改
5. 验收标准验证：
   - AC1: MeasureBaseline 含版本/范围/审批引用/观察窗 → measure-baseline.yaml
   - AC2: 无基线 → needs-baseline（不判措施失效）→ SKILL.md §5.1 + Gotcha #19
   - AC3: 措施刷新只产出提示，不自动改变批准状态 → SKILL.md §5.3 + §5.4 权限模型
   - AC4: 不自动修改下游任务 → SKILL.md §5.5 禁止行为 #5
   - AC5: measure-baseline 模板完整 → measure-baseline.yaml

## Exit Criteria

| 条目 | 状态 |
|---|---|
| 所有输出文件存在且非空 | ✅ |
| §3 内容未被修改 | ✅ |
| §4 内容未被修改 | ✅ |
| data/dao.py 未被修改 | ✅ |
| proposed_status/status 分离完整 | ✅ |
| reviewer 唯一状态变更者门控完整 | ✅ |
| Story Return Packet 已生成 | ✅ |
| Evidence Index 已生成 | ✅ |
| CP6 检查完成 | ✅ |

## 结论

**CP6 结论：PASS**

全部 20 项 checklist 通过。2 个输出文件完整且符合技术说明契约。SKILL.md §3（ST-RA-03）和 §4（ST-RA-04）内容未被修改。data/dao.py 未被修改。准备交付 meta-qa 进行 CP7 静态审查验证。
