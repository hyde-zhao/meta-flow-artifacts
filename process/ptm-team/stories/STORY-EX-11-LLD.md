---
cr_id: "CR-033"
story_id: "STORY-EX-11"
story_name: "诊断与报告（失败诊断 + 结构化 report.md + 幂等容错记录）"
story_slug: diagnose-and-report
lld_policy: full-lld
evidence_type: full-lld
design_evidence_type: full-lld
lld_policy_required_level: full-lld
version: "1.0"
created_at: "2026-07-28T16:10:00+08:00"
author: "meta-dev"
hld_ref: "docs/design/HLD-PTM-TE-EXEC.md v1.6 §12.1[7] / §8 场景4 / UC-EX-06 / §13.4 幂等容错"
adr_ref: "docs/design/ARCHITECTURE-DECISION-PTM-TE-EXEC.md（诊断报告无独立 ADR，承袭 ADR-02 dry-run + 四态分级）"
development_plan_ref: "process/DEVELOPMENT-PLAN.yaml ST-EX-11"
feature_design_ref: "docs/features/case-execution/DESIGN.md（FE-EX-02，CP3 后生成）"
tier: "M"
wave: "W3"
dependencies: ["ST-EX-04", "ST-EX-10"]
dependency_type: "hard"
file_owners:
  - "skills/case-execution/scripts/case_runner.py#generate_report"
  - "skills/case-execution/scripts/case_runner.py#diagnose_failure"
shared_fragments: []
open_items: ["LCQ-ST-EX-11-01", "LCQ-ST-EX-11-02"]
status: "draft"
---

# LLD - ST-EX-11 诊断与报告（失败诊断 + 结构化 report.md + 幂等容错记录）（CR-033）

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-28 | meta-dev | CR-033 ST-EX-11 full-lld 初稿：diagnose_failure + generate_report + 幂等容错识别 + report.md 模板 + 14 章节完整 |

## ADR / HLD 追溯

| 来源 | 章节 | 决策/约束 | 本 LLD 落点 |
|---|---|---|---|
| HLD §12.1[7] | 生成 report.md | 四态统计表 + 失败 step 诊断（error_type/error_code/reason/details/command）+ 幂等容错记录 | §3 接口设计 / §4 核心流程 / §10 数据模型 |
| HLD §8 场景4 | known_issue 四态分级 | report.md 含 KNOWN_FAIL 统计和 DUT 行为差异说明 | §4 核心流程 / §10 数据模型 |
| HLD §13.3 | 幂等容错 | 对象已存在/流不存在/被引用阻止视为期望状态 | §3 接口设计 / §4 核心流程 |
| HLD §13.4 | 安全 | report.md 不含凭据 | §12 安全 |
| HLD §16.1 #7 | 失败自动诊断 | error_type/code/reason/details/command | §3 接口设计 |
| HLD §16.1 #11 | 结构化报告 | report.md 四态统计 + 诊断 + 容错 | §3 接口设计 |
| R-C-004 | 不做 HTML 报告 | report.md 是 Markdown | §3 接口设计 |
| UC-EX-06 | 失败诊断与四态分级 | known_issue 解析 + 失败诊断 + report.md + extract_payload | §4 核心流程 |

---

## 0. 概述

本 LLD 是 ST-EX-11 诊断与报告（失败诊断 + 结构化 report.md + 幂等容错记录）的 full-lld 设计证据，属于 FE-EX-02 case-execution 引擎 Feature 的 Wave 3 报告 Story，承载 case_runner.py 失败诊断与结构化报告生成。

**需求溯源**：ST-EX-11 承载 R-F-010（known_issue 四态分级，与 ST-EX-10 协同）、R-F-011（KNOWN_FAIL 不降级）、R-F-012（失败诊断 error_type/code/reason/details/command）、R-F-013（结构化 report.md）、R-F-014（extract_payload，ST-EX-12 承载）等功能需求；详细需求映射见 §1.1 需求来源。

**模块拆分**：case_runner.py 按职责拆分为 generate_report（report.md 生成）/diagnose_failure（失败诊断信息提取）函数锚点（ST-EX-11 拥有）；check_idempotent_tolerance（幂等容错识别）+ IDEMPOTENT_TOLERANCE_RULES 模块级常量；不改 op_mapper.py（envelope 结构既有）/24 用例 md（ST-EX-13 拥有）。

