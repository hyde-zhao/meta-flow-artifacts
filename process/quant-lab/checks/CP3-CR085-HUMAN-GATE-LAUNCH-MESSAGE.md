请审查：process/checkpoints/CP3-CR085-REMOTE-CLEANUP-HLD-REVIEW.md
自动预检结论：PASS
Context Capsule 摘要：process/context/CP3-CR085-REMOTE-CLEANUP-CONTEXT.yaml（read_profile=compact，status=ready）
本轮待人工决策项：5
决策收集覆盖：已扫描 STATE、CR085 formal CR、CP3 自动预检、只读远端扫描和 CP2 边界；候选问题 14 项，纳入待决策 5 项；discussion log N/A。
如果你回复 approve，表示你接受以下 5 项推荐方案，不表示授权以下不授权项。
待人工决策清单：
| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP3-CR085-01 | architecture | cleanup 架构是否用 clean candidate？ | 采用，先生成候选再审查。 | 直接 push；停止。 | 推荐可审计。 | 发布延后。 |
| DQ-CP3-CR085-02 | security | allow / deny 边界如何定义？ | 保留 `.env.example`；排除 `.env`、`runs/**`、`reports/**`、`data/**`、`process/**`。 | 删除 `.env.example`；放宽 runs。 | 推荐匹配用户边界。 | 影响候选树。 |
| DQ-CP3-CR085-03 | architecture | 当前远端删除如何解释？ | 当前无删除；发现真实禁止路径时普通删除优先。 | 立即删除 `.env.example`；history purge。 | 推荐最小化写入。 | 远端不变。 |
| DQ-CP3-CR085-04 | security | history purge / force 是否进入本轮？ | 不进入。 | 立即纳入；允许 force。 | 推荐保护历史。 | 不能历史层删除。 |
| DQ-CP3-CR085-05 | follow_up_tracking | default branch / tag 是否治理？ | 不治理。 | 同时治理。 | 推荐聚焦 cleanup。 | 默认分支治理未完成。 |
不授权项：
- 不授权 remote deletion、git push、force push、history rewrite、tag 或 default branch governance。
- 不授权上传、读取、打印或保存 .env 或任何凭据。
- 不授权读取 data/reports 内容、NAS/provider/lake/catalog/runtime、CR046 recovery 或 cleanup。
approve
修改: <具体修改点>
reject
