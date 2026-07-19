---
feature_id: "FEAT-AW-05"
change_id: "CR-051"
story_ids: ["ST-AW-005"]
validation_mode: "mixed"
status: "ready-for-story-design"
version: "1.0"
---

# FEAT-AW-05 测试计划

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-18 | meta-se-critical | 建立 manifest 完整性、read-scope、deny-mutation、hash/count、rollback handoff 与 O-AW-03 阈值测试。 |

## 1. 范围与门禁

验证模式为 mixed：schema/unit tests + 临时目录 read-only integration fixture + filesystem/Git/command mutation spy。不得使用真实 project migration、真实 worktree/ref 或 remote mutation作为默认验证。

Entry：DESIGN 与 ST-AW-005 technical note 经 CP5 确认；route/worktree read-only contract 可 fixture；目标路径为隔离临时目录。

Exit：所有 P0/P1 用例通过；manifest 必需分区覆盖率 100%；文件/link/worktree/ref/remote mutation=0；越界读取=0；三阈值边界判定 100% 一致；sync helper create/enable/invoke=0。

## 2. 测试矩阵

| Test ID | 层级 | 场景 | 预期 | 覆盖 |
|---|---|---|---|---|
| TP-AW05-001 | schema | 完整 legacy/project-first fixture | manifest 必需分区/字段 100% 存在，可 canonical serialize/readback | REQ-AW-014/015 |
| TP-AW05-002 | integration | 普通文件、目录、symlink mapping | count/size/hash/link text 与 fixture 一致 | TC-AW-012/013 |
| TP-AW05-003 | negative | source 不可读/EACCES | MANUAL_REVIEW/BLOCKED，列出错误；不报 READY | TC-AW-015 |
| TP-AW05-004 | negative | broken/越界 symlink | 不跟随；记录 link plan 与风险；sibling read=0 | NF-AW-005 |
| TP-AW05-005 | isolation | sibling project/quant-lab sentinel | 访问计数=0，manifest 不泄漏内容/hash | scope boundary |
| TP-AW05-006 | identity | project/route/worktree identity 错配 | BLOCKED，枚举/mutation=0 | REQ-AW-014 |
| TP-AW05-007 | readiness | mapping conflict/missing target/readiness unknown | 非 READY，逐项 reason 与 manual step | TC-AW-015 |
| TP-AW05-008 | handoff | proposed link/worktree/ref plan | 每步含 precondition/auth/validation/rollback；executed=false | REQ-AW-015 |
| TP-AW05-009 | deny mutation | 正常 preflight | write/move/copy/delete/chmod/symlink/worktree/ref/commit/push调用全为0 | RULE-AW-07 |
| TP-AW05-010 | portability | 不同临时根、相同 relative layout | portable mapping/digest一致；absolute diagnostic不入 contract | NF-AW-002 |
| TP-AW05-011 | metrics | 每周2次/4周 | T1=false | O-AW-03 |
| TP-AW05-012 | metrics boundary | 每周3次且连续3周 | T1=false | O-AW-03 |
| TP-AW05-013 | metrics boundary | 每周3次且连续4周 | candidate=true；helper调用=0 | O-AW-03-T1 |
| TP-AW05-014 | metrics boundary | median=10min / 10min+epsilon | 前者false，后者true；只 candidate | O-AW-03-T2 |
| TP-AW05-015 | metrics boundary | avoidable block rate=5% / >5% | 前者false，后者true；只 candidate | O-AW-03-T3 |
| TP-AW05-016 | metrics negative | window incomplete/category unknown | insufficient-data，不生成阈值满足结论 | O-AW-03 |
| TP-AW05-017 | combined | 三阈值任一/多项满足 | 单个 deduplicated follow-up candidate，列出全部 reason | O-AW-03 |
| TP-AW05-018 | regression | preflight 连续执行两次 | 相同输入 manifest content digest 一致，mutation=0 | idempotency |

## 3. Fixture 与观测策略

- 临时 legacy/project-first tree 包含普通文件、空目录、Unicode 名、相对/绝对/broken symlink、EACCES 节点和 sibling sentinel。
- filesystem spy 覆盖 open flags、write、rename、replace、unlink、mkdir、chmod、symlink；Git/command spy 拒绝 worktree/ref/add/commit/push 等命令。
- metrics factory 生成 ISO week buckets、duration samples、blocker categories、missing records，并使用固定时钟。
- before/after snapshot 比较 inode/type/content/link target/Git refs，证明目标 fixture 无变化。

## 4. 证据要求

保存 canonical manifest、schema validation、before/after snapshots、mutation/command spy counts、scope sentinel counts、hash/count expected-vs-actual、read error list 和 threshold table。不能只以“代码没有显式调用 write”证明只读，必须给出 spy 与快照双证据。

## 5. 剩余风险与 Gotchas

不同操作系统的 symlink/permission 行为可能不同；平台差异应让 readiness 降级为 manual review，而不是放宽 scope。达阈值用例只验证 candidate 数据，不应 mock 出可调用 sync helper，否则会在测试 contract 中意外引入实现授权。
