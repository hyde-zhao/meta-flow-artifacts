---
cr_id: "CR-033"
story_id: "STORY-EX-11"
artifact_type: "implementation"
lld_ref: "process/stories/STORY-EX-11-LLD.md"
status: "implemented"
implemented_at: "2026-07-30T10:00:00+08:00"
author: "meta-dev"
---

# ST-EX-11 实现执行证据：诊断与报告（失败诊断 + report.md + 幂等容错）

## 1. 实现对象清单

| 对象 | 文件路径 | 锚点 | 动作 | 行号 |
|---|---|---|---|---|
| IDEMPOTENT_TOLERANCE_RULES | skills/case-execution/scripts/case_runner.py | ST-EX-11 | 新增常量 | L741-L764 |
| _SENSITIVE_KEY_RE | skills/case-execution/scripts/case_runner.py | ST-EX-11 | 新增常量 | L766 |
| check_idempotent_tolerance | skills/case-execution/scripts/case_runner.py#check_idempotent_tolerance | ST-EX-11 | 新增 | L769-L805 |
| _sanitize_dict | skills/case-execution/scripts/case_runner.py#_sanitize_dict | ST-EX-11 | 新增内部函数 | L807-L820 |
| diagnose_failure | skills/case-execution/scripts/case_runner.py#diagnose_failure | ST-EX-11 | 新增 | L822-L872 |
| generate_report | skills/case-execution/scripts/case_runner.py#generate_report | ST-EX-11 | 新增 | L874-L1027 |
| execute_steps 容错+诊断集成 | skills/case-execution/scripts/case_runner.py#execute_steps | ST-EX-11 集成点 | 修改 | L1206-L1222（--execute 失败路径） |
| main [8] generate_report 调用 | skills/case-execution/scripts/case_runner.py#main | ST-EX-11 集成点 | 修改 | L1822-L1831 |

## 2. 设计契约映射

| LLD 契约 | 实现落点 | 一致性 |
|---|---|---|
| §3.1 diagnose_failure 签名（step, envelope, *, command） | L822 签名一致 | ✓ 一致 |
| §3.1 五元组（error_type/error_code/reason/details/command） | L854-L868 提取五元组 | ✓ 一致 |
| §3.1 LCQ-ST-EX-11-01 error_code 双字段（data.error_code/data.code） | L857 data.get("error_code") or data.get("code") | ✓ 一致（推荐方案） |
| §3.1 reason 双字段（data.reason/data.message）兜底"未知原因" | L858 data.get("reason") or data.get("message") or "未知原因" | ✓ 一致 |
| §3.1 details = data dump 截断 500 字符 | L862-L865 json.dumps + 截断 500 | ✓ 一致 |
| §3.1 command 从参数或 data.command 取 | L866-L872 | ✓ 一致 |
| §3.1 envelope 非 dict 兜底 UNKNOWN_ERROR | L841-L852 | ✓ 一致 |
| §3.2 generate_report 签名（run_result, *, output_path） | L874 签名一致 | ✓ 一致 |
| §4.3 report.md 五节模板 | L896-L1022（§1 四态统计/§2 失败诊断/§3 幂等容错/§4 KNOWN_FAIL/§5 runtime_authorization） | ✓ 一致 |
| §3.3 IDEMPOTENT_TOLERANCE_RULES 3 类 | L741-L764（object_already_exists/stream_not_found/object_in_use） | ✓ 一致 |
| §3.3 check_idempotent_tolerance 匹配逻辑（op_id + error_code/reason） | L783-L803 error_code 精确匹配 + reason 子串匹配 | ✓ 一致 |
| §3.3 LCQ-ST-EX-11-02 容错范围先 3 类 | L741-L764 仅 3 类，其他进 BACKLOG | ✓ 一致（推荐方案） |
| §4.1 逐 step 失败时先容错再诊断 | execute_steps L1206-L1222（先 check_idempotent_tolerance，命中则 PASS 不诊断） | ✓ 一致 |
| §4.1 容错命中 idempotent_skip=true + status=PASS | L1210-L1214 | ✓ 一致 |
| §4.1 真实失败调 diagnose_failure | L1217 | ✓ 一致 |
| §12.3 凭据脱敏（password/secret/token 黑名单） | _sanitize_dict L807-L820 + _SENSITIVE_KEY_RE | ✓ 一致 |
| §5.2 report.md 写入失败不中断 | L1024-L1028 try/except + stderr WARNING | ✓ 一致 |
| §5.2 output_path 目录不存在 makedirs | L889-L891 | ✓ 一致 |
| R-C-004 不做 HTML（report.md 是 Markdown） | 全文 Markdown 输出 | ✓ 一致 |
| NO_CREDENTIAL_READ | diagnose_failure details 脱敏；command 含 --password-env <env-name>（环境变量名非密码值） | ✓ 一致 |
| NO_EXTERNAL_PUBLISH | report.md 写本地 runs/ 目录 | ✓ 一致 |

