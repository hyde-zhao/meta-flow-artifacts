---
status: "pass-with-risk"
version: "1.0"
cr_id: "CR-080"
created_at: "2026-06-17T07:14:53+08:00"
---

# TEST-REPORT CR080

## Summary

| 项目 | 内容 |
|---|---|
| 测试结论 | PASS_WITH_RISK |
| 范围 | CR080 local data/reports restore/copy |
| validation_mode | mixed / metadata-only |
| 阻断缺陷 | 0 |
| HIGH 缺陷 | 0 |
| 主要风险 | metadata-only 不证明内容 hash / 内容质量 |

## Checks

| Check ID | 检查项 | 结果 | 证据 |
|---|---|---|---|
| T-CR080-01 | CR tracking consistency | PASS | `CR tracking consistency: PASS` |
| T-CR080-02 | CP2 human gate | PASS | `decision_count=5` |
| T-CR080-03 | CP3 human gate | PASS | `decision_count=5` |
| T-CR080-04 | CP5 human gate | PASS | `decision_count=5` |
| T-CR080-05 | Preflight source/target/space/denylist | PASS | `process/checks/CR080-DATA-REPORTS-RESTORE-PREFLIGHT-2026-06-17.md` |
| T-CR080-06 | Copy execution | PASS | `process/checks/CR080-DATA-REPORTS-RESTORE-EXECUTION-2026-06-17.md` |
| T-CR080-07 | Source/target metadata match | PASS | `process/checks/CP7-CR080-DATA-REPORTS-RESTORE-VERIFICATION-DONE.md` |
| T-CR080-08 | `rsync -an --stats` post-copy quick-check | PASS | 0 created / 0 deleted / 0 transferred |
| T-CR080-09 | Forbidden operations absent | PASS | No content read, checksum, NAS/provider/lake/catalog, remote/runtime, credential read |

## Result Detail

| 范围 | 源 | 目标 | 结果 |
|---|---:|---:|---|
| `reports/` files | 772 | 772 | PASS |
| `reports/` dirs | 126 | 126 | PASS |
| `reports/` bytes | 4,529,789,958 | 4,529,789,958 | PASS |
| `data/` files | 8 | 8 | PASS |
| `data/` dirs | 10 | 10 | PASS |
| `data/` bytes | 61,723,552 | 61,723,552 | PASS |
| target denylist count | N/A | 0 | PASS |

## Skipped / N/A

| 项 | 结果 | 原因 |
|---|---|---|
| 内容读取 / 样本解析 | N/A | CR080 明确不授权。 |
| checksum / hash | N/A | CR080 明确不授权。 |
| NAS compare | N/A | 本轮只使用 local legacy retained assets。 |
| Provider rebuild / lake / catalog | N/A | 不授权。 |
| Remote Git / runtime / CR046 | N/A | 不授权。 |

## Conclusion

CR080 copy result is usable for target root daily workflow with accepted metadata-only risk. It is not a content-level integrity proof.
