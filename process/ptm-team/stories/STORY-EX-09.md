---
cr_id: "CR-033"
story_id: "STORY-EX-09"
title: "retry 轮询（retry 字段解析 + 轮询执行）"
wave: 3
priority: "P1"
lld_policy: "technical-note"
feature_design_refs:
  - "docs/features/case-execution/DESIGN.md"
depends_on: ["ST-EX-04"]
dependency_type: "hard"
status: "lld-ready"
design_evidence_type: "technical-note"
lld_policy_required_level: "technical-note"
source_hld: "docs/design/HLD-PTM-TE-EXEC.md"
source_adr: "docs/design/ARCHITECTURE-DECISION-PTM-TE-EXEC.md"
source_requirements: "docs/product/REQUIREMENTS-PTM-TE-EXEC.md"
created_at: "2026-07-29T16:00:00+08:00"
author: "meta-dev"
---

# ST-EX-09 retry 轮询（retry 字段解析 + 轮询执行）

## 设计证据类型

technical-note（FE-EX-02 required，Feature 级 DESIGN.md 承载 case_runner 函数签名/字段格式；本 Story 内技术说明承载 retry 轮询逻辑的设计证据）。trigger_reasons：retry 字段解析 + 轮询逻辑；rationale：逻辑简单，Story 内技术说明足够。

## 设计依据

| 依据类型 | 引用 | 关联点 |
|---|---|---|
| HLD | §12.1 case_runner 执行流程 [5]（retry 字段 -> 轮询） | retry 在逐 step 执行中的位置 |
| HLD | §10 模块表（case_runner.py retry 字段） | 职责归属 case_runner |
| HLD | §18 下沉（case_steps YAML retry 字段格式 -> FE-EX-02 DESIGN.md） | 字段格式由 Feature DESIGN.md 承载 |
| 需求 | R-F-009（retry 字段 interval/max_attempts/success_condition 轮询） | 成功标准 |
| 开发计划 | DEVELOPMENT-PLAN.yaml Wave 3 ST-EX-09 | file_ownership=case_runner.py#apply_retry，depends_on=ST-EX-04 |
| Gotcha | Gotcha #9（TG op timeout 90s，DUT op 45s） | retry 间隔与 timeout 不冲突 |

## 文件影响范围

| 文件 | 归属段 | 变更类型 | 说明 |
|---|---|---|---|
| `skills/case-execution/scripts/case_runner.py` | `apply_retry` | 新增函数 | retry 字段解析 + 轮询执行 + success_condition 安全求值 |
| `skills/case-execution/scripts/case_runner.py` | `_eval_success_condition` | 新增内部函数 | 受限表达式求值（不使用 eval），支持 ==/!=/>/</>=/<= |
| `docs/features/case-execution/DESIGN.md` | retry 字段格式 + apply_retry 签名 | 补充章节 | 由 ST-EX-04 主导，本 Story 补 retry 段（CP5 后统一确认） |

**file_ownership 声明**（与 DEVELOPMENT-PLAN.yaml 一致）：
- `skills/case-execution/scripts/case_runner.py#apply_retry`

**不触碰文件**：`skills/policy-route-execution/scripts/op_mapper.py`（ST-EX-03/07 拥有）；op 的实际执行仍委托 op_mapper `execute_op`，retry 只在 case_runner 编排层轮询调用，不修改 op_mapper。

## 接口设计要点

### retry 字段格式（case_steps YAML，step 级字段）

retry 与 warming_up/known_issue 同为 step 级字段，位于 `atomic_op` 同级：

```yaml
- step_id: STEP-009
  step_name: 验证流量命中计数
  target: DUT
  atomic_op:
    op_id: fw_verify_policy_route_hitcount
    args: {id: ${STEP-008.id}}
  retry:
    interval: 3          # 轮询间隔（秒），必填
    max_attempts: 5      # 最大尝试次数（含首次），必填，>=1
    success_condition: "hitcount>0"   # 成功条件，必填
  expected_result: 命中计数 > 0
```

**字段约束**：
- `interval`：整数秒，>=1；默认 2s（缺省时）。sleep 在每次"未满足条件"后触发，最后一次不 sleep
- `max_attempts`：整数，>=1；含首次执行，即最多执行 max_attempts 次 op
- `success_condition`：字符串 `"field OP value"`，OP ∈ {`==`,`!=`,`>`,`<`,`>=`,`<=`}，field 从 `envelope.data` 提取；缺省时（无 retry 或无 success_condition）不轮询

