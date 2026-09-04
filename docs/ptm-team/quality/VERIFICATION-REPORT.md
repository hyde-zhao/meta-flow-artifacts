---
cr_id: CR-039
title: ptm-te 工作区已验证代码回源 canonical 验证执行报告（STORY-039-01~04）
version: "1.0"
validation_mode: static-only
source_design: process/stories/STORY-039-01~04.md（technical-note，CP5 approve 2026-08-19）+ process/checkpoints/CP2-CR039.md（DQ-039-01~05 / SM-039-01~05 / RA-039-01~05）
source_cp6: process/checks/CP6-CR039.result.json + process/returns/STORY-039-0{1..4}.return.json + process/stories/STORY-039-0{1..4}-IMPLEMENTATION.md
reviewer: "meta-qa (independent, CP7)"
review_date: "2026-08-19T00:00:00+00:00"
---

# CR-039 验证执行报告（VERIFICATION-REPORT）

> 独立验证声明：本报告由 meta-qa 在 CP7 会话独立复跑验证产出，未盲信 CP6 result。单测/全量 pytest 真实执行并记录输出；真机项 not_authorized（DQ-039-04），未执行、未假装验证通过。

## 1. 结论

| 项目 | 内容 |
|---|---|
| 阶段决策 | **PASS_WITH_RISK** |
| validation_mode | static-only（DQ-039-04 不授权真机：无 trex 发流、无 SSH、无 telnet 真机） |
| 路由 | none（4 Story 可推进 verified；风险项汇入 CP8 Decision Brief / 风险台账，见 §12） |
| 覆盖 Story | STORY-039-01（G1）/ STORY-039-02（G4）/ STORY-039-03（G5）/ STORY-039-04（文档追溯） |
| Workflow eval | N/A（`validation_target.sut_type=code-project`：canonical 源 Python 脚本回源，以原生单测 + 静态断言 + 代码级核验为验证证据） |

**PASS_WITH_RISK 理由**：全部 BLOCKING 维度与 SM-039-01~05 均通过；但 static-only 口径下真机行为（G1 大路由表 verify、sw3 telnet 端到端、telnetlib py>=3.13 分支）未验证（DQ-039-04 用户已决策不授权，RA-039-01/02 + BACKLOG BL-039-02 台账承接）。该风险不阻断交付，但必须汇入 CP8 Decision Brief。

## 2. 验证范围

- G1：`_query_static_route_id` + `handle_rollback` 静态路由无条件 verify 分支（op_mapper.py）+ `tests/test_op_mapper_rollback.py`。
- G4：`resolve_env_refs` 第 10 类占位符 `${ENV.dut.next_hop_a}` + 占位符表/报错文案计数 + `tests/test_op_mapper_next_hop_a.py`。
- G5：op_mapper 六处映射 + sw3 执行段三函数 + validate [5j] + `EXPECTED_OP_COUNT=45` + case_runner sw3 dry-run envelope + `tests/test_tg_op_mapping.py`（含 CR-047 post_delay 精细排除）。
- 文档追溯：SKILL.md Gotcha#8/v1.7/v1.8、op-coverage-matrix sw3 旁路章节、CR-043/044/046 档案、BACKLOG、env-file 样例。
- 不在范围：真机操作（一律未执行）；test_cr018_p2 GATE-4 存量失败（非本 CR 归属，见 §7/§11）。

## 3. 验证对象清单

| 对象 | 路径 | 证据类型 |
|---|---|---|
| op_mapper.py（G1+G4+G5 区） | `skills/policy-route-execution/scripts/op_mapper.py` | 代码 + 单测 + validate + 代码级核验 |
| case_runner.py（sw3 envelope） | `skills/case-execution/scripts/case_runner.py` | 代码 + 单测 + grep 静态断言 |
| G1 单测 | `tests/test_op_mapper_rollback.py` | 单测（mock execute_op） |
| G4 单测 | `tests/test_op_mapper_next_hop_a.py` | 单测（纯解析，无 mock/网络） |
| G5 单测 | `tests/test_tg_op_mapping.py`、`tests/test_op_mapper_pppoe_client.py`（计数载体） | 单测（含 dry-run 计划断言） |
| SKILL.md | `skills/case-execution/SKILL.md` | 文档一致性 grep |
| op-coverage-matrix | `docs/ptm-te/op-coverage-matrix.md` | 文档一致性 grep |
| 追溯档案 | `process/changes/CR-043.md` / `CR-044.md` / `CR-046.md`、`process/BACKLOG-CR-039.md` | 档案结构核验 |
| env-file 样例 | `skills/case-execution/templates/env-file.pppoe-sw3.next_hop_a.example.yaml` | 字段契约 + 凭据安全核验 |
| CP6 实现证据 | `process/returns/STORY-039-0{1..4}.return.json`、`process/stories/STORY-039-0{1..4}-IMPLEMENTATION.md` | 证据消费（read_expansion 已登记） |

