# CP2 Scenario Discussion Log - CR-024

| 字段 | 值 |
|------|------|
| change_id | CR-024-ptm-te-agent |
| workflow_id | WF-PTM-TEAM-20260520-001 |
| 阶段 | requirement-clarification（CP2 前置） |
| 讨论方式 | 计划 v4 4 轮评审 inline 确认（非异步问答） |
| 恢复点 | `process/checks/CP2-DISCUSSION-CHECKPOINT-CR-024.json` |
| 记录时间 | 2026-07-10 |

## 讨论背景

ptm-te agent 实现计划（`/home/hyde/.claude/plans/ptm-te-agent-impl.md`）经过 4 轮评审（v1->v2 CLI 真相->v3 评审点+实测->v4 M1/M3/M4/M5+决策建议），用户在评审中通过 AskUserQuestion 选择 + 评审意见确认了关键方向。本日志固化 4 个 Scenario Gray Areas 的讨论证据，作为 CP2 Decision Brief 的输入。

## 灰区 1：runtime 写操作授权边界（SGQ-01）

**问题**：ptm-te 执行策略路由用例时，`ptm-atomic run ... --execute` 会真实修改设备策略。runtime 写操作的授权边界如何划定？

**候选选项**：
- dry-run 默认门：首期 CP7 默认 `--dry-run`，`--execute` 作为独立 runtime_authorization 决策项单次确认
- 完全放开 --execute：CP7 自动执行写操作，无人工确认
- 全 dry-run：首期 CP7 只 dry-run，--execute 留 v2

**用户回答（计划 v4 评审确认）**：采纳 dry-run 默认门。用户在 v3->v4 评审中要求"决策建议正式化"，未反对 dry-run 默认门方向。

**复述确认**：dry-run 默认门--首期 CP7 默认 dry-run 验证参数路由和 session 有效性；--execute 涉及真实设备策略变更，作为独立 runtime_authorization 决策项需用户单次确认。

**影响面**：CP7 验证策略、设备安全、回滚/清理逻辑可验证性。

**分类**：decision-item -> CR024-DQ-01（runtime_authorization，CP2 决策）。

## 灰区 2：验证设备范围（SGQ-02）

**问题**：CP7 runtime 验证需要真实设备承载。哪台设备承载验证？

**候选选项**：
- hg3250-51（10.113.55.51，DAS-TGFW-A1300-HU，Web 443）
- nxp1046-95（10.113.55.95，DAS-TGFW-1900）
- 两台都验证

**用户回答（计划 v4 评审确认）**：推荐 hg3250-51。manaul 项目日志证明 hg3250-51 策略路由可通，风险最低。用户评审未反对。

**复述确认**：hg3250-51 优先（manaul 已验证策略路由可通），不可达时 switch 到 nxp1046-95；1900 型号留 follow-up（T-02）。

**影响面**：CP7 runtime 可行性、型号覆盖。

**分类**：decision-item -> CR024-DQ-02（scope，CP2 决策）。

## 灰区 3：用例来源（SGQ-03）

**问题**：ptm-te 验证用例从哪来？真实消费 ptm-tde 已产出 PC，还是手写最小 PC？

**候选选项**：
- 手写最小 PC 优先：手写最小策略路由 PC（1 config + 1 verify + 1 delete）验证全链路
- 真实消费 ptm-tde 已产出 PC
- 不消费 PC，纯 CLI 命令验证

**用户回答（计划 v4 评审确认）**：推荐手写最小 PC 优先。理由是覆盖端到端消费链路但不依赖 ptm-tde 产出质量。用户评审未反对。

**复述确认**：手写最小 PC 优先--首期手写最小策略路由 PC 验证 ptm-te 全链路（解析->映射->执行->清理）；真实消费 ptm-tde PC 留 follow-up（T-01），CP3 锁定 args 命名后可切换。

**影响面**：CP7 端到端验证链路、是否阻塞于 ptm-tde 产出质量。

**分类**：decision-item -> CR024-DQ-04（scope，CP2 决策）。

## 灰区 4：ptm-tde PC args 命名约定（SGQ-04）

**问题**：ptm-tde 产出 PC 的 `case_steps[].atomic_op.args` 字段名（如 `src_addr`）与 op yaml `inputs.params`（`source_network`）和 CLI flag（`--source-network`）三层不一致。如何统一？

**候选选项**：
- ptm-te op_mapper 承载翻译：保持 ptm-tde args 不变，op_mapper.py 做两层映射
- ptm-tde 改用 op yaml 参数名：统一为 source_network，映射简化

**用户回答（计划 v4 评审确认）**：标注 CP3 锁定。用户 v3->v4 评审明确要求"§3.2 第二层映射表覆盖全部 7 个 op"（M5），即认可 op_mapper 承载双层映射路径。ptm-tde 侧是否改名留 CP3 与 ptm-tde 契约对齐时决定。

**复述确认**：CP3 HLD 锁定三层映射策略 + op_mapper.py 覆盖全部 7 个 op 的 flag 映射；ptm-tde args 命名是否统一留 CP3 决定（若 ptm-tde 侧不改，op_mapper 承载翻译）。

**影响面**：op_mapper.py 双层映射复杂度、与 ptm-tde 契约漂移风险。

**分类**：技术实现细节，CP3 HLD 锁定（non-blocking-open for CP2，不升级为 CP2 用户决策）。

## Deferred Ideas（未选重点讨论项）

| 项 | 去向 |
|----|------|
| 1900 型号设备验证 | CR-024 T-02 candidate |
| managed rule block v2 | CR-024 T-03 candidate |
| 进程管理 / 串口初始化 | CR-024 T-04 candidate |
| batch policy-route package 编排 | CR-024 T-05 candidate |

## 讨论结论

4 个灰区中，3 个（SGQ-01/02/03）升级为 CP2 用户决策（CR024-DQ-01/02/04），1 个（SGQ-04）在 CP3 HLD 锁定。所有灰区已有用户评审确认的方向，CP2 Decision Brief 将 3 个决策项正式呈现供用户 approve。