**典型 op**：retry 通常作用于 verify 类 op（`fw_verify_policy_route_hitcount` / `tg_verify_traffic_loss`），对 mutation op（config/delete）无意义但语法不禁止（由用例作者负责语义正确）。

### apply_retry 签名

```python
def apply_retry(step: dict, execute_fn: Callable[[], dict], *, dry_run: bool = False) -> dict:
    """按 step.retry 配置轮询执行 op。

    在 case_runner 逐 step 执行流程中，execute_steps 检测到 step 含 retry 字段时
    调用本函数替代单次执行。execute_fn 是绑定好 op_id/args/env_topology 的执行回调，
    内部调用 op_mapper.execute_op 返回 envelope。

    Args:
        step: case_steps 解析出的单步 dict，含 retry 字段（interval/max_attempts/success_condition）
        execute_fn: 执行回调，签名 () -> envelope dict，封装 op_mapper.execute_op 调用
        dry_run: dry-run 模式下只执行一次，不轮询不 sleep（验证命令可构建 + success_condition 可解析）

    Returns:
        最终 envelope：
        - 任一次满足 success_condition -> 返回该次 envelope（status 保持原值，附加 retry_satisfied=true）
        - 全部不满足 -> 返回最后一次 envelope（附加 retry_satisfied=false, retry_attempts=N）
        - dry_run 模式 -> 返回单次 envelope（附加 retry_satisfied=N/A, retry_attempts=1）
    """
```

**返回值扩展**：apply_retry 在原 envelope 上附加 `retry_satisfied`（true/false/N/A）和 `retry_attempts`（实际执行次数）字段，供 classify_result（ST-EX-10）和 report.md（ST-EX-11）消费。不修改 envelope 原有 op_id/status/data/error_type 字段。

### success_condition 安全求值

```python
def _eval_success_condition(condition: str, envelope: dict) -> bool:
    """安全求值 success_condition，禁止使用 eval/exec。

    解析 "field OP value" 三元表达式：
    - field：从 envelope.data 提取（如 hitcount / rx_packets / loss_ratio）
    - OP：==/!=/>/</>=/<=
    - value：整数或字符串字面值

    field 不存在于 envelope.data 或类型不可比较时返回 False（不抛异常，由 retry 继续轮询）。
    condition 语法非法时返回 False 并记录 warning 到 envelope（不阻塞执行）。
    """
```

**安全约束**：严禁 `eval()`/`exec()`，避免代码注入。采用正则解析 + 受限比较，仅支持单层 `field OP value`，不支持嵌套表达式/逻辑运算符/函数调用。field 值为字符串时仅支持 `==`/`!=`；数值支持全部 6 个操作符。

### 与四态分级的交互

- retry 不改变四态分级逻辑（ST-EX-10 `classify_result` 仍按 known_issue 标记 + envelope.status 判定）
- retry 全部不满足时，envelope.status 仍为 op_mapper 返回的原值（如 FAIL），classify_result 按 known_issue 标记决定是 KNOWN_FAIL 还是 FAIL
- retry 满足条件时，envelope.status 保持原值（PASS），附加 `retry_satisfied=true`
- retry 不引入新状态，WARMING_UP 不是四态之一（warming_up 是 ST-EX-08 的 step 字段，控制 post_op 强制执行，非结果态）

### dry-run 行为

dry-run 模式下（HLD §13.3）：apply_retry 只执行一次 execute_fn（op_mapper dry_run=True 只构建命令不实际执行），不 sleep 不轮询；但仍解析 success_condition 语法（验证字段名合法、OP 合法），语法错误时记录 warning。目的：dry-run 校验用例 retry 配置正确性，不等待真实轮询。

## 实施步骤

1. **实现 `_eval_success_condition`**：正则解析 `^(\w+)\s*(==|!=|>=|<=|>|<)\s*(.+)$`，从 envelope.data 取 field 值，按 OP 比较；类型不匹配/field 缺失返回 False；非法语法返回 False + 记录 warning
2. **实现 `apply_retry`**：解析 step.retry（interval/max_attempts/success_condition，缺省 interval=2）；dry_run=True 时单次执行返回；dry_run=False 时循环 max_attempts 次，每次调 execute_fn 取 envelope，_eval_success_condition 为 True 则提前返回，否则 sleep(interval) 继续末次不 sleep；附加 retry_satisfied/retry_attempts 到返回 envelope
3. **接入 execute_steps**：ST-EX-04 的 execute_steps 逐 step 执行时，检测 step.get("retry") 非空则调 apply_retry(step, lambda: op_mapper.execute_op(...), dry_run=dry_run)，否则单次执行；本步在 ST-EX-04 主干完成后接入（depends_on=ST-EX-04 hard）
4. **result.json 记录**：retry_satisfied/retry_attempts 写入 step 结果，供 report.md 展示（ST-EX-11 消费）
5. **dry-run 校验**：dry-run 跑含 retry 的用例，验证 success_condition 语法解析 + 无实际 sleep