**代码结构**：详见 §2 文件影响范围。case_runner.py 新增 generate_report/diagnose_failure/check_idempotent_tolerance 函数 + IDEMPOTENT_TOLERANCE_RULES 常量；只读依赖 op_mapper envelope 结构 + ST-EX-10 classify_result 四态分级结果 + ST-EX-12 extract_payload。

**技术细节**：失败诊断五元组提取（§3.1，error_type/error_code/reason/details/command，消费 op_mapper envelope API）；幂等容错 3 类（§3.3，对象已存在/流不存在/被引用阻止，视为期望状态 idempotent_skip）；report.md 四态统计表（§4.3，PASS/FAIL/KNOWN_FAIL/ERROR）；与 ST-EX-10 四态分级协同（§6.4）；R-C-004 不做 HTML 报告等核心技术细节在各章节展开。

**Definition of Done (DoD)**：CP6 实现完成以 §6 测试设计的 8 单元测试（UT-11-01..08）+ dry-run mock report.md 生成全通过为准；CP7 验证采用 static-only + dry-run-only（DQ-01 推荐）。DoD 详见 §6.2/§6.3/§6.4 测试分层与协同测试。

## 1. 工程依据

### 1.1 需求来源

ST-EX-11 承载 CR-033 改进 #7（失败自动诊断）+ #11（结构化报告）（HLD §16.1 追溯表）。

- 关联需求：R-F-010（known_issue 四态分级，与 ST-EX-10 协同）、R-F-011（KNOWN_FAIL 不降级）、R-F-012（失败诊断 error_type/code/reason/details/command）、R-F-013（结构化 report.md）、R-F-014（extract_payload，ST-EX-12 承载）
- 关联场景：SCN-EX-09（known_issue 四态分级）、SCN-EX-10（失败诊断）、SCN-EX-21（幂等容错）
- 关联 UC：UC-EX-06

### 1.2 问题陈述

exec_v4.py 时代结果只有 PASS/FAIL 二态，失败原因不可审计（只有"失败"二字，不知是 DUT 行为差异、脚本 bug 还是环境问题）。幂等操作（如重复 config 已存在对象）被误判为 FAIL。

### 1.3 设计目标

- 失败诊断：从失败 envelope 提取 error_type/error_code/reason/details/command 五元组，结构化记录
- 结构化报告：report.md 含四态统计表（PASS/FAIL/KNOWN_FAIL/ERROR）+ 失败 step 诊断列表 + 幂等容错记录
- 幂等容错：对象已存在/流不存在/被引用阻止视为期望状态，step 标记 idempotent_skip，不降级用例结果
- 四态协同：与 ST-EX-10 四态分级协同，KNOWN_FAIL 在 report.md 单独统计

### 1.4 量化验收

| 验收项 | 目标值 | 验证方式 |
|---|---|---|
| 结果分级 | 4 态（SM-EX-08） | report.md 含 PASS/FAIL/KNOWN_FAIL/ERROR 四列 |
| 失败诊断字段 | 5 元组（error_type/code/reason/details/command） | dry-run 构造失败用例检查 report.md |
| 幂等容错 | 3 类（对象已存在/流不存在/被引用阻止） | dry-run 构造幂等场景检查 idempotent_skip |

### 1.5 非目标

- 不做 HTML 报告（R-C-004，进 BACKLOG）
- 不实现 extract_payload 字段提取表（ST-EX-12 承载，diagnose_failure 调用）
- 不实现四态分级判定逻辑（ST-EX-10 承载 classify_result，ST-EX-11 消费其结果）
- 不采集 TG 系统快照（R-C-006）

---

## 2. 文件影响范围

### 2.1 文件所有权（DEVELOPMENT-PLAN ST-EX-11）

| 文件 | 符号 | 操作 | 说明 |
|---|---|---|---|
| skills/case-execution/scripts/case_runner.py | generate_report | 新增 | report.md 生成 |
| skills/case-execution/scripts/case_runner.py | diagnose_failure | 新增 | 失败诊断信息提取 |

### 2.2 只读依赖（不修改）

| 文件 | 来源 Story | 用途 |
|---|---|---|
| op_mapper.py execute_op envelope 结构 | ST-EX-03/04 | envelope.data/error_type 是诊断输入 |
| case_runner.py classify_result | ST-EX-10 | 四态分级结果（ST-EX-11 消费） |
| case_runner.py extract_payload | ST-EX-12 | 按 op_id 提取 envelope.data 字段（诊断 details 来源） |
| case_runner.py run_case 主循环 | ST-EX-04 | step 执行结果收集 |

