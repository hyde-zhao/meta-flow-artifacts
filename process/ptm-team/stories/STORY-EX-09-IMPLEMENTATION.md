---
cr_id: "CR-033"
story_id: "STORY-EX-09"
artifact_type: "implementation"
lld_ref: "process/stories/STORY-EX-09.md"
status: "implemented"
implemented_at: "2026-07-30T10:00:00+08:00"
author: "meta-dev"
---

# ST-EX-09 实现执行证据：retry 轮询（字段解析 + 轮询执行）

## 1. 实现对象清单

| 对象 | 文件路径 | 锚点 | 动作 | 行号 |
|---|---|---|---|---|
| _SUCCESS_COND_RE | skills/case-execution/scripts/case_runner.py | ST-EX-09 | 新增常量 | L597 |
| _compare_values | skills/case-execution/scripts/case_runner.py#_compare_values | ST-EX-09 | 新增内部函数 | L600-L640 |
| _eval_success_condition | skills/case-execution/scripts/case_runner.py#_eval_success_condition | ST-EX-09 | 新增内部函数 | L642-L663 |
| apply_retry | skills/case-execution/scripts/case_runner.py#apply_retry | ST-EX-09 | 新增 | L665-L736 |
| execute_steps retry 集成 | skills/case-execution/scripts/case_runner.py#execute_steps | ST-EX-09 集成点 | 修改 | L1140-L1160（dry-run）/ L1196-L1204（--execute） |

## 2. 设计契约映射

| LLD 契约 | 实现落点 | 一致性 |
|---|---|---|
| retry 字段格式（interval/max_attempts/success_condition） | L683-L696 解析三字段，缺省 interval=2/max_attempts=1 | ✓ 一致 |
| apply_retry 签名（step, execute_fn, *, dry_run） | L665 签名一致 | ✓ 一致 |
| 返回值附加 retry_satisfied/retry_attempts | L712-L713（dry-run）/ L728-L729（满足）/ L733-L734（不满足） | ✓ 一致 |
| _eval_success_condition 正则 ^(\w+)\s*(==\|!=\|>=\|<=\|>\|<)\s*(.+)$ | L597 _SUCCESS_COND_RE | ✓ 一致 |
| 严禁 eval/exec | _compare_values 用受限比较；grep 确认无 eval/exec | ✓ 一致 |
| 数值支持 6 操作符 | L611-L626 | ✓ 一致 |
| 字符串仅支持 ==/!= | L633-L637 | ✓ 一致 |
| field 缺失/类型不匹配/语法非法返回 False | L657-L660（field 缺失）/ L652（语法非法） | ✓ 一致 |
| dry-run 单次执行不轮询不 sleep | L705-L718 dry_run=True 分支，retry_attempts=1, retry_satisfied=N/A | ✓ 一致 |
| dry-run 校验 success_condition 语法 | L714-L718 语法非法记录 _retry_warning | ✓ 一致 |
| 真实轮询 sleep（最后一次不 sleep） | L725-L726 attempt < max_attempts 才 sleep | ✓ 一致 |
| 不修改 envelope 原有字段 | execute_steps L1144/L1202 pop 出 retry 附加字段，保持 envelope 纯净 | ✓ 一致 |
| 与四态分级交互（retry 不改变四态） | apply_retry 返回 envelope 后由 classify_result 判定，retry_satisfied 不影响四态 | ✓ 一致 |

## 3. 测试 Fixture

| Fixture | 路径 | 用途 |
|---|---|---|
| 单元测试 | /tmp/cr033-w3-test/unit_tests.py | UT-09-01..06 + dry-run + 语法非法 |
| dry-run 用例 | /tmp/cr033-w3-test/cases/PC-M4-01-01-01-Wave3验证.md | STEP-003 含 retry:{interval:2,max_attempts:3,success_condition:"hitcount>0"} |

## 4. 最小实现切片

ST-EX-09 与 ST-EX-08/10/11 在同一 case_runner.py 实现：
- execute_steps 在 dry-run 和 --execute 两个分支都接入 apply_retry
- dry-run 分支：_dry_run_fn 回调封装 build_command，apply_retry(dry_run=True) 单次执行
- --execute 分支：_execute_fn 回调封装 execute_op，apply_retry(dry_run=False) 轮询
- apply_retry 附加的 retry_satisfied/retry_attempts 由 execute_steps pop 到 step 记录顶层，envelope 保持原始结构

## 5. 本地验证

| 验证项 | 命令 | 结果 |
|---|---|---|
| 语法检查 | python3 -c "import ast; ..." | SYNTAX OK |
| grep 无 eval/exec | grep -nE "\beval\(\|\bexec\(" | NO eval/exec FOUND (PASS) |
| UT-09-01 首次满足 | unit_tests.py | PASS（retry_satisfied=True, attempts=1, calls=1） |
| UT-09-02 全部不满足 | unit_tests.py | PASS（retry_satisfied=False, attempts=3, calls=3） |
| UT-09-03 数值比较 | unit_tests.py | PASS（>/>=/<</==/!= 全覆盖） |
| UT-09-04 字符串比较 | unit_tests.py | PASS（==/!= 命中，> 不命中） |
| UT-09-05 field 缺失 | unit_tests.py | PASS（返回 False 不抛异常） |
| UT-09-06 非法语法 | unit_tests.py | PASS（无 OP/extra/inject 均 False） |
| UT-09 dry-run 单次 | unit_tests.py | PASS（satisfied=N/A, attempts=1, calls=1） |
| UT-09 dry-run 语法告警 | unit_tests.py | PASS（_retry_warning 记录） |
| dry-run 集成 | case_runner.py run --dry-run | step[3] retry_satisfied=N/A, retry_attempts=1 |

## 6. 平台差异

- canonical 源：skills/case-execution/scripts/case_runner.py
- apply_retry/_eval_success_condition 是 Python 纯函数，无平台差异
- 不改 op_mapper.py（op 执行仍委托 execute_op）
- 不引入 pytest（单元测试用 python3 -c assert）

## 7. 交接摘要

- ST-EX-09 apply_retry + _eval_success_condition + _compare_values 实现 + 6 单元测试通过
- execute_steps dry-run 和 --execute 双分支接入 retry
- 禁 eval/exec 安全约束遵循（grep 验证 + 正则解析 + 受限比较）
- O-EX-09-01（AND/OR 复合条件）按推荐：CR-033 范围内单层足够，复合条件进 BACKLOG
- 依赖 ST-EX-04（execute_steps 框架）已就绪
