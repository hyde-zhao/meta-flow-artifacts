请审查：process/checkpoints/CP5-CR085-REMOTE-CLEANUP-READINESS.md
自动预检结论：PASS
Context Capsule 摘要：process/context/CP5-CR085-REMOTE-CLEANUP-CONTEXT.yaml（read_profile=compact，status=ready）
本轮待人工决策项：5
决策收集覆盖：已扫描 STATE、CR085 formal CR、CP5 自动预检、CP2/CP3 门禁和只读扫描；候选问题 15 项，纳入待决策 5 项；discussion log N/A。
如果你回复 approve，表示你接受以下 5 项推荐方案，不表示授权以下不授权项。
待人工决策清单：
| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP5-CR085-01 | runtime_authorization | CP5 是否授权远端写入？ | 不授权远端写入，仅确认策略。 | 授权 push；授权删除。 | 推荐风险最低。 | 远端仍不变。 |
| DQ-CP5-CR085-02 | implementation | approve 后下一步允许做什么？ | 只准备 clean publication manifest / candidate 供审查。 | 直接 push；直接删除；停止。 | 推荐可验证。 | 需要后续 review。 |
| DQ-CP5-CR085-03 | security | `.env` 如何处理？ | 不读取、不上传、不打印、不保存。 | 读取检查；允许上传。 | 推荐符合用户要求。 | 路径级禁止。 |
| DQ-CP5-CR085-04 | risk_acceptance | `.env.example` 是否保留？ | 保留为模板，不删除。 | 删除；history purge。 | 推荐符合用户修正。 | 模板需保持无真实密钥。 |
| DQ-CP5-CR085-05 | security | high-risk Git 操作是否禁止？ | 禁止 force、history rewrite、tag、default branch governance。 | 允许 force/history；允许 tag/default branch。 | 推荐保护远端历史。 | 不能历史层删除。 |
不授权项：
- 不授权 git push、git push -u、remote deletion、remote add/set-url/remove。
- 不授权 fetch 到当前仓库、tag、force push、history rewrite、default branch governance。
- 不授权上传、读取、打印或保存 .env 或任何凭据。
- 不授权 dirty worktree commit、data/reports 内容读取、NAS/provider/lake/catalog/runtime、CR046 recovery 或 cleanup。
approve
修改: <具体修改点>
reject