## 回滚策略

- 本 Story 为 case_runner.py 纯新增函数（apply_retry + _eval_success_condition），不修改 op_mapper，不改变既有 step 执行主干（接入点在 ST-EX-04 execute_steps 内的条件分支）
- 回滚方式：git revert 相关 commit；移除 apply_retry 后，execute_steps 回退到单次执行（step.retry 字段被忽略，不影响其他 step）
- 无运行时副作用：dry-run 模式不实际轮询；--execute 模式的 retry 仍受 ST-EX-05 授权门保护

## 权限与风险

- **权限约束**：遵循 NO_CREDENTIAL_READ / NO_PRODUCTION_WRITE / NO_EXTERNAL_PUBLISH；case_runner 不读取凭据，写操作经 op_mapper -> ptm-atomic 子进程，受 ST-EX-05 授权门保护
- **数据安全**：envelope/result.json 含执行数据（retry_satisfied/retry_attempts）但不含凭据；session 文件不入库
- **失败处理**：retry 全部不满足时 envelope.status 保持原值（FAIL），按 ST-EX-10 classify_result 四态判定；_eval_success_condition 异常返回 False 不崩溃
- **风险**：纯新增函数，无运行时副作用；回滚 git revert

## 测试要点

| 测试项 | 验证方法 | 预期结果 |
|---|---|---|
| retry 字段解析 | 单元测试：step 含 retry:{interval:3,max_attempts:5,success_condition:"rx_packets==0"} | apply_retry 正确解析三字段 |
| 首次满足即停 | 单元测试：mock execute_fn 首次返回 envelope.data.rx_packets=0 | retry_satisfied=true, retry_attempts=1，不 sleep |
| 全部不满足 | 单元测试：mock execute_fn 5 次均返回 rx_packets=100 | retry_satisfied=false, retry_attempts=5 |
| success_condition 数值比较 | 单元测试：条件 "hitcount>0"，data.hitcount=5 | True；data.hitcount=0 -> False |
| success_condition 字符串比较 | 单元测试：条件 "status==active"，data.status="active" | True |
| field 缺失 | 单元测试：条件 "rx_packets==0"，data 无 rx_packets | False，不抛异常 |
| 非法语法 | 单元测试：条件 "rx_packets"（无 OP） | False + warning 记录 |
| dry-run 单次 | dry-run 跑含 retry 用例 | retry_attempts=1, retry_satisfied=N/A，无 sleep |
| 无 eval 注入 | 代码审查 + grep "eval\|exec" | apply_retry/_eval_success_condition 无 eval/exec 调用 |
| timeout 不冲突 | 集成测试：retry interval=3 + TG op timeout=90s | retry 在 timeout 内完成，不叠加 |

## 开放项

| 问题 ID | 问题 | 状态 | 说明 |
|---|---|---|---|
| O-EX-09-01 | success_condition 是否需要支持 AND/OR 复合条件 | OPEN（非阻塞） | 当前只支持单层 `field OP value`；复合条件（如 `rx_packets==0 AND loss_ratio==0`）作为后续增强候选，CR-033 范围内单层足够（R-F-009 示例为单条件） |

## 与 HLD/ADR 追溯一致性

| HLD/ADR 条目 | 本 Story 落实点 |
|---|---|
| HLD §12.1 [5]（retry 字段 -> 轮询） | apply_retry 轮询执行 |
| HLD §10 模块表（case_runner.py retry） | apply_retry 归属 case_runner |
| HLD §18 下沉（retry 字段格式 -> FE-EX-02 DESIGN.md） | retry 字段格式 + apply_retry 签名补入 DESIGN.md |
| R-F-009（interval/max_attempts/success_condition 轮询） | 三字段解析 + 轮询循环 + 安全求值 |
| Gotcha #9（TG 90s / DUT 45s timeout） | retry interval 与 timeout 不冲突（间隔 < timeout） |
| HLD §13.3（dry-run 跳过 mutation 实际执行） | dry-run 单次执行不轮询，仅校验语法 |
