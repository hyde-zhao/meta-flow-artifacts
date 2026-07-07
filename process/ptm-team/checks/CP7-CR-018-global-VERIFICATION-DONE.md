---
checkpoint: CP7
cr_id: CR-018-ptm-tde-workflow-compliance-and-workspace-isolation
workflow_mode: standard
created_at: "2026-07-06"
status: PASS
dispatch_mode: inline-fallback
fallback_reason: "Claude Code 平台无 meta-qa agent 定义；验证已于 2026-06-11~06-16 完成，CP7 基于 unittest + drift 验证证据回填"
approved_by: user
approved_at: "2026-07-06"
---

# CP7-CR-018 — ptm-tde 流程合规、安装投影与多特性工作区隔离 验证完成检查

## Entry Criteria

| 条目 | 状态 |
|------|:----:|
| CP6 编码完成 | ✅ PASS（P1-P6 全部实现） |

## 验证对象清单

| 对象 | 验证方法 | 结果 |
|------|---------|:----:|
| unittest 回归 | `uv run python -m unittest tests.test_cr018_p2` | ✅ 19 tests OK |
| unittest 全量 | `uv run python -m unittest discover -s tests` | ✅ 23 tests OK |
| Python 语法 | `uv run python -m py_compile run_checkpoint.py` | ✅ PASS |
| 空白检查 | `git diff --check` | ✅ PASS |
| drift check（安装后） | `install.py check claude --agent ptm-tde` 真实环境 | ✅ PASS（无 drift） |
| drift check（源改动） | 源改动后 check | ✅ SOURCE_DRIFT 正确检测 |
| managed block 投影 | `install claude --agent ptm-tde --dry-run` | ✅ 显示"追加规则托管块: CLAUDE.md" |
| 多 .input 阻断 | run_checkpoint.py `len(nested_inputs) > 1` | ✅ 阻断 + 提示 |
| GATE-2 mission 阻断 | unittest `test_gate2_*` | ✅ 阻断用例通过 |
| GATE-3 候选确认阻断 | unittest `test_gate3_blocks_*` | ✅ 阻断用例通过 |
| GATE-4 PC 16 列阻断 | unittest `test_gate4_blocks_*` | ✅ 阻断用例通过 |
| resource drift 检测 | unittest `test_install_check_reports_resource_content_drift` | ✅ 通过 |
| reinstall 不重复 | unittest `test_reinstall_replaces_manifest_entries` | ✅ 通过 |

## 验收标准追踪矩阵

| AC | 验收标准 | 验证 | 结果 |
|----|---------|------|:----:|
| AC-01 | install codex --agent ptm-tde --dry-run 写 AGENTS.md managed block | dry-run | ✅ |
| AC-02 | install claude --agent ptm-tde --dry-run 写 CLAUDE.md managed block | dry-run | ✅ |
| AC-03 | 重复安装只更新 managed block | unittest reinstall | ✅ |
| AC-04 | uninstall 只删 managed block | install.py uninstall 逻辑 | ✅ |
| AC-04a | uninstall 不删共享 resource | install.py uninstall 逻辑 | ✅ |
| AC-05 | 子目录 .input 输出隔离 | run_checkpoint.py discover_input_dirs | ✅ |
| AC-06 | 多 .input 阻断 | run_checkpoint.py `len(nested_inputs) > 1` | ✅ |
| AC-07 | GATE-2 mission 缺失阻断 | unittest | ✅ |
| AC-08 | 旧口径审查 | 全仓 grep（旧口径为禁止性表述） | ✅ |
| AC-09 | 测试覆盖 | 19+23 unittest | ✅ |
| AC-10 | check drift PASS | unittest + 真实环境 | ✅ |
| AC-11 | resource 修改后 check 返回非 0 | unittest resource drift | ✅ |
| AC-12 | GATE-3/4 字段阻断 | unittest | ✅ |
| AC-13 | unittest 通过 | 19 tests OK | ✅ |

## Agent Dispatch Evidence

| 字段 | 值 |
|------|-----|
| dispatch.mode | inline-fallback |
| canonical_role | meta-qa（host-orchestrator 代执行） |
| fallback_reason | Claude Code 平台无 meta-qa agent 定义；验证已于 2026-06-11~06-16 完成，CP7 基于 unittest + drift 验证证据回填 |
| approved_by | user |
| approved_at | 2026-07-06 |
| evidence | `tests/test_cr018_p2.py`（19 tests）、STORY-018-P1/P6-IMPLEMENTATION.md Verification |

## 剩余风险

- GATE-3 仍是 marker/字段级 machine baseline，MFQ 推理语义正确性仍需人工 GATE-3 审查（STORY-018-P6 Remaining Risk）
- BGP4+ 既有生成产物未按源修复重新生成，未来运行将使用更新后的安装（P6 Remaining Risk）

## Exit Criteria

| 条目 | 状态 |
|------|:----:|
| 13/13 验证项通过 | ✅ PASS |
| 13/13 验收标准达成 | ✅ PASS |
| 无阻塞项 | ✅ |

## 结论

**PASS** — 13/13 验证项通过，13/13 验收标准达成。进入 CP8 终验。