### 2.3 不触碰文件

- op_mapper.py（envelope 结构既有，不改）
- 24 用例 md（ST-EX-13 拥有）

---

## 3. 接口设计

### 3.1 diagnose_failure 函数签名

```python
def diagnose_failure(
    step: dict,
    envelope: dict,
    *,
    command: Optional[List[str]] = None,
) -> dict:
    """从失败 step 的 envelope 提取结构化诊断信息。

    提取五元组（HLD §16.1 #7）：
    - error_type: envelope.error_type（OP_NOT_FOUND/PARAM_INVALID/VALIDATION_FAILED/
                  EXEC_FAILED/STATE_INVALID/UNKNOWN_ERROR）
    - error_code: envelope.data.error_code（ptm-atomic 业务错误码，如 eBeingReferenced/
                  eObjectExists/stream_not_found；无业务码时为 ""）
    - reason: envelope.data.reason 或 envelope.data.message（人类可读原因）
    - details: extract_payload(op_id, envelope) 提取的 op 特定字段（ST-EX-12）+
               envelope.data 原始 dump（截断 500 字符）
    - command: 触发失败的 ptm-atomic 命令（从 exec-log 或调用方传入）

    Args:
        step: 失败 step dict（含 step_name/op_id/args）
        envelope: op_mapper.execute_op 返回的 envelope dict
        command: 触发失败的命令（可选，None 时从 envelope.data.command 取）

    Returns:
        诊断 dict：
        {
            "step_name": str,
            "op_id": str,
            "error_type": str,
            "error_code": str,
            "reason": str,
            "details": dict,
            "command": list[str] | str,
        }
    """
```

### 3.2 generate_report 函数签名

```python
def generate_report(
    run_result: dict,
    *,
    output_path: str,
) -> str:
    """生成 report.md 结构化报告（HLD §12.1[7]）。

    报告内容：
    1. 四态统计表（PASS/FAIL/KNOWN_FAIL/ERROR 计数 + 百分比）
    2. 失败 step 诊断列表（每个 FAIL/ERROR step 的五元组诊断）
    3. 幂等容错记录列表（每个 idempotent_skip step 的容错类型 + 原始 error_code）
    4. KNOWN_FAIL DUT 行为差异说明（known_issue step 的 DUT 行为 + 用例标注理由）
    5. runtime_authorization 审计摘要（--execute 模式）

    Args:
        run_result: 全量运行结果 dict，含：
            - cases: list[dict]，每个 case 含
              - case_id/case_name/overall_status(PASS/FAIL/KNOWN_FAIL/ERROR)
              - steps: list[dict]，每个 step 含 status/idempotent_skip/known_issue/
                       diagnosis(失败时)/warming_up/auto_post_op
            - summary: {total/pass/fail/known_fail/error}
            - runtime_authorization: dict | None
        output_path: report.md 输出路径（runs/<run-id>/report.md）

    Returns:
        report.md 文件路径
    """
```

### 3.3 幂等容错识别契约

```python
# 幂等容错规则表（HLD §13.3）
IDEMPOTENT_TOLERANCE_RULES: List[dict] = [
    {
        "name": "object_already_exists",
        "op_ids": ["fw_config_object"],
        "error_codes": ["eObjectExists", "already_exists", "object_exists"],
        "reason_substr": ["已存在", "already exists", "exists"],
        "tolerance": "idempotent_skip",  # 视为期望状态
    },
    {
        "name": "stream_not_found",
        "op_ids": ["tg_stop_traffic_stream"],
        "error_codes": ["stream_not_found", "not_found"],
        "reason_substr": ["不存在", "not found", "no such stream"],
        "tolerance": "idempotent_skip",
    },
    {
        "name": "object_in_use",
        "op_ids": ["fw_delete_object"],
        "error_codes": ["eBeingReferenced", "in_use", "referenced"],
        "reason_substr": ["被引用", "referenced", "in use"],
        "tolerance": "idempotent_skip",
    },
]


def check_idempotent_tolerance(
    op_id: str,
    envelope: dict,
) -> Optional[dict]:
    """检查失败 envelope 是否匹配幂等容错规则。

    匹配逻辑（任一命中即视为容错）：
    - error_code 命中规则 error_codes 列表（精确匹配，大小写不敏感）
    - reason 含规则 reason_substr 任一子串（大小写不敏感）
    - 且 op_id 在规则 op_ids 列表中

    Returns:
        匹配时返回 {rule_name, tolerance, original_error_code}；
        不匹配返回 None（非幂等容错，按真实失败处理）
    """
```

