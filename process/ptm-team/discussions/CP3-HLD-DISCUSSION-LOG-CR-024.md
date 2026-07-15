# CP3 HLD Discussion Log - CR-024

| 字段 | 值 |
|------|------|
| change_id | CR-024-ptm-te-agent |
| workflow_id | WF-PTM-TEAM-20260520-001 |
| 阶段 | solution-design（CP3 前置） |
| 讨论方式 | 计划 v4 4 轮评审 + HLD advisor table |
| 恢复点 | `process/checks/CP3-DISCUSSION-CHECKPOINT-CR-024.json` |
| HLD | `process/HLD-CR-024.md` §10 |
| 记录时间 | 2026-07-10 |

## 讨论背景

HLD v1.0（`process/HLD-CR-024.md`）基于计划 v4 §4 产出。3 个 Architecture Gray Areas 已在 HLD §10 以 advisor table 形式记录，本日志固化讨论证据。

## AGA-01：三层命名映射策略

**问题**：ptm-tde PC args（`src_addr`）≠ op yaml params（`source_network`）≠ CLI flag（`--source-network`）。映射由谁承载？

**advisor table**（HLD §10 AGA-01）：
- Option A：ptm-te op_mapper 承载翻译（ptm-tde args 不变）-> 推荐
- Option B：ptm-tde 改用 op yaml 参数名 -> 不推荐（破坏已交付基线）

**讨论结论**：Option A。理由：ptm-tde 已交付（CR-010~017 closed），改 args 命名破坏基线；op_mapper centralize 映射便于维护；三层映射是 ptm-te 消费适配职责。锁定为 ADR-01。

**影响面**：op_mapper.py 实现 + CP7 static 校验（映射一致性）。

## AGA-02：device-management 与 device-connection 拆分边界

**问题**：manaul 的 device-management 只做元数据，连接逻辑在 collect_sysinfo.py。ptm-te 如何组织？

**advisor table**（HLD §10 AGA-02）：
- Option A：拆为两个 skill（元数据 + 连接）-> 推荐
- Option B：合并为一个 skill -> 不推荐（耦合）

**讨论结论**：Option A。理由：manaul collect_sysinfo.py 236 行表明连接逻辑不轻；职责单一利于复用；P1-1 修正已确认拆分。锁定为 ADR-05。

**影响面**：S2 实现（device-management + device-connection 两个 skill）。

## AGA-03：inverse_op 回滚豁免策略

**问题**：`fw_reset_policy_route_hitcount` 是 irreversible（命中计数清零不可恢复）。是否强行回滚？

**advisor table**（HLD §10 AGA-03）：
- Option A：irreversible 类不回滚，SKILL 注明 -> 推荐
- Option B：强行回滚所有 op -> 不推荐（不可逆操作无法真正回滚，误导）

**讨论结论**：Option A。理由：符合 op 语义；不强行不可逆操作；用例设计者需接受副作用或通过用例顺序规避。锁定为 ADR-03。

**影响面**：S3 SKILL 错误表 + Gotchas；§9 回滚策略。

## 讨论结论

3 个 AGA 均已基于计划 v4 评审 + HLD advisor table 确定方向，锁定为 ADR-01/05/03。CP3 人工门禁确认 HLD 整体 + ADR + inline-fallback。