## 4. 验证追踪矩阵（SM × 验证方法 × 结果）

| SM | 验证方法（独立执行，非采信自报） | 结果 |
|---|---|---|
| SM-039-01（EXPECTED_OP_COUNT=45 四处一致） | ① 代码常量 `grep`：op_mapper.py L1080 `EXPECTED_OP_COUNT = 45`；② 独立运行 `validate_mapping_consistency()`：`PASS (45 op_id 全覆盖，三表一致)`，mismatches=[]，len(OP_ID_TO_SUBCOMMAND)=45，sw3 map=('sw3','sync-pppoe-route')；③ 单测断言：test_pass_45_ops / test_expected_op_count_is_45 / test_sw3_six_mappings（test_tg_op_mapping，已复跑 PASS）+ test_op_mapper_pppoe_client::test_expected_op_count（断言 45，全量复跑 PASS）；④ SKILL.md Gotcha#8=45（L315，显式算式 40+2+2+1） | **PASS** |
| SM-039-02（G1 verify 四分支单测） | 复跑 `HandleRollbackStaticRouteVerifyTests`：test_matched_returns_route_id / test_multi_route_next_hop_disambiguates / test_no_match_returns_exec_failed / test_placeholder_next_hop_skips_match 4 分支 + test_vals_direct_data_shape 扩展，全部 PASSED | **PASS** |
| SM-039-03（G4 解析单测） | 复跑 `ResolveNextHopATests`（成功解析 80.0.0.2 / 缺失报错含 nodes.dut1.next_hop_a / 空值报错 / next_hop 回归 110.0.0.2）+ `PlaceholderTableTenthClassTests`（第 10 类条目 + 「不在 10 类」文案），6 用例全部 PASSED | **PASS** |
| SM-039-04（sw3 dry-run envelope 单测，且不含 post_delay） | 复跑 `Cr046Sw3DryRunEnvelopeTests`：test_envelope_success_wraps_plan（断言 status=dry_run、data.command=计划 dict、assertNotIn("ptm-atomic")）/ test_envelope_error_passthrough / test_case_runner_no_post_delay_backport（源码级断言无 post_delay），全部 PASSED；另 `grep -n "post_delay\|_parse_post_delay" skills/case-execution/scripts/case_runner.py` 零命中（exit=1） | **PASS** |
| SM-039-05（CR-043/044/046 档案 + SKILL.md v1.7/v1.8） | 档案存在且结构核验（三份均含背景/变更内容/验证方式/来源/Checkpoint Index/与 CR-039 关系六节，status closed/closed/active）；SKILL.md 修订记录 v1.7（CR-046 sw3 族 44->45）与 v1.8（CR-043/044/046 来源行）均存在 | **PASS** |

## 5. 设计契约验证清单

| 契约 | 核验方式 | 结果 |
|---|---|---|
| DQ-039-01：G1 无条件 verify，无 step-refs fallback | 代码级核验 `handle_rollback` inverse_op 分支：`if op_id == "fw_config_static_route"` 无条件调用 `_query_static_route_id`，位于声明路径（`_load_op_decl`/`build_inverse_args`）之前；rid None -> EXEC_FAILED envelope（reason 含 dst_network 与「无法清理」）；匹配 -> `execute_op("fw_delete_static_route", {"id": rid})`。注释（G5 合入后行号漂移至 L714-716）已更新为「无条件 verify 查询…匹配失败即 EXEC_FAILED，无 step-refs fallback（DQ-039-01 方案 A）」 | **PASS** |
| DQ-039-02：op-coverage-matrix total=124/mapped=21 不变 + sw3 旁路章节 | grep 核验：mapped=21 / unmapped=97 / total=124 计数不变；「sw3 族（telnet 旁路，不计入 ptm-atomic atoms 映射，CR-046 / CR-039 补记）」章节存在（L58-66，含两套口径说明）；修订记录 v1.3 落档 | **PASS** |
| DQ-039-05（B）：CR-047 post_delay 精细排除 | `grep -n "post_delay\|_parse_post_delay" case_runner.py` 零命中（exit=1）+ 源码级守护单测 test_case_runner_no_post_delay_backport PASS；BACKLOG BL-039-01 记录排除行双标注（实测 L2511 / 卡片原写 L2437-2440） | **PASS** |
| DQ-039-03（B）：追溯档案 + SKILL.md 来源行 | 见 SM-039-05 | **PASS** |
| DQ-039-04：不授权真机 | 验证全程仅单测/静态断言/代码级核验，无网络、无 SSH、无 telnet 真机、无凭据读取 | **PASS** |
| 安全：sw3 密码不打印不落盘 | 代码级核验：密码仅 `os.environ.get(password_env)`（L2204）取值，传参 `_exec_sw3_telnet` 后仅 `tn.write(password.encode())`，无 print/log/exec_log 落盘（grep print/logger×password 零命中）；dry-run 计划 dict 仅含 host/username/vpn_instance/planned_commands，不含密码；未设置 -> PARAM_INVALID、未授权 -> EXEC_FAILED 单测均 PASS | **PASS** |
| 安全：env-file 样例无明文凭据 | 通读核验：仅 `password_env: "SW3_PASSWORD"` 环境变量引用，无任何明文密码；YAML 契约字段齐备（dut1.next_hop_a=80.0.0.2 / sw1 node_type=SW / role=pppoe_server / host / username） | **PASS** |
| 安全：危险命令扫描 | 对 7 个改动/新增文件 grep（rm -rf / curl|sh / sudo / chmod 777 / mkfs / dd if= / eval 等）零命中 | **PASS** |

