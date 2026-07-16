# CR-050 Migration

无需业务数据或过程 schema 迁移。现有 `meta-flow cr bootstrap` 行为保持不变；新生命周期是四个显式 additive 命令。

采用新流程的 operator 顺序为：open → 手工选择并 commit 改动 → publish → 独立授权 merge → fresh 授权 finish。旧的外部 merge + 手工清理流程可以继续存在，但不得伪装为已通过 CR-050 的 paired projection/finish evidence。

真实托管平台若强制 PR/merge queue、squash 或 rebase，MVP 会 fail closed；需另开 forge receipt/adapter CR，不得用 patch similarity 猜测已合并。