### 3.4 调用契约

| 调用方 | 调用时机 | 输入 | 输出 | 后续衔接 |
|---|---|---|---|---|
| case_runner.run_case 逐 step | step envelope.status != success 时 | step + envelope | diagnosis dict | 存入 step 记录，供 generate_report |
| case_runner.run_case 逐 step | 失败时先 check_idempotent_tolerance | op_id + envelope | 容错判定 | 命中则 step.idempotent_skip=true，不调 diagnose_failure |
| case_runner.run（全量后） | 全部用例执行完 | run_result | report.md | 写 runs/<run-id>/report.md |

---

## 4. 核心流程

### 4.1 逐 step 诊断 + 幂等容错流程

```
case_runner.run_case 逐 step 执行
  │
  ├─ step 执行返回 envelope
  │
  ├─ [1] envelope.status == "success"？
  │   └─ 是：step.status=PASS，继续下一 step
  │
  ├─ [2] 否（失败）：先 check_idempotent_tolerance(op_id, envelope)
  │   ├─ 命中幂等容错规则：
  │   │   ├─ step.idempotent_skip = true
  │   │   ├─ step.idempotent_rule = <rule_name>
  │   │   ├─ step.status = PASS（视为期望状态，不降级）
  │   │   ├─ step.original_error_code = envelope.data.error_code
  │   │   └─ 不调 diagnose_failure（容错无需诊断）
  │   └─ 未命中（真实失败）：
  │       ├─ step.diagnosis = diagnose_failure(step, envelope, command)
  │       ├─ known_issue=true（ST-EX-10）？ -> step.status=KNOWN_FAIL
  │       │   └─ 否 -> step.status=FAIL
  │       └─ overall 由 ST-EX-10 classify_result 汇总
  │
  └─ 全部 step 完成后，step 记录存入 run_result
```

### 4.2 generate_report 报告生成流程

```
case_runner.run 全部用例执行完
  │
  ├─ [1] 汇总 run_result
  │   ├─ summary = {total, pass, fail, known_fail, error}
  │   ├─ cases = [{case_id, case_name, overall_status, steps:[...]}]
  │   └─ runtime_authorization（--execute 模式）
  │
  ├─ [2] generate_report(run_result, output_path)
  │   ├─ 渲染四态统计表
  │   ├─ 遍历 cases，收集 status in (FAIL, ERROR) 的 step -> 失败诊断列表
  │   ├─ 遍历 cases，收集 idempotent_skip=true 的 step -> 幂等容错记录
  │   ├─ 遍历 cases，收集 known_issue=true 的 step -> KNOWN_FAIL DUT 行为说明
  │   └─ 写 report.md（Markdown）
  │
  └─ [3] 输出 runs/<run-id>/report.md
```

### 4.3 report.md 模板

```markdown
# 测试执行报告 - <run-id>

### 1. 四态统计

| 状态 | 数量 | 百分比 |
|---|---|---|
| PASS | <n> | <%> |
| FAIL | <n> | <%> |
| KNOWN_FAIL | <n> | <%> |
| ERROR | <n> | <%> |
| **合计** | <total> | 100% |

### 2. 失败 step 诊断

#### <case-id> - <case-name> [overall: FAIL]

| step | op_id | error_type | error_code | reason |
|---|---|---|---|---|
| STEP-005 | fw_config_policy_route | VALIDATION_FAILED | | next_hop_ip 非合法 IPv4 |

**details**: <extract_payload 提取字段 + data dump>
**command**: `ptm-atomic run --base-url ... policy-route config ...`

（每个失败 step 重复）

### 3. 幂等容错记录

| case | step | op_id | 容错规则 | 原始 error_code |
|---|---|---|---|---|
| PC-M4-01-09 | STEP-007 | fw_delete_object | object_in_use | eBeingReferenced |

（说明：对象被引用阻止删除，符合期望状态，step 标记 idempotent_skip）

### 4. KNOWN_FAIL DUT 行为差异说明

| case | step | op_id | DUT 行为 | 用例标注理由 |
|---|---|---|---|---|
| PC-M4-01-09 | STEP-007 | fw_delete_object | eBeingReferenced 阻止删除 | 删除被引用对象，DUT 行为符合预期 |

### 5. runtime_authorization 审计（--execute 模式）

- who: <user>
- scope: <op_id> on <base_url>
- authorized_at: <timestamp>
- reason: dry_run=False 用户单次授权
```