## 6. 分层验证计划（static-only 口径声明）

| 层 | 方式 | 状态 |
|---|---|---|
| L1 单元（独立复跑） | 三件套 85 passed；全量 244 passed + 2 failed（存量） | 已执行 |
| L2 静态断言 | grep post_delay 零命中 / EXPECTED_OP_COUNT 四处载体 / 文档计数 | 已执行 |
| L3 代码级契约核验 | handle_rollback 分支、sw3 密码处理、dry-run 计划内容 | 已执行 |
| L4 dry-run（sw3 命令计划） | 单测断言 dry-run 返回 telnet 命令计划 dict（非 ptm-atomic 命令、不连设备） | 已执行 |
| L5 真机 / 集成 | **N/A（DQ-039-04 不授权）**：无 trex 发流、无 SSH 重启、无 sw3 telnet 实连；等价验证为 L1-L4 + 工作区 26/28 step PASS 实测基线；真机回归为独立 runtime_authorization（BACKLOG BL-039-02） | 未执行（决策内） |

## 7. 自动化验证结果（命令与输出摘要）

| # | 命令 | 结果 |
|---|---|---|
| 1 | `uv run --python 3.11 pytest tests/test_op_mapper_rollback.py tests/test_op_mapper_next_hop_a.py tests/test_tg_op_mapping.py -v` | **85 passed**（G1 四分支+扩展 5 / G4 6 / G5 含 sw3 11 / 既有回归），0.18s |
| 2 | `uv run --python 3.11 pytest tests/ -q` | **244 passed + 2 failed**（均为 test_cr018_p2 GATE-4 脚本问题） |
| 3 | `git stash push -u` 后 `uv run --python 3.11 pytest tests/test_cr018_p2.py -q`（对拍后 `git stash pop` 恢复） | 基线（无本 CR 改动）同样 **2 failed**（test_gate4_passes_with_case_steps_atomic_op_contract / test_gate5_pass_sets_completed_phase）-> 独立确认为**存量失败，与本 CR 无关**；工作区恢复完整（9 个改动/新增文件均在） |
| 4 | `validate_mapping_consistency()`（importlib 独立加载 op_mapper） | `ValidationResult: PASS (45 op_id 全覆盖，三表一致)`；passed=True；mismatches=[]；EXPECTED_OP_COUNT=45；len(OP_ID_TO_SUBCOMMAND)=45 |
| 5 | `grep -n "post_delay\|_parse_post_delay" skills/case-execution/scripts/case_runner.py` | 零命中（exit=1） |

## 8. Prompt / Skill Fixture 验证

- Fixture 核验：G1 mock verify envelope 双形态（data.response.vals / data.vals）；G4 `_env_with_next_hop_a` 对齐 env-file nodes.dut1 字段；G5 SW3_TOPOLOGY 对齐工作区 CR-046 验证用例 sw1 节点契约（样例 yaml 由 STORY-039-04 落地，字段互通）。
- mock 边界合规：G1 仅 mock execute_op；G4 零 mock（纯解析）；G5 仅 `mock.patch.dict(os.environ)` 移除 SW3_PASSWORD，无 subprocess mock、无网络。符合 static-only 授权边界。
- workflow eval（Promptfoo/DeepEval 等外部 adapter）：N/A（code-project 回源，未使用网络/凭据/外部模型）。

## 9. 平台适配验证

N/A（本轮无安装器/平台交付对象）。改动对象为 canonical 源 Skill 私有脚本（op_mapper.py / case_runner.py）与仓库级文档/档案，不涉及平台安装目录、发现路径或 PLATFORM-CONTRACTS 分支；回源后经 ptm-atomic 既有安装链生效。env-file 样例位于 `skills/case-execution/templates/`（Skill 同树资产，符合交付脚本边界）。

