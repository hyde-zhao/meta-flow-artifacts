# CR-056 Follow-up 跟踪台账

> 2026-09-04。来源：CP8 分流（CP7 验证 PASS_WITH_RISK）。状态取值：`candidate` / `active` / `blocked` / `spike_candidate` / `converted-to-spike` / `closed` / `cancelled` / `superseded`。只有用户决定推进某项时才创建正式 CR。

| 编号 | 类型 | 摘要 | 建议路由 | 状态 |
|------|------|------|---------|------|
| FU-056-01（R1） | 测试资产 | CR-056 冒烟脚本 6 项中间态断言过期（骨架→实体化演进，非产品缺陷；最终态由 118 pytest 锁定） | 不推进：/tmp 临时脚本不入库，断言蓝本已落盘 test_cr056_mark_lifecycle.py | `closed` |
| FU-056-02（R2） | 风险接受 | HLD G-01 口径 312 文件 vs 实际 311 用例 + 1 README（discover_cases 正确排除 README） | 修订 HLD §13 G-01 口径为 311 用例（文档级，随下次 CR 或直接改） | `candidate` |
| FU-056-03（R3） | 测试资产 | test_install_mapping.py 未含规则 14~17/版本断言（渲染校验当前靠 CP7 逐字节比对人工补足） | 后续 CR 补断言（低成本，可与任一 install 相关 CR 合并） | `candidate` |
| FU-056-04（R4） | 测试资产 | archive-check 正式 pytest 未落盘（08 实体化仅 /tmp 冒烟 30 断言） | 后续 CR 把 archive-check 断言并入 test_cr056 或独立 test 文件 | `candidate` |
| FU-056-05（O-056-06） | 数据回填 | 存量 311 条 te 用例作者元数据四字段未回填（archive-check 不强制存量未动文件） | 批量回填 CR（git 历史推导创建人/创建时间 + 当前操作人刷新）；回填后 ledger 全量建档 | `candidate` |
| FU-056-06（DQ-056-06） | 运行授权 | CICD 无人值守 vs DQ-01 单次授权冲突（预授权凭据 / 设备白名单） | M3 前独立拍板（runtime_authorization），本 CR 不阻塞 | `candidate` |

## 状态摘要

- 已关闭：FU-056-01
- 候选推进（需用户决定才转正式 CR）：FU-056-02/03/04/05/06
- 关联开放：O-056-03（CICD 授权）、O-056-06（存量回填，同 FU-056-05）