---

## 5. 异常处理

### 5.1 诊断阶段异常

| 异常场景 | 处理 | 说明 |
|---|---|---|
| envelope.data 缺 error_code 字段 | error_code="" | 部分错误无业务码（如 EXEC_FAILED 超时） |
| envelope.data 缺 reason/message | reason="未知原因" | 兜底 |
| extract_payload 返回空（ST-EX-12 未匹配 op_id） | details=envelope.data 原始 dump | 兜底用原始 data |
| command 缺失 | command=envelope.data.command 或 "" | exec-log 有则取 |
| envelope 结构异常（非 dict） | diagnosis={error_type:"UNKNOWN_ERROR", reason:"envelope 结构异常"} | 兜底 |

### 5.2 报告生成异常

| 异常场景 | 处理 | 说明 |
|---|---|---|
| output_path 目录不存在 | os.makedirs 创建 | 与 exec-log 一致 |
| report.md 写入失败 | stderr 警告 + run 继续不中断 | 报告是副产物，不阻断 result.json |
| run_result 结构异常 | 报告写"结果异常，无法生成统计" | 兜底 |

### 5.3 幂等容错误判风险

幂等容错将"失败"视为"期望状态"，存在误判风险（真实失败被容错掩盖）。缓解：
- 容错规则严格限定 op_id + error_code/reason 双重匹配（非泛化）
- report.md §3 完整记录所有 idempotent_skip（可审计）
- 容错规则表模块级常量，单点维护，可审查

---

## 6. 测试设计

### 6.1 测试分层

| 层级 | 范围 | 方式 | 覆盖场景 |
|---|---|---|---|
| 单元测试 | diagnose_failure / check_idempotent_tolerance / generate_report 纯函数 | Python assert | §6.2 |
| dry-run 集成 | report.md 生成 | case_runner dry-run + mock envelope | §6.3 |

### 6.2 单元测试用例

| 用例 ID | 输入 | 预期 | 覆盖 |
|---|---|---|---|
| UT-11-01 | envelope={status:error, error_type:VALIDATION_FAILED, data:{reason:"next_hop 非法"}} | diagnosis.error_type=VALIDATION_FAILED, reason 含"next_hop 非法" | 诊断提取 |
| UT-11-02 | fw_config_object + error_code=eObjectExists | check_idempotent_tolerance 返回 object_already_exists | 对象已存在容错 |
| UT-11-03 | tg_stop_traffic_stream + reason="stream not found" | 容错命中 stream_not_found | 流不存在容错 |
| UT-11-04 | fw_delete_object + error_code=eBeingReferenced | 容错命中 object_in_use | 被引用阻止容错 |
| UT-11-05 | fw_config_policy_route + VALIDATION_FAILED | 容错不命中（真实失败） | 非幂等失败 |
| UT-11-06 | run_result 含 4 态 | report.md 含四态统计表 + 诊断 + 容错 + KNOWN_FAIL 说明 | 报告生成 |
| UT-11-07 | envelope.data 缺 error_code | diagnosis.error_code="" | 缺字段兜底 |
| UT-11-08 | envelope 非 dict | diagnosis.error_type=UNKNOWN_ERROR | 结构异常兜底 |

### 6.3 dry-run 集成测试

构造 mock 失败 envelope（不连真实设备），dry-run 模式下 case_runner 收集 step 结果并生成 report.md，校验：
- 四态统计表计数正确
- 失败诊断含五元组
- 幂等容错记录含规则名 + 原始 error_code
- KNOWN_FAIL 说明含 DUT 行为

### 6.4 与 ST-EX-10 协同测试

- ST-EX-10 classify_result 判定 KNOWN_FAIL，ST-EX-11 generate_report 在报告 §1 统计 KNOWN_FAIL 列、§4 展开说明
- 构造 known_issue=true 的失败 step，验证 report.md §4 含 DUT 行为差异说明

---

## 7. 实施步骤