## 10. 人工 / 语义质量审查

- CP6 证据消费：4 份 return packet（boundary_check.allowed_paths_only=true、forbidden_paths_touched=[]、unexpected_imports=[]）+ 4 份 IMPLEMENTATION.md（实现对象清单/契约映射/测试计划/最小切片/平台差异/交接摘要齐备），未见输入缺陷。
- 偏离留痕审查：3 处有意偏离均已在 return packet/IMPLEMENTATION 留痕且合理--① test_op_mapper_pppoe_client 计数载体 44->45 最小同步（SM-039-01 必然要求）；② G4 报错文案/计数注释对齐 10 类（DQ 口径，canonical 为真相源）；③ 工作区行号漂移按内容块定位。
- 文档-代码一致性（方案评审规则第 1 条）：SKILL.md Gotcha#8=45 = 代码常量 45 = 单测断言 45 = validate 三表 45；op-coverage-matrix 两套口径显式声明，无矛盾。
- 逐块合并纪律：op_mapper.py 三波写入（G1->G4->G5）后 G1/G4 区块无回退（三件套单测共同复跑 PASS 即联合回归证据）；未整文件覆盖。

## 11. 问题清单

| # | 问题 | 严重度 | 处置 |
|---|---|---|---|
| P-1 | `tests/test_cr018_p2.py` 2 处失败（GATE-4 脚本问题） | LOW（存量，非本 CR 引入） | 不计入本 CR 缺陷；已独立 stash 对拍确认基线同样失败；建议另行跟踪（不属本 CR 范围，不创建新台账条目，CP8 可提示用户） |
| P-2 | 工作区侧报错文案/计数注释仍为「9 类」，canonical 已对齐「10 类」 | INFO（真相源方向已声明） | 已在 STORY-039-02 return packet notes 声明「后续对拍以 canonical 为真相源」，无需动作 |

无 BLOCKING 问题。未发现任何需要回修（NEEDS_REWORK）或设计澄清（NEEDS_DESIGN_CLARIFICATION）的事项。

## 12. 剩余风险（汇入 CP8 Decision Brief / 风险台账）

| 风险 ID | 描述 | 缓解 / 触发条件 | 分流建议 |
|---|---|---|---|
| R-1（RA-039-01） | G1 真机大路由表 verify 超时 / dst_network 非唯一歧义；分页 >100 条单页查询未覆盖 | next_hop_addr 消歧 + 多路由单测；真机超时按 DQ-039-01 备选 B（step-refs 优先）重评，不静默改 | risk_acceptance（static-only 决策内）+ 真机回归候选 BL-039-02 |
| R-2（RA-039-02） | sw3 telnet 端到端（H3C 登录 / undo+ip route-static 下发）真机未验证 | dry-run 命令计划断言 + 授权门（未授权 EXEC_FAILED）单测；工作区 CR-046 实测 26/28 基线 | runtime_authorization（独立发起，本 CR 不授权） |
| R-3 | telnetlib py>=3.13 不可用分支无环境验证 | 仓内 uv 锁 3.11；惰性导入注释已声明 | follow_up_candidate（升级 Python 时回归） |
| R-4 | sw3 经 case_runner execute_steps 全链路 dry-run（retry/step-refs 包装）未单测 | envelope 构建函数与 execute_op(dry_run=True) 两侧已覆盖 | follow_up_candidate |
| R-5 | 工作区实测 26/28 step PASS 中剩余 2 处 KNOWN_FAIL 流量回退验证未追验 | 已归档工作区；BL-039-03 候选 | follow_up_candidate（用户决定） |

以上风险均为 DQ-039-04（不授权真机）与 DQ-039-05（CR-047 独立回源）决策的已知后果，已由 RA 台账与 BACKLOG-CR-039 承接；**不授权项**：真机下发（trex 发流 / SSH 重启 / sw3 telnet 实连）、凭据明文落盘、CR-047 post_delay 回源，CP7 通过不构成对上述任一项的授权。

## 13. 阶段决策与 CP8 输入

