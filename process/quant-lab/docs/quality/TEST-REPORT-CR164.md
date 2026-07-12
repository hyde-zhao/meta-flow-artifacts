---
status: complete
version: "1.0"
change_id: "CR-164"
decision: "PASS_WITH_RISK"
created_at: "2026-07-12T21:35:00+08:00"
---

# CR-164 Test Report

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-12 | host-orchestrator-inline | 新鲜 CP7 测试结果与全仓差异分类 |

## 执行结果

- CP7 affected regression：`109 passed in 0.98s`。
- Python compile checks：PASS。
- `git diff --check`：PASS。
- 新增测试 provenance：7/7。
- 全仓观察值：`1935 passed, 14 failed`；没有把这 14 项计入 CR-164 PASS，作为独立剩余风险保留。

## 覆盖维度

测试覆盖正常、负向、边界、确定性、输入篡改、证据缺失、方法冲突、权限越界、CR155 历史阻断和相邻 admission projection 回归。未执行真实数据、网络、NAS、QMT、broker、simulation、paper/live 或 Git remote 操作。

## 判定

功能测试 `PASS`；因 inline QA 独立性限制与全仓既有 14 项失败，阶段结论为 `PASS_WITH_RISK`。

