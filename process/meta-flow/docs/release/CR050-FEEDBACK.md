# CR-050 Feedback

建议回流以下信号：

- remote default 或 CR tip 在 plan/execute 间漂移的 false positive/negative；
- branch protection、merge queue 与 ordinary exact-OID push 的拒绝语义；
- artifact-first PARTIAL 恢复中的 projection 误推进；
- recovery ref 命名冲突、remote absent 幂等性与 cleanup resume；
- ref allowlist 对合法 Unicode/组织命名约定的兼容性；
- operator 在 open 后忘记 commit、在 publish 时遇到 dirty tree 的可用性反馈。

后续候选不自动启动：forge-signed merge receipt、PR/merge-queue adapter、squash/rebase 可验证证明、真实双仓 pilot、独立 QA/platform receipt，以及 CR-001..033 历史 index schema 修复，均需要独立 CR/授权。