- **阶段决策：PASS_WITH_RISK**。4 Story 全部满足验收标准：SM-039-01~05 核验通过；BLOCKING 维度（完整性 / 验收标准覆盖 / 安全合规）全部通过；REQUIRED 维度（命名规范--`_query_static_route_id`/`_execute_sw3_op` 等与工作区一致；Frontmatter--文档/档案类 N/A；可安装性--无安装器对象 N/A）通过或 N/A 理由充分。
- **CP8 输入**：① 风险 R-1~R-5 汇入 CP8 Decision Brief（分流建议见 §12）；② 不授权项清单（真机 / 凭据 / CR-047）；③ BACKLOG-CR-039 待 CP8 分流（BL-039-01 CR-047 候选 / BL-039-02 真机回归 / BL-039-03~06）；④ 存量失败 test_cr018_p2 提示（非本 CR 归属）。
- 追溯：TEST-MATRIX N/A（同步回源 CR，CP2 已记录 8/9 产物 N/A）；本报告即 VERIFICATION-REPORT 载体；TEST-REPORT/REVIEW 以本报告 + CP7 result 为等价质量证据（独立复跑即评审载体，低风险文档回源类不另开全文评审，CP7 result 记录 N/A 理由）。

---

**验证执行证据**：调度事件 `ADE-CR039-META-QA-001`（AGENT-DISPATCH-LEDGER）；deny-default 展开 7 项已登记 READ-EXPANSION-LEDGER（RE-20260819T035800Z-CR039-CP7-*）；机器真相源 `process/checks/CP7-CR039.result.json`。

---
---

# CR-047 验证执行报告（VERIFICATION-REPORT，CR-047 版）

---
cr_id: CR-047
title: case_runner post_delay 独立回源验证执行报告（STORY-047-01）
version: "1.0"
validation_mode: static-only
source_design: process/stories/STORY-047-01.md（technical-note，CP5 approve 2026-08-19）+ process/checkpoints/CP2-CR047.md（DQ-047-01/02）+ process/changes/CR-047.md（量化验收 4 条）
source_cp6: process/checks/CP6-CR047.result.json + process/returns/STORY-047-01.return.json + process/evidence/STORY-047-01.index.json
reviewer: "meta-qa (independent, CP7)"
review_date: "2026-08-19T10:30:00+00:00"
---

> 独立验证声明：本报告由 meta-qa 在 CP7 会话独立复跑验证产出，未盲信 CP6 result。单测/全量 pytest 真实执行并记录输出；真机项 not_authorized（DQ-039-04 沿用），未执行、未假装验证通过。本环境部分 grep 管道受干扰，源码核对统一以 python3 读文件方式执行。

## 1. 结论

| 项目 | 内容 |
|---|---|
| 阶段决策 | **PASS_WITH_RISK** |
| validation_mode | static-only（真机不授权沿用：无 trex 发流、无 SSH、无 telnet 真机） |
| 路由 | none（单 Story 可推进 verified；风险项汇入 CP8 Decision Brief / follow-up 台账，见 §12） |
| 覆盖 Story | STORY-047-01（case-runner-post-delay-backport，technical-note） |
| Workflow eval | N/A（`validation_target.sut_type=code-project`：canonical 源 Python 脚本回源，以原生单测 + 静态断言 + 代码级核验为验证证据） |

**PASS_WITH_RISK 理由**：量化验收 4 条（SM-047-01~04）全部通过，BLOCKING 维度全绿；但 static-only 口径下 post_delay 真机实测行为未验证（无 post_delay 字段的真机链路行为不变已由单测锁定；带 post_delay 的真机 tear-down 等待效果属真机回归候选，沿用 DQ-039-04 决策），且 OPI-047-01 负数透传语义（|负值|>=OP_INTERVAL_SEC 时 sleep 总和为负抛 ValueError）为已固化的已知边界，clamp 记 deferred。该风险不阻断交付，必须汇入 CP8 Decision Brief。

## 2. 验证范围

- 回源实现：`_parse_post_delay` 函数（L114-133）+ execute_steps 主循环睡眠行叠加（L2440）+ rollback 清理循环不动（run_cleanup `i > 0` 分支纯 `OP_INTERVAL_SEC`）。
- 单测：`tests/test_case_runner_post_delay.py`（11 用例）+ `tests/test_tg_op_mapping.py` 守护单测反转（`test_case_runner_post_delay_backported`）。
- 文档：`skills/case-execution/SKILL.md` 转换规则表 post_delay 行 + 修订记录 v1.9。
- 不变量：`EXPECTED_OP_COUNT=45`（op_mapper.py）不变、op-coverage-matrix total=124/mapped=21 口径不变。
- 不在范围：真机操作（一律未执行）；test_cr018_p2 GATE-4 存量失败（非本 CR 归属，见 §11）。

## 3. 验证对象清单

