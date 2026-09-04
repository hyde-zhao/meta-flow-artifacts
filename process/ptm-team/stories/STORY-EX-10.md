---
cr_id: "CR-033"
story_id: "STORY-EX-10"
title: "四态分级（known_issue 标记 + 四态判定 PASS/FAIL/KNOWN_FAIL/ERROR）"
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

# ST-EX-10 四态分级（known_issue 标记 + 四态判定 PASS/FAIL/KNOWN_FAIL/ERROR）

## 设计证据类型

technical-note（FE-EX-02 required，Feature 级 DESIGN.md 承载四态分级判定矩阵；本 Story 内技术说明承载 classify_result 判定逻辑的设计证据）。trigger_reasons：known_issue 标记 + 四态判定；rationale：逻辑简单，Story 内技术说明足够。

## 设计依据

| 依据类型 | 引用 | 关联点 |
|---|---|---|
| HLD | §12.1 [5]（known_issue 标记 -> KNOWN_FAIL）+ [7]（report.md 四态统计表） | 四态在执行/报告流程中的位置 |
| HLD | §8 场景 4（known_issue 四态分级 SCN-EX-09） | KNOWN_FAIL 判定流程 |
| HLD | §18 下沉（四态分级判定矩阵 -> FE-EX-02 DESIGN.md） | 判定矩阵由 Feature DESIGN.md 承载 |
| HLD | §21 自审（四态分级 ISTQB 测试结果分级） | 理论依据 ISTQB |
| 需求 | R-F-010（known_issue 标记失败 -> KNOWN_FAIL，未标记 -> FAIL） | known_issue 与 FAIL 区分 |
| 需求 | R-F-011（四态 PASS / FAIL / KNOWN_FAIL / ERROR） | overall 字段四态枚举 |
| 决策 | DQ-CP3-03（24 用例 known_issue 标注完整性纳入 R-F-021 ARP 整改同步检查） | known_issue 标注由 ST-EX-13 同步检查 |
| 开发计划 | DEVELOPMENT-PLAN.yaml Wave 3 ST-EX-10 | file_ownership=case_runner.py#classify_result，depends_on=ST-EX-04 |

## 文件影响范围

| 文件 | 归属段 | 变更类型 | 说明 |
|---|---|---|---|
| `skills/case-execution/scripts/case_runner.py` | `classify_result` | 新增函数 | step 级四态判定 + overall 聚合 |
| `skills/case-execution/scripts/case_runner.py` | `classify_overall` | 新增内部函数 | 用例级 overall 四态聚合（ERROR>FAIL>KNOWN_FAIL>PASS） |
| `docs/features/case-execution/DESIGN.md` | 四态分级判定矩阵 | 补充章节 | 由 ST-EX-04 主导，本 Story 补判定矩阵段 |

**file_ownership 声明**（与 DEVELOPMENT-PLAN.yaml 一致）：
- `skills/case-execution/scripts/case_runner.py#classify_result`

**不触碰文件**：`skills/policy-route-execution/scripts/op_mapper.py`（envelope 由 op_mapper 返回，case_runner 只消费不修改）；`ptm-te/cases/**/*.md`（known_issue 标注由 ST-EX-13 整改，本 Story 只解析不标注）。

## 接口设计要点

### 四态定义（R-F-011，ISTQB 结果分级）

| 状态 | 含义 | 触发条件 |
|---|---|---|
| PASS | step 执行成功 | envelope.status=PASS/SUCCESS，无业务错误 |
| FAIL | step 业务失败（DUT 行为不符合预期，无 known_issue 标记） | envelope.status=FAIL + 业务错误 + step.known_issue 未标记 |
| KNOWN_FAIL | step 业务失败但属已知 DUT 行为差异（known_issue 标记） | envelope.status=FAIL + 业务错误 + step.known_issue=true |
| ERROR | 执行异常（脚本 bug / 认证失败 / 超时 / 参数校验失败） | envelope.error_type ∈ {EXEC_FAILED, AUTH_FAILED, TIMEOUT, VALIDATION_FAILED} 或 envelope.status=ERROR |

