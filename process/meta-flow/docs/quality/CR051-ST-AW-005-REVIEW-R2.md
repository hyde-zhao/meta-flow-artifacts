---
status: "PASS_WITH_RISK"
version: "2.0"
scope: "CR-051 / ST-AW-005 / independent review R2"
created_at: "2026-07-18T16:36:53Z"
---

# ST-AW-005 独立质量评审 R2

## Findings（按严重度）

当前未发现 BLOCKER、HIGH、MEDIUM 或 LOW 实现 finding。

### REV-AW005-001 — CLOSED-R2

- 原严重度：HIGH / BLOCKING。
- 关闭证据：三类独立临时目录 probe 全部通过；3 条 closure、26 条 directed、95 条 adjacent、280+9 exact combined 全绿。
- 代码判断：`_deny_descendant()` 在 descendant lstat/open/hash/mapping/descend 前同时检查 source 与 mapped-target portable path；`_link_info()` 在 denied target probe 前完成 source/mapped-target prefix 判定并脱敏。
- 泄漏判断：policy pattern 可审计保留；denied descendant 路径的 mapping、内容、hash 和 relative link target 不进入 canonical payload。
- 契约判断：manifest schema、glob 语义、readiness enum、no-follow、O-AW-03 和零 mutation 边界未变，不需要 design delta。

## 人工 / 语义质量审查

修复没有把 denied 子树“静默忽略后继续 READY”，而是留下 `denied-descendant` 及 source/target/symlink reason，并将 readiness 降为 `MANUAL_REVIEW`。这符合 fail-closed 设计。错误信息对后续 operator 可行动，也没有把预检结果表达为迁移授权。

## 剩余风险

以下均为已知、非本 R2 实现缺陷：

- MEDIUM `R-AW005-DIR-ENUM-TOCTOU`：未来不可信并发目录访问需要 dirfd/openat full-lld。
- MEDIUM `CR051-RISK-REAL-MIGRATION-UNVERIFIED`：真实迁移未授权、未执行。
- LOW `CR051-RISK-WINDOWS-NATIVE-NOFOLLOW`：Windows native 行为未实机验证，当前能力缺失时 fail closed。
- MEDIUM `CR051-RISK-REMOTE-UNVERIFIED`：托管 remote、凭据、保护规则与 publication 未授权、未验证。

## 结论

`PASS_WITH_RISK`。允许 Host Orchestrator 组装 CP7-R2 result 并将四项风险汇入 CP8；不授权真实 runtime 或 repository mutation。