| 对象 | 路径 | 证据类型 |
|---|---|---|
| case_runner.py（_parse_post_delay + execute_steps 叠加） | `skills/case-execution/scripts/case_runner.py` | 代码级核验（python3 读文件）+ 单测 + 守护单测 |
| 新增单测 | `tests/test_case_runner_post_delay.py` | 单测（mock execute_op/build_command + monkeypatch time.sleep，static-only） |
| 守护反转 | `tests/test_tg_op_mapping.py::test_case_runner_post_delay_backported` | 单测（源码级断言：定义 + 恰 1 调用点 + run_cleanup 零命中） |
| SKILL.md 文档增量 | `skills/case-execution/SKILL.md` | 文档一致性核对（post_delay 行 + v1.9） |
| 范围不变量 | `skills/policy-route-execution/scripts/op_mapper.py`、`docs/ptm-te/op-coverage-matrix.md` | 静态核对（EXPECTED_OP_COUNT=45 / total=124/mapped=21） |
| CP6 实现证据 | `process/returns/STORY-047-01.return.json`、`process/evidence/STORY-047-01.index.json`、`process/checks/CP6-CR047.result.json` | 证据消费（read_expansion 已登记） |

## 4. 验证追踪矩阵（量化验收 × 验证方法 × 结果）

| 量化验收 / SM | 验证方法（独立执行，非采信自报） | 结果 |
|---|---|---|
| 验收-1 / SM-047-01（定义 + 恰 1 调用点 + rollback 纯净） | python3 全文扫描 `_parse_post_delay`：恰 2 处命中（L114 `def _parse_post_delay(val) -> float:` 定义 + L2440 execute_steps 主循环 `time.sleep(OP_INTERVAL_SEC + _parse_post_delay(step.get("post_delay")))` 调用，调用行确认位于 execute_steps（定义于 L2124）函数范围内）；run_cleanup 段独立提取核验：零命中 `_parse_post_delay`，睡眠行保持纯 `time.sleep(OP_INTERVAL_SEC)`；守护单测 `test_case_runner_post_delay_backported`（assertEqual count==2 + run_cleanup 段 assertNotIn）随定向套件 PASS | **PASS** |
| 验收-2 / SM-047-02（单测 ≥8 全过 + 套件无新增失败） | 独立复跑 `uv run pytest tests/test_case_runner_post_delay.py tests/test_tg_op_mapping.py -q`：**58 passed in 0.13s**（新增 11 用例 > 8 全绿）；全量 `uv run pytest tests/ -q`：**2 failed, 255 passed, 8 subtests passed in 13.87s**，2 处失败均为 test_cr018_p2 存量（test_gate4_passes_with_case_steps_atomic_op_contract / test_gate5_pass_sets_completed_phase，CR-039 CP7 期间 git stash 对拍已独立确认基线同样失败，与本 CR 无关），无新增失败 | **PASS** |
| 验收-3 / SM-047-03（守护单测语义反转并通过） | python3 读 `tests/test_tg_op_mapping.py` L622-633：已由 `test_case_runner_no_post_delay_backport`（assertNotIn 零残留）反转为 `test_case_runner_post_delay_backported`（断言源码含 `def _parse_post_delay(` 定义 + `_parse_post_delay(` 命中恰 2 + run_cleanup 段零命中）；随定向套件 58 passed 复跑 PASS | **PASS** |
| 验收-4 / SM-047-04（SKILL.md 补一行 + 修订记录） | python3 核对 SKILL.md：L295 转换规则表新增『step 执行后延迟』行（`post_delay: "5s"` / `"500ms"` / 纯数字秒；叠加 `OP_INTERVAL_SEC`；仅 `--execute` 生效；None/非法按 0；DQ-047-01=A）+ L335 修订记录 v1.9 行（meta-dev（CR-047），含实现摘要） | **PASS** |

## 5. 设计契约验证清单