**状态命名说明**（LCQ-ST-EX-10-01）：四态为 PASS/FAIL/KNOWN_FAIL/ERROR（HLD §12.1/§13 + CR-033 决策表 + R-F-011 三处一致）。`known_issue` 是 step 字段名（`known_issue:true`），`warming_up` 是 ST-EX-08 的 step 字段名（控制 post_op 强制执行），二者均非四态结果名。WARMING_UP 不是结果态。

### classify_result 判定矩阵

```python
# 业务错误 error_type（DUT 行为差异，可 known_issue）
_BUSINESS_ERROR_TYPES = {"eBeingReferenced", "eObjectNotFound", "eObjectExists",
                         "ePolicyNotMatched", "ePortInUse", "BUSINESS_FAILED"}
# 执行异常 error_type（脚本/环境问题，不可 known_issue 豁免）
_EXEC_ERROR_TYPES = {"EXEC_FAILED", "AUTH_FAILED", "TIMEOUT", "VALIDATION_FAILED",
                     "STATE_INVALID", "SUBPROCESS_ERROR"}

def classify_result(step: dict, envelope: dict) -> str:
    """判定单 step 的四态结果。

    Args:
        step: case_steps 解析出的单步 dict，可能含 known_issue: true
        envelope: op_mapper.execute_op 返回的 envelope dict
            （含 status / error_type / data）

    Returns:
        "PASS" | "FAIL" | "KNOWN_FAIL" | "ERROR"
    """
```

| envelope.status | envelope.error_type | step.known_issue | 结果 |
|---|---|---|---|
| PASS / SUCCESS | 空 / None | 任意 | PASS |
| FAIL | ∈ _BUSINESS_ERROR_TYPES | true | KNOWN_FAIL |
| FAIL | ∈ _BUSINESS_ERROR_TYPES | false/缺省 | FAIL |
| FAIL | ∈ _EXEC_ERROR_TYPES | 任意 | ERROR（执行异常不可 known_issue 豁免） |
| FAIL | 其他/未知 error_type | true | KNOWN_FAIL（保守归为业务失败，允许 known_issue） |
| FAIL | 其他/未知 error_type | false | FAIL |
| ERROR | 任意 | 任意 | ERROR |
| 空/异常 | 任意 | 任意 | ERROR（envelope 异常，脚本 bug） |

**判定原则**：
- 业务失败（DUT 行为差异）才允许 known_issue 豁免为 KNOWN_FAIL；执行异常（脚本/环境/认证）一律 ERROR，known_issue 标记不豁免
- 未知 error_type 保守归为业务失败（FAIL/KNOWN_FAIL），避免未知错误被误判为 ERROR 掩盖 DUT 行为问题
- envelope 缺失/异常（如 None、无 status 字段）归为 ERROR（脚本 bug 或 op_mapper 异常）

### classify_overall 用例级聚合

```python
# 严重度优先级：ERROR > FAIL > KNOWN_FAIL > PASS
_SEVERITY_ORDER = {"ERROR": 4, "FAIL": 3, "KNOWN_FAIL": 2, "PASS": 1}

def classify_overall(step_results: list) -> str:
    """聚合用例级 overall 四态。

    Args:
        step_results: 各 step 的 classify_result 结果列表

    Returns:
        overall 四态：取最高严重度
        - 全部 PASS -> PASS
        - 有 KNOWN_FAIL 但无 FAIL/ERROR -> KNOWN_FAIL
        - 有 FAIL 但无 ERROR -> FAIL
        - 有 ERROR -> ERROR
    """
```

**overall 语义**（HLD 场景 4）：known_issue step 不降级 overall 为 FAIL，记为 KNOWN_FAIL，report.md 含 KNOWN_FAIL 统计和 DUT 行为差异说明。

