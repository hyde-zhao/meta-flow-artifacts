# CP5 CR085 Remote Write Readiness 人工门禁发起消息

请审查：`process/checkpoints/CP5-CR085-REMOTE-WRITE-READINESS.md`

自动预检结论：PASS，阻断项 0。

Context Capsule Summary：`process/context/CP5-CR085-REMOTE-CLEANUP-CONTEXT.yaml`，read_profile=compact。

决策收集覆盖：已扫描 5 个来源，发现候选问题 12 个，纳入待决策 4 个；N/A / 缺失来源 0 个，原因见 checklist 的 Decision Collection Coverage。

本轮待人工决策项：4。

如果你回复 approve，表示你接受以下 4 项推荐方案，不表示授权以下禁止操作。

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP5-CR085-WRITE-01 | runtime_authorization | 是否授权一次受控 remote write？ | 授权 exact sequence：只读 remote head check -> 临时 clone/worktree -> 应用 v6 candidate -> 普通 fast-forward push 到 master。 | 只生成 dry-run diff；暂停 CR085。 | 推荐完成远端同步；dry-run 更保守但不完成发布。 | 会改变远端 master。 |
| DQ-CP5-CR085-WRITE-02 | security | 是否接受 v6 candidate 的扫描结果？ | 接受 PASS_WITH_RISK：blocker=0，4 个测试 fixture warning 非阻断。 | 删除 warning 测试并重扫；暂停。 | 推荐保留测试覆盖；删除测试降低覆盖。 | 测试中仍有示例私网 IP / tmp fixture。 |
| DQ-CP5-CR085-WRITE-03 | implementation | 是否禁止使用当前 dirty worktree？ | 禁止；仅用临时 clone/worktree 和 v6 candidate。 | 使用当前分支/当前工作区直接 push。 | 推荐避免混入未审查脏文件。 | 当前未跟踪文件不会进入发布。 |
| DQ-CP5-CR085-WRITE-04 | security | 高风险 Git 操作是否继续禁止？ | 禁止 force、history rewrite、tag、default branch governance、remote deletion。 | 授权 force/history/tag/default branch。 | 推荐风险最低；备选不可逆或扩大治理范围。 | 历史层面不会清除既有对象。 |

不授权项：

- 不授权 force push、history rewrite、tag、default branch governance。
- 不授权 remote deletion。
- 不授权读取、打印、保存、复制或上传 `.env`。
- 不授权提交当前 dirty worktree。
- 不授权 data/reports 内容读取、NAS 内容操作、provider/lake/catalog、runtime、CR046 recovery、old root retirement 或 cleanup。

请回复以下三者之一：

approve

修改: <具体修改点>

reject
