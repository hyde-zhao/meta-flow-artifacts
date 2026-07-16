# CR-050 Rollback

CP8 前若发现回归，保持 CR-050 active，按 ST-GB-001→002→004→003 的 owner/依赖路由回 CP6 或 CP5。代码回退边界是新 lifecycle module、`git_sync.py` adapter、`cr_lifecycle.py` 路由和对应 tests；不改写 CP result 或 ledger，使用 superseding result/correction。

本轮没有执行真实远端操作，因此当前没有 remote ref 需要回滚。未来若显式 merge 发生单仓成功，不允许 reset/force 回滚已发生事实；应保持 PARTIAL、保留 2/2 CR branches、fresh observe 后继续收敛。finish 会先创建 local recovery refs；这些 recovery refs 不自动 push 或删除。

若源码已发布后需要回退，创建正常 revert 提交并重新执行全量验证；禁止 force-push、历史改写或复用旧 typed authorization。
