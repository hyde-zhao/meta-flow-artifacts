---
handoff_id: "CR172-CP5-LLD-R4-UPPER-CONTRACT-CORRECTION"
change_id: "CR-172"
checkpoint: "CP5"
canonical_role: "meta-se"
codex_agent_name: "meta-se-critical"
reasoning_profile: "critical"
context_ref: "process/context/CP5-CR172-PATH-I-LLD-CONTEXT.yaml"
r3_finding_ref: "process/checks/CP5-CR172-LLD-REVIEW-R3-FINDINGS.md#F-CR172-CP5-R3-001"
status: "ready"
---

# CR-172 CP5 LLD R4 上层合同最小整改

只关闭 `F-CR172-CP5-R3-001`，不得扩展范围：

1. 在 HLD §10.2/§11.4 删除“S03 窄 verifier facade”备选；唯一冻结为：S04 的数据只来自 S03 current selected-replica read contract，bytes 验证只由 S04 直接调用 S02 唯一 verifier-library。
2. 在 ADR-004 删除同一 facade 备选并同步唯一依赖方向。
3. HLD/ADR 各追加修订记录，升级补丁版本；不得改变 schema、hash domain、Story DAG、文件 owner、测试数量、授权或 claim ceiling。
4. 证明 `S03 verifier facade` 允许项=`0`、S03新增 digest/verifier facade/data bypass=`0/0/0`、S04 direct S02 verifier=`1`，并执行限定文件 `git diff --check`。

允许写入：

- `docs/design/HLD-TRIAL-RETURN-DEPLOYMENT-CONTRACTS.md`
- `docs/design/ARCHITECTURE-DECISION-TRIAL-RETURN-DEPLOYMENT-CONTRACTS.md`
- `process/handoffs/CR172-CP5-LLD-R4-UPPER-CONTRACT-CORRECTION-META-SE-CRITICAL-RETURN-SUMMARY.md`

禁止修改 Feature、Story、LLD、Plan、源码、测试、fixture、state、ledger、checkpoint；禁止任何真实数据湖/NAS/runtime/R/信号/交易/部署/remote 操作。
