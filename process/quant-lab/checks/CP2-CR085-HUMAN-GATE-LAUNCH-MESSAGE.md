请审查：process/checkpoints/CP2-CR085-REMOTE-CLEANUP-BASELINE.md
自动预检结论：PASS
Context Capsule 摘要：process/context/CP2-CR085-REMOTE-CLEANUP-CONTEXT.yaml（read_profile=compact，status=ready）
本轮待人工决策项：5
决策收集覆盖：已扫描 STATE、CR085 formal CR、CP2 自动预检、只读远端扫描和当前用户输入；候选问题 18 项，纳入待决策 5 项；discussion log N/A。
如果你回复 approve，表示你接受以下 5 项推荐方案，不表示授权以下不授权项。
待人工决策清单：
| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP2-CR085-01 | scope | 是否暂停 CR084 直接 push？ | 暂停，CR085 先收敛 clean publication 边界。 | 继续 CR084 push；停止 CR085。 | 推荐避免误推；继续 push 风险高。 | 远端暂不变。 |
| DQ-CP2-CR085-02 | security | env 文件如何分类？ | `.env.example` 保留；`.env` 禁止上传、读取、打印或保存。 | 删除 `.env.example`；允许提交 `.env`。 | 推荐符合用户修正。 | 影响 allowlist。 |
| DQ-CP2-CR085-03 | scope | 当前远端是否删除文件？ | 当前不删除。 | 删除 `.env.example`；history purge。 | 推荐最小化远端写。 | 当前无删除动作。 |
| DQ-CP2-CR085-04 | risk_acceptance | 是否接受 `d4d2881` 不直接 push？ | 接受，先做 clean candidate。 | 强行 push；停止。 | 推荐可控。 | 发布延后。 |
| DQ-CP2-CR085-05 | follow_up_tracking | CR046/data/reports/process 是否纳入？ | 不纳入，独立治理。 | 同步处理。 | 推荐降低范围。 | 后续治理仍未完成。 |
不授权项：
- 不授权 git push、git push -u、remote deletion、remote add/set-url/remove。
- 不授权 fetch 到当前仓库、tag、force push、history rewrite、default branch governance。
- 不授权上传、读取、打印或保存 .env 或任何凭据。
- 不授权 dirty worktree commit、data/reports 内容读取、NAS/provider/lake/catalog/runtime、CR046 recovery 或 cleanup。
approve
修改: <具体修改点>
reject
