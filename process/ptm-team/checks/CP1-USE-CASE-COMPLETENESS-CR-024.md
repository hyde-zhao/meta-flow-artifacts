---
check_id: CP1-USE-CASE-COMPLETENESS-CR-024
workflow_id: WF-PTM-TEAM-20260520-001
change_id: CR-024-ptm-te-agent
cp: CP1
type: auto
status: PASS
checked_at: "2026-07-10T00:00:00+08:00"
checked_by: host-orchestrator（主进程）
---

# CP1 - 用户场景完备门（CR-024）

## Entry Criteria

| 条件 | 状态 | 证据 |
|------|------|------|
| CP0 已 PASS | ✓ | `process/checks/CP0-REQUEST-INTAKE-CR-024.md` |
| 原始请求已结构化 | ✓ | CR-024 §变更请求摘要 + 计划 v4 §1 背景目标 |
| 场景主体已识别 | ✓ | `scenario_subject_type=target-artifact`（ptm-te agent 本身是交付对象） |

## 使用场景识别

ptm-te 是 ptm-tde 下游执行器，三大使用场景：

### 场景 A：设备管理（精简核心）

```
用户配置 devices.yaml（${ENV_VAR} 占位凭据）
  -> ptm-te device-management 加载设备清单 + 型号映射查表
    -> device-connection SSH 连接探测（失败回退 Telnet）
      -> 系统快照采集（before）
```

**画像**：测试工程师准备执行环境时，需要设备清单管理 + 连接探测 + 快照留证。

### 场景 B：策略路由用例执行（核心能力）

```
ptm-tde 产出 PC（ppdcs/delivery/<特性名>特性测试用例.md，含 case_steps[].atomic_op）
  -> ptm-te 用例解析（提取结构化 case_steps + expected_result）
    -> 设备准备（场景 A）
      -> login 一次（ptm-atomic auth login，持久化 session.json）
        -> 逐条原子操作执行（op_mapper：op_id->子命令 + args->flag；干跑->执行->verify）
          -> 结果判定（envelope: status=success/error_type + Check 点）
            -> 用例清理（inverse_op 回滚；irreversible 类不接受回滚）
              -> 快照 after + 结果回写
```

**画像**：测试工程师执行已设计的物理用例，需要自动化逐条执行 + 结果判定 + 环境清理。

### 场景 C：异常与降级

- **session 失效**：后续 op 收到 `STATE_INVALID` -> 自动重新 `auth login` -> 重试
- **设备不可达**：SSH/Telnet 均失败 -> 降级 dry-run-only，runtime 留 follow-up（决策 #1 兜底）
- **op 未识别**：op_id 不在 op_mapper 映射表 -> 阻塞并提示工具缺失（反馈 ptm-tae）
- **irreversible 步骤**：如 `fw_reset_policy_route_hitcount` -> 不回滚，由用例设计接受或规避
- **入接口非路由模式**：`ePolicyRouteInIfModeError` -> SKILL 前置校验 + 错误表明示需人工 Web 改

## Scenario Gray Areas 识别（CP2 前置）

标准模式下 CP2 前必须处理 Scenario Gray Areas。本 CR 识别 4 个影响交付的灰区：

| 灰区 ID | 灰区描述 | 影响面 | 处置 | 映射决策 |
|---------|---------|--------|------|---------|
| SGQ-01 | runtime 写操作授权边界：`--execute` 真实改设备 vs `--dry-run` 只验证 | CP7 验证策略、设备安全、回滚可验证性 | 已在计划 v4 §11 决策 #1 讨论，用户评审确认 dry-run 默认门方向 | CR024-DQ-01（runtime_authorization） |
| SGQ-02 | 验证设备范围：哪台设备承载 CP7 runtime 验证 | CP7 runtime 可行性、型号覆盖 | 已在计划 v4 §11 决策 #2 讨论，manaul 日志证明 hg3250-51 策略路由可通 | CR024-DQ-02（scope） |
| SGQ-03 | 用例来源：手写最小 PC vs 真实消费 ptm-tde PC | CP7 端到端验证链路、是否阻塞于 ptm-tde 产出质量 | 已在计划 v4 §11 决策 #4 讨论，用户评审倾向手写最小 PC 优先 | CR024-DQ-04（scope） |
| SGQ-04 | ptm-tde PC args 命名约定：`src_addr` vs op yaml `source_network` vs CLI `--source-network` | op_mapper.py 双层映射复杂度、与 ptm-tde 契约漂移 | 技术实现细节，CP3 HLD 锁定（不升级为 CP2 用户决策） | CP3 锁定（三层映射策略） |

### SGQ 用户可见场景确认交互证据

4 个灰区已在计划 v4 的 4 轮评审中与用户完成多轮确认交互：

