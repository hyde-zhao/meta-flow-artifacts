---
handoff_id: "CR172-CP5-LLD-R4-UPPER-CONTRACT-CORRECTION"
change_id: "CR-172"
checkpoint: "CP5"
role: "meta-se-critical"
status: "complete"
completed_at: "2026-07-18T12:44:45+08:00"
review_source: "process/checks/CP5-CR172-LLD-REVIEW-R3-FINDINGS.md"
finding_closed: "F-CR172-CP5-R3-001"
---

# CR-172 CP5 LLD R4 上层合同整改 Return Summary

## 结论

唯一 required finding `F-CR172-CP5-R3-001` 已按最小范围关闭。HLD 与 ADR 已删除 S03 verifier facade 备选，唯一冻结：S04 从 S03 current selected-replica read contract 获取 payload，并直接调用 S02 唯一 verifier-library 做 bytes-level 原 seal 复验。

## 修改文件（3）

1. `docs/design/HLD-TRIAL-RETURN-DEPLOYMENT-CONTRACTS.md`（v1.4）
2. `docs/design/ARCHITECTURE-DECISION-TRIAL-RETURN-DEPLOYMENT-CONTRACTS.md`（v1.4）
3. 本 Return Summary

## 合同闭环

| 检查项 | 冻结结果 |
|---|---:|
| S03 verifier facade 允许项 | `0` |
| S03 新增 digest / verifier facade / data bypass | `0/0/0` |
| S04 direct S02 verifier | `1` |
| S04 payload 数据来源 | 仅 S03 current selected-replica read contract |
| receipt-only trust / secondary digest / re-seal | `0/0/0` |

## 未改变边界

- schema、canonical bytes/hash domain、Story DAG、file owner、测试计数：变化=`0`。
- 六类授权、授权资格 DAG、claim ceiling、runtime/真实数据/NAS/信号/交易边界：变化=`0`。
- Feature、LLD、Story、Development Plan、源码、测试、fixture、state、ledger、checkpoint：修改文件数=`0`。
- 新增 OPEN/Spike/用户决策项=`0/0/0`。

## 验证

- 两份上层合同均固定 `S03 verifier facade allowed=0`、`S03 new digest/verifier facade/data bypass=0/0/0`、`S04 direct S02 verifier=1`。
- 限定文件 `git diff --check`：PASS（exit code `0`）。
- 允许备选残留扫描：`0` matches（`rg` exit code `1`，表示未发现匹配）。