| 步骤 | 操作 | 文件 | 验证 |
|---|---|---|---|
| 1 | 定义 IDEMPOTENT_TOLERANCE_RULES 模块级常量 | case_runner.py | UT-11-02..05 |
| 2 | 实现 check_idempotent_tolerance 函数 | case_runner.py | UT-11-02..05 |
| 3 | 实现 diagnose_failure 函数 | case_runner.py | UT-11-01/07/08 |
| 4 | 实现 generate_report 函数 + report.md 模板 | case_runner.py | UT-11-06 |
| 5 | run_case 逐 step 集成：失败时先容错判定，再诊断 | case_runner.py#run_case | dry-run mock |
| 6 | run 全量后调用 generate_report | case_runner.py#run | dry-run 集成 |
| 7 | 与 ST-EX-10 classify_result 协同（KNOWN_FAIL 统计） | - | §6.4 协同测试 |

### 7.1 完成准则

- diagnose_failure / check_idempotent_tolerance / generate_report 实现 + 8 个单元测试通过
- dry-run mock 场景 report.md 含四态统计 + 诊断 + 容错 + KNOWN_FAIL 说明
- report.md 不含凭据（安全）
- CP6 实现执行证据闭环

---

## 8. 回滚策略

### 8.1 代码回滚

ST-EX-11 新增 diagnose_failure / generate_report / check_idempotent_tolerance + run_case 集成点。回滚 = 移除函数 + 移除 run_case 中容错判定和诊断调用 + 移除 run 中 generate_report 调用。回滚后 case_runner 退化为"只输出 result.json 无 report.md"，与 exec_v4.py 时代一致。

### 8.2 幂等容错回退

若幂等容错误判真实失败（掩盖 bug），可禁用 check_idempotent_tolerance 调用（保留函数定义，注释禁用）。回退后所有失败按真实 FAIL/ERROR 处理，report.md §3 幂等容错记录为空。

### 8.3 report.md 格式回退

若 report.md 模板需调整，generate_report 是单点，修改模板不影响 result.json（result.json 是真相源，report.md 是人类可读视图）。

---

## 9. 平台差异

| 平台 | 适配点 | 说明 |
|---|---|---|
| ptm-team canonical | skill 源 | diagnose_failure/generate_report 源在 case_runner.py |
| ptm-te workspace | skill 安装目标 | install.py 安装到 .claude/skills/case-execution/scripts/ |
| ptm-atomic CLI | 不改本体 | 诊断从 envelope 提取，不改 ptm-atomic 输出 |

无 Claude Code / Codex / Qoder 平台差异--纯 Python 函数 + Markdown 输出。

---

## 10. 数据模型

### 10.1 诊断 diagnosis dict 结构

```python
{
    "step_name": "STEP-005",
    "op_id": "fw_config_policy_route",
    "error_type": "VALIDATION_FAILED",
    "error_code": "",                 # 业务错误码，无则空串
    "reason": "next_hop_ip 值 'xxx' 非合法 IPv4",
    "details": {                      # extract_payload 提取 + data dump
        "next_hop_ip": "xxx",
        "_raw_data": "{...}"[:500],   # 截断
    },
    "command": ["ptm-atomic", "run", "--base-url", "...", "policy-route", "config", ...],
}
```

### 10.2 step 记录（run_result.cases[].steps[]）

```python
{
    "step_name": "STEP-007",
    "op_id": "fw_delete_object",
    "status": "PASS",                 # 幂等容错视为 PASS
    "idempotent_skip": true,
    "idempotent_rule": "object_in_use",
    "original_error_code": "eBeingReferenced",
    "known_issue": false,
    "warming_up": false,
    "auto_post_op": false,
    "diagnosis": null,                # 容错不诊断；真实失败时填
}
```

### 10.3 真实失败 step 记录

```python
{
    "step_name": "STEP-005",
    "op_id": "fw_config_policy_route",
    "status": "FAIL",
    "idempotent_skip": false,
    "known_issue": false,
    "diagnosis": { /* §10.1 */ },
}
```

### 10.4 report.md 数据流

```
envelope (op_mapper) 
  -> check_idempotent_tolerance (容错判定)
  -> diagnose_failure (真实失败诊断)
  -> step 记录 (run_result)
  -> generate_report (report.md)
```

---

## 11. 性能

### 11.1 诊断开销

- diagnose_failure 是纯内存提取，O(1)，开销可忽略
- check_idempotent_tolerance 遍历 3 条规则，O(1)

### 11.2 报告生成开销

- generate_report 遍历全部 case + step，O(N) N=总 step 数（24 用例 × ~10 step = 240）
- Markdown 渲染是字符串拼接，开销 <10ms