### 与 retry / warming_up 的交互

- **retry（ST-EX-09）**：retry 轮询后 envelope.status 仍为 op_mapper 原值。retry 全部不满足 -> status=FAIL -> classify_result 按 known_issue 判定 FAIL/KNOWN_FAIL；retry 满足 -> status=PASS -> PASS。retry_satisfied 字段不影响四态判定，仅记录在 result.json 供报告展示
- **warming_up（ST-EX-08）**：warming_up 是 step 字段控制 post_op 强制执行，不参与四态判定。warming_up step 的执行结果仍按 classify_result 判定四态
- **幂等容错**：op_mapper 幂等容错（对象已存在/流不存在/被引用阻止）返回 status=PASS（视为期望状态），classify_result 判定 PASS；不触发 known_issue

### result.json 四态字段

```json
{
  "case_id": "PC-COMB-M4-01-01",
  "overall": "KNOWN_FAIL",
  "steps": [
    {"step_id": "STEP-007", "result": "KNOWN_FAIL", "known_issue": true,
     "error_type": "eBeingReferenced", "retry_satisfied": "N/A"},
    ...
  ],
  "stats": {"PASS": 9, "FAIL": 0, "KNOWN_FAIL": 1, "ERROR": 0}
}
```

`overall` 字段枚举严格为四态之一（R-F-011），`stats` 四态计数供 report.md（ST-EX-11）统计表消费。

## 实施步骤

1. **定义错误类型常量**：`_BUSINESS_ERROR_TYPES` / `_EXEC_ERROR_TYPES` 两个集合常量，覆盖 op_mapper 现有 error_type；未知 error_type 归业务失败（保守）
2. **实现 `classify_result`**：按判定矩阵逐条判定，输入 step + envelope，输出四态字符串；envelope 缺失/异常归 ERROR
3. **实现 `classify_overall`**：取 step_results 最高严重度；空列表归 PASS（无 step 的空用例）
4. **接入 execute_steps**：ST-EX-04 execute_steps 每个 step 执行后调 classify_result 记录到 step 结果；用例结束调 classify_overall 写入 result.json overall 字段（depends_on=ST-EX-04 hard）
5. **result.json stats 聚合**：用例结束时统计四态计数写入 stats 字段
6. **dry-run 行为**：dry-run 模式下 step 不实际执行（op_mapper 返回 dry_run envelope），classify_result 仍按 envelope.status 判定（dry_run envelope status 通常为 PASS 或 DRY_RUN）；dry_run 状态归为 PASS（命令构建成功）

## 回滚策略

- 本 Story 为 case_runner.py 纯新增函数（classify_result + classify_overall + 常量），不修改 op_mapper，不改变 envelope 结构
- 回滚方式：git revert 相关 commit；移除后 execute_steps 不记录四态（step 结果只有 envelope 原始 status），不影响 op 执行
- 无运行时副作用：四态判定是纯计算，不触发设备操作

## 权限与风险

- **权限约束**：遵循 NO_CREDENTIAL_READ / NO_PRODUCTION_WRITE / NO_EXTERNAL_PUBLISH；classify_result 是纯判定不触发设备操作
- **数据安全**：result.json 含四态结果数据但不含凭据
- **失败处理**：envelope 缺失/异常归 ERROR（脚本 bug）；未知 error_type 保守归业务失败（FAIL/KNOWN_FAIL）；不崩溃
- **风险**：纯新增函数（classify_result + classify_overall），无运行时副作用；回滚 git revert

## 测试要点

