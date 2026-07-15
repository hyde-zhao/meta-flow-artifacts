---
title: "CR-170 修复摘要"
status: "cp7-remediation-complete"
version: "1.0"
cr_id: "CR-170"
created_at: "2026-07-15T16:12:00+08:00"
owner: "host-orchestrator inline meta-qa"
---

# CR-170 修复摘要

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-15 | host-orchestrator inline meta-qa | 记录首次 full suite 6 个失败的治理整改、验证与非语义边界。 |

## 1. 已修复

| Finding | 修复 | 验证 |
|---|---|---|
| F-CR170-001 | 5 个 CP3 专题设计文件移至既有 design archive，回填 approved 状态与 archive refs | design surface PASS |
| F-CR170-002 | `test_cr170_*` 改为 `test_canonical_reliability_regression.py`；3 个新测试登记 provenance | taxonomy/provenance PASS |
| F-CR170-003 | hygiene checker 增加 CR170 active source/Feature scoped 分类及测试 | unclassified=0；governance 27/27 |

## 2. 语义影响

治理整改不修改 21-unit inventory、五态分类、Gate 1-5 consumer、admission truth table、public API、adapter、aggregate 或 runner。S04 LLD 从 1.1 更新到 1.2，仅反映最终测试文件 taxonomy；设计差异=`0`。

## 3. 最终结果

最终完整 `uv run --python 3.11 pytest -q` 为 `2195 passed / 0 failed`；py_compile、diff-check 和 forbidden production diff 均 PASS。没有失败豁免，也没有未知归因。

## 4. 未授权动作

未执行真实数据/provider/NAS/credential、runtime/QMT/trading、aggregate、Stage3、CR155 promotion、publish/deploy/tag/release、commit 或 Git remote write。