| 契约 | 核验方式 | 结果 |
|---|---|---|
| `_parse_post_delay` 解析语义（technical-note §接口/函数设计） | 逐行读 L114-133 函数体与 technical-note 对拍：`None`->0.0；`isinstance(val,(int,float))`->`float(val)`（负数透传）；`str(val).strip().lower()` 空串->0.0；`ms` 后缀 `/1000.0`；`s` 后缀直取；其余 `float(s)`；`ValueError/TypeError`->0.0；docstring 标注 CR-047 用途（断开 PPPoE 后等待会话 tear down 再发流）。与工作区参考实现（return packet 记录逐字一致）语义一致 | **PASS** |
| execute_steps 叠加点（仅 `--execute` 生效） | L2440 叠加行位于 execute_steps `idx < n - 1` 分支（execute 分支内），调用点前存在 dry_run 判定路径；单测 `test_execute_steps_sleep_adds_post_delay`（2 step + post_delay "3s" -> sleep 恰 1 次且值 5.0 = OP_INTERVAL_SEC 2 + 3.0，最后一步不睡眠）+ `test_execute_steps_dry_run_no_sleep`（dry-run 分支 sleep 零调用）双重锁定 | **PASS** |
| 无 post_delay 字段零感知（回归安全） | `step.get("post_delay")` 缺省 None -> 0.0，叠加行为与回源前一致；单测 `test_execute_steps_sleep_without_post_delay`（无字段 -> sleep 值仍为 OP_INTERVAL_SEC）锁定 | **PASS** |
| rollback 不受影响 | run_cleanup 段源码核验零命中 + 单测 `test_rollback_loop_not_affected` 源级守护；`OP_INTERVAL_SEC=2`（L110）/ `CASE_INTERVAL_SEC=8` 常量值不变 | **PASS** |
| 范围不变量（不触碰 op_mapper / op-coverage-matrix） | python3 核对：`op_mapper.py` L1080 `EXPECTED_OP_COUNT = 45` 不变；op-coverage-matrix 保持 total=124（118 fw + 6 tg atoms）/ mapped=21 口径与 sw3 旁路章节双口径声明不变（本 CR 不新增 op，post_delay 为 step 级参数）；git status 中 op_mapper.py / test_op_mapper_rollback.py 的未提交改动为 CR-039/CR-046 回源遗留（基线 5038ce6 之后、本 Story 之前已存在），非本线程写入（return packet boundary_check 留痕一致） | **PASS** |
| 安全合规（ADR-02 沿用） | 本 CR 零凭据接触、零环境变量新增、零外部接口新增；验证全程仅 pytest + python3 读文件，无网络/SSH/telnet 真机/凭据读取；不改 dry-run 默认门与 `--execute` 授权门语义（叠加行在既有 execute 分支内） | **PASS** |

## 6. 分层验证计划（static-only 口径声明）

| 层 | 方式 | 状态 |
|---|---|---|
| L1 单元（独立复跑） | 定向 58 passed（新增 11 用例 + 守护反转）；全量 255 passed + 8 subtests passed + 2 failed（存量） | 已执行 |
| L2 静态断言 | `_parse_post_delay` 恰 2 处命中 / run_cleanup 零命中 / EXPECTED_OP_COUNT=45 / op-coverage-matrix 口径 / SKILL.md v1.9 | 已执行（python3 读文件） |
| L3 代码级契约核验 | 函数体逐行对拍 technical-note + 叠加点位于 execute 分支 + rollback 纯净 | 已执行 |
| L4 dry-run 行为 | 单测断言 dry-run 分支 sleep 零调用（post_delay 仅 --execute 生效） | 已执行 |
| L5 真机 / 集成 | **N/A（真机不授权沿用 DQ-039-04）**：无 trex 发流、无 SSH、无 telnet 真机；等价验证为 L1-L4 + 与工作区已实测基线（2 用例 26/28 step PASS，随 CR-039 验证载体）的函数体/睡眠行逐字一致；post_delay 真机 tear-down 效果实测为真机回归候选（沿用 BL-039-02 / CR-039-FU-RUNTIME） | 未执行（决策内） |

## 7. 自动化验证结果（命令与输出摘要）

| # | 命令 | 结果 |
|---|---|---|
| 1 | `uv run pytest tests/test_case_runner_post_delay.py tests/test_tg_op_mapping.py -q` | **58 passed in 0.13s**（ParsePostDelayTests 7 + ExecuteStepsPostDelayTests 4 + test_tg_op_mapping 全量含守护反转） |
| 2 | `uv run pytest tests/ -q` | **2 failed, 255 passed, 8 subtests passed in 13.87s**；2 失败均为 test_cr018_p2 GATE-4 存量（P-1 台账跟踪，CR-039 CP7 stash 对拍已确认基线同样失败） |
| 3 | python3 全文扫描 `skills/case-execution/scripts/case_runner.py` | `_parse_post_delay` 恰 2 处命中（L114 定义 + L2440 execute_steps 调用）；run_cleanup 段零命中、睡眠行纯 `OP_INTERVAL_SEC`；常量 L110 `OP_INTERVAL_SEC = 2` |
| 4 | python3 核对 `op_mapper.py` / `op-coverage-matrix.md` / `SKILL.md` | EXPECTED_OP_COUNT=45（L1080）；matrix total=124/mapped=21 双口径不变；SKILL.md L295 post_delay 行 + L335 v1.9 |

## 8. Fixture / dry-run / 静态断言核验

- Fixture 核验：新增单测 mock 边界合规--仅 mock `execute_op` / `build_command`（阻断真实执行）+ `mock.patch.object(cr.time, "sleep")`（阻断真实睡眠），零网络、零 subprocess、零凭据，符合 static-only 授权边界。
- dry-run 断言：`test_execute_steps_dry_run_no_sleep` 断言 dry-run 分支 sleep 零调用，锁定 post_delay 仅 `--execute` 模式生效契约。
- 静态断言：守护单测三重断言（定义存在 / 调用恰 1 / rollback 零命中）与 CR-047 验证方式口径自洽；工作区参考实现中 apply_warming_up 内另一处调用（工作区 L1398）不属本 Story 两块回源范围，canonical 未引入，守护『恰 1 调用点』口径与之自洽（return packet notes 留痕一致）。