| 测试项 | 验证方法 | 预期结果 |
|---|---|---|
| PASS 判定 | 单元测试：envelope.status=PASS, error_type=None | classify_result=PASS |
| FAIL 判定 | 单元测试：status=FAIL, error_type=eObjectNotFound, known_issue=false | FAIL |
| KNOWN_FAIL 判定 | 单元测试：status=FAIL, error_type=eBeingReferenced, known_issue=true | KNOWN_FAIL（HLD 场景 4） |
| ERROR-执行异常 | 单元测试：status=FAIL, error_type=EXEC_FAILED, known_issue=true | ERROR（known_issue 不豁免执行异常） |
| ERROR-认证失败 | 单元测试：error_type=AUTH_FAILED | ERROR |
| ERROR-超时 | 单元测试：error_type=TIMEOUT | ERROR |
| ERROR-参数校验 | 单元测试：error_type=VALIDATION_FAILED | ERROR |
| 未知 error_type | 单元测试：error_type=SomeUnknown, known_issue=false | FAIL（保守归业务失败） |
| envelope 缺失 | 单元测试：envelope=None | ERROR |
| overall PASS | 单元测试：全 step PASS | overall=PASS |
| overall KNOWN_FAIL | 单元测试：9 PASS + 1 KNOWN_FAIL | overall=KNOWN_FAIL |
| overall FAIL | 单元测试：1 FAIL + 1 KNOWN_FAIL | overall=FAIL（FAIL 优先） |
| overall ERROR | 单元测试：1 ERROR + 1 FAIL | overall=ERROR（ERROR 优先） |
| overall 空用例 | 单元测试：step_results=[] | overall=PASS |
| 四态枚举完整 | grep result.json overall 字段 | 仅 PASS/FAIL/KNOWN_FAIL/ERROR 四值 |
| dry-run 判定 | dry-run 跑用例 | dry_run envelope 归 PASS |

## 开放项

| 问题 ID | 问题 | 状态 | 说明 |
|---|---|---|---|
| LCQ-ST-EX-10-01 | 四态命名（任务描述 KNOWN_ISSUE/WARMING_UP vs HLD KNOWN_FAIL/ERROR） | OPEN（blocks_lld=false） | 已写入 QUESTION-LEDGER，推荐 A（以 HLD 为准 PASS/FAIL/KNOWN_FAIL/ERROR）。本 Story 按 HLD 实现，待 host-orchestrator 确认 |
| O-EX-10-01 | 未知 error_type 归类 | OPEN（非阻塞） | 当前保守归为业务失败（FAIL/KNOWN_FAIL），允许 known_issue 豁免。若实际出现应归 ERROR 的未知类型，后续补入 _EXEC_ERROR_TYPES。CR-033 范围内 op_mapper 现有 error_type 已覆盖 |
| O-03（HLD） | 24 用例 known_issue 标注完整性 | OPEN（ST-EX-13 同步检查） | known_issue 标注由 ST-EX-13 整改时同步检查（DQ-CP3-03），本 Story 只解析标注不负责标注完整性 |

## 与 HLD/ADR 追溯一致性

| HLD/ADR 条目 | 本 Story 落实点 |
|---|---|
| HLD §12.1 [5]（known_issue -> KNOWN_FAIL） | classify_result KNOWN_FAIL 判定 |
| HLD §12.1 [7]（report.md 四态统计表） | result.json stats 四态计数供报告消费 |
| HLD §8 场景 4（SCN-EX-09 known_issue 四态） | KNOWN_FAIL 判定矩阵 + overall 不降级 |
| HLD §18 下沉（四态判定矩阵 -> FE-EX-02 DESIGN.md） | 判定矩阵补入 DESIGN.md |
| HLD §21 自审（ISTQB 四态分级） | 四态定义引用 ISTQB |
| R-F-010（known_issue 标记 -> KNOWN_FAIL） | classify_result 按 known_issue 区分 FAIL/KNOWN_FAIL |
| R-F-011（四态 PASS/FAIL/KNOWN_FAIL/ERROR） | overall 字段四态枚举 |
| DQ-CP3-03（known_issue 标注纳入 R-F-021） | 标注完整性由 ST-EX-13 检查，本 Story 只解析 |