### 11.3 无性能风险

诊断与报告均在用例执行后（或单 step 失败时）触发，不增加用例执行时间。report.md 生成在全量结束后一次性完成。

---

## 12. 安全

### 12.1 凭据安全

- diagnose_failure 提取的 details 含 envelope.data dump，需确保 envelope.data 不含凭据
- op_mapper envelope 不含密码（fw_login 用 password_env 传环境变量名，不传密码值）
- report.md 脱敏：command 列表中 `--password-env FW_WEB_PASSWORD` 是环境变量名，非密码值，可输出

### 12.2 安全约束遵循

| 约束 | 遵循 |
|---|---|
| NO_CREDENTIAL_READ | 是，诊断不读凭据，envelope 无密码 |
| NO_PRODUCTION_WRITE | 是，report.md 写本地 runs/ 目录 |
| NO_EXTERNAL_PUBLISH | 是，报告不外发 |
| R-C-004 不做 HTML | 是，report.md 是 Markdown |

### 12.3 command 输出脱敏

command 列表含 `--base-url https://<dut-ip>`（IP 非凭据，可输出）；`--password-env <env-name>`（环境变量名，非密码值，可输出）。若 envelope.data 意外含密码字段，diagnose_failure 的 details dump 需过滤 `password`/`secret`/`token` key（黑名单脱敏）。

---

## 13. 依赖

### 13.1 Story 依赖

| 依赖 Story | 类型 | 依赖内容 | 门控 |
|---|---|---|---|
| ST-EX-04 | hard | case_runner.py 主框架 + run_case 主循环 + run 全量收集 | dev_gate=cp5-approved |
| ST-EX-10 | hard | classify_result 四态分级结果（KNOWN_FAIL 判定） | lld_gate=technical-note |
| ST-EX-12 | soft | extract_payload 按 op_id 提取 envelope.data 字段（details 来源） | lld_gate=technical-note |

### 13.2 模块依赖

| 模块 | 依赖方向 | 契约 |
|---|---|---|
| op_mapper.execute_op envelope | case_runner 读 | envelope={op_id, step_name, status, data, error_type, diag_snapshot_ref} |
| op_mapper._build_envelope | case_runner 读 | data 含 reason/error_code（ptm-atomic 返回） |

### 13.3 假设

- ptm-atomic envelope.data 含 error_code 字段（业务错误码）；若实际不含，check_idempotent_tolerance 退化为 reason 子串匹配（兜底）
- envelope.data 含 reason 或 message 字段（人类可读原因）；实际字段名待 LCQ-ST-EX-11-01 确认

---

## 14. 开放项

| 问题 ID | 问题 | 状态 | 影响 | 处理 |
|---|---|---|---|---|
| LCQ-ST-EX-11-01 | ptm-atomic envelope.data 中业务错误码字段名是 `error_code` 还是 `code`？eBeingReferenced/eObjectExists 的确切字符串值？ | OPEN（已写 QUESTION-LEDGER） | 影响幂等容错 error_codes 匹配精度 | 待 host-orchestrator 批量问用户；推荐方案：check_idempotent_tolerance 同时检查 error_code/code 两个字段，error_codes 列表涵盖大小写变体；reason 子串兜底 |
| LCQ-ST-EX-11-02 | 幂等容错规则是否需要扩展到 fw_config_interface（接口已存在）和 fw_config_policy_route（策略已存在）？ | OPEN（已写 QUESTION-LEDGER） | 影响容错覆盖范围 | 推荐：CR-033 先覆盖 3 类（对象已存在/流不存在/被引用阻止），其他 op 的幂等容错进 BACKLOG，按需扩展 |
| O-03 | 24 用例 known_issue 标注完整性 | OPEN（ST-EX-13 整改时同步检查） | 影响 KNOWN_FAIL 统计准确性 | ST-EX-13 整改时校验 |

### 14.1 clarification 记录

LCQ-ST-EX-11-01 / LCQ-ST-EX-11-02 已写入 `process/state/QUESTION-LEDGER.ndjson`。LCQ-ST-EX-11-01 不阻塞 LLD（推荐方案：双字段检查 + reason 兜底，已在 §3.3 设计中体现）。LCQ-ST-EX-11-02 不阻塞（推荐先 3 类，其他进 BACKLOG）。待 host-orchestrator 批量问用户确认。
