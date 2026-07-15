---
cr_id: "CR-047"
checkpoint: "CP7"
rework_required: false
---

# CR-047 Fixes and Residuals

已完成：State/CR/CURRENT 语义校验、CR-033 candidate/index retention、legacy YAML 移除、route metadata 幂等、internal docs canonical 收敛、Doctor lifecycle/read 分类、legacy correction、root wrapper/cache guardrail、Ruff 全清、三平台非交互文档与测试、CR-046 protected-object firewall。

本轮 CP7 未发现需回修代码或重开 CP5 的问题。剩余项全部是已披露风险/后续授权边界：独立 QA、平台 receipt、token telemetry、真实 pilot、远端 commit/push，以及历史 reference-only warning。它们不能通过本 CR 内伪造证据关闭，并将进入 CP8 Decision Brief。

## Post-approval pre-publication correction

- Finding：CP8 向 Gate ledger 追加 CR-047 事件后，旧 selector 因事件正文提到 `CR-046 protected original` 而把无关事件误归入 CR-046 集合，产生 2 个 false-positive firewall findings。
- Fix：manifest verifier 对已有清单按精确 `event_id` 选择对象；新 manifest builder 只消费结构化 CR 字段或事件 ID，不再搜索完整序列化正文。
- Negative guarantee：修改既有受保护事件内容仍会触发 hash finding；新增无关事件即使文字提到 CR-046 也不会改变受保护集合。
- Verification：新增 regression fixture；全量 `401 passed, 70 subtests passed`；57 protected objects，findings=0；Ruff/guardrail 通过。
- Gate impact：属于已批准对象身份契约的 pre-publication correction，不改变 CP8-DQ-01..04、授权边界或 `READY_WITH_RISK` 上限。
