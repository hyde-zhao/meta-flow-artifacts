# CR-047 Rollback

若 CP8 前发现回归，保持 CR-047 active 并按 Story 路由回修；不要修改 CR-046 protected originals。源码回退应按 Story owner 拆分：truth/index、routing、Doctor/correction、guardrail/cache、Ruff、docs/installer、firewall。历史 correction 与 event ledger 是 append-only，不覆盖或删除；错误事件使用 superseding correction。

CP8 批准消息已单独授权两个 `main` 分支的常规 commit/push。若远端留存后需要回滚，应创建新的非破坏性 revert 提交并重新执行双仓验证；本授权不允许 force-push、历史改写、tag/release 或真实运行环境发布。