## 9. 平台适配验证

N/A（无安装器/平台交付对象）。改动对象为 canonical 源 Skill 私有脚本（case_runner.py）与 SKILL.md 文档，不涉及平台安装目录、发现路径或 PLATFORM-CONTRACTS 分支；回源后经 ptm-atomic 既有安装链生效。

## 10. CP6 证据消费与人工审查

- return packet（`process/returns/STORY-047-01.return.json`）核验：allowed_paths_only=true、forbidden_paths_touched=[]、unexpected_imports=[]；touched_files 4 文件与 file_ownership.primary 完全一致；implementation_doc N/A 理由（低风险单 Story 回源，卡片/return packet 承载）符合实现执行门控的低风险豁免条款。
- acceptance_criteria_coverage 4/4 covered 且与本次独立复跑结果一致（数量、行号、口径均对得上）。
- 偏离审查：无设计偏离。工作树中 op_mapper.py / test_op_mapper_rollback.py 的并行未提交改动已正确留痕为其他 CR 线程写入（CR-039/CR-046 回源遗留），与本 Story 边界清晰。

## 11. 问题清单

| # | 问题 | 严重度 | 处置 |
|---|---|---|---|
| P-1 | `tests/test_cr018_p2.py` 2 处失败（GATE-4 脚本问题，test_gate4_passes_with_case_steps_atomic_op_contract / test_gate5_pass_sets_completed_phase） | LOW（存量，非本 CR 引入） | 不计入本 CR 缺陷；CR-039 CP7 期间 git stash 对拍已独立确认基线同样失败；沿用 follow-up 台账 CR-039-FU-P1（candidate），本 CR 台账登记沿用项不重复建项 |

无 BLOCKING 问题。未发现任何需要回修（NEEDS_REWORK）或设计澄清（NEEDS_DESIGN_CLARIFICATION）的事项。

## 12. 剩余风险（汇入 CP8 Decision Brief）

| 风险 ID | 描述 | 缓解 / 触发条件 | 分流建议 |
|---|---|---|---|
| R-1（RA-047-01 / OPI-047-01） | 负数 post_delay 原样透传（`-1` / `"-1"` -> -1.0，与工作区实测一致的固化语义）；当 \|负值\| >= OP_INTERVAL_SEC 时 `time.sleep` 总和为负抛 ValueError | 单测 `test_parse_post_delay_negative_passthrough` 锁定语义；正常用例不会传负值（文档口径 "5s"/"500ms"/纯数字秒）；clamp 记 BACKLOG deferred（post_delay 语义扩展） | risk_acceptance + deferred（CP8 确认） |
| R-2 | post_delay 真机 tear-down 等待效果未实测（static-only 决策内） | 单测锁定叠加行为 + 与工作区已实测基线（26/28 step PASS 载体）逐字一致；真机实测并入既有真机回归候选（BL-039-02 / CR-039-FU-RUNTIME），需独立 runtime_authorization | runtime_authorization（本 CR 不授权） |

**不授权项**：真机下发（trex 发流 / SSH 重启 trex-stl-te / sw3 telnet 实连）、凭据读取/明文落盘、git commit/push，CP7 通过不构成对上述任一项的授权。

## 13. 阶段决策与 CP8 输入

- **阶段决策：PASS_WITH_RISK**。STORY-047-01 全部满足验收标准：量化验收 4 条核验通过；BLOCKING 维度（独立性复跑 / 验收覆盖 / 安全合规）全部通过。
- **CP8 输入**：① 风险 R-1/R-2 汇入 CP8 Decision Brief（分流建议见 §12）；② 不授权项清单（真机 / 凭据 / git）；③ BL-039-01 / CR-039-FU-047 关闭确认（CR-047 回源已完成，台账项应置 closed/superseded）；④ 存量失败 P-1 沿用提示（非本 CR 归属）。
- 追溯：TEST-MATRIX N/A（同步回源类 CR）；TEST-REPORT/REVIEW 以本报告 + CP7 result 为等价质量证据（独立复跑即评审载体，低风险单 Story 回源不另开全文评审）。

---

**验证执行证据（CR-047）**：调度事件 `ADE-CR047-META-QA-001`（AGENT-DISPATCH-LEDGER）；deny-default 展开已登记 READ-EXPANSION-LEDGER（RE-20260819T103000Z0000-c047f001~004）；机器真相源 `process/checks/CP7-CR047.result.json`。
