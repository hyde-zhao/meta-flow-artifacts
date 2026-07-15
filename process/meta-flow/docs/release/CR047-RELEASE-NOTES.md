---
cr_id: "CR-047"
release_decision: "READY_WITH_RISK"
release_artifact_profile: "full"
---

# CR-047 Release Notes

本候选交付把 workflow truth 收敛到 State v2、CR JSON catalog 与 CURRENT 的 source-owned relation graph；修复 process 外置路由幂等性、clean-clone guardrail、Doctor 历史分类、Ruff 基线和三平台非交互安装说明，并增加 CR-046 受保护原件的对象身份 hash firewall。

质量基线：7/7 CP6 PASS；7/7 CP7 PASS_WITH_RISK；CP8 后 publication preflight 新增对象身份回归并达到 401 tests + 70 subtests；Ruff 0；五门 exit 0；三平台 dry-run 3/3；Doctor observed=21/blocker=0/unclassified=0；CR-046 57 个 protected objects 在 CP6/CP7/最终复核均 0 差异。

兼容性：无命令参数、安装路径或持久化 schema 的 breaking change。State/CR/CURRENT 新增语义检查可能把过去机械通过的 stale workspace 正确阻断。用户在 CP8 批准消息中另行授权 `meta-flow` 与 `meta-flow-artifacts` 的配对 commit/push；该 Git 留存不代表真实安装、外部 runtime、凭据、production write/publish 或平台 attestation。