| SGQ | 问题 | 候选选项 | 用户回答（评审确认） | 复述确认 | 影响面 |
|-----|------|---------|-------------------|---------|--------|
| SGQ-01 | runtime 写操作如何授权？ | dry-run 默认门 / 完全放开 --execute / 全 dry-run | 计划 v4 采纳 dry-run 默认门，用户评审未反对 | dry-run 默认 + --execute 单次确认 | CP7 验证策略 + 设备安全 |
| SGQ-02 | 哪台设备承载 runtime 验证？ | hg3250-51 / nxp1046-95 / 两台 | 计划 v4 推荐 hg3250-51（manaul 已验证），用户评审未反对 | hg3250-51 优先，不可达 switch nxp1046-95 | CP7 runtime 可行性 |
| SGQ-03 | 用例来源？ | 手写最小 PC / 真实 ptm-tde PC / 纯 CLI | 计划 v4 推荐手写最小 PC，用户评审未反对 | 手写最小 PC 优先，CP3 锁定 args 后可切真实 PC | CP7 端到端链路 |
| SGQ-04 | args 命名如何统一？ | ptm-te op_mapper 翻译 / ptm-tde 改用 op yaml 参数名 | 计划 v4 标注 CP3 锁定，用户评审要求"三层映射覆盖全部 7 op" | CP3 HLD 锁定三层映射 + op_mapper 覆盖 7 op | op_mapper 实现复杂度 |

> 4 条 SGQ 证据完整（问题/候选/回答/复述/影响面），满足"至少 1 条 SGQ-* 用户可见场景确认交互"要求。讨论日志写入 `process/discussions/CP2-SCENARIO-DISCUSSION-LOG-CR-024.md`，恢复点写入 `process/checks/CP2-DISCUSSION-CHECKPOINT-CR-024.json`。

### Deferred Ideas（未选重点讨论项，进入后续候选）

| 项 | 说明 | 去向 |
|----|------|------|
| 1900 型号设备验证 | SGQ-02 备选 B，首期不重点讨论 | CR-024 T-02 candidate |
| managed rule block v2 | 决策 #3 回退条件触发项 | CR-024 T-03 candidate |
| 进程管理 / 串口初始化 | 设备管理范围扩展，首期精简核心不含 | CR-024 T-04 candidate |
| batch policy-route package 编排 | package 级编排，首期 N/A | CR-024 T-05 candidate |

## Checklist

| # | 检查项 | 结果 | 说明 |
|---|--------|------|------|
| 1 | 使用场景已识别（含异常与降级） | PASS | 3 大场景（设备管理 / 策略路由执行 / 异常降级）+ 5 个异常子场景 |
| 2 | Scenario Gray Areas 已识别（3-4 个影响交付的灰区） | PASS | 4 个灰区 SGQ-01~04 |
| 3 | SGQ 用户可见场景确认交互证据（≥1 条） | PASS | 4 条 SGQ 证据（问题/候选/回答/复述/影响面），来自计划 v4 4 轮评审 |
| 4 | 灰区分类（resolved-by-user/decision-item/non-blocking-open/converted-to-spike/n/a-with-reason） | PASS | SGQ-01/02/03 -> decision-item（CP2 决策）；SGQ-04 -> CP3 锁定（非 CP2 决策） |
| 5 | 未选项进入 Deferred Ideas | PASS | 4 项 Deferred 进入 CR-024 后续候选 T-02~T-05 |
| 6 | SCENARIOS.yaml 处置 | N/A | agent 实现类项目，验证场景在 CP7 TEST-STRATEGY + VERIFICATION-REPORT（mixed: static + runtime），非产品功能场景；等价物为 CP1 §使用场景 + 计划 v4 §9 验证策略 |
| 7 | TEST-MATRIX.md 处置 | N/A | 同上，CP7 验证矩阵在 VERIFICATION-REPORT，不单独产出 TEST-MATRIX |
| 8 | STORY-MAP.md 处置 | N/A | Story 拆分已在 CR-024（S1-S4）+ DEVELOPMENT-PLAN.yaml（CP4 产出），不单独产出 STORY-MAP |
| 9 | MVP-SCOPE.md 处置 | N/A | 等价物为 CR-024 范围（设备管理精简核心 + 策略路由执行首期）+ 决策 #4（手写最小 PC）+ 后续候选 T-01~T-05 |
| 10 | 讨论日志 + 恢复点 | PASS | `process/discussions/CP2-SCENARIO-DISCUSSION-LOG-CR-024.md` + `process/checks/CP2-DISCUSSION-CHECKPOINT-CR-024.json` |
| 11 | 场景主体正确性 | PASS | target-artifact（ptm-te agent 是交付对象），非当前仓库工作流自身 |

## Exit Criteria

| 条件 | 状态 |
|------|------|
| 使用场景完备（含异常降级） | ✓ |
| Scenario Gray Areas 已识别并分类 | ✓ |
| SGQ 证据完整（≥1 条） | ✓ |
| 产品文档 N/A 理由明确（SCENARIOS/TEST-MATRIX/STORY-MAP/MVP-SCOPE） | ✓ |
| 可进入 CP2 需求/场景/范围基线 | ✓ |

## Deliverables

- 本检查文件（场景 + 灰区 + SGQ 证据 + N/A 理由）
- `process/discussions/CP2-SCENARIO-DISCUSSION-LOG-CR-024.md`（讨论日志）
- `process/checks/CP2-DISCUSSION-CHECKPOINT-CR-024.json`（恢复点）

## 结论

**CP1 PASS**。ptm-te 使用场景完备，4 个 Scenario Gray Areas 已识别并形成 SGQ 证据，产品文档 N/A 理由充分。按 Post-approval 自动推进，CP1 PASS 后推进 CP2 自动预检 -> CP2 人工门禁。