## 3. 测试 Fixture

| Fixture | 路径 | 用途 |
|---|---|---|
| 单元测试 | /tmp/cr033-w3-test/unit_tests.py | UT-11-01..08 + 脱敏 + 双字段兜底 |
| dry-run 用例 | /tmp/cr033-w3-test/cases/PC-M4-01-01-01-Wave3验证.md | STEP-004 失败触发 diagnose_failure |
| dry-run report.md | /tmp/cr033-w3-test/runs/run-*/report.md | 五节结构验证 |

## 4. 最小实现切片

ST-EX-11 与 ST-EX-08/09/10 在同一 case_runner.py 实现：
- execute_steps --execute 失败路径：先 check_idempotent_tolerance，命中则 idempotent_skip + PASS，未命中则 diagnose_failure + classify_result
- dry-run 失败路径（build_command 失败）：也调 diagnose_failure（L1157）
- main [8] generate_report 在 result.json 写入后调用（L1822），消费 results + summary + runtime_auth
- extract_payload 待 ST-EX-12 承载，diagnose_failure details 暂用 data dump 兜底（LLD §5.1）

## 5. 本地验证

| 验证项 | 命令 | 结果 |
|---|---|---|
| 语法检查 | python3 -c "import ast; ..." | SYNTAX OK |
| UT-11-01 五元组提取 | unit_tests.py | PASS（error_type/code/reason/details/command） |
| UT-11-02 object_already_exists | unit_tests.py | PASS（eObjectExists 命中） |
| UT-11-03 stream_not_found | unit_tests.py | PASS（reason 子串 "stream not found" 命中） |
| UT-11-04 object_in_use | unit_tests.py | PASS（eBeingReferenced 命中） |
| UT-11-05 非幂等失败 | unit_tests.py | PASS（VALIDATION_FAILED + op 不匹配 均 None） |
| UT-11-06 generate_report | unit_tests.py | PASS（四态统计 + 诊断 + 容错 + KNOWN_FAIL + ra） |
| UT-11-07 缺 error_code | unit_tests.py | PASS（error_code=""） |
| UT-11-08 envelope 非 dict | unit_tests.py | PASS（UNKNOWN_ERROR + "envelope 结构异常"） |
| 脱敏测试 | unit_tests.py | PASS（password/token -> ***REDACTED***） |
| LCQ-ST-EX-11-01 code 字段兜底 | unit_tests.py | PASS（data.code 命中） |
| LCQ-ST-EX-11-01 message 字段兜底 | unit_tests.py | PASS（data.message 命中） |
| dry-run report.md | case_runner.py run --dry-run | 五节齐全：统计 PASS×3 ERROR×2 + 诊断 + 容错(无) + KNOWN_FAIL(无) + dry-run ra |
| dry-run diagnose | case_runner.py run --dry-run | step[4] diagnosis.error_type=VALIDATION_FAILED, reason 含 "source-network" |

## 6. 平台差异

- canonical 源：skills/case-execution/scripts/case_runner.py
- diagnose_failure/generate_report 是 Python 纯函数 + Markdown 输出，无平台差异
- 不改 op_mapper.py（envelope 只消费）
- 不改 24 用例 md（ST-EX-13 拥有）

## 7. 交接摘要

- ST-EX-11 diagnose_failure + check_idempotent_tolerance + generate_report + IDEMPOTENT_TOLERANCE_RULES 实现 + 8 单元测试通过
- execute_steps 失败路径集成容错 + 诊断，main [8] 集成 generate_report
- LCQ-ST-EX-11-01（error_code 字段名）按推荐：双字段检查 data.error_code/data.code + reason/data.message 兜底
- LCQ-ST-EX-11-02（容错范围）按推荐：CR-033 先 3 类，其他 op 进 BACKLOG
- 凭据脱敏实现（password/secret/token 黑名单 + ***REDACTED***）
- extract_payload 待 ST-EX-12 承载，details 暂用 data dump 兜底
- 依赖 ST-EX-04（execute_steps/main）+ ST-EX-10（classify_result）已就绪
