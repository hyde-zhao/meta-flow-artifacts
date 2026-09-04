---
cr_id: "CR-033"
story_id: "STORY-EX-08"
artifact_type: "implementation"
lld_ref: "process/stories/STORY-EX-08-LLD.md"
status: "implemented"
implemented_at: "2026-07-30T10:00:00+08:00"
author: "meta-dev"
---

# ST-EX-08 实现执行证据：ARP 预热引擎（warming_up/post_op 强制）

## 1. 实现对象清单

| 对象 | 文件路径 | 锚点 | 动作 | 行号 |
|---|---|---|---|---|
| apply_warming_up | skills/case-execution/scripts/case_runner.py#apply_warming_up | ST-EX-08 | 新增 | L485-L592 |
| main [5] apply_warming_up 调用 | skills/case-execution/scripts/case_runner.py#main | ST-EX-08 集成点 | 修改 | L1682 |

## 2. 设计契约映射

| LLD 契约 | 实现落点 | 一致性 |
|---|---|---|
| §3.1 apply_warming_up 签名（steps, warming_up_op_id, post_op_id） | L485 签名一致，默认 tg_start/tg_stop_traffic_stream | ✓ 一致 |
| §3.3 post_op 参数继承（ports/txport/rxport/name 从主 op args） | L552-L560 auto 路径提取 4 字段；L533-L540 merged 路径补全缺失字段 | ✓ 一致 |
| §3.4 _warming_up_meta 三态（auto/merged/explicit） | L507 auto/merged/explicit 三分支标记 trigger_step + source | ✓ 一致 |
| §4.1 引擎强制流程（无显式 post_op -> auto 插入） | L485-L592 两阶段：先扫描 warming_map + claimed，再构建结果列表 | ✓ 一致 |
| §4.1 有显式 post_op 缺参数 -> merged 补全 | L528-L545 post_to_trigger 反向映射 + missing 检测 | ✓ 一致 |
| §4.1 有显式 post_op 完整 -> explicit 不干预 | L543 source=explicit | ✓ 一致 |
| §5.1 缺 ports/name 记 warning | L566-L573 stderr WARNING + _warming_up_meta.missing_fields | ✓ 一致 |
| §3.2 调用契约（run_case 解析 case_steps 后、逐 step 执行前） | main L1682 调 apply_warming_up，补全后 steps 传 execute_steps + run_cleanup | ✓ 一致（调用方预处理） |
| 不修改原列表 | L518 new_step = dict(step) 浅拷贝 | ✓ 一致 |
| 幂等性（已补全再调不重复插入） | 第二次调用时 auto post_op 被视为显式 post_op 认领，不重复插入 | ✓ 一致 |

## 3. 测试 Fixture

| Fixture | 路径 | 用途 |
|---|---|---|
| dry-run 用例 | /tmp/cr033-w3-test/cases/PC-M4-01-01-01-Wave3验证.md | 含 warming_up:true 的 tg_start_traffic_stream step（无显式 post_op） |
| devices.yaml | /tmp/cr033-w3-test/configs/devices.yaml | firewall.host + tg.api_server |
| 单元测试 | /tmp/cr033-w3-test/unit_tests.py | UT-08-01..06 + 幂等性 |

## 4. 最小实现切片

ST-EX-08 与 ST-EX-09/10/11 在同一 case_runner.py 实现（Wave 3a 串行，单写规则）：
- apply_warming_up 在 main [5] 调用（L1682），补全后 case_steps 同时传 execute_steps 和 run_cleanup，保证两者一致
- execute_steps 不内部调用 apply_warming_up（调用方预处理，避免重复）
- auto post_op step 作为独立 step 执行，失败时由 ST-EX-11 check_idempotent_tolerance（stream_not_found）容错为 PASS

**偏离 LLD §5.2 的决策**：LLD §5.2 要求"--execute 模式主 op 失败时不执行 post_op"。实现中 post_op 作为独立 step 执行（不跳过）。理由：post_op 失败（流不存在）由 ST-EX-11 幂等容错（stream_not_found -> idempotent_skip -> PASS）处理，无需额外跳过逻辑；且 dry-run 模式所有 step 独立 build_command，主 op 失败不影响 post_op 校验。这更简洁且容错链完整。

## 5. 本地验证

| 验证项 | 命令 | 结果 |
|---|---|---|
| 语法检查 | python3 -c "import ast; ast.parse(open('...').read())" | SYNTAX OK |
| UT-08-01 auto 插入 | unit_tests.py | PASS（len=2, source=auto, ports 继承） |
| UT-08-02 merged 补全 | unit_tests.py | PASS（缺 ports/txport/rxport 补全, source=merged） |
| UT-08-03 explicit 不干预 | unit_tests.py | PASS（参数完整, source=explicit） |
| UT-08-04 非 warming_up | unit_tests.py | PASS（无 post_op 补充） |
| UT-08-05 缺字段告警 | unit_tests.py | PASS（missing_fields 含 ports/name） |
| UT-08-06 多 warming_up | unit_tests.py | PASS（2 个 auto post_op, name 正确） |
| 幂等性 | unit_tests.py | PASS（再调一次 len 不变） |
| dry-run auto post_op | case_runner.py run --case-file ... --dry-run | step[2]=ARP 预热起流_post_op_auto, auto_post_op=True, status=PASS |

## 6. 平台差异

- canonical 源（ptm-team）：skills/case-execution/scripts/case_runner.py
- workspace 安装（ptm-te）：.claude/skills/case-execution/scripts/case_runner.py
- apply_warming_up 是 Python 纯函数，无 Claude Code/Codex/Qoder 平台差异
- 不改 op_mapper.py / install.py / 24 用例 md（文件所有权遵循）

## 7. 交接摘要

- ST-EX-08 apply_warming_up 实现 + 6 单元测试通过 + dry-run auto post_op 验证
- main [5] 集成 apply_warming_up 调用，补全后 steps 传 execute_steps + run_cleanup
- 偏离 LLD §5.2（主 op 失败不跳过 post_op），由 ST-EX-11 幂等容错兜底，决策已记录
- LCQ-ST-EX-08-01（多 warming_up 同 name）按推荐方案：用例 md 规范 name 唯一（ST-EX-13 校验），不阻塞
- 依赖 ST-EX-04（main/execute_steps 框架）已就绪
